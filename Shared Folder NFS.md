

# Shared Folder 

## 1.  NFS Setup (for Linux Clients)


### 1: Install NFS Server on the Storage Node


```bash
sudo apt update && sudo apt install nfs-kernel-server -y   # For Debian/Ubuntu
sudo yum install nfs-utils -y                               # For RHEL/CentOS
```

### 2: Create a Shared Directory
Create a folder on the storage node that will be shared:

```bash
sudo mkdir -p /mnt/shared_folder
sudo chmod -R 777 /mnt/shared_folder
```

You can also assign a specific user:

```bash
sudo chown -R nobody:nogroup /mnt/shared_folder
```

### 3: Configure the NFS Exports
Edit the `/etc/exports` file to define access permissions:

```bash
sudo nano /etc/exports
```

Add the following line:

```
/mnt/shared_folder 192.168.1.0/24(rw,sync,no_root_squash,no_subtree_check)
```

- `192.168.1.0/24` → Grants access to all 20 PCs in your lab.
- `rw` → Read/write access.
- `sync` → Ensures data is written to disk before acknowledging.
- `no_root_squash` → Allows root users to access files as root.

### 4: Restart the NFS Service
Apply the changes:

```bash
sudo exportfs -a
sudo systemctl restart nfs-kernel-server   # Ubuntu/Debian
sudo systemctl restart nfs-server          # RHEL/CentOS
```

### 5: Configure the Client PCs
On each of the **20 lab PCs**, install NFS utilities:

```bash
sudo apt install nfs-common -y    # Debian/Ubuntu
sudo yum install nfs-utils -y     # RHEL/CentOS
```

Then mount the shared folder:

```bash
sudo mount 192.168.1.100:/mnt/shared_folder /mnt/client_shared
```

(Replace `192.168.1.100` with your **storage node IP**)

To make the mount permanent, add this entry in `/etc/fstab`:

```
192.168.1.100:/mnt/shared_folder /mnt/client_shared nfs defaults 0 0
```

---

## 2. Alternative: SMB 


### 1: Install Samba on Storage Node

```bash
sudo apt install samba -y    # Debian/Ubuntu
sudo yum install samba -y    # RHEL/CentOS
```

### 2: Create a Shared Directory

```bash
sudo mkdir -p /mnt/shared_folder
sudo chmod -R 777 /mnt/shared_folder
sudo chown -R nobody:nogroup /mnt/shared_folder
```

### 3: Configure Samba
Edit the Samba configuration file:

```bash
sudo nano /etc/samba/smb.conf
```

Add the following at the end:

```
[SharedFolder]
    path = /mnt/shared_folder
    available = yes
    valid users = nobody
    read only = no
    browsable = yes
    public = yes
    writable = yes
    guest ok = yes
    create mask = 0777
    directory mask = 0777
```

### 4: Restart Samba

```bash
sudo systemctl restart smbd
sudo systemctl enable smbd
```

### 5: Access the Shared Folder from Windows
- Press **Win + R**, type `\\192.168.1.100\SharedFolder`, and press **Enter**.
- Or, map it as a network drive.