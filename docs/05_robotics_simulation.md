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
