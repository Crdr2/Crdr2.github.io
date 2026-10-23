---
title: "Project 3: Forward Kinematics"
---

# Project 3: Forward Kinematics

<video controls loop muted playsinline width="100%" poster="poster.png">
  <source src="fetch_wave.mp4" type="video/mp4">
  Your browser cannot play this video. <a href="fetch_wave.mp4">Download it (MP4)</a>.
</video>

*The Fetch robot waving, driven by my finite state machine. Every frame is drawn from the link
poses my runtime publishes on `/xform_world`. Orange links moved during the recording; blue
links stayed still.*

## The robot

The robot is the **Fetch mobile manipulator**, loaded from its real URDF file, unmodified. It
has 26 links and 25 joints. Fifteen of them can move:

- 2 continuous drive wheels;
- a prismatic torso lift;
- a head that pans and tilts;
- a 7-joint arm;
- 3 prismatic joints for the gripper fingers and the torso bellows.

The other 10 joints are fixed mounts for the cameras, the laser and other parts.

My runtime gets the robot as text: a client sends the URDF to the parameter server. It finds
the root link (`base_link`), builds the kinematic tree, and computes forward kinematics with
unit quaternions:

- each joint's transform is `T_origin · T_motion(q)`;
- each link's pose is its parent's pose composed with that joint transform.

Like a ROS system, the runtime is split into small nodes that talk only through
publish/subscribe topics and services, over a rosbridge-style TCP/JSON gateway:

| Node | Job |
|---|---|
| `param_server` | stores the robot description |
| `robot_state_publisher` | turns joint angles into one transform per joint on `/tf` |
| `robot_world_state_publisher` | composes `/tf` into every link's pose in the world, on `/xform_world` |
| `joint_state_publisher` | servos each joint toward its setpoint |
| `finite_state_machine` | runs the choreography |

## The choreography

The choreography (`fetch_wave`) is a table of four states. Each state holds a target position
for a few joints. The finite state machine sends the current state's target on
`/joint_trajectory`. It then watches `/joint_states`, and when every target joint is within
0.02 of its goal, it takes the state's `on_arrival` transition:

| State | Target | Next |
|---|---|---|
| `home` | torso, shoulder, elbow, wrist and head all at 0 | `raise` |
| `raise` | torso up 0.35 m, shoulder lift −1.0 rad, elbow flex −1.2 rad | `wave_left` |
| `wave_left` | shoulder pan 0.8, wrist flex 1.0, head pan 0.6 (rad) | `wave_right` |
| `wave_right` | shoulder pan −0.8, wrist flex −1.0, head pan −0.6 (rad) | `wave_left` |

So the robot stands up, lifts its arm, and then waves from side to side forever, turning its
head to follow its hand. The joint servo closes 20% of the remaining distance to the setpoint
on each of its 10 updates per second, so the motion slows smoothly into every pose. Each move
takes about two seconds.

## How I drew it

The viewer is a separate program, an ordinary client of the runtime over TCP, just like the
autograder. It subscribes to three topics:

- **`/xform_world`**: every link's pose in the world, as a 4×4 matrix in column-major order.
  Each link is drawn as a dot at the translation part of its matrix (entries 12, 13 and 14).
- **`/tf`**: one entry per joint, from parent link to child link. Each joint is drawn as a line
  from the parent's dot to the child's dot, which gives a skeleton of the robot.
- **`/fsm/status`**: the current state of the choreography, shown in the title.

The viewer records the topics for 24 seconds, then renders each frame with matplotlib's 3D
plotting and encodes them into an MP4 with ffmpeg. The runtime publishes 10 times a second, so
the viewer interpolates link positions in between to get a smooth 30 frames per second. Links
whose position changed by more than 1 cm during the recording are colored orange, and the camera
sways slowly so the motion reads in 3D. Fetch's real meshes are not part of the robot
description, so the robot is drawn as a skeleton.

---

Fetch robot description © Fetch Robotics, Inc., [CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/)
(from [fetch_ros](https://github.com/fetchrobotics/fetch_ros)).
