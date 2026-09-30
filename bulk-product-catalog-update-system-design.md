# Bulk Product Catalog Update — System Design

## Overview

For **bulk-updating an entire product catalog of up to ~200K items**, treat this as a **durable asynchronous data-processing workflow**, not as a giant API request or database transaction.

The core design principle is:

> **The user submits an intent once; the platform materializes that intent into bounded, retryable batches and processes them safely.**

## High-Level Architecture

```text
UI / API
   |
   | POST /catalog/bulk-updates
   v
Bulk Update Service
   |
   +--> Bulk Job DB
   |       job_id
   |       criteria / product snapshot
   |       requested mutation
   |       status / progress
   |
   +--> Resolve affected products
   |
   +--> Work Queue
           |
           +--> Batch 1: products 1..500
           +--> Batch 2: products 501..1000
           +--> ...
           +--> Batch 400
                     |
                     v
              Bulk Update Workers
                     |
              Product Domain APIs / DB
                     |
          +----------+-----------+
          |                      |
     Catalog DB             Events / Outbox
                                |
                   Search / Tax / Inventory /
                   Integrations / downstream
```

For **200K records**, something like **200–1000 products per work item** is reasonable. With 500 products per batch, that's only ~400 queue messages.

---

## 1. API Should Create a Job, Not Perform the Updates

Example:

```http
POST /v1/products/bulk-updates
```

```json
{
  "selection": {
    "type": "filter",
    "filter": {
      "status": "ACTIVE",
      "categoryIds": ["cat-123"]
    }
  },
  "operation": {
    "type": "SET_TAXABLE",
    "taxable": true
  },
  "idempotencyKey": "client-generated-123"
}
```

Response:

```json
{
  "jobId": "bulk_918273",
  "status": "QUEUED",
  "estimatedCount": 186421
}
```

Another example:

```json
{
  "operation": {
    "type": "GENERATE_BARCODE",
    "format": "EAN13"
  }
}
```

Or multiple compatible changes:

```json
{
  "operation": {
    "type": "PATCH",
    "changes": {
      "taxable": true,
      "salesCategoryId": "abc"
    }
  }
}
```

Avoid exposing arbitrary SQL-like updates. Bulk operations should pass through the product domain's invariants.

---

## 2. Define What “All Products” Means

Suppose the user clicks:

> Select all 187,283 products → Mark taxable.

Then processing takes five minutes.

During those five minutes:

- 500 products are created
- 20 products are deleted
- 1,000 products stop matching the original filter

Do those products participate?

Use explicit semantics.

A strong default is **snapshot-at-submission semantics**:

```text
"Update the products that matched this selection when I clicked Apply."
```

Rather than:

```text
"Continuously evaluate this filter while the job runs."
```

### Option A — Materialize IDs

```text
bulk_job_item
-------------
job_id
product_id
status
error_code
version_at_selection
```

At 200K rows, this is a manageable amount of data.

Advantages:

- deterministic
- easy retry
- exact progress
- exact failure reporting
- auditability
- straightforward cancellation

For 200K scale, this is the preferred design.

### Option B — Snapshot Cursor

```text
WHERE filter_conditions
  AND product_id > last_product_id
  AND created_at <= job_created_at
ORDER BY product_id
LIMIT 500
```

This saves storage, but semantics become more complicated when filter attributes change while the job is running.

For financially important systems, prefer **materializing the target IDs**.

---

## 3. Job State Model

```text
bulk_update_job
----------------
id
company_id
operation_type
operation_payload
selection_definition
status
total_count
processed_count
success_count
failure_count
created_by
created_at
started_at
completed_at
cancel_requested
idempotency_key
```

Suggested states:

```text
CREATED
   ↓
RESOLVING_TARGETS
   ↓
QUEUED
   ↓
RUNNING
  ↙   ↘
COMPLETED   COMPLETED_WITH_ERRORS
    \
     CANCELLED
```

Avoid just:

```text
SUCCESS / FAILURE
```

because partial success is normal in bulk systems.

---

## 4. Do Not Make 200K Updates One Transaction

Avoid:

```sql
BEGIN;

UPDATE products
SET taxable = true
WHERE ...  -- 200K rows

COMMIT;
```

Even if the database can technically handle it, this creates undesirable operational properties:

- long-running locks
- huge WAL / replication load
- transaction log pressure
- difficult cancellation
- one bad record can fail everything
- large rollback cost
- replication lag
- downstream CDC bursts

Instead:

