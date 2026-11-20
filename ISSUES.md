# Issues Analysis

> **Instructions**: Document the three highest-priority product issues and the three highest-priority code issues. Rank the issues within each category from highest to lowest priority.
> Replace the example entries with your actual findings and explain why each issue deserves its priority. Complete all four questions in the Edge Case Analysis section.
> Rename this file to `ISSUES.md` before submitting.

## Summary

| Type | Critical | High | Medium | Low | Total |
|------|----------|------|--------|-----|-------|
| Product Issues | 0 | 0 | 0 | 0 | 0 |
| Code Issues | 0 | 0 | 0 | 0 | 0 |

---

## Product Issues

> Product issues are user-facing problems: broken functionality, missing validation, poor UX, data integrity risks visible to users.

### [P1] Example: Missing Confirmation Dialog

**Severity**: Medium

**Location**: `problems/templates/problems/index.html` (migrate buttons)

**Description**:
When clicking "Migrate to Azure", the action executes immediately without asking for confirmation. This could lead to accidental data overwrites.

**Impact**:
- Users might accidentally overwrite test cases
- No way to cancel a mistaken click
- Potential data loss

**Suggested Fix**:
Add a JavaScript confirmation dialog before form submission for destructive actions.

---

### [P2] update the cloud storage

**Severity**: Medium

**Location**: `problems/storage/gdrive_backend.py`

**Description**:
Facing issue with creating account with cloud services 

**Impact**:
it will be impact on ne developers to understand

**Suggested Fix**:
update the env file with respected storage cloud

---

### [P3] Your Issue Title Here

**Severity**: Critical / High / Medium / Low

**Location**: `filename.py:line_number`

**Description**:
[What is the issue?]

**Impact**:
[What goes wrong? Who is affected?]

**Suggested Fix**:
[How would you fix it?]

---

<!-- Keep this section to three product issues, ranked from highest to lowest priority. -->

---

## Code Issues

> Code issues are technical problems: bugs, security vulnerabilities, performance problems, code quality concerns, architectural issues.

### [C1] Example: Unused Imports

**Severity**: Low

**Location**: `problems/views.py:6-8`

**Description**:
```python
from bs4 import BeautifulSoup  # Never used
from django.core.cache import cache  # Never used
```
These imports are declared but never used in the file.

**Impact**:
- Slightly increases memory usage
- Makes code harder to understand (suggests these modules are used when they're not)
- May cause confusion during code review

**Suggested Fix**:
Remove unused imports. Consider using a linter (flake8, ruff) to catch these automatically.

---

### [C2] Your Issue Title Here

**Severity**: Critical / High / Medium / Low

**Location**: `filename.py:line_number`

**Description**:
[What is the issue?]

**Impact**:
[What goes wrong? What's the risk?]

**Suggested Fix**:
[How would you fix it?]

---

### [C3] Your Issue Title Here

**Severity**: Critical / High / Medium / Low

**Location**: `filename.py:line_number`

**Description**:
[What is the issue?]

**Impact**:
[What goes wrong? What's the risk?]

**Suggested Fix**:
[How would you fix it?]

---

<!-- Keep this section to three code issues, ranked from highest to lowest priority. -->

---

## Edge Case Analysis

### Q1

> A Polygon problem has 0 sample test cases but 15 regular test cases. What happens when you migrate this problem?

The problem can be migrated to the database even when it has zero sample test cases. Sample test cases are handled separately during problem migration. The regular test cases are migrated using the "Migrate Test Cases to DB" button. The final result depends on how the migration code handles regular test cases when no sample cases are available.

---

### Q2

> A problem is migrated with 20 test cases. Later, the problem setter removes 8 test cases on Polygon (now 12 remain). The problem is re-migrated. What happens?

During re-migration, the existing test cases are loaded and updated based on their order. The 12 test cases from Polygon update the first 12 existing database records. If the migration code does not delete the remaining records, the other 8 outdated test cases remain in the database. This can cause a mismatch between Polygon and the database.

---

### Q3

> Two different Polygon problems have the exact same title: "Two Sum". You migrate the first one successfully. Then you try to migrate the second one. What happens?

The migration searches for an existing problem using its Polygon ID (`polygon_id`), not its title. Since the two problems have different Polygon IDs, the second problem should be created as a separate database record. Both problems can have the same title without overwriting each other.

---

### Q4

> When test cases are saved to the database via "Migrate Test Cases to DB", some data is intentionally discarded. What data is lost? Why might this cause problems?

During migration, the code saves selected test-case fields such as input, output, sample status, and description. Other original Polygon test-case information may not be stored in the database. Losing this information can make it difficult to reproduce the original test-case configuration or debug problems later.
---

## Severity Guidelines

Use these definitions when assigning severity:

| Severity | Definition | Examples |
|----------|------------|----------|
| **Critical** | System broken, security vulnerability, data loss | SQL injection, authentication bypass, data corruption |
| **High** | Major functionality broken, significant data integrity risk | Feature doesn't work, orphaned records, race conditions |
| **Medium** | Feature partially broken, poor UX, code maintainability | Missing validation, confusing errors, code duplication |
| **Low** | Minor issues, cosmetic, best practice violations | Unused imports, inconsistent formatting, missing logs |

---

## Notes

[Add any additional observations, patterns you noticed, or architectural concerns that don't fit into specific issues above]
- The migration process uses Polygon as the source for problem details and test cases.
- Redis is used to cache test cases before migration.
- Test cases are stored in the database and can also be uploaded to Google Drive.
- Re-migration may cause data consistency issues if outdated database records or cloud files are not removed.
- The database and cloud storage screenshots show different test-case counts, but it is not confirmed whether they belong to the same migration run.
- Some edge-case behaviors are based on code-flow understanding and have not been independently tested.