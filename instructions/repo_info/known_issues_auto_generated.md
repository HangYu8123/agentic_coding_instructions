# bg_rad_core — Known Issues (Auto-Generated)

*Auto-generated during repository initialization. Review and triage manually.*

---

## Architecture Issues

### 1. Cross-Pipeline Dependency: Pipeline C → Pipeline B

**Description:** `gen_initial_pose.py` and `gen_initial_pose_candidate.py` (Pipeline C — real-time policy execution) directly import `ModelLoader`, `DEFAULT_COLLISION_PAIRS`, and `_build_collision_pairs` from `analyze_trajectories.py` (Pipeline B — batch trajectory analysis). This creates a hard upward dependency from the real-time execution pipeline into the batch orchestration script.

**Root Causes:** `gen_initial_pose.py` and `gen_initial_pose_candidate.py` importing from `analyze_trajectories.py`.

**Consequences:** Any refactor of `analyze_trajectories.py` (renaming constants, restructuring) can silently break Pipeline C. The real-time policy path cannot be tested or deployed without dragging in Pipeline B's batch-processing dependencies (`csv`, `concurrent.futures`, `tqdm`, etc.).

---

### 2. `ModelLoader` and Collision Constants Buried in Orchestration Script

**Description:** `ModelLoader`, `DEFAULT_COLLISION_PAIRS`, `DEFAULT_SELF_COLLISION_PAIRS`, `DEFAULT_SCENE_GEOMS`, `_build_collision_pairs`, and `_resolve_geom_names` are defined inside `analyze_trajectories.py` — a large CLI orchestrator. These are core model-loading and collision-configuration primitives used by at least 5 other modules.

**Root Causes:** `analyze_trajectories.py` conflating orchestration logic with reusable model/collision utilities.

**Consequences:** Every consumer must import an orchestration script to get basic utilities. Unit-testing `ModelLoader` requires the entire `analyze_trajectories` module. Adding a new pipeline/tool requires coupling to the batch analyzer.

---

### 3. Duplicated `load_scene_with_robot()` Implementations

**Description:** `load_scene_with_robot()` is independently implemented in `viz_ee_trajectories.py`, `viz_joint_trajectories.py`, and as `ModelLoader.load_scene_with_robot()` in `analyze_trajectories.py`. `convert_joint_to_ee_trajectories.py` imports from `viz_joint_trajectories`, creating a fragile chain.

**Root Causes:** `viz_ee_trajectories.py`, `viz_joint_trajectories.py`, `analyze_trajectories.py` each defining their own copy.

**Consequences:** Bug fixes to XML merging logic must be replicated across three files. Behavioral drift between implementations can cause subtle model differences. A visualization script becomes a runtime dependency of a data-conversion utility.

---

### 4. Duplicated `viz_solved_trajectories.py` Files

**Description:** Both `scripts/umi_eef_to_joint/viz_solved_trajectories.py` and `scripts/policy_next_eef_to_action/viz_solved_trajectories.py` independently implement `load_solved_trajectories()`, `load_model()`, and `verify_trajectory_fidelity()` with the same signatures.

**Root Causes:** Two separate `viz_solved_trajectories.py` files with duplicated CSV parsing and FK verification logic.

**Consequences:** CSV format changes require synchronized edits in two files. FK verification tolerance divergence could cause one visualizer to accept trajectories the other rejects.

---

### 5. Duplicated IK Solving Logic in `viz_ee_trajectories.py`

**Description:** `viz_ee_trajectories.py` contains its own `solve_ik_for_pose()` and `_find_best_seeds_for_pose()` that replicate core `IKSolver` functionality, using `mink` directly rather than the shared `IKSolver` class.

**Root Causes:** `viz_ee_trajectories.py` reimplementing `ik_solver.IKSolver` methods inline.

**Consequences:** IK improvements (CEM seeds, random-walk refinement, dual collision limits) in `IKSolver` are not reflected in the visualizer. The visualizer may produce different IK solutions than the pipelines, generating misleading visual feedback.

---

### 6. Divergent `SolverContext` Classes

**Description:** `gen_initial_pose.py` defines `SolverContext(model, data, ik_solver, hull_pair_ids)`. `gen_initial_pose_candidate.py` defines its own `SolverContext` adding `seed_generator`. Both have identically-named `create_solver()` factories with different signatures.

