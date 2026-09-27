# How Meta Wrote the Rules for AI in Accounting

## Interview at a glance

**Speakers:** Aaron Anderson, Chief Accounting Officer at Meta; John Glasgow, CEO and founder of Campfire.

**Central idea:** Accounting's role in verifying accurate financial information becomes more important as AI handles more work. Responsible adoption takes controlled experimentation, reliable data, sustained testing, and training across the team.

## Key takeaways

- Meta has used machine learning for years. Its newer experiments with agentic AI have taken sustained testing and control work; Aaron said reliable deployment at scale took longer than expected.
- Accounting teams can build on familiar risk assessment and internal control practices, while identifying risks specific to probabilistic systems.
- Aaron described a cross-company effort to develop a principles-based AI control framework, reviewed with the Big Four and discussed with SEC staff. He described the framework as a living document.
- The practical limit for many AI efforts was data quality and coverage. Finance AI needs transactional, non-financial, contextual, and control/testing data, with suitable access controls.
- Meta's clearest live example so far was AI-generated flux analysis, which Aaron said helped move the close from seven days to six. Customer-facing revenue operations agents remained in pilot after about nine months.
- Meta is building a “second brain” for the general ledger and controllership to capture procedures and operational knowledge, support role transitions, and reduce manual outsourced work.
- Aaron's longer-term goal is a more continuous close and continuous control monitoring, with business events flowing into financials and forecasts. He does not see close speed as the end goal.
- For broad adoption, Meta defined baseline AI fluency for finance professionals and planned asynchronous learning plus three in-person training days for each employee.
- Aaron's view of the profession: technology changes accounting work, but the responsibility to verify truth remains.

## 1. Why Meta leaned into AI

- Aaron distinguished traditional machine learning from newer generative and agentic systems. Meta had used machine learning for years before the 2023 surge in generative AI.
- He gave the example of a business receiving an advertising account and pre-approved credit limit through an automated process that can respond in minutes.
- Aaron described 2025 as a year of experimentation: trying different applications, identifying where AI helped, and learning where it did not. In 2025 and 2026, a central concern was how to control AI in finance before putting more agentic workflows into production.
- Meta's innovative culture, leadership support, internal tools, and colleagues in other teams already using AI encouraged finance to experiment. Aaron said finance also needed to understand AI in upstream teams to recognize risks that could reach financial reporting.
- He noted that Meta's access to internal tools is an advantage other companies may not have.

## 2. Defining responsible AI controls

### What the framework effort involved

1. **Cross-company workshop:** Aaron brought finance and accounting peers from Google, Walmart, and ServiceNow together with Meta for a two-day workshop in Menlo Park. Around 15–18 people started a framework for responsible AI use in finance.
2. **Audit firm review:** Around November 2025, the group shared a draft with the national offices of all four major audit firms. The firms disagreed on different points, including whether multi-model validation could count as an independent control. Aaron said dialogue helped resolve differences and there was much greater alignment by the following February.
3. **SEC staff discussions:** The team held two rounds of discussion with SEC staff. Aaron said the objective was a principles-based framework that could support production use of agentic, probabilistic systems while maintaining financial integrity. He said SEC staff encouraged publication. This reflects Aaron's account of the discussions, not formal SEC approval.
4. **Publication:** Aaron said the framework was published through FEI after review and input from about 35 FEI CCR member companies. He expects it to change as technology and practice evolve; he said it is available through FEI behind a paywall.

### The control philosophy

