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

## MCP Server Access Control

The MCP Server exposes tools filtered by two independent dimensions:

| Dimension | Controls |
|:----------|:---------|
| **Isolation Level** | Which records are accessible (`MultiTenant` / `Tenant` / `Division` / `Employee`) |
| **Permission** | Which tools are registered (`Read` / `Write` / `None`) per role |

See `05-mcp-tools.md` for the full tool inventory, permission matrix, and isolation level behaviour.
