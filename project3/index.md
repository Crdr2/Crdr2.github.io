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

Drag to orbit and scroll to zoom. **frames** shows each link's coordinate frame and the `/tf`
tree that connects them. **trail** traces the gripper's path.

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

A Python client subscribes to `/xform_world` and `/fsm/status` and records them. A three.js page
then places each link's mesh at its 4×4 world matrix (`Matrix4.fromArray`, column-major), using
the visual origin from the URDF.

---

Fetch robot description © Fetch Robotics, Inc., [CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/)
(from [fetch_ros](https://github.com/fetchrobotics/fetch_ros)). The meshes were converted to a
compressed glTF file (`fetch.glb`) for the web; that adaptation is shared under the same license.
