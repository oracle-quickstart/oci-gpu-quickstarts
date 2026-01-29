# OCI GPU Quick Start: NVIDIA B200
This document provides hardware specifications, supported OS images, onboarding verification, sample benchmarks, and best-practices for OCI deployments using the NVIDIA B200 GPU shape.

## Table of Contents
1. [Hardware Specifications](#hardware-specifications)
2. [Supported Golden OS Images](#supported-golden-os-images)
3. [Hello World Verification](#hello-world-verification)
4. [Performance Benchmarks](#health-check--baseline-benchmarks)
5. [NCCL & Model Inference Performance](#nccl--model-inference-performance)
6. [Custom OS Image Creation with Packer](#custom-os-image-creation-with-packer)
7. [OKE GPU Getting Started](#oke-gpu-getting-started)
8. [Further Reading & Support](#further-reading--support)

## Onboarding

All details to help you get started with NVIDIA B200 OCI Bare-Metal GPUs.

### Hardware Specifications


| Shape Name        | GPU Model     | GPUs/Node | GPU Memory | System Memory | vCPU | Local Storage | Key Features         |
|-------------------|---------------|-----------|------------|--------------|------|---------------|----------------------|
| BM.GPU.B200.8     | NVIDIA B200   | 8         | 192 GB     | 2048 GB      | 128  | NVMe SSD      | Domain-specific accelerators, latest CUDA |

See the [OCI Compute Shapes Docs](https://docs.oracle.com/en-us/iaas/Content/Compute/References/computeshapes.htm) for up-to-date details.


### Supported Golden OS Images
Always use Oracle-provided or organizationally-approved images for security and supportability. See OS Images Table for region-specific OCIDs and version info.

Use Oracle-provided or approved images for security and supportability.

| OS Version        | Image OCID (phx example)       | Download/Marketplace Link                                                                        |
|-------------------|-------------------------------|--------------------------------------------------------------------------------------------------|
| Oracle Linux 9.3  | `ocid1.image.oc1.phx.<xxxxxx>`| [Oracle Cloud Marketplace OL9.3 GPU (Sample)](https://cloud.oracle.com/marketplace/en_US/listing/xxxxxx) |
| Oracle Linux 8.9  | `ocid1.image.oc1.phx.<yyyyyy>`| [Oracle Cloud Marketplace OL8.9 GPU (Sample)](https://cloud.oracle.com/marketplace/en_US/listing/yyyyyy) |

See [OS Images Table](os-images.md) for region-specific OCIDs and version info.

### Hello World Verification

**1. Confirm Hardware & Drivers**
```sh
nvidia-smi

+-----------------------------------------------------------------------------+
| NVIDIA-SMI ...   Driver Version: 550.xx  CUDA Version: 12.4                 |
| GPU  Name           Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
|  0  NVIDIA B200            On   | 00000000:01:00.0 Off |                    0 |
+----------------------------------------
```
### Hello World CUDA Container
```bash
docker run --rm --gpus all nvcr.io/nvidia/cuda:12.4.1-base-ubuntu22.04 nvidia-smi

```
### OKE GPU Getting Started
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

## Performance & Debugging 

The section blow shares the base line numbers achieved and how to reproduce this with OCI BM GPUs.

### Baseline Benchmarks

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

## Health Checks

DR HPC Based Scrips & or OCI GPU Scanner

## Further Reading & Support
1. Blog on B200 hardware & its multi node performance AI for Employees