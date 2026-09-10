# Security Policy

The Hybrid Production Standard (HPS-1.0) community and the maintainers at TrustNodeLogic take the security and integrity of cryptographic provenance, schema specifications, and reference implementations seriously.

---

## 1. Supported Versions

Security updates and errata are provided for the following specification and schema releases:

| Version | Status | Supported |
| :--- | :--- | :---: |
| **HPS-1.0.x** | Current Standard | :white_check_mark: Yes |
| **Draft / Experimental** | Pre-release / Staging | :x: No |

---

## 2. Reporting a Vulnerability

If you discover a security vulnerability, cryptographic weakness, or parsing defect in the HPS specification, schemas, or reference implementations:

**Please DO NOT report security vulnerabilities through public GitHub issues.**

Instead, report vulnerabilities via:
1. **GitHub Private Vulnerability Reporting**: Submit an advisory directly via the **Security** tab of this repository.
2. **Security Email**: Send an encrypted or secure email to:  
   `security@trustnodelogic.com`

### What to Include
To help us triage and resolve the issue quickly, please include:
- A description of the vulnerability and its potential impact.
- Affected components (e.g., `schema/hps-manifest-1.0.json`, container parsing rules, Ed25519 signature validation, or JSON-LD context).
- Minimal proof-of-concept (PoC) or reproducible test case.
- Any proposed remediation or mitigation steps.

### Response Commitment
- **Acknowledgment**: Within 48 hours of receipt.
- **Initial Assessment**: Within 5 business days, confirming validity and severity.
- **Coordinated Disclosure**: We ask researchers to adhere to a **90-day coordinated disclosure timeline** before public disclosure, allowing time for specification errata or reference patch deployment.

---

## 3. Scope

### In Scope
- **Cryptographic Schemes**: Faults in canonicalization (RFC 8785), Ed25519 signature verification rules, or SHA-256 PCM audio essence digest definitions.
- **Schema & Data Injection**: Deserialization bugs, JSON schema bypasses, or injection vectors in manifest parsing.
- **Container Embedding**: Buffer overrun risks, chunk misalignment, or malleability in RIFF WAV (`hps1`), MP3 (`TXXX`), or DDEX XML specifications.
- **Merkle Revision Audit Chain**: Flaws in append-only log verification or hash chaining.

### Out of Scope
- **User-Declared Truthfulness**: HPS-1.0 is a self-attestation framework. Users intentionally declaring inaccurate axis values (e.g., claiming `H` for `A`) is a self-attestation policy issue, not a technical software vulnerability.
- **Compromised Client OS / Host Environment**: Exploits requiring physical access, compromised operating system keystores, or root-level malware on the creator's machine.
- **Volumetric Denial of Service (DoS)** against public documentation servers or third-party Linked Data endpoints.

---

## 4. Responsible Research & Anti-Circumvention Policy

Security research conducted in accordance with this policy is considered authorized and lawful. 

However, the HPS specification is engineered to uphold copyright attribution, author rights, and regulatory transparency (including Regulation (EU) 2024/1689 Article 50). Therefore:
- Research into developing tools, scripts, or techniques intended to **strip acoustic watermarks, forge digital signatures, or circumvent machine-readable provenance markings** violates this policy and may violate applicable legal provisions, including the Anti-Circumvention rules of 17 U.S.C. § 1201 (DMCA) and the EU Copyright Directive.
- We request that proof-of-concept materials demonstrate technical validity without releasing public automated provenance-stripping utilities.

---

## 5. Security Errata & Attribution

When a reported vulnerability results in a specification clarification or patch:
- We issue a public Security Advisory with CVE assignment where applicable.
- Valid security disclosures are credited in the specification release notes and errata log (unless the researcher requests anonymity).
