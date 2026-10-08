# LLMs for Robot Control

## What It Is
Using Large Language Models to translate natural language commands into executable robot code

## Architecture Pattern

```
Speech → ASR → LLM → Code Generator → Robot Primitives → Actions
  ↓
"Pick up the red apple"
                ↓
        Phi-4 / Qwen2.5VL
                ↓
        prepare_state()
        target = get_pick_pose('red apple')
        move(target)
        suck()
```

## Models in This Suite

### Phi-4 (Microsoft)
- **Path**: Uses `~/ov_models/Phi-4-mini-instruct-int8-ov`
- **Size**: Mini (efficient)
- **Optimization**: INT8 quantized
- **Device**: GPU
- **Purpose**: Generate robot control primitives

### Qwen2.5VL (Multimodal)
- **Capability**: Vision + Language
- **Purpose**: Visual reasoning + action planning

## Robot Primitives

The LLM generates code using these functions:

```python
prepare_state()              # Reset to default position
get_pick_pose(obj_name)      # Get picking position
get_place_pose(obj_name)     # Get placement position
move(target_pose)            # Move arm to position
suck()                       # Activate suction
release()                    # Deactivate suction
```

## Demo: LLM Robotics

**Path**: `src/pipelines/llm-robotics-demo/`

### Components
1. **ASR (FunASR)**: Speech → Text
2. **Vision (MobileSAM)**: Object detection & segmentation
3. **LLM (Phi-4)**: Task planning & code generation
4. **TTS**: Text-to-speech feedback
5. **Robot Control**: Execute primitives via socket

### Data Flow
```
User speaks → ASR → Text command
                ↓
        Camera captures scene
                ↓
        MobileSAM detects objects
                ↓
        LLM generates code
                ↓
        Execute on robot (port 8099)
```

## Key Implementation Details

### LLM Bridge (`llm_bridge.py`)
```python
ov_config = {
    "PERFORMANCE_HINT": "LATENCY",
    "NUM_STREAMS": "1",
    "CACHE_DIR": "model_cache"
}
model = OVModelForCausalLM.from_pretrained(
    model_path,
    device='GPU',
    ov_config=ov_config
)
```

### Executor (`executor.py`)
- Queue-based message passing
- Socket communication to robot controller
- Code verification before execution

## Prompt Engineering

The system uses carefully crafted prompts to:
1. Explain available primitives
2. Provide examples
3. Enforce code-only output
4. Prevent reasoning chains (for efficiency)

## Key Files to Explore
- [ ] `src/pipelines/llm-robotics-demo/LLM/llm_bridge.py`
- [ ] `src/pipelines/llm-robotics-demo/LLM/executor.py`
- [ ] `src/pipelines/llm-robotics-demo/LLM/primitives.py`
- [ ] `src/pipelines/llm-robotics-demo/LLM/main.py`

## My Experiments

### Date: [YYYY-MM-DD]
**Command tested:**

**Generated code:**

**Execution result:**

**Observations:**

---