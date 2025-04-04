
# SMB Troubleshooting 


## 1. Error: `0x800704f8`  
**Cause:** Guest access is blocked by Windows security policy.  
**Fix:** Enable insecure guest login

### Steps:
- Open `gpedit.msc` via Run (`Win + R`)
- Navigate:  
  `Computer Configuration > Administrative Templates > Network > Lanman Workstation`
- Enable: `Enable insecure guest logons`
- Click Apply > OK > Restart

**Alternative (Windows Home):**
- Open `regedit` via Run
- Navigate:  
  `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\LanmanWorkstation\Parameters`
- Add DWORD (32-bit) value: `AllowInsecureGuestAuth = 1`
- Restart the system

---

## 2. Error: `\\<ip>\<share>` Not Accessible  
**Cause:** Incorrect share path, SMB disabled, or firewall is blocking SMB ports  
**Fix:** Validate configuration and services

### Steps:
- Confirm the path: `\\10.0.0.1\shared`
- Check Samba service:
  ```bash
  sudo systemctl status smbd
  ```
- Open SMB ports in firewall:
  ```bash
  sudo ufw allow 137,138,139,445/tcp
  ```

---

## 3. Error: `"Access Denied"`  
**Cause:** Wrong credentials or folder permission issues  
**Fix:** Set proper user permissions

### Steps:
- Add Samba user:
  ```bash
  sudo smbpasswd -a <username>
  ```
- Set Linux folder permissions:
  ```bash
  sudo chown -R <user>:<group> /path/to/share
  sudo chmod -R 755 /path/to/share
  ```
- Configure `smb.conf`:
  ```ini
  [shared]
  path = /path/to/share
  valid users = <username>
  ```

---

## 4. Shared Folder Not Visible in Network  
**Cause:** `browseable = no` in Samba config  
**Fix:** Enable browseable setting

### Steps:
- Edit `/etc/samba/smb.conf`:
  ```ini
  [shared]
  path = /path/to/share
  browseable = yes
  ```
- Restart Samba:
  ```bash
  sudo systemctl restart smbd
  ```

---

## 5. Can't Mount Share on Linux  
**Cause:** Missing `cifs-utils` package  
**Fix:** Install required package

### Steps:
```bash
sudo apt update
sudo apt install cifs-utils
```

Then mount:
```bash
sudo mount -t cifs //192.168.1.100/shared /mnt/share -o username=<user>,password=<pass>
```

---

## 6. Error: Write Not Allowed  
**Cause:** File system is read-only (NTFS/ext4) or root-owned  
**Fix:** Change ownership and permissions

### Steps:
```bash
sudo chown -R nobody:nogroup /path/to/share
sudo chmod -R 777 /path/to/share
```

If NTFS:
```bash
sudo mount -o remount,rw /dev/sdX /mountpoint
```

---

## 7. Error: `Network Path Not Found`  
**Cause:** SMB ports are blocked or Samba service is not active  
**Fix:** Allow SMB ports and restart services

### Steps:
- Open ports:
  ```bash
  sudo ufw allow 137,138,139,445/tcp
  ```
- Start services:
  ```bash
  sudo systemctl enable --now smbd nmbd
  ```

---

```