- Healthy skepticism is part of the accounting profession's responsibility to present and verify the truth. Aaron's practical test: understand the system, verify its results, and be confident signing off on them.
- The workshop found that established methods still provide a foundation: understand the technology and data, assess risks and failure modes, and apply relevant controls. Teams also need to examine what is unique about AI rather than assuming existing controls cover every case.
- John encouraged audited and public-market companies to write down intended AI uses, excluded uses, and planned controls, then discuss the policy with their auditor early.
- Aaron said his team initially wondered whether existing SEC guidance was sufficient. He recalled SEC staff challenging that assumption in December, then later agreeing, after further discussion in May and June, that the framework applied existing guidance and did not require new guidance at that point. This is Aaron's account.
- Aaron also raised a longer-term audit question: whether auditors may need to put more emphasis on whether financial outcomes are accurate, rather than focusing primarily on the technology's process trail. He said this would require the profession to evolve; he did not present it as an established audit approach.

## 3. Where AI has moved the needle at Meta

### Live: flux analysis

- Meta uses AI with general ledger and transaction data to draft explanations for changes in account balances. Accountants review the explanations and supporting information.
- Aaron said the workflow was in its fourth quarter of live use. The team tracked accuracy and how much editing was needed, then expanded the use over successive quarters.
- He attributed a change from a seven-day close to a six-day close to automating flux analysis and related analysis.
- John said flux can be a useful starting point because a team can inspect each output while using AI to scan larger amounts of data and spot patterns an individual analyst might miss.

### In pilot: customer and revenue operations

- Meta was piloting agents to answer customer questions such as requests for invoices or explanations of transactions.
- Aaron said the pilot had lasted about nine months. The team kept modifying the system as it observed errors and unexpected behavior, given the accuracy needed in revenue and customer interactions.
- His deployment principle: a workflow must “earn the right” to move to human-on-the-loop operation through evidence gathered at each step. He said trustworthy deployment took quarters longer than he expected.

### In development: controllership “second brain”

- Meta is building a system around the general ledger and controllership that Aaron called a “second brain.” The goal is to capture what needs to happen and when, reduce reliance on separate desk procedures, and automate some routine activity.
- The documented knowledge could help when employees leave or change roles. Aaron also sees potential to reduce manual outsourced work and its cost.

### What has not worked well yet

- In 2025, Meta experimented with an “AI CFO” concept while its data was incomplete or inaccurate. Aaron said the AI had to guess, which limited its usefulness.
- Many experiments produced useful learning about where AI did not add value.
- Customer-facing agents were still in pilot after about nine months, reflecting the time needed to test and refine them.

## 4. Data and controls as prerequisites

Aaron described Meta's shift from a narrow data strategy to a broader data ecosystem. He said Meta reorganized its data team and strategy in 2024, anticipating AI's dependence on data.

| Data category | Examples Aaron gave | Why it matters |
|---|---|---|
| Transactional | General ledger, debits and credits, payroll, operational records | Grounds analysis in recorded activity |
| Non-financial | Forecasts, budgets, broader employee information | Adds operating context beyond ledger entries |
| Contextual | Business or external context, such as geopolitical supply chain effects | Helps interpret events affecting the business |
| Control and testing | Data sets used to test and monitor AI | Supports continued checks that the system works as intended |

- These data sources need appropriate access rights and controls.
- Aaron said data limitations repeatedly constrained AI projects. His advice was to start with information the team has, understands, and controls.
- Meta has used machine learning in some areas for roughly seven to eight years, newer agentic capabilities for roughly two years, and had spent about a year and a half trying to scale them at the time of the interview.

## 5. Bringing a large finance team along

- Aaron estimated about 1,000 people in Meta Finance and about 550 in his accounting organization, plus outsourced support he described as equivalent to about 650 people. The totals he stated do not reconcile cleanly, so treat the figures as approximate.
- He uses a directional change model: around 20% of people eagerly try new tools, 10% are reluctant, and the middle 70% need support. He focuses on helping the middle group rather than only the early adopters.
- He said even highly capable colleagues felt behind when a small group moved quickly with AI. Meta defined AI fluency for finance professionals so they would not be measured against engineers.
- The four fluency pillars are:
  1. Understanding the tools.
  2. Applying an AI mindset and process to work.
  3. Understanding controls, risk, and responsible use.
  4. Using and designing with data.
