---
title: "“Oops, All Vulnerabilities!” Hackthebox Starting Point Oopsie 2025 writeup"
datePublished: Fri Jul 04 2025 21:27:35 GMT+0000 (Coordinated Universal Time)
cuid: cmcpbsfth000102jxb78rguhn
slug: oops-all-vulnerabilities-hackthebox-starting-point-oopsie-2025-writeup
tags: hacking, bug-bounty, penetration-testing, web-security, hackthebox, reverse-shell, ctf-writeup, privilege-escalation, oopsie, vulnerability-disclosure, technopessimism, access-control-failure

---

## Oopsie: When Access Control is a Suggestion

Let’s set the scene: it’s a Friday in 2025, the world’s still spinning, and I’m jacked into the HackTheBox VPN, ready to see what fresh horrors await in the Oopsie lab. The name alone suggests someone, somewhere, made a mistake. Spoiler: they did.

## Recon: Nmap, the Old Reliable

First, the ritual sacrifice to the gods of enumeration:

```plaintext
bashnmap -sV -p- 10.10.10.28
```

Predictably, ports **22 (SSH)** and **80 (HTTP)** are open[1](https://github.com/alch-1/htb-oopsie-writeup)[4](https://bobmckay.com/i-t-support-networking/ethical-hacking/hack-the-box-walkthrough-oopsie/)[8](https://shapmanasick.gitbook.io/starting-point-htb/oopsie-walkthrough). The web server is running Apache, which is about as surprising as finding out your IoT toaster is mining crypto for someone in Belarus.

## Web Fuzzing: Cookie Monster’s Revenge

A quick trip to the web interface reveals a login page. Time to dust off **Burp Suite** and **wfuzz**. I intercept the login request and notice a cookie called `user`. Fuzzing that cookie value, I discover that swapping it to the admin’s ID—because why not—magically grants admin access[2](https://blog.cyberethical.me/htb-starting-point-oopsie)3[4](https://bobmckay.com/i-t-support-networking/ethical-hacking/hack-the-box-walkthrough-oopsie/). Access control: implemented via the honor system.

## File Upload: PHP Shells and the Art of War

Now wielding admin powers, I find a file upload function. I upload a **PHP reverse shell** (because if you’re not shelling, are you even hacking?) and trigger it. Netcat lights up with a shell as `www-data`3[4](https://bobmckay.com/i-t-support-networking/ethical-hacking/hack-the-box-walkthrough-oopsie/).

## Credential Harvesting: db.php, the Gift That Keeps Giving

A quick rummage through the web root coughs up a `db.php` file, conveniently storing **robert’s database password** in plaintext3[4](https://bobmckay.com/i-t-support-networking/ethical-hacking/hack-the-box-walkthrough-oopsie/). Because why hash when you can hope?

## Lateral Movement: Becoming Robert

Armed with new creds, I switch to user `robert`. It’s almost like they wanted me to.

## Privilege Escalation: SUID, CATastrophe

I enumerate SUID binaries and find `/usr/bin/bugtracker`. Running it as `robert` reveals it calls `cat` with elevated privileges, and the PATH isn’t sanitized. I hijack `cat` by dropping a malicious script in my path, then run bugtracker to read `/root/root.txt` and snag the root flag3[4](https://bobmckay.com/i-t-support-networking/ethical-hacking/hack-the-box-walkthrough-oopsie/).

## The Takeaway

Oopsie is a masterclass in how *not* to do access control, file uploads, or privilege management. If your web app trusts client-side cookies for authentication, you might as well hand out root shells as party favors. But hey, at least it’s educational—if only as a warning.

## Tags

* hackthebox
    
* oopsie
    
* bug bounty
    
* vulnerability disclosure
    
* privilege escalation
    
* web security
    
* reverse shell
    
* CTF writeup
    
* technopessimism
    
* access control failure
    
* hacking
    
* penetration testing
    

*The internet: it giveth, it taketh, and sometimes, it just needs to be burned down and rebuilt from the ashes of its own insecure PHP scripts.*

1. [https://github.com/alch-1/htb-oopsie-writeup](https://github.com/alch-1/htb-oopsie-writeup)
    
2. [https://blog.cyberethical.me/htb-starting-point-oopsie](https://blog.cyberethical.me/htb-starting-point-oopsie)
    
3. [https://www.youtube.com/watch?v=ShQUD1j10MA](https://www.youtube.com/watch?v=ShQUD1j10MA)
    
4. [https://bobmckay.com/i-t-support-networking/ethical-hacking/hack-the-box-walkthrough-oopsie/](https://bobmckay.com/i-t-support-networking/ethical-hacking/hack-the-box-walkthrough-oopsie/)
    
5. [https://help.hackthebox.com/en/articles/6007919-introduction-to-starting-point](https://help.hackthebox.com/en/articles/6007919-introduction-to-starting-point)
    
6. [https://github.com/WilsonHuha/cbr-doc/blob/master/posts/cc/Ssl\_post\_sort\_by\_time.md](https://github.com/WilsonHuha/cbr-doc/blob/master/posts/cc/Ssl_post_sort_by_time.md)
    
7. [https://www.youtube.com/watch?v=UWKkiWl8xaI](https://www.youtube.com/watch?v=UWKkiWl8xaI)
    
8. [https://shapmanasick.gitbook.io/starting-point-htb/oopsie-walkthrough](https://shapmanasick.gitbook.io/starting-point-htb/oopsie-walkthrough)
    
9. [https://www.hackthebox.com/machines/oopsie](https://www.hackthebox.com/machines/oopsie)
    
10. [https://www.youtube.com/watch?v=f2lC6xnDD1A](https://www.youtube.com/watch?v=f2lC6xnDD1A)