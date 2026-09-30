# assessment

Loom Link: https://www.loom.com/share/855a3022e2a247038641eb1dacb4b84b

## Files

| File | What it is |
|---|---|
| `Task1_QA_Report_Arbaj.docx` | Bug table (5 issues) and root-cause analysis |
| `Task2_Workflow_Arbaj.json` | n8n workflow exported from n8n |

## Task 1: Web App QA & Debug Report

**App tested:** OWASP Juice Shop (official online demo), an intentionally vulnerable app, tested manually in the browser only.

| # | Issue | Severity |
|---|---|---|
| 1 | Login bypass via SQL injection | Critical |
| 2 | DOM-based XSS in the search box | High |
| 3 | Public `/ftp` directory listing | High |
| 4 | Error page leaks a stack trace | Medium |
| 5 | Weak passwords accepted at registration | Medium |

**Root-cause analysis (Issue 1):** user input is pasted straight into the SQL query, so `' or 1=1--` makes the login check always true. The fix is parameterized queries, generic error messages, and rate limiting. Full details are in the Word report.

## Task 2: n8n API Integration Workflow

**What it does:** every day at 8:00 AM it builds a "morning brief" of the top GitHub repos for the topic `n8n` and emails it once.

| Step | Node | Purpose |
|---|---|---|
| Trigger | Schedule Trigger | Runs once a day at 8:00 AM |
| API call 1 | Search GitHub Repos | GitHub REST API, top repos for the topic sorted by stars |
| Transform | Top 5 Repos (Code) | Sorts, keeps the top 5, reshapes fields |
| API call 2 | Get README | Fetches each repo's README (second endpoint of the same API) |
| Digest | Build Digest (Code) | Merges repo data with a README snippet, escapes HTML |
| Branch | Stars Threshold Check (IF) | Top repo above 1000 stars sends a "[HOT]" email, otherwise a "quiet day" email |
| Output | Send Email (SMTP) | Delivers the digest |

**Why these choices:** the GitHub API is free, needs no key, and is relevant to a dev team's daily brief. Email was chosen as the output because it is easy to verify. A single email at 8 AM matches how a morning brief is read. The 1000-star threshold comes from the assignment example, and the other branch is tested by raising it.

**Error path:**
- If the search call fails (rate limit, API down) or returns no repos, the error output routes to an **alert email** instead of failing silently.
- If a single README fetch fails, "Continue On Fail" lets the run continue, and that repo shows "README unavailable".
- Known limit: if the email send itself fails, the failure appears only in n8n's execution log.

**Credentials:** the SMTP login is stored in n8n's Credentials store. No secrets are in the exported JSON, and email addresses are replaced with placeholders.

**To run it:** import the JSON in n8n, attach an SMTP credential to the 3 email nodes, set your address, and click Execute workflow for a manual test. Set the workflow to Active for the daily 8 AM run.

## Tests performed

| Test | Result |
|---|---|
| Happy path | [HOT] digest email received |
| Threshold raised to 999999999 | "quiet day" email received |
| Search URL broken on purpose | Alert email received |
