# Unauthenticated Access to AML/KYC Risk API at AMLBot
**A Technically Valid Finding That Taught Me About Scope and Business Context**

> **Status:** Closed as out of scope  
> **Program:** AMLBot (HackenProof)  
> **Category:** Access Control Issues  
> **Severity:** None assigned (out of scope)

This was my most technically thorough finding—and it was closed as out of scope. The API was wide open, exposing sensitive AML/KYC data, no authentication required, no rate limiting, and full documentation publicly available. But the domain I tested wasn't in scope. This taught me that even the best technical finding is meaningless if you haven't verified the scope first.

---

## 1. Background and Motivation

AMLBot provides cryptocurrency compliance tools, including KYC checks and anti-money laundering screening. Their bug bounty program on HackenProof focused on their main web application at `kyc-bounty.amlbot.com`.

My hypothesis was:
- The main app likely relies on backend APIs for risk calculations.
- Those APIs might be less protected than the frontend.
- If I could find the API endpoints, I could test for direct access bypassing the web interface.

This is a common pattern: the shiny frontend is locked down, but the internal APIs are exposed because "they're only used by our app."

---

## 2. Recon and Discovery Process

### 2.1 Finding the API

I started by browsing the main application at `kyc-bounty.amlbot.com` with Burp Suite running. In the network traffic, I noticed requests to `api-riskcalc.amlbot.com`. Bingo.

I immediately checked if that domain was in scope. The program policy listed `*.amlbot.com`—so I assumed subdomains were included. (Spoiler: this assumption would later bite me.)

### 2.2 Initial Probing

I ran basic discovery against the API subdomain:

```bash
# Check if it responds
curl https://api-riskcalc.amlbot.com/health

# Look for documentation (common pattern)
curl https://api-riskcalc.amlbot.com/openapi.json
```

Both returned data. The `/health` endpoint revealed the internal stack:
- JanusGraph (graph database)
- RabbitMQ (message broker)
- Redis (cache)
- Celery (task queue)

The `/openapi.json` was a complete OpenAPI specification documenting every endpoint, including:
- `/asyncprocess` – submit an address for risk analysis
- `/status/{request_id}` – retrieve results
- Authentication requirements: **none listed**

### 2.3 Using AI for Analysis

I fed the OpenAPI spec into **DeepSeek** to analyze the data flows and identify the most sensitive endpoints. DeepSeek helped me understand that:
- The risk scores included category breakdowns (WLT, DEFI, THEFT)
- The API returned USD equivalents for associated funds
- Blacklist status was exposed
- No authentication was documented or enforced

**Perplexity AI** helped me research AML/KYC data sensitivity regulations—GDPR, financial compliance requirements—to understand the business impact.

---

## 3. Vulnerability Hypothesis

> "If the risk calculation API is completely unauthenticated, anyone can query any blockchain address and receive detailed AML/KYC data, including blacklist status and risk scores. This exposes the exact same data the paid KYC service provides, without payment or authentication."

This wasn't just information disclosure—it was a complete bypass of the business model. The main app exists to sell KYC checks. The API was giving them away for free.

---

## 4. Proof of Concept

### 4.1 Direct API Access

**Step 1: Submit an address for analysis**

```bash
curl -X POST https://api-riskcalc.amlbot.com/asyncprocess \
  -H "Content-Type: application/json" \
  -d '{"address": "0xde0b295669a9fd93d5f28d9ec85e40f4cb697bae", "degree": 1, "hop": 1, "type": "both"}'
```

**Response:**
```json
{"request_id":"a497c0f0-f09b-4655-ad52-1d28c1de6b92"}
```

**Step 2: Retrieve the results**

```bash
curl https://api-riskcalc.amlbot.com/status/a497c0f0-f09b-4655-ad52-1d28c1de6b92
```

**Response included:**
- Risk score (numeric)
- Category breakdown (e.g., "WLT": 0.5, "DEFI": 0.3, "THEFT": 0.0)
- USD equivalent values
- Blacklist status (true/false)
- Transaction graph data

All without any API key, token, or authentication.

### 4.2 No Rate Limiting

To test for abuse potential, I ran a quick load test:

```bash
hey -n 20 -c 5 -m POST -H "Content-Type: application/json" \
  -d '{"address": "0x123"}' \
  https://api-riskcalc.amlbot.com/asyncprocess
```

