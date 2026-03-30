> [!WARNING]
> **INTERNAL USE ONLY — NOT FOR PUBLIC DISTRIBUTION**
> This document has not been approved for public release. It must not be committed to a public repository or shared externally until this notice is explicitly removed by the document owner prior to merge.

> [!CAUTION]
> The source material for this shape is marked as **IN PROGRESS**. Content may be incomplete or subject to change. Verify with the document owner before use.

# OCI GPU Quick Start: NVIDIA B300

This document provides hardware specifications, supported OS images, onboarding verification, sample benchmarks, and best-practices for OCI deployments using the NVIDIA B300 GPU shape.

BM.GPU.B300.8 is a high-density GPU bare-metal shape built around eight NVIDIA B300 SXM6 AC GPUs, dual Intel Xeon Platinum 8592+ processors, 4 TB of DDR5 system memory, and high-bandwidth RoCE networking for both single-node and scale-out AI workloads.

# Table of Contents
* [Hardware Specifications](#hardware-specifications)
* [Recommended Operating Systems](#recommended-operating-systems)
    * [Recommended Software Version](#recommended-software-version)
    * [Custom OS Image Creation with Packer](#custom-os-image-creation-with-packer)
    * [Provided Images](#provided-images)
    * [Hello World Verification](#hello-world-verification)
* [Performance Benchmarks](#performance-benchmarks)
* [OKE GPU Getting Started](#oke-gpu-getting-started)
* [Troubleshooting](#troubleshooting)
* [Further Reading & Support](#further-reading--support)

# Hardware Specifications

| Shape Name | GPU Model | GPUs/Node | GPU Memory (GB/GPU) | GPU Memory Total | CPU | # of CPUs | System Memory | Local Storage | Host NIC | RDMA (ROCe) NICs |
|---|---|---|---|---|---|---|---|---|---|---|
| BM.GPU.B300.8 | B300 | 8 | 268 | 2.1 TB | 2 x Intel Xeon Platinum 8592+ @ 1.9 GHz | 128 Cores (256 with HT) | 4 TB DDR5 | 8 x 3.5 TB NVMe = 28 TB | 100 Gb/s | 8 x 2 x 400 Gb/s = 6.4 Tb/s |

See the [OCI Compute Shapes Docs](https://docs.oracle.com/en-us/iaas/Content/Compute/References/computeshapes.htm) for up-to-date details.

# Recommended Operating Systems

- Oracle Linux 9+
- Ubuntu Linux 24.04+

## Recommended Software Version

- DOCA OFED 3.2.1+
- NVIDIA Driver 590+ (Open)
- CUDA 13.1+
- NCCL 2.28.9+
- HPC-X 2.25.1+
- Oracle Cloud Agent 1.57.0+

## Custom OS Image Creation with Packer

To build your images using packer clone the OCI HPC Images repo and run the commands found there [OCI HPC Images GitHub Repo](https://github.com/oracle-quickstart/oci-hpc-images/blob/main/README.md).

## Provided Images

| OS Version | Image Packer Build Details | OCI Platform Image Link | Driver Versions | Build & Dependency Status |
|---|---|---|---|---|
| OCI GPU AI Image with Ubuntu Linux 22.04 | [`Canonical-Ubuntu-22.04-DOCA-OFED-3.2.1-GPU-580-OPEN-CUDA-13.0`](https://github.com/oracle-quickstart/oci-hpc-images/blob/main/images/Ubuntu-22/Canonical-Ubuntu-22.04-DOCA-OFED-3.2.1-GPU-580-OPEN-CUDA-13.0.pkr.hcl) | [PAR Link](https://objectstorage.ca-montreal-1.oraclecloud.com/p/AIo4CP0P_DlUelDlsWgGPWmY6FcBQzJWmmFyGKdY0epkh87a9Q3ndvFYycjIxTQ9/n/idxzjcdglx2s/b/images/o/Canonical-Ubuntu-22.04-2026.02.28-0-DOCA-OFED-3.2.1-GPU-580-OPEN-CUDA-13.0-2026.03.13-0) | NVIDIA OPEN 580, DOCA OFED 3.2.1, CUDA 13.0, OCA 1.57.0 | ![Build](/media/icons/build-passing.svg) ![Build](/media/icons/dependencies.svg) |
| OCI GPU AI Image with Ubuntu Linux 22.04 | [`Canonical-Ubuntu-22.04-DOCA-OFED-3.2.1-GPU-590-OPEN-CUDA-13.1`](https://github.com/oracle-quickstart/oci-hpc-images/blob/main/images/Ubuntu-22/Canonical-Ubuntu-22.04-DOCA-OFED-3.2.1-GPU-590-OPEN-CUDA-13.1.pkr.hcl) | [PAR Link](https://objectstorage.ca-montreal-1.oraclecloud.com/p/AIo4CP0P_DlUelDlsWgGPWmY6FcBQzJWmmFyGKdY0epkh87a9Q3ndvFYycjIxTQ9/n/idxzjcdglx2s/b/images/o/Canonical-Ubuntu-22.04-2026.02.28-0-DOCA-OFED-3.2.1-GPU-590-OPEN-CUDA-13.1-2026.03.13-0) | NVIDIA OPEN 590, DOCA OFED 3.2.1, CUDA 13.1, OCA 1.57.0 | ![Build](/media/icons/build-passing.svg) ![Build](/media/icons/dependencies.svg) |
| OCI GPU AI Image with Ubuntu Linux 24.04 | [`Canonical-Ubuntu-24.04-6.8-DOCA-OFED-3.2.1-GPU-580-OPEN-CUDA-13.0`](https://github.com/oracle-quickstart/oci-hpc-images/blob/main/images/Ubuntu-24/Canonical-Ubuntu-24.04-6.8-DOCA-OFED-3.2.1-GPU-580-OPEN-CUDA-13.0.pkr.hcl) | [PAR Link](https://objectstorage.ca-montreal-1.oraclecloud.com/p/AIo4CP0P_DlUelDlsWgGPWmY6FcBQzJWmmFyGKdY0epkh87a9Q3ndvFYycjIxTQ9/n/idxzjcdglx2s/b/images/o/Canonical-Ubuntu-24.04-2026.02.28-0-6.8-DOCA-OFED-3.2.1-GPU-580-OPEN-CUDA-13.0-2026.03.13-0) | NVIDIA OPEN 580, DOCA OFED 3.2.1, CUDA 13.0, Kernel 6.8, OCA 1.57.0 | ![Build](/media/icons/build-passing.svg) ![Build](/media/icons/dependencies.svg) |
| OCI GPU AI Image with Ubuntu Linux 24.04 | [`Canonical-Ubuntu-24.04-6.8-DOCA-OFED-3.2.1-GPU-590-OPEN-CUDA-13.1`](https://github.com/oracle-quickstart/oci-hpc-images/blob/main/images/Ubuntu-24/Canonical-Ubuntu-24.04-6.8-DOCA-OFED-3.2.1-GPU-590-OPEN-CUDA-13.1.pkr.hcl) | [PAR Link](https://objectstorage.ca-montreal-1.oraclecloud.com/p/AIo4CP0P_DlUelDlsWgGPWmY6FcBQzJWmmFyGKdY0epkh87a9Q3ndvFYycjIxTQ9/n/idxzjcdglx2s/b/images/o/Canonical-Ubuntu-24.04-2026.02.28-0-6.8-DOCA-OFED-3.2.1-GPU-590-OPEN-CUDA-13.1-2026.03.13-0) | NVIDIA OPEN 590, DOCA OFED 3.2.1, CUDA 13.1, Kernel 6.8, OCA 1.57.0 | ![Build](/media/icons/build-passing.svg) ![Build](/media/icons/dependencies.svg) |
| OCI GPU AI Image with Ubuntu Linux 24.04 | [`Canonical-Ubuntu-24.04-6.14-DOCA-OFED-3.2.1-GPU-590-OPEN-CUDA-13.1`](https://github.com/oracle-quickstart/oci-hpc-images/blob/main/images/Ubuntu-24/Canonical-Ubuntu-24.04-6.14-DOCA-OFED-3.2.1-GPU-590-OPEN-CUDA-13.1.pkr.hcl) | [PAR Link](https://objectstorage.ca-montreal-1.oraclecloud.com/p/AIo4CP0P_DlUelDlsWgGPWmY6FcBQzJWmmFyGKdY0epkh87a9Q3ndvFYycjIxTQ9/n/idxzjcdglx2s/b/images/o/Canonical-Ubuntu-24.04-2026.02.28-0-6.14-DOCA-OFED-3.2.1-GPU-590-OPEN-CUDA-13.1-2026.03.13-0) | NVIDIA OPEN 590, DOCA OFED 3.2.1, CUDA 13.1, Kernel 6.14, OCA 1.57.0 | ![Build](/media/icons/build-passing.svg) ![Build](/media/icons/dependencies.svg) |
| OCI GPU AI Image with Oracle Linux 8 | [`Oracle-Linux-8.10-RHCK-DOCA-OFED-3.2.1-GPU-580-OPEN-CUDA-13.0`](https://github.com/oracle-quickstart/oci-hpc-images/blob/main/images/OracleLinux-8/Oracle-Linux-8.10-RHCK-DOCA-OFED-3.2.1-GPU-580-OPEN-CUDA-13.0.pkr.hcl) | [PAR Link](https://objectstorage.ca-montreal-1.oraclecloud.com/p/AIo4CP0P_DlUelDlsWgGPWmY6FcBQzJWmmFyGKdY0epkh87a9Q3ndvFYycjIxTQ9/n/idxzjcdglx2s/b/images/o/Oracle-Linux-8.10-2026.02.28-0-RHCK-DOCA-OFED-3.2.1-GPU-580-OPEN-CUDA-13.0-2026.03.13-0) | NVIDIA OPEN 580, DOCA OFED 3.2.1, CUDA 13.0, OCA 1.57.0 | ![Build](/media/icons/build-passing.svg) ![Build](/media/icons/dependencies.svg) |
| OCI GPU AI Image with Oracle Linux 8 | [`Oracle-Linux-8.10-RHCK-DOCA-OFED-3.2.1-GPU-590-OPEN-CUDA-13.1`](https://github.com/oracle-quickstart/oci-hpc-images/blob/main/images/OracleLinux-8/Oracle-Linux-8.10-RHCK-DOCA-OFED-3.2.1-GPU-590-OPEN-CUDA-13.1.pkr.hcl) | [PAR Link](https://objectstorage.ca-montreal-1.oraclecloud.com/p/AIo4CP0P_DlUelDlsWgGPWmY6FcBQzJWmmFyGKdY0epkh87a9Q3ndvFYycjIxTQ9/n/idxzjcdglx2s/b/images/o/Oracle-Linux-8.10-2026.02.28-0-RHCK-DOCA-OFED-3.2.1-GPU-590-OPEN-CUDA-13.1-2026.03.13-0) | NVIDIA OPEN 590, DOCA OFED 3.2.1, CUDA 13.1, OCA 1.57.0 | ![Build](/media/icons/build-passing.svg) ![Build](/media/icons/dependencies.svg) |
| OCI GPU AI Image with Oracle Linux 9 | [`Oracle-Linux-9.7-RHCK-DOCA-OFED-3.2.1-GPU-580-OPEN-CUDA-13.0`](https://github.com/oracle-quickstart/oci-hpc-images/blob/main/images/OracleLinux-9/Oracle-Linux-9.7-RHCK-DOCA-OFED-3.2.1-GPU-580-OPEN-CUDA-13.0.pkr.hcl) | [PAR Link](https://objectstorage.ca-montreal-1.oraclecloud.com/p/AIo4CP0P_DlUelDlsWgGPWmY6FcBQzJWmmFyGKdY0epkh87a9Q3ndvFYycjIxTQ9/n/idxzjcdglx2s/b/images/o/Oracle-Linux-9.7-2026.02.28-0-RHCK-DOCA-OFED-3.2.1-GPU-580-OPEN-CUDA-13.0-2026.03.13-0) | NVIDIA OPEN 580, DOCA OFED 3.2.1, CUDA 13.0, OCA 1.57.0 | ![Build](/media/icons/build-passing.svg) ![Build](/media/icons/dependencies.svg) |
| OCI GPU AI Image with Oracle Linux 9 | [`Oracle-Linux-9.7-RHCK-DOCA-OFED-3.2.1-GPU-590-OPEN-CUDA-13.1`](https://github.com/oracle-quickstart/oci-hpc-images/blob/main/images/OracleLinux-9/Oracle-Linux-9.7-RHCK-DOCA-OFED-3.2.1-GPU-590-OPEN-CUDA-13.1.pkr.hcl) | [PAR Link](https://objectstorage.ca-montreal-1.oraclecloud.com/p/AIo4CP0P_DlUelDlsWgGPWmY6FcBQzJWmmFyGKdY0epkh87a9Q3ndvFYycjIxTQ9/n/idxzjcdglx2s/b/images/o/Oracle-Linux-9.7-2026.02.28-0-RHCK-DOCA-OFED-3.2.1-GPU-590-OPEN-CUDA-13.1-2026.03.13-0) | NVIDIA OPEN 590, DOCA OFED 3.2.1, CUDA 13.1, OCA 1.57.0 | ![Build](/media/icons/build-passing.svg) ![Build](/media/icons/dependencies.svg) |

## Hello World Verification

Run `nvidia-smi` to verify that all eight GPUs are visible and healthy:

```bash
nvidia-smi
```

Expected output:

```text
+-----------------------------------------------------------------------------------------+
| NVIDIA-SMI 590.48.01    Driver Version: 590.48.01    CUDA Version: 13.1               |
+-----------------------------------------+------------------------+----------------------+
| GPU  Name                 Persistence-M | Bus-Id          Disp.A | Volatile Uncorr. ECC |
| Fan  Temp   Perf          Pwr:Usage/Cap |           Memory-Usage | GPU-Util  Compute M. |
|                                         |                        |               MIG M. |
|=========================================+========================+======================|
|   0  NVIDIA B300 SXM6 AC           On  |   00000000:14:00.0 Off |                    0 |
|  N/A   34C    P0             238W / 1100W |     0MiB / 275040MiB |      0%      Default |
|                                         |                        |           Disabled   |
+-----------------------------------------+------------------------+----------------------+
|   1  NVIDIA B300 SXM6 AC           On  |   00000000:32:00.0 Off |                    0 |
...
|   7  NVIDIA B300 SXM6 AC           On  |   00000000:DD:00.0 Off |                    0 |
+-----------------------------------------------------------------------------------------+
```

You should see all eight GPUs listed with no ECC errors and expected idle power draw.

# Performance Benchmarks

NVIDIA publishes [NCCL](https://developer.nvidia.com/nccl) as the primary collective communication library for multi-GPU AI and HPC workloads. The benchmark data collected for this shape includes both collective communication results and model-training performance, with larger message sizes being the most useful for cluster planning.

* [All Reduce - Single Node](#all-reduce---single-node)
* [All Reduce - 2 Nodes](#all-reduce---2-nodes)
* [All-to-All - Single Node](#all-to-all---single-node)
* [All-to-All - 2 Nodes](#all-to-all---2-nodes)
* [Model Inference Performance](#model-inference-performance)

## All Reduce - Single Node

```bash
. /opt/hpcx-v2.25.1-gcc-doca_ofed-ubuntu24.04-cuda13-x86_64/hpcx-init.sh
hpcx_load
/opt/oci-hpc/nccl-tests/build/all_reduce_perf -b 512k -e 8G -f 2 -g 8
```

```text
# Collective test starting: all_reduce_perf
# nThread 1 nGpus 8 minBytes 524288 maxBytes 8589934592 step: 2(factor) warmup iters: 1 iters: 20 agg iters: 1 validation: 1 graph: 0
NCCL version 2.28.9+cuda13.1
#
#                                                              out-of-place                       in-place
#       size         count      type   redop    root     time   algbw   busbw #wrong     time   algbw   busbw #wrong
#        (B)    (elements)                               (us)  (GB/s)  (GB/s)            (us)  (GB/s)  (GB/s)
# <smaller message sizes truncated>
 8589934592    2147483648     float     sum      -1    18006  477.05  834.84      0    17990  477.49  835.61      0
# Out of bounds values : 0 OK
# Avg bus bandwidth    : 440.608
#
# Collective test concluded: all_reduce_perf
```

## All Reduce - 2 Nodes

```text
NCCL version 2.28.9+cuda13.1
#
#                                                              out-of-place                       in-place
#       size         count      type   redop    root     time   algbw   busbw #wrong     time   algbw   busbw #wrong
#        (B)    (elements)                               (us)  (GB/s)  (GB/s)            (us)  (GB/s)  (GB/s)
# <smaller message sizes truncated>
 8589934592    2147483648     float     sum      -1    21012  408.82  766.54      0    21036  408.35  765.65      0
17179869184    4294967296     float     sum      -1    42007  408.98  766.84      0    41935  409.68  768.15      0
# Out of bounds values : 0 OK
# Avg bus bandwidth    : 405.346
#
# Collective test concluded: all_reduce_perf
```

## All-to-All - Single Node

```text
# Collective test starting: alltoall_perf
# nThread 1 nGpus 8 minBytes 524288 maxBytes 8589934592 step: 2(factor) warmup iters: 1 iters: 20 agg iters: 1 validation: 1 graph: 0
NCCL version 2.28.9+cuda13.1
#
#                                                              out-of-place                       in-place
#       size         count      type   redop    root     time   algbw   busbw #wrong     time   algbw   busbw #wrong
#        (B)    (elements)                               (us)  (GB/s)  (GB/s)            (us)  (GB/s)  (GB/s)
# <smaller message sizes truncated>
 8589934592   268435456     float    none      -1    11126  772.06  675.55      0    11206  766.58  670.76    N/A
# Out of bounds values : 0 OK
# Avg bus bandwidth    : 359.21
#
# Collective test concluded: alltoall_perf
```

## All-to-All - 2 Nodes

For guidance on running additional NCCL collective benchmarks on this GPU family, see the [NCCL user guide](https://docs.nvidia.com/deeplearning/nccl/user-guide/docs/index.html).

```text
# Collective test starting: alltoall_perf
# nThread 1 nGpus 1 minBytes 524288 maxBytes 8589934592 step: 2(factor) warmup iters: 1 iters: 50 agg iters: 1 validation: 1 graph: 0
#
#                                                              out-of-place                       in-place
#       size         count      type   redop    root     time   algbw   busbw #wrong     time   algbw   busbw #wrong
#        (B)    (elements)                               (us)  (GB/s)  (GB/s)            (us)  (GB/s)  (GB/s)
# <smaller message sizes truncated>
 8589934592   134217728     float    none      -1    44498  193.04  180.98      0    44500  193.03  180.97    N/A
# Out of bounds values : 0 OK
# Avg bus bandwidth    : 102.495
#
# Collective test concluded: alltoall_perf
```

## Model Inference Performance

Model-training benchmark results collected for this shape:

| Model | Scale (GPUs) | Dtype | MODEL_TFLOPS/GPU (Mean) |
|---|---|---|---|
| Qwen3 30B (MoE) | 64 | BF16 | 648.77 |
| Qwen3 30B (MoE) | 32 | BF16 | 649.06 |
| Qwen3 30B (MoE) | 16 | BF16 | 649.74 |
| Qwen3 30B (MoE) | 8 | BF16 | 648.11 |
| Llama 3.1 70B | 64 | FP8 | 1762.80 |
| Nemotron-H 56B | 64 | FP8 | 1637.74 |
| Nemotron-H 56B | 32 | FP8 | 1646.43 |

# OKE GPU Getting Started

Information on getting up and running on OKE can be found [here](https://github.com/oracle-quickstart/oci-hpc-oke).

# Troubleshooting

This guide includes a broad health-check set covering GPU visibility, firmware, NUMA topology, RDMA connectivity, storage, and intra-node bandwidth checks.

* [GPU Visibility](#gpu-visibility)
* [VBIOS Version](#vbios-version)
* [NUMA Layout](#numa-layout)
* [RDMA Interface State](#rdma-interface-state)
* [Point-to-Point RDMA Health](#point-to-point-rdma-health)
* [Additional Validation Tools](#additional-validation-tools)

## GPU Visibility

```bash
nvidia-smi
```

Use this to verify that GPUs `0-7` are present and healthy.

## VBIOS Version

```bash
nvidia-smi -q | grep -i "vbios version"
```

Sample output:

```text
VBIOS Version : 97.10.64.00.05
VBIOS Version : 97.10.64.00.05
VBIOS Version : 97.10.64.00.05
VBIOS Version : 97.10.64.00.05
VBIOS Version : 97.10.64.00.05
VBIOS Version : 97.10.64.00.05
VBIOS Version : 97.10.64.00.05
VBIOS Version : 97.10.64.00.05
```

## NUMA Layout

```bash
numactl --hardware
```

The expected topology is a two-node NUMA layout with roughly 2 TB of memory per NUMA node on a healthy system.

## RDMA Interface State

```bash
rdma link
```

The front-end network is `eth0` on `mlx5_6`. The RoCE RDMA interfaces should report `state ACTIVE` and `physical_state LINK_UP`.

## Point-to-Point RDMA Health

`ib_write_bw` and `ib_write_lat` are standard inter-node health checks for this platform.

Healthy target ranges for this platform:

- `ib_write_bw`: about `384+ Gbps`
- `ib_write_lat`: about `~3 usec` for `2-8` byte messages on nodes connected to the same switch

## Additional Validation Tools

Additional recommended validation tools:

- `gpu-fryer` for sustained thermal and FLOPS checks
- `nvbandwidth` for intra-node NVLink validation
- `BabelStream` for memory-bandwidth checks
- `dcgmi diag -r [1,2,3,4]` for layered GPU diagnostics

Example commands:

```bash
docker run --gpus all ghcr.io/huggingface/gpu-fryer:latest 60
```

```bash
git clone https://github.com/NVIDIA/nvbandwidth
cd nvbandwidth
sudo cmake -DCMAKE_CUDA_COMPILER=/usr/local/cuda/bin/nvcc -DCMAKE_CUDA_ARCHITECTURES=native
sudo ./debian_install.sh
sudo chown -R $USER:$USER CMakeFiles
make
./nvbandwidth -t device_to_device_bidirectional_memcpy_read_ce
```

```bash
git clone https://github.com/UoB-HPC/BabelStream.git
cd BabelStream
git checkout v5.0
mkdir build
cmake -B build -DMODEL=cuda -DCMAKE_CUDA_COMPILER=/usr/local/cuda-13.1/bin/nvcc -DCUDA_ARCH=sm_100
cd build
make
./cuda-stream
```

```bash
dcgmi diag -r [1,2,3,4]
```

# Further Reading & Support

Additional references:

- [NVIDIA DGX B300](https://www.nvidia.com/en-us/data-center/dgx-b300/)
- [NVIDIA DGX B300 User Guide](https://docs.nvidia.com/dgx/dgxb300-user-guide/introduction-to-dgxb300.html)
- [NVIDIA DGX SuperPOD with DGX B300 Systems Reference Architecture](https://docs.nvidia.com/dgx-superpod/reference-architecture/scalable-infrastructure-b300/latest/index.html)
- [NVIDIA Collective Communication Library (NCCL) Documentation](https://docs.nvidia.com/deeplearning/nccl/user-guide/docs/index.html)
- [NVIDIA NVBandwidth](https://github.com/NVIDIA/nvbandwidth)

The GPU thermal profile is not perfectly uniform across all eight devices because of board layout and airflow. If some GPUs consistently run warmer than others while staying within normal limits, that can still be expected behavior on this platform.
