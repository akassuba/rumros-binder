
# RuMROS-Binder

[![Binder](https://binder.intel4coro.de/badge_logo.svg)](https://binder.intel4coro.de/v2/gh/akassuba/rumros-binder/HEAD)

## Introduction

This repository creates a Virtual Research Lab (VRL) for [RuMROS](https://github.com/sebastiangoetz/RuMROS) as part of the [EASE Virtual Research Building (VRB)](https://vrb.ease-crc.org/) project. It runs the launch script as a Jupyter Notebook directly in the browser, with a docker backend in the cloud, allowing RuMROS to be tested without installation. To launch it, simply click the binder launch button above or click this [link](https://binder.intel4coro.de/v2/gh/akassuba/rumros-binder/HEAD).

## RuMROS

RuMROS is a runtimemodel driven robotic application with an adaptive GUI. It supports control and simulation (Gazebo) of individual robots, as well as creating applications for multi-robot systems (MRS) and simulating swarm behavior via [ARGoS 3](https://github.com/ilpincy/argos3). 

RuMROS consists of 3 main modules:

- ROS-backend node(s)
- A meta-model with a generated runtime backend
- A web frontend for the runtime model

The application is partially generated from the runtime meta-model. This includes the web frontend, which is fully generic, as well as runtime model backend code and most interface parameters. The Webapp is written in Flask and uses Socket.IO for connecting to the frontend. The runtime meta-model is specified in [JastAdd](https://jastadd.cs.lth.se/web/), enabling automatic backend code generation as well as easy extension by the user.

RuMROS comes with support for two main ways of programming robots: Fully runtime-model-based control, as well as swarm programming. In the former case, Gazebo is used for simulation and the runtime model application controls the robots directly via ROS topics. This allows the user to fully define robot's behavior by computing movement commands in the runtime model. A generic ROS node is provided for each robot type (currently supported are turtlebots and drones, rovers are WIP), that simply steers the robots and sends sensor information to the runtime model.

For MRS applications or swarm programming, RuMROS operates differently, since computing commands in real-time for each robot is not feasible in larger systems. In contrast to the aforementioned approach, each individual robot has to have a certain degree of autonomy, or 'skillset'. This is expressed in a ROS node that runs on every swarm member. It performs sensing and actuating in order to avoid overloading the runtime model controller, but can report certain data back to the controller. The causal connection from the model to simulated robots is realized via control topics, enabling the runtime model controller to switch the behavior of robots. To make addressing in larger systems easier, this is done in groups, which can be specified in the model. The node comes with pre-defined movement behavior blocks, enabling driving in a direction, dispersion, moving to a location with and without obstacle avoidance, flocking to a location, and more. This is the previously mentioned 'skillset', which each robot has. The runtime model monitors robot states and orchestrates behavior switching on the ROS layer. This allows for global control over actions of the MRS while keeping localized behavior, making RuMROS a hybrid approach to MRS and swarm programming.

To provide better guidance on *how* behaviors should be composed in the model, a modeling mechanism similar to hierarchical finite state machines is used. Nodes execute a behavior on individual robots or groups, which is switched using conditions, that can be timed or conditional. This way, complex behavior can be built and *adaptation* of behavior according to data available in the runtime model is made possible. A group can adjust their behavior based on the presence or current task of another group, or based on environmental states, for instance.

For more information on how RuMROS works, check out the repository with the [full source code](https://github.com/sebastiangoetz/RuMROS).

