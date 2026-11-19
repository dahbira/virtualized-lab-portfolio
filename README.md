# Virtualized Lab Portfolio — Windows Workstation & Linux Server

**IT Infrastructure Lab 01** · Built by Toheeb Adigun Gbolahan · ✅ Verified end to end

Build record of a fully functional two-VM environment in VirtualBox — static addressing on an isolated `192.168.1.0/24` segment, hardened with host firewalls and key-only SSH, and verified against every success criterion.

---

## Lab Topology

```
                        Host OS — VirtualBox 7.x
                    vNIC 192.168.1.1 (management)
                                │
                     vboxnet0 switch (host-only)
              192.168.1.0/24 · DHCP off · isolated
                ┌───────────────┴───────────────┐
                │                               │
          WIN-WS01                        LNX-SRV01
        192.168.1.114                   192.168.1.150
     Windows 11 Pro · RDP          Ubuntu 22.04 · SSH
     2 vCPU · 4 GB                  2 vCPU · 2 GB
```

- **2 VMs · 1 subnet** — host-only network, isolated from the LAN
- Each VM also carries a **NAT adapter for outbound updates only** — no default gateway on the host-only interface, so `192.168.1.0/24` stays local to the lab

---

## Task & Scope

> First assignment for a junior IT technician at a tech startup: stand up a fully functional virtualized lab with a Windows workstation and a Linux server on a secure network connecting them — and document the entire setup.

**Success criteria:**

- [x] Both VMs register on `192.168.1.0/24` with static addresses
- [x] Ping succeeds in both directions with 0% packet loss
- [x] SSH from Windows into Linux using key authentication
- [x] RDP from the host into Windows with NLA enforced
- [x] Unsolicited inbound traffic denied on both VMs

---

## Build Process

| # | Step | Detail |
|---|------|--------|
| 1 | **Install the hypervisor** | VirtualBox 7.x with Extension Pack; VT-x/AMD-V enabled in firmware |
| 2 | **Create both VMs** | WIN-WS01 (Windows 11 Pro) and LNX-SRV01 (Ubuntu Server 22.04 LTS), 2 VDI disks |
| 3 | **Wire the network** | Host-only `vboxnet0`, DHCP off, static `.114` / `.150`; NAT adapter kept for updates |
| 4 | **Harden both systems** | Firewalls deny inbound; RDP scoped to the subnet; SSH keys replace passwords (UFW + Defender) |
| 5 | **Verify and document** | Ping, SSH, RDP, and isolation tests recorded; baseline snapshots taken |

Baseline sized for a 16 GB host with both VMs running concurrently. Snapshots `WIN-WS01-BASE` and `LNX-SRV01-BASE` were taken after OS installation, before hardening.

---

## Network Configuration

**Linux — LNX-SRV01** `/etc/netplan/01-lab.yaml`

```yaml
# NIC1 · NAT — outbound, DHCP
enp0s3:
  dhcp4: true
# NIC2 · host-only lab link
enp0s8:
  addresses:
    - 192.168.1.150/24
```

```bash
sudo netplan apply
```

No default gateway on the host-only interface — the segment stays local to the lab.

**Windows — WIN-WS01** (PowerShell, elevated)

```powershell
New-NetIPAddress `
  -InterfaceAlias "Ethernet 2" `
  -IPAddress 192.168.1.114 `
  -PrefixLength 24

# allow ping from the lab subnet
New-NetFirewallRule -DisplayName "Lab ICMP" ...
```

---

## Security Hardening

Smallest working exposure — both host firewalls deny inbound by default.

**Windows — WIN-WS01**
- Defender Firewall on for all profiles — inbound blocked by default
- RDP `:3389` scoped to `192.168.1.0/24`, Network Level Authentication required
- Daily work under a standard user; separate admin account
- Automatic updates enabled

**Linux — LNX-SRV01**
- UFW default deny incoming — only TCP `:22` from `192.168.1.0/24`
- SSH key-only; password authentication and root login disabled
- sudo-only administration, no shared accounts
- `unattended-upgrades` installs security patches

Any new service starts closed until explicitly opened.

---

## Verification & Deliverables

Every success criterion was tested and recorded in the final verification run on the documented baseline:

- ✅ **Live demo** — boot both VMs; ping, SSH, and RDP all working
- ✅ **Network topology diagram** — see Lab Topology above
- ✅ **Configuration documentation** — specs, static addressing, firewall policy

**Next steps:** snapshot both VMs as rollback points → add a pfSense firewall VM between lab and NAT → centralize logs on LNX-SRV01 (first step toward a full SOC monitoring lab with Wazuh).

---

## Skills Demonstrated

`VirtualBox` `Windows 11` `Ubuntu Server 22.04` `Static IP addressing` `Netplan` `PowerShell` `UFW` `Windows Defender Firewall` `SSH key authentication` `RDP / NLA` `Network isolation` `Documentation`

---

📫 **Toheeb Adigun Gbolahan** — teegoldkbs97@gmail.com · [LinkedIn](https://www.linkedin.com/in/toheeb-adigun-a57864380) · Aspiring SOC Analyst
