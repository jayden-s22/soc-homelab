# VSFTPD 2.3.4 Backdoor Detection
## Rules:
1. alert tcp any any -> $HOME_NET 21 (flow:established,to_server; msg:"BACKDOOR TRIGGER DETECTED"; content:"USER "; content:":)"; distance:1; within:100; sid:1000002; rev:1;)
2. alert tcp any any -> $HOME_NET 6200 (flow:stateless; msg:"BACKDOOR CONNECTION DETECTED";sid:1000003; rev:1;)
## Why they exist:
The backdoor was put by attackers using the T1195 technique of Supply Chain Compromise, by altering the original vsftpd package with a malicious version. Rule 1 detects the exploiting username on port 21, which is the T1190 technique. Rule 2 detects any traffic on port 6200, meaning a root shell has been accessed and a connection has been made, making use of technique T1059.004
## MITRE ATT&CK:
- T1190 — Exploit Public-Facing Application
- T1059.004 — Command and Scripting Interpreter: Unix Shell
## Detection:
- Matches "USER" and ":)" from the attacker on port 21
- Detects traffic to port 6200 from the attacker
- Rule 2 fires first because it detects the very first packet on port 6200
- Rule 1 has to wait for the full handshake on port 21 to complete
## Tested With:
Metasploit console from Kali (172.30.1.20) against Metasploitable FTP (172.30.1.28:21/6200)
## Alert output confirmed:
- 172.30.1.20:41895 -> 172.30.1.28:21
- 172.30.1.20:39457 -> 172.30.1.28:6200
## Result
- 06/11-23:50:35.886612 [**] [1:1000003:1] BACKDOOR CONNECTION DETECTED [**] [Priority: 0] {TCP} 172.30.1.20:39457 -> 172.30.1.28:6200
- 06/11-23:50:35.920247 [**] [1:1000002:1] BACKDOOR TRIGGER DETECTED [**] [Priority: 0] {TCP} 172.30.1.20:41895 -> 172.30.1.28:21
## Limitations:
- A limitation of rule 1 is if a username is greater than 100 characters, the rule would not fire, but it is unreasonable to check for a username any longer than that in this lab environment. If needed, the rule could be changed to catch longer usernames
- A limitation of rule 2 is no content filter means any traffic to port 6200 triggers the rule. In a real environment, adding a content match such as root would reduce false positives
