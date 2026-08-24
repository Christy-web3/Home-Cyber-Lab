# Wazuh SSH detection and troubleshooting

**Date:** 24 August 2026  
**Scope:** Authorised Home Cyber Lab only

## Objectives

1. Verify the Wazuh alert pipeline after a restart.
2. Generate and investigate an SSH authentication failure.
3. Trigger a correlated brute-force alert.
4. Observe how Fail2Ban and Wazuh work together.
5. Confirm File Integrity Monitoring is enabled.

## Environment

| System | Address | Function |
| --- | --- | --- |
| pfSense | `192.168.10.1` | Gateway and firewall |
| Wazuh server | `192.168.10.30` | Central monitoring platform |
| Ubuntu.25 | `192.168.10.101` | Monitored endpoint, agent ID `001` |
| Kali Linux | `192.168.10.103` | Controlled test source |

## 1. Wazuh service recovery

After startup, the Wazuh manager failed with a systemd timeout. The indexer, Filebeat, and dashboard were active. A clean manager stop showed a remaining API process, so Wazuh's internal components were stopped before restarting the service.

The root filesystem had previously reached 100% utilisation. Disk analysis isolated abnormal growth in regenerable vulnerability-detection cache paths:

- `/var/ossec/queue/vd/feed`
- `/var/ossec/queue/vd_updater/tmp`

After controlled cleanup, root utilisation fell to approximately 22%. Vulnerability Detection was disabled temporarily because the VM has a small 25 GB virtual disk. All four Wazuh services subsequently reported `active`.

## 2. Agent and pipeline verification

The Ubuntu agent reported `active`, could reach `192.168.10.30`, and appeared as active in `agent_control`:

```text
Agent ID: 001
Agent name: Ubuntu.25
Status: Active
```

A safe synthetic SSH log first confirmed the path:

```text
Ubuntu journald → Wazuh agent → manager → Filebeat → indexer → dashboard
```

## 3. Real SSH authentication failure

Ubuntu's SSH service was started temporarily. Kali could ping Ubuntu but initially could not reach TCP port 22.

The investigation used:

- `ss` to confirm `sshd` was listening.
- `nc` to test the TCP port.
- `iptables` counters to inspect policy order.
- `tcpdump` to confirm SYN packets reached Ubuntu.

A temporary allow rule initially referenced `en0s3` instead of the correct `enp0s3`, so its counter remained at zero. Correcting the interface allowed the controlled SSH attempt.

Wazuh generated:

```text
Rule ID: 5710
Level: 5
Description: sshd: Attempt to login using a non-existent user
Source IP: 192.168.10.103
Source user: wazuh-test
```

## 4. Fail2Ban interaction

Repeated attempts stopped after five failures. Firewall inspection revealed an `f2b-sshd` chain ahead of the temporary allow rule. Fail2Ban had a `maxretry` value of `5` and correctly blocked the Kali address.

This demonstrated two layers of defence:

- **Fail2Ban:** rapid local blocking through firewall rules.
- **Wazuh:** centralised logging, decoding, alerting, and correlation.

For the isolated test only, Fail2Ban's runtime threshold was temporarily raised to `20`. Ten controlled attempts then reached SSH. The value was immediately restored to `5`.

## 5. Correlated brute-force alert

The installed Wazuh rule definition required eight Rule `5710` matches from the same source within 120 seconds:

```xml
<rule id="5712" level="10" frequency="8" timeframe="120" ignore="60">
  <if_matched_sid>5710</if_matched_sid>
  <same_source_ip />
</rule>
```

The successful result was:

```text
Rule ID: 5712
Level: 10
Frequency: 8
Description: sshd: brute force trying to get access to the system. Non existent user.
MITRE ATT&CK: T1110 — Brute Force
Tactic: Credential Access
Source IP: 192.168.10.103
Source user: brute-test
```

One consolidated Level 10 alert represented the correlated pattern.

## 6. File Integrity Monitoring preparation

The agent configuration monitors:

```text
/etc, /usr/bin, /usr/sbin, /bin, /sbin, /boot
```

Agent logs confirmed a successful FIM scan and a default frequency of `43200` seconds. A dedicated directory with real-time monitoring will be configured next.

## Cleanup completed

- Fail2Ban `maxretry` restored to `5`.
- Temporary Kali-to-Ubuntu SSH allow rule removed.
- `ssh.service` and `ssh.socket` stopped.
- Test file removed from `/etc`.
- Wazuh agent returned to `active` status.

## Lessons learned

- A running service is not necessarily reachable through the firewall.
- Packet counters and packet captures help locate the exact blocking layer.
- Rule order matters; Fail2Ban's chain acted before a later allow rule.
- SIEM correlation thresholds can interact with endpoint prevention thresholds.
- Security controls changed for testing must be restored immediately.
- Evidence should show the source, target, rule ID, level, timestamp, and MITRE mapping.

