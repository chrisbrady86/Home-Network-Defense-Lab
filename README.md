# Home Network Defense Lab

Configuration, detection rules, and analysis notes for a segmented home network built as a working defensive lab: pfSense for segmentation and firewall policy, Suricata for intrusion detection, and Splunk for log aggregation and alert triage.

Everything here runs on equipment I own, on a network I control.

---

## Why this exists

Reading about network defense and operating it are different skills. This lab exists so that the second one gets practiced: rules get written, alerts fire, most of them turn out to be nothing, and the work is figuring out which ones aren't.

The repository holds the artifacts — the rules, the searches, the design decisions and why they were made — rather than screenshots of a working setup.

## Architecture

```
                         WAN
                          │
                     ┌────┴────┐
                     │ pfSense │  firewall + inter-VLAN policy
                     └────┬────┘
                          │  (span / inline)
                     ┌────┴────┐
                     │Suricata │  IDS
                     └────┬────┘
                          │  eve.json
          ┌───────────────┼───────────────┐
          │               │               │
      VLAN 10         VLAN 20         VLAN 30
      trusted           IoT             lab
                          │
                     ┌────┴────┐
                     │ Splunk  │  log aggregation + dashboards
                     └─────────┘
```

## Segmentation model

| VLAN | Purpose | Outbound | Inter-VLAN |
|---|---|---|---|
| 10 | Trusted endpoints | Permitted | Initiates to 20 and 30 |
| 20 | IoT and untrusted devices | Restricted | Denied by default |
| 30 | Security lab | Restricted | Denied by default |

The design intent, and the reasoning behind each deny rule, is in `docs/segmentation.md`. The short version: VLAN 20 exists because consumer IoT devices are the least trustworthy things on a home network and should not be able to reach anything else on it.

## Repository layout

```
pfsense/     firewall policy and interface notes (sanitized)
suricata/    custom rules and tuning notes
splunk/      SPL searches and dashboard definitions
docs/        design decisions, detection validation, alert triage notes
```

## Detection validation

A rule that has never fired is a rule you do not know works. `docs/validation.md` records how each detection was exercised — what was generated, whether Suricata caught it, and whether it reached Splunk in a usable form. Rules that produced unusable alerts are documented too, along with what was changed.

## Wireless

The lab includes a wireless segment used for security testing against access points I own, isolated from the rest of the network. Scope and authorization notes are in `docs/wireless-scope.md`. Testing methodology and captures are deliberately not published here.

## Sanitization

Public IPs, internal hostnames, MAC addresses, API keys, and certificates are removed from everything in this repository. `.gitignore` covers the config export paths; anything committed by hand gets checked first. See `docs/sanitization.md`.

## Author

Christopher Brady — Cybersecurity Engineering, University of Alabama in Huntsville
[linkedin.com/in/christopher-j-brady](https://www.linkedin.com/in/christopher-j-brady/)
