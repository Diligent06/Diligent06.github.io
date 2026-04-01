# Computer Vision & 3D Libraries

## OpenCV — Build from Source

```bash
cmake -D CMAKE_BUILD_TYPE=RELEASE \
      -D CMAKE_INSTALL_PREFIX=/usr/local \
      -D OPENCV_EXTRA_MODULES_PATH=../opencv_contrib/modules \
      ..
make -j$(nproc)
sudo make install
```

---

## Pinocchio — Environment Variables

```bash
export PYTHONPATH=$PYTHONPATH:/home/diligent/Desktop/third_part_packages/pinocchio/pinocchio_installation/lib/python3.9/site-packages
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/lib
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/home/diligent/anaconda3/envs/bigai-eai/lib
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/home/diligent/Desktop/third_part_packages/pinocchio/pinocchio_installation/lib
export PKG_CONFIG_PATH=$PKG_CONFIG_PATH:/home/diligent/Desktop/third_part_packages/pinocchio/pinocchio_installation/lib/pkgconfig
```
