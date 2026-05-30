# Ghost Hermes
**Vulnerability Researcher | APIs & Web Security**
Remote, USA | [Bugcrowd](https://bugcrowd.com/h/Liz_Zelda) | ravenfritz98@gmail.com

I hunt for the logic errors that automated scanners miss.

My background isn't in high-volume automation or "point-and-click" pentesting. I specialize in black-box web and API assessments where the vulnerabilities aren't obvious. To date, I’ve submitted over 15 vetted reports to programs on HackenProof, Bugcrowd, and Immunefi. Most were accepted; all were verified manually.

I work in the terminal. My environment is Arch Linux, usually with Burp Suite and a browser on one screen and Python/CLI tools on the other. I use LLMs to parse massive API schemas or deobfuscate dense JS chunks to find patterns, but I never trust their output. I verify every lead by hand.

---

### Selected Vulnerability Case Studies

These findings are redacted to honor NDAs, but they represent my typical focus areas:

*   **Financial Services:** Found an authentication bypass on a production gateway. By injecting specific headers, I pulled internal logs containing employee usernames and operational data. It was confirmed as an out-of-scope finding, but the client patched it immediately.
*   **SaaS Infrastructure:** Identified an unpatched SAP Open Redirect (CVE-2020-26836). It was eventually marked as a duplicate, but the discovery validated my methodology for testing enterprise-grade stacks.
*   **Automotive Tech:** Exploited inconsistent error codes to leak state data (valid vs. invalid accounts). This was triaged as Informational—a common result for "real but low-priority" logic flaws.
*   **Decentralized Finance (DeFi):** Investigated identity canister certificate anomalies. I spent 48 hours mapping the edge case, though the program ultimately dismissed the report. 

I don’t hunt for six-figure bounties. I hunt for the "this shouldn't be public" moments that compromise a company’s integrity.

---

### Technical Workflow

I build my own recon loops using `ffuf`, `gau`, and `katana`. When a target requires something unique, I don't wait for a vendor update. I write custom PoCs in Python or Bash to prove a vulnerability is exploitable and reproducible.

My approach to AI is pragmatic. I use it to generate boilerplate or scaffold complex tools, then I manually harden the code to ensure it meets forensic or professional standards. The result is always a defensible, high-signal report.

---

### Featured Project

**[adtech-forensics-engine](https://github.com/ghosthermes/adtech-forensics-engine)**  
*Playwright • Python • Forensic Verification*

I built this litigation-grade automation tool to support privacy compliance testing and ad-tech forensics. It was designed for a high-stakes legal environment where accuracy is the only metric that matters.

*   **Evidentiary Integrity:** Captures HAR files with hash verification and UTC-synchronized timestamps.
*   **Aggressive Probe Logic:** Triggers `blur` event listeners and other "stealth" tracking triggers used by modern ad-tech.
*   **Consent Mapping:** Compares OneTrust/Optanon initialization states against actual tracker firing times to catch "consent-less" tracking.
*   **Data Deobfuscation:** Automatically decodes Base64 and JSON payloads in cross-site sync data (Prebid/DFP).

I developed this engine in under 48 hours to meet a specific legal deadline. It demonstrates my ability to pivot into niche technical requirements, leverage AI to compress development cycles, and deliver an artifact that holds up under adversarial scrutiny.

---

### Training & Competencies

*   **HTB Academy:** Web Attacks, JS Deobfuscation.
*   **Bugcrowd University:** API Hacking.
*   **Focus:** Three years of testing authentication flows, broken access control, and API business logic.
*   **CTF:** Active participant in HTB 2025 and similar events.

---

### Professional Engagements

I am available for contract pentesting (1099), security assessments, and custom tool development. I don't excel in standard corporate panel interviews. I excel when given a scope and a deadline. 

If you need a custom test harness, a forensic capture engine, or a deep-dive assessment of a "secure" API, let's talk.

*All findings mentioned are redacted per NDA. The bugs were real.*
