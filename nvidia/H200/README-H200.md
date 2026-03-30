> [!WARNING]
> **INTERNAL USE ONLY — NOT FOR PUBLIC DISTRIBUTION**
> This document has not been approved for public release. It must not be committed to a public repository or shared externally until this notice is explicitly removed by the document owner prior to merge.

# OCI GPU Quick Start: NVIDIA H200

This document provides hardware specifications, supported OS images, onboarding verification, sample benchmarks, and best-practices for OCI deployments using the NVIDIA H200 GPU shape.

BM.GPU.H200.8 is a high-bandwidth NVIDIA H200 bare-metal shape intended for large-scale AI and HPC workloads, with eight 141 GB GPUs, dual Intel Xeon Platinum 8480+ processors, 3 TB of DDR5 memory, and RoCE-capable scale-out networking.

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
| BM.GPU.H200.8 | H200 | 8 | 141 | 1128 GB | 2 x Intel Xeon Platinum 8480+ @ 2.0 GHz | 112 Cores | 3 TB DDR5 | 8 x 3.5 TB NVMe = 28 TB | 100 Gb/s | 8 x 1 x 400 Gb/s = 3.2 Tb/s |

See the [OCI Compute Shapes Docs](https://docs.oracle.com/en-us/iaas/Content/Compute/References/computeshapes.htm) for up-to-date details.

# Recommended Operating Systems

- Oracle Linux 8+
- Ubuntu Linux 22.04+

## Recommended Software Version

- OFED 23.10+
- NVIDIA Driver 550+
- CUDA 12.4+
- NCCL 2.21.5+
- HPC-X 2.18+
- Oracle Cloud Agent 1.49.3+

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

You should see all eight H200 GPUs listed with a healthy driver stack and expected memory footprint.

# Performance Benchmarks

NVIDIA publishes [NCCL](https://developer.nvidia.com/nccl) as the primary collective communication library for multi-GPU AI and HPC workloads. The source material for H200 focuses on representative single-node and multi-node NCCL workflows plus the network guidance required for scale-out runs.

* [All Reduce - Single Node](#all-reduce---single-node)
* [Multi-node Guidance](#multi-node-guidance)
* [Model Inference Performance](#model-inference-performance)

## All Reduce - Single Node

```bash
./build/all_reduce_perf -b 8 -e 8G -f 2 -g 8
```

## Multi-node Guidance

The source material highlights a single-subnet network layout for best multi-node NCCL behavior on this family.

For guidance on running additional NCCL collective benchmarks on this GPU family, see the [NCCL user guide](https://docs.nvidia.com/deeplearning/nccl/user-guide/docs/index.html).

## Model Inference Performance

The H200 source page in this workflow is oriented more toward validation, NCCL guidance, and training examples than a clean standalone inference table, so no normalized model-performance table is included here.

# OKE GPU Getting Started

Information on getting up and running on OKE can be found [here](https://github.com/oracle-quickstart/oci-hpc-oke).

# Troubleshooting

This guide includes a broad health-check set covering GPU visibility, NUMA topology, RDMA connectivity, DCGM diagnostics, PCIe bandwidth, and NVLink validation.

* [GPU Visibility](#gpu-visibility)
* [NUMA Layout](#numa-layout)
* [RDMA Interface State](#rdma-interface-state)
* [DCGM Diagnostics](#dcgm-diagnostics)
* [PCIe and NVLink Validation](#pcie-and-nvlink-validation)

## GPU Visibility

```bash
nvidia-smi
```

## NUMA Layout

```bash
numactl --hardware
```

For H200, the source material expects a layout comparable to a 112-core dual-socket system, or fewer visible cores when hyperthreading is disabled.

## RDMA Interface State

```bash
rdma link
```

The source material identifies the front-end network as `eth0` on `mlx5_1`, with RoCE RDMA interfaces expected to report `ACTIVE` and `LINK_UP`.

## DCGM Diagnostics

```bash
dcgmi diag -r 1
dcgmi diag -r 2
dcgmi diag -r 3
```

The source material describes:

- `r1` as a basic metadata and deployment check
- `r2` as a deeper integration and hardware check
- `r3` as a fuller stress-oriented validation pass

## PCIe and NVLink Validation

Additional validation tools referenced in the source material:

- `bandwidthTest` from NVIDIA cuda-samples for PCIe bandwidth
- `nvbandwidth` for NVLink bandwidth validation

# Further Reading & Support

Additional references:

- [NCCL User Guide](https://docs.nvidia.com/deeplearning/nccl/user-guide/docs/index.html)
- [NVIDIA nvbandwidth](https://github.com/NVIDIA/nvbandwidth)

