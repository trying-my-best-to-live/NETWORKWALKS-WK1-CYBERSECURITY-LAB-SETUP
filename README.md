# 🔐 Cybersecurity Lab Environment Setup

**Building an isolated virtual lab for penetration testing and ethical hacking practice**

![Skill-Cybersecurity](https://img.shields.io/badge/Skill-Cybersecurity-404040?style=flat-square&labelColor=C00000)
![VirtualBox](https://img.shields.io/badge/Ver-VirtualBox%207.x-0070C0?style=flat-square&labelColor=000000)
![Kali](https://img.shields.io/badge/Kali%20Linux-v2026.x-E87500?style=flat-square&labelColor=000000&logo=kalilinux&logoColor=white)
![Network](https://img.shields.io/badge/Network-10.0.0.0%2F24-238F89?style=flat-square&labelColor=000000)
![Ethical Hacking](https://img.shields.io/badge/Ethical%20Hacking-E87500?style=flat-square&labelColor=000000&logo=kalilinux&logoColor=white)
![NetworkWalks](https://img.shields.io/badge/NetworkWalks-404040?style=flat-square&labelColor=C00000)

---

## 📌 Project Overview

This project covers **Week 1 of the Network Walks Cybersecurity Internship**:
a Wireshark SOC analysis block (Labs 3–8) followed by building a complete,
isolated penetration-testing laboratory on VirtualBox.

The lab runs three machines on a single private NAT Network so they can talk
to each other while still having outbound internet access — a realistic
attacker / target topology for authorized security testing.

---

## 🎯 Objectives

- Install and configure VirtualBox on a Windows host
- Create a private **NAT Network** (`10.0.0.0/24`)
- Import and configure **Kali Linux** as the attacker VM
- Build an **Android 9 (x86)** target VM
- Install and configure a **Windows 10** target VM
- Assign a consistent static IP to each VM
- Verify network connectivity, DNS resolution, and internet access
- Take a clean VM snapshot on every machine as a recovery baseline
- Document the full setup, including every problem hit along the way

---

## 🛡️ Purpose of the Lab

The lab provides an isolated, controlled environment for cybersecurity learning
and authorized security testing: network reconnaissance, port scanning,
vulnerability assessment, packet analysis, web security testing, and
security-tool experimentation.

⚠️ **Important:** This laboratory is used only on systems I own or have
explicit permission to test. It is never used against unauthorized systems.

---

## 🏗️ Lab Architecture

```
              Host (Windows, i5-4200U, 7.9 GB RAM)
                          │
            ┌─────────────┴─────────────┐
            │     VirtualBox NAT Network │
            │        10.0.0.0/24        │
            │   GW 10.0.0.1  DNS 8.8.8.8│
            └─┬──────────┬──────────┬───┘
              │          │          │
        10.0.0.2    10.0.0.9    10.0.0.10
         Kali         Android 9     Windows 10
       (attacker)    (target)      (target)
```

---

## ⚙️ Lab Configuration

| 🧩 Component | ⚙️ Configuration |
| --- | --- |
| 🖥️ Host OS | Windows 10 |
| 🧠 Host RAM | 7.9 GB |
| ⚡ Processor | Intel Core i5-4200U (2C/4T) |
| 🧰 Hypervisor | VirtualBox 7.x |
| 🐉 Attacker OS | Kali Linux 2026.x (1536 MB RAM) |
| 📱 Target OS 1 | Android 9 (x86) — Live CD (1536 MB RAM) |
| 🪟 Target OS 2 | Windows 10 22H2 (2048 MB RAM, 1 vCPU) |
| 🌐 Virtual Network | NAT Network |
| 📡 Network Address | 10.0.0.0/24 |
| 🐧 Kali IP | 10.0.0.2/24 |
| 📱 Android IP | 10.0.0.9/24 |
| 🪟 Windows IP | 10.0.0.10/24 |
| 🚪 Default Gateway | 10.0.0.1 |
| 🌍 DNS Server | 8.8.8.8 |

---

## 🪜 Lab Setup Procedure (summary)

### 1. Create the NAT Network
VirtualBox → Tools → Network → NAT Networks → create `NatNetwork`,
IPv4 prefix `10.0.0.0/24`, DHCP enabled, IPv6 disabled.

### 2. Kali Linux (attacker)
- Import the official Kali `.ova`/`.7z`; attach NIC1 to `NatNetwork`
- Static IP via NetworkManager: `10.0.0.2/24`, GW `10.0.0.1`, DNS `8.8.8.8`,
  `ipv4.dad-timeout 0`
- Shared folder: host `Downloads` → guest `/downloads` (automount)
- Snapshot: `WK1-PM1-Lab-Setup`

### 3. Android 9 (target)
- Boot the `android-x86_64-9.0-r2.iso` Live CD (interface is `wifi_eth`, no `eth0`)
- Configure at the Alt+F1 console:
  ```
  ifconfig wifi_eth 10.0.0.9 netmask 255.255.255.0 up
  ip route add default via 10.0.0.1 dev wifi_eth
  setprop net.dns1 8.8.8.8
  iptables -F; iptables -X; echo 0 > /proc/sys/net/ipv4/icmp_echo_ignore_all
  ip rule add from all lookup main pref 1
  ip neigh add 10.0.0.1 lladdr 52:54:00:12:35:00 dev wifi_eth
  iptables -t raw -F; iptables -t mangle -F; iptables -t nat -F
  iptables -P INPUT ACCEPT; iptables -P OUTPUT ACCEPT
  ```
- Snapshot: `WK1-PMx-Android-Lab-Setup-v2`

### 4. Windows 10 (target)
- Unattended install from `en-us_windows_10_22h2_x64.iso`
- Static IP via elevated netsh:
  ```
  netsh interface ipv4 set address name=Ethernet static 10.0.0.10 255.255.255.0 10.0.0.1
  netsh interface ipv4 set dns name=Ethernet static 8.8.8.8
  netsh advfirewall firewall add rule name=AllowICMPv4 protocol=icmpv4 dir=in action=allow
  ```
- Snapshot: `WK1-PMx-Win10-Lab-Setup`

---

## 🔎 Lab Verification

| ✅ Test | 🧾 Command | 🎯 Expected Result |
| --- | --- | --- |
| 🐧 Kali IP | `ip a show eth0` | `10.0.0.2/24` |
| 📱 Android IP | `ifconfig wifi_eth` | `10.0.0.9/24` |
| 🪟 Windows IP | `ipconfig` | `10.0.0.10/24`, GW `10.0.0.1` |
| 📡 Kali → Windows | `ping -c 4 10.0.0.10` | 0% packet loss |
| 📡 Kali → Android | `ping -c 4 10.0.0.9` | 0% packet loss |
| 📡 Windows → Kali | `ping 10.0.0.2` | 0% packet loss |
| 🌍 Internet (Windows) | `ping 8.8.8.8` | 0% packet loss |
| 🌍 Internet (Android) | `ping -c 4 8.8.8.8` | 0% packet loss |
| 🔄 Restore snapshot | boot from snapshot | baseline config restored |

Deliverable screenshots are saved as
`Lab 1 / Lab Android / Lab Kali to Android / Lab Windows / Lab Kali to Windows Result - Mazen Ashraf.png`.

---

## 🐞 Problems Encountered & Solutions

### Problem 1 — Windows installer hung (3×)
The install stalled at 30%, 2%, and 50%: guest CPU pegged at ~100% on one
core, **zero disk I/O**, UI frozen, log showing `AHCI#0: Port 0 reset` →
`VD#0: Cancelling all active requests` and `TM: Giving up catch-up attempt`.

**Root cause:** host RAM pressure (0.38 GB free of 7.9 GB) paged ~4 GB of VM
memory out; combined with an SMP spinlock on a 2-core host.

**Fix:** RAM 4096 → 2048, vCPUs 2 → 1, `paravirtprovider hyperv → none`,
SATA `hostiocache off → on`, fresh `Windows10-Lab-2.vdi` instead of the
half-installed one, plus freeing 4.1 GB by killing 40 orphaned MCP
`node.exe` processes. Write rate went from 0 to 500–900 MB/30s and the
install completed.

### Problem 2 — Kali → Android ping: 100% packet loss (one direction only)
Android could ping Kali, but Kali could not ping Android.

**Root cause:** Android's `iptables` **raw** table still had drop rules; my
first `iptables -t raw -F` had a typo and never ran.

**Fix:** flush the raw/mangle/nat tables explicitly:
`iptables -t raw -F; iptables -t mangle -F; iptables -t nat -F`.

### Problem 3 — Guest Additions unavailable on Windows
The unattended `VBOXPOST.CMD` could not find the GA ISO (none was attached
during install), so `guestcontrol` does not work.

**Workaround:** automate the headless guest with `controlvm keyboardputstring`
and `screenshotpng`. GA can be added later (power off → attach
`VBoxGuestAdditions.iso` → run `D:\VBoxWindowsAdditions.exe /S`).

### Problem 4 — Kali DAD timeout on a static IP
Manually set IPv4 sometimes failed DAD. Fixed with
`nmcli connection modify "Wired connection 1" ipv4.dad-timeout 0`.

---

## 💡 What I Learned

1. **NAT vs NAT Network** — a NAT Network lets multiple VMs on the same
   virtual network talk to each other while still getting outbound NAT. That's
   what makes a multi-machine lab possible.
2. **Virtual machine networking** — how VirtualBox adapters bind VMs to
   different network types, and how that choice affects inter-VM communication.
3. **Static IP configuration** — IPv4, subnet mask, gateway, and DNS on three
   different OS families (Linux/NetworkManager, Android/iptables, Windows/netsh).
4. **VM snapshots** — always take a clean snapshot *before* risky or
   experimental work; it's the known-good recovery point.
5. **Documentation** — commands, configuration, screenshots, problems, and
   solutions are part of a professional cybersecurity project, not an afterthought.
6. **Troubleshooting under resource pressure** — reading `VBox.log`
   (`AHCI#0: Port 0 reset`, `TM: catch-up lag`) and correlating with host
   memory/paging is what turned a "slow disk" guess into the real root cause.

---

## 🔐 Security & Ethical Use

This laboratory is intended strictly for educational purposes. It is used only
on systems I own or have explicit permission to test. It is never used to
attack unauthorized systems.

---

## 🔗 Tools & Resources

- **7-Zip:** https://7-zip.org/download.html
- **VirtualBox:** https://virtualbox.org/wiki/Downloads
- **Kali Linux:** https://kali.org/get-kali
- **Android-x86:** https://android-x86.org

---

## 👤 Author

**Mazen Ashraf**  
Cybersecurity Intern — Network Walks

LinkedIn: [your-linkedin-url]  
GitHub: [your-github-username]

---

## 📌 Project Information

**Program:** Network Walks Cybersecurity Internship · **Week:** 01 ·
**Module:** Cybersecurity & Pentesting Lab Setup · **Repository:** GitHub

© 2026 Mazen Ashraf. All rights reserved.
