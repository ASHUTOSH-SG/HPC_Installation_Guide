
# X11 Forwarding

## 1.  Setting a Static IP
CHECK STATIC IP

### **CentOS / Rocky Linux:**
1. **Edit the network configuration file**  
   For CentOS/Rocky Linux, you typically configure network settings in the `/etc/sysconfig/network-scripts/ifcfg-eth0` file (or the relevant network interface file).
   
   ```bash
   sudo nano /etc/sysconfig/network-scripts/ifcfg-eth0
   ```

   Example settings for static IP:
   ```ini
   DEVICE=eth0
   BOOTPROTO=none
   ONBOOT=yes
   IPADDR=192.168.1.100
   NETMASK=255.255.255.0
   GATEWAY=192.168.1.1
   DNS1=8.8.8.8
   DNS2=8.8.4.4
   ```

2. **Restart network service** to apply changes:
   ```bash
   sudo systemctl restart network
   ```

### **Ubuntu:**
1. **Edit netplan configuration**  
   Ubuntu uses `netplan` for network configuration. You need to modify the netplan config file.

   For example, modify the file at `/etc/netplan/00-installer-config.yaml`:
   
   ```bash
   sudo nano /etc/netplan/00-installer-config.yaml
   ```

   Example configuration for static IP:
   ```yaml
   network:
     version: 2
     renderer: networkd
     ethernets:
       eth0:
         dhcp4: false
         addresses:
           - 192.168.1.100/24
         gateway4: 192.168.1.1
         nameservers:
           addresses:
             - 8.8.8.8
             - 8.8.4.4
   ```

2. **Apply changes**:
   ```bash
   sudo netplan apply
   ```

---

## 2. Enable X11 Forwarding (if needed for GUI applications)
As X11 Forwarding is essential for graphical applications, follow the steps below for the server setup:

### **CentOS / Rocky Linux:**
1. **Ensure X11 Forwarding is enabled in SSH**:
   Edit the SSH configuration:
   ```bash
   sudo nano /etc/ssh/sshd_config
   ```
   
   Make sure these lines are present and uncommented:
   ```bash
   X11Forwarding yes
   X11DisplayOffset 10
   ```

2. **Install X11 packages** (if necessary):
   ```bash
   sudo yum install xorg-x11-xauth xorg-x11-utils xorg-x11-apps
   ```

3. **Restart SSH service**:
   ```bash
   sudo systemctl restart sshd
   ```

### **Ubuntu:**
1. **Ensure X11 Forwarding is enabled in SSH**:
   Edit the SSH configuration:
   ```bash
   sudo nano /etc/ssh/sshd_config
   ```
   
   Add or uncomment these lines:
   ```bash
   X11Forwarding yes
   X11DisplayOffset 10
   ```

2. **Install X11 packages**:
   ```bash
   sudo apt update
   sudo apt install xauth xorg
   ```

3. **Restart SSH service**:
   ```bash
   sudo systemctl restart ssh
   ```

---

## 3. Firewall Configuration
Ensure that the firewall on the machine allows SSH access (and other necessary ports if applicable).

### **CentOS / Rocky Linux:**
1. **Open SSH port (default is port 22)**:
   ```bash
   sudo firewall-cmd --zone=public --add-port=22/tcp --permanent
   sudo firewall-cmd --reload
   ```

### **Ubuntu:**
1. **Allow SSH through UFW**:
   ```bash
   sudo ufw allow ssh
   ```

2. **Verify UFW status**:
   ```bash
   sudo ufw status
   ```

---

## 4. User and Permissions Setup
If you haven’t already created users and granted appropriate permissions (e.g., for X11 forwarding, administrative tasks), do so before sharing the static IP.

### **Create a User (if needed):**
Create a new user if necessary:
```bash
sudo useradd username
sudo passwd username
```

To give the user administrative permissions:
```bash
sudo usermod -aG sudo username  # for Ubuntu
sudo usermod -aG wheel username # for CentOS/Rocky
```

---

## 5. SSH Access and Key Setup
s SSH key-based authentication for more secure access.

### **SSH Key Setup:**
1. **Generate SSH key pair on the client machine (if not already)**:
   ```bash
   ssh-keygen -t rsa -b 2048
   ```

2. **Copy the public key to the server**:
   ```bash
   ssh-copy-id username@static-ip
   ```

3. **Disable password authentication for more security** (optional, but recommended):
   Edit `/etc/ssh/sshd_config`:
   ```bash
   sudo nano /etc/ssh/sshd_config
   ```
   Set the following options:
   ```bash
   PasswordAuthentication no
   ChallengeResponseAuthentication no
   ```
   Restart SSH:
   ```bash
   sudo systemctl restart sshd
   ```

---

## 6. Test the SSH and X11 Forwarding Setup


1. **Test SSH access**:
   ```bash
   ssh -X username@static-ip  # Use -X for X11 forwarding
   ```

2. **Test X11 forwarding**:
   After logging in via SSH, try running a GUI-based application, e.g., `xclock` or `gedit`:
   ```bash
   xclock
   ```

---

## 7. Provide the Static IP and Access Instructions to the Team
Once the necessary configurations have been set up and verified, you can provide the following information to the team:

1. **Static IP** of the machine.
2. **SSH login details** (username and the static IP).
3. **X11 Forwarding instructions** (if they need to run graphical apps).
   - Example: `ssh -X username@static-ip`

---

##  Set Up a Monitoring Tool (if needed)
 to keep track of the system’s performance or potential issues (e.g., `htop`, `netstat`, or more advanced solutions like **Nagios**, **Zabbix**, or **Prometheus**), you can install monitoring tools.

### Example for `htop`:
```bash
sudo yum install htop  # CentOS/Rocky
sudo apt install htop  # Ubuntu
```

---

