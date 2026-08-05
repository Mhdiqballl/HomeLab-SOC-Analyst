# HomeLab SOC Analyst

Project homelab ini saya bangun sendiri dari nol sebagai sarana belajar dan pembuktian skill. Semua konfigurasi, troubleshooting, dan dokumentasi saya kerjakan mandiri — mulai dari setup jaringan dengan VLAN, deploy SIEM Wazuh, mengirim log dari pfSense dan Windows, sampai simulasi serangan dan analisis detection-nya.

\---

## 👤 Tentang Saya

Saya adalah seorang **Network Engineer** yang sedang bertransisi ke bidang Cyber Security. Project ini saya bangun untuk mempraktikkan dan mendemonstrasikan kemampuan monitoring, deteksi ancaman, dan incident response menggunakan tools yang umum dipakai di industri.

* LinkedIn:https://www.linkedin.com/in/mhd-iqball/
* Email: mhdibll17@gmail.com
* No. HP : 08529236259

\---

## 🎯 Tujuan Project

* Membangun homelab yang mensimulasikan lingkungan enterprise kecil (Active Directory, endpoint, firewall)
* Mengimplementasikan SIEM untuk log collection \& monitoring
* Melakukan simulasi serangan berbasis **MITRE ATT\&CK** dan membangun detection rule
* Melatih proses incident response dari triage sampai reporting
* Menunjukkan kemampuan analisis dan dokumentasi teknis untuk persiapan melamar sebagai SOC Analyst

\---

## 🏗️ Arsitektur

!\[Network Architecture](diagrams/network-architecture.png)

*(Ganti gambar di atas dengan diagram arsitektur homelab kamu — bisa dibuat di draw.io)*

Ringkasan arsitektur:

* **Firewall/Router**: pfSense/OPNsense dengan segmentasi VLAN (Management, Server, Endpoint)
* **Domain**: Windows Server + Active Directory
* **Endpoint**: Windows 10/11, Ubuntu Server
* **Attacker Machine**: Kali Linux
* **SIEM**: Wazuh
* **Case Management**: TheHive

\---

## 🛠️ Tools yang Digunakan

|Kategori|Tools|
|-|-|
|Virtualisasi|VirtualBox|
|Firewall \& Network|pfSense / OPNsense|
|SIEM/XDR|Wazuh|
|Endpoint Logging|Sysmon, Windows Event Log, auditd|
|Simulasi Serangan|Kali Linux, Atomic Red Team|
|Case Management/IR|TheHive|
|Threat Intelligence|MISP|
|Detection Rules|Sigma rules|
|Framework Referensi|MITRE ATT\&CK|

\---

## 📂 Struktur Project

|Folder|Deskripsi|
|-|-|
|[`01-network-setup`](./01-network-setup)|Setup jaringan, VLAN, firewall rules|
|[`02-siem-wazuh`](./02-siem-wazuh)|Instalasi \& konfigurasi Wazuh, log source integration|
|[`03-detection-engineering`](./03-detection-engineering)|Simulasi serangan, mapping MITRE ATT\&CK, custom detection rules|
|[`04-incident-response`](./04-incident-response)|Case management, contoh incident report|
|[`diagrams`](./diagrams)|Diagram arsitektur jaringan|

\---

## 🚀 Progress / Roadmap

* \[x] Setup infrastruktur dasar (VirtualBox, jaringan)
* \[ ] Instalasi \& konfigurasi SIEM (Wazuh)
* \[ ] Log integration (Sysmon, Windows Event Log, firewall log)
* \[ ] Simulasi serangan \& detection engineering
* \[ ] Incident response case study
* \[ ] Threat hunting

\---

## 🔍 Highlight / Key Findings

*(Bagian ini diisi setelah project berjalan — contoh cara menulisnya:)*

* Berhasil mendeteksi teknik **T1110 (Brute Force)** menggunakan custom rule di Wazuh
* Membuat 3 Sigma rule untuk deteksi lateral movement di lingkungan Active Directory
* Menyelesaikan simulasi insiden end-to-end dari alert hingga incident report

\---

## 📌 Catatan

Seluruh konfigurasi, IP address, dan kredensial pada dokumentasi ini telah disamarkan (sanitized) untuk keperluan publikasi. Project ini dibangun murni untuk tujuan pembelajaran di lingkungan lab yang terisolasi.

