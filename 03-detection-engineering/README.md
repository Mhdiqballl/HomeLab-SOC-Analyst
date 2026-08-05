\# Fase 3: Detection Engineering — Serangan \& Deteksi



\## Tujuan

Mensimulasikan berbagai teknik serangan berbasis MITRE ATT\&CK, kemudian menganalisis apakah teknik tersebut terdeteksi oleh Wazuh, dan membuat custom detection rule bila diperlukan.



\## Tools yang Digunakan

\- Kali Linux (Nmap, Hydra, Metasploit, SQLmap)

\- Mimikatz

\- DVWA (Damn Vulnerable Web Application)

\- Wazuh Dashboard

\- Sysmon



\## Cakupan

Project ini mencakup Tier 1 sampai Tier 3 dari roadmap serangan \& deteksi

\## Ringkasan Case

| Case | Teknik | MITRE ATT\&CK | Tools | Status |

|------|--------|-------------|-------|--------|

| 1 | Port Scanning | T1046 | Nmap | ✅ Done |

| 2 | Brute Force | T1110 | Hydra | ✅ Done |

| 3 | Exploitasi Windows Service | T1210 | Metasploit | ⏳ |

| 4 | Credential Dumping | T1003 | Mimikatz | ⏳ |

| 5 | Lateral Movement | T1021 | PsExec/WMI | ⏳ |

| 6 | SQL Injection | T1190 | SQLmap | ⏳ |



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

Wazuh sebagai HIDS fokus pada aktivitas endpoint, bukan traffic jaringan. Nmap SYN scan tidak membuat koneksi TCP penuh sehingga tidak tercatat di Windows Event Log atau Sysmon Event ID 3.



\### Rekomendasi

Perlu custom detection rule untuk mendeteksi pola scanning (threshold koneksi dari 1 IP dalam waktu singkat) atau integrasi log pfSense ke Wazuh alert pipeline.



\### Screenshot

!\[Nmap Scan](screenshots/case1-nmap-scan.png)

!\[pfSense Firewall Log](screenshots/case1-pfsense-log.png)

!\[UFW Block](screenshots/case1-ufw-block.png)



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

Wazuh default rule berhasil mendeteksi brute force SSH tanpa perlu custom rule. Rule 5760 trigger untuk setiap failed authentication.



\### Screenshot

!\[Hydra Attack](screenshots/case2-hydra.png)

!\[SSH Failed Log](screenshots/case2-ssh-failed.png)

!\[Wazuh Alert](screenshots/case2-wazuh-alert.png)



\---



\## Case 3: Exploitasi Windows Service (⏳)

\- \*\*MITRE ATT\&CK:\*\* T1210 (Exploitation of Remote Services)

\- \*\*Tools:\*\* Metasploit Framework (Kali Linux)

\- \*(dokumentasi setelah dikerjakan)\*



\---



\## Case 4: Credential Dumping (⏳)

\- \*\*MITRE ATT\&CK:\*\* T1003 (OS Credential Dumping)

\- \*\*Tools:\*\* Mimikatz

\- \*(dokumentasi setelah dikerjakan)\*



\---



\## Case 5: Lateral Movement (⏳)

\- \*\*MITRE ATT\&CK:\*\* T1021 (Remote Services)

\- \*\*Tools:\*\* PsExec/WMI

\- \*(dokumentasi setelah dikerjakan)\*



\---



\## Case 6: SQL Injection (⏳)

\- \*\*MITRE ATT\&CK:\*\* T1190 (Exploit Public-Facing Application)

\- \*\*Tools:\*\* SQLmap (Kali Linux)

\- \*\*Target:\*\* DVWA

\- \*(dokumentasi setelah dikerjakan)\*

