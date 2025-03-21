
# CUDA

- parallel computing platform 
- application  programming interface (API)
- harness the power of NVIDIA GPUs for general-purpose processing (GPGPU)

 ### CUDA Architecture
- Host: CPU (Central Processing Unit)
- Device: GPU (Graphics Processing Unit)
- CUDA uses thousands of GPU cores to perform parallel computations.

### key concepts
- Thread: Smallest unit of execution on a GPU.
- Block: Group of threads (e.g., 1024 threads per block max).
- Grid: Collection of blocks.
- Kernel: GPU function that runs in parallel on all threads.

### CUDA Programming Model
- CUDA uses a Single Instruction Multiple Thread (SIMT) model.
 - The same kernel function runs on multiple threads, but each thread has its own unique identifier.
- Thread Identification:
```
int idx = threadIdx.x;       // Thread ID within the block
int bidx = blockIdx.x;       // Block ID
int blockSize = blockDim.x;  // Number of threads per block
int globalIndex = bidx * blockSize + idx;
threadIdx, blockIdx, and blockDim are built-in CUDA variables
```

### Basic CUDA Program Structure
A CUDA program consists of:

Host Code - Runs on the CPU.
Device Code - Runs on the GPU.
Example: Vector Addition in CUDA

```
#include <stdio.h>

__global__ void addVectors(int *a, int *b, int *c, int n) {
    int i = blockIdx.x * blockDim.x + threadIdx.x;
    if (i < n) {
        c[i] = a[i] + b[i];
    }
}

int main() {
    int n = 1024;
    int size = n * sizeof(int);
    int *a, *b, *c;
    int *d_a, *d_b, *d_c;

    // Allocate memory on the host
    a = (int*)malloc(size);
    b = (int*)malloc(size);
    c = (int*)malloc(size);

    // Initialize vectors
    for (int i = 0; i < n; i++) {
        a[i] = i;
        b[i] = i;
    }

    // Allocate memory on the device
    cudaMalloc((void**)&d_a, size);
    cudaMalloc((void**)&d_b, size);
    cudaMalloc((void**)&d_c, size);

    // Copy data from host to device
    cudaMemcpy(d_a, a, size, cudaMemcpyHostToDevice);
    cudaMemcpy(d_b, b, size, cudaMemcpyHostToDevice);

    // Launch kernel
    int threadsPerBlock = 256;
    int blocksPerGrid = (n + threadsPerBlock - 1) / threadsPerBlock;
    addVectors<<<blocksPerGrid, threadsPerBlock>>>(d_a, d_b, d_c, n);

    // Copy result back to host
    cudaMemcpy(c, d_c, size, cudaMemcpyDeviceToHost);

    // Verify result
    for (int i = 0; i < n; i++) {
        if (c[i] != a[i] + b[i]) {
            printf("Error at index %d\n", i);
            return -1;
        }
    }
    printf("Vector addition successful!\n");

    // Free memory
    free(a); free(b); free(c);
    cudaFree(d_a); cudaFree(d_b); cudaFree(d_c);

    return 0;
}
```

### Memory Management in CUDA
- CUDA supports multiple types of memory:

- Global Memory - Accessible by all threads. High latency.
- Shared Memory - Fast memory shared within a block.
- Constant Memory - - Read-only memory optimized for constants.
- Texture and Surface Memory - Used for image data.
#### Memory Allocation APIs:
cudaMalloc(), cudaFree()
cudaMemcpy() for memory transfer between Host and Device.

### Optimizing CUDA Performance
- Maximize occupancy by - using the right number of threads and blocks.
- Use shared memory for frequently accessed data.
- Reduce global memory accesses.
- Apply memory coalescing for efficient memory usage.




###  **1. Check CUDA Installation**
Make sure CUDA is installed and detected using:
```bash
nvcc --version
```

---

###  **2. Check GPU Status using `nvidia-smi`**
You can monitor your GPU usage, temperature, memory consumption, and running processes using:
```bash
nvidia-smi
```
Output Example:
```
+-----------------------------------------------------------------------------+
| NVIDIA-SMI 535.86.05    Driver Version: 535.86.05    CUDA Version: 12.2     |
|-------------------------------+----------------------+----------------------+
| GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
| 0    NVIDIA RTX 3090     Off  | 00000000:01:00.0 Off |                  Off |
+-------------------------------+----------------------+----------------------+
```

---

###  **3. Check CUDA Availability in Python**


- **Using CuPy**:  
```python
import cupy as cp

print("CUDA available:", cp.cuda.is_available())
print("Device name:", cp.cuda.Device(0).name)
```

