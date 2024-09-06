

# MikroTik ISP Router Full Configuration Guide (GUI and CLI)

This guide provides step-by-step instructions for setting up a MikroTik router in an ISP environment using both the **GUI (Winbox)** and **CLI (Command-Line Interface)**. It covers **basic internet setup**, **Hotspot creation**, **DHCP**, **firewall configuration**, **VLAN isolation**, and **bandwidth control**.

## Table of Contents
1. [Initial Setup (GUI and CLI)](#1-initial-setup)
2. [Basic Internet Configuration (GUI and CLI)](#2-basic-internet-configuration)
3. [Creating a Hotspot (GUI and CLI)](#3-creating-a-hotspot)
4. [NAT and Firewall Setup (GUI and CLI)](#4-nat-and-firewall-setup)
5. [Setting Up DHCP Server (GUI and CLI)](#5-setting-up-dhcp-server)
6. [VLAN and User Isolation (GUI and CLI)](#6-vlan-and-user-isolation)
7. [Bandwidth Control (GUI and CLI)](#7-bandwidth-control)
8. [Remote Management (GUI and CLI)](#8-remote-management)
9. [System Monitoring (GUI and CLI)](#9-system-monitoring)
10. [Backup and Security (GUI and CLI)](#10-backup-and-security)
11. [Important Links and Resources](#11-important-links-and-resources)

---

## 1. **Initial Setup (GUI and CLI)**

### GUI Mode (Winbox)

**Step 1: Connect to the MikroTik Router**
1. Open **Winbox** (you can download it [here](https://mikrotik.com/download)).
2. Under **Neighbors**, select the router's MAC address.
3. Click **Connect** (no password by default).

**Step 2: Change the Default Admin Password**
1. Go to **System** > **Users**.
2. Select the **admin** user and click **Set Password**.
3. Enter the new password and click **OK**.

### CLI Mode (SSH)
```bash
/user set 0 password=<new_secure_password>
```

---

## 2. **Basic Internet Configuration (GUI and CLI)**

### GUI Mode (Winbox)

**Step 1: Set the WAN Interface (Internet Connection)**
1. Go to **Interfaces**.
2. Double-click on **ether1** (or whichever is your WAN interface).
3. Change the name to **WAN**.
4. Go to **IP** > **DHCP Client** and click **Add**.
5. Set the **Interface** to **WAN** and click **OK**.

**Step 2: Set the LAN Interface (Internal Network)**
1. Go to **Interfaces** and double-click on **ether2**.
2. Change the name to **LAN**.
3. Go to **IP** > **Addresses** and click **Add**.
4. Set the **Address** to `192.168.88.1/24` and **Interface** to **LAN**. Click **OK**.

### CLI Mode (SSH)
```bash
/interface set ether1 name=WAN
/ip dhcp-client add interface=WAN disabled=no

/interface set ether2 name=LAN
/ip address add address=192.168.88.1/24 interface=LAN
```

---

## 3. **Creating a Hotspot (GUI and CLI)**

### GUI Mode (Winbox)

**Step 1: Setup Hotspot**
1. Go to **IP** > **Hotspot**.
2. Click on **Hotspot Setup**.
3. Select **LAN** (or the bridge) as the interface.
4. Follow the wizard and specify the IP address range, DNS name, and other options.

**Step 2: Add Hotspot Users**
1. Go to **IP** > **Hotspot** > **Users** tab.
2. Click **Add**.
3. Enter **Username** and **Password**, and click **OK**.

### CLI Mode (SSH)
```bash
/ip hotspot setup
/ip hotspot user add name=client1 password=clientpassword profile=default
```

---

## 4. **NAT and Firewall Setup (GUI and CLI)**

### GUI Mode (Winbox)

**Step 1: Enable NAT (Masquerading)**
1. Go to **IP** > **Firewall** > **NAT** tab.
2. Click **Add**.
3. Set **Chain** to `srcnat`, **Out. Interface** to `WAN`, and **Action** to `masquerade`. Click **OK**.

**Step 2: Add Firewall Rules**
1. Go to **IP** > **Firewall** > **Filter Rules** tab.
2. Click **Add**.
3. Set **Chain** to `input`, **Connection State** to `established, related`, and **Action** to `accept`. Click **OK**.
4. Add another rule with **Chain** set to `input`, **In. Interface** to `WAN`, and **Action** to `drop`. Click **OK**.

### CLI Mode (SSH)
```bash
/ip firewall nat add chain=srcnat action=masquerade out-interface=WAN
/ip firewall filter add chain=input action=accept connection-state=established,related
/ip firewall filter add chain=input action=drop in-interface=WAN
```

---

## 5. **Setting Up DHCP Server (GUI and CLI)**

### GUI Mode (Winbox)

**Step 1: Add DHCP Server**
1. Go to **IP** > **DHCP Server** > **Networks** tab and click **Add**.
2. Set the **Address** to `192.168.88.0/24`, **Gateway** to `192.168.88.1`, and click **OK**.

**Step 2: Add DHCP IP Pool**
1. Go to **IP** > **DHCP Server** > **IP Pools** tab.
2. Click **Add** and set the range to `192.168.88.10-192.168.88.254`. Click **OK**.

### CLI Mode (SSH)
```bash
/ip pool add name=dhcp_pool ranges=192.168.88.10-192.168.88.254
/ip dhcp-server add address-pool=dhcp_pool interface=LAN name=dhcp_server
/ip dhcp-server network add address=192.168.88.0/24 gateway=192.168.88.1
```

---

## 6. **VLAN and User Isolation (GUI and CLI)**

### GUI Mode (Winbox)

**Step 1: Create VLAN Interfaces**
1. Go to **Interfaces** > **VLAN**.
2. Click **Add** to create a new VLAN interface. Set **VLAN ID** to `10`, **Interface** to **LAN**, and click **OK**.
3. Repeat for VLAN 20.

**Step 2: Assign IP to VLANs**
1. Go to **IP** > **Addresses** and click **Add**.
2. Assign `192.168.10.1/24` to **vlan10** and `192.168.20.1/24` to **vlan20**.

**Step 3: Create Firewall Rules to Isolate VLANs**
1. Go to **IP** > **Firewall** > **Filter Rules**.
2. Click **Add** and set **In. Interface** to `vlan10`, **Out. Interface** to `vlan20`, and **Action** to `drop`. Repeat for the reverse rule.

### CLI Mode (SSH)
```bash
/interface vlan add name=vlan10 vlan-id=10 interface=LAN
/interface vlan add name=vlan20 vlan-id=20 interface=LAN
/ip address add address=192.168.10.1/24 interface=vlan10
/ip address add address=192.168.20.1/24 interface=vlan20
/ip firewall filter add chain=forward action=drop in-interface=vlan10 out-interface=vlan20
/ip firewall filter add chain=forward action=drop in-interface=vlan20 out-interface=vlan10
```

---

## 7. **Bandwidth Control (GUI and CLI)**

### GUI Mode (Winbox)

**Step 1: Add Bandwidth Limit (Queue)**
1. Go to **Queues**.
2. Click **Add** and set **Target** to `192.168.88.10` (or the IP range for multiple users).
3. Set the **Max Limit** to `2M/2M` for individual users or `10M/10M` for a group. Click **OK**.

### CLI Mode (SSH)
```bash
/queue simple add name="user1" target=192.168.88.10/32 max-limit=2M/2M
/queue simple add name="LAN_Users" target=192.168.88.0/24 max-limit=10M/10M
```

---

## 8. **Remote Management (GUI and CLI)**

### GUI Mode (Winbox)

**Step 1: Enable SSH Access**
1. Go to **IP** >

 **Services**.
2. Double-click on **SSH**, set the **Port** to `22`, and restrict the **Address** to trusted IPs only.

**Step 2: Set Up VPN**
1. Go to **PPP** > **L2TP Server**.
2. Enable **L2TP**, enable **IPsec**, and set the **IPsec Secret**. Click **OK**.

### CLI Mode (SSH)
```bash
/ip service set ssh port=22 address=<trusted_ip> disabled=no
/interface l2tp-server server set enabled=yes use-ipsec=yes ipsec-secret=<vpn_secret>
```

---

## 9. **System Monitoring (GUI and CLI)**

### GUI Mode (Winbox)

**Step 1: Enable Traffic Monitoring**
1. Go to **Tools** > **Graphing** > **Interface** tab.
2. Click **Add** and select **all** interfaces. Click **OK**.

**Step 2: Monitor System Resources**
1. Go to **System** > **Resources** to view CPU, memory, and other system metrics.

### CLI Mode (SSH)
```bash
/tool graphing interface add interface=all
/system resource print
```

---

## 10. **Backup and Security (GUI and CLI)**

### GUI Mode (Winbox)

**Step 1: Backup Configuration**
1. Go to **Files** and click **Backup**.
2. Name the backup and click **OK**.

**Step 2: Update RouterOS**
1. Go to **System** > **Packages**.
2. Click **Check for Updates** and install any available updates.

### CLI Mode (SSH)
```bash
/system backup save name=ISP_backup
/system package update check-for-updates
/system package update install
```

---

## 11. **Important Links and Resources**

- [MikroTik Official Documentation](https://wiki.mikrotik.com)
- [RouterOS Download Page](https://mikrotik.com/download)
- [MikroTik Forum](https://forum.mikrotik.com)
- [Winbox Download](https://mikrotik.com/download)
- [RouterOS Training](https://mikrotik.com/training)

---

## Final Considerations

This guide provides a comprehensive solution for setting up a MikroTik router in an ISP environment using both **GUI (Winbox)** and **CLI (SSH)** methods. Whether you prefer the command-line approach or the graphical interface, this guide has you covered for every step of the configuration process.

---

**Author**: Kamrul Hossain  
**License**: MIT License


