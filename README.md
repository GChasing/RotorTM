# RotorTM
An Aerial Transportation and Manipulation Simulator for Research and Education

## Overview
#### Description
RotorTM is an aerial transportation and manipulation simulator of MAVs with different payloads and passive connection mechanisms. It incorporates full system
dynamics as well as planning, and control algorithms for aerial transportation and manipulation. Furthermore, it includes a hybrid model accounting for the transient hybrid dynamics for aerial systems with cable suspended load to mimic real-world systems. It also provides flexible interfaces to planning and control software modules for the users. 

If you have any question regarding the repo or how to use the simulator please feel free to post questions in the Issues. 

![Screenshot](doc/3mav.png)

**Developer: Guanrui Li, Xinyang Liu<br />
Affiliation: [NYU ARPL](https://wp.nyu.edu/arpl/)<br />
Maintainer: Guanrui Li (lguanrui@nyu.edu), Xinyang Liu (thomas.xinyang.liu@nyu.edu)<br />**

## ROS Organization
The ROS Organization is shown in the table below. Here, # is used to denote MAV number.
|Name|Description|Publications|Subscriptions|Services|
|---|---|---|---|---|
|`/contrller_#`|Control MAV(s) to follow desired trajectory|/controller_#/dragonfly#/fm_cmd|/dragonfly#/odom
|||/controller_#/payload/cen_pl_cmd|/payload/des_traj|
||||/payload/odom|
|`/sim`|Simulate full system dynamics and publish all relevent Odometries for payload and MAV(s)|/dragonfly#/odom|/controller_#/dragonfly#/fm_cmd
|||/payload/odom|
|`/traj`|Compute and publish the payload desired trajectory|/payload/des_traj||/traj_generator/Circle
|||||/traj_generator/Line
|||||/traj_generator/Min_Derivative_Line

## Parameters Files
These files are used to set properties of the MAV(s).
|Name|Description|
|---|---|
|`UAV Params`|Basic UAV parameters like |
|`Payload Params`|Basic payload parameters like mass, moment of inertia etc.|
|`UAV Controller Params`|UAV controller parameters|
|`Payload Controller Params`|Payload controller parameters|
|`Attach Mechanism Params`|Attach mecahnism parameters|

## Dependencies and Installation
The RotorTM package is dependent on `Python 3.8` and `ROS Noetic`. Python packages including `numpy`, `scipy`, and `cvxopt` should be installed with `pip install`. After installation of necessary packages, clone the repo and `catkin_make` the ROS workspace. Source the `setup.bash` file inside the devel folder.

```
$ cd /path/to/your/workspace/
$ git clone --branch Python/ROS https://github.com/arplaboratory/RotorTM.git
$ catkin_make
$ source ~/path/to/your/workspace/devel/setup.bash
```

##  Running
### Initialize the simulator and visualizer
Start roscore and call the launch files to initialize the simulator and the rviz visualization. Each scenario has its own specific launch file. Here is an example of launching 3 MAVs transporting a suspended triangular payload.
```
roslaunch rotor_tm rotortm_cable_3.launch 
```
Below is table describing currently avaiable launch files.
|Launch file name|Chosen Files|
|---|---|
|`rotortm_cable_3.launch`|Initialize 3 MAVs transporting a suspended triangular payload|
|`rotortm_ptmass.launch`|Initialize 1 MAV transporting a suspended point-mass payload|
|`rotortm_rlink_3.launch`|Initialize 3 MAVs transporting a triangular payload with rigid link connect betwen MAVs and the payload|

### Start simulation
After calling the launch file, the MAV(s) is hovering at the predetermined initial location. To start simulation, desired trajectory needs to be generated by `/traj` node. `/traj` node has been setup with three ROS services corresponding to three possible trajectories. 
|Service|Description|
|---|---|
|`/traj_generator/Line`|circular trajectory generator
|`/traj_generator/Circle`|line trajectory generator|unlimited 
|`/traj_generator/Min_Derivative_Line`|minimum derivative trajectory generator

The location of service call definition is `RotorTM/rotor_tm_traj/srv`. Directly calling the services would start the simulation by activating the publication of `payload/des_traj`.

Here is an example of generating a circular trajectory with radius = 1.0 meter , period = 10 seconds, and duration = 10 seconds:
```
$ rosservice call /traj_generator/Circle 1.0 10.0 10.0
```
Another example of generating a line trajectory with from point [0,0,0] to [1,1,1]:
```
$ rosservice call /traj_generator/Line "path:
  - x: 1.0
    y: 1.0
    z: 1.0"
```
## Trajectory
### Line Trajectory
The line trajectory generator will generate a sequence of points connecting the given waypoint. The path simply connects all the points with straight lines. 

### Circular Trajectory	
The circular trajectory generator will generate a circular trajectory such that it will  
 1. ramp up to a constant speed
 2. run several cycles of circles ( depending on the period and duration)
 3. ramp down to zero speed. 

Here are the parameters to set for generating the circular trajectory:
|Name|Chosen Files|
|---|---|
|`Radius`|`The radius of the circular trajectory`|
|`Period`|`The period time of finishing one circle`|
|`Duration`|`The total time of the circular trajectory`|

### Minimum Derivative Trajectory
The minimum_derivative trajectory generator will generate a trajectory that goes through a given path: 
|Name|Chosen Files|
|---|---|
|`Optimization Option`|`Trajectory optimization options`|
|`Payload Path`|`The path that payload needs to go through`|

You can check in [here](https://github.com/arplaboratory/RotorTM/blob/main/doc/Simulator_Params.md) to see different kinds of combinatioins of parameters to simulate different situations. Then you can choose the type of trajectory generator to generate trajectory for the payload. We provide 
