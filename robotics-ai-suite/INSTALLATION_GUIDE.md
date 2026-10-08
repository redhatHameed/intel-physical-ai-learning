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

### D. Apple Silicon under emulation — works, tested

No arm64 build exists, but amd64 containers run fine under Colima and OpenVINO
initializes. Tested on an M3 Pro, 2026-10-08:

```
$ docker run --rm --platform linux/amd64 ubuntu:24.04 uname -m
x86_64

# after adding the OpenVINO repo and installing the versioned packages:
version: 2025.3.0-19807-44526285f24-releases/2025/3
devices: ['CPU']
cpu:     QEMU TCG CPU version 2.5+
```

`pi05-ov`, `rdt-ov` and `llm-robotics` also install cleanly from the Intel repo.

The caveat is `QEMU TCG`: pure software translation, advertising a minimal CPU
model. oneDNN sees no AVX2/AVX-512 and falls back to reference kernels, so it runs
but slowly, and any timing you collect is meaningless as a proxy for Intel silicon.

Good for install mechanics, reading how pipelines load models, and iterating on the
Containerfile without a cluster round-trip. Use OpenShift the moment you want numbers.

Colima defaults to QEMU (`binfmt_misc` shows `qemu-x86_64`). Rosetta is much faster:

```bash
colima stop && colima start --vz-rosetta
```

## What the packages actually contain

Two different shapes, and it matters.

**`llm-robotics` ships real source.** The complete Python app, immediately runnable:

```
/opt/llm-robotics/LLM/llm_bridge.py          # OVModelForCausalLM, Phi-4 loading
/opt/llm-robotics/LLM/executor.py
/opt/llm-robotics/LLM/main.py
/opt/llm-robotics/LLM/primitives.py
/opt/llm-robotics/LLM/utils/{prompt,funasr_client,tts_client,mobilesam_helper}.py
/opt/llm-robotics/LLM/requirement.txt
/opt/llm-robotics/asr-openvino-demo/models/paraformer_bin.py
```

That makes it the best first thing to run locally — no upstream clone, no patching.

**The VLA packages ship only patches.** `pi05-ov` installs exactly this:

```
/opt/pi05-ov/0001-Feat-Add-environment-requirements.patch
/opt/pi05-ov/0002-Feat-Add-script-for-openvino-conversion.patch
/opt/pi05-ov/0003-Feat-Enable-ov-inference-in-pi05-policy.patch
/opt/pi05-ov/0004-Feat-Optimize-implementation-of-openvino-conversion.patch
/opt/pi05-ov/0005-Feat-Add-ALOHA-pipeline-example-for-sim-and-real.patch
/opt/pi05-ov/README.md
```

Patches, nothing else. The pipelines are **upstream projects plus an Intel patch set**,
which matches `.gitmodules` in the source repo:

| Pipeline | Upstream |
|---|---|
| `pi05-rtc-ov` | `huggingface/lerobot` |
| `act-sample` | `tonyzhaozh/act` |
| `orb-slam3-sample` | `UZ-SLAMLab/ORB_SLAM3` |
| `mpc-demo` | `leggedrobotics/ocs2` |

So running a VLA pipeline is: clone upstream → `git am` Intel's patches → build.
The package README spells out the exact sequence:

```bash
git clone https://github.com/huggingface/lerobot && cd lerobot
git am /opt/pi05-ov/0001-Feat-Add-environment-requirements.patch
git am /opt/pi05-ov/0002-Feat-Add-script-for-openvino-conversion.patch
git am /opt/pi05-ov/0003-Feat-Enable-ov-inference-in-pi05-policy.patch
git am /opt/pi05-ov/0004-Feat-Optimize-implementation-of-openvino-conversion.patch
git am /opt/pi05-ov/0005-Feat-Add-ALOHA-pipeline-example-for-sim-and-real.patch
git am /opt/pi05-ov/0001-Fix-Bugs-fix.patch
```

**This is the most useful thing for our comparison.** Intel's entire VLA contribution
is ~7000 lines of patch against lerobot. `0003-Feat-Enable-ov-inference-in-pi05-policy`
touches `src/lerobot/policies/pi05/modeling_pi05.py` and `configuration_pi05.py` —
that diff *is* the answer to "how do you move a VLA model from PyTorch to OpenVINO,"
in reviewable form, no install required. Read the patches before building anything.

## Suggested path

1. Build the container, run it on OpenShift, confirm `available_devices` → `['CPU']`.
2. Run a packaged pipeline (`pi05-ov`, `llm-robotics`) and read how it loads models.
3. Compare that loading/serving path against your KServe/vLLM-Omni setup — notes in
   `comparisons/kserve-vllm-comparison.md`.
4. Only rent a Core Ultra box if you need real NPU/iGPU numbers.
