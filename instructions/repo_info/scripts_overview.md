# bg_rad_core — Scripts Overview

---

## Config / Build Files

### pyproject.toml
Configures `black` (line-length 88) and `ruff` (extending `.ruff-base.toml`) for the `bg_rad_core` repo.

### README.md
Top-level repo README. States `bg_rad_core` hosts RAD-specific core packages; the sole package listed is `bg_mujoco_utils`.

### bg_mujoco_utils/README.md
Package-level usage guide covering two main scripts (`openrave_env_to_mjcf.py`, `urdf_to_mujoco.py`), run modes (direct or `ros2 run`), path modes for meshes (`absolute`, `relative`, `asset-relative`), robot inclusion via `--robot-mjcf`, and environment conversion examples.

### bg_mujoco_utils/CMakeLists.txt
ament_cmake build file. Installs the Python package, all `scripts/*.py` files, `robot_configs/`, and `trajectories/` directories.

### bg_mujoco_utils/package.xml
ROS 2 package manifest (format 2). Version 0.0.1, proprietary license.
- **Build:** `ament_cmake`, `ament_cmake_python`
- **Runtime:** `bg_build`, `bg_logging`, `bg_param`, `bg_planning`, `openrave_colcon`
- **Exec:** `python3-ament-index-python`, `python3-lxml`, `python3-mujoco`, `python3-numpy`

### bg_mujoco_utils/setup.py
Delegates to `bg_build.python_setup.generate_setuptools_setup()` for setuptools configuration.

---

## Library Modules — `bg_mujoco_utils/bg_mujoco_utils/`

### `__init__.py`
Package docstring: "MuJoCo conversion and interop utilities for BG RAD." No code.

---

### `asset_pathing.py`

**Summary:** Shared path-resolution helpers for MuJoCo conversion utilities. Resolves `package://`, `file://`, relative, and absolute mesh URIs to filesystem paths, and renders output mesh paths in multiple modes (absolute, relative, asset-relative).

**Dependencies:** `logging`, `os`, `xml.etree.ElementTree`, `pathlib.Path`, `typing`, `urllib.parse`, `ament_index_python` (optional)

| Function | Parameters | Return | Description |
|----------|-----------|--------|-------------|
| `_candidate_package_roots` | — | `list[Path]` | Returns filesystem roots to scan for `package.xml` when the ament index is incomplete. |
| `_build_package_cache` | — | `Dict[str, Path]` | Builds a package-name → directory cache by recursively finding `package.xml` files. |
| `resolve_package_uri` | `uri: str` | `Path` | Resolves a `package://pkg/path` URI to an absolute filesystem path. |
| `resolve_mesh_uri` | `mesh_uri: str, base_dir: Path` | `Path` | Resolves any mesh URI form to an absolute path. |
| `_cmake_prefix_paths` | — | `list[Path]` | Returns existing `CMAKE_PREFIX_PATH` entries as absolute paths. |
| `resolve_existing_mesh_path` | `raw_path: str` | `Optional[Path]` | Resolves an existing mesh file via cwd and install-prefix heuristics. |
| `render_mesh_path_for_mode` | `resolved_path: Path, output_dir: Path, mode: str, asset_dir: Optional[Path]` | `Tuple[str, Optional[str]]` | Renders mesh path and optional meshdir according to selected path mode. |

---

### `env_source.py`

**Summary:** Environment-loading helpers that fetch OpenRAVE scene XML from either a local bg_param config-loader or Zookeeper-backed parameter storage.

**Dependencies:** `logging`, `os`, `types.SimpleNamespace`, `typing`, `bg_param.commandline`, `bg_param.param`

| Function | Parameters | Return | Description |
|----------|-----------|--------|-------------|
| `_join_namespace` | `namespace: Optional[str], env_name: str` | `str` | Builds a normalized parameter namespace path. |
| `_to_text` | `value: object` | `str` | Converts bytes/objects returned by bg_param to string. |
| `_parse_xacro_arg_pairs` | `xacro_arg_pairs: Optional[Iterable[str]]` | `Optional[list[list[str]]]` | Parses CLI `KEY=VALUE` xacro arguments into bg_param format. |
| `load_env_data_via_config_loader` | `system, environment_name, namespace, overlay, ...` | `Tuple[str, str]` | Loads environment XML using bg_param's config_loader, returning `(env_name, xml_data)`. |
| `load_env_data_from_zookeeper` | `environment_name, namespace, zookeeper_root` | `Tuple[str, str]` | Loads environment XML from Zookeeper-backed bg_param storage. |

---