- **Using Numba**:  
```python
from numba import cuda

print("CUDA available:", cuda.is_available())
print("GPU Device:", cuda.get_current_device().name)
```

---

###  **4. Measure GPU Memory Usage**
You can check available GPU memory using Python:  

- **CuPy**:  
```python
import cupy as cp

free_mem, total_mem = cp.cuda.Device(0).mem_info
print(f"Free memory: {free_mem / 1e9:.2f} GB")
print(f"Total memory: {total_mem / 1e9:.2f} GB")
```

- **Numba**:  
```python
from numba import cuda

device = cuda.get_current_device()
print("Total memory (MB):", device.total_memory() / (1024 * 1024))
```

---

### **5. Monitor GPU Utilization in Real-Time**
You can run this command to watch GPU usage live:  
```bash
watch -n 1 nvidia-smi
```
- **`-n 1`** updates the stats every second.  
- Press `Ctrl + C` to exit.

---


Here are more useful CUDA-related commands for Python, Linux, and CUDA management:  

---

## **General CUDA Commands**  

1. **Check CUDA Version:**  
```bash
nvcc --version
```

2. **Check Driver and GPU Status:**  
```bash
nvidia-smi
```

3. **List All CUDA Capable Devices:**  
```bash
nvidia-smi -L
```
Output Example:  
```
GPU 0: NVIDIA RTX 3090 (UUID: GPU-abc123)
```

4. **Monitor GPU Usage in Real-Time:**  
```bash
watch -n 1 nvidia-smi
```
*(Press `Ctrl + C` to exit)*  

---

## **Memory and GPU Management**  

5. **Check GPU Memory Usage:**  
```bash
nvidia-smi --query-gpu=memory.used,memory.free --format=csv
```

6. **Limit GPU Memory Usage:**  
You can limit the memory visible to your application using the `CUDA_VISIBLE_DEVICES` environment variable:  
```bash
export CUDA_VISIBLE_DEVICES=0
```
*(Restricts the app to use only GPU 0)*

7. **Clear GPU Memory:**  
If GPU memory is stuck, you can reset it:  
```bash
sudo nvidia-smi --gpu-reset -i 0
```
*(Make sure no GPU-intensive programs are running)*  

---

## **Compile and Run CUDA Programs**  

8. **Compile CUDA Code (.cu files):**  
```bash
nvcc filename.cu -o output
```

9. **Run the Compiled CUDA Program:**  
```bash
./output
```

10. **Debug CUDA Applications using CUDA-GDB:**  
```bash
cuda-gdb ./output
```

---

## **Python-Specific CUDA Commands**  

11. **Check CUDA Availability in Python:**  
```python
from numba import cuda
print("CUDA Available:", cuda.is_available())
```

12. **List CUDA Devices:**  
```python
import cupy as cp
print(cp.cuda.runtime.getDeviceCount())
```

13. **Get Device Details:**  
```python
device = cp.cuda.Device(0)
print(f"Device: {device.name}, Memory: {device.mem_info[1]/1e9:.2f} GB")
```

14. **Release GPU Memory (Garbage Collection):**  
```python
import cupy as cp
cp.get_default_memory_pool().free_all_blocks()
```

---

## **Performance Monitoring**  

15. **Monitor GPU Utilization with NVTOP:**  
Install it first:  
```bash
sudo apt install nvtop
```
Run:  
```bash
nvtop
```

16. **Monitor GPU Temperature:**  
```bash
nvidia-smi --query-gpu=temperature.gpu --format=csv
```

17. **Profile CUDA Programs using NVIDIA Nsight Systems:**  
```bash
nsys profile ./output
```

18. **Profile Memory Usage using NVIDIA Nsight Compute:**  
```bash
ncu ./output
```

---

## **CUDA Logs and Error Checking**  

19. **Check CUDA Logs:**  
```bash
dmesg | grep -i nvidia
```

20. **Check Kernel Logs:**  
```bash
sudo cat /var/log/kern.log | grep nvidia
```

21. **Run Diagnostics:**  
```bash
nvidia-smi -q -d ERROR
```

22. **Check CUDA Error in Python:**  
```python
import cupy as cp

try:
    a = cp.array([1, 2, 3])
    b = cp.array([4, 5])
    print(a + b)
except cp.cuda.runtime.CUDARuntimeError as e:
    print("CUDA Error:", e)
```

---

Here's a proper revision on **OpenCL (Open Computing Language)** to get you started:  

---

