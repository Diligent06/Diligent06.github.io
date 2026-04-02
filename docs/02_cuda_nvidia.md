# CUDA & NVIDIA Driver Setup

## CUDA 12.1 Environment Variables

```bash
# Add to ~/.bashrc for permanent effect
export PATH=/usr/local/cuda-12.1/bin:$PATH
export LD_LIBRARY_PATH=/usr/local/cuda-12.1/lib64:$LD_LIBRARY_PATH

# Or use the unversioned symlink (points to current CUDA)
ln -s your_real_cuda_path /usr/local/cuda # create soft link from cuda you actually use to a soft link
export PATH=/usr/local/cuda/bin:$PATH
```


## Install CUDA from Runfile (If you don't have root permission)

```bash
# Download runfile from https://developer.nvidia.com/cuda-downloads
sh cuda_xxx.run --extract-only <your_path>

# Copy executables to bin, and copy nvvm folder from cuda-nvcc into the target cuda directory
cp -r <extracted>/cuda-nvcc/nvvm /usr/local/cuda-<version>/
```