- Meta developed asynchronous learning and aimed to give every finance employee a baseline “Level One” fluency, with room for deeper skill levels. The plan was to follow self-paced work with three in-person training days for each person. Aaron said two pilots were complete and a third was underway during the interview.

## 6. The future of close and the accounting profession

### Beyond a faster close

- Aaron said he wants to extend continuous close practices and continuous control monitoring so teams catch issues as they arise instead of waiting for month-end or quarter-end.
- His broader vision is for business events to flow through financial systems and into forecasts, making close less of a discrete event and enabling more dynamic forecasting. He also wants finance work to cross organizational boundaries more easily.
- Meta already uses continuous practices in some areas, according to Aaron. He expects close speed to improve but does not consider speed the ultimate goal.
- John said Campfire uses continuous close and forecasting internally. He described agents pulling forward-looking data from systems such as CRM and spend tools into forecasts, supporting more granular analysis and tighter variance thresholds.

### How the profession may change

- Aaron recalled carrying a heavy computer to audit engagements early in his career. He also remembered fears that Excel would eliminate accounting jobs. In his view, accounting did not disappear; technology created new capabilities and work, including data analysis.
- He sees the core responsibility of accounting as verifying truth, a responsibility he believes grows more important as AI spreads because fraud and error risks remain.
- He referenced Arizona State University professor Eldar Maksimov's work on the “verification imperative” and connected it to Jevons paradox: greater efficiency can expand activity and the need for verification.

## Key learnings for agentic book close

*Synthesis from the interview, rather than direct claims that Meta has fully implemented an agentic close.*

1. **Define the outcome before chasing speed.** A shorter close is one measure, but the stronger goal is timely, reliable financial information and controls that surface issues as they occur.
2. **Build from a bounded workflow.** Flux analysis is a useful first candidate in the interview because the source data can be inspected and every explanation can be reviewed. It provides evidence about accuracy and edit rates before expanding autonomy.
3. **Make data readiness part of the close design.** A close agent needs more than ledger balances: it may require non-financial drivers, business context, and testing data. Missing or poor-quality inputs can cause plausible guesses.
4. **Keep procedures and accounting judgment available to the agent and the people supervising it.** The “second brain” idea points toward capturing task sequence, timing, evidence, and role knowledge so work is repeatable and transitions are safer.
5. **Earn autonomy through measured performance.** Move from draft and human review toward more autonomous handling only after testing across periods, exceptions, and changing conditions. Retain escalation paths for uncertain or unusual cases.
6. **Design for continuous monitoring.** Treat reconciliations, control checks, and exception detection as ongoing processes where possible, rather than tasks that appear only at period end.
7. **Train the whole team.** A close agent changes how preparers, reviewers, and controllers work. Shared fluency in tools, process, data, and controls helps prevent a small group from carrying all the knowledge.

## Provocations for designing an agentic book close

- What would make the close a non-event: real-time ledger updates, continuously reconciled accounts, or a decision point that only confirms the period is complete?
- Which close tasks can produce a reviewable draft, and which tasks can an agent execute? What evidence earns each increase in autonomy?
- Can each journal entry, accrual, reconciliation, and variance explanation show its source data, rationale, approvals, and changes in a way a reviewer can follow?
- How should the system behave when sources conflict, data is missing, or an event falls outside its tested range: stop, ask for review, or proceed under a defined threshold?
- What control data and repeatable test cases will reveal that an agent's behavior has drifted after a model, policy, source system, or business process changes?
- How will continuous accounting interact with formal period cutoffs, late adjustments, materiality, and the need to freeze reported results?
- What matters more than close duration: fewer late adjustments, lower unexplained variance, more reliable forecasts, reduced manual effort, or earlier risk detection?
- Which knowledge currently lives only in desk procedures or individual employees' heads, and what must be documented before automating the work?
