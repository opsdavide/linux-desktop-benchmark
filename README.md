# Linux Desktop Performance: GNOME vs XFCE

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![OS: Ubuntu 24.04 LTS](https://img.shields.io/badge/Ubuntu-24.04_LTS-E95420.svg?logo=ubuntu&logoColor=white)](https://releases.ubuntu.com/24.04/)
[![Kernel: Linux 6.17](https://img.shields.io/badge/Linux_Kernel-6.17.0--22-FCC624.svg?logo=linux&logoColor=black)](https://kernel.org/)
[![Hypervisor: VirtualBox 7](https://img.shields.io/badge/Hypervisor-VirtualBox_7.x-183A61.svg?logo=virtualbox&logoColor=white)](https://www.virtualbox.org/)
[![Desktop: GNOME 46](https://img.shields.io/badge/GNOME-46-4a86cf.svg?logo=gnome&logoColor=white)](https://www.gnome.org/)
[![Desktop: XFCE 4.18](https://img.shields.io/badge/XFCE-4.18-0055aa.svg?logo=xfce&logoColor=white)](https://xfce.org/)
[![Report: PDF](https://img.shields.io/badge/Report-PDF_Document-red.svg?logo=adobeacrobatreader&logoColor=white)](report/linux_desktop_benchmark_report.pdf)

Comparative benchmark analyzing resource consumption and compositor stability across GNOME 46 and XFCE 4.18 on a virtualized developer workstation.

**Test environment:** Ubuntu 24.04.4 LTS, VirtualBox 7.x, Kernel 6.17.0-22-generic, 4 vCPUs, 4 GB RAM.

---

## Key Results

| Metric | GNOME 46 | XFCE 4.18 | Delta |
|---|---|---|---|
| Idle RAM | 1,266 MB | 934 MB | **-26.2%** |
| Active RAM (IDE + Tools) | 2,464 MB | 2,200 MB | **-10.7% (-264 MB)** |
| Available headroom (active) | 1,449 MB | 1,713 MB | **+18.2%** |
| Active CPU %system | 13.21% | 5.57% | **-7.64 pp (>57% reduction)** |
| Active CPU %idle | 57.13% | 87.80% | **+30.67 pp** |
| Total boot time | 40.250 s | 31.216 s | **-22.4%** |

---

## Technical Report

📄 [linux_desktop_benchmark_report.pdf](report/linux_desktop_benchmark_report.pdf)

The complete report details:
- Architectural differences between GNOME Mutter (Wayland/X11 compositor) and XFCE Xfwm4.
- Mitigation of boot deadlocks caused by `drm_fb_helper_damage_work` soft lockups under VirtualBox on kernel 6.17.0-22-generic.
- Telemetry across N=3 cold-boot iterations with a variance of ±15 MB.
- Workstation tradeoffs across memory footprint, kernel context switching, and display compositing.

---

## Raw Telemetry

### GNOME 46: Idle Baseline
`free -m`: 1,266 MB used | `sar`: %system 4.24%, %idle 87.98% | boot: 40.250 s

![GNOME idle](data/screenshots/gnome_idle_free_sar.png)

---

### GNOME 46: Active Workload (VSCode, LSP, Linter)
`free -m`: 2,464 MB used | `sar`: %system 13.21%, %idle 57.13%

![GNOME active](data/screenshots/gnome_active_free_sar.png)

---

### XFCE 4.18: Idle Baseline
`free -m`: 934 MB used | `sar`: %system 0.62%, %idle 99.12% | boot: 31.216 s

![XFCE idle](data/screenshots/xfce_idle_free_sar.png)

---

### XFCE 4.18: Active Workload (VSCode, LSP, Linter)
`free -m`: 2,200 MB used | `sar`: %system 5.57%, %idle 87.80%

![XFCE active](data/screenshots/xfce_active_free_sar.png)

---

## Reproduction

**Environment:** Ubuntu 24.04.4 LTS, VirtualBox 7.x, 4 vCPUs, 4 GB RAM, VBoxSVGA display adapter (3D acceleration disabled).

```bash
# 1. Install XFCE and monitoring utilities
sudo apt update && sudo apt install -y xfce4 xfce4-goodies lightdm lightdm-gtk-greeter sysstat

# 2. Mask Plymouth services to avoid the DRM framebuffer lockup
sudo systemctl mask plymouth-start.service plymouth-quit-wait.service

# 3. Strip splash from GRUB
# Set GRUB_CMDLINE_LINUX_DEFAULT="quiet" in /etc/default/grub
sudo update-grub

# 4. Collect baseline metrics
free -m
sar -u 1 5
systemd-analyze
```

**Display configuration:**
- Disable Hardware 3D Acceleration in VirtualBox VM settings.
- Disable compositing in XFCE: `Settings > Window Manager Tweaks > Compositor` and uncheck `Enable display compositing`.

---

## License

This project is licensed under the MIT License. See [LICENSE](LICENSE) for details.

---

## Author

**Davide Torelli** | Cybersecurity Master Student, AI Governance & Cloud Compliance  
*Version 1.0 (October 2026)*