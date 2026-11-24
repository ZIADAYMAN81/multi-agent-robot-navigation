# Multi-Agent Library Robot Navigation (Actor–Critic + PSO)

> AI Planning & Robotics coursework, The British University in Egypt.

This project simulates four autonomous robots moving through a library and learning, with reinforcement learning, to reach goals without colliding with walls, furniture, moving cleaning robots or each other. The robots share information with each other, and **Particle Swarm Optimisation** tunes how much each robot listens to the others.

## Environment
- A **Pygame** 2D library map (1500 px wide) with walls, shelves, tables, chairs and a central divider.
- **Dynamic obstacles:** "cleaner" robots that move around the map.
- Collision detection against static obstacles, cleaners and other robots.

## Agents
| | |
|---|---|
| Robots | 4 independent agents |
| Policy | One **Actor–Critic** network per robot (`tf.keras.Model`) |
| Observation | 8 values in [−1, 1]: position (x, y), heading (cos θ, sin θ), direction to goal (dx, dy), distance to nearest obstacle, velocity |
| Actions | 5 (movement + rotation) |
| Reward | `0.5 × progress toward goal − 0.001 per step`; **−1** on collision; **+10** on reaching the goal |
| Training | Discounted returns (γ = 0.99), entropy bonus, Adam |

**Communication.** Each agent's critic value is mixed with the others' through a learned weight vector (`apply_communication`, β = 0.1). Agents can therefore account for how promising their teammates' states look.

## Optimisation pipeline
1. **Stage 1: PSO over the communication weights** (6 particles × 5 iterations). The best score rose from 115.9 to **125.1**.
2. **Stage 2: PSO over the network hyperparameters,** using short evaluation runs. Hidden size settled at 95.
3. **Final training** of all 4 agents with the optimised settings. Weights are saved and reloaded for inference.
4. **Inference scenarios:** *Cross the Divider*, *Parallel Navigation* and *Circular Formation* (targets placed around tables).

## Results and limitations
The optimisation stages work, and communication weights found by PSO beat random initialisations. In the saved inference run, however, the agents **didn't reliably complete** the hardest scenario. For example, in *Cross the Divider* Agent 0 finished 626 px from its goal. Likely improvements:
- longer training (the episode budget was small);
- curriculum learning, starting with short distances;
- a shared critic or centralised-training/decentralised-execution (e.g. MAPPO).

## Run it
```bash
pip install -r requirements.txt
jupyter notebook notebooks/multi_agent_navigation.ipynb
```
A Pygame window opens for the inference scenarios. Close it to continue.

## Tech stack
Python · TensorFlow / Keras · Pygame · NumPy · Particle Swarm Optimisation
