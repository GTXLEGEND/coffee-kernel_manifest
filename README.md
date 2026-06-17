# Realme 12 Pro Plus 5G — Kernel Build Manifest

## Device Information

| Field | Details |
|---|---|
| Device | Realme 12 Pro 5G |
| Codename | `coffee` |
| Platform | `parrot` |
| SoC | Snapdragon 6 Gen 1 (SM6450-AB) |
| Android | Android 16 (Baklava) |
| Kernel variant | GKI (Generic Kernel Image) |

---

## Manifest Structure

```text
.  (realme12p-kernel)
│
├── kernel_platform/
│   ├── build/                            
│   ├── common/                           ← ACK common kernel source
│   │   └── build.config.msm.parrot       ← symlinked from msm-kernel
│   ├── external/
│   ├── kernel/
│   ├── msm-kernel/                       ← Qualcomm MSM kernel source
│   │   ├── build.config.msm.parrot       ← actual file lives here
│   │   ├── build.config.msm.common
│   │   └── build.config.msm.gki
│   ├── oplus/                            ← vendor/devicetree source
│   │   └── config/
│   │       ├── modules.list.oplus
│   │       └── build.config.msm.wapio.oplus  ← optional oplus overlay
│   ├── prebuilts/
│   ├── prebuilts-master/
│   ├── qcom/
│   └── tools/
└── vendor/
```

---

## Repositories

| Repository | Path | Branch |
|---|---|---|
| [realme_12pro_5G-AndroidB-common-source](https://github.com/realme-kernel-opensource/realme_12pro_5G-p1pro_5G-AndroidB-common-source) | `kernel_platform/common` | `master` |
| [realme_12pro_5G-AndroidB-kernel-source](https://github.com/realme-kernel-opensource/realme_12pro_5G-p1pro_5G-AndroidB-kernel-source) | `kernel_platform/msm-kernel` | `master` |
| [realme_12pro_5G-AndroidB-vendor-source](https://github.com/realme-kernel-opensource/realme_12pro_5G-p1pro_5G-AndroidB-vendor-source) | `./` | `master` |

---

## Prerequisites

```bash
# Install repo tool
mkdir -p ~/.bin
curl https://storage.googleapis.com/git-repo-downloads/repo > ~/.bin/repo
chmod a+x ~/.bin/repo
export PATH="${HOME}/.bin:${PATH}"

# Install build dependencies
sudo apt-get install -y \
    bc bison build-essential ccache curl flex \
    g++-multilib gcc-multilib git gnupg gperf \
    imagemagick lib32ncurses5-dev lib32readline-dev \
    lib32z1-dev liblz4-tool libncurses5-dev libsdl1.2-dev \
    libssl-dev libxml2 libxml2-utils lzop pngcrush \
    rsync schedtool squashfs-tools xsltproc zip zlib1g-dev \
    python3 python-is-python3 libelf-dev
```

---

## Setup & Sync

### 1. Initialize Manifest

```bash
# Create working directory
mkdir ~/realme12p-kernel && cd ~/realme12p-kernel

# Initialize repo with the remote manifest
repo init -u https://github.com/GTXLEGEND/coffee-kernel_manifest.git -b master -m realme_12pro_b.xml
```

### 2. Sync Sources

```bash
# Full sync
repo sync -j8 --no-tags --no-clone-bundle

# Or sync step by step (recommended for first time)
repo sync kernel_platform/common -j4 --no-tags
repo sync kernel_platform/msm-kernel -j4 --no-tags
repo sync ./ -j4 --no-tags
```

---

## Build

### Build Command

```bash
./kernel_platform/oplus/build/oplus_build_kernel.sh parrot gki
```

### What This Does

| Argument | Value | Meaning |
|---|---|---|
| Platform | `parrot` | Targets SM7435-AB / Snapdragon 6 Gen 1 SoC |
| Variant | `gki` | Builds Generic Kernel Image (GKI 2.0) |

### Build Variants

```bash
# GKI build (recommended)
./kernel_platform/oplus/build/oplus_build_kernel.sh parrot gki

# Consolidate build (includes all drivers/modules)
./kernel_platform/oplus/build/oplus_build_kernel.sh parrot consolidate
```

### Optional Build Flags

```bash
# Skip LTO for faster debug builds
LTO=none ./kernel_platform/oplus/build/oplus_build_kernel.sh parrot gki

# Enable aging/debug mode (disables KMI strict mode)
AGING_DEBUG_MASK=1 ./kernel_platform/oplus/build/oplus_build_kernel.sh parrot gki

# Specify number of build jobs
PARALLELISM=16 ./kernel_platform/oplus/build/oplus_build_kernel.sh parrot gki
```

---

## Notes
- GKI variant is recommended for shipping builds. Use `consolidate` for development/debugging.
