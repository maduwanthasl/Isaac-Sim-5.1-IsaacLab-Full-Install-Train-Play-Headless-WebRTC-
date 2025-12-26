# Isaac Sim & Isaac Lab Setup Guide

Complete installation and configuration guide for NVIDIA Isaac Sim and Isaac Lab on Ubuntu with GPU support.

**🎯 Designed for GPU servers without root access or Docker** - Perfect for university clusters and shared computing environments.

---

## 📋 System Requirements

- **OS**: Ubuntu 20.04 or 22.04
- **GPU**: NVIDIA GPU with CUDA support
- **CUDA**: 11.8 or 12.x
- **Python**: 3.10 or 3.11
- **RAM**: 32GB recommended
- **Storage**: 50GB+ free space
- **Access Level**: User-level (no root/sudo required)
- **Network**: Internet access for package installation

### ⚙️ Server Constraints This Guide Addresses

✅ **No Docker access** - Uses native Micromamba installation  
✅ **No root/sudo privileges** - All installations in user space  
✅ **Headless environment** - WebRTC streaming for remote visualization  
✅ **SSH-only access** - No direct GUI/X11 display  
✅ **Shared GPU cluster** - Multi-user environment compatible

---

## 🚀 Quick Start

**Important**: This guide uses Micromamba (no root required) and WebRTC streaming (no GUI/X11 needed).

### 1. Install Micromamba (Recommended Package Manager)

Micromamba is perfect for servers without root access - installs entirely in your home directory.

```bash
"${SHELL}" <(curl -L micro.mamba.pm/install.sh)
# Restart terminal or source ~/.bashrc
source ~/.bashrc
```

### 2. Install Isaac Sim via Conda

```bash
# Create environment with Isaac Sim
micromamba create -n isaacsim51_py311 -c https://pypi.nvidia.com -c conda-forge isaacsim-rl isaacsim-replicator isaacsim-extscache-physics isaacsim-extscache-kit-sdk isaacsim-extscache-kit isaacsim-app isaacsim=5.1.0 python=3.11

# Activate environment
micromamba activate isaacsim51_py311
```

### 3. Verify Isaac Sim Installation

```bash
# Test headless mode
python -c "from isaacsim import SimulationApp; app = SimulationApp({'headless': True}); app.close()"

# Test GUI mode (if display available)
isaacsim
```

---

## 🔬 Isaac Lab Setup

### 1. Clone Isaac Lab Repository

```bash
cd ~/ws
git clone https://github.com/isaac-sim/IsaacLab.git
cd IsaacLab
```

### 2. Install Isaac Lab

```bash
# Basic installation
./isaaclab.sh --install

# If egl_probe fails (optional dependency), skip it:
pip install -e source/isaaclab_tasks --no-deps
pip install -e source/isaaclab_rl --no-deps
```

### 3. Verify Isaac Lab Installation

```bash
# Run a test environment
./isaaclab.sh -p source/standalone/tutorials/00_sim/create_empty.py
```

---

## 🎮 Running Reinforcement Learning Training

### Train Anymal-C Robot

```bash
cd ~/ws/IsaacLab

# Training
./isaaclab.sh -p scripts/reinforcement_learning/rsl_rl/train.py \
  --task Isaac-Velocity-Rough-Anymal-C-v0 \
  --num_envs 4096 \
  --headless

# Evaluation/Play
./isaaclab.sh -p scripts/reinforcement_learning/rsl_rl/play.py \
  --task Isaac-Velocity-Rough-Anymal-C-v0 \
  --num_envs 20 \
  --livestream 2 \
  --enable_cameras
```

---

## 🌐 WebRTC Livestreaming (Headless Mode)

**This is the primary visualization method for servers without GUI access.**

### Enable WebRTC for Remote Viewing

```bash
# Start training/simulation with livestream
./isaaclab.sh -p scripts/reinforcement_learning/rsl_rl/play.py \
  --task Isaac-Velocity-Rough-Anymal-C-v0 \
  --num_envs 20 \
  --livestream 2 \
  --enable_cameras

# Access via browser on your local machine
# http://<your-server-ip>:8211/streaming/webrtc-client/
```

### WebRTC Client Options

**Option 1: Browser-based (Recommended)**
- Simply open `http://<server-ip>:8211/streaming/webrtc-client/` in Chrome/Firefox
- No installation needed on your local machine

**Option 2: Standalone AppImage Client**
```bash
# Download on your local Linux machine
wget https://github.com/NVIDIA-Omniverse/kit-app-template/releases/download/v1.1.5/isaacsim-webrtc-streaming-client-1.1.5-linux-x64.AppImage
chmod +x isaacsim-webrtc-streaming-client-1.1.5-linux-x64.AppImage
./isaacsim-webrtc-streaming-client-1.1.5-linux-x64.AppImage --no-sandbox
```

