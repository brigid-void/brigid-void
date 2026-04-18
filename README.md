# Brigid Void
**Vulnerability Researcher (APIs & Web)**
*Remote, USA* | [Bugcrowd](https://bugcrowd.com/h/Liz_Zelda) | ravenfritz98@gmail.com

---

I find bugs.

Not always the kind that pay out. Triage is what it is. But I've sent over 15 reports to programs on HackenProof, Bugcrowd, and Immunefi. Most got read. Some got fixed. I got good at spotting the things scanners ignore.

I stick to black‑box web and API testing. Arch Linux. Terminal on the left, browser on the right. I throw API schemas and JS chunks at DeepSeek or Gemini to surface patterns, but I verify everything by hand. Burp. DevTools. Python one‑liners. No automated junk.

---

### Things I've Found

I can't name names. NDAs. But here's the shape of it.

- **Banking API.** Auth bypass on a production gateway. Injected a header, got back internal logs with employee usernames and ops data. Confirmed. Out of scope. Still fixed it.
- **SaaS platform.** Unpatched SAP Open Redirect (CVE‑2020‑26836). Found it. Duplicate. Still felt good.
- **Auto manufacturer.** Error codes leaked valid vs. invalid state. Informational. That's triage for "real but we're not paying."
- **Social media.** SCIM metadata endpoint open to the world. Told them. They said P4.
- **Crypto.** Identity canister cert weirdness. Spent a weekend on it. Got marked spam. Whatever.

I don't have a $10k bounty story. I have a pile of "this shouldn't be public" moments that companies eventually closed. That's what matters.

---

### How I Work

Dual‑screen Arch. CLI for recon—ffuf, gau, katana, custom bash loops. Browser for poking. LLMs for pattern‑spotting in giant API dumps, never for final judgment.

I write PoCs in Python or Bash. Nothing fancy. Just enough to prove the thing works.

---

### Training & Time Sink

- HTB Academy: Web Attacks, JS Deobfuscation.
- Bugcrowd University: API Hacking.
- Three years of poking at auth flows and access control.
- Some CTF flags. HTB 2025. Nothing legendary.

---

### What I'm Looking For

Contract pentesting. 1099. Assessments. I'm not great at panel interviews. I'm good when you hand me a scope and tell me to find a way in.

*All target details above are redacted per NDA. The bugs were real.*