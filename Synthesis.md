# Synthesis

**Robotics Sandbox — Autonomous Vehicle Simulation & Sensor Fusion Pipeline**

> **Status:** v1.1 (current) → v1.2 in progress (fusion layer) → main
> **Stack:** ROS 2 Humble, CARLA 0.9.16, Docker, Tailscale VPN, Foxglove Studio, PyTorch
> **Role:** Sole developer

---

## Why Synthesis? (Core Motivation)

High-fidelity autonomous vehicle development normally requires expensive, localized hardware to render dense sensor data and run inference. Synthesis decouples compute from user interaction with a remote streaming architecture instead:

* **Deterministic Environment Control:** Synthetic scenes give absolute control over world state — injecting tailored sensor noise profiles and weather (rain, snow) to intentionally close the sim-to-real gap rather than just hoping it's small.
* **Remote Access:** GPU-heavy rendering and physics stay on one host machine; any collaborator connects over Tailscale with a lightweight client, no local powerhouse required.
* **Rapid Dataset Synthesis:** Fast iteration on simulated scenes to generate labeled training data before physical deployment is possible or safe.
* **Bridging the Real-World Data Deficit:** Lets me simulate edge cases that don't exist in real datasets — dense-pedestrian downtown routes, rare vehicle types, low-visibility scenarios.
* **Real-Time Validation:** A dual-tier Region-of-Interest architecture computes live confusion matrices as the simulation runs, surfacing failure cases immediately instead of only after a full training cycle.



Quantitative tracking/confusion-matrix results will be published with the v1.2 fusion update.
---

## System Overview

Three-layer architecture:

| Layer | Stack |
|---|---|
| Simulation Engine | CARLA 0.9.16 (Unreal Engine 4), Windows host |
| Algorithmic Core | `telemetry_bridge` (ROS 2 Humble) + `robot_brain_inference` (PyTorch / CARLA client) — Docker containers |
| UI Client | Foxglove Studio, connected remotely via Tailscale VPN |

<!-- IMAGE: End-to-end pipeline overview -->
<!-- ![Robotics ML Pipeline](./assets/synthesis-pipeline-overview.png) -->
*(Pipeline overview — data collection → synthetic training → sensor fusion → real-world validation)*

The pipeline runs infrastructure and data collection through synthetic training, sensor fusion, real-world validation, and a continuous fine-tune/re-train loop when metrics fail — the same evaluation loop backs both the 2D (YOLO) and 3D (PointPillars) model selection stages.

---

## Network & Container Architecture

Everything is split across two Docker containers on one host, bridged to remote clients over Tailscale rather than exposed directly.

<!-- IMAGE: Docker container / port architecture -->

<!-- ![Container Architecture](./assets/synthesis-container-architecture.png) -->
<img width="1962" height="1740" alt="image" src="https://github.com/user-attachments/assets/651f48d8-1bac-467a-a6e8-25c1396cb6ff" />

