# Duckietown-RL — [Modified Implementation of Sim2Real RL based E2E Vehicle Control](https://arxiv.org/abs/2012.07461)  
(ENPM667 Project)

This repository is a modified fork of the original paper [implementation](https://github.com/kaland313/Duckietown-RL
).

This fork contains all patches required to run training + evaluation on a laptop GPU (GTX 1650 and 8GB RAM) including:

- GPU-compatible Docker workflow
- Reduced batch sizes and memory-friendly hyperparameters
- Fixes for rendering, wrapper compatibility, and video saving
- Script updates to run without GUI (xvfb)
- Logging and reproducibility improvements


## Overview:

The following modifications have been made: 
- Environment & Rendering Fixes
- Enabled headless mode using xvfb-run to support rendering inside Docker.
- Fixed cv2.VideoWriter codecs → playable .mp4 output.
- Wrapper Fixes
- Patched ResizeWrapper (Gym 0.26+ disallowed setting observation_space.shape directly).
- Fixed ForwardObstacleSpawnnigWrapper errors by adding missing attributes and safety checks.
- Added low-memory parameters for GTX 1650
- Added support for saving observations as images, trajectories as .mp4 making evaluation possible entirley headlessly.

  Enabled obstacle spawning through spawn_obstacles: true and spawn_forward_obstacle: true.


## Environment Setup 

We run the simulation inside the docker (headless). To ensure GPU enabled docker follow these steps:
[DockerHub: kaland/duckietown-rl](https://hub.docker.com/repository/docker/kaland/duckietown-rl/)

Requirements: [docker](https://docs.docker.com/get-docker/) or [nvidia-docker](https://github.com/NVIDIA/nvidia-docker) installed on your computer.


1. Pull Docker Image
   `docker pull kaland/duckietown-rl:dtaido5`

2. Run Container with GPU + Mounted Folder

   Replace PATH_TO_LOCAL_FOLDER with your folder containing configs/checkpoints:
   `docker run --gpus all -it --rm  -v PATH_TO_LOCAL_FOLDER:/home/duckie/Duckietown-RL/artifacts   --name dtr_capture kaland/duckietown-rl:dtaido5 /bin/bash`
 
## Training and Evaluation

Since we are running simulation in a headless manner, we are using xfvb. Xvfb is a virtual framebuffer X server that allows graphical applications to run without a physical display, making it useful for automated testing and headless servers.

**Training**

`xvfb-run -a -s "-screen 0 1400x900x24" python -m experiments.train-rllib`

Training logs and checkpoints are stored under: `artifacts/`

**Evaluation**

`xvfb-run -a -s "-screen 0 1400x900x24" python -m experiments.test-rllib`


## Results

- Visualising all trajectories that agent took in the evaluation procedure. Red x shows the starting point of the trajectory.
  
  <img width="640" height="480" alt="Trajectory" src="https://github.com/user-attachments/assets/2928d106-93d8-4214-923f-54c86ade39f9" />

- Demo of PPO trained agent in ETHZ_autolab_technical_track map
  
  https://github.com/user-attachments/assets/5fe7e3b1-edd8-4f23-8f02-ebfe89dfbcba

- Demo of PPO trained agent in huge_loop2
  
  https://github.com/user-attachments/assets/0b73238a-9cc7-4c75-b578-d317d8e86b71

- Salient Object [Visualization](https://arxiv.org/abs/1704.07911) to highlight the visual features essential for lane detection.
 
  https://github.com/user-attachments/assets/e225f8d0-d567-4dcc-9f15-ad868656a3aa

- Salient Object Visualization in huge_loop2 map
  
  https://github.com/user-attachments/assets/164374ab-3330-408c-8e34-0fd04486ff44

- Human controlled agent (teleop) to establish human baseline as mentioned in the paper.
  
  https://github.com/user-attachments/assets/a5204d91-1aee-426b-bab8-483618c6288f



## WandB Results:

- The following image shows the min, max, mean values of the 2 reward policies, velocity (distance based) and orientation based policies.

  <img width="1489" height="640" alt="image" src="https://github.com/user-attachments/assets/85a8578d-216c-4a9b-ba87-fa0bfdf7efa4" />
  Note that the actual training was done for 50 max steps per episode. WandB interpolates these results.

  
- The following image shows the min, max, mean values of the episodic reward.

  <img width="1469" height="298" alt="image" src="https://github.com/user-attachments/assets/4fbf9873-bad3-488e-a21f-4fd6dfb8a28d" />

The full training logs and plots are available at [WandB Project](https://wandb.ai/sid-wakingup-university-of-maryland/duckietown-rllib?nw=nwusersidwakingup)

## Conclusion
While we had to tweak the hyperaparameters like max_steps_episode, batch_size, etc. We were able to reproduce 
the results of the paper with the limted resources of GTX 1650 and 8GB RAM. We were able to show that a PPO trained 
agent performs comparable to that controlled by PID controller. This is significant as the PID controller has access
to high-level data like position and orientation error whereas the PPO agent only has visual signals, images with 
limited FoV. We aim to further evaluate recent RL algorithms like GRPO in this problem scope.
Please find an in-depth analysis of the paper at [ENPM667_PROJECT1.pdf](https://github.com/user-attachments/files/23670587/ENPM667_PROJECT1.pdf)
