# Robotics Simulation Environments

## Coordinate Conventions

| Environment   | Front | Up  | Left |
|---------------|-------|-----|------|
| Habitat       | +Z    | —   | +Y   |
| ManiSkill3    | +X    | +Z  | +Y   |

> **Habitat extra:** down = +X direction.

**RGB axis colors (standard):**
- Red → X axis
- Green → Y axis
- Blue → Z axis

---

## RLBench

### Change Camera Resolution

```python
from rlbench.environment import Environment
from rlbench.observation_config import ObservationConfig, CameraConfig

cam_config = CameraConfig(image_size=(256, 256))
obs_config = ObservationConfig(cam_config, cam_config, cam_config, cam_config, cam_config)
env = Environment(action_mode, obs_config=obs_config)
```

> Remember to set the same resolution in every VoxPoser environment wrapper.

---

## ManiSkill3

- **Action space:** `xyzw` (quaternion order)

### ReKep in ManiSkill3 — Fix Render Shapes

Edit `mani_skill/utils/structs/link.py`, line 140:

```python
if obj.entity.find_component_by_type(
        sapien.render.RenderBodyComponent
) is not None:
    all_render_shapes.append(
        obj.entity.find_component_by_type(
            sapien.render.RenderBodyComponent
        ).render_shapes
    )
```

---

## RoboVerse (MuJoCo / IsaacLab)

### Fix CuRobo CUDA Graph Issue

`/home/diligent/RoboVerse/third_party/curobo/src/curobo/opt/particle/particle_opt_base.py` line 207:

```python
self.use_cuda_graph = False
```

### Initial Joint Configuration (Franka)

```python
init_q = tensor([[ 0.0400,  0.0400,  0.0000, -0.7854,
                   0.0000, -2.3562,  0.0000,  1.5708,
                   0.7854]], device='cuda:0')
```

### Add Segmentation to Isaac Process

1. Add `segmentation` field to `CameraState` in `state.py`.
2. Return it from `get_states()` in `isaaclab.py`.
3. Access via `metasim_env.step()` or `metasim_env.reset()`.

### Add Semantic Tags to Objects

- Add `semantic_tags` argument to `_add_object` and `add_robot` in `isaaclab_help.py` for each object type (articulated, primitive cube, etc.).
- Add `semantic_tags` to `BaseObjCfg` and `BaseRigidObjCfg` in `objects.py`.

### Isaac-Sim UI Robot Debugging

1. Import robot → add **Articulation Root** in the Property window.
2. Enable **Instanceable** in the Property window.
3. Open **Tool → Physics → Physics Inspector** to drag and control joints.

> **Note:** When adding joints to a robot config in RoboVerse, the joint name list must be sorted **alphabetically**.

---

## VoxPoser

Key files for migration:

| File | Purpose |
|------|---------|
| `LMP.py` & `LLM_cache.py` | Core LMP logic |
| `planners.py` | Path optimization |

---

## Manipulate Anything

```bash
python dataset_generator.py \
    eval.checkpoint=./ckpt/pick_and_place.pth \
    eval.mask_thresh=0.0 \
    eval.retract=0.20 \
    rlbench.task_name="lamp_on"
```

---

## hab_franka Robot

```python
import pinocchio as pin

pin_robot = pin.RobotWrapper.BuildFromURDF(
    'robots/hab_franka/panda_arm_umi_hand.urdf',
    package_dirs='./robots/hab_franka/'
)
list_joint_names = [
    'panda_joint1', 'panda_joint2', 'panda_joint3', 'panda_joint4',
    'panda_joint5', 'panda_joint6', 'panda_joint7'
]
ee_link_name = 'panda_hand'
```
