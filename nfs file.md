# NFS Server and Client Setup Guide

## 1. Install NFS Packages

### **On the NFS Server (Server B)**
Install the required NFS packages based on your OS:

```bash
# For CentOS/RHEL
sudo yum install nfs-utils -y  

# For Ubuntu/Debian
sudo apt install nfs-kernel-server -y  

# For OpenSUSE
sudo zypper install nfs-kernel-server  
```

## 2. Start and Enable NFS Services
Enable and start the necessary NFS services:

```bash
sudo systemctl enable nfs-server
sudo systemctl start nfs-server

sudo systemctl enable rpcbind
sudo systemctl start rpcbind

sudo systemctl enable nfs-idmap
sudo systemctl start nfs-idmap
```

## 3. Create the Shared Directory
Create a directory that will be shared:

```bash
sudo mkdir -p /server/AIQT
sudo chmod -R 777 /server/AIQT  # Full permissions (for testing purposes)
sudo chown -R nobody:nogroup /server/AIQT  # Set ownership
```

## 4. Configure NFS Exports
Edit the `/etc/exports` file to define the shared directory and allowed clients:

```bash
sudo nano /etc/exports
```

Add the following line:

```
/server/AIQT 192.168.1.0/24(rw,sync,no_root_squash)
```

- **`/server/AIQT`** – Directory to share.
- **`192.168.1.0/24`** – Allows all machines in the subnet to access.
- **`rw`** – Read/write permissions.
- **`sync`** – Ensures data is written to disk before confirming write success.
- **`no_root_squash`** – Allows root users from the client to access files as root.

## 5. Apply the NFS Configuration
Run:

```bash
sudo exportfs -rav
```

Restart the NFS service:

```bash
sudo systemctl restart nfs-server
```

## 6. Configure Firewall Rules

### **CentOS/RHEL Firewall Rules**
```bash
sudo firewall-cmd --permanent --add-service=nfs
sudo firewall-cmd --permanent --add-service=rpc-bind
sudo firewall-cmd --permanent --add-service=mountd
sudo firewall-cmd --reload
```

### **Ubuntu/Debian Firewall Rules (UFW)**
```bash
sudo ufw allow 111/tcp
sudo ufw allow 111/udp
sudo ufw allow 2049/tcp
sudo ufw allow 2049/udp
sudo ufw allow 20048/tcp
sudo ufw allow 20048/udp
sudo ufw enable
```

---

# Client-Side Configuration (Server A)

## 1. Install NFS Client Packages

```bash
# For CentOS/RHEL
sudo yum install nfs-utils -y  

# For Ubuntu/Debian
sudo apt install nfs-common -y  

# For OpenSUSE
sudo zypper install nfs-client
```

## 2. Create a Mount Directory

```bash
sudo mkdir -p /mnt/AIQT
```

## 3. Mount the NFS Share

```bash
sudo mount 192.168.1.100:/server/AIQT /mnt/AIQT
```

(Replace `192.168.1.100` with the actual NFS server IP.)

## 4. Verify the Mount

```bash
mount | grep nfs
ls /mnt/AIQT
```

## 5. Make the Mount Persistent
To make the NFS mount persistent after reboot, add this line to `/etc/fstab`:

```bash
sudo nano /etc/fstab
```

Add the following line:

```
192.168.1.100:/server/AIQT /mnt/AIQT nfs defaults 0 0
```

---

# **Testing the NFS Setup**

### **Test File Creation**
On the **client machine**:
```bash
touch /mnt/AIQT/testfile.txt
```

On the **server machine**:
```bash
ls /server/AIQT/
```

If the file appears on both machines, the NFS setup is working correctly!

---

# **Advanced NFS Configuration (CentOS/RHEL Only)**

### **Set Static Ports for NFS Services**
Edit the NFS configuration file:
```bash
sudo nano /etc/nfs.conf
```
Add the following lines:

```ini
[nfsd]
vers3=yes
vers4=yes

[mountd]
port=20048

[statd]
port=32765
outgoing-port=32766

[lockd]
port=32767
```

### **Restart NFS Services**
```bash
sudo systemctl restart nfs-config.service
sudo systemctl restart nfs-server
sudo exportfs -rav
```

### **Allow Static Ports in Firewalld**
```bash
sudo firewall-cmd --permanent --add-port=20048/tcp
sudo firewall-cmd --permanent --add-port=32765/tcp
sudo firewall-cmd --permanent --add-port=32767/tcp
sudo firewall-cmd --reload
```

### **Test NFS Connectivity from Client**
```bash
showmount -e 192.168.1.100
```

---

# **Ubuntu/Debian Advanced Configuration**

### **Restart NFS Services**
```bash
sudo systemctl restart nfs-kernel-server
sudo exportfs -rav
```

### **Allow NFS Traffic on Client Side (UFW)**
```bash
sudo ufw allow from 192.168.1.100 to any port nfs
sudo ufw reload
```

### **Test Connection**
```bash
showmount -e 192.168.1.100
```

