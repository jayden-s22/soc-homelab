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

## Skills Demonstrated
- Authored and debugged Snort rules, including a directionality issue by tracing how FTP response codes travel
- Mapped every detection to MITRE ATT&CK techniques (T1110.001, T1190, T1059.004)
- Simulated real attacks (Hydra brute force, vsftpd backdoor exploitation) and validated detection rules fired correctly
- Understood the difference between stateful and stateless detection and chose the correct flow keyword for rules and applied it to match traffic
