\# Fase 2: SIEM Setup — Wazuh



\## Tujuan

Membangun SIEM (Security Information and Event Management) menggunakan Wazuh untuk mengumpulkan, menganalisis, dan mendeteksi ancaman dari seluruh endpoint di jaringan lab.



\## Tools yang Digunakan

\- Wazuh 4.14.6 (Indexer, Server/Manager, Dashboard — all-in-one installer)

\- Ubuntu Server 22.04 LTS

\- Sysmon 15.21 (Windows endpoint logging)

\- Filebeat 7.10.2

\- Windows Server 2022, Windows 11 (agent)

\- pfSense CE 2.8.1 (syslog forwarding)



\## Arsitektur Final (Setelah VLAN)

\- Wazuh Server: Ubuntu 22.04 — `192.168.20.30` (SERVER VLAN 20)

\- Dashboard: `https://192.168.56.104` (Host-Only)

\- Agent Windows Server: `192.168.20.10` (SERVER VLAN 20)

\- Agent Windows 11: `192.168.30.10` (ENDPOINT VLAN 30)

\- Agent DVWA: `192.168.20.40` (DVWA VLAN 20)

\- pfSense Log Forwarding: syslog UDP 514 → Wazuh archives



\## Komponen Status

| Komponen | Status |

|----------|--------|

| Wazuh Indexer | ✅ Running |

| Wazuh Manager | ✅ Running |

| Wazuh Dashboard | ✅ Running (HTTPS) |

| Agent Windows Server (ID 001) | ✅ Active |

| Agent Windows 11 (ID 002) | ✅ Active |

| Sysmon | ✅ Running |

| pfSense Log Forwarding | ✅ Archives |



\## Kredensial

\- User: `admin`

\- Password: (tersimpan di `wazuh-install-files.tar` → `wazuh-passwords.txt`)



\## Langkah yang Dilakukan

1\. Update sistem Ubuntu, konfigurasi static IP dan DNS

2\. Pre-check port 443/80 sebelum instalasi

3\. Download dan jalankan installer Wazuh (`wazuh-install.sh -a`)

4\. Verifikasi akses dashboard dan catat kredensial admin

5\. Buka port firewall (UFW) untuk komunikasi agent (1514, 1515, 55000, 443)

6\. Install Wazuh Agent di Windows Server dan Windows 11

7\. Install Sysmon di kedua endpoint Windows, hubungkan log ke Wazuh Agent

8\. Konfigurasi pfSense remote syslog ke Wazuh (UDP 514)

9\. Update konfigurasi agent dan pfSense setelah perubahan IP VLAN

10\. Verifikasi semua agent berstatus Active di dashboard



\## Troubleshooting Utama



\### Instalasi Wazuh

Proses instalasi Wazuh mengalami beberapa kegagalan berurutan sebelum berhasil:



\- \*\*Ubuntu 26.04\*\*: Wazuh Dashboard gagal install — OS belum didukung resmi oleh Wazuh 4.14

\- \*\*Ubuntu 22.04\*\*: Wazuh Indexer timeout setelah ±25 menit — resource CPU/RAM VM kurang