### `openrave_env_to_mjcf.py`

**Summary:** Converts BG OpenRAVE environments into MuJoCo MJCF scene files. Loads environments via bg_param, resolves parent-frame transforms, extracts geometry and mesh assets from OpenRAVE KinBodies, and emits complete MJCF XML with optional robot inclusion.

**Dependencies:** `argparse`, `hashlib`, `json`, `logging`, `os`, `re`, `dataclasses`, `pathlib.Path`, `typing`, `numpy`, `openravepy`, `lxml.etree`, `bg_logging` (optional), `bg_mujoco_utils.asset_pathing`, `bg_mujoco_utils.env_source`, `bg_planning.openrave.transform_manager`

**Dataclasses:**

| Class | Fields | Description |
|-------|--------|-------------|
| `MeshAsset` (frozen) | `name, file_path, scale_xyz` | A single mesh asset entry for MJCF `<asset>` block. |
| `GeomSpec` | `name, geom_type, pos, quat_wxyz, rgba, size, mesh_name, visible` | Intermediate representation of one MuJoCo geometry element. |
| `BodySpec` | `name, pos, quat_wxyz, geoms` | Intermediate representation of one MuJoCo body with its geometries. |

| Function | Parameters | Return | Description |
|----------|-----------|--------|-------------|
| `_initialize_logging` | `verbose: bool` | `None` | Sets up CLI logging with BG formatting. |
| `_fmt` | `values: Iterable[float]` | `str` | Formats numeric vector for MJCF attributes. |
| `_safe_name` | `raw: str` | `str` | Sanitizes string into a valid MJCF name token. |
| `_unique_name` | `base: str, used: Set[str]` | `str` | Returns unique name by appending numeric suffixes. |
| `_pose_from_transform` | `T: np.ndarray` | `Tuple[np.ndarray, np.ndarray]` | Extracts position and wxyz quaternion from 4×4 transform. |
| `_parse_env_xml` | `environment_data: str` | `etree._Element` | Parses environment XML string into lxml element tree. |
| `_disabled_kinbodies` | `environment_data: str` | `Set[str]` | Returns KinBody names marked `enabled="False"`. |
| `_build_transform_manager_from_env` | `environment_data, env_or, default_frame` | `TransformManager` | Builds transform graph from OpenRAVE XML `<tf>` tags. |
| `_apply_parent_frame_transforms` | `environment_data, env_or` | `None` | Applies parent_frame transforms to live OpenRAVE KinBodies. |
| `_get_rgba` | `geom` | `Tuple[float,float,float,float]` | Extracts diffuse color and transparency as MuJoCo RGBA. |
| `_write_obj_mesh` | `mesh, output_path, dedupe_hash` | `Path` | Writes an OpenRAVE TriMesh to disk as Wavefront OBJ. |
| `_mesh_hash` | `mesh` | `str` | SHA-256 digest of mesh data for deterministic naming. |
| `_resolve_mesh_asset` | `geom, output_xml_path, ...` | `Optional[str]` | Resolves or generates a mesh asset for a geometry. |
| `_collect_body_specs` | `env_or, disabled_kinbodies, ...` | `Tuple[List[BodySpec], List[MeshAsset]]` | Collects MuJoCo body/geom specs from all OpenRAVE KinBodies. |
| `_robot_mjcf_root_tag` | `robot_mjcf: Path` | `Optional[str]` | Returns root XML tag of a robot MJCF file. |
| `_emit_mjcf` | `output_path, model_name, body_specs, mesh_assets, ...` | `None` | Builds and writes the complete MJCF XML file. |
| `_parse_args` | — | `argparse.Namespace` | Parses CLI arguments. |
| `main` | — | `int` | CLI entrypoint: loads env, converts to MJCF. |

---

### `urdf_to_mujoco.py`

**Summary:** Converts URDF robot descriptions to MuJoCo MJCF XML, working around MuJoCo's mesh-loading limitations by resolving mesh URIs and injecting mesh bytes via in-memory assets.

**Dependencies:** `argparse`, `hashlib`, `logging`, `pathlib.Path`, `typing`, `mujoco`, `lxml.etree`, `bg_logging` (optional), `bg_mujoco_utils.asset_pathing`

