

# 🖨️ **Network Printer Access Control with MikroTik**

This project provides a solution for controlling access to network printers using MikroTik's firewall configuration. It ensures that only authorized devices can communicate with the printer, thereby enhancing network security.

---

## 📖 **Overview**

This project enhances security by allowing only pre-approved devices to access network printers, blocking all others. It uses MikroTik’s firewall features to create simple yet effective access control. Both command-line (CLI) and graphical (Winbox GUI) methods are provided for flexibility.

---

## ⚙️ **Features**

- 🔒 **IP Whitelisting:** Only specified devices are allowed to access the printer.
- 🚫 **Access Blocking:** Devices not on the whitelist are automatically blocked.
- 🖥️ **Dual Configuration:** Supports both CLI and GUI (Winbox) setup methods.
- 🛡️ **Enhanced Security:** Protects sensitive network resources by controlling access.
- 🔧 **Easily Scalable:** Simple to add or remove devices from the allowed list.

---

## 🛠️ **Installation Guide**

You can set up the firewall rules using either the **Command Line Interface (CLI)** or **Winbox GUI**. Follow the instructions below depending on your preferred method.

---

### **🔧 Command Line Mode (CLI)**

**Step 1: Add Devices to Allowed List**  
Add devices that should have access to the printer using this command:

```bash
/ip firewall address-list add address=<IP-of-device> list=allowed_to_printer
```
Replace `<IP-of-device>` with the IP address of the device you want to allow.

**Step 2: Allow Access to Printer**  
Create a rule to allow devices on the allowed list to access the printer:

```bash
/ip firewall filter add chain=forward src-address-list=allowed_to_printer dst-address=<IP-of-printer> action=accept
```
Replace `<IP-of-printer>` with the printer’s IP address.

**Step 3: Block All Other Devices**  
Create a rule to block any devices not on the allowed list from accessing the printer:

```bash
/ip firewall filter add chain=forward dst-address=<IP-of-printer> action=drop
```

**Step 4: Verify Firewall Rules**  
Ensure that the allow rule is placed before the block rule:

```bash
/ip firewall filter print
```

---

### **🖱️ GUI Mode (Winbox)**

**Step 1: Add Devices to Allowed List**  
1. Open **Winbox** and log in to your MikroTik router.
2. Go to **IP** > **Firewall** > **Address Lists**.
3. Click the **Add** button (`+`).
4. In the **Address** field, enter the IP address of the device.
5. In the **List** field, type `allowed_to_printer`.
6. Click **OK** to save the changes.

**Step 2: Create a Firewall Rule to Allow Access**  
1. In **Firewall**, go to **Filter Rules**.
2. Click **Add** (`+`).
3. Under **General**, set:
   - **Chain**: `forward`
   - **Src. Address List**: `allowed_to_printer`
   - **Dst. Address**: Enter the printer’s IP address.
4. Under **Action**, select `accept`.
5. Click **OK** to save the rule.

**Step 3: Block All Other Devices**  
1. In the **Filter Rules** tab, click **Add** (`+`).
2. Under **General**, set:
   - **Chain**: `forward`
   - **Dst. Address**: Enter the printer’s IP address.
3. In **Action**, select `drop`.
4. Click **OK** to save the rule.

**Step 4: Verify Firewall Rules**  
Ensure that the `accept` rule is positioned above the `drop` rule in the list of rules. You can drag the rules to reorder them.

---

## 🔧 **Configuration and Management**

- **Adding New Devices:**  
   To add a new device, repeat Step 1 in either CLI or GUI mode.

- **Removing Devices:**  
   - In the CLI, use the following command to remove a device from the allowed list:
   ```bash
   /ip firewall address-list remove [find list=allowed_to_printer address=<IP-of-device>]
   ```
   - In Winbox, go to **IP** > **Firewall** > **Address Lists**, locate the device, and click **Remove**.

- **Maintaining Security:**  
   Ensure that the `drop` rule for blocking unauthorized devices remains active at all times.

---

## 🔐 **Best Practices**

- **🔄 Regular Audits:** Periodically review the allowed list to ensure that only necessary devices have access.
- **💾 Backup:** Always back up your MikroTik configuration before making major changes to the firewall or address lists.
- **⚠️ Testing:** Test your setup by attempting to access the printer from both allowed and unauthorized devices to verify that the rules are functioning correctly.

---

## 🤝 **Contributing**

Contributions are welcome! If you'd like to contribute:

1. Fork the repository.
2. Create a new branch.
3. Make your changes and submit a pull request.
4. Ensure that your contributions follow the project's security and coding standards.

---

## 📜 **License**

This project is licensed under the MIT License. See the [LICENSE](LICENSE) file for full details.

---

## 👤 **Author & Contact**

Developed and maintained by **Kamrullab**.

### **Connect with me:**

- **GitHub**: [Kamrullab](https://github.com/Kamrullab)  
- **Facebook**: [elitekamrul](https://facebook.com/elitekamrul)

Feel free to reach out for more projects, contributions, or networking opportunities.
