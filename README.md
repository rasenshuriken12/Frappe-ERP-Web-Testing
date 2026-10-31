
# 🧪 ERPNext Web Application Testing 

## 📌 Project Overview
This repository contains the assets, documentation, and final report for the Continuous Assessment 3 (CA-3) Software Engineering project. 

The objective was to host an open-source web application locally, design a comprehensive test plan covering six specific areas, execute **17 test cases**, and document **4 defects** with reproducible evidence. 

- **Application Tested:** ERPNext v16.51.1 (Frappe Framework v16)
- **Deployment:** Docker (`frappe_docker` official compose stack)
- **Host Machine:** Ubuntu Linux (x86-64), Docker Engine 29.9.1
- **Report Location:** [`/docs/SE_CA3_Frappe_Web_App_Testing_Report.pdf`](./docs/SE_CA3_Frappe_Web_App_Testing_Report.pdf)
- **Screenshot Assets:** [`/assets`](./assets) (Figures S01 to S20)

---

## 🎯 Why ERPNext? (Scope & Justification)
While the assignment suggested OpenCart (PHP/MySQL), ERPNext was chosen because it is an enterprise-grade ERP with a much richer testing surface:
1. **Complex Forms:** A single "Item" carries mandatory fields, conditional logic, child tables, and numeric allowances.
2. **REST API:** Exposes documented `/api/resource` endpoints, allowing for genuine API testing (which a basic storefront lacks).
3. **Reproducibility:** Uses an official Docker Compose file (`pwd.yml`) that pins the exact version and spins up an 11-container stack (Python, MariaDB, Redis, workers).
4. **Workaround:** Since ERPNext lacks a shopping cart, the **Stock Module** (Item Creation, Stock Entry) was used to test the equivalent logic: mandatory fields, duplicate keys, and boundary values.

**Out of Scope:** Payment gateways, TLS hardening, penetration testing of the host, and modules like Payroll/Manufacturing.

---

## ⚙️ Hosting & Setup (How it was deployed)
Because ERPNext is a Python/MariaDB application, XAMPP cannot host it. Docker was used to deploy the stack locally at `http://localhost:8080`.

1. **Docker Permissions:** Encountered `permission denied` on `docker.sock`. Resolved by adding the user to the `docker` group (`sudo usermod -aG docker $USER`).
2. **Deployment:** Ran `docker compose -f pwd.yml up -d`. Eleven services started (backend, frontend, websocket, queues, scheduler, db, redis).
3. **Initialization:** The `create-site` container successfully ran and exited, creating the MariaDB database. 
4. **Access:** Signed in as `Administrator` and accessed the Desk interface.

---

## 📋 Test Plan & Execution

### Strategy
A black-box, risk-first approach was used. Core user flows were tested first, followed by boundary attacks, security checks, and performance measurement. 

### The 6 Required Testing Areas
| Area | Cases | Checks Executed |
| :--- | :--- | :--- |
| **Frontend (UI)** | 3 | Desk home render, required-field validation, list filtering. |
| **Functional** | 5 | Create item, duplicate-check, edit item, sign in, password reset. |
| **Boundary** | 1 | Negative quantity (`-5`) in a Stock Entry. |
| **Negative** | 2 | Missing mandatory field, malformed login input. |
| **API** | 2 | Ping endpoint, missing record returning 404. |
| **Security** | 2 | SQL injection at login, unauthenticated Desk access (OWASP Top 10). |
| **Performance** | 2 | Projects workspace load time, login page load time. |
| **Cross-Browser** | 1 | Item list compared in Chrome vs. Firefox. |
| **Mobile** | 1 | Layout tested at 375x667 (iPhone SE size). |

### Execution Results
**17 Test Cases** were designed and executed: **13 Passed** and **4 Failed**. 
*(Full steps, expected results, and actual results are documented in Section 4 of the PDF report).*

- **Passed Examples:** Duplicate Item Codes were correctly rejected inline. SQL Injection payloads (`' OR '1'='1`) were safely blocked with generic error messages. Mobile layout correctly collapsed into a hamburger menu.
- **Failed Examples:** Page load times exceeded 3 seconds (Major). Modal validation hid the field it was complaining about. Password reset confirmed success without sending an email. Error message pointed to the wrong field name.

---

## 🐛 Defect Log (Summary of Failures)

Four defects were found and logged. All remain **Open** (as fixing them requires modifying the open-source code).

| ID | Severity | Title & Description | Evidence |
| :--- | :--- | :--- | :--- |
| **D-01** | Minor | **Hidden Validation Error:** Submitting an Item without a Code triggers a modal ("Missing Values Required") that lists the field but *hides* it behind the modal. User is told a field is missing but shown no field. | Fig S06 |
| **D-02** | **Major** | **Severe Performance Lag:** On localhost with no throttling, the Projects page took **11.55s** to load (61 requests, 2.1MB). The login page took **5.78s**. This misses the 3-second target severely. DOMContentLoaded fires at 8.98s, meaning the delay is spread across sub-resources. | Fig S17, S18 |
| **D-03** | Minor | **False Positive on Password Reset:** Clicking "Forgot Password" displays a green success box ("Check your inbox"), but no email is sent because the server lacks SMTP configuration. A dead-end for non-admin users. | Fig S20 |
| **D-04** | Minor | **Misleading Text:** A duplicate-code error says "Select another **name**" on a field that is explicitly labeled "Item **Code**", sending the user to the wrong control. | Fig S07 |

---

## 🛠️ Tools Used
- **Docker Engine (29.9.1):** For hosting the ERPNext stack.
- **Chrome DevTools:** For Network performance timing and Mobile device emulation.
- **Mozilla Firefox:** For cross-browser layout comparisons.
- **Chrome Address Bar:** Used as a lightweight API client to view JSON responses directly for GET requests.

---

## 🚀 How to Replicate This Environment

If you wish to spin up this exact environment to verify the defects:

### 1. Prerequisites
Ensure you have Docker and Docker Compose installed on your machine (Linux recommended, 8GB+ RAM).

### 2. Clone and Start
```bash
git clone https://github.com/frappe/frappe_docker.git
cd frappe_docker
docker compose -f pwd.yml up -d
```
*Wait 3–5 minutes for the `create-site` container to build the database.*

### 3. Access
Go to `http://localhost:8080` in your browser.
- **Username:** `Administrator`
- **Password:** `admin`

### 4. Replicate the Tests
- Navigate to **Stock > Item** to recreate **D-01**, **D-04**, or **TC-02**.
- Open Chrome DevTools (`F12`) > **Network** tab > Refresh the page to recreate the performance issue (**D-02**).
- Log out and click **Forgot Password** to recreate **D-03**.

---
## ⚠️ Disclaimer: 

*All security payloads and tests were executed strictly against this localhost instance. No public or college servers were scanned or probed.*