

#### **1. Install NFS Packages**
On the **server-side (Server B)**, install the necessary NFS packages:

```bash
sudo yum install nfs-utils -y  # For CentOS/RHEL
sudo apt install nfs-kernel-server -y  # For Ubuntu/Debian

sudo zypper install nfs-kernel-server    # for op_slag
```

#### **2. Start and Enable NFS Services**
Enable and start the NFS services:

```bash
sudo systemctl enable nfs-server
sudo systemctl start nfs-server

sudo systemctl start rpcbind
sudo systemctl start nfs-idmap
```

#### **3. Create the Shared Directory**
Create a directory that you want to share:

```bash
sudo mkdir -p /server/apps
sudo chmod -R 777 /server/apps  # Give full permissions (for testing purposes)
sudo chown -R nobody:nogroup /server/apps  # Set ownership
```

#### **4. Configure NFS Exports**
Edit the **/etc/exports** file to define which clients can access the directory:

```bash
sudo nano /etc/exports
```

Add the following line:

```
/server/apps 192.168.1.0/24(rw,sync,no_root_squash)
```

- **`/server/apps`** – Directory to share.
- **`192.168.1.0/24`** – Allows all machines in the subnet to access.
- **`rw`** – Read/write permissions.
- **`sync`** – Ensures data is written to disk before confirming write success.
- **`no_root_squash`** – Allows root users from the client to access files as root.

#### **5. Apply the NFS Configuration**
Run:

```bash
sudo exportfs -rav
```

Restart the NFS service:

```bash
sudo systemctl restart nfs-server
```

---

### Firewall
```
sudo firewall-cmd --permanent --add-service=nfs
sudo firewall-cmd --permanent --add-service=rpc-bind
sudo firewall-cmd --permanent --add-service=mountd
sudo firewall-cmd --reload
```


### **Client-Side Configuration**
On **Server A (Client machine)**:

1. **Install NFS client packages:**
   ```bash
   sudo yum install nfs-utils -y  # CentOS/RHEL
   sudo apt install nfs-common -y  # Ubuntu/Debian


   or 

    sudo zypper install nfs-client

   ```

2. **Create a directory to mount the shared folder:**
   ```bash
   sudo mkdir -p /mnt/nfs_apps
   ```

3. **Mount the NFS share:**
   ```bash
   sudo mount 192.168.1.100:/server/apps /mnt/nfs_apps
   ```

   (Replace `192.168.1.100` with the actual NFS server IP.)

4. **Verify that the mount is successful:**
   ```bash
   df -h | grep nfs

   ls /mnt/nfs_share
   ```

5. **Make the mount persistent (optional)**
   To make the NFS mount persistent after reboot, add this line to **/etc/fstab**:

   ```
   sudo nano /etc/fstab
   
   192.168.1.100:/server/apps /mnt/nfs_apps nfs defaults 0 0
   ```

---

### **Testing**
- Try creating a file from the client:
  ```bash
  touch /mnt/nfs_apps/testfile.txt
  ```
- Check if the file appears on the server:
  ```bash
  ls /server/apps/
  ```

If the file appears on both machines, the NFS setup is working correctly!

---




## CentOS/RHEL (Server & Client Firewall Configuration)

### On the NFS Server

#### Allow NFS Services in Firewalld
```bash
sudo firewall-cmd --permanent --add-service=nfs
sudo firewall-cmd --permanent --add-service=rpc-bind
sudo firewall-cmd --permanent --add-service=mountd
```

#### Set Static Ports for NFS Services
Open the NFS configuration file:
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

#### Restart NFS Service
```bash
sudo systemctl restart nfs-server
sudo exportfs -rav
```

#### Allow Static Ports in Firewalld
```bash
sudo firewall-cmd --permanent --add-port=20048/tcp
sudo firewall-cmd --permanent --add-port=32765/tcp
sudo firewall-cmd --permanent --add-port=32767/tcp
sudo firewall-cmd --reload
```

### On the NFS Client (CentOS/RHEL)

#### Allow NFS Traffic on the Client Side
```bash
sudo firewall-cmd --permanent --add-service=nfs
sudo firewall-cmd --permanent --add-service=rpc-bind
sudo firewall-cmd --permanent --add-service=mountd
sudo firewall-cmd --reload
```

#### Test Connectivity from Client
```bash
showmount -e <NFS-Server-IP>
```

---

## Ubuntu/Debian (Server & Client Firewall Configuration)

### On the NFS Server

#### Allow NFS and RPC Services in UFW (Uncomplicated Firewall)
```bash
sudo ufw allow 2049/tcp
sudo ufw allow 111/tcp
sudo ufw allow 20048/tcp
sudo ufw allow 32765/tcp
sudo ufw allow 32767/tcp
```

#### Enable UFW
```bash
sudo ufw enable
```

#### Restart NFS Services
```bash
sudo systemctl restart nfs-kernel-server
sudo exportfs -rav
```

### On the NFS Client (Ubuntu/Debian)

#### Allow NFS Traffic on the Client Side
```bash
sudo ufw allow from <NFS-Server-IP> to any port nfs
sudo ufw reload
```

#### Test Connection
```bash
showmount -e <NFS-Server-IP>