| Function | Parameters | Return | Description |
|----------|-----------|--------|-------------|
| `_initialize_logging` | `verbose: bool` | `None` | Sets up CLI logging. |
| `_sanitize_name` | `name: str` | `str` | Sanitizes text into MuJoCo-friendly identifier. |
| `_mesh_key_for_resolved_path` | `mesh_path: Path, salt: str` | `str` | Creates deterministic key for in-memory mesh asset injection. |
| `load_urdf_to_mujoco` | `urdf_path: str` | `Tuple[MjModel, Dict[str, Path], int]` | Loads URDF into MuJoCo model using in-memory mesh assets. |
| `save_mujoco_xml` | `model, output_path, mesh_key_to_resolved_path, ...` | `None` | Saves MuJoCo model as XML, rewrites mesh keys to on-disk paths. |
| `convert_urdf_to_mujoco` | `urdf_path, output_path, mesh_file_path_mode, asset_dir` | `str` | High-level conversion: URDF → MJCF, returns output path. |
| `main` | — | `None` | CLI entrypoint. |

---

## Scripts — `scripts/umi_eef_to_joint/`

### `__init__.py`
Empty package initializer.

---

### `trajectory_loader.py`

**Summary:** Utilities for loading end-effector trajectory data from JSON or CSV files, converting poses to numpy arrays, and decoding MongoDB-style binary numpy arrays.

**Dependencies:** `base64`, `csv`, `json`, `pathlib.Path`, `typing`, `numpy`

| Function | Parameters | Return | Description |
|----------|-----------|--------|-------------|
| `build_pose_list` | `ee_poses: List[List[float]]` | `List[Tuple[np.ndarray, np.ndarray]]` | Converts raw 7-element pose rows to (position, quaternion) tuples. |
| `load_trajectory_payload` | `path: Path` | `Dict[str, Any]` | Auto-detects JSON/CSV and loads trajectory data. |
| `_load_json_trajectory_payload` | `path: Path` | `Dict[str, Any]` | Reads and parses a JSON trajectory file. |
| `_load_csv_trajectory_payload` | `path: Path` | `Dict[str, Any]` | Parses CSV trajectory file, grouping by `trajectory_ID`. |
| `filter_entries_by_labels` | `entries, feasible, pick_to_place` | `List[Dict[str, Any]]` | Filters trajectory entries by label values. |
| `decode_numpy_array` | `data_dict: Dict[str, Any]` | `np.ndarray` | Decodes numpy array from MongoDB/JSON format. |
| `load_joint_config_from_trajectory_file` | `trajectory_file, trajectory_index, waypoint_index, use_executed` | `Optional[np.ndarray]` | Loads a specific joint configuration from a trajectories JSON file. |
| `get_trajectory_info` | `trajectory_file: Path` | `List[Dict[str, Any]]` | Returns summary info for all trajectories in a file. |

---

### `ik_solver.py`

**Summary:** Core IK solver using MuJoCo + mink — handles random/CEM seed search, multi-seed trajectory solving, dual collision avoidance (real + padding geometry), pose error calculation, and random-walk seed refinement.

**Dependencies:** `time`, `typing`, `xml.etree.ElementTree`, `mujoco`, `numpy`, `mink`, `tqdm` (optional)

**Class: `IKSolver`**

| Method | Parameters | Return | Description |
|--------|-----------|--------|-------------|
| `__init__` | `model, end_effector_body_name, frame_type, elbow_up, urdf_path, ik_max_iterations, ik_tolerance, num_random_samples, num_seeds_per_pose, enable_collision_check, collision_pairs, collision_min_distance, ...` | — | Initialize solver with model, EE frame, collision config, and seed parameters. |
| `compute_pose_error` | `data, target_position, target_quaternion` | `Tuple[float, float]` | Computes position error (mm) and orientation error (deg). |
| `check_collision_distances` | `configuration, penetration_only, threshold` | `list` | Checks all collision pairs, returns violation tuples. |
| `refine_seeds_by_random_walk` | `position, quaternion, seeds, coll_gid_pairs, ...` | `List[Tuple]` | Greedy random walk to push seeds closer to target with adaptive step. |
| `find_best_seeds_for_pose` | `target_position, target_quaternion` | `List[np.ndarray]` | Finds best initial configs by random FK sampling + collision filtering. |
| `find_best_seeds_with_bayes` | `target_position, target_quaternion, n_rounds, top_fraction, ema_alpha` | `List[np.ndarray]` | CEM-based seed search with iterative distribution narrowing. |
| `solve_ik_for_pose` | `configuration, target_pose, posture_task, orientation_cost, ...` | `Optional[np.ndarray]` | Solves IK for single pose using mink with dual collision limits. |
| `solve_trajectory` | `ee_poses` | `Tuple[List, List, List]` | Solves IK for entire trajectory with multi-seed lookahead. |
| `solve_trajectory_with_initial_config` | `ee_poses, initial_joint_config` | `Tuple[List, List, List]` | Solves trajectory from provided initial config. |
| `solve_trajectory_with_bayes` | `ee_poses` | `Tuple[List, List, List]` | Solves trajectory using CEM-based initial-pose search. |

