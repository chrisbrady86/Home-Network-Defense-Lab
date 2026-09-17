# Build Log — Lab Host and Firewall

Record of how the lab host was built, including the problems hit along the way and how each was isolated. Written as the work happened rather than reconstructed afterward.

---

## Host

Repurposed desktop: Intel i7-10700K, 32 GB DDR4, MSI Z490 board (MS-7C79).

Three drives, assigned by role rather than by convenience:

| Drive | Role | Reasoning |
|---|---|---|
| 240 GB SATA SSD | Proxmox boot and system | Keeps the hypervisor off VM storage — a full VM datastore can't take the host down with it |
| 1 TB NVMe | VM disks (LVM-Thin) | Virtual disks are latency-sensitive and several VMs contend for I/O at once |
| 3 TB HDD | Backups, ISOs, templates (ext4 directory) | Sequential, not latency-sensitive; also the least trustworthy drive (see below) |

Proxmox VE 9.2 installed to the SATA SSD. Enterprise repositories disabled and `pve-no-subscription` enabled, since there's no subscription on this host.

## Drive health assessment

SMART was read on all three drives before assigning roles, rather than after.

The 3 TB HDD reported **218 reallocated sectors** across 237 reallocation events, with a normalized value of 97 against a threshold of 5. Current pending sectors, offline uncorrectable, and UDMA CRC errors were all zero, and power-on hours were 7,623.

Interpretation: sectors have failed and been remapped, but nothing is currently unreadable and the spare pool is barely touched. A single reading can't distinguish a drive that had one bad patch years ago from one in active decline — that requires trending the value over time.

**Decision:** use the drive only for data that is reproducible or is a second copy. Storage content types were explicitly restricted so VM disk images cannot land on it. Reallocation count to be re-checked monthly.

## Problems encountered

### NVMe not visible to the kernel

After the Proxmox install, the disk list showed only the SATA SSD and the HDD. The 1 TB NVMe was absent.

Isolation steps:
- `lsblk` — no `nvme0n1` device node
- `lspci | grep -i nvme` — no NVMe controller enumerated on the PCIe bus

That ruled out a driver problem and a partition-table problem. If the controller isn't enumerated by `lspci`, the OS is not being presented the device at all, which puts the fault at the firmware or physical layer. Windows had previously used this drive, so the hardware was known good.

**Resolution:** corrected in BIOS. *(TODO: record the exact setting that was changed — M.2 slot mode, lane allocation, or SATA port conflict. The MS-7C79 shares PCIe lanes between the M.2 slots and specific SATA ports.)*

After the change, `nvme0n1` enumerated normally and the LVM-Thin pool was created on it.

### pfSense ISO refused to boot under UEFI

The pfSense VM was configured with OVMF (UEFI) firmware and q35 machine type. On first boot the firmware reported:

```
failed to load Boot0002 "UEFI QEMU DVD-ROM" : Access Denied
No bootable option or device was found.
```

"Access Denied" on a device that is present and readable points at Secure Boot rejecting an unsigned bootloader, not at a missing or corrupt image. pfSense CE's loader is not signed with keys in the default OVMF key database.

**Resolution:** entered the VM's UEFI Boot Manager, disabled Attempt Secure Boot under Device Manager, saved and reset. The installer booted normally.

### Subnet collision caught during interface configuration

The pfSense installer proposed `192.168.1.1/24` with a DHCP pool of `192.168.1.100–199` for the LAN interface. The host network is already `192.168.1.0/24` — the same range the Proxmox management interface sits on.

Had this been accepted, pfSense would have had an interface on a subnet identical to its own WAN side, producing overlapping routes and unreachable hosts.

**Resolution:** LAN re-addressed to `10.10.10.1/24` with the DHCP pool moved to `10.10.10.100–199` to match. Worth noting that the installer offers these values as defaults and does not warn about the overlap; the check has to be done by the operator.

## Network design

The physical router does not support VLANs, so segmentation is implemented entirely inside the hypervisor.

Three Linux bridges on the Proxmox host:

- `vmbr0` — bound to the physical NIC. Carries host management traffic and pfSense's WAN side.
- `vmbr1` — no physical port attached. Internal segment.
- `vmbr2` — no physical port attached. Internal segment.

A bridge with no physical port attached exists only in software. Traffic on it cannot leave the host except through a VM that is attached to both it and another bridge. pfSense is that VM.

This produces genuine layer-2 isolation between segments without any physical switch involvement, and the resulting pfSense configuration is the same one that would be written against real hardware.

## Firewall VM

pfSense CE 2.9.0, 2 vCPU, 2 GB RAM, 32 GB virtual disk on the NVMe pool, ZFS on GPT.

Three VirtIO interfaces:

| Interface | Bridge | Role | Addressing |
|---|---|---|---|
| vtnet0 | vmbr0 | WAN | DHCP from the home router |
| vtnet1 | vmbr1 | LAN — first internal segment | 10.10.10.1/24, DHCP server enabled |
| vtnet2 | vmbr2 | Not yet assigned | — |

## State as of this entry

Complete:
- Proxmox host installed, repositories corrected, storage pools created and content-restricted
- SMART baseline recorded for all drives
- Three bridges created, two of them isolated
- pfSense installed with WAN and first LAN segment configured

Not yet done:
- vtnet2 assigned as a second internal segment (OPT1)
- Firewall rules between segments — currently only the default LAN-to-any allow rule exists
- Test hosts on each segment to verify isolation actually holds
- Suricata package installed on pfSense
- Splunk deployment and log forwarding
- Management plane hardening: non-root admin account, two-factor on the Proxmox web interface