```text
Job
 ├─ Batch 1 → transaction
 ├─ Batch 2 → transaction
 ├─ Batch 3 → transaction
 ...
 └─ Batch N → transaction
```

Even within a batch, support **record-level error isolation**.

Example:

```text
Batch 137
  497 succeeded
  3 failed validation
```

Rather than failing all 500.

---

## 5. Mutation Workers Must Be Idempotent

Most queue architectures provide effectively **at-least-once delivery**.

Example:

```text
worker processes batch
        ↓
DB commit succeeds
        ↓
worker crashes before ACK
        ↓
same message delivered again
```

Therefore:

```text
(job_id, product_id, operation_version)
```

should effectively identify an execution.

Example:

```sql
bulk_job_item
PRIMARY KEY (job_id, product_id)
```

Worker behavior:

```text
if item.status == SUCCESS:
    skip
```

Operations like:

```text
taxable = true
```

are naturally idempotent.

But **generate barcode** is not automatically idempotent.

Bad:

```text
retry 1 → generate 938472...
retry 2 → generate 483728...
```

Instead persist the generated value:

```text
bulk_job_item
--------------
product_id
generated_barcode = "890123456789X"
```

Then retries apply **the same barcode**.

---

## 6. Barcode Generation Needs Special Handling

If each product must get a unique barcode, avoid:

```text
worker:
  generate random barcode
  update product
```

because parallel workers can collide.

### Preferred Approach — Sequence-Backed Barcode

```text
barcode_sequence
      ↓
reserve block 10000–10999
      ↓
worker deterministically converts sequence → barcode
```

This scales well.

Also enforce database uniqueness:

```sql
UNIQUE(company_id, barcode)
```

Example:

```text
products
-------
id
barcode

UNIQUE(company_id, barcode)
```

If the barcode format has a checksum:

```text
sequence: 123456789012
checksum(): 8

barcode:
1234567890128
```

The **barcode value should be generated exactly once per job-item**, persisted, then reused by retries.

---

## 7. Concurrency With Normal Edits

Example:

```text
T0: bulk job selects product A
T1: employee manually changes product A.taxable = false
T2: bulk worker reaches product A
```

Should the bulk job overwrite the employee's change?

There are two valid approaches.

### Last Writer Wins

The bulk job applies:

```text
taxable=true
```

regardless.

Simple and often acceptable for explicit administrative operations.

### Optimistic Concurrency

Record:

```text
product.version = 17
```

during selection.

When updating:

```sql
UPDATE product
SET taxable = true,
    version = version + 1
WHERE id = ?
AND version = 17
```

If 0 rows are updated:

```text
CONCURRENT_MODIFICATION
```

and report:

> 199,825 updated  
> 175 skipped because they changed after the bulk update started.

For accounting/inventory product data, support this mechanism at least for fields where overwriting concurrent edits is risky.

You can define semantics **per operation**:

```text
Mark taxable
→ last writer wins

Change inventory account
→ optimistic concurrency

Change costing method
→ perhaps bulk operation not allowed at all
```

---

## 8. Separate Orchestration From Product Business Logic

Avoid:

```text
BulkUpdateWorker
   ↓
UPDATE product SET taxable=...
```

if normal product updates normally go through:

```text
ProductService.updateProduct()
```

Otherwise the bulk path becomes a second mutation engine and may bypass:

- validation
- permission checks
- audit logic
- inventory invariants
- accounting constraints
- domain events

Preferred structure:

```text
Bulk Processor
    |
    | ProductPatchCommand
    v
Product Domain Service
    |
    ├── validation
    ├── persistence
    ├── audit
    └── domain events
```

But avoid making **200K internal HTTP requests** if possible.

Expose an optimized domain-level bulk API:

```text
ProductDomain.bulkPatch(
   List<ProductPatchCommand>
)
```

This preserves domain rules while allowing efficient batching.

---

## 9. Use an Outbox for Downstream Changes

A product update may affect:

```text
Product DB
   ↓
Search index
E-commerce integrations
Tax service
Reporting
Data warehouse
Cache
Sync engine
```

Do not synchronously call all of those systems for every product.

Use a transactional outbox:

```text
BEGIN

UPDATE product

INSERT INTO outbox_event
  ProductUpdated(...)

COMMIT
```

Then:

```text
Outbox Publisher
    ↓
Event Bus
    ↓
Search / Tax / Integrations / etc
```

Bulk updating 200K records could otherwise trigger **200K × N synchronous downstream calls**.

You may also emit a coarse-grained event:

