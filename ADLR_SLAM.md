

### Autonomous Deep Learning Robot (TurtleBot) SLAM with Interactive markers

This document is one in a series meant to capture issues and steps taken to get my autonomous deep learning robot up and running with ROS and other software stacks. This entry covers SLAM (Simultaneous Localization and Mapping) with interactive markers. I plan to document other work in subsequent posts. Please read ADLR_part1.md for bringup and more background.

This document is meant to capture my specific steps and issues and might be helpful to others as a result. Much of it is simply following the TurtleBot tutorials and so serves to document what I did rather than introduce a lot of new material. I will try to document specific issues that I felt were not well explained in the tutorials.

#### Moving the robot with interactive markers
The TurtleBot tutorials cover how to move the robot using interactive markers here:
http://wiki.ros.org/turtlebot_interactive_markers/Tutorials/indigo/UsingTurtlebotInteractiveMarkers

This worked for me and is a much better way to move the robot around then trying to use the keyboard teleop method. You can follow the tutorial or just try the following:

###### On TurtleBot:

`roslaunch turtlebot_bringup minimal.launch`

###### On remote machine:

Terminal 1

`roslaunch turtlebot_interactive_markers interactive_markers.launch --screen`

Terminal 2

`roslaunch turtlebot_rviz_launchers view_robot.launch`

On the rviz gui, make sure interactive markers is selected and then click on the "Interact" tool button at top left of rviz screen. You can then use your mouse to move the robot around very efficiently.

#### SLAM Map Building with Interactive Markers
I followed the tutorial here:
http://wiki.ros.org/turtlebot_navigation/Tutorials/Build%20a%20map%20with%20SLAM

I ran into a problem when I tried to use interactive markers to move the robot around during SLAM. I will repeat the steps I took from the tutorial below and then explain how I fixed the issue:

###### On TurtleBot:

Terminal 1

`roslaunch turtlebot_bringup minimal.launch`

Terminal 2

`roslaunch turtlebot_navigation gmapping_demo.launch`

###### On remote machine

`roslaunch turtlebot_rviz_launchers view_navigation.launch`

At this point the tutorial tells you to use keyboard teleop to move the robot around and also states that you could use a joystick or interactive markers. When rviz comes up, the interactive markers are not in the rviz display panel. I figured I could simply add them using the "add" button at the bottom of the display panel. This seemed to work and added Interactive markers to the display panel with status OK. I then ran the following on another terminal on my remote machine to get an interactive marker server running:

`roslaunch turtlebot_interactive_markers interactive_markers.launch --screen`

I thought all would be good at this point, but when I clicked on the "interact" button, nothing happened. Normally a ring appears  around the robot with arrows so you can use your mouse to move it around. I figured it must be something in the `view_navigation.launch` that was different from the `view_robot.launch`.

To inspect this I used `roscd turtlebot_interactive_markers` which puts you in the directory: `/opt/ros/indigo/share/turtlebot_rviz_launchers` then `ls` to see contents

```
/opt/ros/indigo/share/turtlebot_rviz_launchers$ ls
cmake  launch  package.xml  rviz
```
Then look at the launch directory
```
/opt/ros/indigo/share/turtlebot_rviz_launchers/launch$ ls
view_blind_nav.launch  view_navigation_app.launch  view_robot.launch
view_model.launch      view_navigation.launch      view_teleop_navigation.launch
```
Contents of `view_navigation.launch`:
```
<!--
  Used for visualising the turtlebot while building a map or navigating with
   the ros navistack.
 -->
<launch>
  <node name="rviz" pkg="rviz" type="rviz"
  args="-d $(find turtlebot_rviz_launchers)/rviz/navigation.rviz"/>
</launch>
```
The contents of `view_robot.launch` were identical except that it uses `robot.rviz` instead of `navigation.rviz`

Comparing the two `*.rviz` files, I found that the `robot.rviz` had lines added for the  interactive markers that were not present in the `navigation.rviz` file. The lines are as follows:

```
- Class: rviz/InteractiveMarkers
  Enabled: false
  Name: InteractiveMarkers
  Show Axes: false
  Show Descriptions: true
  Show Visual Aids: false
  Update Topic: /turtlebot_marker_server/update
  Value: false
  ```
I added these lines to the `navigation.rviz` file and then did everything I described previously. Note you need `sudo` to edit this file. I did not have to use the "add" button on the display this time as rviz came up with that now. Clicking on "interact" button now worked as expected and I could move the robot around using the interactive markers. I included my modified `navigation.rviz` file for reference on my GitHub site for this project. I also posted this to the ROS answers forum here:
http://answers.ros.org/question/260899/interactive-markers-does-not-work-with-gmapping_demo/?answer=260908#post-id-260908


#### Building the map
I started navigating the robot around my home using the interactive markers and the map began to build. At some point not far into the process, the scan matching failed, see below output from `gmapping_demo.launch` from TurtleBot terminal:
```
Average Scan Matching Score=583.134
neff= 62.9071
Registering Scans:Done
update frame 3969
update ld=0.260425 ad=3.03173
Laser Pose= -2.0733 0.399629 -0.203371
m_count 18
Scan Matching Failed, using odometry. Likelihood=-2824.34
lp:-1.89031 0.214325 2.82835
op:-2.0733 0.399629 -0.203371
Scan Matching Failed, using odometry. Likelihood=-3917.25
lp:-1.89031 0.214325 2.82835
op:-2.0733 0.399629 -0.203371
```
I believe that once this happens, you no longer are going to get a good map as the "laser scan" is not functioning and the odometer is just not accurate enough. The "laser scan" is in quotes because the Asus 3D camera does not have a laser but simulates it using infra red. Once the above started failing, the map no longer produced any dark (black) edges where walls were and only showed point clouds.

I tried this again and it seems that the Scan Matching Failed will just come and go no matter what. If you are attempting more than one room, as I am, you need to drive the robot around many many times forward and backward through same areas. My battery went low before I could get a good map.

Also, I found that the interactive marker control is not stable. It will just stop working randomly and print the following to the screen:

```
[ INFO] [1493834375.397386596]: Bond broken, exiting
[interactive_marker_velocity_smoother-2] process has finished cleanly
log file: /home/.ros/log/e6ecb116-3028-11e7-8a61-7c5cf8506f59/
       interactive_marker_velocity_smoother-2*.log
```
Sometimes I could Ctrl-C and then start the marker server again and it would  work for awhile longer before same problem. I ended up using the keyboard teleop control as it does not fail randomly. Will post  more on this topic once I get a decent map. Meanwhile, I ordered a larger battery for the Kobuki base. 
