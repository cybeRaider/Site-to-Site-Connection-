# Enterprise-Grade Site-to-Site IKEv2 IPsec VPN & Monitoring Lab

## Overview
This repository documents the architecture and implementation of a secure, segmented Site-to-Site IKEv2 IPsec VPN tunnel established between an Ubuntu Server 24.04 endpoint that is etsablsihed on Raspberry Pi(running StrongSwan) and a pfSense Firewall appliance. 

Designed as a professional portfolio piece, this project demonstrates advanced networking concepts including a custom OpenSSL-based PKI for mutual RSA authentication, advanced firewall design, and virtual network segmentation. 
Additionally, the infrastructure is integrated with a Zabbix Server deployment for active telemetry and node monitoring.

## Core Technologies & Tools
- Operating Systems: Ubuntu Server 24.04 LTS, pfSense CE, Ubuntu Desktop (LAN Client)
- Hypervisor: VMware (Type-2 Hypervisor environment)
- VPN Daemon: StrongSwan (Charon IKE daemon).
- Security & Encryption: OpenSSL (Custom CA & Certificate Generation), IKEv2, Mutual RSA, AES-256-GCM / SHA256.
- Monitoring: Zabbix Server

## Network Architecture & Topology

To mirror enterprise best practices, the Ubuntu Server utilizes a dedicated virtual loopback/dummy interface ("dummy0"). This ensures that home network traffic remains fully isolated and only explicit, segmented traffic passes through the encrypted tunnel.

  Ubuntu Server (Lab WAN: 192.168.X.X)                    pfSense Firewall
+------------------------------------+           +----------------------------------+
|  [Subnet: 10.X.X.0/24]           |             |  [WAN Interface: 192.168.X.X]  |
|  Interface: dummy0 (10.X.X.1)    |             |  [DMZ Interface: 172.16.X.1]   |
|                                  |             |  [LAN Subnet: 192.168.X.0/24]  |
|                                  |             |                                |
|   +----------------------------+ |    IPsec    |   +--------------------------+ |
|   | StrongSwan VPN Daemon      |=|=============|===| pfSense IPsec Core Engine| |
|   +----------------------------+ | (IKEv2/RSA) |   +--------------------------+ |
+------------------------------------+           +----------------------------------+
                                                                    |
                                                                    v
                                                              Ubuntu Linux VM
                                                              (192.168.X.X)
