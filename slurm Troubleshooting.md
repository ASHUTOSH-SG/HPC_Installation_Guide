

## BEFORE INSTALLATION Troubleshooting

###  1. Hostname and `/etc/hosts` Issues
- **Symptom:** `slurmd` or `slurmctld` cannot resolve node names.
- **Fix:**
  - Ensure each node has a **unique hostname** using:
    ```bash
    hostnamectl set-hostname <name>
    ```
  - Ensure `/etc/hosts` file is identical and contains:
    ```bash
    192.168.1.10 controller
    192.168.1.11 compute1
    192.168.1.12 compute2
    ```

###  2. Time Sync (IMPORTANT!)
- **Symptom:** MUNGE fails to authenticate, SLURM reports auth errors.
- **Fix:** Use `chrony` or `ntp` to sync time.
  ```bash
  sudo apt install chrony      # Ubuntu
  sudo yum install chrony      # CentOS/Rocky
  sudo systemctl enable --now chronyd
  ```

###  3. SSH Access Passwordless
- **Symptom:** Can't connect from controller to compute nodes.
- **Fix:** Set up SSH keys:
  ```bash
  ssh-keygen
  ssh-copy-id root@compute1
  ssh-copy-id root@compute2
  ```

###  4. Dependency Conflicts
- **Symptom:** `configure` script fails when building SLURM.
- **Fix:** Ensure all dev packages are installed:
  - Ubuntu:
    ```bash
    sudo apt install build-essential libmunge-dev libssl-dev ...
    ```
  - CentOS/Rocky:
    ```bash
    sudo yum groupinstall "Development Tools"
    sudo yum install munge-devel pam-devel openssl-devel ...
    ```

---

##  MUNGE Troubleshooting

### ❗ Issue 1: MUNGE Not Starting
- **Check:**
  ```bash
  sudo systemctl status munge
  journalctl -xeu munge
  ```
- **Fixes:**
  - Key permissions:
    ```bash
    sudo chown munge:munge /etc/munge/munge.key
    sudo chmod 400 /etc/munge/munge.key
    ```
  - Directory permissions:
    ```bash
    sudo chown -R munge:munge /var/log/munge /etc/munge
    sudo chmod 0700 /etc/munge /var/log/munge
    ```

### ❗ Issue 2: MUNGE Authentication Fails Across Nodes
- **Fix:**
  - Make sure `munge.key` is the **same on all nodes**.
  - Check time sync!
  - Check firewall between nodes (port 1111 for MUNGE by default).
  - Check logs:
    ```bash
    sudo tail -f /var/log/munge/munge.log
    ```

---

##  SLURM Build/Install Troubleshooting

###  Issue 1: `configure` script fails
- **Fix:**
  - Install missing `devel` packages for required libraries.
  - Read the `config.log` for missing dependencies.

###  Issue 2: Make fails
- **Fix:**
  - Ensure GCC is up to date and dev tools are installed.

###  Issue 3: Wrong install path
- **Fix:**
  - Always use `--prefix=/opt/slurm` or similar.
  - Set PATH and LD_LIBRARY_PATH correctly:
    ```bash
    export PATH="/opt/slurm/bin:/opt/slurm/sbin:$PATH"
    export LD_LIBRARY_PATH="/opt/slurm/lib:$LD_LIBRARY_PATH"
    ```

---

## SLURM Runtime Troubleshooting

### Issue 1: `slurmctld` fails to start
- **Check:**
  ```bash
  slurmctld -Dvvv
  ```
- **Common Causes:**
  - Invalid `slurm.conf`: Check with `slurmctld -t`
  - MUNGE not running
  - Controller hostname not resolvable
  - `/var/spool/slurmctld` missing or incorrect ownership:
    ```bash
    sudo mkdir -p /var/spool/slurmctld
    sudo chown slurm: /var/spool/slurmctld
    ```

### Issue 2: `slurmd` fails on compute node
- **Check:**
  ```bash
  slurmd -Dvvv
  ```
- **Common Fixes:**
  - Make sure compute node name matches what’s in `slurm.conf`
  - Check `munge` status
  - Ensure `/var/spool/slurmd` and `/var/log/slurm` exist and are writable.

### Issue 3: Node state shows as `DOWN` or `UNKNOWN`
- **Check:**
  ```bash
  scontrol show nodes
  ```
- **Fix:**
  - Restart `slurmd`
  - Check logs at `/var/log/slurm/slurmd.log`
  - Confirm controller can reach compute nodes via hostname

---

## Debugging SLURM

### Check Config File
```bash
slurmctld -t        # Validate config
```

### Check Services
```bash
systemctl status munge
ps aux | grep slurm
```

### Check Logs
- `slurmctld` log:
  ```bash
  /var/log/slurm/slurmctld.log
  ```
- `slurmd` log:
  ```bash
  /var/log/slurm/slurmd.log
  ```

---

## Post-Install Sanity Tests

### 1. Test Nodes from Controller
```bash
scontrol show nodes
```

###  2. Run Test Job
```bash
srun -N1 hostname
```

### 3. Run Scripted Batch Job
```bash
cat <<EOF > testjob.sh
#!/bin/bash
hostname
sleep 10
EOF

sbatch testjob.sh
```

---

## Firewall / SELinux Issues (Common on CentOS/Rocky)

- Disable for test (optional):
  ```bash
  sudo systemctl stop firewalld
  sudo setenforce 0
  ```

---

##  Clean SLURM Uninstall (if needed)
```bash
cd slurm-*
sudo make uninstall
sudo rm -rf /opt/slurm /etc/slurm* /var/log/slurm /var/spool/slurm*
```
