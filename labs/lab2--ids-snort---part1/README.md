# NIDS using SNORT - Part 1

#### Objectives

- Learn how to install and configure Snort.
- Explore Snort's modes and rule sets.
- Capture and analyze network traffic for intrusion detection.
- Write custom Snort rules for specific attack scenarios.

### Prerequisites

- Wireshark or tcpdump for network traffic analysis (optional)

#### Snort Installation & Configuration

Using your package manager install snort v2, for Debian based distributions (eg. ubuntu) use: `sudo apt install snort`, test if its correctly installed using `snort --version`

- Snort configuration is located at `/etc/snort/snort.config`
- Snort can work either online and offline
- Online => listening to incoming traffic of a net interface

```bash
sudo snort -A <out_mode> -i <if> -c <config_path> -l <out_path>
```

- Offline => analysis of pcaps

```bash
sudo snort -r <pcap_path> -A <out_mode> -i <if> -c <config_path> -l <out_path>
```

#### Practice with pcaps

- Close this repo https://github.com/AbdelliNasredine/IT-D
- Go to lab2 folder and local `scanning.dump` file
- Run snort (offline mode) to analyze the dump file

```bash
sudo snort -r <pcapfile> -c /etc/snort/snort.conf.all -l ./snort-output/phase-1/
```

#### Exercise

Generate you own attack traffic and try to detect intrusion attempts using snort

- steps:
  - use tools for execution of attacks/intrusion attempts
    - scanning (`nmap`)
    - exploitation (`metasploit`)
  - capture network traffic: `sudo tcpdump -i <netif> -w /path/to/dumpfile.pcap`
  - run snort (offline mode) on you pcaps