All 20 requests returned HTTP 200. No throttling, no blocking. An attacker could scrape risk data for thousands of addresses per minute.

### 4.3 Health Endpoint Exposure

```bash
curl https://api-riskcalc.amlbot.com/health
```

This revealed the entire internal infrastructure stack—valuable for reconnaissance.

---

## 5. Impact Analysis

### 5.1 Business Impact

If this API were exploited:
- **Revenue loss:** Users could bypass paid KYC checks by calling the API directly.
- **Data exposure:** Anyone could determine if an address is blacklisted, enabling avoidance of compliance controls.
- **User profiling:** Risk scores and category data could be used to profile cryptocurrency users without consent.
- **Competitive intelligence:** Competitors could analyze AMLBot's risk scoring models.

### 5.2 Regulatory Impact

AML/KYC data is regulated under:
- **GDPR** (if it involves EU persons)
- **Financial services regulations** in multiple jurisdictions
- **Cryptocurrency compliance frameworks**

Exposing this data without authentication could create regulatory liability.

### 5.3 CVSS Score (if scoped)

I calculated CVSS 3.1 as:
- **AV:N/AC:L/PR:N/UI:N/S:C/C:H/I:N/A:N** = **8.6 (High)**
- Confidentiality impact: High (sensitive financial data exposed)
- Scope: Changed (API is separate from main app)
- No authentication required, no user interaction

---

## 6. Program Interaction

The response from HackenProof was professional but disappointing:

> "Your submission has been jointly analyzed by our triage specialists and the AMLBot security team. Unfortunately, theoretical issues and domain you stated fall outside the scope of our bug bounty program."

**Wait, outside scope?** The domain was `api-riskcalc.amlbot.com`—surely `*.amlbot.com` covers that?

I reviewed the scope again. The program explicitly listed:
- `kyc-bounty.amlbot.com`
- `app.amlbot.com`

It *did not* include `*.amlbot.com`. I had assumed wildcard scope, but the program used an explicit list. The API subdomain wasn't listed.

**This was entirely my fault.**

---

## 7. Lessons Learned

- **Verify scope before you test.**  
  Assumptions about wildcards will burn you. Read the policy. Check it again. Bookmark it.

- **"Out of scope" doesn't mean "not vulnerable."**  
  The API was still exposed. I just couldn't get paid for finding it. The program's scope is about their willingness to pay, not about whether the bug exists.

- **Documentation cuts both ways.**  
  The OpenAPI spec made my testing easy—but it also means any attacker could find the same endpoints. I should have checked if the spec was publicly indexed (it was).

- **Technical impact ≠ program impact.**  
  My CVSS score was 8.6, but the program didn't care because the asset wasn't in scope. Programs define their own risk boundaries.

- **Don't let scope frustration blind you to the win.**  
  Even without a payout, I:
  - Developed a methodology for finding hidden APIs
  - Learned to analyze OpenAPI specs for sensitive endpoints
  - Practiced load testing for rate limiting
  - Wrote a detailed, evidence-rich report

---

## 8. What I'd Do Differently Next Time

- **Create a scope checklist before testing.**  
  Before writing a single line of recon, I'll list all in-scope domains and check each target against it.

- **Test the main app's authentication first.**  
  If the main app requires a login, but the API doesn't, that's a red flag. I should have checked if the API respected the same auth.

- **Look for "hidden" scope boundaries.**  
  Some programs list domains but not subdomains. Others list wildcards. Never assume.

- **Report anyway (responsibly).**  
  Even though it was out of scope, I still reported it. That's the right thing to do. The program might quietly fix it, and my reputation stays clean.

---

## 9. Takeaways for Future Hunts

- API discovery is a superpower. Keep doing it.
- OpenAPI specs are treasure maps. Always check for `/openapi.json`, `/swagger.json`, `/docs`.
- Scope matters more than severity. A High finding out of scope pays $0.
- Build relationships, not just reports. The triager's note was kind—they didn't have to explain. I'll remember that.

---

*All testing was performed in accordance with the program's policies as I understood them at the time. The API endpoints have been redacted to respect the program's disclosure wishes. This write-up is a personal learning reflection only.*
