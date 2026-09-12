---
name: db-routing
description: Routing map and SQL syntax reference across sam_hub's 9 live database connections (1 MariaDB + 8 MSSQL). Load when a question needs picking the right connection or writing cross-DB-aware SQL.
version: 1.0.0
---

# sam_hub Database Routing

## Connection → Data Map

| Connection | Server | Type | Contains |
|---|---|---|---|
| `mariadb-210` | 10.10.10.210 | MariaDB | Frappe/ERPNext HR app (~1300 `tab*` DocType tables) plus ~570 custom tables for employee master (`samempmst`), leave/attendance, and RK (ratnakalakar/polisher) production-salary tracking (`Pol_RK_Performance_Data_New`, `RK_Deduction`, `webpolish_labour`, `sderp_*`) — HR/payroll-adjacent, not the main production DBs |
| `sql-123-mis` | 10.10.0.123 | MSSQL | Cross-stage MIS/reporting warehouse spanning rough→polish→SBA: `Kapan`, `Rough`, `Pkt`, `RMSKapanData`, `PolishKapanData`, `JWIsRtn`, `RejIsRtn`, `SBA_*`, `GMgrVarEAV`, `Subsection_Month_Pcs_SMP` — summary/audit tables for management reporting, not the live operational ledgers |
| `sql-123-rms` | 10.10.0.123 | MSSQL | Rough Management System — the rough-side process ledger: Galaxy scan (`Gal_*`), Signer/Planner (`PlannerPlan*`), QC/checker mistake tracking, group-leader production, `IsRtn` (43M-row issue/return ledger), Laser/laser-photo checking |
| `sql-123-pams` | 10.10.0.123 | MSSQL | Polish Assortment Management System — polish-side assort & result ledger: `assortment_issue/return_*`, `PolAss*` stock/result tables, `result_head/detail`, `M2M`, `RateMaster*`, `JangadMaster`, charni/purity result tracking |
| `sql-123-payroll` | 10.10.0.123 | MSSQL | Employee payroll/HR system — attendance (`Emp_attendance_for_date`, `EmpInOut`), leave, salary (`Emp_Salary`, `GrossSalary`), loans/advances, Form16/TDS, KYC, gratuity |
| `sql-123-polish` | 10.10.0.123 | MSSQL | Polish floor production operations — monthly Kapan-detail partitions (`Kd_2024_*`…`Kd_2026_*`), per-process chain tables (Table/Russian/Athapel/4P/Repair opr), P4 photo/QC checking, docket tracking — finer-grained than PAMS's assort/result view |
| `sql-123-ms` | 10.10.0.123 | MSSQL | Maintenance/service management — equipment master (`EqpMaster`), depreciation, preventive maintenance (`PMS`), breakdown/service calls (`ServiceCall`, `PowerBreakDown`), vendor master — plant/equipment upkeep, not diamond stock |
| `sql-183-fda` | 10.10.10.183 | MSSQL | Financial accounting/Tally-linked ledger — `FDA_GL_Entry`, `FDA_Ledger`, `TL_Accounting`, `TL_Bill`, `TL_Voucher`, `Trial_Balance` — general ledger and accounts, not diamond-trade ops |
| `sql-183-sales` | 10.10.10.183 | MSSQL | Mumbai sales/M2M ledger — same PAMS-style assort/result/rate schema plus sales-specific tables: `SalesHead/Detail`, `CustomerMaster`, `memo_sales_*`, `memo_outward_*`, `AngadiaEntMast`, `JangadMaster`, `M2M` — where polished stock sales, memo, and Angadiya movement to Mumbai are tracked |

## MariaDB vs MSSQL syntax

| Feature | MariaDB | MSSQL |
|---|---|---|
| Limit rows | `LIMIT 100` | `TOP 100` |
| Describe table | `DESCRIBE table` | `sp_help 'table'` |
| Current datetime | `NOW()` | `GETDATE()` |
| If-null | `IFNULL(a,b)` | `ISNULL(a,b)` |

## Rules

- All 9 connections are read-only (enforced in each connector's own code).
- Same-server MSSQL databases (123, or 183) can be queried cross-DB with three-part names if the account has grants; different servers normally need their own connection.
- Exception: `sql-123-*` has a real SQL Server Linked Server named `MARIADB` pointing at `mariadb-210`'s database (`_3b41b400ef4d007a`). A plain `SELECT` through `sql-123-*` can reach 210's tables directly this way, e.g. `SELECT * FROM [MARIADB].[_3b41b400ef4d007a]..[tabEmp_Mst]` (double-dot skips the schema, defaults to `dbo`). This is a normal `SELECT` with no forbidden keyword, so sam_hub's connector allows it — it is a genuine second route into 210 data, alongside the direct `mariadb-210` connection. Not yet verified: whether writes are blocked on this path the same way (the linked server may use a different remote login than `mariadb-210`'s own `devro` account) — check this live.

