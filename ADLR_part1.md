

**Autonomous Deep Learning Robot (TurtleBot) Bringup and Learning Part 1**

This document is meant to capture issues and steps taken to get my autonomous deep learning robot up and running with ROS and other software stacks included. This part 1 only covers initial setup and I plan to document other work in subsequent posts. I purchased this robot (based on TurtleBot) from the company Autonomous. A link to the company website and the robot specifications are here: https://www.autonomous.ai/deep-learning-robot

The robot comes with a 3D Depth camera from Asus, Jetson TK1 development board and is pre-loaded with Linux Ubuntu 14.04 and deep learning software including Tensorflow, ROS, Torch, Theano,Caffe,CUDA, and cuDNN. It uses a Kobuki base and runs the same software as a standard TurtleBot.

The following link was extremely useful for getting started and I followed much of it during my bring up and testing:
http://www.artificialhumancompanions.com/autonomous-deep-learning-robot-the-missing-instructions/

This document is meant to capture my specific steps and issues and might be helpful to others as a result. Much of it is simply following the TurtleBot tutorials and the link above and so serves to document what I did rather than introduce a lot of new material. I will try to document specific issues that I felt were not well explained in the tutorials. I also used the ebook "ROS Robotics By Example", by Carol Fairchild.

As a side note, I can not really recommend purchasing this particular robot as the support is virtually non-existent. I believe this product was provided as an afterthought with no intention of providing much technical support on it. That said, once I got a working board from them with the correct password (three weeks from first shipment), everything works so far. I would not have wanted to try to install all the software that this system comes preloaded with. If Autonomous offered a way to reflash the device using their method or provided documentation on how to do that, then I would recommend them. I could not get them to provide that. So if I have a severe crash, I am stuck with reflashing everything from scratch and re-installing all the deep learning software piece by piece.

#### My System

* A dedicated remote PC running native linux (Ubuntu 14.04) with a Titan X Pascal GPU and ROS indigo installed.
* The Autonomous Deep Learning Robot (ADLR) referred to as TurtleBot or ADLR or BOT going forward
* I added an SSD drive to the ADLR and created a swap space. I will explain how I did that later.

#### Test in stand alone mode and set up wireless on ADLR
The Jetson TK1 has a monitor connector so that you can boot up to linux and see a desktop display. You can connect a keyboard and mouse via USB. The initial password was changed  from 'ubuntu' to 'autonomous' in later deliveries of the unit. This caused me much pain as it took multiple weeks to get the company to tell me what the correct password was. I ended up re-flashing the unit myself (watch for another doc on this TBD) before they finally replaced it and gave me the correct password. You should change the password first thing. From the desktop display GUI you can then set up the wireless connection to your home network.

#### Setting up remote login and IP address
One of the first things you need to do is to achieve remote login into the ADLR. After you have successfully connected the ADLR to your wireless, you can then work on setting up a remote connection. I used my router to locate the IP address of the ADLR. I was then able to use `ssh ubuntu@IPaddress` to remote login from my host machine as a test on the connection. I then configured my router (netgear dual band R6400) to use DHCP and then reserved IP addresses for the ADLR and my host machine. This is best of both worlds as it uses DHCP for everything else so others can easily join when visiting and I can have a static IP for the host machine and the ADLR which is desireable for ROS operation and makes remote login much simpler. Modify your `/etc/hostname` file using `sudo vi hostname` and change to a desired name. Then edit your `/etc/hosts` file also using `sudo` and add the reserved IP address and new name for the ADLR. Also add the IP address and name of the host machine in this same file. On the host machine, also edit the `/etc/hosts` file adding both IP adresses and names. Example below for the host machine, do same thing on the ADLR.
```
hostmachine@hostPCname:/etc$ more hosts
127.0.0.1	localhost
192.xxx.1.xx	hostPCname
192.xxx.1.xx  robotName
```
Once you have made these changes and reboot, you should be able to ssh in from the host machine to the ADLR using `ssh ubuntu@robotName`. Here `ubuntu` was the default username which I did not change. If you want to change username also see: https://askubuntu.com/questions/659454/how-to-safely-change-username-and-hostname

[//]: # (Image References)


---
#### Installing ROS on host machine

Since the ADLR has ROS indigo, we want to match it on the host machine. I basically followed the install for Ubuntu here:
http://wiki.ros.org/indigo/Installation/Ubuntu

In my case, I installed this and got everything working and then I later installed python 3 with miniconda for another project. Minconda install will cause the default python version to be whatever the miniconda install used. In my case that was Python 3.4. This will break your previous ROS setup which needs python 2.7. The following lines get added to your `.bashrc` file when miniconda does it's install.
```
# added by Miniconda3 4.2.12 installer
#export PATH="/home/ai/miniconda3/bin:$PATH"
#export PATH="/usr/local/cuda-8.0/bin:$PATH"
#export LD_LIBRARY_PATH="/usr/local/cuda-8.0/lib64:$LD_LIBRARY_PATH"

# change above so ROS will work with python 2.7
export PATH="$PATH:/home/ai/miniconda3/bin"
export PATH="$PATH:/usr/local/cuda-8.0/bin"
export LD_LIBRARY_PATH="$LD_LIBRARY_PATH:/usr/local/cuda-8.0/lib64"
```

Note that they pre-pended the path instead of post pending it. That means the system will see the miniconda version of Python first. To fix this simply post pend the path instead. This fixed my issues and the miniconda install still worked fine as I only use that in a conda enviroment anyway.

Also add the following lines in `.bashrc` on the host machine:
```
export ROS_MASTER_URI=http://rbotName:11311
export ROS_HOSTNAME=hostPCname
```
Similarly on the ADLR in `.bashrc` :
```
export ROS_MASTER_URI=http://rbotName:11311
export ROS_HOSTNAME=rbotName
```


#### Add SSD drive
I utilized the following source (richards technotes) and am just copying it for posterity.

https://richardstechnotes.wordpress.com/2014/07/06/nvidia-jetson-tk1-adding-a-hard-disk/

Using `gparted`, need to install first on the ADLR. You can remote login for this.

`sudo apt-get install gparted`

Then run it:

`sudo gparted`

"By default, gparted will select the onboard flash so you need to select /dev/sda in the drop-down in the upper right of the gparted window. Delete any existing partitions, add a new ext4 partition, apply the changes and then exit gparted once that’s done."

Create a directory for the mount point:
`sudo mkdir  /JetsonSSD`

Then add this line to /etc/fstab

`/dev/sda1 /home/ubuntu/JetsonSSD   ext4  defaults  0  0`

Then
```
sudo mount –a
sudo chown ubuntu:ubuntu /home/ubuntu/JetsonSSD
```
Check with

`mount –l`

#### Add a  Swap space

I followed this:
http://www.jetsonhacks.com/2014/10/04/creating-swapfile-ubuntu-nvidia-jetson-tk1/

Research on web at first seemed to say that you should create a swap partition first instead of just a swap file, but others refuted the performance issue. Much easier to use fallocate as in above url. Went with that.
