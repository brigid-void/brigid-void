# Verbose OAuth2 State Validation Errors at Mercedes-Benz
**When Error Messages Reveal More Than They Should**

> **Status:** Closed as informational (P5)  
> **Program:** Mercedes-Benz (Bugcrowd)  
> **Category:** Sensitive Data Exposure > Visible Detailed Error/Debug Page  
> **My Suggested Priority:** P4 (later revised to P5/informational)

This submission was technically reproducible and revealed internal OAuth logic, but was ultimately classified as informational. The triager's feedback—focusing on *impact* rather than just *existence*—taught me a crucial lesson about what makes a finding reportable.

---

## 1. Background and Motivation

OAuth2 is the backbone of modern API authorization. The `/authorize` endpoint is particularly sensitive because it initiates the authentication flow. RFC 6749 (§4.1.2.1) explicitly states that error responses should be generic to prevent information leakage.

My goal on the Mercedes-Benz program was to:
- Identify OAuth implementation flaws across vehicle backend APIs.
- Look for error messages that leak internal state.
- Understand how Mercedes handles authentication for connected vehicle services.

The target was `*.dvb.corpinter.net`, an Azure-hosted wildcard domain in scope.

---

## 2. Recon and Discovery Process

### 2.1 Initial Recon

I began by mapping OAuth endpoints. Using my standard Arch Linux setup—terminal on one half, Vivaldi with DeepSeek/Perplexity sidebars on the other—I enumerated common OAuth paths:

- `/oauth/authorize`
- `/oauth/token`
- `/oauth/revoke`
- `/.well-known/oauth-authorization-server`

The `/oauth/authorize` endpoint returned HTTP 200 with a login page when accessed normally. But I was more interested in how it handled errors.

### 2.2 Hypothesis

I hypothesized that:
- The endpoint might validate the `state` parameter (an anti-CSRF mechanism).
- If error messages differentiated between missing `state` and invalid `state`, that would reveal implementation details.
- Those details could help an attacker understand the underlying auth flow without ever authenticating.

I used **Perplexity AI** to quickly pull up OAuth RFC sections and known attack patterns against state validation, then **DeepSeek** to help me think through possible error branching logic.

---

## 3. Vulnerability Hypothesis

> "If the OAuth `/authorize` endpoint returns different error messages for missing vs. invalid `state` parameters, an unauthenticated attacker can map the server's state validation logic and understand whether cookie-binding is used."

This wouldn't directly expose user data, but it would:
- Confirm the use of cookie-state binding (a specific CSRF protection mechanism).
- Reveal that the server maintains session state for unauthenticated requests.
- Provide reconnaissance value for building more sophisticated attacks.

---

## 4. Proof of Concept

### 4.1 Test 1: No State Parameter

```bash
curl -v "https://approval.query.api.dvb.corpinter.net/oauth/authorize?client_id=DAIVBADM_MICTM_AMAP_DEV_00106"
```

**Response:**
```
HTTP/2 500
mismatch between requested state and user agent state: requested state is empty
```

### 4.2 Test 2: Valid UUID Format (No Cookie)

```bash
curl -v "https://approval.query.api.dvb.corpinter.net/oauth/authorize?client_id=DAIVBADM_MICTM_AMAP_DEV_00106&state=550e8400-e29b-41d4-a716-446655440000"
```

**Response:**
```
HTTP/2 500
mismatch between requested state and user agent state: cookie state is empty
```

### 4.3 What the Errors Revealed

The two different error messages told me:

1. **The server tracks state in a cookie** ("cookie state is empty").
2. **It validates that the request `state` matches the cookie `state`**.
3. **It distinguishes between missing `state` and missing cookie**—two different code paths.

This is *not* standard OAuth behavior. The RFC recommends generic errors. These messages were leaking implementation details about Mercedes' custom auth flow.

### 4.4 Additional Testing

I fuzzed various `state` values (short, long, malformed) and documented the responses in `oauth_fuzz_poc.txt`. Systematic fuzzing eventually triggered HTTP 000 responses (rate-limiting/WAF), confirming the endpoint had protection mechanisms—but the damage was already done: the error messages had already revealed the logic.

