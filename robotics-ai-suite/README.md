# Intel Robotics AI Suite — working deployment + notes

Part of the [Intel Physical AI Learning Hub](../). Owner: @ahameed.

**Purpose: a demo / enablement asset.** A reproducible deployment of Intel's
Robotics AI Suite on OpenShift that can be shown to customers, partners or the
team, plus the notes behind it.

Upstream: https://github.com/open-edge-platform/robotics-ai-suite

---

## The demo

Intel's own `llm_bridge.LLMBridge`, from the `llm-robotics` Debian package,
loading Phi-4-mini INT8 through OpenVINO on an Intel Xeon, turning plain English
into robot control code:

```
COMMAND: Pick up the red apple and place it into the green box, then return
         the robot to its default position.

#code start
prepare_state()
target_pose = get_pick_pose('red apple')
move(target_pose)
suck()
target_pose = get_place_pose('green box')
move(target_pose)
release()
prepare_state()
#end
```

One line of Intel's source changed: `device='GPU'` → `device='CPU'`.

**→ [openshift/README.md](openshift/README.md) — runbook, ~25 min from empty project.**

---

## Contents

| | |
|---|---|
| [openshift/](openshift/) | Numbered manifests + runbook. **Start here.** |
| [containers/](containers/) | `Containerfile.robotics-sdk` — the suite image |
| [INSTALLATION_GUIDE.md](INSTALLATION_GUIDE.md) | What the suite is, install paths, doc errata |
| [TESTING.md](TESTING.md) | Measurements and further experiments |
| [technologies/](technologies/) | ROS 2, OpenVINO, VLA, diffusion policies, LLM, CV |
| [demos/](demos/) · [benchmarks/](benchmarks/) · [comparisons/](comparisons/) | Logs, perf data, vs KServe/vLLM |

---

## What's established

**The suite is 167 Debian packages**, not a monolithic OS image — the official
installer just adds four APT repos and runs `apt install`. So it containerizes.

**Repo architectures are `i386 amd64`.** No arm64 build exists. Apple Silicon is
out natively; x86 OpenShift nodes are in. (Emulation does work — see below.)

**No prebuilt suite image is published anywhere.** Checked Docker Hub `openvino/`
and `intel/` (~500 repos) and GHCR. We build our own.

**It runs, verified on ROSA:**
```
openvino 2025.3.0   devices ['CPU']   Intel Xeon Platinum 8375C
ROS 2 Jazzy, 200 packages incl. openvino_node
all six pipeline packages under /opt
model load 33.7s · code generation 12.3–17.6s
```

**The VLA pipelines ship only patch files.** `pi05-ov`, `rdt-ov` etc. contain
patches, not applications — each pipeline is an upstream project
(`huggingface/lerobot`, `tonyzhaozh/act`, `ORB_SLAM3`) plus an Intel patch set.
`llm-robotics` is the exception and ships real code, which is why it's the demo.

**Intel's VLA contribution, measured:** 4014 insertions across 22 files against
lerobot — ~2200 lines of ALOHA MuJoCo sim, ~1100 of conversion/benchmark scripts,
and 196 lines in `modeling_pi05.py` that add the OpenVINO inference path behind
an optional `OVConfig(model_dir, device="CPU")`. Clean optional-backend design.

**Local on Apple Silicon works** under Colima amd64 emulation — OpenVINO installs
and reports `['CPU']` — but it's QEMU TCG, so oneDNN uses scalar fallbacks.
Fine for mechanics, useless for timing. `colima start --vz-rosetta` is faster.

---

## Four defects in Intel's shipping artifacts

1. **Wrong tokenizer IDs.** `llm_bridge.py` hardcodes Qwen's `stop_tokens`
   `[151643, 151645]` and `pad_token_id` against a Phi-4 default model. They never
   fire, so generation runs to the token cap — the demo silently emits an
   incomplete robot program for longer commands.
2. **`apt install openvino` names a package that doesn't exist** in the ubuntu24
   repo. Intel's step-by-step doc is wrong, and `libopenvino` pulls no plugins,
   so a literal reading yields `available_devices == []`.
3. **`git am *.patch` is broken** on the `pi05-ov` deb — a rename puts a bugfix
   patch four positions too early. The upstream README prescribes exactly that glob.
4. **The deb trails the git repo by 3 patches**, missing a performance fix and
   the MuJoCo simulator restructure. Intel's own channels also disagree on
   OpenVINO version: `requirement.txt` says 2026.1, apt installs 2025.3.0.

---

## Known gaps

- **CPU only.** OpenVINO's plugins are intel-cpu/gpu/npu; "GPU" means Intel
  iGPU/Arc, never NVIDIA. NPU needs `/dev/accel` + host driver.
- **No IR models** for the ROS vision nodes — the `ros-jazzy-openvino-node`
  debconf preseed doesn't take.
- **No simulator.** Image is `ros-jazzy-ros-base` (no RViz/Gazebo). Intel ships an
  ALOHA MuJoCo sim in the pi0.5 patches; showing it needs Xvfb + software GL +
  noVNC. Not built.
- **Xeon 2nd/3rd gen**, not Xeon 6. Numbers are a floor.

---

## Status

- [x] Technology stack mapped
- [x] Install path verified against live repos
- [x] Image builds and runs on OpenShift
- [x] Intel's own robotics code executing (llm-robotics)
- [x] Deployment made reproducible — numbered manifests + runbook
- [ ] pi0.5 VLA conversion (checkpoint download in progress; may hit the
      `google/paligemma-3b-pt-224` manual gate)
- [ ] Simulator with noVNC
- [ ] KServe/vLLM comparison write-up
