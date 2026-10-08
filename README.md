# Intel Physical AI Learning Hub 🤖

**Team knowledge base** for learning Intel's Physical AI and robotics platforms.

---

## 🎯 Purpose

This repository contains **learning notes, experiments, and insights** from exploring Intel's Physical AI technologies. It's separate from actual code repositories - focused purely on knowledge sharing and team learning.

**What this is:**
- ✅ Learning documentation and experiment logs
- ✅ Technology deep-dives and concept explanations
- ✅ Performance benchmarks and comparisons
- ✅ Best practices and lessons learned
- ✅ Team collaboration and knowledge sharing

**What this is NOT:**
- ❌ Intel source code (use actual repos for that)
- ❌ Production deployment configs
- ❌ Executable applications

---

## 📚 Learning Paths

### ✅ Active

#### [Robotics AI Suite](robotics-ai-suite/)
**Status**: 🟢 Active  
**Owner**: @ahameed  
**Started**: 2026-10-08

Intel's comprehensive robotics platform combining ROS 2, OpenVINO, and Physical AI models.

**Technologies covered:**
- ROS 2 Humble (robot middleware)
- OpenVINO toolkit (AI optimization)
- VLA models (Pi0.5, RDT-1B, Gr00t)
- Diffusion Policies (imitation learning)
- LLMs for robotics (Phi-4, Qwen2.5VL)
- Computer vision (YOLO, SAM, depth estimation)

**Learning goals:**
- Model serving architecture comparison (vs KServe/vLLM-Omni)
- ROS 2 integration patterns
- OpenVINO optimization workflows
- Performance benchmarking for Physical AI

---

### 🔜 Planned

#### OpenVINO Deep Dive
Standalone OpenVINO optimization toolkit learning
- Model conversion (PyTorch/TF → ONNX → IR)
- Quantization techniques (INT8, INT4)
- Device optimization (CPU/GPU/NPU)
- Custom operations
- Performance profiling

#### Edge AI Deployment
Intel edge computing and deployment strategies
- Container optimization
- K8s at the edge
- Resource-constrained inference
- Multi-device orchestration

#### oneAPI Platform
Unified programming model for Intel hardware
- SYCL programming
- DPC++ (Data Parallel C++)
- GPU acceleration patterns
- Cross-architecture code

#### Hardware Optimization
Platform-specific tuning and best practices
- Intel Core vs Core Ultra
- NPU utilization strategies
- Memory optimization
- Thermal management

---

## 📂 Repository Structure

```
intel-physical-ai-learning/
├── README.md                    # This file
├── robotics-ai-suite/          # Current: Robotics AI Suite learning
│   ├── README.md               # Learning path overview
│   ├── technologies/           # Tech deep-dives (ROS 2, OpenVINO, etc.)
│   ├── demos/                  # Hands-on experiment logs
│   ├── benchmarks/             # Performance analysis
│   └── comparisons/            # Platform comparisons
├── [future] openvino/          # OpenVINO deep dive
├── [future] edge-ai/           # Edge deployment
├── [future] oneapi/            # oneAPI learning
└── [future] hardware-tuning/   # Hardware optimization
```

---

## 🤝 Team Collaboration

### How to Contribute

1. **Start a new learning path:**
   ```bash
   mkdir [technology-name]/
   # Use robotics-ai-suite/ as template
   ```

2. **Document your experiments:**
   - **technologies/**: Concept explanations
   - **demos/**: Hands-on labs with results
   - **benchmarks/**: Performance data
   - **comparisons/**: vs other approaches

3. **Share your insights:**
   - Update the learning path README
   - Add to this main README
   - Present findings to team

### Learning Note Template

Each learning path should follow this structure:
```
technology-name/
├── README.md                 # Overview, goals, status
├── technologies/             # Concept deep-dives
│   ├── concept-1.md
│   └── concept-2.md
├── demos/                    # Hands-on experiments
│   ├── demo-1.md
│   └── demo-2.md
├── benchmarks/               # Performance analysis
│   └── performance.md
└── comparisons/              # Platform comparisons
    └── vs-alternatives.md
```

---

## 📊 Learning Dashboard

| Learning Path | Status | Owner(s) | Started | Last Updated |
|---------------|--------|----------|---------|--------------|
| Robotics AI Suite | 🟢 Active | @ahameed | 2026-10-08 | 2026-10-08 |
| OpenVINO Deep Dive | 🔵 Planned | - | - | - |
| Edge AI Deployment | 🔵 Planned | - | - | - |
| oneAPI Platform | 🔵 Planned | - | - | - |
| Hardware Optimization | 🔵 Planned | - | - | - |

**Legend:**  
🟢 Active | 🟡 On Hold | 🔵 Planned | ✅ Complete

---

## 🔗 Resources

### Intel Official
- [Intel Developer Zone](https://www.intel.com/content/www/us/en/developer/overview.html)
- [Intel Robotics Portal](https://developer.robotics.intel.com/)
- [OpenVINO Documentation](https://docs.openvino.ai/)
- [oneAPI Documentation](https://www.intel.com/content/www/us/en/developer/tools/oneapi/overview.html)

### Source Repositories
- [Robotics AI Suite](https://github.com/open-edge-platform/robotics-ai-suite) - Intel's robotics platform

### Team Resources
- Platform comparison docs
- Internal benchmarks
- Best practices wiki

---

## 💡 Learning Principles

1. **Hands-on first** - Run demos before reading docs
2. **Document everything** - Log successes AND failures
3. **Measure impact** - Benchmark performance changes
4. **Share insights** - Your findings help the team
5. **Compare approaches** - How does this fit our stack?

---

## 🎓 Getting Started

**New to the team?**

1. Pick a technology you're exploring
2. Clone this repository
3. Read the learning path README
4. Follow along with demos
5. Document your own experiments
6. Share what you learned

**Questions?** Ask in #physical-ai or tag @ahameed

---

## 📝 Related Repositories

- **[robotics-ai-suite-learning](../robotics-ai-suite-learning/)** - Intel's actual code (keep separate!)
- **Your platform repos** - Compare learnings to our approach

---

**Last Updated**: 2026-10-08  
**Contributors**: @ahameed