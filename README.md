# Research Assignment
### "Integrating expected quality attribute predictions and the TIAGo robot with MROS1"

This fork was made for a research assignment supervised by Carlos Hernandez
Corbato, where work done by Jasper Wijkhuizen on QA predictions was integrated
into the latest version of [MROS1](https://doi.org/10.1109/RoSE52553.2021.00011).
Modifications were also made to the current experimental setup from the original 
repository to work with a new robot, TIAGo. The original README is included below
for completeness. 

## Installation
**Pre-requisites:** It is assumed that you already have an installation of Ubuntu 18.04 
and ROS Melodic. The Python libraries 'numpy' and 'sklearn' are also required.

First, we need to install the pre-requisities for MROS, and get the MROS packages:

- Install java jre

```console
sudo apt-get install openjdk-11-jre
```

- Create a workspace (with any preferred name) and a “src” directory within it:
```console
mkdir -p ~/metacontrol_ws/src
cd metacontrol_ws
```

- Setup the dependencies (note you need wstool for this)
```console
cd ~/metacontrol_ws
wstool init src https://raw.githubusercontent.com/Mramirez25/metacontrol_experiments/models21_experiments/metacontrol_experiments.rosinstall
source /opt/ros/melodic/setup.bash
rosdep install --from-paths ./src -y -i -r --skip-keys="abb_rws_interface"
```

Now we need to get the TIAGo packages. This can be done by following the instructions in the 
[AIRLab repository](https://gitlab.tudelft.nl/airlab-delft/ng-staging/retail_store_simulation).
If you don't have access to this repository, you should also be able to get the needed TIAGo packages 
by following the instructions [here](http://wiki.ros.org/Robots/TIAGo\%2B\%2B/Tutorials/Installation/InstallUbuntuAndROS). 
The steps from the AIRLab repository are also included below:


- Clone the repository and clone the dependencies using vcstool:
```console
cd ~/metacontrol_ws/src
git clone git@gitlab.tudelft.nl:airlab-delft/ng-staging/retail_store_simulation.git
vcs import --input retail_store_simulation/retail_store_simulation.rosinstall .
cd ..
```

- Use rosdep to get all other dependencies:
```console
sudo rosdep init
rosdep update
rosdep install --from-paths src --ignore-src --rosdistro melodic \
--skip-keys="opencv2 opencv2-nonfree pal_laser_filters speed_limit_node \
sensor_to_cloud hokuyo_node libdw-dev python-graphitesend-pip \ 
python-statsd pal_filters pal_vo_server pal_usb_utils pal_pcl \
pal_pcl_points_throttle_and_filter pal_karto pal_local_joint_control \
camera_calibration_files pal_startup_msgs pal-orbbec-openni2 \
dummy_actuators_manager pal_local_planner gravity_compensation_controller \
current_limit_controller dynamic_footprint dynamixel_cpp tf_lookup \ 
slam_toolbox joint_impedance_trajectory_controller \ 
cartesian_impedance_controller omni_base_description omni_drive_controller"
```

Now we have all the packages we need, and can build our workspace:
```console
catkin build
```

Finally, source your workspace and then run the run_single_sim_desktop_tiago.sh bash script:
```console
source devel/setup.bash
roscd metacontrol_experiments
./run_single_sim_desktop_tiago.sh
```

What you will then see is a simulation where TIAGo has the goal of navigating to a point. Apart from a map with 
the known obstacles (visible in RViz), some random unknown obstacles are created (these can be seen in Gazebo as 
the blue pillars). At some point TIAGo will navigate very close to two of these new obstacles, causing the safety 
to go above the given threshold, and triggering a reconfiguration, which can be seen in the terminal showing the 
reasoner output. 

A video showing this simulation being run can be found [here](https://youtu.be/rPMegtRgl_w). The reasoner terminal 
is also shown in this video, where you can see when the reconfiguration occurs. Also note that data about each 
run is recorded in a CSV file in the `metacontrol_experiments/data` directory.

&nbsp;

---


# Original metacontrol_experiments README

This package runs and controls batch simulations of the [metacontrol_sim](https://github.com/rosin-project/metacontrol_sim) package, it can create contingencies that should result in reconfigurations of the navigation stack.

## Installation
### Install java jre

The experiments use the [`mros_reasoner`](https://github.com/tud-cor/mc_mros_reasoner) for the reconfiguration, and to run this a java jre is required.

You can use this command To install the Java Vitual Machine.

```console
sudo apt-get install openjdk-11-jre
```

### Create metacontrol_ws

- We recommend you to create a workspace only for `metacontrol_ws`, for example:

```console
  mkdir -p ~/metacontrol_ws/src
  cd metacontrol_ws
```

### Setup all dependencies using wstool

**Note** This step will include all the necessary packages, including those listed in the [metacontrol_sim](https://github.com/rosin-project/metacontrol_sim) package. Please make sure you're not duplicating them.

In the following instructions, we assume that the ROS workspace is called `metacontrol_ws`, but you can give it any name you want.

```console
cd ~/metacontrol_ws
wstool init src https://raw.githubusercontent.com/Mramirez25/metacontrol_experiments/models21_experiments/metacontrol_experiments.rosinstall
rosdep install --from-paths ./src -y -i -r --skip-keys="abb_rws_interface"
```

Check that all dependencies are installed correctly, please make an issue if you find any errors.

## Build the code

The code can be built using the standard `catkin build` process.

```console
cd ~/metacontrol_ws
source /opt/ros/melodic/setup.bash
catkin build
```

## Run the scripts

There are several scripts, to run a single simulation or to run simulations on a batch.

- `run_single_sim.sh` : Main script, runs a full simulation on a single terminal, the parameter are listed below.
- `run_single_sim_desktop.sh` : Runs a full simulation, but launches three different terminals. Accepts the same parameters as `run_single_sim.sh` Runs a single simulation on a single terminal. **Note:** gnome-terminal needs to be installed for this script to run.
- `run_batch_sim.sh` and  `run_combined_test.sh`: Both are used to run a batch of simulations, they use the `run_single_sim` script whit different configurations at every time.

### Parameters for a single simulation run

The `run_single_sim.sh` and  `run_single_sim_desktop.sh` scripts accept the following parameters:

```bash
-i <init_position: (1 / 2 / 3)>
-g <goal_position: (1 / 2 / 3)>
-n <nav_profile: ("fast" / "standard" / "safe" or fX_vX_rX)>
-r <run mros reconfiguration: ("true" / "false")>
-o <add obstacles: (0 / 1 / 2 / 3)>
-p <increase_power: ([0 -  2.0])>
-b <record rosbags: ("true" / "false")>
-e <nfr energy threshold : ([0 - 1])>
-s <nfr safety threshold : ([0 - 1])>
-l <.csv log frequency : [0 - 10])  - 0 Means no .csv logs will be recorded>
-c <close reasoner terminal after execution : ("true" / "false")>
-v <Run RVIZ : ("false" / "true" )>
```

If no parameters are given the first value is used by default

### Example of single run

This launch should result in one or two reconfigurations

```console
source ~/metacontrol_ws/devel/setup.bash
roscd metacontrol_experiments
./run_single_sim_desktop.sh -i 1 -g 1 -n "f1_v2_r2" -o 2 -r "true" -p 1.7 -c "false"
```

#### Expected outcome

The script opens three additional terminals and one rviz window

  1. roscore
  1. Gazebo simulation with MVP metacontrol world.launch
  1. mros reasoner

The output of the main console (where the script is executed) should be like this

```console
$ ./run_single_sim.sh -i 1 -g 1 -n "f1_v2_r2" -o 2 -r "true" -p 1.7 -c "false"
Make sure there is no other gazebo instances or ROS nodes running:
kill -2 6644
kill -2 6884
kill -2 7017

Start a new simulation - Goal position: 1 - Initial position  1 - Navigation profile: f1_v2_r2

Launch roscore
Launching: MVP metacontrol world.launch
Launching: mros reasoner
Running log and stop simulation node
... logging to /home/mario/.ros/log/23173622-d894-11ea-a643-3448ed792014/roslaunch-mario-laptop-7964.log
Checking log directory for disk usage. This may take a while.
Press Ctrl-C to interrupt
Done checking log file disk usage. Usage is <1GB.

started roslaunch server http://mario-laptop:45471/

SUMMARY
========

PARAMETERS
 * /create_unknown_obstacles_node/goal_nr: 1
 * /create_unknown_obstacles_node/obstacles: 2
 * /rosdistro: melodic
 * /rosversion: 1.14.6
 * /stop_simulation_node/data_log_folder: /home/mario/ros/m...
 * /stop_simulation_node/increase_power_factor: 1.7
 * /stop_simulation_node/store_data_freq: 2.0

NODES
  /
    create_unknown_obstacles_node (metacontrol_experiments/create_unknown_obstacles.py)
    stop_simulation_node (metacontrol_experiments/stop_simulation_node)

ROS_MASTER_URI=http://localhost:11311

process[stop_simulation_node-1]: started with pid [8035]
process[create_unknown_obstacles_node-2]: started with pid [8038]
[ INFO] [1596794138.256581525]: [STOP SIM] - Start node
[ INFO] [1596794138.272768379]: [STOP SIM] Will increase power consumption by 1.70 after 0.6 of the path is completed
[ INFO] [1596794138.284240283]: [STOP SIM] Logging to /home/mario/ros/metacontrol_ws/src/metacontrol_experiments/data/log_Metacontrol_sim_07-11_55_38.csv
[ INFO] [1596794138.284347624]: [STOP SIM] Node Initialization Completed
[ INFO] [1596794138.284372739]: Initialized an async multi-thread node.
[ INFO] [1596794138.284544551]: [STOP SIM] - Wait for goal msg
[ INFO] [1596794143.261260103]: [LogData :: goal_callback] - Goal Pos x -8.500000 - y 7.000000
[ INFO] [1596794143.284665200]: [STOP SIM] - Initial goal distance: 19.10
[ INFO] [1596794143.284697029]: [STOP SIM] - Wait for robot to reach the goal
[INFO] [1596794144.564601]: Adding obstacles: 2
[INFO] [1596794144.598565]: Adding obstacles.
[INFO] [1596794144.647856]: SpawnModel: Successfully spawned entity
[INFO] [1596794144.870407]: SpawnModel: Successfully spawned entity
[create_unknown_obstacles_node-2] process has finished cleanly
log file: /home/mario/.ros/log/23173622-d894-11ea-a643-3448ed792014/create_unknown_obstacles_node-2*.log
[ INFO] [1596794178.584661119]: [STOP SIM] - 2 / 3 of the route completed
[ INFO] [1596794178.585825909]: Power consumption increased  
[ INFO] [1596794203.200138944]: [LogData :: reconfig_callback] - New configuration requested f1_v2_r1
[ INFO] [1596794207.149323292]: [LogData :: goal_callback] - Goal Pos x -8.500000 - y 7.000000
[ INFO] [1596794210.412765830]: [LogData :: reconfig_callback] - New configuration requested f1_v1_r1
[ INFO] [1596794214.475655073]: [LogData :: goal_callback] - Goal Pos x -8.500000 - y 7.000000
[ INFO] [1596794218.887366611]: [STOP SIM :: stop_simulation] Killing nodes
killing /fake_localization
killed
================================================================================REQUIRED process [stop_simulation_node-1] has died!
process has finished cleanly
log file: /home/mario/.ros/log/23173622-d894-11ea-a643-3448ed792014/stop_simulation_node-1*.log
Initiating shutdown!
================================================================================
[stop_simulation_node-1] killing on exit
shutting down processing monitor...
... shutting down processing monitor complete
done
exit
Simulation Finished!!
kill -2 6644
kill -2 6884

```

##### Notes

- At the beggining of the script, all previous gazebo and ROS nodes are killed, if there is a gazebo process running or lagging, this may take some time and a message `-- gzserver still running` may appear.

- The `===REQUIRED process [stop_simulation_node-1] has died!` part, appears red, but it's not an error, is due to the process being terminated at the end of the simulation.

- RViz and the three terminals should close at the end of the simulation. However, if the `-c "false"` option is given, the `mros_reasoner` terminal will wait for an external input before closing.

### Run simulations in batch

Edit the for loops inside `./run_batch_sim.sh` to define set of parameters and whether or not to run reconfiguration

Run the script

```console
./run_batch_sim.sh
```

## Getting logs

Two type of logs are available

### csv files

The `stop_simulation_node` subscribes to the topics containing the relevant info for the logs, it extracts the corresponding info (i.e. corresponding `key` value from the `diagnostics` topic for the QA) and stores them in a `.csv` at a given frequency as

- The default storage folder is `"$(find metacontrol_experiments)/data/"`
- The storage folder can be modified in the `launch/stop_simulation.launch` file
- Store frequency can be modified in the `launch/stop_simulation.launch` file ( 0 means no recording)

### rosbags

- `.bag` files can be stored if the parameter -b true is given to the `run_single_sim.sh` scripts
- The default storage folder is `"$(find metacontrol_experiments)/bags/"`
- The storage folder can be modified in the `launch/stop_simulation.launch` file
