> # ⚠️ ARCHIVED — moved into `play_launch`
>
> This repository is **read-only** and no longer developed. Its full history
> and every file were folded into the `play_launch` repository on 2026-08-03
> (`git subtree`, so `git log` there covers this repo's commits too):
>
> **→ https://github.com/NEWSLabNTU/play_launch — directory `src/ros-launch-resolve/`**
>
> The final commit here, `90b18ca`, is the exact tree that was absorbed;
> nothing was left behind.
>
> **Nothing about the layering changed.** This is still RFC-0060 layer 2, still
> its own cargo workspace, still builds and runs with no ROS installation, and
> consumers still depend on layers 1–2 and never on layer 3. What went away is
> the *repository* boundary, which was buying nothing the cargo `exclude` and
> the resolver-is-a-binary process boundary did not already provide, while
> costing three levels of submodule nesting. nano-ros RFC-0060 was amended to
> match (`9baebb2eb`); the reasoning is in
> `docs/design/launch-toolchain-topology.md` in play_launch.
>
> **To build it now:** `git clone https://github.com/NEWSLabNTU/play_launch`
> then `cargo build -p ros-launch-resolve-cli`. No submodules need
> initialising and no ROS install is required — verified, 10.9s from cold.
>
> The `ros-launch-manifest` dependency is now a git dependency pinned to
> `v0.1.0` rather than a submodule.

# ros-launch-resolve

Resolve a ROS 2 launch tree into a checked **SystemModel**.

This is the middle layer of the launch toolchain (RFC-0060):

| layer | repo | needs |
| --- | --- | --- |
| spec, theory, proofs, algorithms | [`ros-launch-manifest`](https://github.com/NEWSLabNTU/ros-launch-manifest) | nothing beyond serde |
| **launch tree → SystemModel** | **this repo** | CPython (for `.launch.py`) |
| Linux runtime + binary | [`play_launch`](https://github.com/NEWSLabNTU/play_launch) | ROS, rclrs, colcon |

## Why it is its own repo

The resolve pipeline used to live inside `play_launch`, next to the ROS graph
client and the process runtime. Anything wanting only "launch tree →
SystemModel" had to link all of it — including `play_launch_msgs`, which is not
a registry crate but is generated from an ament environment by
`colcon-cargo-ros2`. So reusing the resolver implied installing ROS and running
play_launch's colcon setup, which is impossible for the embedded consumers that
want it most.

Splitting it out makes the dependency honest: **this workspace builds under
plain `cargo`, with no ROS, no ament and no colcon.**

## Layout

```
resolve/   the pipeline: launch dump -> manifests -> model -> schedule
cli/       the `ros-launch-resolve` binary (resolve / dump / contract / plot)
parser/    play_launch_parser — launch XML and .launch.py (pyo3)
third-party/ros-launch-manifest   the spec
```

## Status

Extracted from `play_launch` with history preserved (`git log --follow` works
across the move). Wiring the crate boundaries is in progress — see nano-ros
phase-312.
