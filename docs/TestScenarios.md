# AI Context — Test Scenarios

Practical scenarios for verifying that the AI context package works correctly.
Each scenario can be run independently using a Claude.ai Project.

---

## Scenario 1 — Regulator: Generate a Regulation

**Role:** Regulator

**Setup — Claude.ai Project Knowledge:**
```
context/01-pe-overview.md
context/02-exchange-schema.md
context/03-scripting-api.md
context/04-nocode-actions.md
context/06-conventions.md
context/07-examples.md
prompts/regulator.md
```

**Test prompt:**
```
Create a country-neutral regulation for a company with:
- Monthly salary (CalendarPeriod)
- 13th month bonus paid in December (Moment)
- Progressive income tax via range lookup (10% up to 5000, 20% above)
- No-Code only, no C# scripting

Include a payroll test for November 2025 (no bonus) and December 2025 (with bonus).
```

**What to verify:**

| Check | Expected |
|:------|:---------|
| `$schema` reference present | ✓ |
| `createdObjectDate` present | ✓ |
| All names PascalCase | ✓ |
| Bonus field uses `"timeType": "Moment"` | ✓ |
| Salary field uses `"timeType": "CalendarPeriod"` | ✓ |
| `GetRangeLookup` used for tax calculation | ✓ |
| Test file covers November (no bonus) and December (with bonus) | ✓ |
| No invented API methods or property names | ✓ |

**Note:** The MCP Server is not involved. Claude generates Exchange JSON as text output. The regulator copies it to a file and imports it with `pecmd`. No MCP call is made.

**Backend validation (optional):**
```powershell
pecmd import GeneratedRegulation.json
pecmd employeeTest Test.et.json
```

---

## Scenario 2 — Provider: Employee Payslip Assistant

**Role:** Provider

**Setup — API system prompt:**
```
prompts/provider.md
context/05-mcp-tools.md
```

**MCP Server configuration:**
```json
{
  "McpServer": {
    "IsolationLevel": "Employee",
    "TenantIdentifier": "acme-corp",
    "EmployeeIdentifier": "mario.nunez@acme.com",
    "Permissions": {
      "HR":      "Read",
      "Payroll": "Read"
    }
  }
}
```

**Test prompt (from employee):**
```
Why is my salary lower in March than in February?
```

**Expected agent behaviour:**
1. Calls `get_consolidated_payroll_result` for February
2. Calls `get_consolidated_payroll_result` for March
3. Compares wage type results between the two periods
4. Explains the difference in plain language, e.g.: *"In March the performance bonus from February was not paid again — it is a one-time Moment value. This accounts for the difference of CHF 1,200."*

**What to verify:**

| Check | Expected |
|:------|:---------|
| Agent uses `get_consolidated_payroll_result`, not a hallucinated tool | ✓ |
| Agent correctly identifies Moment as one-time | ✓ |
| Explanation is in the employee's language | ✓ |
| No payroll data from other employees returned (Employee isolation) | ✓ |

**Note:** This scenario uses only read operations. The PEOS MCP Server remains Read-Only.

---

## Scenario 3 — Automator: CI/CD Debugging

**Role:** Automator

**Setup — system prompt:**
```
prompts/automator.md
context/06-conventions.md
```

**Test input — CI pipeline failure log:**
```
Test failed: EmployeeTest 'Test.et.json'
  WageType 210 expected: 450.00, actual: 0.00
```

**Test prompt:**
```
What could cause WageType 210 to return 0 instead of 450 in an employee payroll test?
```

**Expected response:**
Claude identifies the most likely causes based on PE knowledge:
- Collector `GrossIncome` not filled (upstream wage type returned `null`)
- `CaseValue` missing for the test period (case data not imported)
- `retroPayMode=None` silently consumed the pending mutation
- Wrong `createdObjectDate` causing CalendarPeriod resolution to miss the value

And suggests concrete `pecmd` debug commands:
```powershell
pecmd import Payroll.Values.json        # re-import case values
pecmd employeeTest Test.et.json         # re-run test
```

**What to verify:**

| Check | Expected |
|:------|:---------|
| Mentions `retroPayMode=None` as a known silent-failure cause | ✓ |
| Mentions `createdObjectDate` / CalendarPeriod resolution | ✓ |
| Suggests `pecmd` commands, not generic .NET debugging | ✓ |
| Does not invent non-existent PE configuration options | ✓ |

**Note:** No MCP Server involved. Claude analyses log output and provides diagnosis based on PE domain knowledge from the context files.

---

## Common Verification Criteria

Across all scenarios, the context is working correctly when Claude:

- Uses **PascalCase** for all payroll object names without being reminded
- Applies the correct **time types** (`Moment`, `CalendarPeriod`, `Period`, `Timeless`) without guessing
- References **real API methods** from `03-scripting-api.md` (e.g. `GetRangeLookup`, `CaseValue[]`)
- Uses **real MCP tool names** from `05-mcp-tools.md` (e.g. `get_consolidated_payroll_result`)
- Knows that **PEOS MCP is Read-Only** and does not suggest write operations via MCP
- Applies **`createdObjectDate`** in generated Exchange JSON without being asked
