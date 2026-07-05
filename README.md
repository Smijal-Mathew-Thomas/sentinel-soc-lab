# Microsoft Sentinel SOC Lab

A set of KQL detection rules, hunting queries, and notes I put together while learning Microsoft Sentinel. On a real SOC team this is the part that lives in version control: Sentinel itself runs up in Azure, but the detection logic sits in Git where you can read it, tune it, and push it back out.

I built this working through CyberPlatter's Sentinel course on YouTube. The queries and write-ups here are mine. The course gets credit for the walkthrough.

## Before you clone this

You can't deploy Sentinel straight from a repo. It's an Azure service, so the workspace gets set up in the portal (steps are further down), and everything here is the content you build inside that workspace once it exists. That split confused me at first, so I'm calling it out early.

## What's in here

The `detections/` queries are meant to run on a schedule and turn into alerts:

| File | What it catches |
|------|-----------------|
| `aad-bruteforce.kql` | 10+ failed Entra ID sign-ins from one IP in 10 minutes (T1110) |
| `aad-success-after-bruteforce.kql` | A successful login right after a wall of failures, i.e. a likely compromise (T1110 / T1078) |
| `aad-signin-unusual-country.kql` | Successful sign-in from a country you don't expect (T1078) |
| `aad-disabled-account-signin.kql` | Someone hammering disabled accounts (T1078) |
| `win-failed-logons.kql` | Windows 4625 failed-logon brute force (T1110) |

The `hunting/` queries are for digging around by hand rather than alerting:

| File | What it looks for |
|------|-------------------|
| `encoded-powershell.kql` | Base64 or otherwise obfuscated PowerShell |
| `privileged-group-changes.kql` | New members added to admin groups |

There's also `analytics-rules/aad-bruteforce.yaml`, which is the brute-force rule written as code in Sentinel's own rule schema. Same logic as the KQL file, just in the format you'd actually deploy through the API or a pipeline. And `watchlists/known-bad-ips.csv` is a small bad-IP list you can import and join against.

## What you need to actually run these

An Azure subscription (the free credit covers a lab), a Log Analytics workspace with Sentinel turned on, and at least one connector feeding it logs. The Entra ID connector gives you `SigninLogs`, which the `aad-*` queries need. The Windows Security Events connector gives you `SecurityEvent` for the `win-*` and hunting queries.

## Getting it running

Create a Log Analytics workspace in the portal and add the Microsoft Sentinel solution to it. Connect Microsoft Entra ID under Data connectors, and Windows Security Events too if you want the endpoint queries. Once logs are flowing, open Logs and paste in any file from `detections/` to test it. To make a query into a real alert, go to Analytics, create a scheduled query rule, and drop the KQL in with whatever frequency and threshold you want. The YAML file shows that same rule already written out.

No Azure yet? You can still read every query. They're plain KQL against documented tables, so they make sense on their own.

## Fair warning

The watchlist IPs are made-up samples, so swap in a real threat feed before you lean on them. The thresholds (10 failures, that sort of thing) are just starting points. Expect to tune them once you see your own traffic or you'll drown in false positives. And everything here is defensive. Nothing in this repo attacks anything.

## Still on my list

A Logic App playbook that auto-enriches an incident with IP reputation, a workbook that maps sign-in failures by country, and a few more detections across other MITRE tactics. Screenshots of live incidents too, once I've caught some good ones.

## License

MIT, see [LICENSE](LICENSE).
