# OCI GPU Quick Starts: AMD MI355X Instinct GPUs
This document provides hardware specifications, supported OS images, onboarding verification, sample benchmarks, and best-practices for OCI deployments using the AMD MI355X GPU bare-metal shape.

MI355X shapes represent a specialized class of heterogeneous accelerated compute in Oracle Cloud Infrastructure (OCI), built around the AMD Instinct MI355X GPU. These shapes provide 8 MI355X GPUs per node, each with 288 GiB of high-bandwidth HBM memory, targeting large-scale AI/ML workloads and HPC.

MI355X systems leverage AMD’s XGMI (Inter-GPU Express Global Memory Interconnect) for direct GPU-to-GPU connectivity within each node. On a typical MI355X system, all 8 GPUs are fully connected via XGMI, providing a single-hop, high-bandwidth, low-latency peer-to-peer path between any pair of GPUs. This topology is symmetrical, meaning all GPUs enjoy equal connectivity for workloads that depend on frequent collective or point-to-point communication.

## Table of Contents
* [Onboarding](#onboarding)
  * [Hardware Specifications](#hardware-specifications)
  * [Supported OS Images](#supported-os-images)
* [Getting Started & Validations](#getting-started--validations)
  * [Bare-metal node validations](#bare-metal-node-validations)
  * [Container based validations](#container-based-validations)
    * [Hello World ROCm PyTorch Container](#hello-world-rocm-pytorch-container)
  * [Kubernetes based validations](#kubernetes-based-validations)
* [Performance](#performance)
  * [RCCL Benchmarks](#rccl-benchmarks)
  * [LLM Inference Performance Numbers](#llm-lnference-performance-numbers)
* [Healthcheck](#health-checks)
* [Troubleshooting](#troubleshooting)
* [Further Reading & Support](#further-reading--support)


## Onboarding

All details to help you get started with AMD MI355X OCI Bare-Metal GPUs.

### Hardware Specifications

| Shape Name        | GPU Model     | GPUs/Node | GPU Memory | GPU Memory Total |             CPU           | # of CPUs | System Memory | Local Storage	 | Host NIC | RDMA (ROCe) NICs |
|-------------------|---------------|-----------|------------|----------------- |---------------------------|-----------|---------------|----------------|----------|-----------|
| BM.GPU.MI355x.v0.8| AMD MI355X    |   8       | 288 GB/GPU |8x288GB = 2.3 TB  | 2 x AMD TURIN 9575F 3.3Ghz| 128 Cores | 3TB DDR	| 8xNVMe 7.68TB/disk = 61.44TB |NVIDIA CX-7 2x200GBps = 400GBps| AMD Pensando™ AI NIC, 8 1x400Gb/s Ethernet=3.2 Tb/s
| BM.GPU.MI355x.v1.8| AMD MI355X    |   8       | 288 GB/GPU |8x288GB = 2.3 TB  | 2 x AMD TURIN 9575F 3.3Ghz| 128 Cores | 3TB DDR	| 8xNVMe 7.68TB/disk = 61.44TB |NVIDIA CX-7 2x200GBps = 400GBps| Mellanox ConnectX-7, 8 1x400Gb/s=3.2 Tb/s

See the [OCI Compute Shapes Docs](https://docs.oracle.com/en-us/iaas/Content/Compute/References/computeshapes.htm) for up-to-date details.


### Supported OS Images
Always use Oracle-provided or organizationally-approved images for security and supportability. See OS Images Table for region-specific OCID's and version info.

Use Oracle-provided or approved images for security and supportability. To build your images using packer use this clone this repo and run these commands [OCI HPC Images GitHub Repo](https://github.com/oracle-quickstart/oci-hpc-images/blob/main/README.md).

| OS Version        | Image Packer Build Details       | OCI Platform Image Link                                                                        | Driver Versions | Build & Dependency Status | 
|-------------------|-------------------------------|------------------------------------------------------------------------------------------------------------|--------------|--------------------------|
| OCI GPU AI Image with Oracle Linux 9.5  | `OracleLinux-9/Oracle-Linux-9.5-RHCK-DOCA-OFED-3.0.0-AMD-ROCM-700.pkr.hcl`| [PAR Link](https://cloudmarketplace.oracle.com/marketplace/en_US/listing/134254210) | ROCM 7.0.2, OFED 24.10-1.1.4.0, OCA 1.52, HPC-X 2.23 | NA
| OCI GPU AI Image with Ubuntu Linux 24.04  | `Ubuntu-24/Canonical-Ubuntu-24.04-DOCA-OFED-3.0.0-AMD-ROCM-710.pkr.hcl`| [PAR Link](https://objectstorage.us-chicago-1.oraclecloud.com/p/Bv7qO-ALCgpnHE-0whrjmfMIENDz8DFRf30urxyVZ2hjSRQyTyqKwdXV316QxSWy/n/iduyx1qnmway/b/oci-hpc-images/o/Canonical-Ubuntu-24.04-2025.10.31-0-OCA-DOCA-OFED-3.1.0-AMD-ROCM-710-2025.12.05) | ROCM 7.1.0, RCCL 2.26.6, OFED: 28.40.1202, Oracle Cloud Agent 1.54.0+, HPCX 2.22+ | ![Build](/media/icons/build-passing.svg) ![Build](/media/icons/dependencies.svg) 
| OCI GPU AI Image with Ubuntu Linux 22.04  | `Ubuntu-22/Canonical-Ubuntu-22.04-DOCA-OFED-3.0.0-AMD-ROCM-710.pkr.hcl`| [PAR Link](https://objectstorage.us-chicago-1.oraclecloud.com/p/ADstfRticRqOIwBHKo9HoFOheSC9mzHTx2mdx_j8e-aB9tGdPAVGsm1jzVEfs27G/n/iduyx1qnmway/b/oci-hpc-images/o/Canonical-Ubuntu-22.04-2025.10.31-0-OCA-DOCA-OFED-3.1.0-AMD-ROCM-710-2025.12.05) |  ROCM 7.1, OFED 24.10-1.1.4.0, OCA 1.52, HPC-X 2.23 | ![Build](/media/icons/build-passing.svg) ![Build](/media/icons/dependencies.svg) 
| OCI GPU AI Image with Ubuntu Linux 22.04  | `Ubuntu-22/Canonical-Ubuntu-24.04-DOCA-OFED-3.0.0-AMD-ROCM-641.pkr.hcl`| [PAR Link](https://objectstorage.us-chicago-1.oraclecloud.com/p/HeINbvoAL3xKI_rvUyMdnW_j-CCqCsqsUZ76arKCkW-4iMeQRAt_Wq_5HDZlEBSs/n/iduyx1qnmway/b/oci-hpc-images/o/Canonical-Ubuntu-22.04-2025.10.31-0-DOCA-OFED-3.0.0-AMD-ROCM-641-2025.12.05) |  ROCM 6.4, OFED 24.10-1.1.4.0, OCA 1.52, HPC-X 2.23 | ![Build](/media/icons/build-passing.svg) ![Build](/media/icons/dependencies.svg) 

## Getting Started & Validations

Follow the details here to download the OS images into your tenancy. Once downloaded choose this custom image to deploy your bare metal instance. More details here on your deployment options. Once successfully booted up check for the below.

### Bare-metal node validations

**1. Confirm Hardware & Drivers**
```sh
amd-smi
```
```
============================================= ROCm System Management Interface =============================================
======================================================= Concise Info =======================================================
Device  Node  IDs              Temp        Power     Partitions          SCLK     MCLK     Fan  Perf  PwrCap   VRAM%  GPU%  
              (DID,     GUID)  (Junction)  (Socket)  (Mem, Compute, ID)                                                     
============================================================================================================================
0       3     0x75a3,   21010  44.0°C      257.0W    NPS1, SPX, 0        797Mhz   2000Mhz  0%   auto  1400.0W  17%    1%    
1       5     0x75a3,   56525  44.0°C      272.0W    NPS1, SPX, 0        1027Mhz  2000Mhz  0%   auto  1400.0W  17%    2%    
2       4     0x75a3,   28206  45.0°C      260.0W    NPS1, SPX, 0        771Mhz   2000Mhz  0%   auto  1400.0W  17%    1%    
3       2     0x75a3,   28720  43.0°C      256.0W    NPS1, SPX, 0        496Mhz   2000Mhz  0%   auto  1400.0W  17%    1%    
4       7     0x75a3,   25266  41.0°C      259.0W    NPS1, SPX, 0        712Mhz   2000Mhz  0%   auto  1400.0W  17%    1%    
5       9     0x75a3,   20206  45.0°C      288.0W    NPS1, SPX, 0        2386Mhz  2000Mhz  0%   auto  1400.0W  17%    1%    
6       8     0x75a3,   16143  43.0°C      285.0W    NPS1, SPX, 0        2404Mhz  2000Mhz  0%   auto  1400.0W  17%    1%    
7       6     0x75a3,   8465   44.0°C      260.0W    NPS1, SPX, 0        722Mhz   2000Mhz  0%   auto  1400.0W  17%    1%    
============================================================================================================================
=================================================== End of ROCm SMI Log ====================================================
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
### Container based validations

#### Hello World ROCm PyTorch Container

You will need docker installed on the OS image before you can run this command. More on different functions that can be run [here](https://rocm.docs.amd.com/projects/install-on-linux/en/latest/install/3rd-party/pytorch-install.html#using-docker-with-pytorch-pre-installed)

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
### Kubernetes based validations
**OKE & Containers Getting Started**
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
## Performance 

The section below has the base line numbers achieved and how to reproduce this with MI355X. When you run the same there is typically a variance of 5%-8% which is acceptable. 

### RCCL Benchmarks
Collective communications results single node.

#### All Reduce

```bash
/opt/rccl-tests/build/all_reduce_perf -b 512k -e 16G -f 2 -g 8
```
Results

```bash
# Collective test starting: all_reduce_perf
# nThread 1 nGpus 8 minBytes 524288 maxBytes 17179869184 step: 2(factor) warmup iters: 5 iters: 20 agg iters: 1 validation: 1 graph: 0
#
rccl-tests: Version HEAD:e1b8a3a
# Using devices
#  Rank  0 Group  0 Pid   9632 on   GPU-6403 device  0 [0000:72:00] AMD Instinct MI355X
#  Rank  1 Group  0 Pid   9632 on   GPU-6403 device  1 [0000:0a:00] AMD Instinct MI355X
#  Rank  2 Group  0 Pid   9632 on   GPU-6403 device  2 [0000:5a:00] AMD Instinct MI355X
#  Rank  3 Group  0 Pid   9632 on   GPU-6403 device  3 [0000:23:00] AMD Instinct MI355X
#  Rank  4 Group  0 Pid   9632 on   GPU-6403 device  4 [0000:f1:00] AMD Instinct MI355X
#  Rank  5 Group  0 Pid   9632 on   GPU-6403 device  5 [0000:8b:00] AMD Instinct MI355X
#  Rank  6 Group  0 Pid   9632 on   GPU-6403 device  6 [0000:d9:00] AMD Instinct MI355X
#  Rank  7 Group  0 Pid   9632 on   GPU-6403 device  7 [0000:a4:00] AMD Instinct MI355X
#                                                              out-of-place                       in-place          
#       size         count      type   redop    root     time   algbw   busbw #wrong     time   algbw   busbw #wrong
#        (B)    (elements)                               (us)  (GB/s)  (GB/s)            (us)  (GB/s)  (GB/s)       
  8589934592    2147483648     float     sum      -1    36956  232.43  406.76      0    36911  232.72  407.26      0
 17179869184    4294967296     float     sum      -1    73913  232.43  406.76      0    73917  232.42  406.73      0
# Errors with asterisks indicate errors that have exceeded the maximum threshold.
# Out of bounds values : 0 OK
# Avg bus bandwidth    : 273.886 
#
# Collective test concluded: all_reduce_perf
```
#### All 2 All

```bash
/opt/rccl-tests/build/alltoall_perf -b 512k -e 16G -f 2 -g 8
```
Results

```bash
# Collective test starting: alltoall_perf
# nThread 1 nGpus 8 minBytes 524288 maxBytes 17179869184 step: 2(factor) warmup iters: 5 iters: 20 agg iters: 1 validation: 1 graph: 0
#
rccl-tests: Version HEAD:e1b8a3a
# Using devices
#  Rank  0 Group  0 Pid   9878 on   GPU-6403 device  0 [0000:72:00] AMD Instinct MI355X
#  Rank  1 Group  0 Pid   9878 on   GPU-6403 device  1 [0000:0a:00] AMD Instinct MI355X
#  Rank  2 Group  0 Pid   9878 on   GPU-6403 device  2 [0000:5a:00] AMD Instinct MI355X
#  Rank  3 Group  0 Pid   9878 on   GPU-6403 device  3 [0000:23:00] AMD Instinct MI355X
#  Rank  4 Group  0 Pid   9878 on   GPU-6403 device  4 [0000:f1:00] AMD Instinct MI355X
#  Rank  5 Group  0 Pid   9878 on   GPU-6403 device  5 [0000:8b:00] AMD Instinct MI355X
#  Rank  6 Group  0 Pid   9878 on   GPU-6403 device  6 [0000:d9:00] AMD Instinct MI355X
#  Rank  7 Group  0 Pid   9878 on   GPU-6403 device  7 [0000:a4:00] AMD Instinct MI355X
#
#                                                              out-of-place                       in-place          
#       size         count      type   redop    root     time   algbw   busbw #wrong     time   algbw   busbw #wrong
#        (B)    (elements)                               (us)  (GB/s)  (GB/s)            (us)  (GB/s)  (GB/s)       
 8589934592     268435456     float    none      -1    20837  412.23  360.71      0    20817  412.64  361.06    N/A
 17179869184     536870912     float    none      -1    41593  413.05  361.41      0    41618  412.80  361.20    N/A

 # Errors with asterisks indicate errors that have exceeded the maximum threshold.
# Out of bounds values : 0 OK
# Avg bus bandwidth    : 236.187 
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

## Health Checks

The [AMDs ROCm Validation Suite (RVS)](https://rocm.docs.amd.com/projects/ROCmValidationSuite/en/latest/) is a system validation and diagnostics tool for monitoring, stress testing, detecting, and troubleshooting issues that affect the functionality and performance of AMD GPUs operating in a high-performance/AI/ML computing environment. RVS is enabled using the ROCm software stack on a compatible software and hardware platform and is part of the OS images from OCI. 

```bash
cd /opt/rocm/bin
./rvs
```
Result
```bash
+=====================================================================+
|                 ROCm Validation Suite (RVS) Summary                 |
+=====================================================================+
|                           System Overview                           |
+---------------------------------------------------------------------+
| Operating System                 | Ubuntu 24.04.3 LTS               |
| RVS version                      | 1.3.0                            |
| ROCm version                     | 7.0.2-56                         |
| amdgpu version                   | 6.14.14                          |
| GPUs                             | 8                                |
+---------------------------------------------------------------------+
| GPU Name - GPU ID                |                                  |
| ID - Node ID - BDF               |                                  |
+---------------------------------------------------------------------+
| AMD Instinct MI355X - 21010      | AMD Instinct MI355X - 56525      |
| 0 - 3 - 0000:0a:00.0             | 1 - 5 - 0000:23:00.0             |
+---------------------------------------------------------------------+
| AMD Instinct MI355X - 28206      | AMD Instinct MI355X - 28720      |
| 2 - 4 - 0000:5a:00.0             | 3 - 2 - 0000:72:00.0             |
+---------------------------------------------------------------------+
| AMD Instinct MI355X - 25266      | AMD Instinct MI355X - 20206      |
| 4 - 7 - 0000:8b:00.0             | 5 - 9 - 0000:a4:00.0             |
+---------------------------------------------------------------------+
| AMD Instinct MI355X - 16143      | AMD Instinct MI355X - 8465       |
| 6 - 8 - 0000:d9:00.0             | 7 - 6 - 0000:f1:00.0             |
+=====================================================================+
| Action Name                      | Module         | Result          |
+=====================================================================+
| action_1                         | GPUP           | PASS            |
| action_2                         | PEQT           | PASS            |
| action_3                         | PEBB           | PASS            |
| action_4                         | PBQT           | PASS            |
| action_5                         | IET            | PASS            |
| action_6                         | GST            | PASS            |
| action_7                         | BABEL          | PASS            |
| action_8                         | MEM            | PASS            |
| action_9                         | PESM           | PASS            |
+---------------------------------------------------------------------+
[RESULT] [ 82145.821495] [module_terminate] PCIe monitoring ended after wait duration.
```
## Troubleshooting

The recommended tool for identifying the issues is ROCM AGFHC (AMD GPU Field Health Check). This tool is only made available to NDA OCI customers. Reach out to us to us to get access to this tool in container format. [More about AGHFC.](https://instinct.docs.amd.com/projects/gpu-operator/en/latest/test/agfhc.html).

We recommend you use OCI GPU Scanner to manage the OKE cluster health and GPU health. GPU scanner also provies continu=ious health check and auto-remidiation. [OCI GPU Scanner - Cluster Health Management Solution](https://github.com/oracle-quickstart/oci-gpu-scanner)

## Further Reading & Support

### AMD Technical Documents
1. [AMD ROCM Software Compatibility Matrix](https://rocm.docs.amd.com/en/latest/compatibility/compatibility-matrix.html1)
2. [AMD ROcm Software Examples](https://github.com/ROCm/rocm-examples)
3. [AMD Kubernetes GPU Operator](https://github.com/ROCm/gpu-operator)

### OCI AMD 355X Blogs
1. [OCI AMD MI355X - GA Announcement](https://www.oracle.com/news/announcement/ai-world-oracle-and-amd-expand-partnership-to-help-customers-achieve-next-generation-ai-scale-2025-10-14/)

### OCI AMD 355X Technical Docs & Supported Solutions

1. [OCI GPU Scanner - Cluster Health Management Solution](https://github.com/oracle-quickstart/oci-gpu-scanner)
2. [OCI HPC SLURM Deployed Terraform Stack](https://github.com/oracle-quickstart/oci-hpc)
3. [OCI HPC OKE Deployment Stack](https://github.com/oracle-quickstart/oci-hpc-oke)