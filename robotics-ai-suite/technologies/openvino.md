# OpenVINO™ (Open Visual Inference & Neural Network Optimization)

## What It Is
Intel's AI inference optimization toolkit for accelerating models on Intel hardware (CPU/GPU/NPU)

## Core Workflow

```
PyTorch/TensorFlow → ONNX → OpenVINO IR → Deploy (CPU/GPU/NPU)
    (Training)       (Export) (Optimize)    (Inference)
```

## Key Concepts

### Model Conversion
1. **Export to ONNX**: `torch.onnx.export(model, ...)`
2. **Convert to IR**: `onnx_to_ov_ir.py` → `.xml` + `.bin` files
3. **Load in OpenVINO**: `OVModelForCausalLM.from_pretrained()`

### Quantization
- **FP32 → INT8**: 4x smaller, faster inference
- **NNCF**: Neural Network Compression Framework
- **PTQ**: Post-Training Quantization (no retraining needed)

### Device Targeting
```python
ov_config = {
    "PERFORMANCE_HINT": "LATENCY",  # or "THROUGHPUT"
    "NUM_STREAMS": "1",
    "CACHE_DIR": "model_cache"
}
model = OVModelForCausalLM.from_pretrained(
    model_path,
    device='GPU',  # or 'CPU', 'NPU'
    ov_config=ov_config
)
```

## In This Suite

### Model Optimization Examples
- `src/pipelines/vla-pi0.5-openvino/nncf_int8_compression.py`
- `src/pipelines/vla-pi0.5-openvino/onnx_to_ov_ir.py`
- `src/pipelines/vla-pi0.5-openvino/convert_pytorch_onnx.py`

### Inference Examples
- `src/pipelines/llm-robotics-demo/LLM/llm_bridge.py`
- Object detection with YOLO models

## Performance Characteristics

| Device | Use Case | Typical Latency |
|--------|----------|-----------------|
| CPU | Deterministic control | Varies |
| iGPU | Vision models, LLMs | Medium |
| NPU | Lightweight perception | Low |

## Key Files to Explore
- [ ] Model conversion scripts
- [ ] Inference code in pipelines
- [ ] OpenVINO configs

## Learning Resources
- Official docs: https://docs.openvino.ai/
- Model zoo: https://github.com/openvinotoolkit/open_model_zoo

## My Experiments

### Date: [YYYY-MM-DD]
**Model optimized:**

**Quantization approach:**

**Performance results:**

**Learnings:**

---