# Next-Generation-Oracle-ERP-Dashboard-Using-Oracle-APEX-and-REST-APIs
This project demonstrates a modern, low-code Oracle APEX application that acts as a unified ERP dashboard by integrating Oracle E-Business Suite (EBS R12) and Oracle Fusion Cloud using REST APIs.  The solution provides real-time KPIs, analytics, and monitoring capabilities for enterprise users.

# Next-Generation Oracle ERP Dashboard Using Oracle APEX and REST APIs

This project demonstrates a modern, low-code Oracle APEX application that
acts as a unified ERP dashboard by integrating Oracle E-Business Suite (EBS R12)
and Oracle Fusion Cloud using REST APIs.

The solution provides real-time KPIs, analytics, and monitoring capabilities
for enterprise users.

---

## 🚀 Features
- Oracle APEX executive dashboard
- Integration with Oracle EBS R12 using SQL & PL/SQL
- Integration with Oracle Fusion Cloud using REST APIs
- Real-time KPIs (AP, AR, GL)
- REST API logging and error handling
- JSON parsing and reporting
- Modular and reusable architecture

---

## 🛠️ Technologies Used
- Oracle APEX
- Oracle SQL / PL/SQL
- Oracle E-Business Suite R12
- Oracle Fusion Cloud REST APIs
- JSON_TABLE
- REST Web Services

---

## 🏗️ Architecture Overview

User → Oracle APEX UI  
Oracle APEX → PL/SQL Packages  
PL/SQL → EBS Tables (SQL Queries)  
PL/SQL → Fusion Cloud (REST APIs)  

---

## 📦 Modules 
1. Executive Dashboard (KPIs)
2. AP Invoice Status (EBS + Fusion)
3. AR Aging Report
4. GL Balance Summary
5. REST API Monitor
6. Error Logging Dashboard

---

## ▶️ Setup Instructions

1. Create tables using scripts from `/sql`
2. Configure Web Credentials in Oracle APEX
3. Compile PL/SQL packages
4. Import APEX application
5. Run KPI refresh procedure
6. Access dashboard

---

## 📸 Screenshots & Demo
Screenshots and demo video are available in the repository.

---

## 🎯 Oracle ACE Contribution
This project is created as part of the Oracle ACE Associate journey
to demonstrate real-world Oracle APEX and ERP integration use cases.

---

SQL SCRIPTS (READY)
sql/01_tables.sql
CREATE TABLE erp_kpi_summary (
  kpi_name VARCHAR2(100),
  kpi_value NUMBER,
  last_refresh DATE DEFAULT SYSDATE
);

sql/02_rest_log_table.sql
CREATE TABLE erp_rest_log (
  log_id NUMBER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
  api_name VARCHAR2(100),
  request_date DATE DEFAULT SYSDATE,
  response_clob CLOB,
  status VARCHAR2(30),
  error_msg VARCHAR2(4000)
);

sql/03_plsql_packages.sql
CREATE OR REPLACE PACKAGE fusion_rest_pkg AS
  FUNCTION get_ap_invoices RETURN CLOB;
END fusion_rest_pkg;
/
CREATE OR REPLACE PACKAGE BODY fusion_rest_pkg AS
  FUNCTION get_ap_invoices RETURN CLOB IS
    l_response CLOB;
  BEGIN
    l_response := apex_web_service.make_rest_request(
      p_url => 'https://<fusion-host>/fscmRestApi/resources/latest/invoices',
      p_http_method => 'GET',
      p_credential_static_id => 'FUSION_REST_CRED'
    );

    INSERT INTO erp_rest_log(api_name, response_clob, status)
    VALUES ('FUSION_AP_INVOICES', l_response, 'SUCCESS');

    RETURN l_response;
  EXCEPTION
    WHEN OTHERS THEN
      INSERT INTO erp_rest_log(api_name, status, error_msg)
      VALUES ('FUSION_AP_INVOICES', 'FAILED', SQLERRM);
      RETURN NULL;
  END;
END fusion_rest_pkg;
/

sql/04_kpi_refresh.sql
CREATE OR REPLACE PROCEDURE refresh_kpis IS
  v_cnt NUMBER;
BEGIN
  SELECT COUNT(*) INTO v_cnt
  FROM ap_invoices_all
  WHERE approval_status <> 'APPROVED';

  DELETE FROM erp_kpi_summary;

  INSERT INTO erp_kpi_summary
  VALUES ('Pending AP Invoices', v_cnt, SYSDATE);

  COMMIT;
END;
/

sql/05_ebs_queries.sql
-- AP Pending Invoices
SELECT invoice_num, invoice_amount, approval_status
FROM ap_invoices_all
WHERE approval_status <> 'APPROVED';

-- AR Aging
SELECT customer_id, SUM(amount_due_remaining)
FROM ar_payment_schedules_all
GROUP BY customer_id;

-- GL Balances
SELECT code_combination_id,
       SUM(accounted_dr - accounted_cr)
FROM gl_je_lines
GROUP BY code_combination_id;

📄 docs/architecture.md
This document describes the technical architecture of the Oracle APEX ERP Dashboard.

The system uses Oracle APEX as the presentation layer,
PL/SQL as the business logic layer,
Oracle EBS tables as the data source,
and Oracle Fusion REST APIs for cloud data integration.

## 🤝 Contributions
Feedback, suggestions, and collaboration are welcome.
