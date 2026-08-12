\# Fase 3: Detection Engineering — Serangan \& Deteksi



\## Tujuan

Mensimulasikan berbagai teknik serangan berbasis MITRE ATT\&CK, kemudian menganalisis apakah teknik tersebut terdeteksi oleh Wazuh, dan membuat custom detection rule bila diperlukan.



\## Tools yang Digunakan

\- Kali Linux (Nmap, Hydra, Metasploit, SQLmap)

\- Mimikatz

\- DVWA (Damn Vulnerable Web Application)

\- Wazuh Dashboard

\- Sysmon

\- PsExec / WMI



\## Cakupan

Project ini mencakup Tier 1 sampai Tier 3 dari roadmap serangan \& deteksi



\## Ringkasan Case

| Case | Teknik | MITRE ATT\&CK | Tools | Status |

|------|--------|-------------|-------|--------|

| 1 | Port Scanning | T1046 | Nmap | ✅ Done |

| 2 | Brute Force | T1110 | Hydra | ✅ Done |

| 3 | Exploitasi Windows Service | T1210 | Metasploit | ✅ Done |

| 4 | Credential Dumping | T1003 | Mimikatz | ✅ Done |

| 5 | Lateral Movement | T1021 | PsExec/WMI | ✅ Done |

| 6 | SQL Injection | T1190 | SQLmap | ✅ Done |



\---



\## Case 1: Port Scanning / Reconnaissance

\- \*\*MITRE ATT\&CK:\*\* T1046 (Network Service Discovery)

\- \*\*Tools:\*\* Nmap (Kali Linux)



\### Skenario

Simulasi reconnaissance phase di mana attacker (Kali Linux, 192.168.10.100) melakukan network scanning terhadap subnet ENDPOINT (192.168.30.0/24) untuk menemukan host aktif dan port terbuka. Target utama adalah Windows 11 (192.168.30.10).



\### Langkah Serangan

1\. \*\*Discovery Scan:\*\* `nmap -sn 192.168.30.0/24` — menemukan host aktif di subnet target

2\. \*\*Port Scan Detail:\*\* `nmap -sV -Pn 192.168.30.10` — mengidentifikasi port dan service yang berjalan

3\. \*\*Hasil:\*\* Ditemukan 3 port terbuka di Windows 11 (135/tcp MSRPC, 139/tcp NetBIOS, 445/tcp SMB)



\### Deteksi

\- ❌ \*\*Wazuh tidak mendeteksi\*\* — tidak ada built-in rule untuk network port scanning

\- ✅ \*\*pfSense Firewall Log\*\* — traffic ICMP/TCP SYN dari 192.168.10.100 ke 192.168.30.0/24 tercatat

\- ✅ \*\*UFW Block (Ubuntu)\*\* — firewall mencatat upaya koneksi dari Kali



\### Analisis

Wazuh sebagai HIDS fokus pada endpoint, bukan traffic jaringan. Perlu custom rule untuk deteksi port scanning.



\### Rekomendasi

Perlu custom detection rule untuk mendeteksi pola scanning (threshold koneksi dari 1 IP dalam waktu singkat) atau integrasi log pfSense ke Wazuh alert pipeline.



\---



\## Case 2: Brute Force Authentication

\- \*\*MITRE ATT\&CK:\*\* T1110 (Brute Force)

\- \*\*Tools:\*\* Hydra (Kali Linux)



\### Skenario

Attacker (Kali Linux) melakukan brute force SSH ke Ubuntu-Wazuh (192.168.20.30) menggunakan wordlist sederhana berisi 10 password umum.



\### Langkah Serangan

1\. \*\*Wordlist:\*\* Dibuat 10 password umum (password, 123456, admin, dll)

2\. \*\*Hydra:\*\* `hydra -l wazuhadmin -P wordlist.txt ssh://192.168.20.30`

3\. \*\*Hasil:\*\* 10 percobaan login gagal (0 valid password)



\### Deteksi

\- ✅ \*\*Wazuh mendeteksi\*\* — alert `rule.id: 5760` (sshd: authentication failed) dari IP 192.168.10.100

\- ✅ \*\*Journal SSH Ubuntu\*\* — 10 failed password attempt tercatat



\### Analisis

Wazuh default rule berhasil mendeteksi brute force SSH tanpa perlu custom rule.



\---



\## Case 3: Exploitasi Windows Service

\- \*\*MITRE ATT\&CK:\*\* T1210 (Exploitation of Remote Services)

\- \*\*Tools:\*\* Metasploit Framework (Kali Linux)



\### Skenario

Attacker mencoba mengeksploitasi Windows 11 menggunakan EternalBlue. Setelah gagal (target sudah dipatch), digunakan SMB Delivery untuk menangkap NTLMv2 hash kredensial Administrator.



\### Langkah Serangan

1\. \*\*EternalBlue:\*\* gagal — target tidak vulnerable

