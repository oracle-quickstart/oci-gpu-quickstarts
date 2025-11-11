# OCI GPU Quick Start: AMD MI355X
This document provides hardware specifications, supported OS images, onboarding verification, sample benchmarks, and best-practices for OCI deployments using the AMD MI355X GPU bare-metal shape.

MI355X shapes represent a specialized class of heterogeneous accelerated compute in Oracle Cloud Infrastructure (OCI), built around the AMD Instinct MI355X GPU. These shapes provide 8 MI355X GPUs per node, each with 288 GiB of high-bandwidth HBM memory, targeting large-scale AI/ML workloads and HPC.

MI355X systems leverage AMD’s XGMI (Inter-GPU Express Global Memory Interconnect) for direct GPU-to-GPU connectivity within each node. On a typical MI355X system, all 8 GPUs are fully connected via XGMI, providing a single-hop, high-bandwidth, low-latency peer-to-peer path between any pair of GPUs. This topology is symmetrical, meaning all GPUs enjoy equal connectivity for workloads that depend on frequent collective or point-to-point communication.

## Table of Contents
* [Onboarding](#onboarding)
  * [Hardware Specifications](#hardware-specifications)
  * [Supported OS Images](#supported-os-images)
  * [Getting Started]()
4. [Performance Benchmarks](#health-check--baseline-benchmarks)
5. [NCCL & Model Inference Performance](#nccl--model-inference-performance)
6. [Custom OS Image Creation with Packer](#custom-os-image-creation-with-packer)
7. [OKE GPU Getting Started](#oke-gpu-getting-started)
8. [Further Reading & Support](#further-reading--support)

## Onboarding

All details to help you get started with AMD MI355X OCI Bare-Metal GPUs.

### Hardware Specifications

| Shape Name        | GPU Model     | GPUs/Node | GPU Memory | GPU Memory Total |             CPU           | # of CPUs | System Memory | Local Storage	 | Host NIC | RDMA NICs |
|-------------------|---------------|-----------|------------|----------------- |---------------------------|-----------|---------------|----------------|----------|-----------|
| BM.GPU.MI355x.v0.8| AMD MI355X    |   8       | 288 GB/GPU |8x288GB = 2.3 TB  | 2 x AMD TURIN 9575F 3.3Ghz| 128 Cores | 3TB DDR	| 8xNVMe 7.68TB/disk – 61.44TB |NVIDIA CX-7 2x200GBps = 400GBps| AMD Pensando™ Pollara 400 AI NIC, 8 1x400Gb/s Ethernet (3.2 Tb/s total bandwidth, RoCE + RDMA)
| BM.GPU.MI355x.v1.8| AMD MI355X    |   8       | 288 GB/GPU |8x288GB = 2.3 TB  | 2 x AMD TURIN 9575F 3.3Ghz| 128 Cores | 3TB DDR	| 8xNVMe 7.68TB/disk – 61.44TB |NVIDIA CX-7 2x200GBps = 400GBps| Mellanox Technologies MT2910 Family [ConnectX-7], 8 1x400Gb/s NVIDIA CX-7 Ethernet (3.2 Tb/s total bandwidth, RoCE + RDMA)

See the [OCI Compute Shapes Docs](https://docs.oracle.com/en-us/iaas/Content/Compute/References/computeshapes.htm) for up-to-date details.


### Supported OS Images
Always use Oracle-provided or organizationally-approved images for security and supportability. See OS Images Table for region-specific OCIDs and version info.

Use Oracle-provided or approved images for security and supportability.

| OS Version        | Image OCID PAR Link (ORD example)       | Download/Marketplace Link                                                                        | GPU Driver Version | Build & Dependency Status | 
|-------------------|-------------------------------|------------------------------------------------------------------------------------------------------------|--------------|--------------------------|
| Oracle Linux 9.3  | `ocid1.image.oc1.iad.aaaaaaaasgnnzqa7os7eljhv72stlgz3r6l64mhfbhxvriwtlvq6ulq75xyq`| [Oracle Linux - GPU Cluster Networking Image OL9.3 (ROCM)](https://cloudmarketplace.oracle.com/marketplace/en_US/listing/134254210) | ROCM 6.3.2, OFED 24.10-1.1.4.0, OCA 1.52, HPC-X 2.23 | NA
| Ubuntu Linux 24.04  | `ocid1.image.oc1.iad.aaaaaaaasgnnzqa7os7eljhv72stlggghh7j6l64mhfbhxvriwtlvq6ulq75xyq`| NA | ROCM 7.0.2, RCCL 2.26.6, OFED: 28.40.1202, Oracle Cloud Agent 1.54.0+, HPCX 2.22+ | ![Build](/media/icons/build-passing.svg) ![Build](/media/icons/dependencies.svg) 
| Ubuntu Linux 22.04  | `ocid1.image.oc1.iad.aaaaaaaasgnnzqa7os7eljhv72stlgz3r6l64mhfbhxvriwtlvq6ulq75xyq`| NA |  ROCM 6.4, OFED 24.10-1.1.4.0, OCA 1.52, HPC-X 2.23 | ![Build](/media/icons/build-passing.svg) ![Build](/media/icons/dependencies.svg) 

### Getting Started

Follow the details here to download the OS images into your tenancy. Once downloaded choose this custom image to deploy your bare metal instance. More details here on your deployment options. Once successfully booted up check for the below.

### Quick Verification

**1. Confirm Hardware & Drivers**
```sh
amd-smi
```
![amd-smi-mi355x](/amd/MI355X/media/amd-smi-355x-display.png "AMD SMI Print")

```sh
numactl --hardware
```
![numactl--mi355x](/amd/MI355X/media/amd-355x-numactl-display.png "Numa Ctl Print")

### Hello World Rocm PyTorch Container

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
### OKE & Containers Getting Started
1. Create OKE Cluster via OCI Console or CLI.
2. Add GPU Node Pool: Choose MI355X shape, use golden OS image.
3. Install AMD Device Plugin:
```bash
kubectl apply -f https://github.com/AMD/k8s-device-plugin/raw/main/AMD-device-plugin.yml
```
4. Run GPU-powered Kubernetes workloads:
Example pod resource spec:
```yaml
resources:
  limits:
    AMD.com/gpu: 1
```
5. Verify with Hello World CUDA container pod.
See detailed instructions in OKE Onboarding Guide .

## Performance 

The section below has the base line numbers achieved and how to reproduce this with MI355X.

### Baseline RCCL Benchmarks
Collective communications results single node.

### All Reduce

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
### All 2 All

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

### Training Numbers

Example output:
```shell
Train Step Timing and TFLOPS Analysis (iterations 35-44)
================================================================================
Experiment                                                                                  Status Time Mean (s) Time Std (s) TFLOPS_per_GPU Mean TFLOPS_per_GPU Std
------------------------------------------------------------------------------------------ -------- ------------- ------------ ------------------- ------------------
pretrain_nemotron4_340b_fp8_gpus128_tp8_pp4_cp1_vp12_mbs1_gbs512_389757                     Success        21.542        0.010             1600.74               0.78
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


## Further Reading & Support

### AMD Technical Documents
1. [AMD ROCM Software Compatibility Matrix](https://rocm.docs.amd.com/en/latest/compatibility/compatibility-matrix.html1)
2. [AMD ROcm Software Examples](https://github.com/ROCm/rocm-examples)
3. [AMD Kubernetes GPU Operator](https://github.com/ROCm/gpu-operator)

### OCI AMD 355X Blogs

### OCI AMD 355X Technical Docs & Supported Solutions

1. [OCI GPU Scanner - Cluster Health Management Solution](https://github.com/oracle-quickstart/oci-gpu-scanner)
2. [OCI HPC SLURM Deployed Terraform Stack](https://github.com/oracle-quickstart/oci-hpc)
3. [OCI HPC OKE Deployment Stack](https://github.com/oracle-quickstart/oci-hpc-oke)