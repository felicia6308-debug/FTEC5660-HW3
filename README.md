# FTEC5660 Homework 3: Fisher's 15-Point Investment Agent

Build a LangChain agent that researches a company with tools and applies the
framework from Philip Fisher's *Common Stocks and Uncommon Profits* (Part One,
Chapter 3 "What to buy: the fifteen points to look for in a common stock", and
Chapter 4 "What to buy: applying this to your own needs"). You choose 3 of
Fisher's 15 points and decide yourself what PASS means for each. For every
company your agent gives a PASS/FAIL verdict on your 3 points and a short
summary.

The companies are fictional, so the model cannot answer from memory. Unlike a
pile of reports, the evidence is spread over three sources, and your agent has
to go and get it with tools, the way Fisher's "scuttlebutt" method sends an
investor to talk to customers, competitors, suppliers and staff.

## Data and tools

`public_test/` has one folder per company. Your code receives four tools per
company (built by the provided code in `hw3.py`):

| Tool | What it gives you |
| --- | --- |
| `list_documents()` | the company's six documents and their dates |
| `read_document(name)` | one document: the 2025 letter to shareholders, four earnings calls (Q3 2025 to Q2 2026) and a news file (July 2025 to July 2026) |
| `sql_query(query)` | one read-only SQLite query on the company's database; at most 50 rows come back, so use `SUM`, `GROUP BY` and `WHERE` |
| `interview(role, topic)` | what one person says about the company; at most **10 interviews per company** |

The documents are what management says. Management's own claims (such as "we
have never cut research" or "our people are our greatest strength") are not
evidence on their own: check them against the database and the interviews.

**Database tables** (amounts in USD thousands; dates `YYYY-MM-DD`, months
`YYYY-MM`, quarters like `2025Q3`; the fiscal year is the calendar year):

