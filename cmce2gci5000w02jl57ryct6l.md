---
title: "Writeup: Manual Login and Flag Discovery on Starting Point - Crocodile - HTB Labs"
datePublished: Fri Jun 27 2025 00:20:46 GMT+0000 (Coordinated Universal Time)
cuid: cmce2gci5000w02jl57ryct6l
slug: writeup-manual-login-and-flag-discovery-on-starting-point-crocodile-htb-labs
tags: programming, pentesting, cybersecurity-1

---

**Introduction**

During a recent web penetration testing lab, I encountered a login page that proved resistant to automated brute-force attacks. Despite trying multiple tools and scripts, the flag remained elusive—until a manual approach led to success.

**Challenge Overview**

The target was a simple web application with a login form at `/login.php`. The lab provided a userlist (`allowed.userlist`) and a password list (`allowed.userlist.passwd`). Most automated tools failed because the login page always returned the same HTML, regardless of whether credentials were correct or not.

**Approach**

1. **Automated Attempts:**
    
    * I used Hydra and custom bash scripts to test all username/password combinations.
        
    * No visible difference in the HTML response made it impossible for tools to detect successful logins.
        
    * No redirects or error messages appeared in the server’s response.
        
2. **Manual Testing:**
    
    * I logged in manually using each username and password from the provided lists.
        
    * With `admin:rKXM59ESxesUFHAd`, the page changed, revealing the flag directly on the webpage.
        

**Key Takeaways**

* **Manual testing is sometimes necessary:** When automated tools fail due to lack of feedback or unique responses, manual login attempts can uncover hidden success conditions.
    
* **Persistence pays off:** Trying all combinations manually may seem tedious, but it’s effective when automation is stymied.
    
* **Documentation is crucial:** Keeping track of what you’ve tried and what you observe helps avoid repetition and leads to success.
    

**Conclusion**

This challenge reinforced the importance of flexibility in penetration testing. Not all vulnerabilities are exploitable with automated tools, and sometimes the best way forward is to try things manually and observe the results.

**Flag:**  
`c7110277ac44d78b6a9fff2232434d16`