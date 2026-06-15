# SYN Scan sfPortscan
## sfPortscan Parameters
- proto: tcp, udp, icmp, ip, all; I chose tcp because the Nmap SYN scan is TCP-based
- sense_level:
  - low: static 60-second window, fires on error/RST responses
  - medium: tracks connection counts
  - high: continuous tracking with a sliding window and very sensitive
      - I chose to use the "high" parameter because this is a smaller, controlled lab with only one traffic source. I also wanted guaranteed visibility into the alert format.
- memcap: caps memory allocated for tracking.
    - I left this as the default of 10000000, again because of the small size of the lab
- scan_type: I chose portscan, which is many ports on one target, over portsweep, which is for one port on many hosts
- logfile: resolves relative to the configured Snort log directory (/var/log/snort/)

## Config Line Used
```
preprocessor sfportscan: proto { tcp } memcap { 10000000 } sense_level { high } scan_type { portscan } logfile { portscan.log }
```
## Attack
- Command used: sudo nmap -sS 172.30.1.28 (performs a SYN scan, with root required)
- Prep:
  - Went over the TCP handshake (SYN -> SYN-ACK -> ACK)
  - SYN scan is "half-open" because it never completes the handshake, which means it doesn't show up in app-level logs
  - Open ports return SYN-ACK and closed ports return RST
- Why it's detectable: one SYN appears the same as regular traffic, but multiple SYN's to multiple ports in a short amount of time is the scan signature. 

## Prediction vs. Result
- Before executing the scan and after configuring snort, I predicted that the alerts would come as one batch, with one alert per port, and there would be a message like "portscan detected"
- I was correct on them coming as a batch, but there was not one alert per port because sfPortscan aggregates, and the actual log output was one entry totaling 10 lines

## Log Output
```
Time: 06/15-05:24:58.188618
event_ref: 0
172.30.1.20 -> 172.30.1.28 (portscan) TCP Portscan
Priority Count: 6
Connection Count: 10
IP Count: 1
Scanner IP Range: 172.30.1.20:172.30.1.20
Port/Proto Count: 10
Port/Proto Range: 22:8080
```

## Log Fields Interpretation
- "Connection Count: 10": amount of connections observed within sfPortscan's detection window, not a running total every port that was scanned
- "Priority Count: 6": number of RST/unreachable responses observed within the window. It has a higher priority signal than SYN-ACK because regular clients don't accumulate RSTs, unlike port scans
- "Port/Proto Range: 22:8080": appears to be the range of ports in this snapshot, probably near the end of Nmap's scan

## Why I Chose No Custom Rule
A custom rule would either perform the same action as sfPortscan with weaker logic, since the "threshold" and "detection_filter" parameters can only track packet counts and not unique ports, or it could risk false positives on legitimate multi-connection traffic, like a browser opening parallel connections to one server.

## MITRE Mapping
- T1595 - Active Scanning: this is the parent technique of the Reconnaissance tactic, and for this attack, none of the sub-techniques matched up well
