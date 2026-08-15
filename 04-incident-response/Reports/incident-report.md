# Incident Report: Credential Dumping via Mimikatz

**Incident ID:** `IR-2026-001`

**Severity:** 🔴 **Critical (Level 12)**

**Status:** 🟢 **Resolved**

**Date:** August 7, 2026

**Analyst:** Muhammad Iqbal

---

## Executive Summary

Pada **7 Agustus 2026 pukul 00:07 UTC**, Wazuh mendeteksi aktivitas mencurigakan pada **Windows Server (`WIN-IQBGMC8820T`)**.

Proses `mimikatz.exe` terdeteksi mengakses `lsass.exe`, yang merupakan indikasi **Credential Dumping — MITRE ATT&CK T1003**. Investigasi mengonfirmasi bahwa Mimikatz berhasil mengekstrak **NTLM hash** dari kredensial Administrator.

> **Key Finding:** Mimikatz berhasil melakukan credential dumping setelah Windows Defender dinonaktifkan.

---

## Timeline

| Waktu (UTC)  | Kejadian                                             | Sumber             |
| ------------ | ---------------------------------------------------- | ------------------ |
| **00:05**    | Windows Defender real-time protection dimatikan      | Windows Event Log  |
| **00:07:02** | `mimikatz.exe` dijalankan dari Desktop Administrator | Sysmon Event ID 1  |
| **00:07:12** | `mimikatz.exe` mengakses `lsass.exe`                 | Sysmon Event ID 10 |
| **00:07:18** | NTLM hash Administrator berhasil diekstrak           | Mimikatz output    |
| **00:08:54** | Wazuh memicu Rule 92900 — Level 12                   | Wazuh Dashboard    |

---

## Indicator of Compromise (IOC)

| Tipe          | Nilai                                         | Deskripsi                      |
| ------------- | --------------------------------------------- | ------------------------------ |
| **File**      | `C:\Users\Administrator\Desktop\mimikatz.exe` | Tool credential dumping        |
| **Hash NTLM** | `af96405e0271c2fc308be85d9e487f7c`            | Hash Administrator yang dicuri |
| **Process**   | `mimikatz.exe → lsass.exe`                    | Process access ke LSASS        |
| **User**      | `HOMELAB\Administrator`                       | Akun yang dikompromikan        |
| **IP Target** | `192.168.20.10`                               | Windows Server terdampak       |

---

## Impact Assessment

* **Kredensial terdampak:** Administrator domain
* **Sistem terdampak:** Windows Server (`192.168.20.10`)
* **Potensi lanjutan:** Lateral movement ke endpoint lain dan akses ke Domain Controller
* **Data yang dicuri:** NTLM hash dan Kerberos ticket

### Impact Level

**🔴 Critical**

Credential dumping terhadap akun Administrator berpotensi memungkinkan attacker melakukan **privilege abuse** dan melanjutkan serangan ke sistem lain.

---

## Root Cause Analysis

1. **Windows Defender dinonaktifkan**
   Memungkinkan Mimikatz berjalan tanpa deteksi awal.

2. **LSA Protection tidak aktif**
   `lsass.exe` dapat diakses oleh proses non-system.

3. **Executable tidak dikenal dijalankan dengan hak Administrator**
   User menjalankan executable dari Desktop dengan privilege tinggi.

---

## Containment & Eradication

| Tindakan                             | Status |
| ------------------------------------ | ------ |
| Isolasi Windows Server dari jaringan | ✅ Done |
| Reset password Administrator domain  | ✅ Done |
| Hapus `mimikatz.exe`                 | ✅ Done |
| Aktifkan kembali Windows Defender    | ✅ Done |
| Enable LSA Protection via registry   | ✅ Done |

---

## Detection Coverage

| Layer                  | Status     | Keterangan                 |
| ---------------------- | ---------- | -------------------------- |
| **Sysmon Event ID 10** | ✅ Detected | Credential Dumping (T1003) |
| **Wazuh Rule 92900**   | ✅ Detected | Level 12 alert             |
| **Windows Defender**   | ❌ Bypassed | Dimatikan sebelum eksekusi |

### Detection Flow

```text
Mimikatz Execution
        ↓
Access to LSASS
        ↓
Sysmon Event ID 10
        ↓
Wazuh Rule 92900
        ↓
Level 12 Alert
        ↓
Incident Investigation
```

---

## Recommendations

1. **Enable Attack Surface Reduction (ASR) Rules**
   Block executable dari folder Desktop/Downloads.

2. **Implement LSA Protection**
   Mencegah akses tidak sah terhadap `lsass.exe`.

3. **Audit Privileged Account Usage**
   Pantau setiap penggunaan akun Administrator.

4. **Deploy Application Whitelisting**
   Hanya izinkan aplikasi yang telah disetujui.

5. **Integrate Windows Defender with Wazuh**
   Kirim alert Defender ke Wazuh untuk membangun deteksi berlapis.

---

## Appendix

| Evidence | File |
|---|---|
| Sysmon Event 10 | [Sysmon Event 10](./screenshots/sysmon-event10.png) |
| Wazuh Alert | [Wazuh Alert](./screenshots/wazuh-rule.id-92900.png) |
| Mimikatz Output | [Mimikatz Output](./screenshots/mimikatz-output.png) |

---

## Report Information

**Report prepared by:** Muhammad Iqbal

**Date:** August 15, 2026

**Classification:** `Internal`
