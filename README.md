# SOC Homelab

## Purpose
A homelab for documenting network architecture, IDS deployment, and detection engineering with Snort, Kali, and Metasploitable 2. Includes rule writing, attack simulation, and MITRE ATT&CK mapping.

## Environment
[Network Topology](network/topology.md)

## Detections Built
| Rule Name | MITRE Technique | Tool | Status |
|---|---|---|---|
| FTP Brute Force | T1110.001 | Snort | Tested |
| VSFTPD 2.3.4 Backdoor | T1190/T1059.004 | Snort | Tested |
| SYN Scan sfPortscan| T1595 | Snort | Tested |
| Reverse Shell | T1095/T1059.004 | Snort | Tested |

## Skills Demonstrated
- Authored and debugged Snort rules, including a directionality issue by tracing how FTP response codes travel
- Mapped every detection to MITRE ATT&CK techniques
- Simulated real attacks and validated detection rules fired correctly
- Understood the difference between stateful and stateless detection and chose the correct flow keyword for rules and applied it to match traffic
- Identified the limitations of custom Snort rules for port scan detection and chose the sfPortscan preprocessor instead, which was able to track unique destination ports where Snort's threshold rules could not
- Changed from signature to behavioral detection by using role-violation as the basis for detection when generic reverse shells contain no universal content signature
- Traced Snort's flow keyword assignment of client and server roles based on the original SYN packet to confirm 2 rules that watched the same IP wouldn't trigger on each other's traffic 
