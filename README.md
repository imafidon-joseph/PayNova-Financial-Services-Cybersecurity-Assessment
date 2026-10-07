# PayNova-Financial-Services-Cybersecurity-Assessment
End-to-end risk assessment, vulnerability management and compliance roadmap for the simulated PayNova fintech laboratory.

	
Client (simulated)	PayNova Financial Services Ltd
Prepared by	Team Vanguard, Infoassure Limited (Cybersecurity Graduate Internship Programme Capstone)
Assessment window	17-26 September 2026
Report version / date	v1.0, 27-28 September 2026
Classification	Confidential, client only

Lab-only project. Every system, record and card number in this project belongs to an isolated Docker laboratory. No real systems were tested and no real financial data is involved.

Overview

PayNova is a simulated mid-sized fintech offering card payments, mobile wallets and micro-lending on a hybrid on-premises architecture. This project assesses its security posture across the network, Active Directory, web application, API and database layers, maps each confirmed finding to recognised control frameworks, and sets out a prioritised remediation roadmap.

The project is an integrated set of deliverables (see Deliverables), with the Cybersecurity Assessment Report as the primary technical narrative.

Environment under test
Segment	Subnet	Hosts
DMZ	172.20.0.0/24	paynova-portal (nginx), paynova-webapp (OWASP Juice Shop)
Payment	172.21.0.0/24	paynova-db (MySQL 8.0), paynova-cardscheme-api, paynova-creditbureau-api (Flask/Werkzeug)
Corporate	172.22.0.0/24	paynova-ad / PAYNOVA-DC (Samba AD)
Analyst toolbox	.99 on all three	paynova-toolbox, multi-homed by design and not treated as a segmentation finding

Perspective: testing started from the analyst toolbox (an authorised internal workstation) and also used namespace-pinned tests from inside application containers to model what an attacker could reach after compromising a segment.

Rules of engagement: lab only, no external systems, no real financial data, non-destructive testing, evidence collected for significant findings, and findings validated before being classed as confirmed.

Results at a glance

23 confirmed findings, plus 5 positive controls and 3 negative test results.

Severity	Count
Critical	4
High	6
Medium	9
Low	4
Critical findings

All four involve unauthorised access to, or insecure storage of, regulated payment or authentication data.

ID	Finding
WEB-SQLI-001	SQL injection leading to authentication bypass to an administrator account (customer web app)
PAYNOVA-API-001	Unauthenticated card data enumeration via GET /cards (Card Scheme API)
F-DB-001	Remote MySQL root@'%' account with excessive privileges
F-DB-002	Card PAN and CVV stored in the database (PCI DSS v4.0.1 Req. 3.3 violation)
Other notable High findings
PN-WEB-001: unauthenticated exposure of a customer backup file (PANs, CVVs, wallet PINs) via a directory listing
F-NET-002: unrestricted DMZ-to-Payment network reachability
PAYNOVA-INFO-001: debug configuration endpoint leaking environment details and the JWT signing secret
F-DB-003, F-DB-004, F-DB-005: cleartext customer PII, over-privileged application DB account, MD5 password hashing
Positive controls observed
SMB message signing enabled and required on the domain controller
DMZ-to-Corporate and Payment-to-Corporate segmentation blocked the tested AD services
LDAP requires authentication for directory operations (validated with ldapsearch, which revised an earlier suspected finding)
Anonymous SMB access correctly denied share content
The paynova_readonly database account is restricted to SELECT

The full register of IDs, severities and report sections is in §10.3 of the report.

Methodology and tooling

Six technical phases: network and infrastructure, OS and Active Directory/identity, web application, API, database, plus negative-test validation.

Tool	Use
Nmap 7.93	Host discovery, service/version enumeration, NSE scripts
Burp Suite	Request capture and Repeater for authorisation, injection and session testing
curl	Direct API endpoint testing
Docker CLI + BusyBox netcat	Container/network inspection and namespace-pinned segmentation tests
PowerShell	Wrapper for docker exec commands
Web browser (Edge)	Web-tier reconnaissance and XSS proof of concept
Compliance mapping

Each confirmed finding is mapped in §11 to:

ISO/IEC 27001:2022 Annex A controls
PCI DSS v4.0.1 requirements
Regulatory and guidance sources: CBN RBCF, NDPA 2023, GDPR / UK GDPR, NIST SP 800-41, OWASP ASVS and API Top 10, CIS Benchmarks
Remediation roadmap
Priority	Timeframe	Focus
P0 Emergency	0-7 days	Treat as in-flight incidents: take the exposed backup and the /cards and debug endpoints offline, remove remote root and rotate credentials, remove stored CVV values, rotate the JWT secret and invalidate tokens, patch the SQL injection, and coordinate breach response and notification through the Incident Response Plan
P1 Immediate	0-30 days	Restrict DMZ-to-Payment reachability, strengthen the AD password and lockout policy, add server-side object-level authorisation, enforce role checks, and address cleartext PII and over-privileged DB accounts
Later phases	See report	Remaining Medium/Low items and continuous-improvement areas (§12)

Open gaps noted in the report: Credit Bureau API testing was still pending in this iteration, and incident-response tabletop exercises and named on-call staffing remain outstanding.

Deliverables
Deliverable	Purpose
Cybersecurity Assessment Report	Primary technical findings across network, AD/IAM, web, API and database
Risk Register	Tracked risks with likelihood, impact, controls, residual scores and target dates
Privacy Impact Assessment	DPIA against NDPA 2023, GDPR and UK GDPR (15 privacy risks)
Incident Response Plan	Detect/contain/eradicate/recover/notify framework, using PN-WEB-001 as the worked example
Executive Briefing	13-slide non-technical briefing for the executive panel

Version numbers differ between sections of the source report, so confirm the final version of each before publishing (see Notes).

Suggested reading order
Audience	Path
Executive	Report Executive Summary, then the PIA summary, then IRP §1 and §3
Technical	Full report, then the Risk Register
Compliance / audit	Report compliance mapping, then the PIA in full, then the Risk Register by framework
Incident response	IRP in full, then the report's web (PN-WEB-001) and API findings
Repository layout


The report includes lab evidence. In a live engagement, apply the redaction practice in Appendix C before any distribution:

The report and Risk Register are living documents. The PIA and IRP are reviewed at least annually and after any material change or SEV-1/SEV-2 incident. A change in one document is applied to the related documents in the same change window.

Disclaimer

This assessment covers an isolated laboratory environment for educational and capstone purposes. Findings, evidence and techniques must not be applied to systems you do not own or have explicit written authorisation to test.
