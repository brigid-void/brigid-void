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
