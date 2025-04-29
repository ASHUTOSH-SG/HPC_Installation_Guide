
# Slurm Cluster Setup with Rocky Linux Controller and Ubuntu/CentOS Compute Nodes



---

## GOAL

- **Controller (Master)**: Rocky Linux 8/9  
- **Compute Nodes**: Ubuntu 20.04+/22.04+ and CentOS 7/8  
- **Slurm Version**: Same on all nodes  
- **Authentication**: MUNGE  
- **Optional Shared Storage**: `/opt/slurm`, `/home`, or `/etc/slurm`

---

## STEP 1: Preparation (All Nodes)

### 1.1 Set Hostnames & IPs

Assign static IPs and hostnames:

```bash
# Example (on all nodes)
sudo hostnamectl set-hostname rocky-controller  # or ubuntu-node1, centos-node1, etc.
```

Add IPs and hostnames to `/etc/hosts` on all nodes:

```bash
# /etc/hosts
192.168.1.10 rocky-controller
192.168.1.11 ubuntu-node1
192.168.1.12 centos-node1
```

### 1.2 Disable Firewall & SELinux (or allow ports)

```bash
# Disable SELinux (Rocky/CentOS)
sudo setenforce 0
sudo sed -i 's/^SELINUX=enforcing/SELINUX=disabled/' /etc/selinux/config

# Stop and disable firewalld
sudo systemctl disable --now firewalld
```

### 1.3 Install NTP/Chrony (All nodes)

```bash
sudo dnf install -y chrony       # Rocky/CentOS
sudo apt install -y chrony       # Ubuntu

sudo systemctl enable --now chronyd
```

---

## STEP 2: Install and Configure MUNGE

MUNGE is used for authentication between nodes.

### 2.1 Install MUNGE (All Nodes)

#### Rocky/CentOS

```bash
sudo dnf install -y epel-release
sudo dnf install -y munge munge-libs munge-devel
```

#### Ubuntu

```bash
sudo apt update
sudo apt install -y munge libmunge-dev libmunge2
```

### 2.2 Create and Sync MUNGE Key (Controller)

```bash
sudo create-munge-key
sudo chown munge:munge /etc/munge/munge.key
sudo chmod 400 /etc/munge/munge.key
```

Copy the key to all compute nodes:

```bash
scp /etc/munge/munge.key user@ubuntu-node1:/tmp/
ssh ubuntu-node1 "sudo mv /tmp/munge.key /etc/munge/ && sudo chown munge:munge /etc/munge/munge.key && sudo chmod 400 /etc/munge/munge.key"
```

Repeat for all nodes.

### 2.3 Start MUNGE (All Nodes)

```bash
sudo systemctl enable --now munge
```

---

## STEP 3: Install Slurm (Build from Source)

Using `/opt/slurm` as the install prefix for consistency.

### 3.1 Prerequisites

#### Rocky/CentOS

```bash
sudo dnf groupinstall -y "Development Tools"
sudo dnf install -y munge-devel readline-devel openssl-devel pam-devel \
                   libibmad libibumad numactl-devel hwloc-devel
```

#### Ubuntu

```bash
sudo apt install -y build-essential libmunge-dev libssl-dev libpam0g-dev \
                    libhwloc-dev libnuma-dev libreadline-dev
```

### 3.2 Download and Build Slurm

On all nodes:

```bash
cd /usr/local/src
wget https://download.schedmd.com/slurm/slurm-23.11.5.tar.bz2
tar -xvjf slurm-23.11.5.tar.bz2
cd slurm-23.11.5

./configure --prefix=/opt/slurm
make -j$(nproc)
sudo make install
```

Set environment globally:

```bash
echo 'export PATH=/opt/slurm/bin:/opt/slurm/sbin:$PATH' | sudo tee /etc/profile.d/slurm.sh
```

---

## STEP 4: Slurm Configuration

### 4.1 Create Slurm User and Directories (All Nodes)

```bash
sudo useradd -m slurm
sudo mkdir -p /var/spool/slurmd /var/log/slurm /opt/slurm/etc
sudo chown -R slurm: /var/spool/slurmd /var/log/slurm /opt/slurm/etc
```

Controller only:

```bash
sudo mkdir -p /var/spool/slurmctld
sudo chown -R slurm: /var/spool/slurmctld
```

### 4.2 Generate slurm.conf (Controller Only)

Use Slurm’s [online configurator](https://slurm.schedmd.com/configurator.html), or use this example:

**/opt/slurm/etc/slurm.conf**

```ini
ClusterName=linuxcluster
ControlMachine=rocky-controller
SlurmUser=slurm
SlurmdUser=root
StateSaveLocation=/var/spool/slurmctld
SlurmdSpoolDir=/var/spool/slurmd
AuthType=auth/munge
CacheGroups=0
SlurmctldPort=6817
SlurmdPort=6818
SwitchType=switch/none
MpiDefault=none
SlurmctldPidFile=/var/run/slurmctld.pid
SlurmdPidFile=/var/run/slurmd.pid
ProctrackType=proctrack/pgid
ReturnToService=1
SchedulerType=sched/backfill
SelectType=select/cons_res
SelectTypeParameters=CR_Core
NodeName=ubuntu-node1 CPUs=4 State=UNKNOWN
NodeName=centos-node1 CPUs=4 State=UNKNOWN
PartitionName=debug Nodes=ALL Default=YES MaxTime=INFINITE State=UP
```

Distribute this file to all compute nodes:

```bash
scp /opt/slurm/etc/slurm.conf ubuntu-node1:/opt/slurm/etc/
```

---

## STEP 5: Start Slurm Daemons

### 5.1 Controller (Rocky)

```bash
sudo /opt/slurm/sbin/slurmctld -D
```

### 5.2 Compute Nodes (Ubuntu/CentOS)

```bash
sudo /opt/slurm/sbin/slurmd -D
```

---

## STEP 6: Test and Verify

On the controller:

```bash
sinfo
srun hostname
```

You should see compute node names and their states as `IDLE`.

---

## STEP 7: Create Systemd Units

### 7.1 Controller: slurmctld.service

Create `/etc/systemd/system/slurmctld.service`

```ini
[Unit]
Description=Slurm controller daemon
After=munge.service network.target

[Service]
Type=simple
User=slurm
ExecStart=/opt/slurm/sbin/slurmctld -D

[Install]
WantedBy=multi-user.target
```

### 7.2 Compute: slurmd.service

Create `/etc/systemd/system/slurmd.service`

```ini
[Unit]
Description=Slurm node daemon
After=munge.service network.target

[Service]
Type=simple
User=root
ExecStart=/opt/slurm/sbin/slurmd -D

[Install]
WantedBy=multi-user.target
```

Reload and enable services:

```bash
sudo systemctl daemon-reexec
sudo systemctl daemon-reload
sudo systemctl enable --now slurmctld  # On controller
sudo systemctl enable --now slurmd     # On compute nodes
```

---

## STEP 8: Troubleshooting Tips

| Problem                  | Solution                                                             |
|--------------------------|----------------------------------------------------------------------|
| Node `DOWN` in `sinfo`   | Check slurmd logs: `journalctl -u slurmd`                           |
| MUNGE authentication     | Ensure same munge.key and correct permissions on all nodes          |
| `srun` fails or hangs    | Check firewall, ports 6817/6818, and hostname resolution             |
| Slurm version mismatch   | Ensure same version on all nodes                                    |
| MUNGE not starting       | Check logs: `journalctl -u munge`                                   |

---

