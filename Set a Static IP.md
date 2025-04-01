### Set a Static IP on Windows:

1. **Open Network Settings**:
   - Press `Win + R`, type `ncpa.cpl`, and hit **Enter**.
   
2. **Select Network Adapter**:
   - Right-click on your network adapter (Ethernet or Wi-Fi) and click **Properties**.

3. **Configure TCP/IPv4**:
   - Click on **Internet Protocol Version 4 (TCP/IPv4)** → **Properties**.
   - Select **Use the following IP address**.

4. **Enter IP Settings**:
   - **IP Address**: `192.168.1.X` (e.g., `192.168.1.10`)
   - **Subnet Mask**: `255.255.255.0`
   - **Default Gateway**: `192.168.1.1` (router IP)

5. **Optional DNS**:
   - **Preferred DNS Server**: `8.8.8.8` (Google DNS)

6. **Save and Apply**:
   - Click **OK** → **Apply**.

---

###  Static IP on Linux (Ubuntu Example

1. **Edit Network Settings**:
   - Open terminal and type:  
     ```bash
     sudo nano /etc/netplan/00-installer-config.yaml
     ```

2. **Configure Static IP**:
   - Add or modify the following lines (adjust IP addresses as needed):
     ```yaml
     network:
       version: 2
       renderer: networkd
       ethernets:
         eth0:
           dhcp4: no
           addresses:
             - 192.168.1.X/24
           gateway4: 192.168.1.1
           nameservers:
             addresses:
               - 8.8.8.8
               - 8.8.4.4
     ```

3. **Apply Changes**:
   - Run the following command to apply:
     ```bash
     sudo netplan apply
     ```


---

