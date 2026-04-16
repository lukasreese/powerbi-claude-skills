# Question Bank — Power BI Requirements Gathering v1

10 phases. For each: questions, red flags, best practice tips.

---

## Phase 1 — Business Context & Sponsorship

**Goal:** Understand WHY this report exists and WHO is truly behind it. Vague scope and weak sponsorship are the two most common reasons projects stall before they ship.

**Questions:**
1. What is the main business problem this report needs to solve?
2. What decisions will people make using this report — and who makes those decisions?
3. Who are the primary end users — executives, managers, analysts, frontline operations?
4. How technically confident are the end users — do they know Power BI or are they just consumers?
5. Is there an existing report or tool being replaced? If so, what is wrong with it?
6. What does success look like 3 months after go-live?
7. Who is the executive sponsor — the person who owns the outcome, controls the budget, and will champion this if there is resistance?
8. Is the sponsor genuinely engaged or nominally attached — do they attend reviews, or are they just a name on an email?
9. Has Power BI been tried before in this organisation and abandoned? If so, why?
10. Is there resistance from people who currently own the reports being replaced — Excel owners, legacy system owners?

**Red flags:**
- "We just need a dashboard" with no specific decision identified — probe until there is a real answer
- No executive sponsor identified, or sponsor is an IT manager rather than a business leader
- Sponsor is nominally attached — signed off the budget but is not engaged — this project will drift when priorities shift
- Power BI tried before and abandoned — find out why before repeating the same mistakes
- Resistance from Excel or legacy report owners — this is a people problem, not a technical one, and it needs a plan
- "Everyone will use it" — vague audience means vague scope

**Best practice tip (surface when sponsorship is weak):**
*"Active executive sponsorship is the single strongest predictor of BI project success. The sponsor doesn't need to know Power BI. They need to hold the vision, remove blockers, and make their team use it. A sponsor who is only 'aware' of the project is not a sponsor."*

**Best practice tip (surface when success criteria are vague):**
*"A useful test: if someone opens this report and sees a number they don't expect, what do they do next? If you can answer that, you've defined success."*

---

## Phase 2 — Data & Sources

**Goal:** Understand the raw material. Data is almost never as clean or accessible as clients believe.

**Questions:**
1. Where does the data live today — Excel files, SQL Server, cloud database, ERP, API, SharePoint, Dataverse?
2. How many distinct data sources are involved?
3. Has the data quality been validated — are there known issues like duplicates, nulls, inconsistent formats, missing values?
4. Who owns each data source and who has permission to pull from it?
5. What is the required data refresh frequency — real-time, hourly, daily, weekly, monthly?
6. How far back does historical data need to go?
7. Is there an existing data warehouse or staging layer, or does Power BI connect directly to source systems?
8. Can you get a sample data extract in the first week?
9. Are there any known data issues that have caused reporting problems in the past?
10. Is the data structured, semi-structured, or unstructured — or a mix?

**Red flags:**
- Direct connection to live ERP without a staging layer — performance and stability killer
- No data owner identified — access problems surface at the worst moment
- "The data is clean" with no evidence — assume it is not until proven otherwise
- Real-time refresh requested without understanding the architectural and licensing implications
- Multiple data sources with no common keys or identifiers — join logic must be agreed before building
- Historical data needed but no archive exists — scope problem

**Best practice tip (surface when direct ERP is mentioned):**
*"Connecting directly to a live ERP is possible but usually painful — query timeouts, locks under load, refresh failures. A lightweight staging view in SQL Server buys stability for very little effort and is almost always worth it."*

**Best practice tip (surface when data quality is uncertain):**
*"Always request a sample extract in week one. Never design a data model based on a description of the data. Only design based on what you can actually see and query."*

---

## Phase 3 — Data Modelling & Semantic Layer

**Goal:** The data model is the foundation of every Power BI project. A bad model cannot be fixed with good DAX. These decisions must be made before any visuals are built.

