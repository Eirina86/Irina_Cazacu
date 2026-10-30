# Irina Cazacu — Microsoft Fabric Data Engineering Portfolio

[![AZ-900 Certified](https://img.shields.io/badge/Microsoft-AZ--900%20Certified-0078D4?style=flat&logo=microsoft)](https://learn.microsoft.com/en-us/credentials/certifications/azure-fundamentals/)
[![DP-900 Certified](https://img.shields.io/badge/Microsoft-DP--900%20Certified-0078D4?style=flat&logo=microsoft)](https://learn.microsoft.com/en-us/credentials/certifications/azure-data-fundamentals/)
[![DP-700 In Progress](https://img.shields.io/badge/DP--700-In%20Progress-orange?style=flat&logo=microsoft)](https://learn.microsoft.com/en-us/credentials/certifications/fabric-data-engineer-associate/)
[![DP-600 Planned](https://img.shields.io/badge/DP--600-Planned-lightgrey?style=flat&logo=microsoft)](https://learn.microsoft.com/en-us/credentials/certifications/fabric-analytics-engineer-associate/)
[![Microsoft Fabric](https://img.shields.io/badge/Microsoft%20Fabric-Data%20Engineering-0078D4?style=flat)](https://learn.microsoft.com/en-us/fabric/)

---

## About me

I am a data professional currently pursuing the **DP-700: Microsoft Fabric Data Engineer Associate** certification, with **DP-600: Fabric Analytics Engineer Associate** to follow, building hands-on expertise in designing and implementing scalable analytics solutions on the Microsoft Fabric platform.

**Current certifications:** AZ-900: Microsoft Azure Fundamentals & DP-900: Microsoft Azure Data Fundamentals  

**Target certifications:** DP-700: Microsoft Fabric Data Engineer Associate (November 2026) & DP-600: Fabric Analytics Engineer Associate (February 2027)  

**Focus areas:** Lakehouse architecture · Data pipelines · Real-time analytics (KQL) · PySpark transformations · Security and governance  

**Available for:** Freelance data engineering projects · Full-time Azure Data Engineer roles

📧 [cazacuirina@ymail.com]  
🔗 [(www.linkedin.com/in/irinacazacu/)]  
🔗 [(https://eirina86.github.io/)]

📍 London, UK

---

## Portfolio projects

Each project is built in a Microsoft Fabric trial workspace and documents both the technical implementation and the architectural decisions behind it.

| # | Project | Technologies | Status |
|---|---------|-------------|--------|
| 1 | [Lakehouse Foundation](#project-1--lakehouse-foundation) | Fabric · Delta Lake · PySpark · SQL endpoint | ✅ Complete (local environment) — Fabric implementation coming Week 5 |
| 2 | [End-to-End Data Pipeline](#project-2--end-to-end-data-pipeline) | Fabric Pipelines · Dataflow Gen2 · Power BI Direct Lake | ⏳ Planned |
| 3 | [Real-Time Analytics with KQL](#project-3--real-time-analytics-with-kql) | Eventstream · Eventhouse · KQL · Alerting | ⏳ Planned |
| 4 | [Security and Governance Framework](#project-4--security-and-governance-framework) | RLS · Sensitivity labels · OneLake access roles · Microsoft Purview | ⏳ Planned |

---

## Project 1 — Lakehouse Foundation

**Goal:** Demonstrate end-to-end data flow from raw files to structured Delta tables, served via the SQL analytics endpoint.

**What I built:**
- Ingested a public dataset (UK energy consumption data from BEIS) into a Fabric Lakehouse using a Copy Data pipeline
- Applied data quality checks and schema validation using PySpark notebooks
- Wrote the cleaned data as a managed Delta table with appropriate partitioning strategy
- Exposed the data through the SQL analytics endpoint and validated with T-SQL queries
- Implemented OPTIMIZE and VACUUM maintenance jobs for Delta table health

**Key decisions documented:**
- Why managed Delta tables over external tables for this use case
- Partitioning strategy selection based on query patterns
- V-Order vs standard Delta write — performance trade-offs

📁 [View project files](./projects/01-lakehouse-foundation/)

---

## Project 2 — End-to-End Data Pipeline

**Goal:** Build a production-style automated pipeline with incremental loading, transformation, and Power BI reporting.

📁 [View project files](./projects/02-pipeline-e2e/) *(coming soon)*

---

## Project 3 — Real-Time Analytics with KQL

**Goal:** Ingest streaming data via Eventstream into an Eventhouse, detect anomalies with KQL time-window queries, and configure alerting.

📁 [View project files](./projects/03-streaming-kql/) *(coming soon)*

---

## Project 4 — Security and Governance Framework

**Goal:** Implement a layered security model across a Fabric workspace, covering row-level security, sensitivity labels, and OneLake access roles.

📁 [View project files](./projects/04-security-rls/) *(coming soon)*

---

## Home lab

Alongside the Fabric work, I run a self-hosted Ubuntu server on a Dell Inspiron 3793, administered remotely over SSH. It's where I practise Linux administration and the infrastructure side of data engineering.

| # | Write-up | Skills | Status |
|---|---------|-------------|--------|
| 1 | [Recovering from a GRUB boot failure](./homelab/01-boot-failure-recovery/) | GRUB shell · UEFI / `efibootmgr` · Dell BIOS · SSH · Root-cause analysis | ✅ Resolved |

**Case study highlight:** after a routine update, the server stopped at the bootloader with every command failing. I mapped the disks from the GRUB shell, booted the kernel by hand, found a working boot path, ruled out file corruption with checksums over SSH from my phone, then made a preventive firmware change on-site. The write-up covers each step, what it ruled out, and where the root cause is still unproven.

---

## Technical skills

| Category | Technologies |
|----------|-------------|
| Platform | Microsoft Fabric, Azure Data Lake Storage Gen2, OneLake |
| Data engineering | PySpark, Delta Lake, T-SQL, KQL, Dataflow Gen2 |
| Orchestration | Fabric Data Pipelines, schedule triggers, notebook orchestration |
| Security | Row-Level Security, sensitivity labels, Microsoft Purview, OneLake access roles |
| Development tools | Azure Data Studio, VS Code, Jupyter, Git |
| Monitoring | Fabric Monitoring Hub, Capacity Metrics App, Query Insights |
| Linux & infrastructure | Ubuntu Server administration, SSH, GRUB / UEFI boot troubleshooting, disk and partition management |

---

## Study resources

I am working through a structured 22-week study plan that combines Microsoft Learn, hands-on Fabric trial work, and free community resources. Key resources I use:

- [Microsoft Learn — DP-700 study guide](https://learn.microsoft.com/en-us/credentials/certifications/resources/study-guides/dp-700)
- [Official Fabric labs (GitHub)](https://microsoftlearning.github.io/mslearn-fabric/)
- [MindMesh Academy — free DP-700 guide](https://mindmeshacademy.com)
- [KQL playground](https://aka.ms/kql)

---

*Last updated: October 2026*
