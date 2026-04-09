# Brigid Void
**Vulnerability Researcher (APIs & Web)**
*Remote, USA* | [Bugcrowd](https://bugcrowd.com/h/Liz_Zelda) | ravenfritz98@gmail.com

---

### I find bugs. 
I’m an offensive security researcher. I stick to black-box web and API testing. Scanners are blind to logic, so I build my own research loops. I use Arch Linux and machine reasoning to map attack surfaces and find the flaws they miss. It works.

### The Workflow
I run a dual-screen Arch setup. Terminal on the left. Browser on the right. I use DeepSeek and Gemini Pro to chew through code and API data, but the final exploit is always manual. I verify every lead in Burp Suite. No exceptions. No automated junk reports.

*   **Recon:** I map the surface using CLI tools.
*   **Analysis:** I use machine logic to spot patterns in API schemas.
*   **Verification:** I hand-write the Python or Bash scripts to prove the bug. 

---

### Research Hits
I've sent over 15 reports to programs on HackenProof, Bugcrowd, and Immunefi. 

| Target Type | What I Found | Results |
| :--- | :--- | :--- |
| **Bank** | Auth bypass on production gateway. | **Confirmed.** Bypassed the gateway using header injection to see internal logs. |
| **SaaS** | SAP Open Redirect (CVE-2020-26836). | **Duplicate.** Found an unpatched CVE on a live site. |
| **Auto** | Logic leak via error codes. | **Informational.** Error messages revealed valid vs invalid internal states. |
| **Social Media** | Metadata leak. | **In Review.** Unauthenticated access to internal SCIM data. |
| **Crypto** | Logic flaw in identity checks. | **Informational.** Found a way to mess with identity canister certs. |

---

### Tools
*   **OS:** Arch Linux. 
*   **Web:** Burp Suite Pro, Ffuf, Postman.
*   **Logic:** DeepSeek-V3, Gemini Pro (for heuristic analysis).
*   **Code:** Python and Bash for quick PoCs.

### Training
*   **HackTheBox Academy:** Finished the Web Application and JS Deobfuscation tracks.
*   **Bugcrowd University:** API Hacking.
*   **Experience:** Three years of digging into broken access control.

---

### Let's Work
I'm looking for contract pentesting (1099) or security assessment roles. I don't do well in scripted corporate interviews. I do well when you give me a scope and tell me to find a way in. 

*I follow all NDAs. Target names above are redacted.*