### Firewall Configuration (if needed)

If you can't connect, ensure port 8211 is open:
```bash
# Check if port is listening (on server)
ss -tulpn | grep 8211

# Contact your system administrator to open port 8211 if blocked
```

---

## 🐛 Common Issues & Fixes

### Issue 1: h5py Import Error (HDF5 Symbol Not Found)

**Error**: `undefined symbol: H5Pget_fapl_direct`

**Root Cause**: HDF5 library version mismatch between conda and system

**Fix**:
```bash
micromamba activate isaacsim51_py311

# Uninstall problematic h5py
pip uninstall h5py -y

# Install HDF5 via conda first
micromamba install -c conda-forge hdf5

# Reinstall h5py
pip install --no-cache-dir h5py
```

### Issue 2: Multiple Conda Environments Conflict

**Error**: Conda and Micromamba paths interfering

**Fix**:
```bash
# Completely remove old conda
rm -rf ~/miniconda3 ~/.conda

# Clean PATH in ~/.bashrc
nano ~/.bashrc
# Remove all conda/miniforge references

# Restart terminal
source ~/.bashrc
```

### Issue 3: USD Asset Not Found

**Error**: `'None/Isaac/IsaacLab/Robots/ANYbotics/ANYmal-C/anymal_c.usd'`

**Fix**:
```bash
# Install Isaac Lab assets
cd ~/ws/IsaacLab
./isaaclab.sh --install

# Or skip egl_probe if it fails
pip install -e source/isaaclab_tasks --no-deps
pip install -e source/isaaclab_rl --no-deps
```

### Issue 4: GUI Won't Start (X11 Error)

**Error**: `Could not initialize GLX`

**Fix** (for SSH/remote):
```bash
# Use headless mode
export DISPLAY=

# Or use WebRTC livestreaming
--livestream 2
```

### Issue 5: CMake Version Too Old (egl_probe)

**Error**: `Compatibility with CMake < 3.5 has been removed`

**Root Cause**: egl_probe is an optional dependency for GPU device detection

**Fix**: Skip egl_probe installation (not needed for WebRTC headless mode)
```bash
# egl_probe is only for local GPU rendering detection
# Not required when using WebRTC streaming
# Simply continue without it - training will work fine
```

### Issue 6: No Root Access / Cannot Install System Packages

**Solution**: This entire guide uses Micromamba - no root needed!

All packages install to:
- `~/micromamba/` (Micromamba itself)
- `~/micromamba/envs/isaacsim51_py311/` (Isaac Sim environment)
- `~/.local/` (Python user packages)

### Issue 7: Cannot Connect to WebRTC Stream

**Checklist**:
1. Verify server is running with `--livestream 2`
2. Check port 8211 is accessible: `ss -tulpn | grep 8211`
3. Try browser first before AppImage client
4. Check firewall rules with system administrator
5. Use server's actual IP, not `localhost` from your local machine

---

## 📊 Performance Tuning

### Multi-GPU Training

```bash
# Use multiple GPUs
./isaaclab.sh -p scripts/reinforcement_learning/rsl_rl/train.py \
  --task Isaac-Velocity-Rough-Anymal-C-v0 \
  --num_envs 8192 \
  --device cuda:0
```

### Adjust Number of Environments

```bash
# Fewer envs for debugging
--num_envs 32

# More envs for faster training (GPU memory permitting)
--num_envs 8192
```

---

## 📦 Environment Management

### List Environments

```bash
micromamba env list
```

### Remove Environment

```bash
micromamba env remove -n isaacsim51_py311
```

### Export Environment

```bash
micromamba env export -n isaacsim51_py311 > environment.yml
```

### Create from Export

```bash
micromamba env create -f environment.yml
```

---

## 🔗 Useful Resources

- [Isaac Sim Documentation](https://docs.omniverse.nvidia.com/isaacsim/latest/index.html)
- [Isaac Lab Documentation](https://isaac-sim.github.io/IsaacLab/)
- [Isaac Lab GitHub](https://github.com/isaac-sim/IsaacLab)
- [NVIDIA Omniverse](https://www.nvidia.com/en-us/omniverse/)

---

## 📝 Notes

- Always activate the correct conda environment before running Isaac Sim/Lab
- **No Docker or root access needed** - Everything runs in user space via Micromamba
- **WebRTC livestreaming is mandatory** for headless servers without X11/GUI
- GPU memory usage scales with `--num_envs` parameter
- **Port 8211 must be accessible** from your local machine for WebRTC
- For university clusters: contact admin if firewall blocks port 8211

---

