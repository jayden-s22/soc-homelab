# Homelab Network Topology

## VMs and their roles:
- Kali (172.30.1.20) — Attacker
- Ubuntu Desktop (172.30.1.26) — SIEM host (planned)
- Snort (172.30.1.27) — IDS
- Metasploitable 2 (172.30.1.28) — Vulnerable target

### Internal Network
        |--- Kali

        |--- Ubuntu
        
        |--- Snort
        
        |--- Metasploitable ← only lives here
        
### Host-Only
        |--- Ubuntu

## Network: 
All VMs on VirtualBox Internal Network (172.30.1.0/24) for isolation

## Snort promiscuous mode: 
Allow All
