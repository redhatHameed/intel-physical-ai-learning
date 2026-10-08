# Installing the Robotics AI Suite

Verified against the live Intel APT repo on 2026-10-08.

## What the suite actually is

Not a framework, not a pip package. It is **~167 Debian packages** served from
`https://amrdocs.intel.com/repos/{noble,jammy}`. The "installer" is a wrapper that adds
four APT repos and runs `apt install`. That is the whole thing.

```
Origin: eci.intel.com
Codename: amr
Architectures: i386 amd64        <-- no arm64
```

The four repos:

| Repo | Provides |
|---|---|
| `packages.ros.org/ros2` | ROS 2 Jazzy (24.04) / Humble (22.04) |
| `amrdocs.intel.com/repos/noble` | The suite itself — 167 packages |
| `apt.repos.intel.com/oneapi` | Intel compilers, MKL, TBB |
| `apt.repos.intel.com/openvino/2025` | OpenVINO 2025.3.0 runtime |

Because it is plain APT on Ubuntu, **it containerizes cleanly**. My earlier claim that
it's an inseparable OS-level stack was wrong — only the driver layer is.

## What's in the repo

**Physical AI / VLA** — the packaged form of `src/pipelines/*`:
`pi05-ov`, `rdt-ov`, `diffusion-policy-ov`, `idp3-ov`, `act-ov`, `act-ipex`,
`llm-robotics`, `funasr`, `eci-inference-embodied`

**Perception / SLAM / nav:**
`ros-jazzy-collab-slam-{sse,avx2,lze}`, `ros-jazzy-fast-mapping`,
`ros-jazzy-its-planner`, `ros-jazzy-adbscan-ros2`, `ros-jazzy-wandering`,
`ros-jazzy-openvino-yolov8`, `ros-jazzy-pointcloud-groundfloor-segmentation`

**Metapackage:** `ros-jazzy-robotics-sdk` pulls 12 components (collab-slam,
fast-mapping, adbscan-follow-me, gesture-recognition, its-planner,
its-relocalization, picknplace-simulation, groundfloor-segmentation,
realsense2-tutorial-demo, text-to-speech, wandering, rvc-simulation).
`ros-jazzy-robotics-sdk-complete` adds ~20 GB of bag files.

## Two errors in Intel's own step-by-step doc

Found by diffing the documented commands against the published package lists.

**1. `sudo apt install openvino` fails.** There is no `openvino` metapackage in
`openvino/2025 ubuntu24`, and no `openvino-libraries-dev` either — so the doc's
`Pin: openvino` / `Pin: openvino-libraries-dev` stanzas are no-ops. Only versioned
components are published. Install these instead:

```
libopenvino-2025.3.0=2025.3.0.19807
python3-openvino-2025.3.0=2025.3.0.19807
```

**2. `libopenvino` pulls no plugins.** Its only deps are libc/libgcc/libstdc++/libtbb.
Install it alone and `Core().available_devices` returns `[]` — inference fails at
runtime with "device CPU not found". The plugins and frontends are separate:

```
libopenvino-intel-cpu-plugin-2025.3.0     # required for any CPU inference
libopenvino-intel-gpu-plugin-2025.3.0
libopenvino-intel-npu-plugin-2025.3.0
libopenvino-ir-frontend-2025.3.0          # required to read .xml/.bin IR models
libopenvino-onnx-frontend-2025.3.0
libopenvino-pytorch-frontend-2025.3.0
```

`ros-jazzy-openvino-node` hard-pins `libopenvino-2025.3.0 (= 2025.3.0.19807)`, so
version drift breaks the ROS bridge. It also pulls `ros-jazzy-librealsense2`
(userspace lib — installs fine in a container) and `intel-opencl-icd`.

`containers/Containerfile.robotics-sdk` already works around both.

## Three ways to install

### A. Container on OpenShift — recommended for you

`containers/Containerfile.robotics-sdk` is ready. Build it on an x86 node:

```bash
oc new-project robotics-learning
cd containers/
oc new-build --binary --strategy=docker --name robotics-sdk
oc start-build robotics-sdk --from-dir=. --follow

oc run robotics -it --image=image-registry.openshift-image-registry.svc:5000/robotics-learning/robotics-sdk \
  --restart=Never -- bash
```

Then inside:
```bash
source /opt/ros/jazzy/setup.bash
ros2 pkg list | grep openvino
python3 -c "import openvino; print(openvino.__version__); \
  print(openvino.Core().available_devices)"
```

**Works:** ROS 2, OpenVINO on CPU, all the VLA/LLM pipeline packages, colcon builds,
bag-file replay, headless Gazebo.
**Doesn't:** NPU (needs `/dev/accel` + host driver), Intel iGPU (needs device plugin +
`/dev/dri`), RealSense cameras (DKMS kernel module + USB), RViz/Gazebo GUI without a
VNC sidecar.

So: full software stack, CPU-only inference. Fine for learning the architecture and
comparing serving patterns against KServe/vLLM — not for reproducing Intel's
NPU/iGPU benchmark numbers.

### B. Intel cloud VM — closest to a real install

Any Intel-Xeon instance (AWS `c7i`/`m7i`, GCP `C3`, Azure `Dv5`) with Ubuntu 24.04:

```bash
wget https://amrdocs.intel.com/downloads/robotics-installer
wget https://amrdocs.intel.com/downloads/robotics-installer.sha256
sha256sum -c robotics-installer.sha256 && chmod +x robotics-installer
sudo -E ./robotics-installer     # pick the "complete" profile
```

Real install, kernel modules, reboot, GUI via VNC. Still no NPU or Intel iGPU —
cloud Xeons don't have them. ~$0.50/hr, a few dollars for a day of experimenting.

### C. Intel Core Ultra hardware — only path to NPU/iGPU

An Intel Core Ultra NUC or laptop with Ubuntu 24.04, then option B's installer plus
the NPU driver (step 7 of the step-by-step guide). This is the only configuration
where `openvino.Core().available_devices` reports `['CPU', 'GPU', 'NPU']` and the
published benchmarks reproduce.

### Not an option: Apple Silicon

`Architectures: i386 amd64`. No arm64 build exists. Docker's x86 emulation will
install the packages but OpenVINO's x86 kernels under QEMU are slow enough to be
useless for anything but a smoke test.

## Suggested path

1. Build the container, run it on OpenShift, confirm `available_devices` → `['CPU']`.
2. Run a packaged pipeline (`pi05-ov`, `llm-robotics`) and read how it loads models.
3. Compare that loading/serving path against your KServe/vLLM-Omni setup — notes in
   `comparisons/kserve-vllm-comparison.md`.
4. Only rent a Core Ultra box if you need real NPU/iGPU numbers.
