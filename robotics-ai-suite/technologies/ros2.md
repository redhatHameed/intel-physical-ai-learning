# ROS 2 (Robot Operating System 2)

## What It Is
Middleware framework for robot software development - the communication backbone between sensors, AI models, and robot actuators.

## Core Concepts

### Nodes
Independent processes that perform specific tasks (e.g., camera driver, object detector, motion planner)

### Topics
Message streams for publish-subscribe communication
- **Example**: Camera publishes images on `/camera/image_raw` topic
- Subscribers receive messages asynchronously

### Services
Request-response communication pattern
- **Example**: Request robot position, receive coordinates

### Actions
Long-running tasks with feedback
- **Example**: "Move to position" with progress updates

### Launch Files
Orchestration scripts to start multiple nodes together

## In This Suite

ROS 2 is used across all three robot collections:
- **AMR**: Navigation stack, SLAM, sensor fusion
- **Humanoid**: Motion planning, perception, control
- **Stationary Arm**: Vision control, manipulation

## Key Files to Explore
- [ ] Launch files in `src/components/`
- [ ] ROS 2 node implementations
- [ ] Message definitions

## Learning Resources
- Official docs: https://docs.ros.org/en/humble/
- Tutorials: https://docs.ros.org/en/humble/Tutorials.html

## My Experiments

### Date: [YYYY-MM-DD]
**What I tried:**

**What I learned:**

**Issues/Questions:**

---