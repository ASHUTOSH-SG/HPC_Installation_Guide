
# Guide: File Systems for HPC, AI, and Cloud (Ubuntu & CentOS)

## 1. Lustre File System

### When to Use:
Use Lustre for high-performance computing (HPC) and AI workloads requiring fast, parallel access to large datasets, such as genome sequencing or model training.

### Advantages:
- Extremely high throughput
- Parallel I/O access for large files
- Scales to thousands of nodes and petabytes of storage

### Limitations:
- Complex setup and tuning
- Requires compatible kernel modules

### Installation (Client-Side):

**Ubuntu:**
```bash
sudo apt update
sudo apt install lustre-client-modules lustre-client-utils
```

**CentOS:**
```bash
sudo yum install epel-release
sudo yum install lustre-client
```

### Mount Example:
```bash
mount -t lustre 192.168.1.100@tcp:/lustrefs /mnt/lustre
```

### Common Lustre Commands (with Purpose):
1. `lfs df -h`: Display disk space usage for Lustre file systems.
2. `lfs find /mnt/lustre -type f`: Search for all files in a Lustre mount point.
3. `lfs getstripe <file>`: Retrieve stripe configuration for a file.
4. `lfs setstripe -c 4 <dir>`: Set a stripe count of 4 for improved parallel I/O.
5. `lfs quota -u <user> /mnt/lustre`: Check disk quota for a specific user.
6. `lfs check /mnt/lustre`: Check the health status of the file system.
7. `lfs osts /mnt/lustre`: List available Object Storage Targets (OSTs).
8. `lctl get_param -n version`: Show the currently running Lustre version.
9. `lctl dl`: Display loaded Lustre kernel modules.
10. `lfs hsm_state <file>`: Check Hierarchical Storage Management (HSM) state of a file.
11. `lfs lock_show <file>`: Display active file locks.
12. `lfs mirror extend <file>`: Extend a mirrored file to new OSTs.
13. `lfs mirror resync <file>`: Resynchronize mirrored file data.
14. `lfs setstripe -i <index> <file>`: Assign file striping to a specific OST index.
15. `lfs find -size +100M`: Find files larger than 100MB.
16. `lfs filelayout <file>`: Get file layout including RAID and stripe info.
17. `lctl list_param`: List all tunable Lustre parameters.
18. `lctl set_param`: Set or change runtime parameters.
19. `lfs changelogs`: Monitor changes in metadata operations.
20. `lfs help`: Display a list of available Lustre commands.

---

## 2. BeeGFS File System

### When to Use:
Use BeeGFS in research labs or AI/ML workloads that involve many small files and require high-speed metadata operations.

### Advantages:
- Easy installation and management
- High metadata and data throughput
- Scalable on commodity hardware

### Limitations:
- Less widespread in large-scale HPC systems
- Smaller support community

### Installation (Client-Side):

**Ubuntu:**
```bash
wget -qO - https://www.beegfs.io/release/beegfs_7/dists/beegfs-deb.pub | sudo apt-key add -
echo "deb http://www.beegfs.io/release/beegfs_7/dists/beegfs-7.3.3/ubuntu focal non-free" | sudo tee /etc/apt/sources.list.d/beegfs.list
sudo apt update
sudo apt install beegfs-client beegfs-utils
```

**CentOS:**
```bash
sudo yum install https://www.beegfs.io/release/beegfs_7/dists/beegfs-rhel7.repo
sudo yum install beegfs-client beegfs-utils
```

### Mount Example:
```bash
mount -t beegfs 192.168.1.101:/ /mnt/beegfs
```

### Common BeeGFS Commands (with Purpose):
1. `beegfs-ctl --mounts`: Show all mounted BeeGFS file systems.
2. `beegfs-ctl --version`: Display the version of BeeGFS installed.
3. `beegfs-fsck`: Run file system check for consistency issues.
4. `beegfs-ctl --listnodes --nodetype=storage`: List all storage nodes.
5. `beegfs-ctl --removenode --nodetype=client --nodeid=1001`: Remove a specific client node.
6. `beegfs-ctl --addnodesfile=nodes.txt`: Add multiple nodes from a configuration file.
7. `beegfs-ctl --listtargets`: Show all available targets for storage.
8. `beegfs-netbench --numtargets=2 --loops=100`: Benchmark BeeGFS network performance.
9. `beegfs-ctl --getentryinfo <file>`: Retrieve entry information such as ID and stripe info.
10. `beegfs-ctl --createdir --path=/mnt/beegfs/data`: Create a directory within BeeGFS.
11. `beegfs-ctl --refreshstoragepools`: Refresh storage pool list.
12. `beegfs-ctl --storagepools`: List all configured storage pools.
13. `beegfs-ctl --setpattern --chunksize=1M`: Set stripe pattern chunk size.
14. `beegfs-ctl --readdir`: Read directory contents for debugging.
15. `beegfs-ctl --removetarget`: Remove a storage target.
16. `beegfs-ctl --addtarget`: Add a new storage target.
17. `beegfs-ctl --help`: Display all available command options.
18. `beegfs-fsck --dir=/mnt/beegfs`: Check a specific BeeGFS directory.
19. `beegfs-fsck --lostandfound`: Show files moved to lost+found.
20. `beegfs-fsck --inodesize`: Show inode usage stats.

---

## 3. CephFS File System

### When to Use:
CephFS is best for cloud-native platforms needing unified object, block, and file storage. Excellent for OpenStack or Kubernetes environments.

### Advantages:
- Unified storage (file, object, block)
- High availability and self-healing
- Compatible with OpenStack

### Limitations:
- Complex deployment
- Slightly lower raw performance for HPC

### Installation (Client-Side):

**Ubuntu/CentOS:**
```bash
sudo apt install ceph-deploy   # or sudo yum install ceph-deploy
```

### Setup:
```bash
ceph-deploy new mon1
ceph-deploy install mon1 osd1 osd2
ceph-deploy mon create-initial
ceph-deploy osd create osd1:/dev/sdb
ceph fs volume create cephfs
```

### Mount Example:
```bash
mount -t ceph 192.168.1.102:6789:/ /mnt/cephfs -o name=admin,secretfile=/etc/ceph/admin.keyring
```

### Common CephFS Commands (with Purpose):
1. `ceph -s`: Show overall health and status of the cluster.
2. `ceph df`: Display usage statistics for the cluster.
3. `ceph fs ls`: List available CephFS file systems.
4. `ceph fs volume create cephfs`: Create a new file system volume.
5. `ceph auth list`: List authentication keys and their capabilities.
6. `ceph osd tree`: Display object storage daemons in a tree format.
7. `ceph health`: Show current health status of the cluster.
8. `ceph osd status`: Display status of OSDs (up/down/in/out).
9. `ceph mds stat`: Show metadata server (MDS) status.
10. `ceph mon stat`: Display status of monitor nodes.
11. `ceph osd df`: Show disk usage by each OSD.
12. `ceph osd pool ls`: List all OSD pools.
13. `ceph fs status`: Display status of the CephFS.
14. `ceph mds dump`: Show dump of all MDS information.
15. `ceph osd crush rule dump`: Show CRUSH rule configuration.
16. `ceph fs snapshot create cephfs snap1`: Create a snapshot.
17. `ceph fs snapshot ls cephfs`: List snapshots.
18. `ceph fs snapshot rm cephfs snap1`: Remove a snapshot.
19. `ceph pg dump`: Display placement group details.
20. `ceph versions`: Show versions of all cluster components.
