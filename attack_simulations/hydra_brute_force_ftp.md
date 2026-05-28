# MITRE ATT&CK: T1110.001 - Brute Force: Password Guessing

## Objective
Simulating a realistic brute force attack with Hydra against an FTP server in Metasploitable to test Snort detection rule with sid:1000001.

## Environment
- Attacker:  Kali (172.30.1.20)
- Target:    Metasploitable 2 (172.30.1.28)
- Service:   FTP (port 21)
- Tool used: Hydra

## Prerequisites
- Hydra installed on Kali
- Snort running with sid:1000001 active
- Custom wordlist at ~/wordlist.txt (min. 10 entries to trigger threshold)
- FTP service running on Metasploitable (default)

## What The Attack Does
The Hydra tool on Kali attempted to execute a brute force attack to gain access to the root account on Metasploitable. The attacker was attempting to get privileged access by using a wordlist, sending successive login attempts to exhaust credentials.

## Command Used
hydra -l root -P ~/wordlist.txt ftp://172.30.1.28

## Attacker Perspective
Hydra output: [ 1 of 1 target completed, 0 valid password found ].
Hydra did not find credentials and the attack took about 5 seconds.This simulates a real-world scenario where wordlist selection largely impacts attack success. Attackers typically use lists such as rockyou.txt containing millions of common credentials. The small wordlist used here intentionally kept the attack contained while still generating enough failed attempts to trigger detection. As an attacker, I would be using a much larger wordlist to execute a brute force attack to give a higher chance of login success.

## Defender Perspective
Snort output: [ 05/28-03:52:27.801438 [**] [1:1000001:1] BRUTE FORCE DETECTED [**] [Priority: 0] {TCP} 172.30.1.28:21 -> 172.30.1.20:45972 ].
Snort generated alert sid:1000001 approximately 5 seconds after the hydra attack began. The alert showed the correct sending of the server response code 530 back to the attacker, indicating a failed login.

## Detection Validated
The rule behaved as expected, where only 1 alert went off since there were only 11 login attempts, and if Hydra were to have had 40+ attempts within 10 seconds, I would expect snort to send 4+ alerts of brute force. The ~5 second attack time was expected due to Hydra, in this instance, only using an 11 attempt wordlist.

## Limitations Observed
If the attack were done slower, the snort rule wouldn't have caught it due to the rule configuration. A rate such as 2 attempts every minute would be an attack that this specific rule would not be able to detect as currently configured. To catch a slower attack, the snort rule would need to be looking for a much lower threshold of login attempts within a 10 second interval. One way to help improve this rule is with SIEM correlation rules to look for patterns within logs to better detect suspicious activity.

## Lessons Learned
Before this exercise I understood brute force generally but hadn't considered the detection directionality problem, where the 530 code travels from server to client, not toward the server at port 21. This changed how I think about writing Snort rules, now I trace the exact packet path before deciding rule direction. This exercise also raised the question of how would I detect credential attacks against a service that doesn't return standardized error codes like Metasploitable.

## Recommendations
- Complement this rule with a SIEM correlation rule to catch 
  slower brute force attempts across longer time periods
- Consider logging all 530 responses regardless of threshold 
  to a separate log for historical analysis

## References
- MITRE ATT&CK T1110.001: https://attack.mitre.org/techniques/T1110/001/
- Snort Rule Documentation: https://docs.snort.org
