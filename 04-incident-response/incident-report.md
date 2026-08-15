\# Incident Report: Credential Dumping via Mimikatz



\*\*Incident ID:\*\* IR-2026-001

\*\*Severity:\*\* Critical (Level 12)

\*\*Status:\*\* Resolved

\*\*Date:\*\* August 7, 2026

\*\*Analyst:\*\* Muhammad Iqbal



\---



\## Executive Summary



Pada 7 Agustus 2026 pukul 00:07 UTC, Wazuh mendeteksi aktivitas mencurigakan di Windows Server (WIN-IQBGMC8820T). Proses `mimikatz.exe` terdeteksi mengakses `lsass.exe`, yang merupakan indikasi Credential Dumping (MITRE ATT\&CK T1003). Investigasi mengkonfirmasi bahwa tool Mimikatz berhasil mengekstrak NTLM hash kredensial Administrator.



\---



\## Timeline



| Waktu (UTC) | Kejadian | Sumber |

|-------------|----------|--------|

| 00:05 | Windows Defender real-time protection dimatikan | Windows Event Log |

| 00:07:02 | `mimikatz.exe` dijalankan dari Desktop Administrator | Sysmon Event ID 1 |

| 00:07:12 | `mimikatz.exe` mengakses `lsass.exe` | Sysmon Event ID 10 |

| 00:07:18 | NTLM hash Administrator berhasil diekstrak | Mimikatz output |

| 00:08:54 | Wazuh trigger alert Rule 92900 Level 12 | Wazuh Dashboard |



\---



\## Indicator of Compromise (IOC)



| Tipe | Nilai | Deskripsi |

|------|-------|-----------|

| File | `C:\\Users\\Administrator\\Desktop\\mimikatz.exe` | Tool credential dumping |

| Hash NTLM | `af96405e0271c2fc308be85d9e487f7c` | Hash Administrator yang dicuri |

| Process | `mimikatz.exe → lsass.exe` | Process access ke LSASS |

| User | `HOMELAB\\Administrator` | Akun yang dikompromikan |

| IP Target | `192.168.20.10` | Windows Server terdampak |



\---



\## Impact Assessment



\- \*\*Kredensial terdampak:\*\* Administrator domain

\- \*\*Sistem terdampak:\*\* Windows Server (192.168.20.10)

\- \*\*Potensi lanjutan:\*\* Lateral movement ke endpoint lain, akses ke Domain Controller

\- \*\*Data yang dicuri:\*\* NTLM hash, Kerberos ticket



\---



\## Root Cause Analysis



1\. Windows Defender dinonaktifkan — memungkinkan Mimikatz berjalan tanpa deteksi

2\. Tidak ada LSA Protection — `lsass.exe` dapat diakses oleh proses non-system

3\. User menjalankan executable tidak dikenal dari Desktop dengan hak Administrator



\---



\## Containment \& Eradication



| Tindakan | Status |

|----------|--------|

| Isolasi Windows Server dari jaringan | ✅ Done |

| Reset password Administrator domain | ✅ Done |

| Hapus file `mimikatz.exe` | ✅ Done |

| Aktifkan kembali Windows Defender | ✅ Done |

| Enable LSA Protection via registry | ✅ Done |



\---



\## Detection Coverage



| Layer | Status | Keterangan |

|-------|--------|------------|

| Sysmon Event ID 10 | ✅ Detected | Credential Dumping (T1003) |

| Wazuh Rule 92900 | ✅ Detected | Level 12 alert |

| Windows Defender | ❌ Bypassed | Dimatikan sebelum eksekusi |



\---



\## Recommendations



1\. Enable Attack Surface Reduction (ASR) rules — block executable dari folder Desktop/Downloads

2\. Implement LSA Protection — cegah akses ke `lsass.exe`

3\. Audit privileged account usage — pantau setiap penggunaan akun Administrator

4\. Deploy application whitelisting — hanya izinkan aplikasi yang disetujui

5\. Konfigurasi alert Defender — kirim ke Wazuh untuk deteksi berlapis



\---



\## Appendix



\- \*\*Screenshot Sysmon:\*\* `tier2-case4-sysmon-event10.png`

\- \*\*Screenshot Wazuh:\*\* `tier2-case4-wazuh-rule.id-92900.png`

\- \*\*Screenshot Mimikatz:\*\* `tier2-case4-mimikatz-output.png`



\---



\*\*Report prepared by:\*\* Muhammad Iqbal

\*\*Date:\*\* August 15, 2026

\*\*Classification:\*\* Internal

