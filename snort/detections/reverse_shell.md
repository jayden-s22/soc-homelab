# Reverse Shell Detection
## Rule:
`alert tcp 172.30.1.28 any -> 172.30.1.20 any (flow:established,to_server; msg:"REVERSE SHELL DETECTED"; sid:1000004; rev:1;)`
## Why it exists:
Metasploitable's normal role in this lab is to be only a server, where it hosts services like FTP and only ever accepts inbound connections. It has no reason to initiate an outbound connection to anything in this environment. A reverse shell flips that, where the server starts behaving like a client and initiates an outbound connection back to an attacker-controlled listener. This rule detects that role violation directly because a generic reverse shell has no universal content signature once the session is established, unlike the VSFTPD backdoor.
## MITRE ATT&CK:
- T1095 — Non-Application Layer Protocol
- T1059.004 — Command and Scripting Interpreter: Unix Shell
## Detection:
- Header matches IP direction only: src=172.30.1.28 (Metasploitable), dst=172.30.1.20 (Kali), any port on either side
- Destination port left as `any` — a reverse shell listener can bind to any port chosen by the attacker, so there is no fixed port to detect
- Source port left as `any` — ephemeral source ports are used by virtually all outbound client connections from a Linux host, so specifying a source port range added no real detection value
- `flow:established,to_server` is the main part of the rule: it requires the packet to belong to a fully established TCP session, and to be traveling toward the side that received the original SYN (the server) within that session. In a reverse shell session, Metasploitable is the client and Kali is the server, so a `to_server` packet here means "Metasploitable talking outbound to Kali", which is the role violation this rule is built for
- The rule fires on the ACK packet that completes the three-way handshake, detecting the connection itself instead of any shell commands or content match
- Set to known lab IPs only (172.30.1.28 -> 172.30.1.20) rather than $HOME_NET -> $EXTERNAL_NET, since both the attacker and victim sit inside the same internal network. This rule is explicitly not generalizable, and is only valid because Metasploitable has no reason to initiate outbound traffic in this specific lab
## Tested with:
Manual Netcat reverse shell — listener on Kali (172.30.1.20), reverse connection triggered from Metasploitable (172.30.1.28) on port 443
## Alert output confirmed:
172.30.1.28:46547 -> 172.30.1.20:443
## Result:
[ 06/16-22:21:05.626949 [**] [1:1000004:1] REVERSE SHELL DETECTED [**] [Priority: 0] {TCP} 172.30.1.28:46547 -> 172.30.1.20:443 ]
## Limitations:
- Detects the connection establishment only, not shell content, so any TCP connection from Metasploitable to Kali that completes a handshake will trigger this
- Not generalizable outside this lab since it relies on hardcoded IPs rather than $HOME_NET/$EXTERNAL_NET scoping, and assumes Metasploitable's role is a fixed, known fact about this environment
- No mechanism for true behavioral baseline. Snort evaluates packets/flows statelessly against static rules with no persistent memory of long-term host behavior; that level of detection belongs to a SIEM/ layer
