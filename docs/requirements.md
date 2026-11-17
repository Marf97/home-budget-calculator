# Fixed product and engineering requirements

This document carries the settled execution specification for implementation workers. It records decisions, not evidence that an application or its checks exist yet.

## Product scope
- Single-household monthly category planner; no dated transaction register.
- Czech end-user UI, validation, empty states, chart labels and user guide; locale `cs-CZ`; currency CZK only.
- Primary dense spreadsheet-like monthly worksheet, with income/expense sections, group subtotals, sticky headers, right-aligned amounts, keyboard-friendly cells, save/unsaved/error state, explicit save and dirty-navigation protection.
- Views: **Měsíční plán**, **Přehled**, **Kategorie**. Monthly actual/planned income, expenses and difference (income minus expenses, not balance/net worth); expense category bars; planned/actual comparison; rolling 12-calendar-month income/expense/net trend ending at selected month. Variance is actual minus planned for both kinds. Tables/text alternatives reconcile with charts. No fabricated history, zero-income percentages, or transaction view.
- Category taxonomy is editable and starts with categories only, never financial values:
  - Příjmy: Mzda; Vedlejší příjmy; Dávky a příspěvky; Ostatní příjmy.
  - Bydlení: Hypotéka / nájem; Údržba domácnosti; Pojištění domácnosti.
  - Energie a služby: Vytápění; Elektřina; Voda; Internet a telefon.
  - Jídlo a domácnost: Potraviny; Restaurace a rozvoz; Drogerie a potřeby pro domácnost.
  - Doprava: Palivo; Veřejná doprava; Servis auta; Pojištění auta; Parkování.
  - Zdraví: Léky; Lékařská péče.
  - Rodina a vzdělávání: Péče o děti; Škola a kroužky.
  - Osobní a volný čas: Oblečení; Sport a kultura; Předplatné; Dovolená; Dárky.
  - Ostatní výdaje: Poplatky; Splátky ostatních úvěrů; Ostatní.
- No transaction register, CSV, bank/chat ingestion, externally supported/public integration API, formula engine, accounts, transfers, net-worth, investment tracking, multi-currency, bank sync, or cloud processing. The same-origin frontend/backend transport below is private application plumbing, not a public API.

## Fixed technical stack and privacy
- React + TypeScript + Vite frontend; Node.js 24 + TypeScript backend; npm lockfile; server-side SQLite.
- Bundle runtime dependencies/assets locally; no analytics/CDNs/external fonts/cloud processing.
- Production container serves frontend/backend same-origin. SQLite stored outside the image in persistent volume. Compose publishes `127.0.0.1:3000` only; container-internal `0.0.0.0` is acceptable. No LAN/public exposure, authentication platform, deployment, or OS/security changes.
- No auth platform for loopback MVP. Reject unexpected Host/Origin for mutations, no wildcard CORS, parameterized SQL, bounded JSON bodies, no sensitive request-body/value logs. SQLite is unencrypted; explain limits.
- Ignore databases, SQLite WAL/SHM sidecars, backups, secrets, runtime data and `.worktrees`. No actual household data in source, Git, tests, logs or task comments; synthetic fixtures only.

## Data and money contracts
- Category shape: `{id, kind:'income'|'expense', group, name, position, archived}`. Stable opaque IDs; kind immutable. Create, rename, archive; no destructive category deletion. Archive hides unused categories in new worksheets but keeps categories visible in months with data and preserves amounts. Rename preserves historical amounts on same ID.
- Entry shape: `{categoryId, plannedMinor:number|null, actualMinor:number|null}`, unique by month/category. Money is integer haléře, never binary-float arithmetic. `null` means unentered; `0` is explicitly entered. Calculations treat null as zero but UI/empty trend states retain distinction. Clearing cells clears values, not categories.
- Planned amounts are nonnegative. Signed actual totals allow refunds/corrections: negative expense reduces expenses, negative income corrects income; explain in Czech. No transfer/account/net-worth accounting. Validate safe integers, bounded input and safe aggregate totals, valid month and known category IDs server-side.

## Input and paste
- Browser cell editing and rectangular spreadsheet paste only; no CSV or clipboard-read permission.
- Paste begins at selected numeric cell and fills visible category rows and the two editable amount columns. Validate entire rectangle before mutation; reject overflow/invalid cells atomically and preserve prior values on error.
- Parse Czech decimal commas, regular/NBSP grouping spaces and optional `Kč`; document any supported dot-decimal convention unambiguously. No formulas or HTML interpretation. Keyboard Tab/Enter and normal clipboard paste must work.

## Same-origin private transport contract
- `GET /api/categories -> {revision,categories}`
- `POST /api/categories {revision,kind,group,name} -> {revision,category}`
- `PATCH /api/categories/:id {revision,group?,name?,archived?} -> {revision,category}`
- `GET /api/months/:month -> {month,revision,entries}`
- `PUT /api/months/:month {revision,entries} -> {month,revision,entries}` atomically upserts submitted category rows and preserves omitted rows; rejects duplicate IDs; null clears cells.
- `GET /api/reports?from=YYYY-MM&to=YYYY-MM -> {months:[{month,entries}]}` returns every month in inclusive range, including empty months; maximum 120 months. Categories are supplied separately.
- Global category revision for writes. Revision mismatch is HTTP 409; other validation HTTP 400; missing entities HTTP 404. Errors `{error:{code,message}}` use safe Czech message and contain no stack/private values. SQLite write transactions. On conflict preserve unsaved browser input and offer reload, never silently overwrite.

## Storage and recovery
- Persistent server-side SQLite. Document local full backup/restore and exercise against isolated synthetic data. Use SQLite-consistent backup or stop app before copying; never naïvely copy a live DB file. Restore tests must not overwrite real household data.

## Acceptance checklist
- Dense Czech workbook/category management; create/edit/clear/archive/rename, keyboard/paste, validation and durable save/conflict/error behavior.
- Integer money, month/report correctness, null/zero/refunds, year boundaries, empty/zero-income months; tables and charts reconcile.
- Restart persistence; real container with loopback-only published port; isolated full backup/restore test; no sensitive artifacts in Git/logs.
- Unit tests, SQLite/API integration, frontend interaction, production build/typecheck and real-browser synthetic core-flow E2E. Desktop and narrow-screen readability, focus/labels, chart alternatives, accidental English UI audit.
- Reproducible Czech run/backup guide with exact commands and evidence. No paid services, deployment, public exposure, OS/security changes, or agent merge to main.

## Research references
Qualitative discovery only; this is not a statistical ranking of popular categories/charts.
1. Actual Budget reports: https://actualbudget.org/docs/reports
2. Actual Budget custom reports: https://actualbudget.org/docs/reports/custom-reports
3. FinancialAha monthly budget: https://www.financialaha.com/essentials-spreadsheet-templates/personal-budgeting/monthly-budget
4. TheGoodocs simple monthly household budget: https://thegoodocs.com/budget-templates/simple-monthly-household-budget-template.php
5. consumer.gov budget worksheet: https://consumer.gov/system/files/consumer_gov/pdf/1041A_BudgetWorksheet-Fillable.pdf

Research notes: workbook-style editable budget/actual columns, subtotals, prominent income/expense/difference figures and category/month comparisons are useful patterns. Category examples span housing, utilities, food, transport, health, family/school, personal and other. Difference here is cash flow only, not account balance/net worth.