**Root Causes:** `gen_initial_pose.py`, `gen_initial_pose_candidate.py` with no shared ABC.

**Consequences:** Cannot swap initial-pose strategies without changing solver context wiring. `test_pipeline.py` is implicitly coupled to one specific `SolverContext` shape. Adding a third strategy requires yet another clone.

---

### 7. Hardcoded Collision Pair Constants

**Description:** All collision pair constants (`DEFAULT_ROBOT_BODIES`, `DEFAULT_SCENE_GEOMS`, `DEFAULT_COLLISION_PAIRS`, `DEFAULT_TOTE_HULL_COLLISION_PAIRS`, etc.) are Python-level constants baked into source code, referencing specific geom/body names.

**Root Causes:** Constants in `analyze_trajectories.py` and `gen_initial_pose.py`.

**Consequences:** Changing cell geometry requires source-code edits. Different RAD cell variants cannot be supported without forking. The split across two files means cell changes require coordinated edits in both.

---

### 8. Posture Constants Spread Across Modules

**Description:** `POSTURE_REFERENCE_7DOF`, `POSTURE_RANKING_WEIGHTS_7DOF`, `POSTURE_RANKING_BLEND_WEIGHT`, `POSTURE_IK_COSTS_7DOF`, and `POSTURE_IK_BLEND_WEIGHT` are defined in `gen_initial_pose.py` and consumed by `next_joint_pose.py` and `eef_pose_generator.py`.

**Root Causes:** `gen_initial_pose.py` as the sole source; consumers import across pipeline boundaries.

**Consequences:** Tuning posture weights requires editing Python source. A/B testing different profiles is not possible without code changes. Deepens coupling between `next_joint_pose.py` and `gen_initial_pose.py`.

---

### 9. Tight Coupling to Single Robot Configuration

**Description:** The entire codebase is specialized for ABB IRB1300 with 7-DOF yawing gripper in `rad_abb_fa` cell. Joint counts are hardcoded (e.g., `expand_joint_values_to_nq` takes `joint_values_7`), posture constants are `*_7DOF`, and `singulator_grasp_frame` is assumed throughout.

**Root Causes:** All `*_7DOF` constants, `expand_joint_values_to_nq`, all `DEFAULT_*` collision pair constants.

**Consequences:** Supporting a different robot, gripper, or cell layout requires forking most scripts. The package name `bg_mujoco_utils` suggests general utility but the implementation is cell-specific.

---

### 10. No Automated Test Suite

**Description:** No `tests/` directory, no `test_` prefixed library modules, and no test dependencies in `package.xml`. `test_pipeline.py` is an integration test script, not a unit test framework runner.

**Root Causes:** Entire repository structure.

**Consequences:** Regressions in core utilities go undetected until integration. Refactoring (e.g., extracting `ModelLoader`, deduplicating `load_scene_with_robot()`) cannot be done safely. CI cannot gate on correctness.

---

### 11. `viz_collision_bodies.py` Uses Independent Geom Classification

**Description:** `viz_collision_bodies.py` defines its own 12-category `GeomCategory` enum and `classify_geom()` rules that must stay in sync with collision pair constants in `analyze_trajectories.py` and `gen_initial_pose.py`, but there is no shared source of truth.

**Root Causes:** `viz_collision_bodies.py` `GeomCategory`/`classify_geom`, `analyze_trajectories.py` `DEFAULT_*` constants, `gen_initial_pose.py` `DEFAULT_TOTE_HULL_COLLISION_PAIRS`.

**Consequences:** Adding a new collision geom requires updating three separate locations. The visualizer may colour-code geoms differently from how the solver treats them.

---

## Code-Level Issues

### 12. Posture Cost 10x Mismatch Between Standard and Bayesian Initial-Pose Search

**Description:** `_get_top_initial_poses_bayes()` uses `posture_cost = 0.5` when `elbow_up` is active, while `_get_top_initial_poses()` uses `posture_cost = 0.05`. The Bayesian version's comment claims it "matches `_get_top_initial_poses()` for consistent behaviour" — but the values differ by 10x.

