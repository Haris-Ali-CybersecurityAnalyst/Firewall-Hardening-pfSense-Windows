# Firewall Hardening — pfSense & Windows Firewall

Hands-on notes from configuring network- and host-based firewall rules to block insecure ports and reduce attack surface, plus a controlled DoS test used to validate response behavior.

## Part 1: Network-Level Firewall (pfSense)

### Objective
Block commonly abused insecure ports at the network boundary rather than relying solely on host-based controls.

### Ports Blocked (examples)
| Port | Service | Reason |
|------|---------|--------|
| 23 | Telnet | Unencrypted remote access |
| 21 | FTP | Unencrypted, credential exposure |
| 445 | SMB (external-facing) | Common ransomware/worm propagation vector (EternalBlue, WannaCry) |
| 3389 | RDP (external-facing) | High brute-force target unless behind VPN/MFA |

### Approach
1. Default-deny inbound rule set, explicit allow for required services only
2. Logging enabled on deny rules to capture attempted access for later review
3. Rules tested by attempting connection from an external test host — confirmed blocked and logged

## Part 2: Host-Level Firewall (Windows Firewall)

- Applied the same default-deny inbound philosophy at the host level as defense-in-depth
- Restricted inbound RDP to specific management subnet only
- Verified via `netsh advfirewall firewall show rule name=all` that no unexpected allow rules existed

## Part 3: Controlled DoS Test (hping3)

### Objective
Validate that firewall and monitoring correctly detect and respond to a volumetric traffic test — in an isolated lab environment only.

```bash
# SYN flood test against isolated lab target (never run against systems you don't own/control)
hping3 -S -p 80 --flood <lab-target-ip>
```

### What I observed
- Confirmed firewall connection-rate limiting triggered as expected
- Cross-checked resulting traffic spike against SIEM/log source to confirm visibility (ties into [Log Analysis Cheatsheet](../08-Log-Analysis-Cheatsheet))
- Documented baseline traffic vs. attack traffic for future detection tuning

## ⚠️ Safety Note
All testing here was performed against isolated lab infrastructure I own/control. Running these techniques against systems without explicit authorization is illegal.
