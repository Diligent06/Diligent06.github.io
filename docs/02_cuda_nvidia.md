# CUDA & NVIDIA Driver Setup

## CUDA 12.1 Environment Variables

```bash
# Add to ~/.bashrc for permanent effect
export PATH=/usr/local/cuda-12.1/bin:$PATH
export LD_LIBRARY_PATH=/usr/local/cuda-12.1/lib64:$LD_LIBRARY_PATH

# Or use the unversioned symlink (points to current CUDA)
export PATH=/usr/local/cuda/bin:$PATH
```

> **Note:** Alternatively, add `/usr/local/cuda-12.1/lib64` to `/etc/ld.so.conf` and run `sudo ldconfig`.

## Switching Between CUDA Versions

```bash
# Add to ~/.bashrc, replacing "version" with e.g. "12.1", "11.8"
export PATH=/usr/local/cuda-<version>/bin:$PATH
export LD_LIBRARY_PATH=/usr/local/cuda-<version>/lib64:$LD_LIBRARY_PATH
```

## Install CUDA from Runfile (No Driver Override)

```bash
# Download runfile from https://developer.nvidia.com/cuda-downloads
sh cuda_xxx.run --extract-only <your_path>

# Copy executables to bin, and copy nvvm folder from cuda-nvcc into the target cuda directory
cp -r <extracted>/cuda-nvcc/nvvm /usr/local/cuda-<version>/
```