---

### `evaluator.py`

**Summary:** Trajectory evaluation — computes error statistics, diagnostics (spikes, joint jumps, persistent errors), and generates human-readable reports.

**Dependencies:** `typing`, `numpy`

**Class: `TrajectoryEvaluator`**

| Method | Parameters | Return | Description |
|--------|-----------|--------|-------------|
| `__init__` | `pos_error_threshold, angle_error_threshold` | — | Initialize with error thresholds. |
| `evaluate_trajectory` | `pos_errors, angle_errors` | `Dict[str, Any]` | Computes statistics (failed/max/mean errors, threshold exceedances). |
| `compute_diagnostics` | `pos_errors, angle_errors, joint_configs, ee_poses` | `Dict[str, Any]` | Computes error spikes, joint jumps, persistent high-error regions. |
| `print_diagnostics` | `diagnostics, trajectory_idx` | `None` | Prints diagnostics in readable format. |
| `print_trajectory_summary` | `trajectory_index, source_index, end_effector, phase, stats, diagnostics` | `None` | Prints summary for a single trajectory. |
| `print_overall_summary` | `results` | `None` | Prints aggregate summary across all trajectories. |
| `get_error_details` | `pos_errors, angle_errors` | `Dict[str, Any]` | Returns percentile distributions (p50, p90, p95, p99). |

---

### `analyze_trajectories.py`

**Summary:** Main orchestration script — loads trajectory data, sets up MuJoCo models, runs IK solving (sequential or parallel via `ProcessPoolExecutor`), evaluates results, and exports solved trajectories to CSV.

**Dependencies:** `argparse`, `csv`, `os`, `sys`, `tempfile`, `threading`, `time`, `traceback`, `concurrent.futures`, `multiprocessing`, `pathlib.Path`, `typing`, `mujoco`, `numpy`, `lxml.etree`, `ik_solver`, `evaluator`, `trajectory_loader`, `tqdm` (optional)

**Class: `ModelLoader`**

| Method | Parameters | Return | Description |
|--------|-----------|--------|-------------|
| `load_scene_only` | `scene_xml_path` | `MjModel` | *(static)* Loads scene XML into MuJoCo model. |
| `load_scene_with_robot` | `robot_xml_path, scene_xml_path, ee_body_name, skip_merge` | `MjModel` | *(static)* Merges robot + scene XMLs into single model. |
| `merge_robot_and_scene_xml` | `robot_xml_path, scene_xml_path, ee_body_name` | `Tuple[str, str]` | *(static)* Merges XMLs, returns `(merged_xml, base_dir)`. |
| `load_from_xml_string` | `merged_xml, base_dir` | `MjModel` | *(static)* Loads model from pre-merged XML string. |

**Key Constants:** `DEFAULT_ROBOT_BODIES`, `DEFAULT_SCENE_GEOMS`, `DEFAULT_SCENE_GEOMS_REAL`, `DEFAULT_SCENE_GEOMS_PADDING`, `DEFAULT_SELF_COLLISION_PAIRS`, `DEFAULT_COLLISION_PAIRS`, `DEFAULT_PADDING_COLLISION_PAIRS`

| Function | Parameters | Return | Description |
|----------|-----------|--------|-------------|
| `_filter_valid_geom_names` | `model, names` | `List[str]` | Filters to only existing geom/body names. |
| `_resolve_geom_names` | `model, names` | `List[str]` | Resolves body names to their child geom names. |
| `_build_collision_pairs` | `model, pairs` | `List[Tuple[List[str], List[str]]]` | Resolves all geom names in collision pair specifications. |
| `_solve_single_trajectory` | `idx, entry, merged_xml, ...` | `dict` | Worker function for parallel IK solving. |
| `visualize_single_trajectory` | `model, joint_configs, ee_poses, ...` | `None` | Visualizes single trajectory in MuJoCo viewer. |
| `_write_solved_csv` | `csv_path, all_solved, model, ...` | `None` | Writes solved trajectories to CSV. |
| `evaluate_all_trajectories` | `trajectory_file, robot_xml, scene_xml, ...` | `None` | Evaluates all trajectories with optional parallelism. |
| `analyze_single_trajectory` | `trajectory_file, trajectory_index, ...` | `None` | Analyzes a single trajectory with optional visualization. |
| `main` | — | `None` | CLI entry point. |

