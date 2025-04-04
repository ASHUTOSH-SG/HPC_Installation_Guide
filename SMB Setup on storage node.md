
### **SMB (Samba) Setup**

#### **Step 1: Install & Configure Samba on OpenSLAG Storage Node**

1. **Install Samba**  
   Since OpenSLAG is Linux-based, install Samba:

   ```bash
   sudo apt update && sudo apt install samba -y  # For Debian-based OS
   sudo yum install samba -y                      # For RHEL-based OS
   ```

2. **Create a Shared Directory**  
   ```bash
   sudo mkdir -p /storage/AIQT
   sudo chmod 777 /storage/AIQT  # Give full permissions for now (Adjust as needed)
   ```

3. **Edit Samba Configuration File**  
   Edit the `smb.conf` file:

   ```bash
   sudo nano /etc/samba/smb.conf
   ```

   Add the following configuration at the end of the file:

   ```ini
   [AIQT]
      path = /storage/AIQT
      browsable = yes
      writable = yes
      guest ok = yes
      create mask = 0777
      directory mask = 0777
      force user = nobody
   ```

4. **Restart Samba Service**  
   ```bash
   sudo systemctl restart smbd
   sudo systemctl enable smbd
   ```

5. **Allow SMB in Firewall (If Enabled)**  
   ```bash
   sudo ufw allow samba                      # For Ubuntu/Debian
   sudo firewall-cmd --permanent --add-service=samba  # For RHEL/CentOS
   sudo firewall-cmd --reload
   ```

---

#### **Step 2: Access Shared Folder from Windows Clients**

1. **Connect Using File Explorer**  
   Press `Win + R`, then type:

   ```
   \\storage_node_IP\AIQT
   ```

   *(Replace `storage_node_IP` with your actual server IP.)*

   If prompted for credentials:
   - **Username:** guest  
   - **Password:** (Leave blank, as we enabled guest access.)

2. **Map as a Network Drive (Persistent Access)**  
   - Open File Explorer → Right-click **This PC** → **Map Network Drive**.  
   - Choose a drive letter (e.g., `Z:`).  
   - Enter:  
     ```
     \\storage_node_IP\AIQT
     ```  
   - Check **Reconnect at sign-in** → Click **Finish**.

---

#### **Step 3: Access Shared Folder from Linux Clients**

1. **Install CIFS Utilities**  
   On each Linux client, install CIFS (Samba client package):

   ```bash
   sudo apt install cifs-utils -y  # Ubuntu/Debian
   sudo yum install cifs-utils -y  # RHEL/CentOS
   ```

2. **Mount the SMB Share**  
   ```bash
   sudo mount -t cifs //storage_node_IP/AIQT /mnt -o guest,uid=$(id -u),gid=$(id -g)
   ```

   *(This mounts the shared folder at `/mnt`.)*

3. **Make the Mount Persistent**  
   Edit the `/etc/fstab` file:

   ```bash
   sudo nano /etc/fstab
   ```

   Add this line:

   ```
   //storage_node_IP/AIQT  /mnt  cifs  guest,uid=1000,gid=1000  0  0
   ```

   Save and exit. Now, the shared folder will mount automatically on reboot.

---

#### **Step 4: Verify & Test**

- **Windows Clients:** Open `\\storage_node_IP\AIQT` in File Explorer.  
- **Linux Clients:** Check `/mnt` for access.

---

#### **Optional: Password-Protect the SMB Share**

If you want to secure the shared folder, follow these steps:

1. **Create a Samba User:**  
   ```bash
   sudo smbpasswd -a username
   ```

2. **Update `smb.conf`:**  
   ```ini
   [AIQT]
      path = /storage/AIQT
      browsable = yes
      writable = yes
      valid users = username
      create mask = 0777
      directory mask = 0777
      force user = username
   ```

3. **Restart Samba:**  
   ```bash
   sudo systemctl restart smbd
   ```

---

Let me know if you'd like further tweaks! 🚀
