

##  Prepare All Nodes

### Common for All (Ubuntu, CentOS, Rocky)

#### 1. Set Hostnames
Set hostname on each machine:

- Ubuntu (Controller):
  ```bash
  sudo hostnamectl set-hostname controller
  ```

- CentOS:
  ```bash
  sudo hostnamectl set-hostname compute1
  ```

- Rocky:
  ```bash
  sudo hostnamectl set-hostname compute2
  ```

> Also update `/etc/hosts` on **all nodes** to include:
```text
192.168.1.10 controller
192.168.1.11 compute1
192.168.1.12 compute2
```

---

## Networking & SSH

- Set **static IPs** for each node.
- From controller, test SSH:
  ```bash
  ssh root@compute1
  ssh root@compute2
  ```
- If SSH isn’t installed:
  ```bash
  sudo apt install openssh-server      # Ubuntu
  sudo yum install openssh-server      # CentOS/Rocky
  ```

---

##  Install and Configure MUNGE

### On All Nodes:

#### 1. Install MUNGE
- **Ubuntu (Controller):**
  ```bash
  sudo apt install munge libmunge-dev
  ```

- **CentOS/Rocky:**
  ```bash
  sudo yum install epel-release
  sudo yum install munge munge-libs munge-devel
  ```

#### 2. On Controller (Ubuntu): Create Key
```bash
sudo create-munge-key
sudo chown munge: /etc/munge/munge.key
sudo chmod 400 /etc/munge/munge.key
```

#### 3. Copy MUNGE Key to Compute Nodes
```bash
scp /etc/munge/munge.key root@compute1:/etc/munge/
scp /etc/munge/munge.key root@compute2:/etc/munge/
```

#### 4. Set Permissions (All Nodes)
```bash
sudo chown -R munge: /etc/munge /var/log/munge
sudo chmod 0700 /etc/munge /var/log/munge
sudo systemctl enable munge
sudo systemctl start munge
sudo systemctl status munge
```

---

## Install SLURM

### On Controller (Ubuntu)

#### 1. Install Dependencies
```bash
sudo apt install -y build-essential munge libmunge-dev libmunge2 \
libmysqlclient-dev libssl-dev libpam0g-dev libnuma-dev perl
```

#### 2. Download & Build SLURM
```bash
wget https://download.schedmd.com/slurm/slurm-21.08.8.tar.bz2
tar -xvjf slurm-21.08.8.tar.bz2
cd slurm-21.08.8
./configure --prefix=/opt/slurm
make
sudo make install
```

#### 3. Create Config Directories
```bash
sudo mkdir /etc/slurm /etc/slurm-llnl /var/spool/slurmctld /var/log/slurm
sudo touch /etc/slurm/slurm.conf
```

#### 4. Generate slurm.conf
Use the official [SLURM Config Generator](https://slurm.schedmd.com/configurator.html)  
Or copy this basic one:

```bash
cat <<EOF | sudo tee /etc/slurm/slurm.conf
ClusterName=cluster
SlurmctldHost=controller
AuthType=auth/munge
SlurmUser=root
AccountingStorageType=accounting_storage/none
NodeName=compute1 CPUs=1 State=UNKNOWN
NodeName=compute2 CPUs=1 State=UNKNOWN
PartitionName=debug Nodes=ALL Default=YES MaxTime=INFINITE State=UP
EOF
```

#### 5. Set environment (add in `/etc/bash.bashrc`)
```bash
export PATH="/opt/slurm/bin:/opt/slurm/sbin:$PATH"
export LD_LIBRARY_PATH="/opt/slurm/lib:$LD_LIBRARY_PATH"
```

```bash
source /etc/bash.bashrc
```

---

### On Compute Nodes (CentOS and Rocky)

#### 1. Install Dependencies
```bash
sudo yum groupinstall "Development Tools"
sudo yum install munge munge-devel numactl numactl-devel pam-devel perl \
openssl-devel
```

#### 2. Copy and Build SLURM (Same as controller)
You can copy the tar.bz2 file or download again:
```bash
wget https://download.schedmd.com/slurm/slurm-21.08.8.tar.bz2
tar -xvjf slurm-21.08.8.tar.bz2
cd slurm-21.08.8
./configure --prefix=/opt/slurm
make
sudo make install
```

#### 3. Create config folders
```bash
sudo mkdir /etc/slurm /etc/slurm-llnl /var/spool/slurmd /var/log/slurm
```

#### 4. Copy `slurm.conf` from controller
```bash
scp root@controller:/etc/slurm/slurm.conf /etc/slurm/
cp /etc/slurm/slurm.conf /etc/slurm-llnl/
```

#### 5. Set Environment (like controller)
```bash
echo 'export PATH="/opt/slurm/bin:/opt/slurm/sbin:$PATH"' >> ~/.bashrc
echo 'export LD_LIBRARY_PATH="/opt/slurm/lib:$LD_LIBRARY_PATH"' >> ~/.bashrc
source ~/.bashrc
```

---

## Start Services

### On Controller
```bash
sudo systemctl start munge
sudo slurmctld -Dvvv     # Start SLURM Controller in debug (or use `systemctl`)
```

### On Compute Nodes
```bash
sudo systemctl start munge
sudo slurmd -Dvvv        # Start SLURM daemon in debug
```

---

##  Test the Setup

From Controller, run:
```bash
scontrol show nodes
```

You should see both compute nodes listed.  
To test scheduling:
```bash
srun -N1 hostname
```

---

##  Optional: Disable Firewall Temporarily (for testing)
```bash
sudo systemctl stop firewalld     # CentOS/Rocky
sudo ufw disable                  # Ubuntu
```
