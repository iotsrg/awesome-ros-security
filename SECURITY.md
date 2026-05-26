# Security Policy

This is a documentation repository (a curated awesome-list); it does not ship executable code. Still, "documentation security" matters here:

## What we treat as a security issue

- **Misleading or wrong information** that could cause a reader to misdiagnose a real device. Example: a CVE attributed to the wrong vendor, an arXiv ID pointing to an unrelated paper. We handle these as P0.
- **A linked third-party resource that has been compromised** (typosquat, malicious mirror, dead domain reassigned). Report immediately.
- **Personal information about researchers** appearing without consent.

## How to report

Open an issue on this repository with the title prefix `[security]` , or for non-public concerns, email the maintainers via the contact on https://iotsrg.org/.

## Out of scope

Pull requests against entries you think are inaccurate. Use the normal PR flow (see [CONTRIBUTING.md](CONTRIBUTING.md)).

## Audit history

The README has been audited multiple times for fabricated content and wrong attributions; every fix is in the git log. Recent passes:

- Removed fabricated GitHub tool URLs (rosmap, secros, AlexandreBlanchard/DroneSploit, etc.).
- Removed hallucinated arXiv paper IDs (papers that pointed to unrelated work on chatbots, astronomy, etc.).
- Corrected CVE vendor attributions (CVE-2020-10268 was misattributed to NAO/Pepper, actually KUKA; CVE-2020-10281 was misattributed to ABB, actually MAVLink; etc.).
- Replaced unverified "Boston Dynamics Spot" and "Tesla Optimus" entries with verified Unitree CVEs.

If you find anything we missed, please open a `[security]` issue.
