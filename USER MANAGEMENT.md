
## USER MANAGEMENT

---

### **CentOS / Rocky Linux: Create and Delete User**

#### **Creating a User**
1. **Log in as root or use `sudo`**  
   You need administrative privileges.

2. **Create the user**  
   ```bash
   sudo useradd username
   ```
   Replace `username` with the desired user name.

3. **Set the user's password**  
   ```bash
   sudo passwd username
   ```
   Enter the password when prompted.

4. **Add the user to a group (optional)**  
   To give the user additional permissions (e.g., `wheel` group for sudo access):
   ```bash
   sudo usermod -aG groupname username
   ```
   Replace `groupname` (e.g., `wheel`) and `username` with the actual names.

---

#### **Deleting a User**
1. **Delete the user (without removing home directory)**  
   ```bash
   sudo userdel username
   ```

2. **Delete the user and remove their home directory**  
   ```bash
   sudo userdel -r username
   ```

3. **Verify the user has been deleted**  
   ```bash
   cat /etc/passwd
   ```

4. **Remove the group (if necessary)**  
   If the user was the only one in the group and you want to remove the group:
   ```bash
   sudo groupdel groupname
   ```

5. **Check if the user is logged in and kill active processes (optional)**  
   ```bash
   ps -u username
   sudo pkill -u username
   ```

---

### **Ubuntu: Create and Delete User**

#### **Creating a User**
1. **Log in as root or use `sudo`**  
   You need administrative privileges.

2. **Create the user**  
   ```bash
   sudo adduser username
   ```
   Replace `username` with the desired user name. This command will also ask for details such as the password.

3. **Add the user to a group (optional)**  
   To give the user additional permissions (e.g., `sudo` group for administrative access):
   ```bash
   sudo usermod -aG sudo username
   ```

---

#### **Deleting a User**
1. **Delete the user (without removing home directory)**  
   ```bash
   sudo deluser username
   ```

2. **Delete the user and remove their home directory**  
   ```bash
   sudo deluser --remove-home username
   ```

3. **Verify the user has been deleted**  
   ```bash
   cat /etc/passwd
   ```

4. **Remove the group (if necessary)**  
   If the user was the only one in the group and you want to remove the group:
   ```bash
   sudo delgroup groupname
   ```

5. **Check if the user is logged in and kill active processes (optional)**  
   ```bash
   ps -u username
   sudo pkill -u username
   ```

---

### **General Notes for Both Systems**
- **Check if a user exists**  
  You can check if a user exists on the system by viewing `/etc/passwd`:
  ```bash
  cat /etc/passwd
  ```

- **Important Considerations**  
  - Deleting a user doesn’t automatically remove the associated group (unless specified).
  - If the user is currently logged in, kill their processes before deletion if necessary.
  
---
