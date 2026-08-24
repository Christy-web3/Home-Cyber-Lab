# Command reference

These commands are for the isolated Home Cyber Lab described in this repository. Confirm the target VM and prompt before running a command.

## Network checks

Show concise interface information:

```bash
ip -br addr
```

Test the Ubuntu-to-Wazuh path:

```bash
ping -c 4 192.168.10.30
```

Test the Kali-to-Ubuntu path:

```bash
ping -c 4 192.168.10.101
```

Test whether SSH is reachable from Kali:

```bash
nc -vz -w 5 192.168.10.101 22
```

Show the selected route from Kali:

```bash
ip route get 192.168.10.101
```

## Wazuh server health

Check the full alert pipeline:

```bash
sudo systemctl is-active wazuh-indexer wazuh-manager filebeat wazuh-dashboard
```

Check a registered agent:

```bash
sudo /var/ossec/bin/agent_control -i 001
```

Inspect internal manager components:

```bash
sudo /var/ossec/bin/wazuh-control status
```

Check root filesystem capacity:

```bash
df -h /
```

Locate high-level disk usage without crossing filesystems:

```bash
sudo du -xhd1 /var 2>/dev/null | sort -h
sudo du -xhd1 /var/ossec 2>/dev/null | sort -h
```

Do not delete Wazuh queue data until the exact subdirectory and its purpose have been verified.

## Ubuntu Wazuh agent

Check agent status:

```bash
sudo systemctl is-active wazuh-agent
```

Inspect recent agent logs:

```bash
sudo tail -n 30 /var/ossec/logs/ossec.log
```

Treat a log containing non-text bytes as text for searching:

```bash
sudo grep -ai "syscheck" /var/ossec/logs/ossec.log | tail -n 20
```

## SSH service checks

Start SSH temporarily on Ubuntu:

```bash
sudo systemctl start ssh
```

Confirm the listener:

```bash
sudo ss -tlnp | grep ':22'
```

Stop both the service and socket after testing:

```bash
sudo systemctl stop ssh.service ssh.socket
```

## Controlled SSH detection test

Run only against the owned Ubuntu lab VM.

Single nonexistent-user attempt from Kali:

```bash
ssh -o PreferredAuthentications=password \
  -o PubkeyAuthentication=no \
  -o NumberOfPasswordPrompts=1 \
  wazuh-test@192.168.10.101
```

The Wazuh dashboard should record Rule `5710`, Level `5`.

## Fail2Ban inspection

Show the SSH jail:

```bash
sudo fail2ban-client status sshd
```

Read the retry threshold:

```bash
sudo fail2ban-client get sshd maxretry
```

The lab temporarily changed this runtime value only to observe Wazuh's higher threshold. Restore it immediately after testing:

```bash
sudo fail2ban-client set sshd maxretry 5
```

Never weaken a production jail to generate test traffic.

## Firewall investigation

List ordered input rules and counters:

```bash
sudo iptables -L INPUT -n -v --line-numbers
```

Inspect the active nftables representation:

```bash
sudo nft list ruleset
```

Capture SSH packets on Ubuntu:

```bash
sudo tcpdump -ni enp0s3 tcp port 22
```

Packet capture revealed inbound SYN packets when firewall policy prevented the SSH handshake from completing.

## File Integrity Monitoring

Locate configured directories:

```bash
sudo grep -R "<directories" /var/ossec/etc 2>/dev/null
```

Check FIM scan activity:

```bash
sudo grep -ai "syscheck" /var/ossec/logs/ossec.log | tail -n 20
```

The default scheduled frequency observed in this lab is `43200` seconds. A dedicated real-time test directory will be configured in a future exercise.

## Shutdown

Linux VMs:

```bash
sudo shutdown now
```

Recommended order:

1. Kali
2. Ubuntu
3. Wazuh server
4. pfSense last using console option `6` (Halt system)

