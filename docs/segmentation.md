# Segmentation Design

## Intent

Default deny between VLANs. Traffic is permitted only where a stated need exists, and each permitted flow is recorded here with the reason.

## VLAN 10 — Trusted endpoints
Workstations and personal devices. Permitted to initiate outbound to the internet and to VLAN 20 and 30. Nothing initiates inbound.

## VLAN 20 — IoT and untrusted devices
Consumer IoT hardware. These devices ship with firmware that is rarely patched, often phones home to vendor infrastructure, and cannot be assumed trustworthy. Permitted outbound to the internet on required ports only. Denied all inter-VLAN traffic.

Known exception flows (each needs a written reason):
- (none yet)

## VLAN 30 — Security lab
Isolated systems used for testing. Denied inbound from and outbound to other VLANs. Internet access restricted to package repositories.

## Rules to revisit
- Whether VLAN 10 needs to reach VLAN 20 at all, or whether device management can live on 30.
- Whether outbound DNS should be forced through the local resolver for every VLAN.
