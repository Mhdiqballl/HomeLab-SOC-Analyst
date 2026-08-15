# Fase 1: Network Setup

## Tujuan
Membangun fondasi jaringan homelab dengan pfSense sebagai firewall/router, 
segmentasi VLAN (Management, Server, Endpoint), dan Windows Server sebagai 
Domain Controller.

## Tools yang Digunakan
- VirtualBox 7.2.12 r174389
- pfSense CE 2.8.1
- Windows Server 2022
- Windows 11
- Ubuntu 22.04 Server
- Kali Linux 2026.2

## Langkah yang Dilakukan
1. Install pfSense sebagai virtual firewall/router
2. Konfigurasi interface WAN, LAN, dan Host-only Adapter (OPT1) untuk akses GUI dari host
3. Buat 3 VLAN: Management (10), Server (20), Endpoint (30)
4. Assign VLAN jadi interface aktif dengan IP static masing-masing
5. Setup DHCP server untuk tiap VLAN
6. Install Windows Server 2022, promosikan jadi Active Directory Domain Controller
7. Join Windows 11 endpoint ke domain HOMELAB.local
8. Tambah adapter Internal Network terpisah via VBoxManage CLI (karena keterbatasan UI VirtualBox)
9. Pindahkan semua VM ke Internal Network sesuai VLAN
10. Set static IP di setiap VM sesuai subnet VLAN

## Skema IP Final (Setelah VLAN)
| Interface | IP | DHCP Range |
|---|---|---|
| LAN | 192.168.1.1/24 | - |
| MGMT (VLAN 10) | 192.168.10.1/24 | .100-.200 |
| SERVER (VLAN 20) | 192.168.20.1/24 | .100-.200 |
| ENDPOINT (VLAN 30) | 192.168.30.1/24 | .100-.200 |

## Pembagian VM per VLAN
| VM | VLAN | IP |
|----|------|----|
| Kali Linux | MGMT (10) | 192.168.10.100 (DHCP) |
| Windows Server AD | SERVER (20) | 192.168.20.10 |
| Ubuntu-Wazuh | SERVER (20) | 192.168.20.30 |
| Windows 11 | ENDPOINT (30) | 192.168.30.10 |

## Troubleshooting VLAN
### Masalah: VLAN ID Tidak Muncul di Device Manager
Driver virtual Intel PRO/1000 MT di VirtualBox tidak mendukung VLAN tagging secara penuh.
Opsi "Priority & VLAN" hanya mengaktifkan fitur VLAN, tetapi tidak menyediakan field untuk
memasukkan VLAN ID (10, 20, 30).

### Solusi: Internal Network Terpisah via VBoxManage CLI
VBoxManage modifyvm "pfSense-Firewall" --nic5 intnet --intnet5 "LabNet-MGMT"
VBoxManage modifyvm "pfSense-Firewall" --nic6 intnet --intnet6 "LabNet-SERVER"
VBoxManage modifyvm "pfSense-Firewall" --nic7 intnet --intnet7 "LabNet-ENDPOINT"

## Hasil
- MGMT (VLAN 10) → LabNet-MGMT (em3)
- SERVER (VLAN 20) → LabNet-SERVER (em4)  
- ENDPOINT (VLAN 30) → LabNet-ENDPOINT (em5)

## Kendala yang Ditemui
- Instalasi pfSense sempat lambat/stuck karena resource laptop, diatasi dengan Machine Reset
- Firewall OPT1 awalnya memblokir semua traffic, diatasi dengan menambah firewall rule Allow All
- VLAN ID tidak tersedia di driver VirtualBox, diatasi dengan Internal Network terpisah via CLI

## Hasil Akhir
- ✅ Jaringan lab dengan 3 VLAN tersegmentasi dan Domain Controller aktif
- ✅ Semua VM dapat berkomunikasi lintas VLAN melalui pfSense
- ✅ DHCP berfungsi di setiap VLAN

## Screenshot

### pfSense Configuration

#### Pfsense Console
![Pfsense Console](screenshots/pfsense-console.png)

