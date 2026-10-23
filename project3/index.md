---
title: "Project 3: Forward Kinematics"
---

# Project 3: Forward Kinematics

<video autoplay loop muted playsinline controls width="100%" poster="poster.jpg">
  <source src="teaser.mp4" type="video/mp4">
  Your browser cannot play this video. <a href="teaser.mp4">Download it (MP4)</a>.
</video>

A Fetch robot says hello. Every pose in this video comes from my own forward-kinematics runtime:
no kinematics or transform libraries, just quaternions and a walk down the URDF tree.

## Try it yourself

Fetch starts at **home**. Pick a state below the robot, and Fetch performs that move as my
runtime recorded it. The FSM only runs forward, so a state further along the cycle is reached by
passing through the ones in between.

Click any part of the robot to see that link's world matrix from `/xform_world`: rotation on the
left, position in orange. Drag to orbit, scroll to zoom.

<iframe src="viewer.html" title="Interactive 3D replay of the Fetch choreography"
        style="width: 100%; aspect-ratio: 16 / 9; min-height: 420px; border: 0; border-radius: 8px;"
        loading="lazy" allowfullscreen></iframe>

## The robot

The Fetch mobile manipulator, loaded from its real, unmodified URDF: 26 links and 25 joints, 15
of which can move. A client sends the URDF to my parameter server.

- `robot_state_publisher` turns joint positions into one transform per joint on `/tf`.
- `robot_world_state_publisher` composes those transforms into every link's pose in the world on
  `/xform_world`.

All the nodes talk through a rosbridge-style TCP/JSON gateway.

## The choreography

`fetch_wave` is a finite state machine with seven states:

**home → hello → wave_out → wave_in → wave_out_again → wave_in_again → sweep → home**

- **hello** raises the torso and lifts the arm out to the left, with the forearm up and the head
  turned toward the hand.
- The four **wave** states swing the forearm by rolling the upper arm ±0.7 rad and flexing the
  wrist.
- **sweep** unfolds the arm into one long arc to the right.

Each state moves on when every joint it targets is within 0.04 rad (or m) of its goal. A joint
servo closes 20% of the remaining gap 10 times a second.

## How I drew it

![The same pose three ways: the link frames from /xform_world, the URDF meshes attached to them, and the finished robot](drawing_steps.jpg)

My viewer is a separate client of my runtime, and it never computes kinematics. Every pose it
shows comes from `/xform_world`, which holds one 4×4 world matrix for each link. Drawing the
robot takes three steps:

1. **Link frames.** Each link's matrix is drawn as a small x/y/z axis triad, and lines along the
   `/tf` tree join the triads. This skeleton is exactly what my runtime computed.
2. **Attach the parts.** In the URDF, each link's `<visual>` names a mesh and gives an
   `<origin>` offset inside the link's frame. Each mesh is placed at the link's world matrix
   times that offset.
3. **The robot.** With every part in place, the result is Fetch at that pose.

![The runtime's nodes and topics from the choreography to /xform_world, one /xform_world entry with its column-major matrix layout, and the forward-kinematics formula](drawing_pipeline.svg)

**Where the matrices come from.** `robot_state_publisher` combines the URDF with the joint angles
q from `/joint_states` and publishes one transform per joint on `/tf`. Each transform is
T_origin · T_motion(q): the joint's fixed offset, then its motion (a rotation for a revolute
joint, a slide for a prismatic one). `robot_world_state_publisher` starts at the root link and
walks down the tree, multiplying as it goes:
X_world(child) = X_world(parent) · T_origin · T_motion(q).

**Reading a matrix.** Each `/xform_world` entry stores its 16 numbers in column-major order,
`m[4·col + row]`. The upper-left 3×3 block is the link's rotation, and `m[12..14]` is its
position.

---

Fetch robot description © Fetch Robotics, Inc., [CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/)
(from [fetch_ros](https://github.com/fetchrobotics/fetch_ros); meshes converted for the web).