##  **Introduction to OpenCL**
- OpenCL is an open standard for writing programs that run across heterogeneous platforms, including CPUs, GPUs, FPGAs, and other processors.  
- It provides a unified programming environment to develop parallel programs using C or C++ like syntax.  
- Mainly used for tasks like **High-Performance Computing (HPC)**, **Machine Learning** model training, **Image Processing**, and **Scientific Simulations**.  

---

## **Key Concepts in OpenCL**
1. **Platform Model**  
   - OpenCL uses a host (typically CPU) to control multiple compute devices (e.g., GPU, FPGA).  
   - Devices contain one or more Compute Units (CUs), which further contain Processing Elements (PEs).  

2. **Execution Model**  
   - Code written for OpenCL consists of a **host code** (runs on CPU) and **kernel code** (runs on GPU or other devices).  
   - The host manages memory, enqueues kernels, and handles synchronization.  
   - Kernels are functions that are executed in parallel by multiple threads.

3. **Memory Model**  
   OpenCL defines four distinct memory regions:  
   - **Global Memory**: Accessible by all compute units.  
   - **Constant Memory**: Read-only memory that remains unchanged during execution.  
   - **Local Memory**: Shared within a compute unit.  
   - **Private Memory**: Private to each processing element.  

---

##  **Development Workflow**
1. **Platform Initialization**  
   - Query available platforms and devices using `clGetPlatformIDs()` and `clGetDeviceIDs()`.  

2. **Context and Command Queue**  
   - Create an OpenCL context using `clCreateContext()`.  
   - Create a command queue with `clCreateCommandQueue()` for managing commands sent to devices.  

3. **Program and Kernel Creation**  
   - Load and build the kernel source using `clCreateProgramWithSource()` and `clBuildProgram()`.  
   - Extract a kernel using `clCreateKernel()`.  

4. **Memory Management**  
   - Allocate buffers using `clCreateBuffer()`.  
   - Transfer data using `clEnqueueWriteBuffer()` and retrieve results using `clEnqueueReadBuffer()`.  

5. **Kernel Execution**  
   - Set kernel arguments using `clSetKernelArg()`.  
   - Launch kernels using `clEnqueueNDRangeKernel()`.

---

## **Basic OpenCL Code Structure**
```c
// Platform and Device Initialization
cl_platform_id platform;
cl_device_id device;
clGetPlatformIDs(1, &platform, NULL);
clGetDeviceIDs(platform, CL_DEVICE_TYPE_GPU, 1, &device, NULL);

// Context and Command Queue
cl_context context = clCreateContext(NULL, 1, &device, NULL, NULL, NULL);
cl_command_queue queue = clCreateCommandQueue(context, device, 0, NULL);

// Program and Kernel
const char *kernelSource = "__kernel void add(__global int* a, __global int* b, __global int* c) { "
                            "  int id = get_global_id(0); "
                            "  c[id] = a[id] + b[id]; "
                            "}";
cl_program program = clCreateProgramWithSource(context, 1, &kernelSource, NULL, NULL);
clBuildProgram(program, 1, &device, NULL, NULL, NULL);
cl_kernel kernel = clCreateKernel(program, "add", NULL);

// Memory Management
int N = 1024;
int a[N], b[N], c[N];
cl_mem d_a = clCreateBuffer(context, CL_MEM_READ_ONLY | CL_MEM_COPY_HOST_PTR, sizeof(int) * N, a, NULL);
cl_mem d_b = clCreateBuffer(context, CL_MEM_READ_ONLY | CL_MEM_COPY_HOST_PTR, sizeof(int) * N, b, NULL);
cl_mem d_c = clCreateBuffer(context, CL_MEM_WRITE_ONLY, sizeof(int) * N, NULL, NULL);

// Set Kernel Arguments and Execute
clSetKernelArg(kernel, 0, sizeof(cl_mem), &d_a);
clSetKernelArg(kernel, 1, sizeof(cl_mem), &d_b);
clSetKernelArg(kernel, 2, sizeof(cl_mem), &d_c);

size_t globalSize = N;
clEnqueueNDRangeKernel(queue, kernel, 1, NULL, &globalSize, NULL, 0, NULL, NULL);
clEnqueueReadBuffer(queue, d_c, CL_TRUE, 0, sizeof(int) * N, c, 0, NULL, NULL);
```

---

## **Debugging and Optimization Tips**
- Use `clGetProgramBuildInfo()` to retrieve build errors.  
- Profile using OpenCL profiling tools like **Intel VTune** or **NVIDIA Nsight**.  
- Optimize memory usage by minimizing global memory access.  
- Maximize work group utilization for better parallelization.

---
