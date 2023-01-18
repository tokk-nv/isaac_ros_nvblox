# Tutorial for Setting Up Jetson on a Robot

This tutorial demosntrates how to setup a Jetson developer kit mounted on a robot with a Realsense camera with a separate PC for remote command and visualization.

> Note: This tutorial has been tested with the following combinations.
> | Jetson | Robot | PC | Check status |
> |:--|:--|:--|:--|
> | AGX Orin Developer Kit | Create 3 | x86 desktop with Volta GPU | ⚠️ |
> | Xavier NX Developer Kit | Create 3 | x86 laptop with Maxwell GPU | ⚠️✅ |

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
                               │ 192.168.1.10                │ 192.168.1.11
                               ▼                             ▼
 ┌──────────┐              ┌────────┐                     ┌───────┐
 │ Create 3 │◄────────────►│ Jetson │                     │  PC   │
 └──────────┘   Ethernet   └────────┘                     └───────┘
                over USB                                 /       /
                                                        ─────────
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

