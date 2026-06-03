# OmniROM 12.1 GitHub Actions Build Guide for H616/H618

## Quick Start with GitHub Actions

### Step 1: Enable GitHub Actions
1. Go to your repository: `https://github.com/wolfhamad/android_apollo-p2_local_manifest`
2. Click **Settings** → **Actions** → **General**
3. Enable: "Allow all actions and reusable workflows"

### Step 2: Trigger Your First Build

1. Go to **Actions** tab
2. Select **"Build OmniROM 12.1"** workflow (left sidebar)
3. Click **"Run workflow"** (blue button, top right)
4. Fill in options:
   - **Device**: Select `apollo_p3` (for Transpeed 8K618-T H618) or `apollo_p2` (H616)
   - **Build type**: Select `eng` (engineering, for testing)
5. Click **"Run workflow"** button

### Step 3: Monitor Build

- Workflow will appear in the Actions tab
- Click on the running workflow to see real-time logs
- Build takes **2-4 hours** depending on network

### Step 4: Download ROM

Once complete (green checkmark):
1. Click on the successful workflow run
2. Scroll down to **Artifacts** section
3. Download: `omnirom-12.1-apollo_p3-eng` (or your device)
4. Extract the `.zip` file - this is your flashable ROM!

---

## GitHub Actions - Full Details

### When It Builds

The workflow triggers in these scenarios:

```yaml
# Manual trigger (recommended)
- Click "Run workflow" in Actions tab

# Automatic on schedule (optional)
- Every day at 2 AM UTC (can be customized)

# On repository push (optional, can enable)
- When you push to main branch
```

### What The Workflow Does

1. **Setup** (5 min)
   - Installs dependencies: `bc`, `bison`, `build-essential`, `git`, `repo`, Java, etc.
   - Creates build directories

2. **Sync** (30-60 min)
   - Downloads OmniROM 12.1 source code (~20GB)
   - Downloads device tree + kernel + vendor blobs
   - Applies your local manifest

3. **Patch U-Boot** (2 min)
   - Your `174-define-DRAM_MAX_SIZE-to-4GB.patch` is applied
   - Your `configs-Transpeed-8K618-T-Add-Transpeed-Secure-Boot.patch` is applied

4. **Compile** (60-120 min)
   - Compiles U-Boot
   - Compiles Linux kernel
   - Compiles Android framework
   - Builds OmniROM 12.1 ROM

5. **Package** (10 min)
   - Creates flashable `.zip` file
   - Uploads artifacts to GitHub

6. **Complete** ✅
   - ROM ready for download
   - Build logs saved
   - Artifacts kept for 30 days

---

## File Explanations

### `.github/workflows/build-omnirom.yml`
**GitHub Actions workflow configuration**

Key sections:
```yaml
on:
  workflow_dispatch:           # Manual trigger via UI
    inputs:
      device:                  # Choose: apollo_p2 or apollo_p3
      build_type:              # Choose: eng, userdebug, or user

jobs:
  build:
    runs-on: ubuntu-latest     # Uses free GitHub-hosted runner
    timeout-minutes: 480       # 8 hour timeout
```

### Build Commands in Workflow
```bash
# 1. Initialize repo
repo init -u https://github.com/omnirom/android.git -b android-12.1 --depth=1

# 2. Sync all sources (including your patches via local_manifest.xml)
repo sync -j$(nproc --all) --no-clone-bundle

# 3. Apply U-Boot patches (your 2 patches here)
bash apply-uboot-patches.sh

# 4. Configure device
lunch omni_apollo_p3-eng

# 5. Build ROM
mka -j$(nproc --all) bacon
```

### `.patches/` Directory
Contains your two U-Boot patches:
- `174-define-DRAM_MAX_SIZE-to-4GB.patch` - 4GB RAM support
- `configs-Transpeed-8K618-T-Add-Transpeed-Secure-Boot.patch` - Secure boot

### `apply-uboot-patches.sh`
**Applies patches to U-Boot source**

This script:
1. Waits for `repo sync` to complete
2. Locates U-Boot directory
3. Applies both patches in order
4. Verifies success