| Table | Columns |
| --- | --- |
| `company` | name, industry |
| `industry_medians` | industry, revenue_growth_2y_pct, new_product_share_pct, new_product_sales_per_rd, operating_margin_pct (one row per industry; use the row for the company's own industry) |
| `orders` | order_id, order_date, product, customer, amount, note (about 3,000 to 4,000 orders from January 2023 to June 2026; revenue is the sum of order amounts) |
| `products` | product, launch_date |
| `monthly_costs` | month, cost_of_sales, selling_admin, research_development |
| `quarterly_cash` | quarter, operating_cash_flow, capex, cash_end |
| `capex_plan` | year, planned_capex (2026 to 2028) |
| `returns` | month, units_shipped, units_returned |
| `insider_trades` | trade_date, person, title, side (`buy` or `sell`), shares, price |

**Terms used in the data.**

| Term | Meaning |
| --- | --- |
| FY2025 | Fiscal year 2025. Here the fiscal year is the calendar year, so FY2025 is January to December 2025. |
| Quarter | Three months of the year: Q1 is January to March, Q2 April to June, Q3 July to September and Q4 October to December. The database writes `2025Q3`; the documents write "third quarter 2025". |
| Amounts | The database is in **USD thousands** (676,100 means US$676.1 million). The documents use USD millions. |
| Revenue | Money from sales: the sum of `amount` in `orders`. |
| Cost of sales | The cost of making the products that were sold. |
| Selling and admin | The cost of selling and of running the company (sales staff, offices, management). |
| R&D | Research and development spending, mainly on new products. |
| Operating margin | Operating profit as a share of revenue: (revenue − cost of sales − selling and admin − R&D) ÷ revenue. |
| Operating cash flow | Cash that the day-to-day business brings in during a quarter. |
| Cash at the end of a quarter | The cash the company holds on the last day of that quarter (`cash_end`). |
| Capex | Capital expenditure: spending on factories, machines and other long-term assets. `capex_plan` holds the planned capex for 2026 to 2028. |
| Return rate | Units returned ÷ units shipped, from the `returns` table. |
| Industry median | The middle value for companies in the same industry, from `industry_medians`. |
| Insider | A director or senior manager of the company. `insider_trades` lists the shares they bought (`buy`) or sold (`sell`). |
| Earnings call | A meeting each quarter where management presents the quarter's results to investors. Each call quotes that quarter's revenue. |

**Interviews.** Roles: `customer`, `current_employee`, `former_employee`,
`competitor`, `supplier`. Topics: `sales_team`, `working_conditions`,
`top_management`, `decision_making`, `competitive_edge`. Only some people know
about each topic; the others say they can't comment, and that answer still
uses up one interview.

| Topic | Who knows about it |
| --- | --- |
| `sales_team` | customer, competitor |
| `working_conditions` | current_employee, former_employee |
| `top_management` | former_employee, supplier |
| `decision_making` | supplier, customer |
| `competitive_edge` | competitor, customer |

People who know about a topic always agree with each other, so one good answer
is enough.

## Choose your 3 points and your rules

Pick any 3 of Fisher's 15 points and write them in `CHOSEN_POINTS` near the
top of `hw3.py`, e.g. `CHOSEN_POINTS = [1, 7, 15]`. The full questions are in
`FISHER_POINTS` in `hw3.py` and in the book.

| # | Fisher's point | Where the evidence is |
| --- | --- | --- |
| 1 | Market potential for sales growth | database |
| 2 | New products as current lines mature | database |
| 3 | Effective research and development | database |
| 4 | Above-average sales organization | interviews (`sales_team`) |
| 5 | Worthwhile profit margin | database |
| 6 | Maintaining or improving margins | database |
| 7 | Labour and personnel relations | interviews (`working_conditions`) |
| 8 | Executive relations | interviews (`top_management`) |
| 9 | Depth of management | interviews (`decision_making`) |
| 10 | Cost analysis and accounting controls | earnings calls, checked against the database |
| 11 | Advantage over competitors | interviews (`competitive_edge`) |
| 12 | Long-range outlook on profits | database |
| 13 | Growth without dilutive equity financing | database |
| 14 | Management talks freely about troubles | earnings calls, checked against the database |
| 15 | Management of unquestionable integrity | news file, checked against the database |

For each of your points, **you design the rule**: which evidence to use, what
to compute or whom to ask, and where the line between PASS and FAIL lies. PASS
means the company looks favourable on that point, as Fisher would see it.

We grade your verdicts against our own answer key. It uses one fixed rule per
point, which we do not publish, and the data never puts a company close to
the line of that rule, so a sensible reading of Fisher usually gives the same
answer. **Test your rules on the public answers**:
`public_test/ground_truth.json` gives the expected verdict on all 15 points
for each of the 20 public companies, and every point fails in 6 of them. If
your rule disagrees with the public answers, rethink it.

**Tip: calculate with tools, not in your head.** Do every sum, ratio and
comparison in SQL or Python, including the number of days between two dates
(SQLite's `julianday()` gives a date as a number of days).

## Student task

Fill in the three places in `hw3.py` marked `### YOUR CODE HERE`. You may add
imports, constants and helper functions above them, but do not change the
provided code below them:

- `CHOSEN_POINTS` lists your 3 points.
- `build_agent()` creates your model and anything else you need once.
- `analyze_companies(agent, companies)` receives
  `{company_folder: [list_documents, read_document, sql_query, interview]}`
  and returns, for every company,
  `{"points": {1: "PASS", 7: "FAIL", 15: "PASS"}, "summary": "..."}` (here with
  `CHOSEN_POINTS = [1, 7, 15]`).

Use the course techniques that fit your points: tool use for the database and
the interviews, planning which interviews to spend, routing each point to the
right source, prompt chaining, parallel calls, reflection to check your
answers, or a combination. Your code must read company data **only through
the four tools**; do not open `company.db` or `interviews.dat` yourself. Do
not hard-code folder names, company names or the public answers.

## Setup and public test

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
echo "DEEPSEEK_API_KEY=your_key_here" > .env
python3 hw3.py --data-folder public_test
```

Use your own DeepSeek API key (from https://platform.deepseek.com). The
required model is `deepseek-v4-flash`. We grade with our own key, so never put
a key in your code; `.env` must stay out of git.

The program creates `results.csv` with the columns `company`, `points`,
`points_correct`, `point_errors`, `interviews_used` and `summary`, and prints
the point score on your 3 points (the percentage after "balanced"). The point
score is the average of two shares: the share of truly passing verdicts that
you marked PASS and the share of truly failing verdicts that you marked FAIL.
For example, 90% and 60% give (90% + 60%) / 2 = 75%, and marking every point
PASS gives (100% + 0%) / 2 = 50%. Once `CHOSEN_POINTS` is set, the starter
runs before you add any API code (it returns no verdicts, so it scores 0%).

`public_test/ground_truth.json` gives the expected verdicts on all 15 points
for each public company. Grading uses a private set of about 20 unseen
companies with the same tools and the same answer key rules, but different
names and wording, and scores your 3 points only.

## Homework 3 solution

### My three points and rules

I selected points **4, 7 and 8** because the supplied interviews directly describe sales service, employee experience and executive cooperation. These topics also demonstrate Fisher's scuttlebutt approach: asking people with relevant experience. The assignment states that knowledgeable respondents agree, so one appropriate respondent per topic is sufficient in this dataset.

| Point | Why selected and evidence | PASS rule |
| --- | --- | --- |
| 4 — Above-average sales organization | Ask a customer about `sales_team`. Fisher emphasizes customer satisfaction, repeat business, capable salespeople and continuing improvement in sales service. | PASS when concrete behavior demonstrates knowledgeable, proactive, effective customer service and no unresolved serious negative evidence is present. Poor product knowledge, merely sending prices or customers seeking competitors' advice supports FAIL. |
| 7 — Outstanding labor and personnel relations | Ask a current employee about `working_conditions`. Fisher discusses fair treatment, retention, morale, wages and effective grievance handling. | PASS when concrete evidence supports fair treatment, healthy morale, stable retention or responsive grievance handling without unresolved serious problems. Persistent unfairness, low morale, frequent departures or unresolved grievances supports FAIL. |
| 8 — Outstanding executive relations | Ask a former employee about `top_management`. Fisher emphasizes trust, cooperation, fair advancement and constructive handling of friction. | PASS when executives demonstrate trust, teamwork or constructive conflict resolution without unresolved serious negative evidence. Persistent factional conflict, refusal to communicate or conflict-driven departures supports FAIL. |

The model interprets the meaning of each interview. Unionization alone does not establish poor labor relations, and absence of strikes does not establish good relations. Normal executive disagreements can coexist with good teamwork when they are resolved constructively. External recruitment alone is neutral. Product success alone does not establish sales excellence. Unmentioned factors remain unknown. Mixed evidence is assessed for severity, persistence and resolution. If available evidence cannot establish PASS, the result is FAIL with an explicit evidence limitation.

Only the interview tool is used because Section 3.3 assigns all three selected points to interviews and states that informed respondents agree. Documents and SQL are available but are not required to answer these selected topics; they would not automatically provide independent evidence about interpersonal behavior.

These are operational rules derived from Fisher's discussion. Interviews may omit compensation, promotion policies or training, so a verdict is limited to the supplied evidence. It does not establish every aspect of a real company's organizational quality.

### Design

```mermaid
flowchart TD
    A[Provided runner loads company tools] --> B[Student workflow selects informed roles]
    B --> C[Customer: sales_team]
    B --> D[Current employee: working_conditions]
    B --> E[Former employee: top_management]
    C --> F[LangChain ChatDeepSeek: deepseek-v4-flash]
    D --> F
    E --> F
    F --> G[Parse JSON and validate each verdict and reason]
    G -->|Invalid or missing points| H[Retry missing points once using existing interviews]
    H --> G
    G -->|Valid results or retry exhausted| I[Return company verdicts and combined summary]
    I --> J[Provided runner reads public answer key]
    J --> K[Write results.csv and calculate balanced accuracy]
```

The workflow plans three interviews per company and invokes the provided LangChain interview tool with the appropriate role and topic. One model request normally evaluates all three interviews together under the Fisher-based prompt. Up to five companies are processed concurrently. The model returns JSON containing three verdicts and evidence-based reasons; each valid point is retained, and missing or invalid points receive one additional application-level attempt without repeating interviews. API calls have a 60-second request timeout and one client retry. Persistent model failures are recorded in the summary and unresolved points remain missing, which the provided scorer correctly treats as wrong. An outer per-company exception handler also preserves a record for unexpected ordinary errors, so another company can continue. Explicit task cancellation is allowed to propagate. Verdict strings are stripped of surrounding whitespace and normalized to uppercase before validation. The model receives interview evidence and rules; the public answer key is loaded by the provided runner after analysis.

### Public results

On 19 November 2026, the published fork was cloned directly from GitHub into a clean directory and run using the existing Python environment. The run processed all 20 public companies in 12.5 seconds, with **60/60 correct and 100.0% balanced accuracy**, three interviews per company and no missing verdicts. The cloned submission files matched the local files, and results.csv was generated successfully. This is a public-sample result, not a private-test guarantee.

| Point | Correct / total | True PASS correctly identified | True FAIL correctly identified |
| --- | --- | --- | --- |
| 4 | 20/20 | 14/14 | 6/6 |
| 7 | 20/20 | 14/14 | 6/6 |
| 8 | 20/20 | 14/14 | 6/6 |
| Total | 60/60 | 42/42 | 18/18 |

Balanced accuracy = 0.5 × (42/42 + 18/18) = **100%**. There are **no companies whose final verdicts disagree with the public answers**, so there is no final-run disagreement list to explain.

Earlier development considered points 1, 2 and 5. The model struggled to distinguish future growth potential from historical product sales, and ongoing development from repeated sales of previously launched products. I changed the selected points after reviewing Fisher and the available evidence. Interview topics align more directly with points 4, 7 and 8. The final workflow uses semantic interpretation rather than company-specific answers or exact phrase matching. Public-test success still leaves uncertainty about new wording, mixed evidence and API failures.

### Reproduction and reference

Use Python 3.10 or later and install `requirements.txt` in a virtual environment. Set `DEEPSEEK_API_KEY` in the environment or a local `.env` file. This implementation also accepts the local name `HW3`, which takes precedence if both are present.

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
python3 hw3.py --data-folder public_test
```

The script writes `results.csv` in its working directory. Keep credentials in the environment or ignored `.env` file. All company evidence is accessed through the provided tools. The provided tools, runner and scoring functions retain their original code.

Reference: Philip A. Fisher, *Common Stocks and Uncommon Profits*, Part One, Chapter 3, points 4, 7 and 8, using the supplied edition. The descriptions above are paraphrases, not verbatim quotations.

Task 2 is included in [REFLECTION.md](REFLECTION.md).