2\. \*\*SMB Delivery:\*\* berhasil — payload dijalankan via `rundll32.exe`

3\. NTLMv2 hash Administrator tertangkap



\### Deteksi

\- ✅ \*\*Wazuh Rule 92031\*\* — Discovery activity executed

\- ✅ \*\*pfSense\*\* mencatat traffic SMB port 445 antar VLAN



\---



\## Case 4: Credential Dumping

\- \*\*MITRE ATT\&CK:\*\* T1003 (OS Credential Dumping)

\- \*\*Tools:\*\* Mimikatz



\### Skenario

Mimikatz dijalankan di Windows Server untuk mengekstrak kredensial dari memory LSASS.



\### Langkah Serangan

1\. Download \& jalankan Mimikatz

2\. `privilege::debug` → `sekurlsa::logonpasswords`

3\. NTLM hash berhasil diekstrak



\### Deteksi

\- ✅ \*\*Sysmon Event ID 10\*\* — Credential Dumping (T1003), mimikatz.exe → lsass.exe

\- ✅ \*\*Wazuh Rule 92900 Level 12\*\* — Lsass process accessed by mimikatz.exe



\---



\## Case 5: Lateral Movement

\- \*\*MITRE ATT\&CK:\*\* T1021 (Remote Services)

\- \*\*Tools:\*\* WMI (wmic)



\### Skenario

Setelah mendapatkan kredensial dari Mimikatz, attacker melakukan lateral movement dari Windows Server ke Windows 11 menggunakan WMI.



\### Langkah Serangan

1\. Konfigurasi Local Security Policy di target

2\. `wmic /node:192.168.30.10 process call create "cmd.exe"`

3\. ReturnValue = 0 (sukses)



\### Deteksi

\- ✅ \*\*Sysmon Event ID 1\*\* — cmd.exe via WmiPrvSE.exe (T1047)

\- ✅ \*\*Wazuh Rule 92031\*\* — Discovery activity executed



\---



\## Case 6: SQL Injection

\- \*\*MITRE ATT\&CK:\*\* T1190 (Exploit Public-Facing Application)

\- \*\*Tools:\*\* Manual SQL Injection (browser)

\- \*\*Target:\*\* DVWA (192.168.20.40)



\### Langkah Serangan

1\. Setup DVWA di Ubuntu Server, install Wazuh Agent

2\. Set security level Low

3\. Error confirmation: `1'` → 500 Internal Server Error

4\. Dump semua user: `1' OR '1'='1` → 5 user muncul

5\. Apache access log mencatat semua traffic SQLi dari Kali



\### Deteksi

\- ✅ Apache access log mencatat serangan (500 error + 200 dump)

\- ✅ Wazuh Agent dvwa-webapp active (Rule 503/506)

\- ❌ Alert SQL Injection spesifik tidak muncul (perlu custom decoder Apache)



\### Analisis

Wazuh default tidak memiliki decoder untuk Apache access log. Perlu custom rule untuk mendeteksi pola SQL Injection (UNION SELECT, OR '1'='1, error 500 berturut-turut).



\---



\## Screenshot



\### Case 1: Port Scanning

!\[Nmap Scan](screenshots/tier1-case1-kali-nmap-scan-port.png)

!\[pfSense Firewall Log](screenshots/tier1-case1-pfsense-filter-log.png)

!\[UFW Block](screenshots/tier1-case1-ufw-block-wazuh-archives.png)



\### Case 2: Brute Force

!\[Hydra Attack](screenshots/tier1-case2-hydra-attack.png)

!\[Wazuh Alert](screenshots/tier1-case2-wazuh-alert.png)



\### Case 3: Exploitasi Windows

!\[Metasploit Exploit](screenshots/tier2-case3-metasploit-exploit.png)

!\[Wazuh Detection](screenshots/tier2-case3-wazuh-92031.png)



\### Case 4: Credential Dumping

!\[Mimikatz Output](screenshots/tier2-case4-mimikatz-output.png)

!\[Sysmon Event 10](screenshots/tier2-case4-sysmon-event10.png)

!\[Wazuh Level 12](screenshots/tier2-case4-wazuh-rule.id-92900.png)



\### Case 5: Lateral Movement

!\[WMI Command](screenshots/tier2-case5-wmi-command.png)

!\[Sysmon Event 1](screenshots/tier2-case5-sysmon-event1.png)

!\[Wazuh Detection](screenshots/tier2-case5-wazuh-92031.png)



\### Case 6: SQL Injection

!\[DVWA Setup](screenshots/tier3-case6-dvwa-setup.png)

!\[SQLi Normal](screenshots/tier3-case6-sqli-normal.png)

!\[SQLi Error](screenshots/tier3-case6-sqli-error.png)

!\[SQLi Dump](screenshots/tier3-case6-sqli-dump.png)

!\[Apache Log](screenshots/tier3-case6-apache-log.png)

!\[Wazuh Agent Alert](screenshots/tier3-case6-wazuh-agent-alert.png)

