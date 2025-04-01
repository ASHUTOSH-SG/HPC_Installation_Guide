

#### **1. Install NFS Packages**
On the **server-side (Server B)**, install the necessary NFS packages:

```bash
sudo yum install nfs-utils -y  # For CentOS/RHEL
sudo apt install nfs-kernel-server -y  # For Ubuntu/Debian
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

### **Client-Side Configuration**
On **Server A (Client machine)**:

1. **Install NFS client packages:**
   ```bash
   sudo yum install nfs-utils -y  # CentOS/RHEL
   sudo apt install nfs-common -y  # Ubuntu/Debian
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
   ```

5. **Make the mount persistent (optional)**
   To make the NFS mount persistent after reboot, add this line to **/etc/fstab**:

   ```
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

