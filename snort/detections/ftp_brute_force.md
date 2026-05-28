# FTP Brute Force Detection
## Rule: 
alert tcp $HOME_NET 21 -> any any (msg:"BRUTE FORCE DETECTED"; content:"530"; flow:established,to_client; threshold: type threshold, track by_dst, count 10, seconds 10; sid:1000001; rev:1;)

## Why it exists:
This rule detects repeated FTP authentication failures, which indicate a brute force attack. The 530 response code is issued by the FTP server on failed login attempts.

## MITRE ATT&CK: 
T1110.001 — Brute Force: Password Guessing

## Detection: 
- Matches the "530" in server response traffic from port 21
- flow:established,to_client ensures directionality
- Threshold activates after 10 failed logins within 10 seconds to reduce false postives from regular mistyped passwords

## Tested with: 
Hydra brute force from Kali (172.30.1.20) against Metasploitable FTP (172.30.1.28:21)

## Alert output confirmed: 
172.30.1.28:21 -> 172.30.1.20

## Result: 
[ 05/28-03:52:27.801438 [**] [1:1000001:1] BRUTE FORCE DETECTED [**] [Priority: 0] {TCP} 172.30.1.28:21 -> 172.30.1.20:45972 ]

## Limitations:
- Will miss slow brute-force attempts (ex. 1 attempt per minute)
- Threshold count may need tuning for larger environments
