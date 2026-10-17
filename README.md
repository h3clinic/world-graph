# world-graph

**An explicit, versioned 3D scene graph as world memory for consistent video generation, driven by small (≤33B) open models.**

Video world models forget. When the camera looks away and comes back, the room has changed. Recent work fixes this by *learning* a memory (surfel indexes, implicit 3D state, octree latents). `world-graph` takes the other route: keep the world as an explicit, editable scene graph, build it in Blender with code written by a small local LLM, and render depth/control passes that a video model restyles. Because the world lives in a graph and not in model weights, revisiting a place is a lookup, not a hope.

## Pipeline

```
prompt ──► planner LLM ──► scene graph (versioned) ──► bpy code ──► Blender
                 ▲                │                                   │
                 │                └──── diff / edit / commit ◄────────┤
                 │                                                    ▼
                 └──────── render critique (multi-view) ◄──── depth + control passes
                                                                      │
                                                                      ▼
                                                     depth-controlled video model (Wan 2.1 VACE)
                                                                      │
                                                                      ▼
                                                                    video
```

1. **Plan**: a small coder model turns a prompt into a scene graph: objects, attributes, and spatial relations, compiled into numeric layout constraints (after SceneCraft).
2. **Persist**: the graph is versioned. Every edit is a commit you can diff, branch, and roll back.
3. **Build**: the model writes `bpy` scripts executed in Blender over a local socket bridge (blender-mcp style), then checks its work from multi-angle and wireframe renders.
4. **Render**: camera trajectories produce depth and control video from the same persistent scene.
5. **Restyle**: a depth-controlled video model (Wan 2.1 VACE / Fun-Control, 1.3B or 14B) turns control video into final frames.

## What's new here

The bridge and the scene-graph planning already exist. The contributions of this repo are:

- **A persistent, versioned scene graph** that outlives a single generation and can be edited between shots.
- **An evaluation** on *cycle trajectories* (camera leaves a region and returns) reporting revisit PSNR and object-position error, against a plain video model and learned-memory baselines.
- **A small-model constraint**: every model in the loop fits under 33B parameters and is meant to run locally (target: a ~14B coder alongside Wan in 24 GB).

## Related work

### LLM writes Blender code

| Project | Notes |
| --- | --- |
| [blender-mcp](https://github.com/ahujasid/blender-mcp) | Blender add-on with a JSON-over-TCP socket server (port 9876) behind an MCP server. Arbitrary `bpy` execution plus multi-angle and wireframe views. The socket is unauthenticated, so keep it on localhost. |
| [BlenderLLM](https://github.com/FreedomIntelligence/BlenderLLM) | Qwen2.5-Coder-7B fine-tuned for Blender CAD scripts, with the CADBench benchmark. Basic single-turn modeling only; license not stated. |
| [LL3M](https://github.com/threedle/ll3m) ([paper](https://arxiv.org/abs/2508.08228v1)) | Multi-agent plan, retrieve API docs, write, debug, and critique from renders. Design reference only: built on Claude Sonnet 3.7, hosted server discontinued, academic/evaluation-only license. |
| [SceneCraft (ICML 2024)](https://mlanthology.org/icml/2024/hu2024icml-scenecraft) | Scene graph → numeric layout constraints → Blender scripts for scenes up to ~100 assets, refined from renders; scenes also used to guide video generation. No code release found. Template for the scene-graph step. |

### Memory for consistent video

| Project | Notes |
| --- | --- |
| [VMem](https://arxiv.org/html/2506.18903v3) | Indexes past views by the 3D surfels they saw and conditions new frames on the most relevant ones. 15.72 vs 11.82 PSNR on loop trajectories, ~12x faster. No code found. |
| [Video World Models with Long-term Spatial Memory (NeurIPS 2025)](https://proceedings.neurips.cc/paper_files/paper/2025/hash/467655d26fcc207bca08915dc91964c6-Abstract-Conference.html) | Learned long-term spatial memory.* |
| [3D Scene Prompting](https://arxiv.org/html/2510.14945v1) | Implicit 3D scene memory.* |
| [OctWorld](https://arxiv.org/pdf/2609.03919) | Learned 3D memory.* |

\* Described from titles and abstracts only; less verified.

### Depth-controlled video generation

- [Wan 2.1 VACE / Fun-Control ComfyUI workflows](https://www.comfyui.org/en/wan2-1-vace-video-transformation-workflow): restyle depth-style control video. Available in 1.3B and 14B variants.

## Status

Early research scaffold. Nothing to install yet.

## License

TBD.
