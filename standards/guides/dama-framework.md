# Lightweight DAMA-Based Data Governance Framework

![Status](https://img.shields.io/badge/Status-Active-brightgreen)
![Scope](https://img.shields.io/badge/Scope-BI_Data-blue)
![Alignment](https://img.shields.io/badge/Alignment-DAMA--DMBOK-orange)

This repository contains a **lightweight adaptation of the DAMA-DMBOK framework** designed for a **small BI team** (including a single-person team).  
It provides governance structure, processes, and policies that are **practical, scalable, and aligned with industry standards**.

---

## 1. Strategy & Objectives
- **Mission:** Ensure BI reports and models are built on consistent, accurate, and trusted data.  
- **Scope:** Apply to all operational, HR, FSQA, and financial analytics data sources.  
- **Goals:**
  - Improve trust in BI reports.  
  - Standardize naming and definitions.  
  - Reduce manual rework through improved data quality.  
  - Stay compliant with security and privacy requirements.  

---

## 2. Roles & Responsibilities
Since the BI team is small, one person may cover multiple roles:
- **Data Owner:** Accountable for governed BI datasets.  
- **Data Steward:** Maintains data quality, metadata, and naming conventions.  
- **Data Custodian:** Manages access, refreshes, and infrastructure in Power BI/Fabric.  
- **Business Stakeholders:** Review and validate KPI definitions and report logic.  

*Optional: Create a lightweight “Data Council” with quarterly check-ins with Operations, HR, FSQA leads.*

---

## 3. Policies & Standards
- **Naming Conventions**
  - **DB/ETL:** `snake_case` (e.g., `plant_cd`)  
  - **Semantic Model:** Proper Case with spaces (e.g., `Plant Cd`)  
  - **Measures:** Clear business-friendly names (e.g., *Wages per LBS $*)  

- **Data Definitions**
  - Maintain a **business glossary** in Excel/SharePoint.  

- **Access & Security**
  - Use Power BI workspace roles (Viewer, Contributor, Admin).  
  - Apply **least privilege** principle.  

- **Retention**
  - Follow IT/legal standards.  
  - If not defined, adopt a “retain for 7 years” rule (adjust per policy).  

- **Data Quality**
  - Apply minimum checks: row counts, null checks, variance-to-source comparisons.  

---

## 4. Processes & Workflows
- **Data Intake:** Document owner, refresh frequency, quality expectations for new data.  
- **Change Management:** Keep a **changelog** in SharePoint/OneNote for schema/report updates.  
- **Data Quality Review:** Monthly validation against source systems.  
- **Issue Escalation:** Log issues, notify business owner, and document resolution.  

---

## 5. Tools & Technology
- **Core Tools:** Power BI, Fabric Dataflows (Bronze/Silver/Gold layers).  
- **Documentation:** SharePoint/Excel (glossary, changelog, data quality log).  
- **Metadata:** Power BI Data Catalog or descriptions in the model.  
- **Optional Automation:** Power Automate for scheduled checks/alerts.  

---

## 6. Metrics & Monitoring
- **Data Quality Scorecard:** % of sources passing quality checks.  
- **Glossary Coverage:** % of KPIs/columns with documented definitions.  
- **User Adoption:** Track Power BI report usage.  
- **Issue Log:** # of recurring vs resolved issues.  

---

## 7. Roadmap
- **Phase 1 (Now):** Implement core governance (standards, glossary, validation checks).  
- **Phase 2 (6–12 months):** Formalize quarterly data council meetings.  
- **Phase 3 (Future):** Add automation, MDM practices, and broader DAMA alignment.  

---

## 📌 Summary
This **lightweight DAMA-based framework** provides just enough structure to:  
- Ensure trust in BI reports  
- Keep data definitions consistent  
- Build credibility with leadership  
- Scale toward full DAMA coverage as the BI team grows
