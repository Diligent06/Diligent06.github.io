# ML Models & AI APIs

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
