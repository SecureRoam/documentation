# Snort

Snort is a free and open-source network intrusion detection system (IDS) that monitors network traffic in real-time and alerts you to potential security threats. It is widely used to detect and prevent attacks such as port scans, denial-of-service attacks, and buffer overflows.

From the [Snort website](https://www.snort.org/):

> Snort is the foremost Open Source Intrusion Prevention System (IPS) in the world. Snort IPS uses a series of rules that help define malicious network activity and uses those rules to find packets that match against them and generates alerts for users.

## Configuration

The main configuration file is located at `/etc/snort/snort.lua`.

Local configuration can be set in`/etc/snort/local.lua`. By default the rules are stored under, `$RULE_PATH/snort.rules`.

Home network can be set in `/etc/snort/homenet.lua`.

```lua  
HOME_NET = [[ 10.0.0.0/8 172.16.0.0/12 192.168.0.0/16 ]]
```
