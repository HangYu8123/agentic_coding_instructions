# bg_rad_core — Codebase Overview

## Description

`bg_rad_core` is a ROS 2 meta-package centered on `bg_mujoco_utils`, which provides **offline IK solving, trajectory analysis, and real-time policy execution tooling** for the ABB IRB1300 robot with a 7-DOF yawing gripper (6 arm joints + `cup_mount_joint`) operating in a RAD FAT cell (`rad_abb_fa` environment). The repo converts legacy BG environment descriptions (OpenRAVE XML) and robot descriptions (URDF) into MuJoCo MJCF format, then uses the MuJoCo physics engine with the **mink** differential-IK library for:

- **Batch trajectory IK solving** — converting recorded end-effector (EEF) trajectories (from UMI demonstrations) into joint-space trajectories.
- **Real-time VLA policy execution** — translating a learned policy's delta-EEF actions into feasible joint commands at ~10 Hz with collision avoidance.
- **Visualization & diagnostics** — replaying, colour-coding, and analysing trajectories and collision geometry.

The end-effector frame tracked throughout is `singulator_grasp_frame`.

---

## Pipeline Diagram

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    PIPELINE A — Scene & Robot Model Conversion              │
│                           (one-time setup)                                  │
│                                                                             │
│  OpenRAVE env XML (bg_param)        URDF (irb1300_yawing.urdf)             │
│         │                                  │                                │
│         ▼                                  ▼                                │
│  [openrave_env_to_mjcf.py]          [urdf_to_mujoco.py]                    │
│  Uses: env_source.py,               Uses: asset_pathing.py, mujoco        │
│        asset_pathing.py, lxml                                               │
│         │                                  │                                │
│         ▼                                  ▼                                │
│  rad_abb_fa scene MJCF              abb_with_yawing.xml / abb_yawing.xml   │
│         │                                  │                                │
│         └────────────┬─────────────────────┘                                │
│                      ▼                                                      │
│           rad_abb_fa_with_robot.xml  (merged scene + robot)                 │
└──────────────────────┬──────────────────────────────────────────────────────┘
                       │
          ┌────────────┴────────────┐
          ▼                         ▼
┌─────────────────────────┐  ┌────────────────────────────────────────────────┐
│  PIPELINE B             │  │  PIPELINE C                                    │
│  Batch Trajectory IK    │  │  Real-Time Policy Execution                    │
│  (umi_eef_to_joint/)    │  │  (policy_next_eef_to_action/)                  │
│                         │  │                                                │
│  EEF trajectories       │  │  EEF trajectory store (CSV)                    │
│  (JSON/CSV)             │  │         │                                      │
│       │                 │  │         ▼                                      │
│       ▼                 │  │  [eef_pose_generator.py]                       │
│  [trajectory_loader.py] │  │  Loads & serves EEF poses + deltas             │
│  Loads & filters data   │  │         │                                      │
│       │                 │  │         ▼                                      │
│       ▼                 │  │  ┌──────────────────────────────────────┐      │
│  [analyze_trajectories  │  │  │  [test_pipeline.py] (ORCHESTRATOR)   │      │
│   .py] (ORCHESTRATOR)   │  │  │  Calls:                              │      │
│  Calls:                 │  │  │  ├─ eef_pose_generator               │      │
│  ├─ trajectory_loader   │  │  │  │  → get_initial_pose()             │      │
│  ├─ ik_solver           │  │  │  ├─ gen_initial_pose                 │      │
│  │  → solve per pose    │  │  │  │  → generate_initial_joint_pose()  │      │
│  └─ evaluator           │  │  │  └─ next_joint_pose                  │      │
│     → error stats       │  │  │     → step_to_pose() per delta       │      │
│       │                 │  │  └──────────────────────────────────────┘      │
│       ▼                 │  │         │                                      │
│  CSV (joint states +    │  │         ▼                                      │
│  EEF poses + errors)    │  │  CSV (per-frame joints + EEF + errors)         │
│       │                 │  │         │                                      │
│       ▼                 │  │         ▼                                      │
│  [viz_solved_           │  │  [viz_solved_trajectories.py]                  │
│   trajectories.py]      │  │  MuJoCo replay with EEF overlay               │
│  MuJoCo replay          │  │                                                │
└─────────────────────────┘  └────────────────────────────────────────────────┘

Cross-pipeline dependency:
  gen_initial_pose.py (Pipeline C) imports ModelLoader and collision pairs
  from analyze_trajectories.py (Pipeline B)

┌─────────────────────────────────────────────────────────────────────────────┐
│                PIPELINE D — Visualization & Conversion Utilities            │
│                            (viz_utils/ — standalone tools)                  │
│                                                                             │
│  [convert_joint_to_ee_trajectories.py] FK: joint → EEF pose conversion     │
│  [viz_collision_bodies.py]             Colour-coded collision geom viewer   │
│  [viz_csv_trajectory.py]               Overlay CSV EEF path as line segs   │
│  [viz_ee_trajectories.py]              Animate robot following EEF via IK  │
│  [viz_initial_poses.py]                Visualize initial EEF target poses  │
│  [viz_joint_trajectories.py]           Replay joint-space trajectories     │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Key Shared Dependencies

| Module | Used by |
|--------|---------|
| `ik_solver.IKSolver` | `analyze_trajectories`, `gen_initial_pose`, `gen_initial_pose_candidate`, `next_joint_pose`, `viz_ee_trajectories`, `viz_initial_poses` |
| `analyze_trajectories.ModelLoader` | `gen_initial_pose`, `gen_initial_pose_candidate`, `next_joint_pose` |
| `trajectory_loader` | `analyze_trajectories`, `eef_pose_generator` |
| `load_scene_with_robot()` | `convert_joint_to_ee_trajectories`, `viz_ee_trajectories`, `viz_initial_poses`, `viz_joint_trajectories` |
| `mink` (external) | `ik_solver`, `next_joint_pose`, `gen_initial_pose`, `viz_ee_trajectories` |
| `mujoco` (external) | Every script |
| `bg_param` (external) | `env_source` only (scene loading) |
| Robot config XMLs | Every script that instantiates a MuJoCo model |

## External Package Dependencies (from package.xml)

- **Build:** `bg_build`, `bg_cmake`
- **Runtime:** `bg_logging`, `bg_param`, `bg_planning`, `openrave_colcon`
- **Python:** `ament-index-python`, `lxml`, `mujoco`, `numpy`, `mink`
