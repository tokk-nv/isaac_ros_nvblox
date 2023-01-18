# Tutorial for Setting Up Jetson on a Robot

This tutorial demosntrates how to setup a Jetson developer kit mounted on a robot with a Realsense camera with a separate PC for remote command and visualization.

> Note: This tutorial has been tested with the following combinations.
> | Jetson | RealSense | Robot | PC | Check status |
> |:--|:--|:--|:--|:--|
> | AGX Orin Developer Kit | D435I | Create 3 | x86 desktop with Volta GPU | ⚠️ |
> | Xavier NX Developer Kit | D435I | Create 3 | x86 laptop with Maxwell GPU | ⚠️✅ |

## [1] RealSense setup with Jetson on a Robot

### Machines Setup

```
                                       Internet
                                           ▲
                                           │
                                     ┌─────┴────┐
                           Wi-Fi     │ Wireless │   Wi-Fi/Ethernet
                           ┌───────► │  router  │ ◄──────┐
                           │         └──────────┘        │
                           │                             │
                           │                             │
                           ▼ 192.168.1.10                ▼ 192.168.1.11
 ┌───────────┐   USB   ┌────────┐                     ┌───────┐
 │ RealSense ├─────────┤ Jetson │                     │  PC   │
 └───────────┘         └────────┘                     └───────┘
                           ▲                         /       /
                 Ethernet  │                        ─────────
                 over USB  │
                           ▼
                      ┌──────────┐
                      │ Create 3 │
                      └──────────┘

```

### Initial setup 

#### 1. Initial Setup on PC

We will use this PC for any visualization, i.e. running `realsense-viewer` X11 app, RViz, etc.

For accepting X11 forwarding from any machine to display X11 window on this PC, issue this on your PC terminal.

```bash
xhost +
```

Then use this PC to SSH into Jetson, as `ssh -X {USER_ON_JETSON}@{IP_OF_JETSON} `.<br>
For example;

```bash
ssh -X jetson@192.168.1.10 
```

#### 2. Initial setup on Jetson

When you first SSH into Jetson from your PC, you should see error messages like these.

> ```
> /usr/bin/xauth:  /home/jetson/.Xauthority not writable, changes will be ignored
> /usr/bin/xauth:  /home/jetson/.Xauthority not writable, changes ignored
> X11 connection rejected because of wrong authentication.
> X11 connection rejected because of wrong authentication.
> ```

Delete the `.Xauthority` directory to let it be recreated with your user permission, then exit.

```bash
sudo rm -rf ~/.Xauthority
exit
```

Once exited, SSH back into Jetson from your PC.<br>
You should see a login message like this.

> ```
> /usr/bin/xauth:  file /home/jetson/.Xauthority does not exist
> ```

This is expected and indicates that the directory has been recreated in a way it is accessible by your user.

Test by running an X11 application on Jetson and see if you see the GUI on your PC.

![](../resources/x11forward-xeyes.png)

If this does not work, check your `DISPLAY` environment variable, and manually reset if neccesary.

```bash
echo $DISPLAY
export DISPLAY=192.168.1.11:10.0
```

### Steps for RealSense setup and test

> This is based on [Isaac ROS Realsense Setup](https://github.com/NVIDIA-ISAAC-ROS/.github/blob/main/profile/realsense-setup.md)

1. Clone the `librealsense` repo setup udev rules. Remove any connected RelaSense cameras when prompted:

   ```bash
    cd /tmp && \
    git clone https://github.com/IntelRealSense/librealsense && \
    cd librealsense && \
    ./scripts/setup_udev_rules.sh
    ```

2. Clone the isaac_ros_common and the `4.51.1` release of the realsense repository:

    ```bash
    mkdir -p ~/workspaces/isaac_ros-dev/src && cd ~/workspaces/isaac_ros-dev/src
    ```

    ```bash
    git clone https://github.com/NVIDIA-ISAAC-ROS/isaac_ros_common
    ```

    ```bash
    git clone https://github.com/IntelRealSense/realsense-ros.git -b 4.51.1
    ```

3. Plug in your RelaSense camera before launching the docker container in the next step.

4. Configure the container created by `isaac_ros_common/scripts/run_dev.sh` to include `librealsense`. Create the `.isaac_ros_common-config` file in the `isaac_ros_common/scripts` directory:

    ```bash
    cd ~/workspaces/isaac_ros-dev/src/isaac_ros_common/scripts && \
    touch .isaac_ros_common-config && \
    echo CONFIG_IMAGE_KEY=humble.nav2.realsense > .isaac_ros_common-config
    ```

5. Launch the Docker container using the `run_dev.sh` script:

    ```bash
    cd ~/workspaces/isaac_ros-dev/src/isaac_ros_common && \
      ./scripts/run_dev.sh
    ```

6. At this point, you can check that the RealSense camera is connected by running `realsense-viewer`:

   ```bash
   realsense-viewer
   ```

   You should see a new window pop up **on your PC**.

   If you turn on the "Stereo Module" in the GUI, you should see something like the following:

    ![](../resources/x11forward-realsense-viewer.png)

## [2] RealSense-based Reconstruction with Jetson on a Robot

> This is based on [Tutorial For RealSense-based Reconstruction](./tutorial-nvblox-vslam-realsense.md)

### Machines Setup

> Essentially same as above ([1])

```
                                       Internet
                                           ▲
                                           │
                                     ┌─────┴────┐
                           Wi-Fi     │ Wireless │   Wi-Fi/Ethernet
                           ┌───────► │  router  │ ◄──────┐
                           │         └──────────┘        │
                           │                             │
                           │                             │
                           ▼ 192.168.1.10                ▼ 192.168.1.11
 ┌───────────┐   USB   ┌────────┐                     ┌───────┐
 │ RealSense ├─────────┤ Jetson │                     │  PC   │
 └───────────┘         └────────┘                     └───────┘
                           ▲ 192.168.186.3           /       /
                 Ethernet  │                        ─────────
                 over USB  │
                           ▼ 192.168.186.2
                      ┌──────────┐
                      │ Create 3 │
                      └──────────┘
```

### Update RealSense firmware

> Based on [RealSense online documentaiton](https://dev.intelrealsense.com/docs/firmware-update-tool).

On your Jetson connected to the RealSense, run the container to have access to the librealsense installed.

```bash
cd ~/workspaces/isaac_ros-dev/src/isaac_ros_common && ./scripts/run_dev.sh
```

Once in the container, run the following to check.

```bash
rs-fw-update -l
```

You should get something like this.

> ```bash
> Connected devices:
> 1) Name: Intel RealSense D435I, serial number: 140122076447, update serial number: 137623063958, firmware version: 05.12.07.150, USB type: 3.2
> 
> ```

Download the specified version of the firmware and apply.<br>
**Be sure to replace the S/N with of your own RealSense.**

```bash
wget https://www.intelrealsense.com/download/19295/?_ga=2.244073541.1768311857.1674071420-176555355.1674071420 -O Signed_Image_UVC_5_13_0_50.zip
unzip Signed_Image_UVC_5_13_0_50.zip
sudo su
rs-fw-update -s 140122076447 -f D400_Series_FW_5_13_0_50/Signed_Image_UVC_5_13_0_50.bin
```

If it shows "`Device is in recovery mode, use -r to recover`", then issue the follwing.

```bash
rs-fw-update -r -f D400_Series_FW_5_13_0_50/Signed_Image_UVC_5_13_0_50.bin
```

If it shows something like following, exit and restart the container to run `rs-fw-update -l` to check the firmware version.

```bash
Firmware update done
Waiting for new device...
... timed out!
```
