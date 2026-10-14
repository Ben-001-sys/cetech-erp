# CETECH ERPNext — Exploration, Configuration and Test Register

**Environment:** Hetzner via Dokploy, site `erpnext.benuel.dedyn.io`.
**Purpose:** Sandbox evaluation of ERPNext capabilities, configuration, security, and operational workflows; **not production accounting or real CETECH transactions**.
**Date opened:** 2026-10-14.
**Installed evidence:** Frappe 16.52.0, ERPNext 16.50.0; site created and onboarding wizard completed; scheduler enabled; HTTPS sign-in and Setup Home UI visible. Admin UI login succeeded (setup workspace visible). Current Compose instance: `cetecherpnext-erpfrontend-otl3fx`.

## Status meanings

- **RESEARCHED** — official documentation reviewed; functionality not yet exercised.
- **CONFIGURED** — settings saved and observed in the UI.
- **TESTED** — positive/negative functional test with outcome and evidence.
- **BLOCKED** — issue prevents a test.
- **DEFERRED** — intentionally postponed, with reason.

## Working approach

1. Keep all demonstration customers, suppliers, items, invoices, stock and payments clearly prefixed **TEST-**. Do not send customer communications or enter real financial transactions.
2. Read the purpose of each screen, record existing defaults, change only one relevant setting at a time, and validate the result.
3. Do not invite external users before outgoing email/SMTP is configured. Create dedicated low-privilege **System User** test identities for access-control testing; do not use Administrator for ordinary workflow acceptance.
4. Test positive and negative role permissions, including attempts to access modules and records that should be prohibited.
5. Capture outcomes in this register; mark **TESTED** only after evidence exists. Back up site/database and preserve encryption keys before destructive tests.
6. Do not delete Docker volumes or rerun site creation; verify recovery from a separate environment before real-world use.

## Exploration matrix

| Area | Focus | Status | Evidence / next action |
| --- | --- | --- | --- |
| Company | Name, abbreviation, country, GHS, fiscal year, chart of accounts, warehouse defaults | CONFIGURED | 2026-10-14: Company form observed as CETECH Ghana; details reviewed, Accounts tab not yet inspected |
| System Settings | Time zone, language, number/date format, site defaults, password policy | RESEARCHED | Review against Africa/Accra and CETECH sandbox needs |
| Organization | Departments, organizational structure, employees/contacts | RESEARCHED | Examine available models; do not import real staff |
| Access | System vs Website users, roles and role profiles, permissions, restrictions | RESEARCHED | Two test users, allow and deny checks |
| Accounts | Chart of accounts, cost centers, tax templates, journal and GL impact | RESEARCHED | Review defaults; then post labeled test data |
| Selling | Customer, item, quotation, order, invoice, payment | RESEARCHED | Test complete sale cycle after prerequisites |
| Buying | Supplier, material request, purchase order, receipt, invoice | RESEARCHED | Test approval and purchasing cycle |
| Stock | Warehouses, inventory, valuation, entries, transfers | RESEARCHED | Controlled test material receipt/transfer |
| Projects | Project, task, milestones, timesheets and ownership | RESEARCHED | Test task/project workflow |
| Assets | Asset category, purchase, assignment, lifecycle | RESEARCHED | Test sample asset |
| Manufacturing | BOM, work order, production | DEFERRED | Only if CETECH workflow needs manufacturing |
| Quality | Inspection and quality procedures | DEFERRED | Prioritize after stock/purchasing |
| Printing | PDF print formats, letterhead, company branding | RESEARCHED | Test printable records |
| Email | SMTP, queue, templates, notifications | BLOCKED | No outgoing email verified yet; do not assume delivery |
| Integrations | ERPNext API, Nextcloud/n8n future workflows | DEFERRED | After core role and data model validation |
| Operations | HTTPS, persistent volumes, scheduler, backups and restore | CONFIGURED | HTTPS/login and scheduler observed; backups/restore still outstanding |

## First interactive checkpoint

1. Click **Setup > Company** and open the existing company created by the wizard.
2. Record Company name, abbreviation, default currency, country, parent/group flag and accounting defaults.
3. Compare them to the intended sandbox configuration and do not create a second company merely to test.
4. Capture a screenshot with no personal data or secrets, update status after evidence is reviewed.

