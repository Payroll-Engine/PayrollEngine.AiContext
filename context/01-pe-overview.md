# Payroll Engine — Overview

## What is Payroll Engine?

Payroll Engine (PE) is an open-source, regulation-based payroll calculation framework built on .NET. A single engine instance can run regulations for multiple countries simultaneously.

- License: MIT
- Runtime: .NET
- API: REST (OpenAPI / Swagger)
- Website: https://payrollengine.org

## Core Concepts

### Tenant
Top-level isolation unit. Each tenant has its own regulations, employees, payrolls, and data. Multi-tenant deployments share one backend instance.

### Employee
An employee belongs to a tenant and a division. Employee data (salary, address, tax class, etc.) is managed as **case values** — typed, time-aware fields with validity periods.

### Payroll
A payroll defines which regulation layers apply and in which order. Employees are assigned to a payroll via their division.

### Payrun
A payrun is the configuration for recurring payroll execution (monthly, weekly, etc.). A **Payrun Job** is a single execution instance — it calculates wage types and collectors for all assigned employees.

### Wage Type
A single payslip line item. Examples: `BaseSalary`, `IncomeTax`, `SocialSecurity`.

### Collector
Aggregation bucket accumulating wage type results. Examples: `GrossIncome`, `Deductions`, `NetPay`.

## Two User Roles (MCP Context)

| Role | Description | Primary Tools |
|:-----|:------------|:--------------|
| **Provider** | EOR, Bureau, PEO, HCM vendor integrating via REST API or MCP Server | REST API, MCP Server, WebApp |
| **Automator** | DevOps / integration engineer | `pecmd` CLI, Client SDK, MCP Server |

## Architecture

```
REST API (PayrollEngine.Backend)
  ├── Payroll Engine Core (calculation)
  ├── Persistence (SQL Server / MySQL)
  └── Scripting Runtime (Roslyn)

WebApp (Blazor)
  └── Provider UI — case entry, payrun execution, reports

MCP Server OSS (Read-Only)
  └── AI agent read access to payroll data via natural language

MCP Server Pro (Read + Write)
  ├── All OSS read tools
  └── Write tools: employee management, case changes, payrun execution
  └── Consolidation tools: cross-tenant payroll aggregation (MultiTenant)

pecmd CLI
  └── Exchange import, test execution, report export
```

## Global Cases

Global Cases provide a standardised, country-agnostic integration interface for data onboarding. Each payroll domain (identity, employment, salary, tax, social security, banking, …) is assigned a **cluster tag** prefixed with `GC.`:

| Cluster Tag | Domain | Example Fields |
|:------------|:-------|:---------------|
| `GC.Employee` | Identity / personal data | NationalId, TaxId, DateOfBirth, Gender |
| `GC.Employment` | Contract, dates, hours | StartDate, EndDate, WeeklyHours, EmploymentType |
| `GC.Salary` | Remuneration | MonthlySalary, HourlyRate, PayType |
| `GC.Tax` | Tax withholding | TaxClass, Religion, TaxCode |
| `GC.SocialSecurity` | Social insurance | InsuranceType, HealthInsuranceMandatory |
| `GC.Banking` | Bank account | Iban, AccountNumber, SortCode |
| `GC.Pension` | Pension / bAV | PensionScheme, EmployeeContribution |
| `GC.Time` | Period hours | OvertimeHours, NightHours, SickDays |
| `GC.Benefits` | Benefits in kind | CompanyCar, MealVouchers |
| `GC.Garnishment` | Court orders | GarnishmentAmount, GarnishmentType |
| `GC.Termination` | Exit / ETP | TerminationDate, SeverancePay |
| `GC.Leave` | Leave entitlements | VacationDays, LeaveBalance |

### How it works

1. **Regulation authors** tag their country-specific cases with `GC.*` clusters (e.g. Germany's `Arbeitnehmer` case carries `clusters: ["GC.Employee"]`).
2. **Adapters** write data using the domain cluster tags — the framework resolves `GC.Employee` to the country-specific case name (`DE.Arbeitnehmer`, `UK.PersonalDetails`, etc.) at runtime.
3. **MCP/API consumers** query available cases per domain via `clusterSetName=GlobalCase` or the `build_case` tool.

This allows the **same adapter code** to onboard employees into any country regulation without knowing the target case names.

## MCP Server Access Control

The MCP Server exposes tools filtered by two independent dimensions:

| Dimension | Controls |
|:----------|:---------|
| **Isolation Level** | Which records are accessible (`MultiTenant` / `Tenant` / `Division` / `Employee`) |
| **Permission** | Which tools are registered (`Read` / `Write` / `None`) per role |

See `05-mcp-tools.md` for the full tool inventory, permission matrix, and isolation level behaviour.