---

### `viz_solved_trajectories.py` (umi version)

**Summary:** Reads solved IK joint-space trajectories from CSV and replays them in MuJoCo viewer with FK fidelity verification and optional collision detection.

**Dependencies:** `argparse`, `csv`, `sys`, `threading`, `time`, `collections`, `pathlib.Path`, `typing`, `mujoco`, `numpy`

| Function | Parameters | Return | Description |
|----------|-----------|--------|-------------|
| `load_solved_trajectories` | `csv_path, episode_index` | `Tuple[Dict, List[str]]` | Loads solved trajectories from CSV. |
| `load_model` | `robot_xml, scene_xml` | `MjModel` | Loads MuJoCo model from scene XML. |
| `verify_trajectory_fidelity` | `model, frames, ee_body_name, tol_mm` | `Tuple[bool, List[str]]` | Verifies FK of joint configs matches CSV EE poses. |
| `detect_collisions` | `model, data` | `List[Dict]` | Detects collisions in current configuration. |
| `visualize_trajectory` | `model, frames, episode_index, ...` | `None` | Replays trajectory in MuJoCo viewer. |
| `main` | — | `None` | CLI entry point. |

---

## Scripts — `scripts/policy_next_eef_to_action/`

### `__init__.py`
Package docstring declaring three core modules: `gen_initial_pose`, `next_joint_pose`, `eef_pose_generator`.

---

### `eef_pose_generator.py`

**Summary:** Policy trajectory simulator that loads real EEF trajectories from CSV and provides initial poses + delta actions, simulating VLA policy output for end-to-end pipeline testing.

**Dependencies:** `argparse`, `dataclasses`, `sys`, `time`, `pathlib.Path`, `typing`, `mujoco`, `numpy`, `scipy.spatial.transform` (optional), `trajectory_loader` (umi), `gen_initial_pose`, `next_joint_pose`

**Dataclasses:**

| Class | Fields | Description |
|-------|--------|-------------|
| `TrajectoryInfo` | `traj_id, num_frames, feasible, pick_to_place` | Metadata for a loaded trajectory. |
| `DeltaAction` | `delta_pos, delta_quat, rotvec, frame_id, target_frame_id, is_last_frame` | Delta between two trajectory frames. |

**Class: `EEFTrajectoryStore`** — Read-only trajectory store indexed by `source_index`.

| Method | Parameters | Return | Description |
|--------|-----------|--------|-------------|
| `get_trajectory_ids` | — | `List[int]` | Sorted list of available trajectory IDs. |
| `get_initial_pose` | `traj_id` | `Tuple[np.ndarray, np.ndarray]` | Frame 0 (position, quat_wxyz). |
| `get_delta` | `traj_id, frame_id` | `DeltaAction` | Delta between frame_id and frame_id + 1. |
| `iterate_deltas` | `traj_id` | `Iterator[DeltaAction]` | Yields DeltaAction for each consecutive frame. |
| `iterate_deltas_downsampled` | `traj_id, factor` | `Iterator[DeltaAction]` | Downsampled delta iteration. |

**Class: `EEFPoseGenerator`** — Wraps store for data access + optional IK integration.

| Method | Parameters | Return | Description |
|--------|-----------|--------|-------------|
| `initial_pose` | `traj_id` | `Tuple[np.ndarray, np.ndarray]` | First frame pose. |
| `next_delta` | `traj_id, frame_id` | `DeltaAction` | Delta to next frame. |
| `run_trajectory` | `traj_id, solver_ctx, njp_solver, ...` | `Dict[str, Any]` | Full pipeline run: initial pose → step through deltas. |

---

### `gen_initial_pose.py`

**Summary:** Generates collision-free initial joint pose from a single EEF pose using 4-stage pipeline: IK candidates → hull collision check → posture-based re-ranking → final FK verification. Defines shared posture constants.

**Dependencies:** `argparse`, `sys`, `time`, `pathlib.Path`, `typing`, `mujoco`, `numpy`, `ik_solver.IKSolver`, `analyze_trajectories.ModelLoader/DEFAULT_COLLISION_PAIRS/_build_collision_pairs`

**Key Constants:** `POSTURE_REFERENCE_7DOF`, `POSTURE_RANKING_WEIGHTS_7DOF`, `POSTURE_RANKING_BLEND_WEIGHT`, `POSTURE_IK_COSTS_7DOF`, `POSTURE_IK_BLEND_WEIGHT`, `DEFAULT_TOTE_HULL_COLLISION_PAIRS`, `MAX_INIT_POSE_ATTEMPTS`