```text
ProductsBulkUpdated {
   jobId,
   companyId,
   fieldsChanged: ["taxable"]
}
```

But do not replace individual events if downstream systems depend on item-level state transitions.

---

## 10. Backpressure and Fairness

A 200K-item job should not take down the catalog service for normal users.

Use queue-based throttling:

```text
interactive traffic
        |
        v
Product Service
        ↑
bulk workers
```

Bulk workers should have controls for:

```text
max concurrency per tenant
global concurrency
rate limits
DB connection limits
adaptive batch size
```

Example:

```text
global workers = 40
per company max = 4
batch size = 500
```

This prevents one company running:

```text
5 × 200K updates
```

from exhausting the system.

Use a **dedicated bulk-work queue / worker pool**, not the same priority path as user-facing traffic.

---

## 11. UX and Progress Reporting

The UI should not keep an HTTP request open while saying:

```text
Updating 200,000 products...
```

Instead, use job polling or push updates:

```text
Updating 187,325 products

92,500 processed
92,470 successful
30 failed

49%
```

Status API:

```http
GET /bulk-updates/{jobId}
```

Example response:

```json
{
  "status": "RUNNING",
  "total": 187325,
  "processed": 92500,
  "success": 92470,
  "failed": 30
}
```

Failure details:

```http
GET /bulk-updates/{jobId}/failures
```

Example:

```text
Product     Reason
ABC         Invalid tax category
XYZ         Concurrent modification
PQR         Product archived
```

Optionally support exporting failures as CSV.

---

## 12. Cancellation Semantics

Cancellation should mean:

```text
stop processing new batches
```

not:

```text
rollback everything already completed
```

Example:

```text
100,000 products updated
user clicks Cancel

→ remaining 100,000 aren't updated
→ existing 100,000 remain updated
```

If the product requires **Undo**, treat that as a separate feature.

Persist prior values:

```text
previous_value
```

per product:

```text
product_id
field
old_value
new_value
```

Then generate a **reverse bulk job**.

This is safer than trying to roll back a long-running distributed process.

---

## 13. Audit Logging

For catalog/accounting software, make auditability first class.

Avoid exposing 200K top-level audit entries like:

```text
Naveen changed Product A
Naveen changed Product B
...
```

Instead, create a parent operation:

```text
Bulk Update #9382
Initiated by: user-123
At: 2026-09-29 15:40
Selection: All active products
Operation: taxable false → true
Affected: 187,325
Succeeded: 187,300
Failed: 25
```

Then retain item-level entries underneath for investigation.

Conceptually:

```text
BulkChange
   |
   +-- BulkChangeItem
   +-- BulkChangeItem
   +-- BulkChangeItem
```

---

## Recommended Data Model

```text
bulk_job
--------
id
tenant_id
operation
payload_json
selection_json
status
target_count
processed_count
success_count
failure_count
created_by
created_at
started_at
completed_at
idempotency_key


bulk_job_item
-------------
job_id
product_id
product_version
status
retry_count
prepared_payload_json
error_code
error_message
processed_at

PK(job_id, product_id)
```

For barcode generation:

```json
{
  "barcode": "8901234567890"
}
```

stored in `prepared_payload_json`.

This makes retries deterministic.

---

## Full Lifecycle

```text
User
 |
 | Select all 200K
 | Generate barcode
 v
Bulk API
 |
 | create job
 v
Target Resolver
 |
 | snapshot 200K product IDs
 v
bulk_job_item
 |
 | partition into batches
 v
Queue
 |
 +-------+-------+-------+
 |       |       |       |
Worker Worker  Worker  Worker
 |       |       |       |
 v       v       v       v
Product Domain Service
 |
 | transactions of ~500
 |
 +--> Product DB
 |
 +--> Outbox
         |
         v
      Event Bus
         |
 Search / integrations / reporting
```

## Architectural Principle

> **Bulk update is orchestration over normal domain mutations, not an alternative mutation path.**

This becomes especially important once the product model has things like:

- inventory tracking
- accounting mappings
- tax configuration
- units
- cost codes
- barcode/SKU uniqueness
- external integrations

A direct database `UPDATE` can look attractive at 200K scale, but over time it creates a second path that bypasses business invariants.

For **200K records**, you do not need exotic big-data infrastructure.

A solid design can be built with:

- relational database
- durable job tables
- queue
- bounded worker pool
- transactional outbox

The harder problems are not raw throughput. They are:

- snapshot semantics
- idempotency
- concurrent edits
- per-field invariants
- partial failures
- auditability
- downstream side effects
