# Ursim docker

This repo hosts Dockerfiles for the ursimulator. These dockerfiles are hosted as
images on Docker Hub. These images are taged with the official release of the ursimulator
that the dockerfile builds.

The container sets up a VNC server and exposes a port in which you can access the
robots GUI to control the robot. It also exposes all other available interfaces
in which you can control the robot remotely.

There are two different versions of the simulator an e-series robot and a CB3
robot. There is an image for both CB3 and e-series.

You can find pre-buit cb3 image on [ursim_cb3](https://hub.docker.com/r/universalrobots/ursim_cb3).

And e-series image on [ursim_e-series](https://hub.docker.com/r/universalrobots/ursim_e-series).

**Disclaimer** The purpose for the docker image is to facilitate and improve the access of the simulator.

The docker image is created by UR Labs. It is not yet apart of the official release.

# Why using URSim to provide a fake robot

Ever since ur_rtde is used, URFakeDriver is no longer real enough to fake the robot (i.e. cannot provide speedL/servoJ etc). Instead we can use URSim.

## How to use this image with [ur_rtde_node](https://github.com/HKCLR2021/ur_rtde_node)

#### Prerequisite: Installing Oracle HotSpot JRE
Follow https://ubuntu.com/tutorials/install-jre#3-installing-oracle-hotspot-jre

#### Make docker image
Follow https://github.com/szzhu-hkclr/ursim_docker

i.e. clone repo, `cd ursim_docker`, then

For old CB3 models (eg UR5)
```
docker build ursim/cb3 -t ursim_cb3 --build-arg VERSION=3.15.8.106339 --build-arg URSIM="https://s3-eu-west-1.amazonaws.com/ur-support-site/172182/URSim_Linux-3.15.8.106339.tar.gz"
```
OR for e-series (eg UR5e)
```
docker build ursim/e-series -t myursim_eseries --build-arg VERSION=5.12.4.1101661 --build-arg URSIM="https://s3-eu-west-1.amazonaws.com/ur-support-site/174207/URSim_Linux-5.12.4.1101661.tar.gz"
```
#### Spin up container
For old CB3 models (eg UR5)\
`docker run --rm -it -p 5900:5900 -p 29999:29999 -p 30001-30004:30001-30004 ursim_cb3`

OR for e-series (eg UR5e)\
`docker run --rm -it -p 5900:5900 -p 29999:29999 -p 30001-30004:30001-30004 ursim_eseries`

Now the robot is up but not in working state, you still need to click the init robot button via GUI at the next VNC step.

#### VNC to access UR5's GUI
Now you can use a VNC application to view the robots GUI, by connecting to localhost:5900.
And you will have a fully functional simulator which can be used inside applications
or testing pipelines.

install a VNC viewer eg:

`sudo apt-get install tigervnc-viewer`

connect to `0.0.0.0:5900`:

`vncviewer 0.0.0.0:5900`

#### Run rtde node and moveit server for controlling robot
`ros2 run ur_rtde_node URNodeTester --ros-args -p fake:=false -p simTimeScale:=1.0 -p ip:="0.0.0.0"`

`ros2 launch ur5_moveit_config ur5_moveit.launch.py launch_rviz:=true`


## Parameters

You can use the container with the different parameters, all parameters available
can be seen below.

| Parameter                | Description |
| ---                      | ---                                                                                                                            |
| `-e ROBOT_MODEL=UR5`     | Specify robot model to simulate. Valid options are UR3, UR5 and UR10. Defaults to UR5.                                         |
| `-e TZ=Asia/Hong_Kong`   | Specify a timezone to use. Defaults to Asia/Hong_Kong.                                                                      |
| `-p 5900`                | Allows VNC access to the robots interface.                                                                                     |
| `-p 502`                 | Allows access to Universal Robots Modbus port.                                                                                 |
| `-p 29999`               | Allows access to Universal Robots dashboard server interface port.                                                             |
| `-p 30001`               | Allows access to Universal Robots primary interface port.                                                                      |
| `-p 30002`               | Allows access to Universal Robots secondary interface port.                                                                    |
| `-p 30003`               | Allows access to Universal Robots real-time interface port.                                                                    |
| `-p 30004`               | Allows access to Universal Robots RTDE interface port.                                                                         |

## Programs

The `/ursim/programs` is used for storing robot programs created when using the
simulator. If one wishes to persist these files beyond the lifecycle of the container,
the `/ursim/programs` can be mounted to an external volume on the host.

For example, if one wants to save the programs in a `~/programs` folder we
can simply launch the container with an additional volume argument:

```bash
docker run --rm -it -p 5900:5900 -v "${HOME}/programs:/ursim/programs" ursim_cb3
```

## URCaps

It is possible to use this simulator together with URCaps. For using this simulator
with URCaps, you can follow the [e-series](#e-series-URCaps-installation) instructions
or the [CB3](#CB3-URCaps-installation) instruction.

### e-series URCaps installation

This example will show how to install the **externalcontrol-1.0.5.urcap** in the
e-series simulator, the same guide can be used to install any other URCap in the
simulator.

First you will have to create a volume for storing polyscope changes:

```bash
docker volume create ursim-gui-cache
```

We also need a volume for storing the URCaps build:

```bash
docker volume create urcap-build-cache
```

Now we need to copy the URCap to the `/ursim/programs` folder, this can be done
by mounting the folder to an external volume on the host as explained in the [programs
section](#programs) and then copy the URCap to that folder.

Now we can start the simulator.

```bash
docker run --rm -it --mount source=ursim-gui-cache,target=/ursim/GUI --mount source=urcap-build-cache,target=/ursim/.urcaps -p 5900:5900 -v "${HOME}/programs:/ursim/programs" ursim_cb3
```

Now use a VNC application to view the robots GUI, by connecting to localhost:5900.

On the welcome screen click on the hamburger menu in the top-right corner and select
*Settings* to enter the robot's setup.  There select *System* and then *URCaps*
to enter the URCaps installation screen.

 ![Welcome screen of an e-Series robot](doc/urcap_setup_images/es_01_welcome.png)

There, click the little plus sign at the bottom to open the file selector. There
you should see
all urcap files stored inside the robot's programs folder. Select and open
the **externalcontrol-1.0.5.urcap** file and click *open*. Your URCaps view should
now show the **External Control** in the list of active URCaps and a notification
to restart the robot. Do that
now.

 ![URCaps screen with installed urcaps](doc/urcap_setup_images/es_05_urcaps_installed.png)

When the simulator reeboots the container will stop, and you will have to rerun
the container, with the same command as above.

After the reboot you should be able to find the **External Control** URCaps inside
the *Installation*
section.
For this select *Program Robot* on the welcome screen, select the *Installation*
tab and select
**External Control** from the list.

 ![Installation screen of URCaps](doc/urcap_setup_images/es_07_installation_excontrol.png)

To use the new URCap, create a new program and insert the **External Control** program
node into the program tree

![Program view of external control](doc/urcap_setup_images/es_11_program_view_excontrol.png)

Now you have a fully functional URCap in your simulator.

### CB3 URCaps installation

This example will show how to install the **externalcontrol-1.0.5.urcap** in the
CB3 simulator, the same guide can be used to install any other URCap in the simulator.

First you will have to create a volume for storing polyscope changes:

```bash
docker volume create ursim-gui-cache
```

We also need a volume for storing the URCaps build:

```bash
docker volume create urcap-build-cache
```

Now we need to copy the URCap to the `/ursim/programs` folder, this can be done
by mounting the folder to an external volume on the host as explained in the [programs
section](#programs) and then copy the URCap to that folder.

Now we can start the simulator.

```bash
docker run --rm -it --mount source=ursim-gui-cache,target=/ursim/GUI --mount source=urcap-build-cache,target=/ursim/.urcaps -p 5900:5900 -v "${HOME}/programs:/ursim/programs" ursim_cb3
```

Now use a VNC application to view the robots GUI, by connecting to localhost:5900.

On the welcome screen select *Setup Robot* and then *URCaps* to enter the URCaps
installation
screen.

 ![Welcome screen of a CB3 robot](doc/urcap_setup_images/cb3_01_welcome.png)

There, click the little plus sign at the bottom to open the file selector. There
you should see
all urcap files stored inside the robot's programs folder. Select and open
the **externalcontrol-1.0.5.urcap** file and click *open*. Your URCaps view should
now show the **External Control** in the list of active URCaps and a notification
to restart the robot. Do that
now.

 ![URCaps screen with installed urcaps](doc/urcap_setup_images/cb3_05_urcaps_installed.png)

When the simulator reeboots the container will stop, and you will have to rerun
the container, with the same command as above.

After the reboot you should be able to find the **External Control** URCaps inside
the *Installation*
section.
For this select *Program Robot* on the welcome screen, select the *Installation*
tab and select
**External Control** from the list.

 ![Installation screen of URCaps](doc/urcap_setup_images/cb3_07_installation_excontrol.png)

To use the new URCap, create a new program and insert the **External Control** program
node into the program tree

![Program view of external control](doc/urcap_setup_images/cb3_11_program_view_excontrol.png)

Now you have a fully functional URCap in your simulator.

## URCaps CI pipelines

For using URCaps inside a CI pipeline a folder `/urcaps` has been created inside
the container. This folder can be mounted to an external folder on the host, storing
the URCaps. **Note It should be the `.jar` file and not the `.urcap` file that
is located inside the folder on the host**.

For example, if one has a folder called `~/urcaps` the container could simply be
launched with the following command:

```bash
docker run --rm -it -p 5900:5900 -v "${HOME}/urcaps:/urcaps" ursim_cb3
```

This will install the URCap when running the container and without having to restart
the simulator.