**Class: `SolverContext`** — Holds MuJoCo model, IKSolver, and hull collision pair IDs.

| Function | Parameters | Return | Description |
|----------|-----------|--------|-------------|
| `expand_joint_values_to_nq` | `model, joint_values_7` | `np.ndarray` | Expand 7-DOF to full nq-length qpos. |
| `create_solver` | `scene_xml, robot_xml, urdf_path, ...` | `SolverContext` | Factory: builds pre-configured IKSolver. |
| `_check_hull_collisions` | `model, data, hull_pair_ids, threshold` | `List[Tuple]` | Check tote hull collisions via `mj_geomDistance`. |
| `generate_initial_joint_pose` | `ctx, eef_position, eef_quaternion, ...` | `Optional[np.ndarray]` | 4-stage pipeline: IK → hull check → re-rank → verify. |
| `generate_initial_joint_pose_with_retry` | `ctx, eef_position, eef_quaternion, ...` | `Tuple[Optional[np.ndarray], float, int]` | Retry wrapper with RNG re-seeding. |
| `main` | — | `None` | CLI entry point. |

---

### `gen_initial_pose_candidate.py`

**Summary:** Drop-in alternative for `gen_initial_pose.py` — decouples seed generation using local `CandidateSeedGenerator` for FK seeds (Stage 1a) then refines through external IKSolver (Stage 1b). Stages 2–4 identical.

**Dependencies:** `argparse`, `sys`, `time`, `pathlib.Path`, `typing`, `mujoco`, `numpy`, `mink`, `ik_solver`, `analyze_trajectories`, `candidate_seed_generator`

**Class: `SolverContext`** — Like `gen_initial_pose.SolverContext` but also holds `seed_generator`.

| Function | Parameters | Return | Description |
|----------|-----------|--------|-------------|
| `create_solver` | `...` | `SolverContext` | Builds IKSolver + CandidateSeedGenerator. |
| `generate_initial_joint_pose` | `ctx, eef_position, eef_quaternion, ...` | `Optional[np.ndarray]` | FK seeds → IK refinement → hull check → re-rank → verify. |
| `main` | — | `None` | CLI entry point. |

---

### `next_joint_pose.py`

**Summary:** Real-time IK tracker (`NextJointPoseSolver`) — computes collision-free next joint config from current state + delta EEF pose, using warm-started IK with posture regularisation and 4-tier retry cascade. Designed for ~10 Hz.

**Dependencies:** `argparse`, `dataclasses`, `sys`, `time`, `pathlib.Path`, `typing`, `mujoco`, `numpy`, `mink`, `scipy` (optional), `ik_solver.IKSolver`, `gen_initial_pose`

**Dataclasses:**

| Class | Fields | Description |
|-------|--------|-------------|
| `StepResult` | `q, pos_error_mm, angle_error_deg, target_pos, target_quat, converged, collision_free, hull_collision_free, error_msg, was_clamped, original_target_pos, solve_tier` | Result of a single step. |
| `CollisionReport` | `is_collision_free, real_violations, hull_violations` | Collision check result. |

**Class: `NextJointPoseSolver`**

| Method | Parameters | Return | Description |
|--------|-----------|--------|-------------|
| `step` | `current_q, delta_pos, delta_rot, rot_type, delta_frame` | `StepResult` | Open-loop: apply delta to current EEF, solve IK. |
| `step_to_pose` | `current_q, target_pos, target_quat_wxyz` | `StepResult` | Closed-loop: solve IK toward absolute target. |
| `get_current_eef_pose` | `q` | `Tuple[np.ndarray, np.ndarray]` | FK: joint config → EEF pose. |
| `validate_joint_config` | `q` | `CollisionReport` | Check config for real + hull collisions. |

| Function | Parameters | Return | Description |
|----------|-----------|--------|-------------|
| `create_next_joint_pose_solver` | `ctx, ...` | `NextJointPoseSolver` | Factory: creates ready-to-use solver. |
| `main` | — | `None` | CLI entry point. |

---

### `test_pipeline.py`

**Summary:** End-to-end IK pipeline test — runs all trajectories through `gen_initial_pose` → `next_joint_pose`, collects per-frame/per-trajectory statistics, supports parallel execution, and saves CSV compatible with `viz_solved_trajectories.py`.

**Dependencies:** `argparse`, `csv`, `dataclasses`, `os`, `sys`, `threading`, `time`, `concurrent.futures`, `multiprocessing`, `pathlib.Path`, `typing`, `numpy`, `tqdm` (optional), `eef_pose_generator`, `gen_initial_pose`, `next_joint_pose`