#### VLAN Server Config
![VLAN Server Config](screenshots/pfsense-interface-server-config.png)

#### ARP Table
![ARP Table](screenshots/pfsense-arp-table.png)

#### Interface Assignments
![Interface Assignments](screenshots/pfsense-interface-assignments.png)

#### Management Interface
![Management Interface](screenshots/pfsense-interface-mgmt-config.png)

#### Server Interface
![Server Interface](screenshots/pfsense-interface-server-config.png)

#### Endpoint Interface
![Endpoint Interface](screenshots/pfsense-interface-endpoint-config.png)

#### DHCP Management
![DHCP Management](screenshots/pfsense-dhcp-mgmt.png)

#### DHCP Server
![DHCP Server](screenshots/pfsense-dhcp-server.png)

#### DHCP Endpoint
![DHCP Endpoint](screenshots/pfsense-dhcp-endpoint.png)

#### Firewall Rules Management
![Firewall Rules Management](screenshots/pfsense-firewall-rules-mgmt.png)

#### Firewall Rules Server
![Firewall Rules Server](screenshots/pfsense-firewall-rules-server.png)

#### Firewall Rules Endpoint
![Firewall Rules Endpoint](screenshots/pfsense-firewall-rules-endpoint.png)

#### Firewall Rules OPT1
![Firewall Rules OPT1](screenshots/pfsense-firewall-rules-opt1.png)

#### VLAN List
![VLAN List](screenshots/pfsense-vlan-list.png)


### Windows Server (Active Directory)

#### AD DS DNS Active
![AD DS DNS Active](screenshots/windows-server-adds-dns-active.png)

#### Static IP Configuration
![Static IP Configuration](screenshots/ip-static-windows-server.png)


### Windows 11 Endpoint

#### Static IP Configuration
![Static IP Configuration](screenshots/ip-static-windows-endpoint.png)

#### Domain Login
![Domain Login](screenshots/windows-endpoint-login-domain.png)


### Ubuntu Server

#### Static IP Configuration
![Static IP Configuration](screenshots/ip-static-ubuntu-wazuh.png)

#### SSH Login
![SSH Login](screenshots/ubuntu-server-login-ssh.png)


### VirtualBox Configuration

#### Host-Only Network Manager
![Host-Only Network Manager](screenshots/virtualbox-host-only-network-manager.png)

#### pfSense Internal Network
![pfSense Internal Network](screenshots/virtualbox-network-settings-pfsense-2.png)

#### pfSense Internal Network
![pfSense Internal Network](screenshots/virtualbox-network-settings-pfsense-3.png)

#### Ubuntu Server Adapter 1
![Ubuntu Server Adapter 1](screenshots/ubuntu-server-adapter-1.png)

#### Ubuntu Server Adapter 2
![Ubuntu Server Adapter 2](screenshots/ubuntu-server-adapter-2.png)

#### Windows Endpoint
![Windows Endpoint](screenshots/windows-endpoint-adapter1.png)

#### Windows Server
![Windows Server](screenshots/windows-server-adapter1.png)

#### VBoxManage VM Modification
![VBoxManage VM Modification](screenshots/VBoxManage-modifyvm.png)


### Verifikasi Konektivitas

#### Ubuntu Wazuh Connectivity
![Ubuntu Wazuh Connectivity](screenshots/verifikasi-konektivitas-ubuntu-wazuh.png)

#### Windows Server Connectivity
![Windows Server Connectivity](screenshots/verifikasi-konektivitas-windows-server.png)

#### Windows Endpoint Connectivity
![Windows Endpoint Connectivity](screenshots/verifikasi-konektivitas-windows-endpoint.png)

#### Kali Linux Connectivity
![Kali Linux Connectivity](screenshots/verifikasi-konektivitas-kali-linux.png)


### Masalah: VLAN ID Tidak Muncul di Device Manager

#### VLAN Device Manager Issue
![VLAN Device Manager Issue](screenshots/windows-server-error-vlan-id-device-manager.png)