## Sources

- https://docs.frappe.io/erpnext/setting-up
- https://docs.frappe.io/erpnext/company-setup
- https://docs.frappe.io/erpnext/system-settings
- https://docs.frappe.io/erpnext/adding-users
- https://docs.frappe.io/erpnext/permissions
- https://docs.frappe.io/erpnext/role-and-role-profile
- https://docs.frappe.io/erpnext/selling
- https://docs.frappe.io/erpnext/stock-entry
- https://docs.frappe.io/erpnext/email-account

## Evidence update — 2026-10-14: Company details reviewed

**Source:** User-provided ERPNext Setup > Company > Details screenshot. **Status:** CONFIGURED / VISUALLY VERIFIED (not transaction-tested).

- Company name displayed: **CETECH Ghana**; abbreviation **CETECH-GH**.
- Country **Ghana**; default and reporting currency **GHS**; **Is Group** unchecked; Parent Company unset.
- Default Letter Head **Company Letterhead - Grey**; Default Holiday List **Christmas Holiday** (contents not yet checked); website/domain field `cetechbpa.com/gh` (purpose and correctness not yet verified).
- Date of Establishment empty; Tax ID/registration details present but intentionally **not copied into this public repository**.
- Activity history indicates default Cash `Cash - CETECH-GH`, Receivable `Debtors - CETECH-GH`, Payable `Creditors - CETECH-GH`, and Payment Discount `Cash - CETECH-GH`. **Accounting review required**: Cash is generally not an appropriate payment-discount posting account; verify type/usage before test transactions.
- Activity history shows Monthly Sales Target of **GHS 500,000** and Credit Limit of **GHS 1,000**. Confirm whether these are intentional sandbox values before workflow tests.
- **Next action:** Open the existing Company record's **Accounts** tab, capture the settings without pressing Save or editing; review defaults, discount postings, cost centers, write-off/exchange gains, and tax configuration. Then proceed to Buying and Selling and Stock tabs.

**No credentials, official tax ID values, or personal information should be committed to this public register.**

## Evidence update — 2026-10-14: Company > Accounts tab

**Source:** Screenshot of CETECH Ghana > Accounts (user-provided). **Status:** CONFIGURED / REVIEWED (not transaction-tested).

- Chart of Accounts created from **Standard Template**, template **Standard**.
- Bank account **unset**; Cash account **Cash - CETECH-GH**.
- Receivable **Debtors - CETECH-GH**, Payable **Creditors - CETECH-GH**.
- Sales income **Sales - CETECH-GH**, Cost of Goods Sold **Cost of Goods Sold - CETECH-GH**, Write Off **Write Off - CETECH-GH**.
- Default Cost Center **Main - CETECH-GH**; Default Payment Discount Account **Cash - CETECH-GH**: **HIGH-PRIORITY ACCOUNTING REVIEW**, likely incorrect posting account for settlement discounts. Do not submit test payments with a discount before rectification.
- Exchange Gain/Loss and Round Off accounts populated; Unrealized Exchange Gain/Loss account unset; revaluation automation unchecked.
- Round-off Cost Center **Main - CETECH-GH**. Default Payment Terms Template, Finance Book, deferred revenue and deferred expense accounts are blank.
- **Book Advance Payments in Separate Party Account** unchecked; allocation preference **Oldest of Invoice or Advance**.
- Asset depreciation/disposal/Capital Work In Progress/Asset Received Not Billed defaults mostly present; some text truncated in screenshot so exact account names require UI check.
- **Exception Budget Approver Role** unset; Series for Asset Depreciation Entry unset.
- **Next check:** Open Chart of Accounts under Accounting. Inspect whether `Discount Allowed - CETECH-GH` (expense), `Discount Received - CETECH-GH` (income), valid bank ledger(s), and other account types exist. Verify posted ledger structure in sandbox only after defaults are appropriate. Do not create financial transactions or change the default Discount Account to Cash again.

Source references:
- https://docs.frappe.io/erpnext/company-setup
- https://docs.frappe.io/erpnext/managing-invoice-discount-in-the-payment-entry
- https://docs.frappe.io/erpnext/accounts-settings
- https://docs.frappe.io/erpnext/chart-of-accounts
