# OCI GPU Quick Starts: AMD MI355X Instinct GPUs

This document provides hardware specifications, supported OS images, onboarding verification, sample benchmarks, and best-practices for OCI deployments using the AMD MI355X Pollara GPU bare-metal shape.

MI355X shapes represent a specialized class of heterogeneous accelerated compute in Oracle Cloud Infrastructure (OCI), built around the AMD Instinct MI355X GPU. These shapes provide 8 MI355X GPUs per node, each with 288 GiB of high-bandwidth HBM memory, targeting large-scale AI/ML workloads and HPC.

MI355X systems leverage AMD’s XGMI (Inter-GPU Express Global Memory Interconnect) for direct GPU-to-GPU connectivity within each node. On a typical MI355X system, all 8 GPUs are fully connected via XGMI, providing a single-hop, high-bandwidth, low-latency peer-to-peer path between any pair of GPUs. This topology is symmetrical, meaning all GPUs enjoy equal connectivity for workloads that depend on frequent collective or point-to-point 
communication.

## Pollara
Pollara nodes are a variant of the MI355X bare metal GPU shape family in OCI that pair 8x AMD Instinct MI355X GPUs with the AMD Pensando™ Pollara 400 AI NIC for high-bandwidth scale-out training and HPC workloads over Ethernet (RoCE/RDMA). These systems are designed for multi-node collective performance and high-throughput east–west traffic.

Within each node, all 8 MI355X GPUs are connected via AMD XGMI, enabling a symmetrical, single-hop, high-bandwidth GPU-to-GPU topology suitable for communication-heavy workloads (e.g., model parallelism, pipeline parallelism, large batch training, and dense collectives).

# Table of Contents

