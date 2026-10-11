# Compliance & Data Protection Regulations

> Compliance = demonstrating your organization follows rules about data handling, security risk, and operations. For tech companies, it's a **legal requirement**, an **enterprise sales gate**, and a **trust signal**. You typically layer multiple frameworks.
>
> Sources: [Formly — Compliance Guide 2026](https://formly.tools/blog/compliance-ai-soc2-iso27001-hipaa-gdpr-guide) | [GDPRLocal — Data Compliance Standards](https://gdprlocal.com/data-compliance-standards/)

---

## Why Compliance Matters for Tech Companies

1. **Legal requirement** — GDPR applies to any company processing EU residents' data (anywhere); HIPAA applies to US healthcare data
2. **Enterprise sales requirement** — 70%+ of enterprise procurement includes a security questionnaire; many require SOC 2/ISO 27001 before signing
3. **Trust signal** — compliance badges on pricing pages convert prospects

Fines for non-compliance: GDPR (€20M or 4% of global revenue), HIPAA ($1.9M per violation type), PCI DSS (fines + lost payment processing).

---

## The Major Frameworks (Quick Comparison)

| Framework | Scope | Who Needs It | Type | Certification? |
|-----------|-------|-------------|------|----------------|
| **GDPR** | EU personal data | Any company processing EU residents' data | Regulation | ❌ Legal compliance (no cert) |
| **CCPA/CPRA** | California consumer data | Companies with CA customers | Law | ❌ Legal compliance |
| **SOC 2** | Data protection, security, availability, confidentiality, privacy | SaaS, cloud providers, B2B tech | Audit standard | ✅ Type I / Type II |
| **ISO 27001** | Information Security Management System (ISMS) | International/multinational companies | International standard | ✅ Certified |
| **HIPAA** | US healthcare PHI | Healthtech, medical apps, healthcare vendors | US federal law | ❌ Legal compliance (Risk Assessments) |
| **PCI DSS** | Payment card data | E-commerce, any payment processing | Industry standard | ✅ Required for card acceptance |
| **NIST CSF** | Cybersecurity risk management | US federal, critical infrastructure | Framework | ❌ Voluntary |
| **CIS Controls** | Prioritized security best practices | Startups, any org without a security program | Best practices | ❌ Self-implemented |

---

## GDPR (European Data Privacy)

Applies to any organization collecting/processing/storing **personal data of EU/EEA residents**, regardless of where you're based.

### Core Principles
- **Lawfulness** — you need a legal basis for processing
- **Purpose limitation** — collect only for stated purposes
- **Data minimization** — collect only what you need
- **Accuracy** — keep data correct
- **Storage limitation** — don't keep data longer than needed
- **Integrity & confidentiality** — secure by design
- **Accountability** — document everything

### The 8 Data Subject Rights
1. Right to Access
2. Right to Rectification
3. Right to Erasure ("right to be forgotten")
4. Right to Restrict Processing
5. Right to Data Portability
6. Right to Object
7. Rights related to automated decision-making
8. Right to withdraw consent

### Practical Requirements
- GDPR-compliant privacy policy
- Documented legal basis for each processing activity
- **Record of Processing Activities (ROPA)**
- **Data Processing Agreements (DPAs)** with all vendors
- **72-hour breach notification** procedure
- **30-day response window** for data subject requests
- **Data Protection Officer (DPO)** if you meet thresholds

### Fines
- €10M or 2% of global revenue (procedural violations)
- €20M or 4% of global revenue (core principle violations)

---

## SOC 2 (US Enterprise Certificate)

Auditing standard by the AICPA, evaluating **five Trust Services Criteria** (TSC):
1. **Security** (minimum — everyone needs this)
2. Availability
3. Processing Integrity
4. Confidentiality
5. Privacy

### Type I vs Type II
- **Type I** — point-in-time audit (controls exist as of a date)
- **Type II** — 6-12 month monitoring period (controls operated effectively over time) — enterprise customers require this

### Timeline & Cost
- Gap assessment + remediation: 2-4 months
- Monitoring period: 6-12 months
- Audit: 2-4 months
- **Total: 10-18 months** for first Type II
- **Cost: $15K-$80K+**

### What Auditors Check
- MFA on all systems
- Patch management & vulnerability scanning
- Incident response procedures
- Encryption (at rest + in transit)
- Vendor access controls
- Background checks
- Change management
- Business continuity plans

**Who needs it:** SaaS companies, cloud providers, B2B tech selling to US enterprises

---

## ISO 27001 (Global Information Security)

International standard for an **Information Security Management System (ISMS)**. Recognized in 160+ countries.

### The 2022 Revision (93 controls, 4 themes)
| Theme | Covers |
|-------|--------|
| **Organizational** | Policies, roles, training, supplier relationships |
| **People** | Screening, training, dismissal, remote work security |
| **Physical** | Secure areas, equipment, workstations |
| **Technological** | Access control, encryption, malware, backups, network security |

### Certification Process
1. Implement the ISMS
2. Complete a risk assessment
3. Document a **Statement of Applicability (SoA)**
4. Pass a two-stage audit by an accredited certification body

**Who needs it:** European/international enterprises, government contractors, companies wanting global recognition. **~60-70% of controls overlap with SOC 2** — implementing one makes the other faster.

---

## HIPAA (US Healthcare)

Federal law protecting **Protected Health Information (PHI)**. Mandatory for healthcare providers, insurers, and their **Business Associates** (any vendor handling PHI).

### Three Rules
1. **Privacy Rule** — who can access PHI and why
2. **Security Rule** — administrative, physical, technical safeguards for ePHI
3. **Breach Notification Rule** — how to report compromises

### Key Requirements
- Appoint a **Privacy Officer** and **Security Officer**
- Sign **Business Associate Agreements (BAAs)** with all vendors
- Conduct annual **Security Risk Assessments**
- **Encrypt PHI**
- Implement **access controls**
- Train all staff annually
- Breach notification within 60 days

### Fines
- $100 to $50,000+ per violation, capped at $1.9M per violation category per year

---

## PCI DSS v4.0 (Payment Card Data)

Mandatory for any organization that processes, stores, or transmits **payment card data**.

### 12 Requirements
1. Install/patch firewall configuration
2. Change vendor defaults (no default passwords)
3. Protect stored cardholder data
4. Encrypt cardholder data in transit
5. Protect against malware
6. Develop/secure systems and applications
7. Restrict access by need-to-know
8. Identify/authenticate access to systems
9. Restrict physical access
10. Log/monitor all access
11. Test security systems
12. Information security policy

### v4.0 New Requirements
- **MFA for all access** to the cardholder data environment (not just admins)
- Enhanced phishing countermeasures
- Customized approach allows alternative methods (risk-assessed)

---

## Other Frameworks

| Framework | Description | Use Case |
|-----------|-------------|----------|
| **NIST CSF v2.0** | 6 functions: Govern, Identify, Protect, Detect, Respond, Recover | US critical infrastructure, federal agencies |
| **CIS Controls v8** | 18 prioritized security best practices (IG1, IG2, IG3) | Organizations with no formal security program; ideal starter |

---

## Compliance Roadmap: Where to Start

| Situation | Start Here | Timeline |
|-----------|-----------|----------|
| Zero formal security program | **CIS Controls IG1** (6 foundational controls) + Internal Compliance policies | 1-3 months |
| SaaS selling to US enterprises | **SOC 2** (Type II) | 10-18 months |
| EU customers (legal obligation) | **GDPR** compliance (data mapping → ROPA → DPAs) | 6-10 weeks |
| Health data (legal obligation) | **HIPAA** (appoint officers → risk assessment → BAAs) | 2-6 months |
| Global enterprise certification | **ISO 27001** | 6-18 months |

> **Key insight:** ISO 27001 and SOC 2 overlap ~60-70%. Implement ISO 27001 as your foundation, then SOC 2 becomes much faster and cheaper.

---

## Compliance Is Never "Done"

- **Ongoing monitoring** — track unauthorised access, policy violations, vulnerabilities
- **Regular audits** — internal annually, external per certification cycle
- **Incident management** — detect, contain, report, review
- **Standards evolution** — regulations change; stay updated
- **Cloud shared responsibility** — you're still accountable for data you store/process, even on managed infrastructure. Verify provider certifications (SOC 2, ISO 27001) and data residency.

> Compliance marks the **beginning**, not the end.

---

## References
- [GDPR Official Text](https://gdpr-info.eu/)
- [GDPRLocal Data Compliance Standards](https://gdprlocal.com/data-compliance-standards/)
- [Formly — Complete Compliance Guide](https://formly.tools/blog/compliance-ai-soc2-iso27001-hipaa-gdpr-guide)
- [SOC 2 Trust Services Criteria](https://www.aicpa-cima.com/resources/whitepaper/soc-2-compliance-handbook)
- [ISO 27001:2022 Standard](https://www.iso.org/standard/27001)
- [PCI DSS v4.0](https://www.pcisecuritystandards.org/document_library)
- [HIPAA Rules](https://www.hhs.gov/hipaa/for-professionals/laws/index.html)