**Dataclasses:**

| Class | Key Fields | Description |
|-------|------------|-------------|
| `FrameResult` | `frame_id, converged, pos_error_mm, angle_error_deg, joint_config, ...` | Per-frame result. |
| `TrajectoryStats` | `traj_id, num_frames, error_rate, largest_gap, ...` | Per-trajectory statistics. |
| `PipelineReport` | `trajectory_stats, overall_error_rate, total_time_s, ...` | Global aggregate report. |

| Function | Parameters | Return | Description |
|----------|-----------|--------|-------------|
| `run_single_trajectory` | `store, solver_ctx, njp_solver, traj_id, ...` | `TrajectoryStats` | Runs one trajectory through full pipeline. |
| `run_pipeline` | `trajectory_file, feasible, ...` | `PipelineReport` | Orchestrator: runs all trajectories (sequential or parallel). |
| `print_pipeline_report` | `report` | `None` | Prints formatted results table. |
| `save_solved_trajectories_csv` | `report, output_path` | `None` | Saves to CSV for viz_solved_trajectories.py. |
| `main` | — | `None` | CLI entry point. |

---

### `viz_solved_trajectories.py` (policy version)

**Summary:** Visualizes solved IK trajectories from CSV in MuJoCo viewer with EEF trajectory overlay (colour-coded by IK status: blue=Success, orange=Out-of-Limits, magenta=Collision), original UMI reference path, and FK fidelity verification.

**Dependencies:** `argparse`, `csv`, `select`, `sys`, `threading`, `time`, `collections`, `pathlib.Path`, `typing`, `mujoco`, `numpy`, `trajectory_loader` (umi)

| Function | Parameters | Return | Description |
|----------|-----------|--------|-------------|
| `load_solved_trajectories` | `csv_path, episode_index` | `Tuple[Dict, List[str]]` | Loads solved trajectories from CSV. |
| `load_original_trajectory_positions` | `trajectory_path` | `Dict[int, np.ndarray]` | Loads full original UMI positions keyed by source_index. |
| `load_model` | `robot_xml, scene_xml` | `MjModel` | Loads MuJoCo model. |
| `verify_trajectory_fidelity` | `model, frames, ee_body_name, tol_mm` | `Tuple[bool, List[str]]` | FK verification of joint configs vs CSV EE poses. |
| `_update_eef_overlay` | `viewer, all_reference_pos, ...` | `None` | Update EEF trajectory overlay with colour-coded trail. |
| `visualize_trajectory` | `model, data, viewer, frames, ...` | `bool` | Replays trajectory with overlay, returns viewer status. |
| `main` | — | `None` | CLI entry point. |

---

## Scripts — `scripts/viz_utils/`

### `convert_joint_to_ee_trajectories.py`

**Summary:** Converts joint-space trajectory data to EEF pose trajectories by running MuJoCo FK for each waypoint.

**Dependencies:** `argparse`, `base64`, `json`, `sys`, `pathlib.Path`, `typing`, `mujoco`, `numpy`, `viz_joint_trajectories.load_scene_with_robot`

| Function | Parameters | Return | Description |
|----------|-----------|--------|-------------|
| `_decode_numpy_payload` | `payload` | `np.ndarray` | Decodes binary numpy array from JSON format. |
| `_load_trajectory_data` | `path` | `Any` | Loads parsed JSON trajectory file. |
| `_build_joint_index_map` | `model, joint_names, joint_prefix` | `List[int]` | Maps joint names to qpos indices. |
| `_frame_pose_wxyz` | `data, frame_id, frame_type` | `List[float]` | Returns [x,y,z,qw,qx,qy,qz] from body/site. |
| `main` | — | `None` | CLI entry point. |

---

### `viz_collision_bodies.py`

**Summary:** Visualizes collision geometry with 12-category colour-coding (robot visual/collision, tote hull, padding, etc.) for inspection.

**Dependencies:** `argparse`, `enum`, `time`, `dataclasses`, `pathlib.Path`, `typing`, `mujoco`, `mujoco.viewer`

**Enum: `GeomCategory`** — 12 categories: `ROBOT_VISUAL`, `ROBOT_COLLISION`, `ROBOT_EXCLUDED`, `VISUAL_ONLY`, `TOTE_DIVISION`, `TOTE_HULL`, `TOTE_HULL_PAIR`, `PADDING_GUARD`, `TOTE_WALL_FLOOR`, `ROI_BOX`, `REAL_COLLISION`, `UNKNOWN`

