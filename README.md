# 🛡️ WAF Defense Validation Lab — ModSecurity + Apache

> A hands-on defensive security lab demonstrating real-time attack detection and blocking using ModSecurity WAF. Attacks were first confirmed successful against DVWA and PrestaShop, then neutralized after custom WAF rules were deployed — all in a controlled virtual environment.

---

## 📌 Project Overview

| | |
|---|---|
| **Objective** | Deploy and validate a Web Application Firewall (WAF) to block real-world web attacks |
| **Attacks Tested** | SQL Injection (SQLi) · Stored XSS |
| **Targets** | DVWA · PrestaShop |
| **WAF Engine** | ModSecurity 2.9.5 on Apache/2.4.52 (Ubuntu) |
| **Scope** | Isolated virtual lab — non-production, educational use only |

---

## 🖥️ Lab Environment

| Component | Details |
|---|---|
| **Attacker** | Kali Linux — `192.168.56.102` |
| **Victim Server** | Ubuntu (Apache/2.4.52) — `192.168.56.110` |
| **Applications** | DVWA · PrestaShop |
| **WAF** | ModSecurity with OWASP CRS 3.3.2 |
| **Network** | Host-Only Adapter (isolated) |

---

## ⚔️ Phase 1 — Attack Confirmation (Pre-WAF)

Before deploying defenses, attacks were executed to confirm both applications were vulnerable.

### SQL Injection — DVWA

**Payload used:** `' OR '1'='1`

All database records returned — authentication logic bypassed completely.

![SQL Injection Attack Success](screenshots/01_DVWA_attack_success.png)

### Stored XSS — DVWA

**Payload used:** `<script>alert('XSS')</script>`

Script executed in the browser — XSS alert popup triggered and stored in the database.

![XSS Attack Success](screenshots/04_DVWA_XSS_attack_success.png)

---

## 🔧 Phase 2 — WAF Deployment (ModSecurity)

ModSecurity was enabled and configured in **blocking mode** with custom rules targeting each attack type.

### Enabling ModSecurity

```bash
sudo a2enmod security2
sudo systemctl restart apache2
sudo apachectl -M | grep security
# → security2_module (shared)   ✅ confirmed active
```

![ModSecurity Enabled](screenshots/10_ModSecurity_Enabled.png)

### Custom WAF Rules

**XSS Rule (ID: 1001)**
```apache
SecRule ARGS "<script>" \
"id:1001,\
phase:2,\
deny,\
status:403,\
log,\
msg:'XSS attempt detected and blocked'"
```

![XSS WAF Rule Config](screenshots/06_DVWA_XSS_Configured_WAF_rule_file.png)

**SQL Injection Rule (ID: 1002)**
```apache
SecRule ARGS "(?i:(union select|or 1=1|' or '|--|drop table|information_schema))" \
"id:1002,phase:2,deny,status:403,log,msg:'SQL Injection attempt detected and blocked'"
```

![SQLi WAF Rule Config](screenshots/03_DVWA_SQL_config_WAF_rule_file.png)

---

## ✅ Phase 3 — Defense Validation

With ModSecurity active and rules deployed, both attacks were re-tested.

### SQL Injection — Blocked ✅

Same payload now returns **HTTP 403 Forbidden** — blocked before reaching the backend.

![SQL Injection Blocked](screenshots/02_DVWA_SQL_injection_attack_block.png)

### XSS — Blocked ✅

Same XSS payload now returns **HTTP 403 Forbidden** — script tag intercepted in Phase 2.

![XSS Blocked](screenshots/05_DVWA_XSS_attack_block.png)

---

## 🌐 Phase 4 — Extended Testing (PrestaShop)

The same WAF rules were validated against PrestaShop using `curl` from the Kali attacker machine.

### XSS Attempt — Blocked ✅

```bash
curl -i "http://192.168.56.110/prestashop/?q=<script>alert(1)</script>"
# → HTTP/1.1 403 Forbidden
```

