# Firewall Policy

Sanitized record of the rule set. Order matters — rules are evaluated top down and the first match wins.

## Interface: VLAN 10 (trusted)

| # | Action | Source | Destination | Port | Reason |
|---|---|---|---|---|---|
| 1 | Pass | VLAN10 net | any | any | General outbound |

## Interface: VLAN 20 (IoT)

| # | Action | Source | Destination | Port | Reason |
|---|---|---|---|---|---|
| 1 | Block | VLAN20 net | RFC1918 | any | Deny inter-VLAN by default |
| 2 | Pass | VLAN20 net | any | 443, 123 | Vendor cloud + NTP only |

## Interface: VLAN 30 (lab)

| # | Action | Source | Destination | Port | Reason |
|---|---|---|---|---|---|
| 1 | Block | VLAN30 net | RFC1918 | any | Full isolation |

## Change log
| Date | Change | Reason |
|---|---|---|
