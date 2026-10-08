# Intel Robotics AI Suite - Learning Path

**Part of**: [Intel Learning Hub](../)  
**Date Started**: 2026-10-08  
**Owner**: @ahameed  
**Status**: 🟢 Active

**Source Repository**: https://github.com/open-edge-platform/robotics-ai-suite  
**Purpose**: Explore Intel's robotics AI suite for Physical AI platform integration

---

## 📘 About This Learning Path

Intel Robotics AI Suite is a comprehensive platform for building AI-powered robots with:
- ROS 2 middleware integration
- OpenVINO-optimized AI models
- Physical AI (VLA, Diffusion Policies)
- Vision, navigation, and manipulation stacks

**Why we're learning this:**
Compare Intel's approach to our KServe/vLLM-Omni platform and identify integration opportunities.

---

## 📁 Folder Structure

```
robotics-ai-suite/
├── README.md (this file)
├── technologies/          # Technology deep-dives
│   ├── ros2.md
│   ├── openvino.md
│   ├── vla-models.md
│   ├── diffusion-policies.md
│   ├── llm-robotics.md
│   └── computer-vision.md
├── demos/                 # Hands-on demo experiments
│   ├── object-detection.md
│   ├── llm-robotics-demo.md
│   ├── diffusion-policy.md
│   └── vla-pi05.md
├── benchmarks/            # Performance testing notes
│   └── performance-analysis.md
└── comparisons/           # Platform comparisons
    └── kserve-vllm-comparison.md
```

---

## 🎯 Learning Goals

1. **Understand model serving architecture** - How they serve VLA models, diffusion policies
2. **ROS 2 integration patterns** - Bridges, message formats, latency considerations
3. **Hardware optimization** - OpenVINO tooling, quantization, optimization
4. **Deployment patterns** - Container structure, dependencies, GPU allocation
5. **Benchmarking methodology** - What metrics matter for Physical AI

---

## 🗓️ Learning Path

### ✅ Phase 1: Foundation (Week 1-2)
- [x] Understand technology stack
- [ ] Set up development environment
- [ ] Run documentation site locally

### ⏳ Phase 2: Simple Demos (Week 3-4)
- [ ] Object Detection demo
- [ ] SLAM/Navigation
- [ ] Simulation experiments

### ⏳ Phase 3: AI Robotics (Week 5-8)
- [ ] LLM Robotics Demo
- [ ] Diffusion Policy
- [ ] VLA model (Pi0.5)
- [ ] Benchmarking

### ⏳ Phase 4: Deep Experimentation (Week 9+)
- [ ] Model comparisons
- [ ] Optimization experiments
- [ ] Platform comparison (KServe/vLLM-Omni)

---

## 📝 Quick Links

- [Technology Stack Overview](technologies/)
- [Demo Experiments](demos/)
- [Performance Benchmarks](benchmarks/)
- [Platform Comparisons](comparisons/)

---

## 🔗 Resources

- **Main Documentation**: Run `make serve` → http://localhost:3000
- **Intel Robotics Portal**: https://developer.robotics.intel.com/
- **ROS 2 Docs**: https://docs.ros.org/en/humble/
- **OpenVINO Docs**: https://docs.openvino.ai/