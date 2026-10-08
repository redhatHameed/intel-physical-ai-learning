# Testing the suite on OpenShift

Cluster: `api.c9q3b0q0y2m7x4a.ylf4.p1.openshiftapps.com` · project `robotics-learning`
Image: `image-registry.openshift-image-registry.svc:5000/robotics-learning/robotics-sdk:latest`

## Working as of 2026-10-08

Built first try, 19 steps, no fixes needed.

```
openvino  2025.3.0-19807
devices   ['CPU']
CPU       Intel(R) Xeon(R) Platinum 8375C @ 2.90GHz    (Ice Lake, m6i)
          Intel(R) Xeon(R) Platinum 8259CL @ 2.50GHz   (Cascade Lake, m5)

end-to-end inference (2x 1024x1024 fp32 matmul + relu, batch 1):
  latency     0.154 ms
  throughput  6501 infer/s
  compute     27.3 GFLOP/s

ROS 2 Jazzy    200 packages, incl. openvino_node / openvino_wrapper_lib / openvino_msgs
suite packages /opt/{pi05-ov,rdt-ov,diffusion-policy-ov,idp3-ov,act-ov,llm-robotics}
```

27 GFLOP/s is modest because batch-1 matmul is memory-bound, not compute-bound. It
proves the stack computes; it is not a peak-throughput figure.

## Rebuild after editing the Containerfile

```bash
cd /path/to/robotics-ai-suite/containers
mkdir -p /tmp/rbuild && cp Containerfile.robotics-sdk /tmp/rbuild/Dockerfile
cd /tmp/rbuild && oc start-build robotics-sdk --from-dir=. --follow
```

## Quick smoke test

```bash
oc run ovtest --rm -i --restart=Never \
  --image=image-registry.openshift-image-registry.svc:5000/robotics-learning/robotics-sdk:latest \
  --command -- python3 -c "
import openvino; c=openvino.Core()
print(c.available_devices, c.get_property('CPU','FULL_DEVICE_NAME'))"
```

## Known gap: no IR models in the image

The debconf preseed for `ros-jazzy-openvino-node` did not take — `/opt/ros/jazzy/share/openvino_node/data/` is empty, so the ROS vision nodes have nothing to run. The
package ships a `model_list` describing what it wants. Either fix the preseed key or
fetch models explicitly with `omz_downloader`. Needed before any `openvino_node` test.

## Next tests, roughly in order of value

### 1. Pin to Ice Lake for the better number
Two CPU generations are in play and the scheduler picks either. `8375C` (m6i) is the
newer part. Pin it:

```yaml
spec:
  nodeSelector:
    node.kubernetes.io/instance-type: m6i.2xlarge
```

### 2. `llm-robotics` — the only package with runnable code
Ships the full app at `/opt/llm-robotics/LLM/` (`llm_bridge.py`, `executor.py`,
`primitives.py`, `requirement.txt`). Needs a Phi-4 OpenVINO model, which is **not** in
the image — `llm_bridge.py` expects `~/ov_models/Phi-4-mini-instruct-int8-ov` and
hardcodes `device='GPU'`, so it needs patching to `'CPU'` on this cluster.

```bash
pip install optimum[openvino]
optimum-cli export openvino --model microsoft/Phi-4-mini-instruct \
  --weight-format int8 ~/ov_models/Phi-4-mini-instruct-int8-ov
```

Put the model on a PVC rather than baking it into the image. This is the most direct
comparison against vLLM — same class of model, different runtime.

### 3. INT8 vs FP32 on the same model
The whole Intel pitch is quantization. `nncf_int8_compression.py` in the source tree
does the conversion. Measure both, on the same node, same input. This is the number
worth putting in the comparison doc.

### 4. The pi0.5 patch set — highest signal, zero infrastructure
No cluster needed. `src/pipelines/pi05-rtc-ov/patches/0003-Feat-Enable-ov-inference-in-pi05-policy.patch`
is ~676 lines touching `modeling_pi05.py` and `configuration_pi05.py`. That diff *is*
the PyTorch→OpenVINO migration for a VLA model. Read it before building anything.

To actually run it:
```bash
git clone https://github.com/huggingface/lerobot && cd lerobot
for p in /opt/pi05-ov/0001-*.patch /opt/pi05-ov/0002-*.patch /opt/pi05-ov/0003-*.patch \
         /opt/pi05-ov/0004-*.patch /opt/pi05-ov/0005-*.patch; do git am "$p"; done
```

### 5. CUDA vs OpenVINO on the same model
`rdt-ov` ships both backends for RDT-1B:
`0002-add-MUJOCO-pipeline-for-cuda.patch` and `0003-add-MUJOCO-OpenVINO-pipeline.patch`.

Same model, same task, two runtimes — Xeon CPU vs NVIDIA L40S. Scale the GPU
machineset only when ready to run it, so it isn't billed while idle:

```bash
oc scale machineset c9q3b0q0y2m7x4a-57rqr-gpu-nodes-us-east-1a \
  -n openshift-machine-api --replicas=1     # g6e.xlarge, 1x L40S 48GB
```

Note OpenVINO cannot target NVIDIA — plugins are intel-cpu/intel-gpu/intel-npu only.
The GPU is for running the *upstream CUDA* path as a baseline, nothing else.

### 6. ROS 2 graph
After fixing the model gap, run `openvino_node` and inspect the pub/sub topology —
that is Intel's perception-to-inference integration pattern, the part with no direct
KServe analogue.

## Ceiling on this cluster

NPU and Intel iGPU are unreachable: NPU wants `/dev/accel` plus a host driver, Intel
iGPU wants `/dev/dri` plus a device plugin, and the GPU nodes are NVIDIA. CPU is the
ceiling for OpenVINO here. Reproducing Intel's published NPU numbers needs Core Ultra
hardware.
