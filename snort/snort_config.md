# Snort IDS Notes
$HOME_NET: 172.30.1.0/24

$EXTERNAL_NET: !$HOME_NET

[Rule file location](rules/local.rules)

Alert log location: /var/log/snort/alert

Key concepts understood:
- Promiscuous mode allows Snort to see traffic not addressed to it
- Snort is stateless by default, flow keyword adds directionality awareness
- sid range for local rules: 1000000-1999999