![PrestaShop XSS Blocked](screenshots/07_Prestashop_XSS_Attempt_Command_XSS_403_Response.png)

### SQL Injection Attempt — Blocked ✅

```bash
curl -i "http://192.168.56.110/prestashop/?id=1%20OR%201=1--"
# → HTTP/1.1 403 Forbidden
```

![PrestaShop SQLi Blocked](screenshots/08_Prestashop_SQLi_Attempt_Command_and_SQLi_403_Response.png)

---

## 📋 Phase 5 — Audit Log Verification

ModSecurity audit logs confirmed rule execution for every blocked request.

| Field | XSS Block | SQLi Block |
|---|---|---|
| **Rule ID** | 1001 | 1002 |
| **Phase** | 2 (request body) | 2 (request body) |
| **Action** | Intercepted | Intercepted |
| **Engine** | ENABLED | ENABLED |
| **Producer** | ModSecurity 2.9.5 / OWASP CRS 3.3.2 | ModSecurity 2.9.5 / OWASP CRS 3.3.2 |

![ModSecurity Audit Log](screenshots/09_ModSecurity_Audit_Log.png)
![SQL Defense Log](screenshots/SQL_defense_log.png)
![XSS Defense Log](screenshots/XSS_defense_Log.png)

---

## 📊 Results Summary

| Attack | Pre-WAF | Post-WAF | Rule | HTTP Status |
|---|---|---|---|---|
| SQL Injection (DVWA) | ✅ Successful | 🚫 Blocked | ID:1002 | 403 |
| Stored XSS (DVWA) | ✅ Successful | 🚫 Blocked | ID:1001 | 403 |
| XSS (PrestaShop) | ✅ Successful | 🚫 Blocked | ID:1001 | 403 |
| SQL Injection (PrestaShop) | ✅ Successful | 🚫 Blocked | ID:1002 | 403 |

---

## 🧰 Tools & Technologies

| Tool | Purpose |
|---|---|
| **ModSecurity 2.9.5** | Web Application Firewall engine |
| **Apache 2.4.52** | Web server hosting DVWA & PrestaShop |
| **OWASP CRS 3.3.2** | Core Rule Set baseline |
| **DVWA** | Vulnerable web application target |
| **PrestaShop** | Real-world CMS target |
| **Kali Linux** | Attacker machine |
| **curl** | HTTP request testing tool |

---

## 📁 Repository Structure

```
📦 waf-defense-validation-lab/
├── 📄 README.md
├── 📄 index.html
├── 📄 Defense_Validation_Report.docx
├── 📄 PrestaShop_Attack_Simulation_Report.docx
└── 📁 screenshots/
    ├── 01_DVWA_attack_success.png
    ├── 02_DVWA_SQL_injection_attack_block.png
    ├── 03_DVWA_SQL_config_WAF_rule_file.png
    ├── 04_DVWA_XSS_attack_success.png
    ├── 05_DVWA_XSS_attack_block.png
    ├── 06_DVWA_XSS_Configured_WAF_rule_file.png
    ├── 07_Prestashop_XSS_Attempt_Command_XSS_403_Response.png
    ├── 08_Prestashop_SQLi_Attempt_Command_and_SQLi_403_Response.png
    ├── 09_ModSecurity_Audit_Log.png
    ├── 10_ModSecurity_Enabled.png
    ├── SQL_defense_log.png
    └── XSS_defense_Log.png
```

---

## ⚠️ Disclaimer

> All activities were conducted in an **isolated, non-production virtual lab** using intentionally vulnerable systems. This project is strictly for **educational and skill-development purposes**. Unauthorized use of these techniques against real systems is illegal and unethical.

---

## 👤 Skills Demonstrated

- Web Application Firewall (WAF) deployment and configuration
- ModSecurity custom rule writing (Phase 2 blocking)
- Attack simulation and defense validation methodology
- Audit log analysis and incident evidence documentation
- Apache module management on Ubuntu Server

---

*⭐ If this project was helpful, consider starring the repo!*
