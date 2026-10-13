# Intel Robotics AI Suite on OpenShift — Runbook

A reproducible deployment of Intel's Robotics AI Suite on OpenShift, ending in a
live demo: **a natural-language instruction turned into robot control code by an
Intel-optimized LLM running on Intel Xeon via OpenVINO.**

Verified end-to-end on ROSA, 2026-10-08.

---

## The demo in one screen

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

That is Intel's own `llm_bridge.LLMBridge` class, from the `llm-robotics` Debian
package, loading Phi-4-mini INT8 through OpenVINO on a Xeon. Exactly one line of
Intel's source is changed: `device='GPU'` → `device='CPU'`.

---

## Prerequisites

- OpenShift cluster with **x86_64 worker nodes** (the Intel repo publishes
  `i386 amd64` only — there is no arm64 build)
- `oc` logged in, able to create projects and builds
- A default StorageClass (manifests use `gp3`)
- ~60 Gi PVC capacity, ~25 min total

No GPU required. No Intel GPU or NPU is usable here anyway — see *Limits*.

---

## Run it

```bash
oc new-project robotics-learning

# 1. storage + build config
oc apply -f 01-storage-and-build.yaml

# 2. build the image (~7 min)
mkdir -p /tmp/rbuild
cp ../containers/Containerfile.robotics-sdk /tmp/rbuild/Dockerfile
cd /tmp/rbuild && oc start-build robotics-sdk --from-dir=. --follow && cd -

# 3. smoke test (~30 s) — stop here if it fails
oc apply -f 02-smoke-test.yaml && oc logs -f job/smoke-test

# 4. export Phi-4 to OpenVINO INT8 (~10 min, one time)
oc apply -f 03-export-phi4.yaml && oc logs -f job/export-phi4

# 5. THE DEMO (~1 min)
oc apply -f 04-llm-robotics-demo.yaml && oc logs -f job/llm-robotics-demo
```

Re-run step 5 any time; steps 1–4 are one-time.

```bash
oc delete job llm-robotics-demo && oc apply -f 04-llm-robotics-demo.yaml
```

---

## Expected smoke-test output

```
openvino : 2025.3.0-19807-44526285f24-releases/2025/3
devices  : ['CPU']
CPU      : Intel(R) Xeon(R) Platinum 8375C CPU @ 2.90GHz
ros2 packages        : 200
ros2 openvino packages: 5
suite packages       : act-ov diffusion-policy-ov idp3-ov llm-robotics pi05-ov rdt-ov
```

Seeing a real Xeon part here is the point. Under emulation (Apple Silicon) this
reads `QEMU TCG CPU version 2.5+` and oneDNN falls back to scalar kernels.

---

## Measured

| | |
|---|---|
| Image build | ~7 min, 832 packages |
| Phi-4 INT8 export | ~10 min, ~4 GB |
| Model load | 33.7 s |
| Code generation | 12.3 s / 17.6 s (~100 tokens, CPU, INT8) |
| Synthetic matmul | 0.154 ms, 6501 infer/s, 27.3 GFLOP/s |

Nodes seen: Xeon Platinum **8375C** (Ice Lake, m6i), **8259CL** (Cascade Lake, m5)
and **8175M** (Skylake, m5). Three CPU generations — the scheduler picks any of
them, so repeat runs vary. To pin the fastest:

```yaml
nodeSelector:
  node.kubernetes.io/instance-type: m6i.2xlarge
```

---

## Limits

**CPU only.** OpenVINO ships `intel-cpu`, `intel-gpu`, `intel-npu` plugins — "GPU"
means *Intel* iGPU/Arc via Level Zero, never NVIDIA. An NVIDIA GPU node does
nothing for this stack. NPU additionally needs `/dev/accel` plus a host driver.

**No IR models for the ROS vision nodes.** The debconf preseed for
`ros-jazzy-openvino-node` does not take, so
`/opt/ros/jazzy/share/openvino_node/data/` is empty. The package ships a
`model_list` describing what it wants. Unfixed.

**No simulator.** The image uses `ros-jazzy-ros-base`, which excludes RViz and
Gazebo. Intel ships an ALOHA MuJoCo sim in the pi0.5 patches, but running any of
it visually needs Xvfb + software GL + a noVNC sidecar. Not built.

---

## Gotchas, all hit for real

| Symptom | Cause |
|---|---|
| `apt install openvino` → no such package | No `openvino` metapackage exists in the ubuntu24 repo. Intel's step-by-step doc is wrong. Install versioned components. |
| `available_devices` → `[]` | `libopenvino` pulls no plugins. Must name `libopenvino-intel-cpu-plugin` and `-ir-frontend` explicitly. |
| `ensurepip is not available` | Ubuntu 24.04 ships no `python3-venv`. OpenShift's random UID blocks runtime `apt`, so it must go in the image. |
| Pod `Pending`, "Insufficient cpu" | 8-core nodes, already loaded. Keep `requests.cpu` ≤ 2. |
| `does not appear to have a file named configuration_phi3.py` | `optimum-cli` leaves `auto_map` in config.json but never copies the module. Strip it; transformers ≥5 has native Phi3. |
| ConfigMap `Too long: may not be more than 262144 bytes` | `oc apply` writes the whole payload into `last-applied-configuration`. Use `oc create`, or tar first. |
| `failing to pull image robotics-sdk:latest` | The ImageStream predates these manifests (e.g. created by `oc new-build`) and has `lookupPolicy.local: false`. Fix: `oc patch is robotics-sdk --type=merge -p '{"spec":{"lookupPolicy":{"local":true}}}'`. A clean `oc apply -f 01-...` sets this correctly. |

---

## Defects found in Intel's shipping artifacts

Worth knowing before relying on any of this.

1. **Wrong tokenizer IDs.** `llm_bridge.py` sets `stop_tokens = [151643, 151645]`
   and `pad_token_id = 151645` — **Qwen** IDs — while `model_path` defaults to
   Phi-4. They never fire on Phi-4, so generation runs to the 100-token cap.
   Visible in the demo: *"Move the blue cup to the tray"* truncates after
   `move(target_pose)`, silently emitting an incomplete robot program.
2. **`git am *.patch` is broken.** The `pi05-ov` deb renames
   `0006-Fix-Bugs-fix.patch` → `0001-Fix-Bugs-fix.patch`, so glob order applies it
   four patches early and it fails on `examples/aloha/eval_aloha.py: does not
   exist in index`. The upstream README tells you to use exactly that glob.
3. **The deb trails the repo by 3 patches** — missing the ALOHA MuJoCo
   restructure, a pos-embedding fp32 *performance* fix, and a tempfile security fix.
4. **Version skew between Intel's own channels.** `llm-robotics`'s
   `requirement.txt` pins `openvino==2026.1`; the apt packages install `2025.3.0`.

---

## Teardown

```bash
oc delete project robotics-learning
```