*(`telemetry_bridge` — foxglove bridge on :9090, TCP sockets 5555/5556/5557 — and `robot_brain_inference` — traffic loading, device bridge, pre-processing — both bound through the host's Tailscale mesh IP)*

**Why this matters architecturally:** `telemetry_bridge` must use bridge networking, not `--network host` — host networking silently drops the `-p` port mappings Foxglove depends on. That's a real failure mode I hit and fixed, not a theoretical concern.

### ROS 2 Node Communication Model

<!-- IMAGE: Nodes, Topics, and Services dataflow -->
<img width="854" height="480" alt="Nodes-TopicandService" src="https://github.com/user-attachments/assets/65a84061-de3c-472e-acc2-3c1c9f85292e" />

<!-- ![ROS2 Node/Topic/Service Model](./assets/nodes-topic-and-service.gif) -->
*(Animated: how publish/subscribe topics and request/response services move data between nodes and the host)*

---

## Transport Redundancy (In Progress)

<!-- IMAGE: Transport redundancy / multi-WAN concept -->
<img width="1780" height="1770" alt="image" src="https://github.com/user-attachments/assets/153e16ad-6ac3-4a53-be5d-878cc0590881" />

*(Per-node ROS2 callback isolation — one failing node doesn't take the system down. Multi-WAN bonding — cellular + satellite + WiFi fallback — is a pending idea, not yet built.)*

This is deliberately marked as proposed, not built. The node-level isolation (one failing ROS callback doesn't cascade) is live today; the multi-WAN bonding for field-deployed high-uptime scenarios is a documented idea pending outreach, not a finished feature.

---

## Sensor Fusion — Where v1.1 Ends and v1.2 Begins

The video shows v1.1 in its current state. You can see radar tracking working — raw points come in, clusters form, and the UKF predicts vehicle paths. The video quality is a bit rough because I'm still tuning the ROS 2 bridge's publish rate and the Tailscale VPN adds jitter. That's actually why I built the transport redundancy architecture — so a flaky network link doesn't kill the system. The video demonstrates the core functionality; the polish comes after I lock in the sensor fusion layer.



https://github.com/user-attachments/assets/39bd4d05-cb2b-41e4-b455-6166319256ea


Radar processing is built and live today. Camera and LiDAR tooling are the current work. Fusion is the next milestone before a 1.2 branch.

<!-- IMAGE: Current dataflow — built vs proposed -->
<img width="2276" height="1060" alt="image" src="https://github.com/user-attachments/assets/6c1f8591-a41c-433f-923a-ad4a54c96d3e" />

*(Stage A/B: built, live radar processing — DBSCAN clustering, Hungarian track association, UKF/CTRV tracking. Stage C: proposed late object-level fusion pulling in LiDAR and camera tracks.)*

### Fusion Architecture Options Under Consideration

Before committing to one fusion design for v1.2, I mapped out five structures and their real trade-offs rather than picking the first one that sounded state-of-the-art:

<!-- IMAGE: Fusion structure comparison (5 approaches) -->
<!-- ![Fusion Structure Options](./assets/synthesis-fusion-options.png) -->


*(Late/object-level, track-to-track with covariance intersection, early fusion, BEV-space learned fusion, transformer/cross-attention fusion)*

| Structure | Ease | Est. Timeline | Fusion Point |
|---|---|---|---|
| Late / Object-Level | Easiest | Weeks | After per-sensor tracking |
| Track-to-Track + Covariance Intersection | Moderate | Weeks | After tracking, with calibrated uncertainty |
| Early Fusion (raw concat) | Hard | Months | Before any per-sensor processing |
| BEV-Space Learned | Hard | Months | After per-modality encoding, shared spatial grid |
| Transformer / Cross-Attention | Hardest | Months+ | After tokenizing, no fixed grid |

**Decision so far:** starting with **late/object-level fusion** — reuses the per-sensor trackers I already have working for radar, and is the most broadly deployed pattern for traffic monitoring specifically, precisely because it's fastest to build and debug. It structurally can't recover information a sensor discards before tracking, which is the known trade-off I'm accepting for v1.2; track-to-track with covariance intersection is the likely next step once late fusion is validated, since it adds calibrated confidence without jumping straight to a learned BEV encoder.

---

## Map & Scenario Design

Synthesis uses CARLA's built-in maps and actor catalogue to build scenario-specific tests — Town 10 is the closest stock map to a downtown LA / Santa Monica–style layout, and traffic generation is tunable per-scenario (vehicle class, pedestrian density, emergency vehicle behavior) to construct edge cases that don't exist in public datasets.
<img width="1254" height="637" alt="image" src="https://github.com/user-attachments/assets/09b85bf8-25e2-4326-8037-e2f80641709b" />

---

## Repository Structure

```
synthesis/
├── docker-compose.yml
├── shenron/                        # Shenron  experimental tools
│   ├── Data_Collection_Scripts/    # CARLA scenario data collection automation
│   ├── Evaluation_Scripts/          # Benchmark and agent execution evaluation scripts
│   ├── leaderboard/                 # Autonomous Driving Challenge suite
│   ├── team_code/                   # Transfuser model components, pipelines & shell tools
│   │   ├── transfuser.py            # Benchmark/comparison tooling
│   │   ├── sensor_agent.py          # Real-time inference agent skeleton
│   │   └── sim_radar_utils/        # End-to-end radar data processors
│   └── README.md
├── src/                            # ROS 2 custom production bridge environment
│   ├── generate_traffic.py          # T2 — Traffic generation engine
│   ├── ros2_bridge_receiver.py      # T3 — ROS 2 TCP bridge receiver
│   ├── stream_sensors.py            # T4 — Sensor streamer (LIDAR + camera)
│   └── radar_ppi_viewer.py          # T5 — Radar Plan Position Indicator (PPI) Real-time Viewer
└── README.md

```

Full operations manual (network pairing, 5-terminal launch protocol, shutdown procedure) lives in the repo README — this write-up focuses on architecture and decisions, not runbook steps.

---

## What's Next

- [ ] Camera + LiDAR tooling (in progress)
- [ ] Radar noise filtering — current filtering is too aggressive/noisy for clean tracks
- [ ] Late object-level fusion implementation → v1.2 branch
- [ ] Far/Inner ROI ground-truth monitor with live confusion matrix
- [ ] Package loss handling for the new radar data stream over the network link
- [ ] Custom vehicle model import (rigged truck models) for expanded actor catalogue
- [ ] Train fallback CV toolings to fallback when the system is overloaded
- [ ] Implement either early fusion or BEV fusion (Noting here more Vram is required for training)
---

## Citation

```bibtex

@inproceedings{Dosovitskiy17,
  title = {{CARLA}: {An} Open Urban Driving Simulator},
  author = {Alexey Dosovitskiy and German Ros and Felipe Codevilla and Antonio Lopez and Vladlen Koltun},
  booktitle = {Proceedings of the 1st Annual Conference on Robot Learning},
  pages = {1--16},
  year = {2017}
}
```

```bibtex
@INPROCEEDINGS{11310463,
  author    = {Srivastava, Satyam and Li, Jerry and Mishra, Pushkal and Bansal, Kshitiz and Bharadia, Dinesh},
  booktitle = {2025 IEEE 102nd Vehicular Technology Conference (VTC2025-Fall)},
  title     = {A Realistic Radar Simulator for End-to-End Autonomous Driving in CARLA},
  year      = {2025},
  pages     = {1--6},
  doi       = {10.1109/VTC2025-Fall65116.2025.11310463}
}
```

---

## Interview Quick-Summary (60-Second Version)

> *"Synthesis is a remote-streamed CARLA simulation pipeline for autonomous vehicle sensor development. A host machine runs the simulator and GPU-heavy inference inside Docker containers, and collaborators connect over Tailscale with a lightweight Foxglove client — so nobody needs local high-end hardware. Radar processing with clustering and tracking is built and live. I'm finishing camera and LiDAR tooling now, then implementing late object-level fusion for v1.2 — I evaluated five fusion architectures and picked late fusion first because it reuses my existing per-sensor trackers and is fastest to validate, with track-to-track covariance fusion as the planned next step."*
