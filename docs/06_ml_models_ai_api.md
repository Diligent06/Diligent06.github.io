# ML Models & AI APIs

---

## M2T2 Model

### Input Data Format

**Scene inputs:**

```python
{
    'input':    torch.cat([xyz - xyz.mean(dim=0), rgb], dim=1),  # (N, 6)
    'points':   xyz,       # (N, 3)
    'seg':      seg,       # (N,)
    'cam_pose': cam_pose   # (4, 4)
}
```

**Pick inputs (per object):**

```python
{
    'object_inputs':  torch.cat([obj_xyz - obj_xyz.mean(dim=0), obj_rgb], dim=1),  # (M, 6)
    'ee_pose':        ee_pose,               # (4, 4)
    'bottom_center':  bottom_center,         # (3,)
    'object_center':  obj_xyz.mean(dim=0)    # (3,)
}
```

**Dummy/test inputs:**

```python
{
    'object_inputs':  torch.rand(1024, 6),
    'ee_pose':        torch.eye(4),
    'bottom_center':  torch.zeros(3),
    'object_center':  torch.zeros(3)
}
```

---

## Distributed Training

```python
import torch.distributed as dist
from torch.distributed.device_mesh import init_device_mesh

def init_mesh() -> DeviceMesh:
    dist.init_process_group("nccl")
    rank = dist.get_rank()
    world_size = dist.get_world_size()
    torch.cuda.set_device(rank)

    mesh = init_device_mesh(
        device_type="cuda",
        mesh_shape=(2, 2),
        mesh_dim_names=("ip", "tp"),
    )
    return mesh
```

---

## Hugging Face

```bash
# Download a dataset with auth token
wget --header="Authorization: Bearer <your_token>" \
     https://huggingface.co/datasets/Pointcept/scannet-compressed/resolve/main/scannet.tar.gz

# Download a repo to a local directory (using hf CLI)
hf download yjguo/Ctrl-World --local-dir ./Ctrl-World
```

---

## Grasp Detection (GPD)

```bash
./detect_grasps ../cfg/eigen_params.cfg "/home/diligent/tmp/temp.pcd"
./detect_grasps_plot ../cfg/eigen_params_plot.cfg "/home/diligent/Desktop/BIGAI-EAI-Su/test.pcd"
```

> **Grasp metric error:** If the dot product of the object surface normal and gripper in-direction is < 0, the surface normal calculation is wrong — check your normal estimation step.
