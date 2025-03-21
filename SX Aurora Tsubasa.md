## SX Aurora Tsubasa 
Installing and using **SX Aurora Tsubasa** in a cluster requires a systematic approach, as it involves setting up both hardware and software components. Here’s how you can proceed:

---

## **Prerequisites**

1. **Hardware Requirements**  
   - **Vector Engine (VE)** cards installed in each node (usually in PCIe slots).  
   - **Vector Host (VH)** servers to manage VEs.  
   - **High-speed interconnect** like InfiniBand for cluster communication.  

2. **Software Requirements**  
   - NEC VE OS (Operating System)  
   - NEC Compilers (Fortran, C, C++)  
   - MPI for cluster communication  
   - NEC SDK and Libraries  
   - Linux OS (RHEL or CentOS recommended)  

---

## **Step 1: Hardware Installation**

1. **Install Vector Engine (VE)**  
    - Physically install the **SX Aurora Tsubasa VE** into the PCIe slot of the server.  
    - Ensure proper cooling and power supply.  

2. **Connect to Vector Host (VH)**  
    - Vector Host acts as a management node.  
    - Use SSH for remote access to VH.

3. **Set Up Cluster Network**  
    - Establish an **InfiniBand** or **Ethernet** network.  
    - Configure IP addresses and enable SSH access between nodes.  

---

## **Step 2: Install NEC VE OS and Drivers**

1. Download the NEC VE OS from the official NEC site.  
2. Install it on the Vector Host using:  
    ```bash
    sudo yum install veos-release
    sudo yum install veos-kernel
    ```
3. Install the necessary drivers:  
    ```bash
    sudo yum install nec-ve-driver
    ```
4. Verify Vector Engine presence:  
    ```bash
    /opt/nec/ve/bin/veinfo
    ```

---

## **Step 3: Install SDK and Development Tools**

1. Install the **NEC SDK** for compiling and executing applications:  
    ```bash
    sudo yum install nec-ve-sdk
    ```
2. Install compilers:  
    ```bash
    sudo yum install nec-cc nec-fc
    ```
3. Install MPI for distributed computing:  
    ```bash
    sudo yum install nec-mpi
    ```

---

## **Step 4: Configure the Cluster**

1. **Set Up Cluster Nodes**  
    - Configure each node with its IP address and hostname.  
    - Ensure nodes can communicate using SSH without a password.  
    ```bash
    ssh-keygen -t rsa
    ssh-copy-id user@nodeX
    ```

2. **Configure MPI**  
    - Create a `hostfile` listing all the nodes:  
    ```bash
    node1 slots=4
    node2 slots=4
    ```
3. **Test MPI Communication**  
    ```bash
    mpirun -np 8 -hostfile hostfile hostname
    ```

---

##  **Step 5: Compile and Run OpenCL Programs**

1. **Write an OpenCL Program**  
    Save a sample kernel code (`vector_add.cl`):
    ```c
    __kernel void vector_add(__global float* a, __global float* b, __global float* c, int n) {
        int id = get_global_id(0);
        if (id < n) {
            c[id] = a[id] + b[id];
        }
    }
    ```
2. **Compile the Program**  
    ```bash
    nec_cc -o vector_add vector_add.c
    ```
3. **Run the Program**  
    ```bash
    ./vector_add
    ```

4. **Run Using MPI Across Nodes**  
    ```bash
    mpirun -np 8 -hostfile hostfile ./vector_add
    ```

---

## **Monitoring and Debugging**

- Use `veinfo` to check the status of the Vector Engine.  
- Track performance using NEC profiling tools:  
    ```bash
    veperf stat ./vector_add
    ```
- Log errors using:  
    ```bash
    dmesg | grep ve
    ```

---

## **Conclusion**  
You’re now ready to install, configure, and run applications on an SX Aurora Tsubasa cluster. Let me know if you'd like further assistance on specific configurations or troubleshooting!
