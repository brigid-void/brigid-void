# Unauthenticated Access to SCIM Metadata Endpoints at SoundCloud  
**A Learning Journey in Information Disclosure vs. Actual Impact**

> **Status:** Closed as informational  
> **Program:** SoundCloud (Bugcrowd)  
> **Category:** Cloud Security – Misconfigured Services and APIs  
> **My Suggested Priority:** P4 (later revised)

This report was ultimately marked informational—the exposed metadata was expected behavior for SoundCloud’s SCIM implementation. However, the discovery process, my initial impact assessment, and the subsequent re-evaluation taught me more than any “valid” finding could. Here’s what I learned about SCIM, recon, and knowing when a finding is actually a finding.

---

## 1. Background and Motivation

SCIM (System for Cross‑domain Identity Management) is commonly used by enterprises to automate user lifecycle management (provisioning, de‑provisioning, group management). Misconfigurations here can be high impact because they sit close to identity and access control.

My recon goal on SoundCloud was:

- Map internal identity-related services and admin surfaces.
- Look for **low‑noise, unauthenticated** endpoints that might leak structure or configuration rather than obvious user data.
- Treat every “non‑consumer‑facing” hostname as a possible admin or integration surface.

This led me to look specifically for subdomains and paths mentioning `scim`, `identity`, `users`, `groups`, etc.

---

## 2. Recon and Discovery Process

I worked in a dedicated Arch Linux environment, with my usual split: terminal on one half, Vivaldi on the other with DeepSeek and Perplexity sidebars for rapid research.

### 2.1 Enumerating Identity‑Related Hosts

I used my usual mix of subdomain enumeration and fuzzing. Over time, this produced a large set of result files like:

- `fuzz_scim-1password.soundcloud.org.json`
- `fuzz_staging-scim-1pass.soundcloud.org.json`
- `subs_soundcloud.txt`
- `scim_subs.txt`

During this phase I:

- Focused on `.soundcloud.com` and `.soundcloud.org` infrastructure.
- Flagged anything with `scim` in the hostname as “high interest”.
- Logged potentially sensitive hosts into `high_value_live.txt` and `internal_subs.txt`.

This is where the host I eventually reported showed up (redacted here as `staging-scim-*.soundcloud.org`).

### 2.2 Path and Endpoint Enumeration

Once I had the SCIM‑related host, I switched to path discovery and protocol‑aware guesses:

- I generated SCIM‑specific wordlists and common paths into `scim_paths.txt`.
- I used **Perplexity AI** to quickly pull up SCIM RFC excerpts and known vendor endpoint patterns, then **DeepSeek** to analyze any JavaScript I found for SCIM-related strings—this helped me prioritize which paths to fuzz first.
- I brute‑forced for typical SCIM metadata and resource endpoints listed in the SCIM RFC and vendor docs:

  - `/Schemas`
  - `/ServiceProviderConfig`
  - `/ResourceTypes`
  - `/Users`
  - `/Groups`

This produced a focused list of candidate endpoints in `scim_api_endpoints.txt`.

---

## 3. Vulnerability Hypothesis

The initial hypothesis was:

> “If SCIM metadata endpoints are exposed without authentication, they may disclose enough internal configuration to aid further attacks (e.g., privilege escalation or targeted fuzzing of user/group operations).”

Concretely, I expected to see:

- Detailed schema definitions for user and group objects (attribute names, types, mutability flags).
- Supported features and authentication schemes for the SCIM provider.
- A list of resource types and their canonical endpoints.

If those were exposed to unauthenticated clients on an internal‑looking hostname, they might cross the line from “benign metadata” to “useful internal mapping for an attacker.”

---

## 4. Proof of Concept

*Note: All commands below are against a redacted hostname and are shown for educational purposes only. I am not sharing raw responses, credentials, or any sensitive identifiers.*

### 4.1 Accessing Metadata Endpoints

From a non‑authenticated shell, I was able to retrieve SCIM metadata:

```bash
# Schemas
curl https://staging-scim-*.soundcloud.org/Schemas | jq '.'

# Service Provider configuration
curl https://staging-scim-*.soundcloud.org/ServiceProviderConfig | jq '.'

# Resource Types
curl https://staging-scim-*.soundcloud.org/ResourceTypes | jq '.'
```
The responses were:

- HTTP 200 OK
- JSON SCIM objects consistent with the specification
- No authentication header required

For documentation and later review, I saved the responses locally (not shared publicly to respect the program’s disclosure policy).

### 4.2 What the Metadata Contained (at a High Level)

Without reproducing any proprietary fields, the metadata effectively revealed:

- **Schemas** – definitions for `User` and `Group` resources:
  - Core attributes (e.g., IDs, names, emails, group membership)
  - Attribute metadata (types, mutability, multi‑valued flags, required/optional)
- **ServiceProviderConfig** – feature toggles and capabilities:
  - Whether `PATCH`, `filter`, and `sort` are supported
  - Documentation of supported authentication schemes (e.g., bearer token)
