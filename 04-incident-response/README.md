\# Fase 4: Incident Response



\## Tujuan

Melatih proses investigasi insiden end-to-end menggunakan TheHive, dari triage alert sampai penulisan incident report resmi.



\## Tools yang Digunakan

\- TheHive 3.5.1 (Case Management)

\- Elasticsearch 7.17.0 (Database)

\- Docker (Container runtime)

\- Wazuh (sumber alert)



\## Langkah yang Dilakukan

1\. Install Elasticsearch + TheHive via Docker

2\. Pilih Case 4 (Mimikatz) dari Fase 3 untuk diinvestigasi

3\. Buat case di TheHive dengan timeline, task, dan IOC

4\. Tulis Incident Report resmi



\## Case yang Diinvestigasi

\*\*Case 4: Credential Dumping via Mimikatz\*\*

\- MITRE ATT\&CK: T1003

\- Severity: Critical (Level 12)

\- Target: Windows Server (192.168.20.10)



\## Ringkasan Investigasi



\### Deteksi Awal

Wazuh alert \*\*Rule 92900 Level 12\*\* — "Lsass process was accessed by mimikatz.exe"



\### Timeline

| Waktu (UTC) | Kejadian |

|-------------|----------|

| 00:05 | Windows Defender dimatikan |

| 00:07:02 | mimikatz.exe dijalankan |

| 00:07:12 | mimikatz.exe mengakses lsass.exe |

| 00:08:54 | Wazuh trigger alert Level 12 |



\### IOC (Indicator of Compromise)

| Tipe | Nilai |

|------|-------|

| File | C:\\Users\\Administrator\\Desktop\\mimikatz.exe |

| Hash NTLM | af96405e0271c2fc308be85d9e487f7c |

| User | HOMELAB\\Administrator |

| IP Target | 192.168.20.10 |



\### Root Cause

Windows Defender dinonaktifkan, tidak ada LSA Protection, dan user menjalankan executable tidak dikenal dengan hak Administrator.



\### Rekomendasi

1\. Enable Attack Surface Reduction (ASR) rules

2\. Implement LSA Protection

3\. Audit privileged account usage

4\. Deploy application whitelisting

5\. Konfigurasi alert Defender ke Wazuh



\## Hasil

\- ✅ TheHive berjalan dengan Elasticsearch

\- ✅ Case lengkap dengan task \& IOC

\- ✅ Incident Report tersedia di \[`incident-report.md`](incident-report.md)



\## Screenshot

!\[TheHive Login](screenshots/thehive-login.png)

!\[TheHive User](screenshots/thehive-user-management.png)

!\[TheHive Case](screenshots/thehive-case.png)

!\[TheHive Tasks](screenshots/thehive-tasks.png)

!\[TheHive IOC](screenshots/thehive-ioc.png)

!\[Wazuh Proof](screenshots/wazuh-proof.png)

