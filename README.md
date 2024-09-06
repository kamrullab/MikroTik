
# MikroTik ISP Router Full Configuration Guide

This comprehensive guide provides all the necessary steps and configurations for setting up a MikroTik router in an ISP environment. It includes instructions for **basic internet configuration**, **Hotspot setup**, **DHCP server**, **NAT/firewall rules**, **VLAN isolation**, **bandwidth management**, **remote access**, and **system monitoring**.

## Table of Contents
1. [Initial Setup](#1-initial-setup)
2. [Basic Internet Configuration](#2-basic-internet-configuration)
3. [Creating a Hotspot](#3-creating-a-hotspot)
4. [NAT and Firewall Setup](#4-nat-and-firewall-setup)
5. [Setting Up DHCP Server](#5-setting-up-dhcp-server)
6. [VLAN and User Isolation](#6-vlan-and-user-isolation)
7. [Bandwidth Control](#7-bandwidth-control)
8. [Remote Management](#8-remote-management)
9. [System Monitoring](#9-system-monitoring)
10. [Backup and Security](#10-backup-and-security)
11. [Important Links and Resources](#11-important-links-and-resources)

---

## 1. **Initial Setup**

Before proceeding with any configuration, it's important to secure your MikroTik router by changing the default password and setting up basic network interfaces.

**Step 1: Login to MikroTik Router**

You can log into your router using **Winbox** or **SSH**.

- **Winbox**: Connect via Winbox utility and click on **Neighbors** to discover your router.
- **SSH**: Open a terminal and type:
  ```bash
  ssh admin@<router_ip_address>
  ```

**Step 2: Change the Default Admin Password**
To enhance security, change the default admin password:
```bash
/user set 0 password=<new_secure_password>
```

---

## 2. **Basic Internet Configuration**

Setting up basic internet access is essential to allow users on your LAN to connect to the internet.

### Step 1: Configure WAN (Internet) Interface

Configure the WAN interface to receive an IP address from the upstream ISP:
```bash
/interface set ether1 name=WAN
/ip dhcp-client add interface=WAN disabled=no
```

If your ISP provides a static IP:
```bash
/ip address add address=<your_static_ip>/24 interface=WAN
/ip route add gateway=<gateway_ip>
```

### Step 2: Configure LAN (Internal) Interface

Set up the internal network with a static IP for the LAN interface:
```bash
/interface set ether2 name=LAN
/ip address add address=192.168.88.1/24 interface=LAN
```

---

## 3. **Creating a Hotspot**

A **Hotspot** is useful for controlling user access and managing authentication. Users will need to authenticate through a captive portal to use the internet.

### Step 1: Set Up the Hotspot
Run the following command to initiate the Hotspot setup wizard:
```bash
/ip hotspot setup
```

- Select **ether2** (LAN interface) or your bridge interface.
- Choose the IP pool range (for example, 192.168.88.10-192.168.88.100).
  
### Step 2: Configure Hotspot Pool and Profile
Define a pool of IP addresses for the Hotspot:
```bash
/ip pool add name=hotspot_pool ranges=192.168.88.10-192.168.88.100
/ip hotspot profile set 0 use-radius=no dns-name=hotspot.local
```

### Step 3: Create Hotspot User
Create users for the Hotspot login:
```bash
/ip hotspot user add name=client1 password=clientpassword profile=default
```

---

## 4. **NAT and Firewall Setup**

Enabling **NAT (Network Address Translation)** allows users on your internal network to access the internet through a single public IP. The firewall adds protection to your network.

### Step 1: Enable NAT (Masquerading)
```bash
/ip firewall nat add chain=srcnat action=masquerade out-interface=WAN
```

This will allow all outgoing traffic to be translated through the public WAN IP.

### Step 2: Firewall Rules
Set up basic firewall rules to secure your router.

- Allow established and related connections:
  ```bash
  /ip firewall filter add chain=input action=accept connection-state=established,related
  ```

- Block all incoming traffic from the WAN interface:
  ```bash
  /ip firewall filter add chain=input action=drop in-interface=WAN
  ```

---

## 5. **Setting Up DHCP Server**

To automatically assign IP addresses to devices connected to your LAN, set up a **DHCP server**.

### Step 1: Create DHCP IP Pool
```bash
/ip pool add name=dhcp_pool ranges=192.168.88.10-192.168.88.254
```

### Step 2: Add DHCP Server
```bash
/ip dhcp-server add address-pool=dhcp_pool interface=LAN name=dhcp_server
/ip dhcp-server network add address=192.168.88.0/24 gateway=192.168.88.1
```

---

## 6. **VLAN and User Isolation**

For ISPs or large networks, VLANs allow you to isolate traffic between different users or user groups.

### Step 1: Create VLAN Interfaces
```bash
/interface vlan add name=vlan10 vlan-id=10 interface=LAN
/interface vlan add name=vlan20 vlan-id=20 interface=LAN
```

### Step 2: Assign IP Addresses to VLANs
```bash
/ip address add address=192.168.10.1/24 interface=vlan10
/ip address add address=192.168.20.1/24 interface=vlan20
```

### Step 3: Isolate VLANs (Inter-VLAN Firewall)
Block communication between VLANs to ensure user isolation:
```bash
/ip firewall filter add chain=forward action=drop in-interface=vlan10 out-interface=vlan20
/ip firewall filter add chain=forward action=drop in-interface=vlan20 out-interface=vlan10
```

---

## 7. **Bandwidth Control**

To manage bandwidth usage among users, set up **queues** to limit speeds for individual users or user groups.

### Step 1: Limit Bandwidth for a Single User
```bash
/queue simple add name="user1" target=192.168.88.10/32 max-limit=2M/2M
```

### Step 2: Limit Bandwidth for Multiple Users
```bash
/queue simple add name="LAN_Users" target=192.168.88.0/24 max-limit=10M/10M
```

---

## 8. **Remote Management**

Secure remote access is crucial for managing the router when you're not on the local network. Use **SSH** or configure a **VPN** for secure access.

### Step 1: Enable SSH Access
```bash
/ip service set ssh port=22 address=<trusted_ip> disabled=no
```

### Step 2: Set Up L2TP VPN
You can set up a **L2TP VPN** for secure remote access:
```bash
/interface l2tp-server server set enabled=yes use-ipsec=yes ipsec-secret=<vpn_secret>
```

---

## 9. **System Monitoring**

MikroTik offers built-in tools for monitoring network traffic and resource usage.

### Step 1: Enable Traffic Graphing
```bash
/tool graphing interface add interface=all
/tool graphing queue add queue=all
```

### Step 2: Monitor System Resources
To print resource usage such as CPU and memory:
```bash
/system resource print
/system resource cpu print
```

---

## 10. **Backup and Security**

Regular backups and keeping the system secure are vital for long-term maintenance of your MikroTik router.

### Step 1: Backup Router Configuration
To back up your current configuration:
```bash
/system backup save name=ISP_backup
```

### Step 2: Keep RouterOS Updated
It’s important to keep RouterOS up to date for security and feature improvements:
```bash
/system package update check-for-updates
/system package update install
```

---

## 11. **Important Links and Resources**

Here are some useful links and resources to help you with MikroTik configurations:

- [MikroTik Official Documentation](https://wiki.mikrotik.com)
- [RouterOS Download Page](https://mikrotik.com/download)
- [MikroTik Forum](https://forum.mikrotik.com)
- [Winbox Download](https://mikrotik.com/download)
- [RouterOS Training](https://mikrotik.com/training)
- [Network Monitoring Tools](https://www.paessler.com/prtg)

---

## Final Considerations

This guide provides a complete configuration solution for MikroTik routers in an ISP environment, including **Hotspot**, **DHCP**, **VLANs**, **Firewall**, **Bandwidth Control**, and **Remote Management**. Each of these sections can be customized to suit your particular needs.

---

**Author**: Kamrul Hossain  
**License**: MIT License

---

### Notes:
- Replace any placeholder values (like `<vpn_secret>` or `<trusted_ip>`) with actual information.
- For larger ISPs or more

 advanced setups, consider adding features such as **BGP**, **MPLS**, or advanced routing protocols.

---