| Function | Parameters | Return | Description |
|----------|-----------|--------|-------------|
| `classify_geom` | `model, geom_id` | `ClassifiedGeom` | Classifies a single geom by priority rules. |
| `classify_all_geoms` | `model` | `List[ClassifiedGeom]` | Returns classification for every geom. |
| `apply_collision_colors` | `model, classifications, ...` | `None` | Mutates model.geom_rgba based on classification. |
| `main` | — | `None` | CLI entry point. |

---

### `viz_csv_trajectory.py`

**Summary:** Visualizes a CSV EEF trajectory as a colored line overlay in MuJoCo viewer with downsampling.

**Dependencies:** `argparse`, `math`, `sys`, `time`, `pathlib.Path`, `typing`, `mujoco`, `mujoco.viewer`, `numpy`, `trajectory_loader`

| Function | Parameters | Return | Description |
|----------|-----------|--------|-------------|
| `_extract_positions` | `payload, trajectory_index` | `np.ndarray` | Extracts XYZ positions from trajectory entry. |
| `_downsample_indices` | `num_points, max_segments, stride` | `np.ndarray` | Computes downsampled index array. |
| `_add_line_segments` | `viewer, positions, indices, ...` | `None` | Draws line segments in viewer's user scene. |
| `main` | — | `None` | CLI entry point. |

---

### `viz_ee_trajectories.py`

**Summary:** Animates robot following EEF pose trajectories by solving IK (via mink) for each waypoint — global random-seed search for frame 0, then continuous tracking.

**Dependencies:** `argparse`, `base64`, `json`, `tempfile`, `time`, `pathlib.Path`, `typing`, `mujoco`, `mujoco.viewer`, `numpy`, `lxml.etree`, `mink`

| Function | Parameters | Return | Description |
|----------|-----------|--------|-------------|
| `load_scene_with_robot` | `robot_xml_path, scene_xml_path, ee_body_name, skip_merge` | `MjModel` | Merges robot + scene XMLs and loads combined model. |
| `solve_ik_for_pose` | `configuration, ee_task, target_pose, ...` | `np.ndarray` | Solves IK for single EE pose using mink. |
| `_find_best_seeds_for_pose` | `model, target_pos, target_quat, ...` | `List[np.ndarray]` | Random FK sampling for initial seeds. |
| `visualize_trajectory` | `model, ee_poses, ...` | `None` | Computes IK for all poses then animates in viewer. |
| `main` | — | `None` | CLI entry point. |

---

### `viz_initial_poses.py`

**Summary:** Visualizes initial EEF target poses as colored spheres in MuJoCo, computes IK solutions with collision checking and random-walk refinement, and cycles through seeds interactively.

**Dependencies:** `argparse`, `sys`, `time`, `threading`, `pathlib.Path`, `typing`, `mujoco`, `mujoco.viewer`, `numpy`, `trajectory_loader`, `ik_solver`, `analyze_trajectories`

| Function | Parameters | Return | Description |
|----------|-----------|--------|-------------|
| `build_ik_solver` | `model, elbow_up, ...` | `IKSolver` | Constructs IKSolver with collision pairs matching analyze_trajectories. |
| `compute_raw_seeds` | `ik_solver, ee_poses, ...` | `list` | Generates FK seeds, filters by collision, refines via random walk. |
| `visualize_all` | `model, entries, ...` | `None` | Overlays all trajectories as colored spheres. |
| `visualize_single_with_seeds` | `model, entries, traj_idx, ...` | `None` | Visualizes single trajectory + cycles through IK seeds. |
| `main` | — | `None` | CLI entry point. |

---

### `viz_joint_trajectories.py`

**Summary:** Loads joint-space trajectories from JSON and replays in MuJoCo viewer with configurable speed, looping, and step-through modes.

**Dependencies:** `argparse`, `base64`, `json`, `sys`, `time`, `pathlib.Path`, `typing`, `tempfile`, `mujoco`, `mujoco.viewer`, `numpy`, `lxml.etree`

| Function | Parameters | Return | Description |
|----------|-----------|--------|-------------|
| `load_scene_with_robot` | `robot_xml_path, scene_xml_path, ee_body_name, skip_merge` | `MjModel` | Merges robot + scene XMLs and loads combined model. |
| `_load_trajectory_data` | `path` | `Any` | Loads JSON trajectory file. |
| `_build_joint_index_map` | `model, joint_names, joint_prefix` | `List[int]` | Maps joint names to qpos indices. |
| `main` | — | `None` | CLI entry point with time-based/step-through playback. |
