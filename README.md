<div align="center">

![Oracle Cloud Infrastructure](https://img.shields.io/badge/Oracle_Cloud_Infrastructure-F80000?style=for-the-badge&logo=oracle&logoColor=white)&nbsp;&nbsp;
![NVIDIA](https://img.shields.io/badge/NVIDIA-76B900?style=for-the-badge&logo=nvidia&logoColor=white)&nbsp;&nbsp;
![AMD](https://img.shields.io/badge/AMD-ED1C24?style=for-the-badge&logo=amd&logoColor=white)

# OCI GPU Quickstarts
</div>

## Oracle Cloud Infrastructure GPU Onboarding Guides

This repository hosts onboarding content for Oracle Cloud Infrastructure GPU offerings, organized by GPU family. Each guide provides:

1. A snapshot of each GPU hardware configuration
2. A quick-start guide to provision GPU resources on OCI
3. A "Hello World" sample to verify compute and driver/toolchain readiness
4. Links to approved OS images and drivers
5. Guidance for running on Oracle Kubernetes Engine (OKE)
6. Basic and advanced health and issue diagnosis tools
7. Links to blogs and partner content covering advanced configurations and specifications

---

## Table of Contents

| Vendor | Shape | Guide | Status |
|--------|-------|-------|--------|
| ![NVIDIA](https://img.shields.io/badge/NVIDIA-76B900?style=flat-square&logo=nvidia&logoColor=white) | GB300 | [README-GB300.md](nvidia/GB300/README-GB300.md) | ✅ Complete |
| ![NVIDIA](https://img.shields.io/badge/NVIDIA-76B900?style=flat-square&logo=nvidia&logoColor=white) | GB200 | [README-GB200.md](nvidia/GB200/README-GB200.md) | ✅ Complete |
| ![NVIDIA](https://img.shields.io/badge/NVIDIA-76B900?style=flat-square&logo=nvidia&logoColor=white) | H100 | [README-H100.md](nvidia/H100/README-H100.md) | 🚧 In Progress |
| ![NVIDIA](https://img.shields.io/badge/NVIDIA-76B900?style=flat-square&logo=nvidia&logoColor=white) | H200 | [README-H200.md](nvidia/H200/README-H200.md) | 🚧 In Progress |
| ![NVIDIA](https://img.shields.io/badge/NVIDIA-76B900?style=flat-square&logo=nvidia&logoColor=white) | B200 | [README-B200.md](nvidia/B200/README-B200.md) | 🚧 In Progress |
| ![NVIDIA](https://img.shields.io/badge/NVIDIA-76B900?style=flat-square&logo=nvidia&logoColor=white) | B300 | [README-B300.md](nvidia/B300/README-B300.md) | 🚧 In Progress |
| ![AMD](https://img.shields.io/badge/AMD-ED1C24?style=flat-square&logo=amd&logoColor=white) | MI355X | [README-MI355X.md](amd/MI355X/README-MI355X.md) | ✅ Complete |
| ![AMD](https://img.shields.io/badge/AMD-ED1C24?style=flat-square&logo=amd&logoColor=white) | MI355X (Pollara) | [README-MI355X-Pollara.md](amd/MI355X/README-MI355X-Pollara.md) | ✅ Complete |
| ![AMD](https://img.shields.io/badge/AMD-ED1C24?style=flat-square&logo=amd&logoColor=white) | MI300X | [README-MI300X.md](amd/MI300X/README-MI300X.md) | 🚧 In Progress |

---

## Repository Structure

```
/nvidia
    /B200
        README-B200.md
    /H100
        README-H100.md
    /H200
        README-H200.md
    /GB200
        README-GB200.md
    /GB300
        README-GB300.md
/amd
    /MI300X
        README-MI300X.md
    /MI355X
        README-MI355X.md
        README-MI355X-Pollara.md
        /k8s
            [Example Kubernetes YAMLs]
```

---

### Questions

Reach out to [amar.gowda@oracle.com](mailto:amar.gowda@oracle.com)

## Contributing

This project welcomes contributions from the community. Before submitting a pull request, please [review our contribution guide](./CONTRIBUTING.md).

## Security

Please consult the [security guide](./SECURITY.md) for our responsible security vulnerability disclosure process.

## License

Copyright (c) 2024 Oracle and/or its affiliates.

Released under the Universal Permissive License v1.0 as shown at
<https://oss.oracle.com/licenses/upl/>.