---

## Device Selection

### For Transpeed 8K618-T (H618)
```
Device: apollo_p3
RAM: 4GB (with patch 1)
Storage: eMMC/microSD
Build type: eng (for testing) or userdebug
```

### For Orange Pi Zero 2 (H616)
```
Device: apollo_p2
RAM: 2GB (default) or 4GB (optional)
Storage: eMMC/microSD
Build type: eng or userdebug
```

---

## Build Time & Costs

### First Build
- **Duration**: 2-4 hours
- **Cost (free tier)**: FREE ✅
- **Storage**: ~50GB on runner (temporary)
- **Output size**: ~500MB ROM

### Rebuilds (incremental)
- **Duration**: 30-60 min (with ccache)
- **Cost**: FREE
- **Output**: Same ~500MB ROM

### GitHub Actions Free Tier Limits
- **Public repos**: Unlimited minutes ✅
- **Private repos**: 2,000 minutes/month
- **Artifact storage**: 1GB
- **Concurrent jobs**: 20

Since your repo is PUBLIC, you get **UNLIMITED free build minutes!** 🎉

---

## Customization Options

### Change Default Build Type

Edit `.github/workflows/build-omnirom.yml`:
```yaml
default: 'userdebug'  # Change from 'eng' to 'userdebug' or 'user'
```

### Add Scheduled Builds

Already included! Builds automatically daily at 2 AM UTC. To disable, remove:
```yaml
schedule:
  - cron: '0 2 * * *'
```

### Build on Push

To build automatically when you push to main:
```yaml
on:
  workflow_dispatch: ...
  push:
    branches:
      - main
      - omni-12.1
```

---

## Troubleshooting

### Build Failed - Check Logs

1. Go to **Actions** → Failed workflow
2. Click on **"build"** job
3. Expand each step to find error
4. Most common issues:
   - **Out of disk space**: Remove old artifacts
   - **Repo sync failed**: Try again (network issue)
   - **Patch conflict**: Check U-Boot source compatibility

### Out of Disk Space

GitHub Actions runners have ~160GB, but OmniROM takes ~50GB:

```bash
# Build logs show this:
No space left on device

# Solution: Clean old artifacts
# Go to Settings → Actions → Artifacts and Logs
# Select artifacts older than 7 days and delete
```

### Slow Network

If `repo sync` is slow:
```bash
# Already optimized in workflow, but you can:
# 1. Wait during off-peak hours
# 2. Run locally on Codespaces instead
# 3. Pre-cache sources (advanced)
```

### Patches Don't Apply

Check U-Boot version matches:
1. `apply-uboot-patches.sh` looks for: `kernel/allwinner/u-boot-2022.10`
2. If path is different, edit the script:
```bash
UBOOT_DIR="kernel/allwinner/u-boot-YOUR-VERSION"
```

---

## Download & Flash ROM

### After Successful Build

1. **Download ROM**
   ```
   Actions → Latest successful build → Artifacts
   → omnirom-12.1-apollo_p3-eng
   → OmniROM-12.1-apollo_p3-eng-*.zip
   ```

2. **Flash to Device**

   **Using TWRP Recovery**:
   ```
   1. Copy ROM .zip to device microSD/USB
   2. Boot into TWRP recovery
   3. Wipe: System + Vendor + Cache
   4. Install: Select ROM .zip
   5. Reboot
   ```

   **Using Fastboot** (H618/H616):
   ```bash
   fastboot flash boot boot.img
   fastboot flash system system.img
   fastboot flash vendor vendor.img
   fastboot reboot
   ```

---

## Next Steps

1. ✅ Confirm these files are created:
   - `.github/workflows/build-omnirom.yml`
   - `.patches/` directory with your 2 patches
   - `apply-uboot-patches.sh`

2. ✅ Push to GitHub (if not already there)

3. ✅ Go to Actions tab → Run workflow

4. ✅ Monitor build progress

5. ✅ Download ROM when complete

---

**Questions?** Check the logs in the Actions tab for detailed error messages!