\- \*\*Solusi\*\*: naikkan RAM VM ke 6GB, extend timeout systemd (`TimeoutStartSec`), matikan VM lain saat instalasi, set heap Java ke 512MB, gunakan discovery.type: single-node`

\- \*\*Disk penuh (100%)\*\*: `/var/ossec` tumbuh hingga 17GB → resize LVM dari 29GB ke 57GB



\### pfSense Log Forwarding

\- \*\*Masalah\*\*: log pfSense tidak muncul di dashboard meskipun konfigurasi terlihat benar

\- \*\*Diagnosis\*\*: `tcpdump` menunjukkan 0 packets — pfSense tidak mengirim data

\- \*\*Akar masalah #1\*\*: rule firewall pfSense tidak mencentang opsi "Log packets" → filterlog tidak tercatat

\- \*\*Akar masalah #2\*\*: pfSense mengecualikan `filterlog` dari remote syslog (`!filterlog` di `pfSense.conf`)

\- \*\*Akar masalah #3\*\*: Wazuh menolak syslog dari non-agent → daftarkan pfSense sebagai agent, update `allowed-ips`

\- \*\*Solusi akhir\*\*: hapus pengecualian `!filterlog`, aktifkan logging di firewall rules, set `allowed-ips: 192.168.20.1`, restart syslogd pfSense



\### Konfigurasi pfSense (Remote Syslog):

\- Remote Syslog Server: `192.168.20.30:514`

\- Source Address: SERVER (`192.168.20.1`)

\- Centang Firewall Events

\- Hapus pengecualian `!filterlog` di `/var/etc/syslog.d/pfSense.conf`



\### Konfigurasi Wazuh (ossec.conf):

xml

<remote>

&#x20; <connection>syslog</connection>

&#x20; <port>514</port>

&#x20; <protocol>udp</protocol>

&#x20; <allowed-ips>192.168.20.1</allowed-ips>

</remote>



\## Agent Windows (ossec.conf)

<address>192.168.20.30</address>



\## Contoh Data pfSense (archives.json)

{

&#x20; "predecoder": {

&#x20;   "program\_name": "filterlog",

&#x20;   "hostname": "pfsense.home.arpa"

&#x20; },

&#x20; "decoder": { "name": "pf" },

&#x20; "data": {

&#x20;   "protocol": "udp",

&#x20;   "action": "pass",

&#x20;   "srcip": "192.168.1.20",

&#x20;   "srcport": "49957",

&#x20;   "dstip": "8.8.8.8",

&#x20;   "dstport": "53"

&#x20; },

&#x20; "location": "192.168.20.1"

}





\\\\## Hasil

✅ Wazuh fully operational dengan dashboard HTTPS

✅ 2 agent Windows aktif mengirim log (security events + Sysmon)

✅ pfSense firewall log berhasil masuk ke Wazuh archives

✅ Alert Windows logon failure, Sysmon process access, dan SSH brute force terkonfirmasi



\## Konfigurasi Network Ubuntu (netplan)

network:

&#x20; ethernets:

&#x20;   enp0s3:

&#x20;     dhcp4: no

&#x20;     addresses: \[192.168.20.30/24]

&#x20;     routes:

&#x20;       - to: default

&#x20;         via: 192.168.20.1

&#x20;     nameservers:

&#x20;       addresses: \[192.168.20.1, 8.8.8.8]

&#x20;   enp0s8:

&#x20;     dhcp4: true

&#x20; version: 2



\## Screenshot

\### Instalasi Wazuh

!\[Log Instalasi Berhasil](screenshots/wazuh-installation-done.png)

!\[Dashboard Login](screenshots/wazuh-dashboard-first-login.png)



\### Agents

!\[Agents Active](screenshots/dashboard-wazuh-agent-active.png)

!\[Agents Active](screenshots/dashboard-wazuh-agent-dvwa-active.png)

!\[Update Wazuh Agent](screenshots/update-wazuh-agent-windows.png)





\### Sysmon

!\[Sysmon Configuration - Windows Server](screenshots/config-sysmon-windows-server.png)

!\[Sysmon Configuration - Windows Endpoint](screenshots/config-sysmon-windows-endpoint.png)



\### Alerts

!\[Login Failure Alert - Windows Server](screenshots/alert-login-failure-windows-server.png)

!\[Login Failure Alert - Windows Endpoint](screenshots/alert-login-failure-windows-endpoint.png)



\### pfSense Log Forwarding

!\[pfSense Remote Syslog Configuration](screenshots/pfsense-remote-syslog-config.png)

!\[OSSEC Log Forwarding Configuration](screenshots/konfigurasi-ossec.conf-log-forwarding-pfsense.png)

!\[pfSense Filterlog Archives](screenshots/pfsense-filterlog-archives.png)

