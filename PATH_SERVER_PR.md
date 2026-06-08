# [PR] feat: Core MAPF Path Server, Plan Executor, and Visual Tooling

## Overview

This Pull Request introduces the foundational Multi-Agent Pathfinding (MAPF) trajectory generation, decoupled execution tracking, mock simulation scaffolding, and basic visual verification tools for the Next Generation RMF prototype. 

By cleanly separating high-level conflict-free path planning from runtime trajectory execution, this architecture enables highly scalable fleet coordination, dynamic participant discovery, explicit progress signaling via safe zones, and rapid visualization.

*(Note: Destination Server and Reservation System components have been cleanly decoupled and will be introduced in a subsequent PR.)*

---

## Architectural Components & Packages

### 1. Planning & Execution (Decoupled Pipeline)
- **`rmf_path_server`:** Core ROS 2 Rust planning node wrapping the MAPF backend (`pibt`). Implements asynchronous background planning. It also supports traffic dependency calculation which can then be used for dynamic safe zone grid allocation, traffic dependency visualization, footprint enforcement, and explicit waypoint release signaling by the `rmf_plan_executor` and dashboard.
- **`rmf_plan_executor`:** Runtime execution coordinator. Subscribes to generated multi-agent trajectories, manages dynamic participant tracking, and synchronizes real-time waypoint progress via explicit `PlanRelease` signaling.

### 2. Interactive Demos & Visual Tooling
- **`rmf_path_server_demo`:** An interactive web dashboard (HTML5/CSS3/VanillaJS) and scenario runner (`demo.launch.py`) allowing users to interactively spawn robots, click to assign goals, and observe dynamic conflict-free planning on a live HTML canvas.
- **`rmf_path_server_test`:** Visual tooling and automated verification suite. Provides robust visualization tracking via **`visualize.py`** alongside multi-robot intersection and follow scenarios.

### 3. Simulation & Supporting Infrastructure
- **`rmf_mock_robot_sim`:** Lightweight Python simulation node modeling waypoint following behavior and publishing active robot state.
- **`rmf_participant_discovery`:** Extended participant lifecycle messages (onboarding/offboarding) supporting fleet discovery.

---

## Verification & Testing

### Setup & Building

1. **Import Workspace Dependencies (via `.repos` file)**  
   Create a workspace and import the required ROS 2 Rust and navigation dependencies using a `.repos` file (such as `setup.repos`):
   ```bash
   mkdir -p rmf_ws/src
   cd rmf_ws
   vcs import src < setup.repos
   ```

2. **Build the PR Packages**  
   Ensure you are at the workspace root inside the `jazzy` distrobox container, then build the relevant packages:
   ```bash
   colcon build --packages-select rmf_prototype_msgs rmf_participant_discovery rmf_path_server rmf_plan_executor rmf_mock_robot_sim rmf_path_server_demo rmf_path_server_test
   ```

### Running Automated Integration Tests
Verify core scenario coordination and robust following behavior:
```bash
colcon test --packages-select rmf_path_server_test --event-handlers console_direct+
```

### Running the Interactive Web Demonstration
Launch the fully standalone path server dashboard:
```bash
ros2 launch rmf_path_server_demo demo.launch.py
```
1. Open `http://localhost:8080` in your web browser.
2. Click **Add Robot** to drop active participants onto the canvas.
3. Select a robot and click a cell to place its goal.
4. Click **Send Scenario** to observe multi-agent trajectory generation and live execution progress.

Note: The visualizers are largely vibe-coded and meant to be used during development and debugging. I fully expect `rmf-site` to replace them in our first release. I would not personally bother with scrutinizing the quality of the code there.

Additional note: Currently `mapf_post` is pulled via git directly from the `arjo/feat/more_efficient_checks` branch.
