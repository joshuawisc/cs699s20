# Final Report

## Contents
1. [Results](#results)
2. [Work](#work)
3. [Lessons](#lessons-learned)
4. [Self-Evaluation](#self-evaluation)

## Results

Most of my work involved the Kinova Movo, which is a bimanual mobile robot with two 7DOF Kinova Jaco arms.
 I worked on two main projects this semester: an improved mimicry control system for the Movo and a simulated
  environment for the Movo. I also worked on other smaller items throughout the semester.

### Mimicry Control - [files](https://github.com/joshuawisc/movo-control-2)

The mimicry control system that was previously built had the main functionalities of controlling the position and 
rotation of both the arms using two Vive controllers. This semester, I worked on making the system more usable and 
adding more features. 

#### Features:

The new system has the ability to work with just one Vive controller. By pressing a button, the controller alternates 
between controlling the right arm, controlling nothing, and controlling the left arm. The controller can also be used 
for full control of the Movo’s base, i.e, forward / backward, left / right, and clockwise / anti-clockwise rotational 
movements. The base is controlled using the circular touchpad on the Vive controller. Clicking on the left / right / up 
/ down directions on the touchpad moves the base in that direction. Drawing circular gestures on the touchpad without 
clicking rotates the base clockwise or anti-clockwise depending on how you draw them. The system starts with the arms 
and the end effector at a position that aligns well with the natural holding position of the controller, which allows 
for more intuitive control. The grippers can be open or closed using the triggers on the controllers. The system also 
has collision detection, so the arms will stop if there is a collision imminent. 

The system requires a Windows machine along with a Linux machine, which in this case is the Movo’s computer. There are 
two parts to the system, the Unity project which runs on Windows and the ROS nodes that run on Linux. The Unity project
 gets positional, rotational and button data from one or both controllers and sends that to the Linux machine via an 
 Ethernet connection into the Movo. On the Linux side, a ROS node retrieves the data, parses out the information, and 
 broadcasts it to the other nodes. Another node uses that data and broadcasts useful information like desired
  end-effector position / rotation and desired base movements. The desired end-effector information is received by 
  RelaxedIK, which is the inverse kinematics solver for both the arms of the Movo. The joint states calculated by 
  this IK node are then broadcasted out. The velocity controller node is the hub for all movement messages sent to 
  the Movo. It reads the joint states, desired base movements and Vive trigger positions from various other nodes 
  and sends the required messages to move the Movo. This node also does some basic collision detection and does not 
  send a command that might result in a collision.

There are also some smaller features included in the Movo mimicry control ROS package. One of them allows for control
 of the Movo using an Xbox controller. There are two nodes available for this, one to read and broadcast the input 
 from the Xbox controller and another to convert the Xbox controller inputs into movements for the Movo. There is
  also a set of nodes that adds latency into the Movo control system. There is a ROS launch file which starts these
   nodes that add a user-defined amount of delay.


#### Limitations:

The whole system requires a wired Ethernet connection from the Windows machine to the Movo, which reduces the 
mobility of the Movo. The method for switching arms could be improved. Currently, a single button controls which 
arm to move as well as whether or not the controller controls anything. So switching arms takes 2 button presses
 and pausing and unpausing control for the same arm takes 4 button presses. Separating these two functions would 
 improve usability. Another possible place for improvement is in the gripper control. Currently, the gripper can
  only be fully open or fully closed but it is possible to add granular positions for the gripper, as the Vive 
  trigger returns incremental values from 0 to 1. Finally, even though the gripper control should theoretically
   work with both grippers, only one gripper was ever connected so it wasn’t tested in practice.


### Movo Simulation - [files](https://github.com/joshuawisc/movo-sim-scene)

#### Features:

This project contains a scene that can be loaded into CoppeliaSim, a robot simulation program. The scene
contains a working model of the Movo, a table and a glass. While running, the simulation outputs joint states
  from the arms and reads information from various ROS topics to mimic an actual Movo. The simulation contains
   topics for moving the arms using angular velocities, controlling the pan / tilt joints of the head using positional
    data and moving the linear actuator of the Movo up and down. There are vision and depth cameras in the simulation 
    that output images onto ROS topics. There are also topics for gripper control. There is a primitive grasping system
     that allows you to grab the glass and move it around. 


#### Limitations:

There are many ROS topics on the real Movo that are not present in the simulation, for example base control topics 
and position control for the arm joints. Also, the grasping mechanism works programmatically by sensing whether an 
object is nearby and whether the gripper is closed, rather than simulating a physical grasp. This causes the gripper 
to pass through the object it is trying to grasp.

### Unity Models - [files](https://github.com/joshuawisc/UnityRobotModels)

I did some work with importing models into Unity. This repo contains various models I was able to add into Unity like 
the Movo, Panda and the Robotiq gripper. I also included a Unity package that includes the gripper along with some
 scripts for controlling it.

## Work

For the first half of the semester, I worked mainly on the mimicry control system. I edited the Unity code to send more 
data from the controllers to Linux. This allowed me to use more of the controller and add more functionality. I was able
 to use the touchpad to control the base and triggers to control the grippers. The Unity code initially outputted data 
 for only one controller but I modified it to send both. I then worked on getting a better starting position for the
  Movo’s arms. The arms were initially in a tucked position with the grippers facing upwards. This was a weird starting
   position as the kinematic solver had to move the joints awkwardly to keep the gripper in the upward position. 
   It also made controlling the arm harder as the controller had to be rotated 180 degrees downwards to pick up 
   something. The new position I configured had the gripper facing forward and the fingers parallel to the ground. 
   In this way, the grippers aligned with the circular ring on the Vive controller which felt more intuitive and also 
   made the movements more smooth. I had to read through the Movo's startup code to find the files that controlled the 
   boot-up sequence. I edited that code to allow the Movo to automatically move the arms to the better position on boot-up.

Since we were considering experimenting with different forms of bimanual control, we decided to have two ways of 
controlling the arms: with two Vive controllers simultaneously or with one controller that can switch which arm 
is being controlled. So I worked on that feature next. There was a button that was being used to control the 
clutch feature. Clicking this button stops the controller from moving the arm and allows the user to reposition 
the controller to a more comfortable position before regaining control. I decided to use this button to switch 
between controlling the arms while also pausing control between each switch to allow the user to re-centre the 
controller. So the button alternates between [right control, no control, left control, no control] and loops 
between these states.

I then worked on adding base control into the mimicry system. Since the touchpad was not being used for any 
other feature, I decided to use it for the base control. I used the up / down / left / right clicks for the 
corresponding movements on the Movo. Since I could track the touch position on the trackpad, I used that to 
control the base rotation. I decided to use circular gestures to control the rotation instead of just left / 
right touches as I didn’t want the Movo to move due to accidental touches. A downside to this mapping is 
that the rotational and positional movement can’t be controlled at the same time.

There are some things I worked on that I wasn’t able to complete. One of them is trying to fix the networking 
issues on the Movo. There is a WiFi access point on the Movo which should theoretically allow for remote control 
of the Movo, but this connection is not very stable and unpredictably drops after some time period. I tried many 
solutions to fix this including editing the networking device settings and changing the firewall settings on the 
Movo, but I wasn’t able to get anything to work. We bought a small external router to try and fix this issue but 
I was not able to get this configured before the lab closed. I also tried using the Rust version of RelaxedIK but 
was unable to get it properly working with the Movo. I was having some issues with how the starting locations were 
being calculated and was unable to fix it.

There were some other issues with the Movo that still haven’t been dealt with yet. One of them is that the Movo 
randomly stops outputting joint states from one or both the arms. This prevents my velocity controller from working 
correctly as it doesn’t know where the arms are. I tried restarting the nodes responsible and finding the cause of 
this issue but I was unsuccessful. I decided to record the information and post an issue on the Movo’s github repo 
but the lab closed before I could do this. Another issue is that the replacement coupling that we received from 
Kinova did not fit on the arm, so we were never able to test both grippers simultaneously.

After the lab closed, I started working on simulations. I initially worked in Unity using 
[Anthony / Jack's project](https://github.com/jackyangzzh/VR-Robot-Simulation). I worked on importing the Movo model 
into Unity and was able to do that successfully. I then configured the model in Unity so that the arms were controllable
 using ROS and relaxedIK joint angles. I also worked on getting the Robotiq gripper into Unity and getting the joints to 
 open and close correctly. Although the models worked in Unity, I decided to use CoppeliaSim (the newer version of V-REP) 
 to continue working on simulations. CoppeliaSim had a better physics system and dealt with gravity and collisions with 
 minimal setup. It had a simpler interface with ROS compared to Unity. It also came with many built in models of objects 
 as well as various cameras and sensors that were easy to place and interact with. So for the last few weeks, I worked on 
 getting the Movo working in a CoppeliaSim simulated scene. 

Getting the program installed was a bit difficult as many of the plugins required for using urdf models were not 
working correctly. After I got it working I was able to get the Movo model into the scene. I then worked on getting 
the model to do something during the simulation. The first thing I worked on was moving the linear actuator using ROS 
as it required a very simple message type. I then worked on adding more ROS topics to output arm joint states, 
control the arms, control the head and control the gripper. I named the topics the same as the actual Movo to allow 
all the pre-existing ROS packages to work with the simulation. I had to download and recompile one of the ROS plugins 
for CoppeliaSim as I required some custom Movo message types for ROS that were not available in the default version. 
I also added vision and depth sensors onto the Movo to simulate the Kinect camera. I added topics for outputting the 
images generated by these sensors. Towards the last week, I worked on getting a primitive grasping system working in 
simulation. I wasn’t able to get physically simulated grasping to work, so I created something that acted like a grasp. 
I used the distance of the object from the gripper and the open / closed position of the gripper to decide whether to 
grab an object. If an object met the criteria, it would just be added as a child element to the gripper. I also turned 
off the collisions for the gripper’s fingers to prevent it from colliding with the object. 

To test the simulation I worked on a bimanual control system for the Movo that uses an Xbox controller. I found a 
library that gets input from an Xbox controller into Python. Since this library was event based and stalled execution 
while waiting for the event, I had to create two ROS nodes. One node waits for events and broadcasts them onto ROS 
topics. Another node reads the input from the topics, converts them into Movo movements and publishes them on the 
required topics. This separation of nodes allows the system to be reused for other robots as the general Xbox inputs 
are easily accessible on ROS topics and custom nodes can be written for different robots that read this data. 
I created an initial mapping that I thought would feel intuitive. The controller is split into two halves with each 
half controlling the respective arm. I used the analog stick to control the position of the arm in the plane parallel 
to the ground. I used the shoulder and trigger buttons to control the height of the end-effector as the position of 
the buttons matched the up / down movements. The D-Pad and ABXY buttons were used to control the left / right / up / 
down rotations of the left and right arms respectively. I mapped the click of the analog sticks to the gripper control. 
This mapping felt pretty good to use but it lacked rotational control along one of the axes. I thought of some 
improvements to the system that I wasn’t able to fully implement. One of them was holding down the shoulder button 
to allow the analog stick to control a different axis. This would give two additional axes of control to each arm. 
Another was mapping the trigger to the gripper to allow for gradual control of the finger position.

Finally, to make the simulation perform more closely to the real system, I created a latency system. This includes a 
node that takes input from one topic, waits for a specified time and then published on a different topic. There is 
also a launch file which launches this node as well as the velocity controller node. The launch file remaps the 
velocity controller node such that it reads from the delayed topic. This launch file also allows the user to specify 
the desired time delay.


## Lessons Learned

I worked with many different programs and in many languages this semester, so I was able to learn a lot. Continuing 
to work in ROS allows me to strengthen my knowledge with the framework. I was able to create more complex nodes and 
launch files and work with more features like remapping.  I also did a lot of work in Unity for the mimicry control 
system and while importing various models. I learned more about VR development and how to use the SteamVR plugin in 
Unity to create controller mappings and get data from the controllers. I also improved my C# programming skills by 
writing various scripts in Unity. Since I had to import models into various programs, I learned how to read the URDF 
files and how to convert them into various types. I also had some practical learning with networking as I tried to 
fix the Movo’s issues. Coincidentally, I was taking a networking course at the same time, so a lot of stuff I was 
observing was being discussed in class as well. Working with CoppeliaSim, I had to learn a bit of Lua which I had 
never used before. I also had to learn more about building and compiling various programs while trying to get 
CoppelliaSim working.

Since I did most of my work in the lab, I got to see what other people were working on. Ziyad often asked me to 
test out the latest changes in his program and it was interesting to see how the program was being designed and 
modified based on various study considerations. I was also able to work more collaboratively as some of my projects 
had overlap with others, like the gripper models in Unity that Jack also needed or the collision information from 
the Movo for Sayem’s program. I also focused more on the readability and succinctness of my code. Coming back after 
the break, I realized my code was a bit difficult to understand and contained a lot of unnecessary code, so 
I trimmed what was not required and made sure to review the code I was writing more often.


## Self-Evaluation

I am largely satisfied with the work I have done throughout the semester. I feel like I was a bit slow at the 
start but got better with more defined goals later on. I am pretty disappointed that we weren’t able to use 
the mimicry control system for something valuable due to the closure of the lab. The Movo was easier to work 
with this semester as compared to previous ones, which made things better. I think the Movo simulation in 
CoppeliaSim was a good project to work on after the closure. It will be useful to test stuff in simulation 
before trying it on the real robot and I think it is something that is easily usable by others to test their 
code on. I didn’t put as many hours in as I would’ve liked to as I was busier than I expected this semester. 
If I could change something, I would try to create a fixed working schedule early on that I could stick to as 
my schedule was very disorganized and kept changing, which reduced my productivity.
