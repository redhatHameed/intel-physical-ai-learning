# Intel Robotics AI Suite — Learning Path

Part of the [Intel Physical AI Learning Hub](../). Owner: @ahameed. Started 2026-10-08.

Upstream: https://github.com/open-edge-platform/robotics-ai-suite
(also vendored at `edge-ai-suites/robotics-ai-suite` in the umbrella repo — same content)

**Why:** compare Intel's model-serving and optimization approach against our
KServe/vLLM-Omni platform, and find anything worth adopting.

## Contents

| | |
|---|---|
| [INSTALLATION_GUIDE.md](INSTALLATION_GUIDE.md) | What the suite actually is, how to install it, two bugs in Intel's docs |
| [containers/](containers/) | `Containerfile.robotics-sdk` — buildable suite image for OpenShift |
| [technologies/](technologies/) | ROS 2, OpenVINO, VLA, diffusion policies, LLM robotics, CV |
| [demos/](demos/) | Hands-on experiment logs |
| [benchmarks/](benchmarks/) | Performance data |
| [comparisons/](comparisons/) | Intel vs KServe/vLLM-Omni |

## What we've established

- The suite is **167 Debian packages** from `amrdocs.intel.com`, not a monolithic
  OS image. The "installer" adds four APT repos and runs `apt install`.
- Repo architectures are **`i386 amd64`** — no arm64 build exists, so Apple Silicon
  is out and OpenShift x86 nodes are in.
- No prebuilt suite container is published anywhere (checked Docker Hub `openvino/`
  and `intel/` orgs, ~500 repos; nothing on GHCR). We build our own.
- The pipeline packages (`pi05-ov`, `rdt-ov`, …) ship **only patch files**. Each
  pipeline is an upstream project (`huggingface/lerobot`, `tonyzhaozh/act`,
  `ORB_SLAM3`) plus an Intel patch set. Running one = clone upstream, `git am`,
  build. Intel's whole VLA contribution is ~7000 lines of diff against lerobot.
- **Local on Apple Silicon works** (tested M3 Pro): amd64 containers run under
  Colima, OpenVINO 2025.3.0 installs, `available_devices` → `['CPU']`. But it's
  QEMU TCG, so oneDNN falls back to reference kernels — functional, not fast,
  and useless for timing. Rosetta (`colima start --vz-rosetta`) is much quicker.
- NPU needs `/dev/accel` + host driver; iGPU needs `/dev/dri` + device plugin;
  RealSense needs a DKMS kernel module. None of those work in a container.

## Status

- [x] Technology stack mapped
- [x] Docs site building locally (needs `brew install enchant`; `make serve` fails
      on macOS because `flock` is missing — run the `npm ci && npm run build &&
      npm run serve` steps from `docs/website/` directly)
- [x] Install path verified against the live repos
- [ ] Build the container on OpenShift
- [ ] Run a packaged pipeline, trace how it loads models
- [ ] Write up the KServe/vLLM comparison

## Next

```bash
oc new-project robotics-learning
cd containers/
oc new-build --binary --strategy=docker --name robotics-sdk
oc start-build robotics-sdk --from-dir=. --follow
```

Then confirm the stack is live:

```bash
source /opt/ros/jazzy/setup.bash
python3 -c "import openvino; print(openvino.Core().available_devices)"   # expect ['CPU']
ros2 pkg list | grep openvino
```
