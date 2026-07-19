# Enterprise-Grade Site-to-Site IKEv2 IPsec VPN & Monitoring Lab

## Overview
This repository documents the architecture and implementation of a secure, segmented Site-to-Site IKEv2 IPsec VPN tunnel established between an Ubuntu Server 24.04 endpoint that is etsablsihed on Raspberry Pi (running StrongSwan) and a pfSense Firewall appliance running as a Type-2 VM on VMware. The tunnel uses IKEv2 with mutual RSA certificate authentication, providing encrypted communication between two dedicated subnets without the use of Pre-Shared Keys (PSK).

## Core Technologies & Tools
- Operating Systems: Ubuntu Server 24.04 LTS, pfSense CE, Ubuntu Desktop (LAN Client)
- Hypervisor: VMware (Type-2 Hypervisor environment)
- VPN Daemon: StrongSwan (Charon IKE daemon).
- Security & Encryption: OpenSSL (Custom CA & Certificate Generation), IKEv2, Mutual RSA, AES-256-GCM / SHA256.
- Monitoring: Zabbix Server

## Network Architecture & Topology

To mirror enterprise best practices, the Ubuntu Server utilizes a dedicated virtual loopback/dummy interface ("dummy0"). This ensures that home network traffic remains fully isolated and only explicit, segmented traffic passes through the encrypted tunnel.

Raspberry Pi (Ubuntu server) -> eth0 (Lab WAN: 192.168.X.X)
  - dummy interface -> (10.X.X.X)                   
  
pfSense Firewall -> [WAN Interface: 192.168.X.X], [DMZ Interface: 172.16.X.X], [LAN Subnet: 192.168.X.X] 
  
IPsec Tunnel (IKEv2 + Mutual RSA):
10.20.20.0/24 <══════════════════> 192.168.50.0/24                                                        

## PKI Architecture

A dedicated Certificate Authority was created using OpenSSL to issue and sign certificates for both endpoints. This eliminates the need for Pre-Shared Keys and provides cryptographic proof of identity for each side. Each certificate includes:

- 4096-bit RSA key
- Subject Alternative Name (SAN) matching the endpoint IP
- Extended Key Usage: serverAuth

## Key Implementation details 

Virtual (Dummy) Interface -> Instead of advertising the entire home network through the tunnel, a dedicated virtual interface (dummy0) was created with a isolated subnet. This cleanly separates tunnel traffic from home network traffic and accurately reflects how production environments segment VPN traffic from other interfaces.

Split Tunnel Design -> the tunnel is configured as a split tunnel by design. Only traffic between dummy subnet and LAN subnet is encrypted and routed through the IPsec SA. All other traffic (internet, home LAN) continues normally through eth0 without touching the tunnel.

Certificate-Based Authentication vs PSK -> Pre-Shared Keys are simpler to configure but share a single secret between both sides but if it is compromised, the entire tunnel is at risk. Mutual RSA authentication with certificates means each side has its own private key that never leaves the machine. Authentication is proven cryptographically through digital signatures verified against the shared CA, which is a significantly stronger security model.