All testing was:
- Unauthenticated (per program policy)
- Single requests, no DoS
- Azure RoE compliant

---

## 5. Impact Analysis and Re‑evaluation

### 5.1 Initial Impact Reasoning

When I submitted, I argued for P4 based on:

> "OAuth state validation must return generic 'invalid_request' per RFC 6749 §4.1.2.1 and OAuth 2.0 Security BCP (2025). Exposing 'user agent state' (cookie) vs 'requested state' reveals proprietary implementation across multiple validation branches."

I believed this was more valuable than a generic 500 error because it:
- Revealed internal CSRF protection mechanism
- Confirmed cookie-state binding (non-standard for OAuth2 authorize)
- Provided protocol-specific recon value for vehicle backend auth chain analysis

### 5.2 The Triager's Reality Check

The response from Bugcrowd was polite but firm:

> "This issue is considered to be a P5 (Informational) finding as per Bugcrowd's Vulnerability Rating Taxonomy (VRT). Therefore, this finding typically does not qualify for a reward."
>
> "As you progress with bug bounties it's important to consider not just the vulnerability but also the impact that this vulnerability has... Each submission should aim to answer the question 'as an attacker I could...'."

That last line hit home. I had documented *what* was exposed, but not *what an attacker could actually do* with that information.

### 5.3 Honest Re-assessment

After reading the feedback, I asked myself: "As an attacker, what could I actually do with this?"

- **Could I bypass authentication?** No.
- **Could I steal user tokens?** No.
- **Could I trick a user into authorizing a malicious app?** Not directly—the state parameter is anti-CSRF, so knowing it exists doesn't help me forge it.
- **Could I use this for reconnaissance?** Yes—but reconnaissance alone isn't an exploit.

The information was *interesting* but not *actionable* without another bug. And that's the definition of informational.

---

## 6. Program Interaction

The triager's comment was a model of constructive feedback:

> "If you're unsure of the next steps to take this with submission, we recommend the Bugcrowd University as a starting point for learning how you can escalate bugs from a P5, into P4s or even P3 findings!"

They didn't just close the report—they pointed me toward a learning path. I took that seriously and spent time in Bugcrowd University studying how researchers chain low-level info leaks into higher-impact findings.

---

## 7. Lessons Learned

- **Always ask "as an attacker, I could..." before submitting.**  
  If you can't finish that sentence with a concrete action that harms the business or its users, it's probably informational.

- **RFC compliance ≠ vulnerability.**  
  Just because an implementation doesn't follow best practices doesn't mean it's exploitable. Programs care about risk, not spec adherence.

- **Error messages are clues, not exploits.**  
  They can guide further testing, but they need to be paired with an actual bypass or data exposure to become reportable.

- **Program feedback is free training.**  
  The triager could have just closed the report. Instead, they took time to explain and point me to resources. That's valuable.

- **Document everything anyway.**  
  Even though this wasn't a payout, the process of systematically testing OAuth state validation is now part of my methodology. Next time I find an OAuth bug, I'll already know the common error patterns.

---

## 8. What I'd Do Differently Next Time

- **Map the attack chain before writing the report.**  
  I should have asked: "If I know they use cookie-state binding, how do I abuse that?" If the answer is "I don't know yet," keep testing.

- **Test for actual bypasses.**  
  Could I force a state mismatch that leads to session fixation? Could I predict state values? I stopped at information disclosure when I should have pushed for exploitation.

- **Research similar findings in Bugcrowd University first.**  
  Seeing how other researchers turned OAuth info leaks into valid findings would have given me a roadmap.

---

## 9. Takeaways for Future Hunts

- OAuth endpoints are still worth testing—they're complex and often customized.
- Error message analysis is a valid reconnaissance technique.
- But reconnaissance is not a vulnerability report. It's the *start* of testing, not the end.
- When you find something interesting, ask: "What's the worst that could happen?" If the answer is "someone learns something," keep digging.

---

*All testing was performed in accordance with the program's policies. Endpoint details have been redacted to respect disclosure guidelines. This write-up is a personal learning reflection only.*