**Questions:**
1. Is there an existing data model in Power BI Desktop or Analysis Services — or are we starting from scratch?
2. Is the data currently structured as a star schema — fact tables and dimension tables — or is it flat, denormalised tables?
3. If flat — is there an understanding of what the fact tables and dimensions should be before building starts?
4. Are there multiple fact tables — for example sales, inventory, and returns — that need to share common dimensions?
5. Are metric and KPI definitions documented anywhere — a data dictionary, a business glossary, or even a shared spreadsheet?
6. Are there calculated columns or measures that currently exist in Excel or other tools that need to be replicated in DAX?
7. Are naming conventions defined — for tables, columns, measures, and calculated fields?
8. Is there a standard measure naming format — for example prefixes like [Sales], [# Count], [% Margin]?
9. Will multiple developers or teams work on this project simultaneously?
10. If multiple developers — how will conflicts be managed? Is there a branching strategy or will developers work on separate files?
11. Is Git integration being used or planned — Power BI Projects format (.pbip) enables source control with Azure DevOps or GitHub?
12. Are there existing CI/CD pipelines for data or reporting assets that Power BI should integrate with?
13. Is the plan to use a shared certified dataset that multiple reports connect to, or will each report have its own model?

**Red flags:**
- Flat denormalised tables with hundreds of columns — will cause slow performance, confusing DAX, and maintenance pain
- No star schema plan — the model will grow into an unmaintainable mess
- Metric definitions only exist in someone's head — DAX built on undocumented logic will be disputed after go-live
- No naming conventions — multiple developers will produce inconsistent, unreadable models
- Every report has its own dataset with duplicated logic — no single source of truth, metrics drift over time
- Calculated columns used where measures should be — a common beginner mistake that destroys model performance
- Multiple developers with no Git integration — one person overwrites another's work with no recovery path
- PBIX format used for team development — PBIX is binary and cannot be diffed or merged in Git. Power BI Projects (.pbip) format is required for meaningful source control
- No branching strategy on a multi-developer project — direct commits to main guarantee conflicts

**Best practice tip (surface when Git or multi-developer is mentioned):**
*"Power BI Projects format (.pbip) splits the model and report into readable JSON and TMDL files that Git can actually diff and merge. If two or more developers are working on the same project, this is not optional. Save in .pbip format from day one — migrating later is painful. Use feature branches, review before merging to main, and connect to Azure DevOps or GitHub. This is now the professional standard."*

**Link:** https://learn.microsoft.com/en-us/power-bi/developer/projects/projects-overview

**Best practice tip (surface when flat tables are mentioned):**
*"A flat table with 50 columns is the most common starting point and the most expensive technical debt in Power BI. The star schema — one fact table in the centre, dimension tables around it — is not optional. It is the foundation of everything: performance, correct DAX, clean relationships. Time spent restructuring now saves weeks of debugging later."*

**Best practice tip (surface when naming conventions are missing):**
*"Naming conventions sound boring until three developers are working on the same model and every measure is named differently. Agree on a standard before anyone writes a line of DAX. A simple prefix system — [Sales Amount], [# Orders], [% Margin] — makes the model self-documenting."*

**Best practice tip (surface when shared dataset is discussed):**
*"A certified shared dataset is one of the most underused features in Power BI. Build the model once, certify it, and let multiple report authors connect to it. This is how you enforce a single version of the truth. Without it, every team builds their own version of revenue and they will never match."*

**Link:** https://learn.microsoft.com/en-us/power-bi/guidance/star-schema

---

## Phase 4 — Performance & Scale

**Goal:** Architecture decisions must be made in requirements. The wrong storage mode is expensive to reverse and the wrong data volume assumptions will cause failures in production.

**Questions:**
1. Approximately how many rows of data — thousands, millions, tens of millions, hundreds of millions?
2. How many concurrent users at peak time?
3. Is Import mode acceptable — data loaded into Power BI memory — or does it need to be live?
4. Are there existing performance complaints about current reporting tools?
5. What is the expected data growth rate — will this double in 12 months?
6. Are there large historical datasets that could be archived or aggregated rather than loaded in full?
7. Are there high-cardinality columns in the data — for example free-text fields, GUIDs, long product codes?
8. Will aggregation tables or incremental refresh be needed to manage volume?
9. Are there complex DAX calculations expected — time intelligence, rolling averages, period-over-period comparisons?
10. What is the acceptable report load time — is 3 seconds acceptable, or does it need to be under 1 second?

**Red flags:**
- 100M+ rows planned for Import mode without Premium capacity — model will be slow or fail to refresh
- "We need real-time" without understanding DirectQuery performance tradeoffs and query load on the source system
- No thought given to data growth — a model that works today may not work in 18 months
- High-cardinality text columns in fact tables — massive impact on model size and performance
- No performance target defined — impossible to know if the build is successful

**Best practice tip (surface when volume or real-time is mentioned):**
*"Storage mode is the most consequential technical decision in a Power BI project. Import is fastest but data is a snapshot. DirectQuery is live but every visual generates a database query. Direct Lake — Fabric only — is the best of both but requires data in OneLake as Delta Parquet. This decision shapes everything. Make it now."*

**Best practice tip (surface when high cardinality is mentioned):**
*"The VertiPaq engine compresses repeated values extremely well. A column with 10 distinct values across 10 million rows is almost free. A column with 10 million unique GUIDs or free-text values is the single biggest threat to model size. Identify and remove high-cardinality columns that are not needed in the report before building."*

**Link:** https://learn.microsoft.com/en-us/power-bi/connect-data/service-dataset-modes-understand

---

## Phase 5 — Admin, Infrastructure & Storage Mode

**Goal:** The operational environment determines whether the report works day-to-day after go-live. Refresh failures, gateway misconfigurations, and capacity limits are silent project killers.

**Questions:**
1. What Power BI infrastructure exists — shared capacity, Premium Per User, Premium capacity, or Fabric?
2. Is Microsoft Fabric in use or being evaluated — is Direct Lake on the roadmap?
3. If Direct Lake — is the data already in OneLake as Delta Parquet?
4. What is the maximum acceptable data latency — how stale can data be?
5. How many scheduled refreshes per day, and during which hours?
6. Are existing datasets already hitting refresh limits?
7. Is there an on-premise data gateway — who manages it and how often is it maintained?
8. Are deployment pipelines in use — dev, test, and production environments?
9. What happens when a refresh fails — is there monitoring and alerting?
10. Who is the Power BI Service admin?
11. What tenant-level settings are relevant — is export to Excel allowed, can users download PBIX files, is publish to web enabled or blocked?
12. Are workspace-level permissions managed centrally by IT, or self-managed by report owners?
13. Is there a process for provisioning new workspaces — or does anyone create workspaces freely?

**Red flags:**
- Hourly refresh on Import dataset on Pro license — Pro allows 8 refreshes per day maximum
- Direct Lake requested but data is in on-premise SQL Server — requires OneLake first
- No gateway owner for on-premise sources — when it breaks, nobody knows how to fix it
- No deployment pipeline — publishing directly to production is a governance risk
- No refresh failure alerting — silent failures mean stale data nobody notices
- No tenant setting review — exporting PBIX files means the entire data model leaves the building
- Workspaces created freely with no naming or governance — report sprawl guaranteed

**Best practice tip (surface when tenant settings are mentioned):**
*"Tenant settings are often ignored in requirements and cause surprises at go-live. Can users download PBIX files? Can they export to Excel? Is publish to web enabled? These are security and governance decisions, not technical ones. They need to be agreed with IT before the report goes live, not after."*

**Best practice tip (surface when gateway is mentioned):**
*"The on-premise gateway is the most frequently broken component in any Power BI environment. It runs on a server that sometimes gets rebooted, patched, or decommissioned without the BI team being told. Name the gateway owner. Document the server. Set up refresh failure alerts. This is not optional for production workloads."*

**Best practice tip (surface when refresh limits are mentioned):**
*"Refresh limits by license: Pro = 8/day. Premium Per User = 48/day. Premium capacity = 48/day or higher with XMLA. Fabric = flexible by SKU. If the client needs hourly refresh and is on Pro, this is a licensing conversation before it is a technical one."*

**Link:** https://learn.microsoft.com/en-us/fabric/get-started/microsoft-fabric-overview

---

## Phase 6 — Report & Visual Requirements

**Goal:** Understand what they actually want to see — and manage expectations early. More pages and more visuals is not better.

**Questions:**
1. Roughly how many reports and pages are needed?
2. Is this a one-shot delivery or iterative — v1 delivered and improved, or one go-live date?
3. Are there existing mockups, sketches, or examples from other tools they like?
4. What are the 5 most important KPIs — can they rank them in priority order?
5. What time dimensions matter most — daily, monthly, YTD, rolling 12 months, fiscal year?
6. Does the organisation have a fiscal year that differs from the calendar year?
7. Do visuals need to follow company branding — specific colours, fonts, logo placement?
8. Are there any regulatory requirements on how data is displayed or labelled?
9. Will users need to drill down into detail or is high-level summary sufficient?
10. Do reports need to work on mobile devices?
11. Are tooltips, bookmarks, or custom navigation expected?
12. Are custom visuals required — third party visuals from AppSource, Zebra BI, Deneb?
13. Is there a report theme file already defined, or does one need to be created?

**Red flags:**
- More than 8-10 pages requested upfront — negotiate scope down or split into phases
- No KPI prioritisation — "everything is equally important" produces reports that communicate nothing
- "Make it look like this Excel file" with 40+ columns — education needed
- Fiscal year not discussed — time intelligence DAX will be wrong
- Mobile layout mentioned late — significant additional design effort
- Custom visuals requested without understanding licensing and support implications — AppSource visuals can break with Power BI updates

**Best practice tip (surface when KPIs cannot be prioritised):**
*"Ask the stakeholder: 'If you could only see one number every morning, what would it be?' Build from that outward. Reports that try to show everything communicate nothing."*

**Best practice tip (surface when iterative delivery is not planned):**
*"A single well-adopted report with clean data is worth more than ten half-finished reports nobody trusts. Plan for a v1 that proves value fast, then iterate. Big-bang go-lives carry all the risk."*

**Best practice tip (surface when report theme is not mentioned):**
*"A report theme file locks in colours, fonts, and visual defaults across every page. It takes 30 minutes to create and saves hours of per-visual formatting. It also makes the report look like it belongs to the organisation, not like a default Power BI template."*

---

## Phase 7 — Security, Access & Licensing

**Goal:** Security designed after the report is built is painful and expensive to retrofit. Licensing surprises kill go-live timelines.

**Questions:**
1. Who needs access — internal staff only, or also external guests or partners?
2. Do different users need to see different subsets of data — is Row Level Security needed?
3. If RLS — is it based on user email, department, region, management hierarchy, or a combination?
4. How are users authenticated — Microsoft 365, Azure AD, other?
5. What Power BI licenses do users currently have — Free, Pro, or Premium Per User?
6. Is there Premium capacity or Fabric capacity?
7. How many people need to VIEW the report — all consumers, not just developers?
8. Will the report be embedded in an external app, SharePoint, Teams, or a public website?
9. Are there data sensitivity or compliance requirements — GDPR, HIPAA, SOC2, financial regulations?
10. Who manages user access on an ongoing basis after go-live?
11. Are Microsoft Information Protection sensitivity labels in use — are datasets classified as confidential?
12. Is there a data loss prevention policy that could block certain exports or sharing actions?

**Red flags:**
- "Everyone can see everything" — probe whether that includes HR, payroll, salary data
- 200+ viewers on Pro licenses with no Premium — licensing cost will be a shock
- External embedding without understanding Power BI Embedded licensing — different commercial model entirely
- GDPR-sensitive data with no sensitivity labels or governance plan
- No ongoing access manager — self-managed access becomes a security and compliance risk
- DLP policies in place but not reviewed — a DLP policy can block legitimate report sharing silently

**Best practice tip (surface when RLS is mentioned):**
*"Two RLS approaches: static roles defined in Desktop, and dynamic RLS using a DAX security table mapped to user emails. Dynamic is more flexible but more complex. The right choice depends on how often security rules change. Design this before the first visual. Retrofitting RLS into a finished report is painful and error-prone."*

**Best practice tip (surface when viewer count comes up):**
*"At roughly 25+ regular viewers, Premium Per User becomes more cost-effective than individual Pro licenses. Model the cost before committing to a licensing approach."*

**Link:** https://learn.microsoft.com/en-us/power-bi/enterprise/service-admin-rls

---

## Phase 8 — Integration & Business Logic

**Goal:** Understand what Power BI connects to and what business rules must be captured before the model is designed.

**Questions:**
1. Does the report need to trigger alerts or push notifications when thresholds are crossed?
2. Is Power Automate integration needed — approvals, workflows, or actions from Power BI?
3. Are other BI tools currently in use — Tableau, Qlik, SSRS, Excel reports, Looker?
4. Are KPI definitions agreed and documented across departments?
5. Are there complex business rules that need documenting before building starts?
6. Are there currency conversions, time zone adjustments, or fiscal year differences?
7. Is there a data glossary or data dictionary?
8. Will data ever flow back from Power BI into source systems?
9. Does this project depend on other projects — a data warehouse migration, an ERP upgrade, a cloud move?
10. Are there APIs that need to be connected — external data providers, weather data, financial market feeds?

**Red flags:**
- "Revenue" defined differently across departments — must be resolved before building measures, not after
- No data dictionary and complex business rules — assumptions baked into DAX will be disputed
- Currency conversions needed but no exchange rate source identified
- Multiple BI tools with no clear delineation of purpose — political complexity ahead
- Dependencies on other in-flight projects — a delayed ERP migration will delay this project too
- Data writeback requested — Power BI is not designed as a transaction system

**Best practice tip (surface when KPI conflicts emerge):**
*"KPI definition conflicts between departments are one of the most common reasons Power BI reports lose stakeholder trust after go-live. Finance says revenue is X, sales says it's Y. Both are right by their own logic. This must be resolved, documented, and signed off before any DAX is written. It is a business alignment problem, not a technical one."*

---

## Phase 9 — Governance & Workspace Standards

**Goal:** Governance sounds boring until the Power BI environment has 200 reports, nobody knows which ones are accurate, and three versions of the same metric exist in different workspaces. These questions prevent that.

**Questions:**
1. Is there a naming convention for workspaces — for example Finance-Prod, Sales-Dev?
2. Is there a naming convention for reports and datasets — who defines and enforces it?
3. Are there development standards — which visuals are approved, which are banned, what formatting rules apply?
4. Is there a certification or endorsement process for datasets — promoted vs certified?
5. Who can publish reports to production — is it anyone, or is there an approval process?
6. Is there a process for retiring or archiving old reports — or do unused reports accumulate forever?
7. Is there a Center of Excellence or a BI governance team — or is Power BI self-managed by each department?
8. How is version control handled — are PBIX files saved to SharePoint, Git, or just on local machines?
9. Is lineage tracking used — does the organisation know which reports depend on which datasets?
10. Are usage metrics monitored — does anyone track which reports are actually used and which are abandoned?
11. Is there a formal process for requesting new reports or changes — a backlog, a ticketing system?
12. Are there workspace app settings configured — who can see the workspace app vs the workspace itself?

**Red flags:**
- No naming conventions — workspace sprawl and report duplication are inevitable
- Anyone can publish to production — untested reports go live and break trust
- No dataset certification process — every team builds their own version of the same data
- PBIX files saved on local machines — one laptop failure destroys months of work
- No usage monitoring — you will never know if the report is actually being used
- No retirement process — abandoned reports accumulate, confuse users, and waste capacity
- No Center of Excellence — Power BI governance does not self-organise

**Best practice tip (surface when naming conventions are not defined):**
*"Naming conventions need to be defined before the first workspace is created, not after. Retrofitting names across 50 workspaces and 200 reports is a painful project nobody wants to do. A simple standard — Department-Environment, for example Finance-Prod — takes 30 minutes to agree and saves months of confusion."*

**Best practice tip (surface when version control is mentioned):**
*"PBIX files saved on local machines or shared drives is not version control. It is a disaster waiting to happen. At minimum, store files in SharePoint with version history. For teams doing serious development, Power BI Projects format with Git integration is now available and is the professional standard."*

**Best practice tip (surface when dataset certification is discussed):**
*"Promoted means 'the owner recommends this'. Certified means 'the organisation has validated this as the official source of truth'. Only certified datasets should be used in executive reporting. This distinction prevents the proliferation of slightly-different versions of the same metric."*

**Link:** https://learn.microsoft.com/en-us/power-bi/collaborate-share/service-endorse-content

---

## Phase 10 — Change Management, Training & Adoption

**Goal:** The best Power BI report in the world fails if nobody uses it. Adoption does not happen by accident. It is designed. This phase is the most commonly skipped and the most commonly regretted.

**Questions:**
1. Is there a formal adoption plan — or is the expectation that people will just start using it?
2. Who will train end users — the developer, a dedicated trainer, recorded walkthroughs, Microsoft Learn?
3. When does training happen — before go-live, at go-live, or after?
4. Are different user groups getting different training — executives need different training than analysts?
5. Is there a support channel after go-live — who do users contact when something looks wrong or they have a question?
6. How will adoption be measured — is there a target for active users, report views, or business decisions influenced?
7. Are there Power BI champions identified in each department — internal advocates who can support peers?
8. Is there a feedback mechanism — how do users request changes or report data issues?
9. Is there known resistance to adopting Power BI — people attached to Excel, existing reports, or other tools?
10. Is there a plan for handling that resistance — communication, involvement in design, executive mandate?
11. Will the report replace a tool people currently rely on — and is there a transition plan or cut-over date?
12. Is there ongoing training planned for when Power BI features change or new reports are added?

**Red flags:**
- "People will figure it out" — they won't. Unused reports are the most common outcome of Power BI projects with no adoption plan
- Training planned for after go-live — by then users have already formed a negative first impression
- No support channel — users who hit a problem and can't get help stop using the tool
- No adoption measurement — you will never know if the project succeeded
- Known resistance with no mitigation plan — this will surface at go-live as a political problem
- One-size-fits-all training — executives who sit through an analyst-level training will disengage immediately
- No champions — central IT cannot support distributed Power BI adoption at scale

**Best practice tip (surface when adoption plan is missing):**
*"Adoption does not happen because the report is good. It happens because the right people were involved in designing it, trained before go-live, and have someone to call when something goes wrong. Without those three things, even a perfect report gets abandoned within 90 days."*

**Best practice tip (surface when resistance is identified):**
*"The fastest way to neutralise resistance is involvement. The person most attached to the Excel report they built over 5 years is also the most knowledgeable about the business logic it contains. Involve them in the Power BI design. They become a champion instead of a blocker."*

**Best practice tip (surface when champions are discussed):**
*"A champion network — one or two power users per department who get advanced training and early access — scales adoption without scaling the support burden. They are not IT. They are business users who understand their team's data problems. They are the most effective adoption mechanism that exists."*

**Link:** https://learn.microsoft.com/en-us/power-bi/guidance/fabric-adoption-roadmap-change-management
