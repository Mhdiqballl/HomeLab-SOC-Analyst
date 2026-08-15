# Fase 2: SIEM Wazuh

## Tujuan

Membangun SIEM (Security Information and Event Management) menggunakan Wazuh untuk mengumpulkan, menganalisis, dan mendeteksi ancaman dari seluruh endpoint di jaringan lab.

## Tools yang Digunakan

* Wazuh 4.14 (Indexer, Server/Manager, Dashboard — metode all-in-one)
* Ubuntu Server 22.04 LTS
* Sysmon (Windows endpoint logging)

## Langkah yang Dilakukan

1. Update sistem Ubuntu, konfigurasi static IP dan DNS
2. Pre-check port 443/80 sebelum instalasi
3. Download dan jalankan installer Wazuh (`wazuh-install.sh -a`)
4. Verifikasi akses dashboard dan catat kredensial admin
5. Buka port firewall (UFW) untuk komunikasi agent
6. Install Wazuh Agent di Windows Server dan Windows 10/11
7. Install Sysmon di kedua endpoint Windows, hubungkan log ke Wazuh Agent
8. Verifikasi semua agent berstatus Active di dashboard

## Kendala yang Ditemui (Bagian Penting — Proses Troubleshooting)

Proses instalasi Wazuh mengalami beberapa kegagalan berurutan sebelum berhasil, didokumentasikan sebagai bukti proses debugging nyata:

1. **Percobaan di Ubuntu 26.04**: instalasi Wazuh Dashboard gagal (`ERROR: Wazuh dashboard installation failed`). Setelah riset, ditemukan Ubuntu 26.04 belum masuk daftar OS resmi yang didukung Wazuh 4.14.
2. **Riset kompatibilitas OS**: dokumentasi resmi Wazuh dan laporan bug komunitas (termasuk kasus serupa di Ubuntu 24.04 fresh install) mengarahkan ke kesimpulan bahwa **Ubuntu 22.04 LTS** adalah pilihan paling stabil karena paling matang diuji.
3. **Percobaan di Ubuntu 22.04**: Wazuh Indexer gagal start dengan error `timeout was exceeded` setelah proses hang selama ±25 menit tanpa pesan error yang jelas di log.
4. **Diagnosis**: dicek `vm.max\\\\\\\_map\\\\\\\_count` (sudah sesuai standar `262144`), RAM, dan disk space — mengarah pada dugaan resource CPU/RAM VM kurang untuk proses bootstrap OpenSearch yang berat.
5. **Solusi yang diterapkan**: menaikkan RAM VM, extend timeout systemd (`TimeoutStartSec`), dan mematikan VM lain saat instalasi supaya resource laptop fokus ke satu proses.

### Troubleshooting Log Forwarding pfSense → Wazuh

Setelah Wazuh Server, Agent, dan Sysmon berhasil berjalan normal, muncul kendala baru: log firewall pfSense yang diteruskan lewat syslog **tidak muncul sebagai alert** di dashboard, meskipun konfigurasi terlihat benar. Proses debugging-nya:

1. **Cek konektivitas dasar**: `tcpdump` di sisi Ubuntu awalnya menunjukkan **0 packets captured** di port 514 — artinya pfSense belum benar-benar mengirim data apapun.
2. **Cek isi file log pfSense di Ubuntu**: ternyata hanya berisi log sistem biasa (ntpd, cron), tidak ada event firewall (`filterlog`) sama sekali.
3. **Ditemukan akar masalah #1**: rule firewall pfSense (**"Default allow LAN to any rule"**) defaultnya **tidak mencentang opsi "Log packets that are handled by this rule"** — pfSense tidak otomatis mencatat traffic yang lewat kecuali logging diaktifkan manual per-rule.
4. **Setelah logging diaktifkan**: log `filterlog` mulai muncul dengan format lengkap dan berhasil di-decode oleh decoder bawaan Wazuh (`0455-pfsense\\\\\\\_decoders.xml`) — field seperti `srcip`, `dstip`, `protocol`, `action` berhasil ter-parsing dengan benar (dikonfirmasi lewat `archives.json` setelah `logall\\\\\\\_json` diaktifkan).
5. **Tetap tidak muncul di dashboard sebagai alert**: investigasi ke rule bawaan Wazuh (`0540-pfsense\\\\\\\_rules.xml`) mengungkap penyebabnya — rule `87701` (pfSense firewall drop event) memang sengaja diberi `<options>no\\\\\\\_log</options>` oleh Wazuh, supaya setiap single block event tidak membanjiri dashboard dengan noise.
6. **Solusi**: ditemukan rule `87702` yang levelnya lebih tinggi (level 10) dan **tidak** di-suppress — trigger ketika ada **≥18 block event dalam 45 detik dari sumber IP yang sama** (MITRE ATT\&CK T1110 - Brute Force). Dengan generate traffic block berulang dalam waktu singkat, alert ini berhasil muncul di dashboard.

**Pembelajaran**: ini mengajarkan cara kerja nyata SIEM tuning — tidak semua log yang berhasil di-parse otomatis jadi alert; ada logika threshold dan suppression yang disengaja untuk mengurangi alert fatigue, dan kemampuan membaca ruleset (`decoders`/`rules` XML) untuk mendiagnosis kenapa sebuah event tidak muncul adalah skill inti SOC/detection engineering.

Contoh potongan data hasil parsing yang berhasil (dari `archives.json`):

```json
{
  "predecoder": {
    "program\\\\\\\_name": "filterlog",
    "hostname": "pfsense.home.arpa"
  },
  "decoder": { "name": "pf" },
  "data": {
    "protocol": "udp",
    "action": "pass",
    "srcip": "192.168.1.20",
    "srcport": "49957",
    "dstip": "8.8.8.8",
    "dstport": "53"
  },
  "location": "/var/log/pfSense.log"
}
```

## Hasil

*(isi setelah Wazuh berhasil terinstall dan agent aktif)*

## Screenshot

### Instalasi Wazuh

!\[Log Instalasi Berhasil](screenshots/wazuh-installation-done.png)
!\[Dashboard Login](screenshots/wazuh-dashboard-first-login.png)

### Agents

!\[Agents Active](screenshots/dashboard-wazuh-agent-active.png)
!\[Agent DVWA Active](screenshots/dashboard-wazuh-agent-dvwa-active.png)
!\[Update Wazuh Agent](screenshots/update-wazuh-agent-windows.png)

### Sysmon

!\[Sysmon Windows Server](screenshots/config-sysmon-windows-server.png)
!\[Sysmon Windows Endpoint](screenshots/config-sysmon-windows-endpoint.png)

### Alerts

!\[Login Failure Windows Server](screenshots/alert-login-failure-windows-server.png)
!\[Login Failure Windows Endpoint](screenshots/alert-login-failure-windows-endpoint.png)

### pfSense Log Forwarding

!\[Remote Syslog Config](screenshots/pfsense-remote-syslog-config.png)
!\[OSSEC Config](screenshots/konfigurasi-ossec.conf-log-forwarding-pfsense.png)
!\[Filterlog Archives](screenshots/verifikasi-filterlog-archives.png)



### Konfigurasi Network Ubuntu (netplan)

```yaml
network:
  ethernets:
    enp0s3:
      dhcp4: no
      addresses: \\\\\\\[192.168.1.30/24]
      routes:
        - to: default
          via: 192.168.1.1
      nameservers:
        addresses: \\\\\\\[192.168.1.1, 8.8.8.8]
  version: 2
```

