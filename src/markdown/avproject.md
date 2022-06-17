# Description

The goal of this project was to determine to what extent yard tractors (or
'port haulers' colloquially) can be converted into autonomous vehicles
in-house. These vehicles haul container carrying trailers from quayside to
storage yard in the ports. The company was already developing a
non-autonomous port hauler, but the team anticipated that autonomy would
eventually become an entry to market requirement and I initiated this
project. By developing an autonomy kit, retro-fitting would also be made
possible on older vehicles. 

# Software Development Approach

Early on a decision had to be made whether to develop from scratch, or
leverage open source. Given the available resources and the availability of
the open source [Robot Operating System (ROS) framework](https://ros.org/),
I decided to pursue this route.

This decision yielded rapid progress, greater number of features and higher
quality software. Exposure to an established codebase and community yielded
secondary benefits such as enhanced learning, adoption of best practices,
broader applicability and code maintainability. 

# Hardware Architecture and Approach

As technical lead of the project, I chose to demonstrate the feasibility of
in-house development using a scale prototype. In deciding on the system
architecture and design philosophy, to keep costs low and development
velocity high, standard off-the-shelf components such as remote controlled
(RC) vehicles and hobby-grade microcontrollers were used. The image below
shows a first iteration of the prototype vehicle.

![Early Prototype AV RC car](media/avproject/proto1.jpg)

Thanks to hardware modularity, this prototype was later easily refactored
to more accurately mimic the real vehicle as shown below. 

![Early Prototype AV RC car](media/avproject/proto2_open_front.jpg)
![Early Prototype AV RC car](media/avproject/proto2_open_rear.jpg)

![Early Prototype AV RC car](media/avproject/proto2_lowangle.jpg) ![Early
Prototype AV RC car](media/avproject/proto2_highangle.jpg) 

# Simulation

One strong advantage of `ROS` is its integration with
[`gazebo`](https://gazebosim.org/) for simulation. By the time that the
hardware was ready, most of the mapping, planning and control code had
already been set up and tested in simulation.  `Gazebo` simulated the
vehicle odometry, laser scanner, IMU and camera that were on the vehicle,
in some cases, long before the physical hardware was available. Creating
the simulations required developing `.URDF` files describing the robot
hardware. I parameterized this as much as possible using `.xacro` files, in
anticipation for future refactoring. 

Interfacing `ROS` components were configured in the simulation building
phase using `roslaunch` configurations which remained handy throughout the
project. A great deal of the project revolved around parametrising and
setting up the `roslaunch` files to start the various components that we
were relying on, I will refer to this as 'configuration' throughout.

# Integrations

The development process included design, fabrication and software
development. The hardware and software had to form a coherent whole which
required configuration and development of standard and bespoke components.
The following sections discuss some aspects that were successfully
configured in simulation and later transferred to real hardware, making for
impressive feature additions.

### Sensors and Sensor fusion

Since the robot had multiple data streams that were useful for navigation,
a high quality position estimate could be obtained by fusing these streams
using the
[`robot_localization`](http://docs.ros.org/noetic/api/robot_localization/html/index.html)
package, which implements a [Kalman
filer](https://en.wikipedia.org/wiki/Kalman_filter).   

### Simultaneous Localization and Mapping (SLAM)

The `ROS` navigation stack uses [OpenSlam](https://openslam-org.github.io/)
as its default implementation.  Both real and simulated robots were capable
of online mapping, which is surprising considering the hobby grade main
controller used.

### Path planning

The navigation stack provides a default global planner, which was good
enough for our needs. This planner doesn't take into account the vehicle
kinematics, and is performed against a static costmap. The default local
planner however, was replaced with the Timed Elastic Band local planner
([`teb_local_planner`](http://wiki.ros.org/teb_local_planner)), which takes
the kinematics of our [Ackermann
steering](https://en.wikipedia.org/wiki/Ackermann_steering_geometry)
vehicle into account. 

Shown below is a screen recording for assigning a target destination of the
simulated robot. The target pose is shown by the purple arrow at the start
of the video. The global planner's path is shown in green and the local
planner's path in red. The output of a simulated camera as well as the
simulator's GUI with a bird's eye view are shown in separate windows. The
red/blue shaded area around the vehicle is the costmap taken into account
by the local planner which is capable of handling dynamic obstacles. 

<video width="320" height="240" alt="Navigation Simulation Demo" controls>
<source src="media/avproject/navSim.mp4" type="video/mp4"> </video>

Unfortunately, the requirements of `teb_local_planner` combined with online
localization and the high overhead of the `ros_serial` package meant that
the hardware was not powerful enough to support navigation on the physical
vehicle. This is due, in part to `ros_serial` only being available with a
`Python` implementation.

### Control 

The main controller ran `linux` to support `ROS` and was not very powerful.
This made it inadequate for the real-time nature of controlling motor
speed.  Instead a dedicated controller was used which ran a PID loop and
handled encoder pulses for speed feedback. On the main controller, the
bespoke interfaces for `ros_control` were developed to pass speed and
steering angle data back for integration into pose estimates. For hardware
integration, this required tapping in to the servo motor control pot.
Getting odometry data from the standard RC motor encoder required splicing
its signal cable and adding interrupt code to the low level controller on
the vehicle. 

## Visualization and remote control

Another convenient `ROS` feature is the availability of various input
device capabilities. A joystick, keyboards and software input devices were
configured and used. `Rviz` is yet another great advantage of the
framework, allowing simultaneous visualization and interaction with real or
simulated robots. The configuration of these tools required minimal
modifications for use with the physical vehicle and makes development as
immersive as playing video games!

The video below shows a mapping session using a joystick as input device
from a remote PC. The sensor data streams are converted to a map of the
environment computed on the embedded controller and displayed in `Rviz` on
the remote machine. The odometry calculations are also visualized as a red
path trailing the vehicle.

 <video width="320" height="240" alt="Navigation Simulation Demo" controls>
<source src="media/avproject/mapping.mp4" type="video/mp4"> </video>

# Conclusion

Unfortunately, the project budget was deferred before completion due to
company wide difficulties resulting in cost cutting. I left the company
soon after. The project revealed that internal development would be
possible given dedicated resources. This is largely made possible due to
open source and the permissively licensed `ROS`. Taking into account that
this was my first robot, I am proud of how far we had come and the state
that I left the project in.
