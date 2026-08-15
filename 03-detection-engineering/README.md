# Fase 3: Detection Engineering — Serangan \& Deteksi


## Tujuan

Mensimulasikan berbagai teknik serangan berbasis MITRE ATT\&CK, kemudian menganalisis apakah teknik tersebut terdeteksi oleh Wazuh, dan membuat custom detection rule bila diperlukan.


## Tools yang Digunakan

- Kali Linux (Nmap, Hydra, Metasploit)

- Mimikatz

- DVWA (Damn Vulnerable Web Application)

- Wazuh Dashboard

- Sysmon


## Cakupan

Project ini mencakup Tier 1 sampai Tier 3 dari roadmap serangan \& deteksi.


## Ringkasan Case

| Case | Teknik | MITRE ATT&CK | Tools | Status |
|------|--------|---------------|-------|--------|
| 1 | Port Scanning | T1046 | Nmap | ✅ Done |
| 2 | Brute Force | T1110 | Hydra | ✅ Done |
| 3 | Exploitasi Windows Service | T1210 | Metasploit | ✅ Done |
| 4 | Credential Dumping | T1003 | Mimikatz | ✅ Done |
| 5 | Lateral Movement | T1021 | WMI | ✅ Done |
| 6 | SQL Injection | T1190 | Manual | ✅ Done |

---


## Case 1: Port Scanning / Reconnaissance

**MITRE ATT\&CK:** T1046 (Network Service Discovery)  

**Tools:** Nmap (Kali Linux)  

**Target:** Windows 11 (192.168.30.10)


### Langkah Serangan

1. `nmap -sn 192.168.30.0/24` — discovery scan

2. `nmap -sV -Pn 192.168.30.10` — port scan detail

3. Hasil: 3 port terbuka (135, 139, 445)


### Deteksi

| Layer | Status | Keterangan |
|-------|--------|------------|
| Wazuh | ❌ Tidak mendeteksi | Tidak ada built-in rule untuk network scan |
| pfSense Firewall Log | ✅ Tercatat | Traffic ICMP/TCP SYN dari Kali |
| UFW Block Ubuntu | ✅ Tercatat | Upaya koneksi diblokir |


### Analisis

Wazuh sebagai HIDS fokus pada aktivitas endpoint, bukan traffic jaringan. Perlu custom rule untuk deteksi port scanning.

---

## Case 2: Brute Force Authentication

**MITRE ATT\&CK:** T1110 (Brute Force)  

**Tools:** Hydra (Kali Linux)  

**Target:** Ubuntu-Wazuh (192.168.20.30)


### Langkah Serangan

1. Buat wordlist 10 password umum

2. `hydra -l wazuhadmin -P wordlist.txt ssh://192.168.20.30`

3. Hasil: 10 percobaan gagal (0 valid)



### Deteksi

| Layer | Status | Keterangan |
|-------|--------|------------|
| Wazuh Rule 92031 | ✅ Terdeteksi | Discovery activity executed |
| pfSense | ✅ Tercatat | Traffic SMB port 445 |

---

## Case 3: Exploitasi Windows Service

**MITRE ATT&CK:** T1210 (Exploitation of Remote Services)  

**Tools:** Metasploit (Kali Linux)  

**Target:** Windows 11 (192.168.30.10)


### Langkah Serangan

1. EternalBlue: gagal — target tidak vulnerable

2. SMB Delivery: berhasil

3. NTLMv2 hash Administrator tertangkap


### Deteksi

| Layer | Status | Keterangan |
|-------|--------|------------|
| Wazuh Rule 5760 | ✅ Terdeteksi | sshd authentication failed |
| Journal SSH Ubuntu | ✅ Tercatat | 10 failed password attempt |

---

## Screenshot


### Case 1: Port Scanning

## Nmap Scan
![Nmap Scan](screenshots/tier1-case1-kali-nmap-scan-port.png)

## pfSense Firewall Log
![pfSense Firewall Log](screenshots/tier1-case1-pfsense-filterlog.png)

![UFW Block](screenshots/tier1-case1-ufw-block-wazuh-archives.png)



### Case 2: Brute Force

![Hydra Attack](screenshots/tier1-case2-hydra-attack.png)

![Wazuh Alert](screenshots/tier1-case2-wazuh-alert.png)


### Case 3: Exploitasi Windows

![Metasploit Exploit](screenshots/tier2-case3-metasploit-exploit.png)

![Wazuh Detection](screenshots/tier2-case3-wazuh-92031.png)


### Case 4: Credential Dumping

![Mimikatz Output](screenshots/tier2-case4-mimikatz-output.png)

![Sysmon Event 10](screenshots/tier2-case4-sysmon-event10.png)

![Wazuh Level 12](screenshots/tier2-case4-wazuh-rule.id-92900.png)


### Case 5: Lateral Movement

![WMI Command](screenshots/tier2-case5-wmi-command.png)

![Sysmon Event 1](screenshots/tier2-case5-sysmon-event1.png)

![Wazuh Detection](screenshots/tier2-case5-wazuh-92031.png)



### Case 6: SQL Injection

![DVWA Setup](screenshots/tier3-case6-dvwa-setup.png)

![SQLi Normal](screenshots/tier3-case6-sqli-normal.png)

![SQLi Error](screenshots/tier3-case6-sqli-error.png)

![SQLi Dump](screenshots/tier3-case6-sqli-dump.png)

![Apache Log](screenshots/tier3-case6-apache-log.png)

![Wazuh Agent Alert](screenshots/tier3-case6-wazuh-agent-alert.png)

