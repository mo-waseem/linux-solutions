# ASUS ROG Zephyrus G14 (GA401QH) Linux Stability & Thermal Fix

This document explains how we permanently fixed random reboots, thermal crashes, and ACPI firmware faults on the **ASUS ROG Zephyrus G14 GA401QH** running Ubuntu Linux.

---

## Install Rust (Required)

```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
source ~/.cargo/env
```

---

## 1. Build and Install `asusctl`

```bash
sudo apt install -y make cargo gcc pkg-config openssl libasound2-dev cmake build-essential python3 libfreetype6-dev libexpat1-dev libxcb-composite0-dev libssl-dev libx11-dev libfontconfig1-dev curl libclang-dev libudev-dev checkinstall libseat-dev libinput-dev libxkbcommon-dev libgbm-dev

git clone https://gitlab.com/asus-linux/asusctl.git
cd asusctl
make
sudo make install
sudo systemctl enable asusd --now
```

---

## 2. Build and Install `supergfxctl`

```bash
sudo apt update && sudo apt install -y curl git build-essential
git clone https://gitlab.com/asus-linux/supergfxctl.git
cd supergfxctl
make && sudo make install
sudo systemctl enable supergfxd.service --now
```

---

## 3. Set ASUS Profile

```bash
asusctl profile -P Quiet
```

---

## 4. Enable Fan Control

```bash
asusctl fan-curve -m balanced -e true
asusctl fan-curve -m balanced -p cpu 20 30 45 55 70 85 105 140
asusctl fan-curve -m balanced -p gpu 30 40 60 75 90 110 135 155
```

---

### 4.1 Install thermald

```bash
sudo apt install -y thermald
sudo systemctl enable thermald --now
```

---

## 5. Lock CPU Governor

```bash
sudo apt install -y linux-tools-common linux-tools-$(uname -r)
sudo cpupower frequency-set -g powersave
```

---

## 6. Disable Broken IOMMU

Edit `/etc/default/grub`:

```
GRUB_CMDLINE_LINUX_DEFAULT="quiet splash amd_iommu=off iommu=soft"
```

Apply:

```bash
sudo update-grub
```

---

## 7. Reboot

```bash
sudo reboot
```

---

## 8. Stress Test

```bash
sudo apt install -y stress-ng lm-sensors
sudo sensors-detect
watch -n 1 sensors
stress-ng --cpu 8 --vm 4 --vm-bytes 90% --timeout 10m
```

CPU temperature must stay under **83°C**.
