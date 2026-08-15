# Fase 2: SIEM Setup — Wazuh

## Tujuan

Membangun SIEM (Security Information and Event Management) menggunakan Wazuh untuk mengumpulkan, menganalisis, dan mendeteksi ancaman dari seluruh endpoint di jaringan lab.

## Tools yang Digunakan

* Wazuh 4.14.6 (Indexer, Server/Manager, Dashboard — all-in-one installer)
* Ubuntu Server 22.04 LTS
* Sysmon 15.21
* Filebeat 7.10.2
* Windows Server 2022, Windows 11

## Arsitektur Final

* **Wazuh Server:** Ubuntu 22.04 — 192.168.20.30 (SERVER VLAN 20)
* **Dashboard:** https://192.168.56.104 (Host-Only)
* **Agent Windows Server:** 192.168.20.10 (SERVER VLAN 20)
* **Agent Windows 11:** 192.168.30.10 (ENDPOINT VLAN 30)
* **pfSense Log Forwarding:** syslog UDP 514

## Komponen Status

| Komponen | Status |

|----------|--------|

| Wazuh Indexer | ✅ Running |

| Wazuh Manager | ✅ Running |

| Wazuh Dashboard | ✅ Running |

| Agent Windows Server (ID 001) | ✅ Active |

| Agent Windows 11 (ID 002) | ✅ Active |

| Agent DVWA (ID 003) | ✅ Active |

| Sysmon | ✅ Running |

| pfSense Log Forwarding | ✅ Archives |

## Kredensial

* **User:** admin
* **Password:** *(tersimpan di wazuh-passwords.txt)*

## Langkah yang Dilakukan

1\. Update sistem Ubuntu, konfigurasi static IP

2\. Download dan jalankan installer Wazuh (`wazuh-install.sh -a`)

3\. Verifikasi akses dashboard, catat kredensial admin

4\. Buka port firewall (UFW)

5\. Install Wazuh Agent di Windows Server \& Windows 11

6\. Install Sysmon di kedua endpoint Windows

7\. Konfigurasi pfSense remote syslog

8\. Update agent \& pfSense setelah perubahan IP VLAN

9\. Verifikasi semua agent Active

## Troubleshooting

### Instalasi Wazuh

| Masalah | Solusi |

|---------|--------|

| Ubuntu 26.04 dashboard gagal | Ubuntu belum didukung — turunkan ke 22.04 LTS |

| Indexer timeout ±25 menit | Naikkan RAM VM, extend timeout systemd |

| Disk penuh 100% | Resize LVM dari 29GB ke 57GB |

| SSL error | Reinstall bersih dengan installer resmi |

### pfSense Log Forwarding

| Masalah | Solusi |

|---------|--------|

| tcpdump 0 packets | Aktifkan logging di firewall rule pfSense |

| filterlog tidak terkirim | Hapus pengecualian `!filterlog` di pfSense.conf |

| Syslog ditolak Wazuh | Daftarkan pfSense sebagai agent, update allowed-ips |

## Hasil

* ✅ Wazuh fully operational
* ✅ 3 agent Active (Windows Server, Windows 11, DVWA)
* ✅ Sysmon berfungsi
* ✅ pfSense firewall log masuk archives

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

!\[Filterlog Archives](screenshots/pfsense-filterlog-archives.png)