- **ResourceTypes** – mappings of resource types to endpoints and schemas:
  - The resource type names (`User`, `Group`, etc.)
  - Endpoint patterns for each resource type
  - Which schemas each resource type uses

At this point, I believed this was an information disclosure affecting an internal identity surface: it gave an attacker a map of how SoundCloud’s SCIM service is structured, which attributes exist, and how to talk to it.

---

## 5. Impact Analysis and Re‑evaluation

### 5.1 Initial Impact Reasoning

Initially, I argued that:

- The metadata could be used to:
  - Tailor fuzzing and injection attempts against the SCIM API.
  - Understand how groups and users are modeled internally.
  - Discover which authentication flows are expected.
- While no user data was directly exposed, the leak violated least‑privilege principles and could be chained with other bugs (e.g., an auth bypass on `/Users` or `/Groups`).

This led me to submit the issue under:

- **Category:** Insecure API Endpoints / Misconfigured Services and APIs
- **Severity proposal:** P4 (information disclosure with potential to be used in chaining).

### 5.2 Follow‑Up Testing and Realization

After submitting, I took a step back and re‑validated the entire attack surface, especially:

- `/Users` and `/Groups` endpoints.
- Any non‑metadata endpoints revealed by `ResourceTypes`.

**Key observations from my re‑test:**

- All data‑bearing endpoints I tried required valid authentication.
- There was no way for an unauthenticated user to list, create, update, or delete users or groups.
- The behavior of the metadata endpoints was consistent with many SCIM implementations, where these are intentionally exposed as discovery endpoints.

**The critical test was this:** if metadata exposure alone constituted a vulnerability, then every SCIM implementation following RFC standards would be vulnerable. Since that’s clearly not the case, the bar must be higher—exposure of operational data (users, groups, attributes) or authentication bypass. Neither was present here.

At this point, I concluded:

- The behavior is **by design** for this deployment.
- The metadata did not cross into clearly sensitive territory for this program’s expectations.
- Treating this as anything beyond “low‑value informational” would be overstating the risk.

I updated the Bugcrowd submission accordingly, explicitly asking the triager to close it or mark it informational and noting that I’m new and misjudged the impact.

---

## 6. Program Interaction

The triager responded asking:

> “Can you please let us know what sensitive information are you able to observe on affected endpoints?”

That question was a useful reality check: if I can’t point to specific sensitive fields (credentials, secrets, non‑public identifiers, internal IPs, etc.) and only to “structure,” then what I have is closer to expected protocol metadata than to a reportable leak.

My final stance to the program was:

- Acknowledge that the metadata appears intentional.
- Clarify that no user data or privileged operations were exposed.
- Accept an informational/closed resolution and treat this purely as a learning opportunity.

---

## 7. Lessons Learned

- **Understand the protocol’s normal behavior first.**  
  SCIM, OpenAPI, and similar systems often expose metadata endpoints by design. Before reporting, check vendor docs and “normal” deployments.

- **Information disclosure needs a clear “why this matters.”**  
  Being able to say “this specific field or value is sensitive in this business context” is critical. Generic structure alone often isn’t enough.

- **Test the authenticated endpoints first.**  
  Before assuming metadata is sensitive, try to access the actual resources it describes. If they’re protected, the metadata is likely just documentation.

- **Re‑validate your own findings before submitting.**  
  A second pass on `/Users`, `/Groups`, and auth behavior would have made it obvious that the impact was minimal.

- **Be willing to downgrade or withdraw your own report.**  
  Owning the mistake, apologizing, and asking for closure is far better than defending a weak impact argument. It protects your reputation with triagers.

- **Keep the recon pipeline; just tighten the filter.**  
  The SCIM host and endpoints were discovered through a solid, repeatable recon + fuzzing process. The process is good—the bar for “reportable” needs to be higher.

---

## 8. What I’d Do Differently Next Time

- **Add a “pre-report validation” step:** Before writing the submission, I’ll force myself to articulate the exact sensitive data exposed and why an attacker could use it.
- **Research the target’s known stack:** If SoundCloud uses a standard SCIM provider, I’d check that provider’s docs to see what’s normally public.
- **Ask myself:** “If I were the developer, would I consider this a bug?” Sometimes that perspective helps.

---

## 9. Takeaways for Future Hunts

For future bounties, I’ll:

- Continue targeting identity and admin surfaces (they’re high‑value).
- Combine metadata findings with a hard push on auth / access control before deciding to report.
- Only submit when I can articulate a concrete exploitation path or a clearly sensitive leak.

Even though this report was ultimately a non‑issue, it improved my intuition around where the “informational vs vuln” line is drawn in real bug bounty programs.

---

*Out of respect for SoundCloud’s disclosure policy, the actual endpoint names and response data remain private. This write‑up is a personal learning reflection only.*
```

---