**Root Causes:** `ik_solver.py` ~line 1696-1700 (`posture_cost = 0.5`) vs ~line 1311 (`posture_cost = 0.05`).

**Consequences:** When `--use-bayes` is active, IK solutions for the initial pose are strongly biased toward `q=0` instead of the true solution. This degrades seed quality, defeats the CEM-based search, and can cause initial-pose generation to fail for in-tote target poses.

---

### 13. CSV Output Writes `qpos[:7]` Instead of Using Actuated-Joint QPos Addresses

**Description:** `save_solved_trajectories_csv()` extracts joint values as `jc[:7].tolist()` — blindly taking the first 7 elements of the full `nq`-length qpos array, while the rest of the codebase carefully maps 7-DOF values via `_actuated_qaddrs` / `expand_joint_values_to_nq()`.

**Root Causes:** `test_pipeline.py` ~line 1609-1613, ~line 1636-1641.

**Consequences:** If the MuJoCo model ever has `nq > 7` (e.g., a free-floating object), the CSV silently contains misaligned joint values. Currently masked because the robot is the only DOF source in the scene.

---

### 14. Multi-Seed Retry (Tier 2) Zeros Non-Joint QPos Addresses

**Description:** `_try_multi_seed()` in `NextJointPoseSolver` generates perturbed seeds with Gaussian noise across all `nq` dimensions, then clips to `[self._jnt_lo, self._jnt_hi]`. Bounds arrays are initialized to `np.zeros(nq)` for non-joint indices, so clipping forces those addresses to exactly 0.

**Root Causes:** `next_joint_pose.py` ~line 299-310 (`_jnt_lo`/`_jnt_hi` init) combined with ~line 846-847 (clip operation).

**Consequences:** If the scene model ever has free joints (quaternion qpos), Tier 2 retry seeds will have zeroed quaternion elements `[0,0,0,0]`, causing `mj_kinematics` to produce NaN. Currently latent because `nq == njnt` for the fixed-base ABB IRB1300.

---

### 15. `_build_success_result` Reports `collision_free=True` Unconditionally

**Description:** `_build_success_result()` always sets `collision_free=True` in the returned `StepResult`, regardless of whether real-geometry collisions were checked or whether the solution came from Tier 3 (collision-relaxed).

**Root Causes:** `next_joint_pose.py` ~line 904 — `collision_free=True` is hardcoded.

**Consequences:** Downstream code relying on `StepResult.collision_free` as a safety indicator will never see `False` for converged solutions, even from Tier 3 results. Misleading for safety-critical consumers.

---

### 16. Deprecated `_get_coll_gid_sets()` with Known Self-Collision Bug Still Present

**Description:** `_get_coll_gid_sets()` flattens all collision pair groups into two flat lists, creating spurious cross-checks when self-collision pairs are included. Its own docstring warns about this and marks it deprecated, but the method remains importable.

**Root Causes:** `ik_solver.py` ~line 607-627.

**Consequences:** If any downstream code calls `_get_coll_gid_sets()`, it will produce false-positive collision detections (self-geom distance = 0) causing valid configurations to be rejected.

---

### 17. Parallel Worker Overhead: Full XML Re-parse Per Process

**Description:** `_solve_single_trajectory` and `test_pipeline` workers use `ProcessPoolExecutor`, where each worker receives `merged_xml` as a string and must call `mj_loadXML` to rebuild the full MuJoCo model from scratch because `MjModel` is not pickle-serializable.

**Root Causes:** `analyze_trajectories._solve_single_trajectory`, `test_pipeline._worker_run_trajectory`.

**Consequences:** Significant startup overhead per worker (XML parsing, mesh loading, model compilation repeated N times). Memory scales linearly with workers. For large scenes with many meshes, this can dominate wall-clock time for short trajectories.

---

### 18. Duplicated `_initialize_logging()` Across CLI Entry Points

**Description:** Both `openrave_env_to_mjcf.py` and `urdf_to_mujoco.py` define identical `_initialize_logging(verbose)` functions with the same optional `bg_logging` import and fallback pattern.

**Root Causes:** `openrave_env_to_mjcf.py`, `urdf_to_mujoco.py`.

**Consequences:** Logging format inconsistencies across tools. Changes to logging setup require touching every script.
