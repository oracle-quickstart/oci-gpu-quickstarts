# OCI GPU Quick Start: NVIDIA B200
This document provides hardware specifications, supported OS images, onboarding verification, sample benchmarks, and best-practices for OCI deployments using the NVIDIA B200 GPU shape.

## At a Glance

- Shape: `BM.GPU.B200.8`
- GPU configuration: `8 x NVIDIA B200`
- Recommended OS baseline: `Oracle Linux 9+` or `Ubuntu Linux 22.04+`
- Recommended software baseline: `DOCA 3.0.0+, NVIDIA Driver 570+ (Open), CUDA 12.9+, NCCL 2.25.1+`
- Primary verification command: `nvidia-smi`
- Operational profile: `single-node and multi-node AI training workloads`

# Table of Contents
* [Hardware Specifications](#hardware-specifications)
* [Recommended Operating Systems](#recommended-operating-systems)
    * [Recommended Software Version](#recommended-software-version)
    * [Custom OS Image Creation with Packer](#custom-os-image-creation-with-packer)
    * [Provided Images](#provided-images)
    * [Hello World Verification](#hello-world-verification)
* [Performance Benchmarks](#performance-benchmarks)
    * [NCCL](#nccl)
    * [Model Inference Performance](#model-inference-performance)
* [OKE GPU Getting Started](#oke-gpu-getting-started)
* [Troubleshooting](#troubleshooting)
* [Further Reading & Support](#further-reading--support)

# Hardware Specifications

| Shape Name        | GPU Model     | GPUs/Node | GPU Memory | System Memory | vCPU | Local Storage | Key Features         |
|-------------------|---------------|-----------|------------|--------------|------|---------------|----------------------|
| BM.GPU.B200.8     | NVIDIA B200   | 8         | 192 GB     | 2048 GB      | 128  | NVMe SSD      | Domain-specific accelerators, latest CUDA |

*See the [OCI Compute Shapes Docs](https://docs.oracle.com/en-us/iaas/Content/Compute/References/computeshapes.htm) for up-to-date details.

# Recommended Operating Systems
• Oracle Linux 9+\
• Ubuntu Linux 22.04+

## Recommended Software Version
• DOCA 3.0.0+\
• NVIDIA Driver 570+ (Open)\
• CUDA 12.9+\
• NCCL 2.25.1+\
• HPCX 2.21+\
• Oracle Cloud Agent 1.51.0+

## Custom OS image Creation with Packer

To build your images using packer clone the OCI HPC Images repo and run the commands found there [OCI HPC Images GitHub Repo](https://github.com/oracle-quickstart/oci-hpc-images/blob/main/README.md).

## Provided Images

It is recommended to use Oracle provided or organizationally-approved images for security and supportability. See OS Images Table for region-specific OCIDs and version info.

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

# Hello World Verification

Run `nvidia-smi` to verify that all eight GPUs are visible and healthy:

```bash
nvidia-smi
```

## Hello World CUDA Container

```bash
docker run --rm --gpus all nvcr.io/nvidia/cuda:13.1.0-base-ubuntu24.04 nvidia-smi

```

# Performance Benchmarks

NVIDIA publishes their [NCCL](https://developer.nvidia.com/nccl) (Nvidia
Collective Communication Library) software as a toolkit for
pre-defined 
routines which are optimized for their hardware. This software is meant
to accelerate Artificial Intelligence & 
Machine Learning workloads running on NVIDIA GPU clusters. Detailed
documentation can be found
[here](https://docs.nvidia.com/deeplearning/nccl/user-guide/docs/index.html).
The NCCL operations which are important for this document are AllReduce
and AlltoAll operations. 

If the NCCL tests are not on your system, then you can build them using
the commands below

    git clone <https://github.com/NVIDIA/nccl-tests.git>
    cd nccl-tests
    make

All tests run as follows with appropriate `--np` and `--hostfile` values
provided:

```bash
mpirun --bind-to numa
        --mca pml ucx
        --mca coll ^hcoll
        -x coll_hcoll_enable=0         
        -x NCCL_DEBUG=WARN
        -x NCCL_MNNVL_ENABLE=1
        -x NCCL_CUMEM_ENABLE=1
        -x UCX_NET_DEVICES=eth0
        -x NCCL_IB_HCA==mlx5_0,mlx5_1,mlx5_2,mlx5_3,mlx5_5,mlx5_6,mlx5_7,mlx5_8
        -x NCCL_NET_PLUGIN=/opt/hpcx-v2.24.1-gcc-doca_ofed-ubuntu24.04-cuda13-aarch64/nccl_rdma_sharp_plugin/lib/[libnccl-net.so](http://libnccl-net.so).0
        -x NCCL_NVLS_ENABLE=1
        -x NCCL_SOCKET_IFNAME=eth0
        -x NCCL_IB_GID_INDEX=3
        -x NCCL_IB_TC=41
        -x NCCL_IB_SL=0
        -x NCCL_IB_TIMEOUT=22
        -x RX_QUEUE_LEN=8192
        -x IB_RX_QUEUE_LEN=8192
        -x HCOLL_ENABLE_MCAST_ALL=0
        -x NCCL_BUFFSIZE=16777216
        -x NCCL_IB_QPS_PER_CONNECTION=4
        -x NCCL_IB_SPLIT_DATA_ON_QPS=0
        -x NCCL_NET_GDR_C2C=1
        -x NCCL_MNNVLS_ENABLE=1
        -x NCCL_DMABUF_ENABLE=1
        --np xxx --hostfile ./yyy all_reduce_perf -b 512K -e 16G -f 2 -g 1 -n 50
```

## NCCL

For guidance on installing and running additional NCCL benchmarks on this family, see the [NCCL user guide](https://docs.nvidia.com/deeplearning/nccl/user-guide/docs/index.html).

The source material for B200 includes both all-reduce and all-to-all coverage for single-node and multi-node deployments. When `nccl-tests` is installed, the baseline single-node all-reduce command is:

```bash
/opt/oci-hpc/nccl-tests/build/all_reduce_perf -b 512k -e 8G -f 2 -g 8
```

## Model Inference Performance

15 billion parameter variant (FP8/BF16)

- At least 16 GPUs with at least 180GB memory each.

340 billion parameter variant (FP8/BF16)

- At least 128 GPUs with at least 180GB memory each.
- The B200 recipes listed below progressively increase GPU count, with configurations weak-scaled to match.

| Size | Precision | GPUs | SeqLen | Layers | TP  | PP  | CP  | EP  | DP  | VP  | MBS | GBS  | GA  |
|------|:---------:|:----:|:------:|:------:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:----:|:---:|
| 15b   | BF16/FP8 | 16   | 4096  | 32     | 1   | 1   | 1   | NA  | 16   | NA  | 2   | 64   | 2 |
| 15b   | BF16/FP8 | 32   | 4096  | 32     | 1   | 1   | 1   | NA  | 32   | NA  | 2   | 128   | 2 |
| 15b   | BF16/FP8 | 64   | 4096  | 32     | 1   | 1   | 1   | NA  | 64   | NA  | 2   | 256   | 2 |
| 15b   | BF16/FP8 | 128   | 4096  | 32     | 1   | 1   | 1   | NA  | 128   | NA  | 2   | 512   | 2 |
| 15b   | BF16/FP8 | 256   | 4096  | 32     | 1   | 1   | 1   | NA  | 256   | NA  | 2   | 1024   | 2 |
| 15b   | BF16/FP8 | 512   | 4096  | 32     | 1   | 1   | 1   | NA  | 512   | NA  | 2   | 2048   | 2 |
| 15b   | BF16/FP8 | 1024   | 4096  | 32     | 1   | 1   | 1   | NA  | 1024   | NA  | 2   | 4096   | 2 |

| Size | Precision | GPUs | SeqLen | Layers | TP  | PP  | CP  | VP  | MBS | GBS | DP   | GA  |
|------|:--------:|:------:|:------:|:------:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| 340b  | BF16/FP8 | 128  | 4096   | 96     | 8   | 4   | 1   | 12  | 1   | 32   |4    | 8   |
| 340b  | BF16/FP8 | 256  | 4096   | 96     | 8   | 4   | 1   | 12  | 1   | 64   |8    | 8   |
| 340b  | BF16/FP8 | 512  | 4096   | 96     | 8   | 4   | 1   | 12  | 1   | 128  |16   | 8   |
| 340b  | BF16/FP8 | 1024 | 4096   | 96     | 8   | 4   | 1   | 12  | 1   | 256  |32   | 8   |

- Strong Scaling, where GBS remains constant across increasing GPU counts and reducing gradient accumulation.

| Size | Precision | GPUs | SeqLen | Layers | TP  | PP  | CP  | VP  | MBS | GBS | DP  | GA   |
|------|:--------:|:------:|:------:|:------:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| 340b  | BF16/FP8 | 128  | 4096   | 96     | 8   | 4   | 1   | 12  | 1   | 512 | 4   | 128  |
| 340b  | BF16/FP8 | 256  | 4096   | 96     | 8   | 4   | 1   | 12  | 1   | 512 | 8   | 64   |
| 340b  | BF16/FP8 | 512  | 4096   | 96     | 8   | 4   | 1   | 12  | 1   | 512 | 16  | 32   |
| 340b  | BF16/FP8 | 1024 | 4096   | 96     | 8   | 4   | 1   | 12  | 1   | 512 | 32  | 16   |

### Training Numbers

Example output:
```shell
Train Step Timing and TFLOPS Analysis (iterations 35-44)
================================================================================
Experiment                                                                                  Status Time Mean (s) Time Std (s) TFLOPS_per_GPU Mean TFLOPS_per_GPU Std
------------------------------------------------------------------------------------------ -------- ------------- ------------ ------------------- ------------------
pretrain_nemotron4_340b_fp8_gpus128_tp8_pp4_cp1_vp12_mbs1_gbs512_389757                     Success        21.542        0.010             1600.74               0.78
```

To obtain throughput as a tokens per second measurement, follow this formula: 
```shell
(sequence length) * (global batch size) / (training_step_timing) = (throughput in tokens per second)
```

E.g. 4096 * 256 / 2.693 = 389371

To calculate time to train estimate:
```shell
(total tokens) / (throughput in tokens per second) / (number of seconds in a day) = (time to train in days) 
```
E.g. 1e12 / 389371 / 86400 = 29.7 days 


To calculate the model flops utilization (MFU):
```shell
MFU = (global batch size) * (model flops) / (training step time) / (number of GPUs) / (peak GPU FLOPS)
```
The model flops for Nemotron4 15b for GBS=1 is 3.85e14. Calculation shown [here](#mfu-formula).

E.g. NeMotron4 15b BF16 on 64x H100 GPUs (GBS=256)
```shell
peak FLOPS for H100 BF16 = 989 TFLOPS
training step time = 2.693 s
model flops = 3.85e14

MFU = 256 * 3.85e14 / 2.693 / 64 / 989e+12 = 57.82%
```

**Peak theoretical throughput across GPUs and Data Types (in TFLOPS)**

| Data Type | B200  | GB200 | H100 |
| --------  | :---: | :---: | :---:|
| BF16      | 2250  | 2450  | 989  |
| FP8       | 4500  | 4900  | 1979 |  


# OKE GPU Getting Started

1. Create OKE Cluster via OCI Console or CLI.
2. Add GPU Node Pool: Choose B200 shape, use golden OS image.
3. Install NVIDIA Device Plugin:
```bash
kubectl apply -f https://github.com/NVIDIA/k8s-device-plugin/raw/main/nvidia-device-plugin.yml
```
4. Run GPU-powered Kubernetes workloads:
Example pod resource spec:
```yaml
resources:
  limits:
    nvidia.com/gpu: 1
```
5. Verify with Hello World CUDA container pod.
See detailed instructions in OKE Onboarding Guide .

Useful B200-specific OKE starting points in `oci-hpc-oke`:

- [B200 NCCL test manifest](https://github.com/oracle-quickstart/oci-hpc-oke/blob/main/manifests/nccl-tests/kueue/BM.GPU.B200.8.yaml)
- [Running active health checks on OKE](https://github.com/oracle-quickstart/oci-hpc-oke/blob/main/docs/running-active-health-checks.md)
- [Running ib_write_bw on OKE](https://github.com/oracle-quickstart/oci-hpc-oke/blob/main/docs/running-ib-write-bw-test.md)

# Troubleshooting
Here you can find suggested troubleshooting methods.

## Health Checks

**DR HPC Based Scrips & or OCI GPU Scanner Content**

## Further Reading & Support
1. Blog on B200 hardware & its multi node performance AI for Employees