* [Hardware Specifications](#hardware-specifications)
* [Recommended Operating Systems](#recommended-operating-systems)
    * [Recommended Software Version](#recommended-software-version)
    * [Custom OS Image Creation with Packer](#custom-os-image-creation-with-packer)
    * [Provided Images](#provided-images)
* [Hello World Verification](#hello-world-verification)
* [Performance Benchmarks](#performance-benchmarks)
    * [RCCL & Model Inference Performance](#rccl--model-inference-performance)
* [OKE GPU Getting Started](#oke-gpu-getting-started)
* [Troubleshooting](#troubleshooting)
* [Further Reading & Support](#further-reading--support)

# Hardware Specifications

Note: Pollara nodes may use a different system image than ConnectX-based nodes. Align to the [OCI-provided image](#provided-images) for the rack.

| Shape Name        | GPU Model     | GPUs/Node | GPU Memory (GB/GPU) | GPU Memory Total |             CPU           | # of CPUs | System Memory | Local Storage	 | Host NIC | RDMA (ROCe) NICs |
|-------------------|---------------|-----------|------------|----------------- |---------------------------|-----------|---------------|----------------|----------|-----------|
| BM.GPU.MI355x.v0.8| AMD MI355X    |   8       | 288 | 2.3 TB  | AMD EPYC 9575F (x2)| 64 (128 Cores) | 3TB	| 8 x 7.68TB NVMe  | NVIDIA CX-7 2x200GBps = 400GBps| AMD Pensando™ Pollara 400 AI NIC, 8x400Gb/s Ethernet = 3.2 Tb/s

See the [OCI Compute Shapes Docs](https://docs.oracle.com/en-us/iaas/Content/Compute/References/computeshapes.htm) for up-to-date details.

# Recommended Operating Systems

• Ubuntu 22.04+\
• Kernel: 5.15.0-1074-oracle + (Use OCI-provided images for the Pollara variant whenever possible to ensure the correct kernel, drivers, and firmware alignment.)\
• AINIC FW: 1.117.5-a-56 (For 8 x Pensando Backend NICs) 

## Recommended Software Version

• ROCm: 7.2.0+\
• RCCL: 2.26.6\
• Oracle Cloud Agent: 1.55.0+\
• Networking stack note (Pollara)\
• OFED: MLNX OFED 5.9 for 22.04 and 24.10 for 24.04 (For 2 x CX7 Frontend NICs)\
• OpenMPI: 4.1.6, 5.0.8\
• amd-anp (amd ainic network plugin designed to enhance performance for RCCL collective communications library)\
• amd-argus

## Custom OS image Creation with Packer

To build your images using packer clone the OCI HPC Images repo and run the commands found there [OCI HPC Images GitHub Repo](https://github.com/oracle-quickstart/oci-hpc-images/blob/main/README.md).

## Provided Images

It is recommended to use Oracle-provided or organizationally-approved images for security and supportability. See OS Images Table for region-specific OCID's and version info.

| OS Version        | Image Packer Build Details       | OCI Platform Image Link                                                                        | Driver Versions | Build & Dependency Status | 
|-------------------|-------------------------------|------------------------------------------------------------------------------------------------------------|--------------|--------------------------|
| OCI GPU AI Image with Ubuntu Linux 22.04  | To be updated | Under construction — image link will be provided in a future update. | ROCM 7.0.2, RCCL 2.26.6, OFED 5.9, Oracle Cloud Agent 1.57.0 | ![Build](/media/icons/build-passing.svg) ![Build](/media/icons/dependencies.svg) |
| OCI GPU AI Image with Ubuntu Linux 24.04.4 LTS | To be updated | Under construction — image link will be provided in a future update. |  MLNX OFED LINUX-24.10-1.1.4.0, ROCm 7.0.2, Pollara NIC Firmware 1.117.5-a-56, OpenMPI 5.0.8, OCA: 1.57.0-2248 |  ![Build](/media/icons/build-passing.svg) ![Build](/media/icons/dependencies.svg) |
| OCI GPU AI Image with Ubuntu Linux 24.04.4 LTS | To be updated | Under construction — image link will be provided in a future update. | MLNX OFED LINUX-24.10-1.1.4.0, ROCm 7.2, Pollara NIC Firmware: 1.117.5-a-56, OpenMPI 5.0.8, OCA: 1.57.0-2248 | ![Build](/media/icons/build-passing.svg) ![Build](/media/icons/dependencies.svg) | 

***Note that ROCM 7.2 is experimental and not currently recommended for MI355x-Pollara**

# Hello World Verification

You will need docker installed on the OS image before you can run this command. More information on different functions that can be run is found [here](https://rocm.docs.amd.com/projects/install-on-linux/en/latest/install/3rd-party/pytorch-install.html#using-docker-with-pytorch-pre-installed)

```bash
docker pull rocm/pytorch:latest
docker run -it \
    --cap-add=SYS_PTRACE \
    --security-opt seccomp=unconfined \
    --device=/dev/kfd \
    --device=/dev/dri \
    --group-add video \
    --ipc=host \
    --shm-size 8G \
    rocm/pytorch:latest
```


# Performance Benchmarks

The section below has the base line numbers achieved and how to reproduce this with MI355X. When you run the same there is typically a variance of 5%-8% which is acceptable. 

For guidance on installing and running RCCL benchmarks, see the [AMD RCCL documentation](https://rocm.docs.amd.com/projects/rccl).

## RCCL & Model Inference Performance

### All Reduce - Single Node

```bash
#
#                                                              out-of-place                       in-place          
#       size         count      type   redop    root     time   algbw   busbw #wrong     time   algbw   busbw #wrong
#        (B)    (elements)                               (us)  (GB/s)  (GB/s)            (us)  (GB/s)  (GB/s)       
# <smaller message sizes truncated>
  8589934592    2147483648     float     sum      -1    36863  233.02  407.79      0    36835  233.20  408.10      0
 17179869184    4294967296     float     sum      -1    73544  233.60  408.80      0    73681  233.17  408.04      0
# Errors with asterisks indicate errors that have exceeded the maximum threshold.
# Out of bounds values : 0 OK
# Avg bus bandwidth    : 169.455 
#
# Collective test concluded: all_reduce_perf
```
### All 2 All - Single Node

```bash
#                                                              out-of-place                       in-place          
#       size         count      type   redop    root     time   algbw   busbw #wrong     time   algbw   busbw #wrong
#        (B)    (elements)                               (us)  (GB/s)  (GB/s)            (us)  (GB/s)  (GB/s)       
# <smaller message sizes truncated>
  8589934592     268435456     float    none      -1    20839  412.20  360.67      0    20893  411.14  359.75    N/A
 17179869184     536870912     float    none      -1    41626  412.72  361.13      0    41546  413.52  361.83    N/A
# Errors with asterisks indicate errors that have exceeded the maximum threshold.
# Out of bounds values : 0 OK
# Avg bus bandwidth    : 133.643 
```
### All Reduce - 2 Nodes
```bash
#                                                              out-of-place                       in-place          
#       size         count      type   redop    root     time   algbw   busbw #wrong     time   algbw   busbw #wrong
#        (B)    (elements)                               (us)  (GB/s)  (GB/s)            (us)  (GB/s)  (GB/s)       
# <smaller message sizes truncated>
  8589934592    2147483648     float     sum      -1    43674  196.68  368.78      0    43684  196.64  368.70      0
 17179869184    4294967296     float     sum      -1    86980  197.52  370.34      0    86978  197.52  370.35      0
# Errors with asterisks indicate errors that have exceeded the maximum threshold.
# Out of bounds values : 0 OK
# Avg bus bandwidth    : 135.701 
#
# Collective test concluded: all_reduce_perf
```

### All 2 All - 2 Nodes
```bash
#                                                              out-of-place                       in-place          
#       size         count      type   redop    root     time   algbw   busbw #wrong     time   algbw   busbw #wrong
#        (B)    (elements)                               (us)  (GB/s)  (GB/s)            (us)  (GB/s)  (GB/s)       
# <smaller message sizes truncated>
  8589934592     134217728     float    none      -1    92988   92.38   86.60      0    92968   92.40   86.62    N/A
 17179869184     268435456     float    none      -1   185711   92.51   86.73      0   185928   92.40   86.63    N/A
# Errors with asterisks indicate errors that have exceeded the maximum threshold.
# Out of bounds values : 0 OK
# Avg bus bandwidth    : 35.7757 
#
# Collective test concluded: alltoall_perf
```

### All Reduce - 4 Nodes
```bash
#                                                              out-of-place                       in-place          
#       size         count      type   redop    root     time   algbw   busbw #wrong     time   algbw   busbw #wrong
#        (B)    (elements)                               (us)  (GB/s)  (GB/s)            (us)  (GB/s)  (GB/s)       
# <smaller message sizes truncated>        
  8589934592    2147483648     float     sum      -1    44952  191.09  370.24      0    44947  191.11  370.28      0
 17179869184    4294967296     float     sum      -1    89839  191.23  370.51      0    89846  191.22  370.48      0
# Errors with asterisks indicate errors that have exceeded the maximum threshold.
# Out of bounds values : 0 OK
# Avg bus bandwidth    : 117.167 
#
# Collective test concluded: all_reduce_perf
```

### All 2 All - 4 Nodes
```bash
#                                                              out-of-place                       in-place          
#       size         count      type   redop    root     time   algbw   busbw #wrong     time   algbw   busbw #wrong
#        (B)    (elements)                               (us)  (GB/s)  (GB/s)            (us)  (GB/s)  (GB/s)       
# <smaller message sizes truncated>
  8589934592      67108864     float    none      -1   165769   51.82   50.20      0   164522   52.21   50.58    N/A
 17179869184     134217728     float    none      -1   328749   52.26   50.63      0   327875   52.40   50.76    N/A
# Errors with asterisks indicate errors that have exceeded the maximum threshold.
# Out of bounds values : 0 OK
# Avg bus bandwidth    : 20.8042 
#
# Collective test concluded: alltoall_perf
```

### LLM Inference Performance Numbers

The below output is for vLLM Throughput running the below command and the container release. For latest container version refer [here.](https://rocm.docs.amd.com/en/docs-7.0-docker/benchmark-docker/inference-vllm-gpt-oss-120b.html) 

Example output:
```bash
docker pull rocm/7.x-preview:rocm7.2_preview_ubuntu_22.04_vlm_0.10.1_instinct_20251029
```
While vLLM can download model weights at runtime, it’s recommended to download ahead of time. You will need:

A valid Hugging Face access token. Remember to set HF_TOKEN to your access token.

Access granted to the specific model from your Hugging Face account

```bash
model=openai/gpt-oss-120b

pip install huggingface_hub[cli] hf_transfer hf_xet
HF_HUB_ENABLE_HF_TRANSFER=1 \
HF_HOME=/data/huggingface-cache \
HF_TOKEN="<HF_TOKEN>" \ # Replace with your HF_TOKEN Hugging Face access token.
huggingface-cli download ${model} --exclude "original/*"

```

Start the container on MI355X. You will be inside the container once this command is successful.

```bash
docker run -it \
  --ipc=host \
  --network=host \
  --privileged \
  --cap-add=CAP_SYS_ADMIN \
  --device=/dev/kfd \
  --device=/dev/dri \
  --cap-add=SYS_PTRACE \
  --security-opt seccomp=unconfined \
  -v /data:/data \
  -e HF_HOME=/data/huggingface-cache \
  -e HF_HUB_OFFLINE=1 \
  --name vllm-server \
  rocm/7.x-preview:rocm7.2_preview_ubuntu_22.04_vlm_0.10.1_instinct_20251029
```
Start the vLLM server

```bash
model=openai/gpt-oss-120b
max_model_len=10368           # 1.125 x (input sequence length + output sequence length); e.g. 1.125 x (8192 + 1024) = 10368.
max_seq_len_to_capture=10368  # Beneficial to set this to max_model_len.
max_num_seqs=1024             # Set to max_concurrency of the client to get better throughput.
tensor_parallel_size=8

export VLLM_USE_AITER_UNIFIED_ATTENTION=1
export VLLM_ROCM_USE_AITER_MHA=0
export VLLM_ROCM_USE_AITER_FUSED_MOE_A16W4=1

vllm serve ${model} \
    --port 8000 \
    --swap-space 64 \
    --max-model-len ${max_model_len} \
    --tensor-parallel-size ${tensor_parallel_size} \
    --max-num-seqs ${max_num_seqs} \
    --gpu-memory-utilization 0.95 \
    --max-seq-len-to-capture ${max_seq_len_to_capture} \
    --compilation-config '{"compile_sizes":[1,2,4,6,8,10,12,14,16,18,20,22,24,26,28,30,32,34,36,38,40,42,44,46,48,50,52,54,56,58,60,62,64,66,68,70,72,74,76,78,80,82,84,86,88,90,92,94,96,98,100,102,104,106,108,110,112,114,116,118,120,122,124,126,128,256,512,1024,2048,8192] , "cudagraph_capture_sizes":[1,2,4,6,8,10,12,14,16,18,20,22,24,26,28,30,32,34,36,38,40,42,44,46,48,50,52,54,56,58,60,62,64,66,68,70,72,74,76,78,80,82,84,86,88,90,92,94,96,98,100,102,104,106,108,110,112,114,116,118,120,122,124,126,128,136,144,152,160,168,176,184,192,200,208,216,224,232,240,248,256,264,272,280,288,296,304,312,320,328,336,344,352,360,368,376,384,392,400,408,416,424,432,440,448,456,464,472,480,488,496,504,512,520,528,536,544,552,560,568,576,584,592,600,608,616,624,632,640,648,656,664,672,680,688,696,704,712,720,728,736,744,752,760,768,776,784,792,800,808,816,824,832,840,848,856,864,872,880,888,896,904,912,920,928,936,944,952,960,968,976,984,992,1000,1008,1016,1024,2048,4096,8192] , "cudagraph_mode": "FULL_AND_PIECEWISE"}' \
    --block-size=64 \
    --no-enable-prefix-caching \
    --async-scheduling

 # Wait for model to load and server is ready to accept requests.
```
Open another terminal on the same machine, connect to your running vllm-server container, and run the benchmark with the appropriate options. For example:

```bash
# Connect to server
docker exec -it vllm-server bash
```
Inside the container 

```bash
# Run the client benchmark
model=openai/gpt-oss-120b
input_tokens=1024 # you can toggle the sizes here
output_tokens=1024 #you can toggle the sizes here
max_concurrency=4
num_prompts=32

python3 /app/vllm/benchmarks/benchmark_serving.py --host localhost --port 8000 \
    --model ${model} \
    --dataset-name random \
    --random-input-len ${input_tokens} \
    --random-output-len ${output_tokens} \
    --max-concurrency ${max_concurrency} \
    --num-prompts ${num_prompts} \
    --percentile-metrics ttft,tpot,itl,e2el \
    --ignore-eos
```
#### Example Results
```sh
============ Serving Benchmark Result ============
Successful requests:                     32        
Maximum request concurrency:             4         
Benchmark duration (s):                  29.20     
Total input tokens:                      32601     
Total generated tokens:                  32768     
Request throughput (req/s):              1.10      
Output token throughput (tok/s):         1122.33   
Total Token throughput (tok/s):          2238.93   
---------------Time to First Token----------------
Mean TTFT (ms):                          89.77     
Median TTFT (ms):                        51.25     
P99 TTFT (ms):                           522.23    
-----Time per Output Token (excl. 1st token)------
Mean TPOT (ms):                          3.48      
Median TPOT (ms):                        3.46      
P99 TPOT (ms):                           3.81      
---------------Inter-token Latency----------------
Mean ITL (ms):                           3.49      
Median ITL (ms):                         3.37      
P99 ITL (ms):                            8.25      
----------------End-to-end Latency----------------
Mean E2EL (ms):                          3648.97   
Median E2EL (ms):                        3591.23   
P99 E2EL (ms):                           4080.27   
==================================================
```

## Sweep Results

| Input Tokens | Output Tokens | Output Throughput (tok/s) | Total Throughput (tok/s) |
|-------------|--------------|--------------------------|-------------------------|
| 4,096 | 4,096 | 964.73 | 1,929.46 |
| 4,096 | 32,768 | 1,145.46 | 1,288.65 |
| 4,096 | 65,536 | 1,149.56 | 1,221.41 |
| 16,384 | 131,072 | 1,145.09 | 1,288.22 |
| 65,381 | 4,096 | 984.90 | 16,706.06 |
| 65,381 | 65,536 | 1,135.33 | 2,267.98 |
| 65,381 | 131,072 | 1,139.77 | 1,708.31 |
| 130,970 | 65,536 | 1,123.75 | 3,369.50 |
| 262,135 | 65,536 | 1,102.41 | 5,511.91 |
| 65,381 | 262,144 | 1,141.67 | 1,426.41 |

---

# OKE GPU Getting Started

1. Create OKE Cluster via OCI Console or CLI.
2. Add GPU Node Pool as self managed node: More info [here](https://docs.oracle.com/en-us/iaas/Content/ContEng/Tasks/contengworkingwithselfmanagednodes.htm). You can use the same OS Image listed above and bootstrap the kubernetes components and registration to Kubernetes control plane. More abut adding it as [cloud init script here](https://github.com/oracle-quickstart/oci-hpc-oke/blob/main/files/oke-ubuntu-cloud-init.sh)
3. Install AMD Device Plugin on OKE:
```bash
kubectl apply -f https://github.com/AMD/k8s-device-plugin/raw/main/AMD-device-plugin.yml
```
4. Run GPU-powered Kubernetes workloads:
Example pod resource spec:

[AMD RVS Test Yaml](/amd/MI355X/k8s/active-health-checks-rvs.yaml)

5. Verify with Hello World ROCm container pod is deployable and you can see the AMD GPUs within it

```bash
kubectl create -f ./amd/MI355X/k8s/hello-world-rocm-oke.yaml
```
Output
```bash
kubectl logs amd-smi

============================================= ROCm System Management Interface =============================================						
======================================================= Concise Info =======================================================						
Device  Node  IDs              Temp        Power     Partitions          SCLK     MCLK     Fan  Perf  PwrCap   VRAM%  GPU%						
              (DID,     GUID)  (Junction)  (Socket)  (Mem, Compute, ID)                                                     						
============================================================================================================================						
0       3     0x75a3,   21010  54.0°C      1224.0W   NPS1, SPX, 0        2358Mhz  2000Mhz  0%   auto  1400.0W  98%    93% 						
1       5     0x75a3,   56525  54.0°C      1231.0W   NPS1, SPX, 0        2366Mhz  2000Mhz  0%   auto  1400.0W  98%    95% 						
2       4     0x75a3,   28206  55.0°C      1219.0W   NPS1, SPX, 0        2361Mhz  2000Mhz  0%   auto  1400.0W  98%    94% 						
3       2     0x75a3,   28720  56.0°C      1249.0W   NPS1, SPX, 0        2340Mhz  2000Mhz  0%   auto  1400.0W  98%    96% 						
4       7     0x75a3,   25266  52.0°C      1275.0W   NPS1, SPX, 0        2374Mhz  2000Mhz  0%   auto  1400.0W  98%    100%						
5       9     0x75a3,   20206  56.0°C      1267.0W   NPS1, SPX, 0        2377Mhz  2000Mhz  0%   auto  1400.0W  98%    100%						
6       8     0x75a3,   16143  54.0°C      1273.0W   NPS1, SPX, 0        2382Mhz  2000Mhz  0%   auto  1400.0W  98%    100%						
7       6     0x75a3,   8465   54.0°C      1317.0W   NPS1, SPX, 0        2374Mhz  2000Mhz  0%   auto  1400.0W  98%    100%						
============================================================================================================================						
=================================================== End of ROCm SMI Log ====================================================	

AMDSMI Tool: 24.6.2+2b02a07 | AMDSMI Library version: 24.6.2.0 | ROCm version: 7.1.2

```

# Troubleshooting

Here you can find suggested troubleshooting methods.

* [Confirm Hardware & Drivers](#confirm-hardware--drivers)
* [AINIC](#ainic-pesando)
* [RDMA Link](#rdma-link)
* [IB Write](#ib-write)

## Confirm Hardware & Drivers
```sh
amd-smi

+------------------------------------------------------------------------------+
| AMD-SMI 26.0.2+39589fda      amdgpu version: 6.16.13  ROCm version: 7.0.2    |
| Platform: Linux Baremetal                                                    |
|-------------------------------------+----------------------------------------|
| BDF                        GPU-Name | Mem-Uti   Temp   UEC       Power-Usage |
| GPU  HIP-ID  OAM-ID  Partition-Mode | GFX-Uti    Fan               Mem-Usage |
|=====================================+========================================|
| 0000:0d:00.0    AMD Instinct MI355X | N/A        N/A   0          N/A/1400 W |
|   0       1       7        SPX/NPS1 | N/A        N/A           283/294896 MB |
|-------------------------------------+----------------------------------------|
| 0000:26:00.0    AMD Instinct MI355X | N/A        N/A   0          N/A/1400 W |
|   1       3       5        SPX/NPS1 | N/A        N/A           283/294896 MB |
|-------------------------------------+----------------------------------------|
| 0000:5d:00.0    AMD Instinct MI355X | N/A        N/A   0          N/A/1400 W |
|   2       2       6        SPX/NPS1 | N/A        N/A           283/294896 MB |
|-------------------------------------+----------------------------------------|
| 0000:75:00.0    AMD Instinct MI355X | N/A        N/A   0          N/A/1400 W |
|   3       0       2        SPX/NPS1 | N/A        N/A           283/294896 MB |
|-------------------------------------+----------------------------------------|
| 0000:8e:00.0    AMD Instinct MI355X | N/A        N/A   0          N/A/1400 W |
|   4       5       0        SPX/NPS1 | N/A        N/A           283/294896 MB |
|-------------------------------------+----------------------------------------|
| 0000:a7:00.0    AMD Instinct MI355X | N/A        N/A   0          N/A/1400 W |
|   5       7       4        SPX/NPS1 | N/A        N/A           283/294896 MB |
|-------------------------------------+----------------------------------------|
| 0000:dc:00.0    AMD Instinct MI355X | N/A        N/A   0          N/A/1400 W |
|   6       6       1        SPX/NPS1 | N/A        N/A           283/294896 MB |
|-------------------------------------+----------------------------------------|
| 0000:f4:00.0    AMD Instinct MI355X | N/A        N/A   0          N/A/1400 W |
|   7       4       3        SPX/NPS1 | N/A        N/A           283/294896 MB |
+-------------------------------------+----------------------------------------+
+------------------------------------------------------------------------------+
| Processes:                                                                   |
|  GPU        PID  Process Name          GTT_MEM  VRAM_MEM  MEM_USAGE     CU % |
|==============================================================================|
|  No running processes found                                                  |
+------------------------------------------------------------------------------+
```
```sh
amd-smi topology

ACCESS TABLE:
             0000:0d:00.0 0000:26:00.0 0000:5d:00.0 0000:75:00.0 0000:8e:00.0 0000:a7:00.0 0000:dc:00.0 0000:f4:00.0
0000:0d:00.0 ENABLED      ENABLED      ENABLED      ENABLED      ENABLED      ENABLED      ENABLED      ENABLED
0000:26:00.0 ENABLED      ENABLED      ENABLED      ENABLED      ENABLED      ENABLED      ENABLED      ENABLED
0000:5d:00.0 ENABLED      ENABLED      ENABLED      ENABLED      ENABLED      ENABLED      ENABLED      ENABLED
0000:75:00.0 ENABLED      ENABLED      ENABLED      ENABLED      ENABLED      ENABLED      ENABLED      ENABLED
0000:8e:00.0 ENABLED      ENABLED      ENABLED      ENABLED      ENABLED      ENABLED      ENABLED      ENABLED
0000:a7:00.0 ENABLED      ENABLED      ENABLED      ENABLED      ENABLED      ENABLED      ENABLED      ENABLED
0000:dc:00.0 ENABLED      ENABLED      ENABLED      ENABLED      ENABLED      ENABLED      ENABLED      ENABLED
0000:f4:00.0 ENABLED      ENABLED      ENABLED      ENABLED      ENABLED      ENABLED      ENABLED      ENABLED
WEIGHT TABLE:
             0000:0d:00.0 0000:26:00.0 0000:5d:00.0 0000:75:00.0 0000:8e:00.0 0000:a7:00.0 0000:dc:00.0 0000:f4:00.0
0000:0d:00.0 0            0            0            0            0            0            0            0
0000:26:00.0 0            0            0            0            0            0            0            0
0000:5d:00.0 0            0            0            0            0            0            0            0
0000:75:00.0 0            0            0            0            0            0            0            0
0000:8e:00.0 0            0            0            0            0            0            0            0
0000:a7:00.0 0            0            0            0            0            0            0            0
0000:dc:00.0 0            0            0            0            0            0            0            0
0000:f4:00.0 0            0            0            0            0            0            0            0
HOPS TABLE:
             0000:0d:00.0 0000:26:00.0 0000:5d:00.0 0000:75:00.0 0000:8e:00.0 0000:a7:00.0 0000:dc:00.0 0000:f4:00.0
0000:0d:00.0 0            1            1            1            1            1            1            1
0000:26:00.0 1            0            1            1            1            1            1            1
0000:5d:00.0 1            1            0            1            1            1            1            1
0000:75:00.0 1            1            1            0            1            1            1            1
0000:8e:00.0 1            1            1            1            0            1            1            1
0000:a7:00.0 1            1            1            1            1            0            1            1
0000:dc:00.0 1            1            1            1            1            1            0            1
0000:f4:00.0 1            1            1            1            1            1            1            0
LINK TYPE TABLE:
             0000:0d:00.0 0000:26:00.0 0000:5d:00.0 0000:75:00.0 0000:8e:00.0 0000:a7:00.0 0000:dc:00.0 0000:f4:00.0
0000:0d:00.0 SELF         XGMI         XGMI         XGMI         XGMI         XGMI         XGMI         XGMI
0000:26:00.0 XGMI         SELF         XGMI         XGMI         XGMI         XGMI         XGMI         XGMI
0000:5d:00.0 XGMI         XGMI         SELF         XGMI         XGMI         XGMI         XGMI         XGMI
0000:75:00.0 XGMI         XGMI         XGMI         SELF         XGMI         XGMI         XGMI         XGMI
0000:8e:00.0 XGMI         XGMI         XGMI         XGMI         SELF         XGMI         XGMI         XGMI
0000:a7:00.0 XGMI         XGMI         XGMI         XGMI         XGMI         SELF         XGMI         XGMI
0000:dc:00.0 XGMI         XGMI         XGMI         XGMI         XGMI         XGMI         SELF         XGMI
0000:f4:00.0 XGMI         XGMI         XGMI         XGMI         XGMI         XGMI         XGMI         SELF
NUMA BW TABLE:
             0000:0d:00.0 0000:26:00.0 0000:5d:00.0 0000:75:00.0 0000:8e:00.0 0000:a7:00.0 0000:dc:00.0 0000:f4:00.0
0000:0d:00.0 N/A          0-0          0-0          0-0          0-0          0-0          0-0          0-0
0000:26:00.0 0-0          N/A          0-0          0-0          0-0          0-0          0-0          0-0
0000:5d:00.0 0-0          0-0          N/A          0-0          0-0          0-0          0-0          0-0
0000:75:00.0 0-0          0-0          0-0          N/A          0-0          0-0          0-0          0-0
0000:8e:00.0 0-0          0-0          0-0          0-0          N/A          0-0          0-0          0-0
0000:a7:00.0 0-0          0-0          0-0          0-0          0-0          N/A          0-0          0-0
0000:dc:00.0 0-0          0-0          0-0          0-0          0-0          0-0          N/A          0-0
0000:f4:00.0 0-0          0-0          0-0          0-0          0-0          0-0          0-0          N/A
CACHE COHERANCY TABLE:
             0000:0d:00.0 0000:26:00.0 0000:5d:00.0 0000:75:00.0 0000:8e:00.0 0000:a7:00.0 0000:dc:00.0 0000:f4:00.0
0000:0d:00.0 SELF         C            C            C            C            C            C            C
0000:26:00.0 C            SELF         C            C            C            C            C            C
0000:5d:00.0 C            C            SELF         C            C            C            C            C
0000:75:00.0 C            C            C            SELF         C            C            C            C
0000:8e:00.0 C            C            C            C            SELF         C            C            C
0000:a7:00.0 C            C            C            C            C            SELF         C            C
0000:dc:00.0 C            C            C            C            C            C            SELF         C
0000:f4:00.0 C            C            C            C            C            C            C            SELF
ATOMICS TABLE:
             0000:0d:00.0 0000:26:00.0 0000:5d:00.0 0000:75:00.0 0000:8e:00.0 0000:a7:00.0 0000:dc:00.0 0000:f4:00.0
0000:0d:00.0 SELF         64,32        64,32        64,32        64,32        64,32        64,32        64,32
0000:26:00.0 64,32        SELF         64,32        64,32        64,32        64,32        64,32        64,32
0000:5d:00.0 64,32        64,32        SELF         64,32        64,32        64,32        64,32        64,32
0000:75:00.0 64,32        64,32        64,32        SELF         64,32        64,32        64,32        64,32
0000:8e:00.0 64,32        64,32        64,32        64,32        SELF         64,32        64,32        64,32
0000:a7:00.0 64,32        64,32        64,32        64,32        64,32        SELF         64,32        64,32
0000:dc:00.0 64,32        64,32        64,32        64,32        64,32        64,32        SELF         64,32
0000:f4:00.0 64,32        64,32        64,32        64,32        64,32        64,32        64,32        SELF
DMA TABLE:
             0000:0d:00.0 0000:26:00.0 0000:5d:00.0 0000:75:00.0 0000:8e:00.0 0000:a7:00.0 0000:dc:00.0 0000:f4:00.0
0000:0d:00.0 SELF         T            T            T            T            T            T            T
0000:26:00.0 T            SELF         T            T            T            T            T            T
0000:5d:00.0 T            T            SELF         T            T            T            T            T
0000:75:00.0 T            T            T            SELF         T            T            T            T
0000:8e:00.0 T            T            T            T            SELF         T            T            T
0000:a7:00.0 T            T            T            T            T            SELF         T            T
0000:dc:00.0 T            T            T            T            T            T            SELF         T
0000:f4:00.0 T            T            T            T            T            T            T            SELF
BI-DIRECTIONAL TABLE:
             0000:0d:00.0 0000:26:00.0 0000:5d:00.0 0000:75:00.0 0000:8e:00.0 0000:a7:00.0 0000:dc:00.0 0000:f4:00.0
0000:0d:00.0 SELF         T            T            T            T            T            T            T
0000:26:00.0 T            SELF         T            T            T            T            T            T
0000:5d:00.0 T            T            SELF         T            T            T            T            T
0000:75:00.0 T            T            T            SELF         T            T            T            T
0000:8e:00.0 T            T            T            T            SELF         T            T            T
0000:a7:00.0 T            T            T            T            T            SELF         T            T
0000:dc:00.0 T            T            T            T            T            T            SELF         T
0000:f4:00.0 T            T            T            T            T            T            T            SELF
```

```sh
amd-smi firmware
GPU: 0
    FW_LIST:
        FW 0:
            FW_ID: CP_MEC1
            FW_VERSION: 41
        FW 1:
            FW_ID: CP_MEC2
            FW_VERSION: 41
        FW 2:
            FW_ID: RLC
            FW_VERSION: 43
        FW 3:
            FW_ID: SDMA0
            FW_VERSION: 12
        FW 4:
            FW_ID: SDMA1
            FW_VERSION: 12
        FW 5:
            FW_ID: VCN
            FW_VERSION: 09.10.90.31
        FW 6:
            FW_ID: RLC_RESTORE_LIST_GPM_MEM
            FW_VERSION: 4
        FW 7:
            FW_ID: RLC_RESTORE_LIST_SRM_MEM
            FW_VERSION: 4
        FW 8:
            FW_ID: RLC_RESTORE_LIST_CNTL
            FW_VERSION: 4
        FW 9:
            FW_ID: PSP_SOSDRV
            FW_VERSION: 00.45.00.29
        FW 10:
            FW_ID: TA_RAS
            FW_VERSION: 1B.45.00.0A
        FW 11:
            FW_ID: TA_XGMI
            FW_VERSION: 20.00.00.14
        FW 12:
            FW_ID: PM
            FW_VERSION: 04.86.15.106
        FW 13:
            FW_ID: PLDM_BUNDLE
            FW_VERSION: 01.25.17.07
```

```sh
numactl --hardware
```

```
numactl --hardware
available: 2 nodes (0-1)
node 0 cpus: 0 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19 20 21 22 23 24 25 26 27 28 29 30 31 32 33 34 35 36 37 38 39 40 41 42 43 44 45 46 47 48 49 50 51 52 53 54 55 56 57 58 59 60 61 62 63 128 129 130 131 132 133 134 135 136 137 138 139 140 141 142 143 144 145 146 147 148 149 150 151 152 153 154 155 156 157 158 159 160 161 162 163 164 165 166 167 168 169 170 171 172 173 174 175 176 177 178 179 180 181 182 183 184 185 186 187 188 189 190 191
node 0 size: 1547872 MB
node 0 free: 1352749 MB
node 1 cpus: 64 65 66 67 68 69 70 71 72 73 74 75 76 77 78 79 80 81 82 83 84 85 86 87 88 89 90 91 92 93 94 95 96 97 98 99 100 101 102 103 104 105 106 107 108 109 110 111 112 113 114 115 116 117 118 119 120 121 122 123 124 125 126 127 192 193 194 195 196 197 198 199 200 201 202 203 204 205 206 207 208 209 210 211 212 213 214 215 216 217 218 219 220 221 222 223 224 225 226 227 228 229 230 231 232 233 234 235 236 237 238 240 241 242 243 244 245 246 247 248 249 250 251 252 253 254 255
node 1 size: 1548129 MB
node 1 free: 1456225 MB
node distances:
node   0   1 
  0:  10  32 
  1:  32  10 
```

## AINIC (Pesando)

```sh
sudo nicctl show port

NIC  : 42424650-4c32-3532-3430-353433000000 (0000:05:00.0)

Port : 04908139-90d8-4242-4242-000011010000 (eth1/1)
  Spec:
    Ifindex                                  : 0x11010000
    Type                                     : ETH
    speed                                    : 400G
    Admin state                              : UP
    FEC type                                 : RS
    Pause type                               : PFC
    Number of lanes                          : 4
    MTU                                      : 9000
    TX pause                                 : enabled
    RX pause                                 : enabled
    Auto negotiation                         : enabled
-------------------------------------------------------------------------------------

```
## RDMA Link
To see the interfaces run the command “rdma link”. The front-end network is mlx5_0 (eth0). The RoCE RDMA interfaces are the ionic* interfaces. The state should show up as “ACTIVE” and the physical_state as “LINK_UP”. 
```sh
rdma_link

link ionic_4/1 state ACTIVE physical_state LINK_UP netdev enp112s0 
link ionic_0/1 state ACTIVE physical_state LINK_UP netdev enp8s0 
link ionic_3/1 state ACTIVE physical_state LINK_UP netdev enp88s0 
link ionic_2/1 state ACTIVE physical_state LINK_UP netdev enp33s0 
link ionic_9/1 state ACTIVE physical_state LINK_UP netdev enp239s0 
link ionic_5/1 state ACTIVE physical_state LINK_UP netdev enp137s0 
link ionic_8/1 state ACTIVE physical_state LINK_UP netdev enp215s0 
link ionic_7/1 state ACTIVE physical_state LINK_UP netdev enp162s0 
link mlx5_0/1 state ACTIVE physical_state LINK_UP netdev ens9np0 
link mlx5_1/1 state ACTIVE physical_state LINK_UP netdev ens49np1 
```

## IB Write
The following tests allows users to verify performance and to diagnose and troubleshoot network and RDMA related issues.

E.g Running ib writes against ionic_7

Server Side:
```numactl -N 0 -m 0 ib_write_bw -d ionic_7  -i 1  -q 4 -a --report_gbits --tx-depth 1024```
Client Side:
```numactl -N 0 -m 0 ib_write_bw -d ionic_7  -i 1  -q 4 -a --report_gbits --tx-depth 1024 <svr_ip>```

Expected Performance:
```bash
---------------------------------------------------------------------------------------
 #bytes     #iterations    BW peak[Gb/sec]    BW average[Gb/sec]   MsgRate[Mpps]
 8388608    20000            391.34             391.34 		     0.005831
---------------------------------------------------------------------------------------
```

# Further Reading & Support

This section has additional reference material which you may find useful.

## AMD Technical Documents

1. [AMD ROCM Software Compatibility Matrix](https://rocm.docs.amd.com/en/latest/compatibility/compatibility-matrix.html1)
2. [AMD ROcm Software Examples](https://github.com/ROCm/rocm-examples)
3. [AMD Kubernetes GPU Operator](https://github.com/ROCm/gpu-operator)

## OCI AMD 355X Technical Docs & Supported Solutions

1. [OCI HPC SLURM Deployed Terraform Stack](https://github.com/oracle-quickstart/oci-hpc)
2. [OCI HPC OKE Deployment Stack](https://github.com/oracle-quickstart/oci-hpc-oke)
