# CarND-Path-Planning-Project
Self-Driving Car Engineer Nanodegree Program Project Submission

## Project Writeup
The goal of the project is to create a pipeline for path planning that allows the vehicle to navigate safely in its environment. The vehicle must not violate any of the set threshold values such as jerk and acceleration and must not come in contact with other vehicles on the road. For this, two main modules are used: the first uses sensor data to plan according to the current state of the environment by deciding what to do when closing in to a car and the second module uses this information to generate a safe trajectory so that the car navigates the road. The following sections explain the implementation of these modules.

### Path Planning

In order to plan the behavior of our vehicle we need to use the sensor data passed from the simulator that contains a list of vehicles detected. We go through the list of detections and calculate what the best behavior of the vehicle should be based on the current state. We first define that our desired behavior is for the vehicle to remain in its current lane at a set velocity unless there the car is approaching another car going at a lower speed. If this is true, we will slow down to avoid a front collision and check if it is safe to make a lane change. We use the detection data that contains the vehicle speed and the s coordinate in order to check against a desired distance threshold from our current position and predict a position in the future using velocity (lines 305-310).

For this we first check if there are any cars in our lane in front of our car that are getting too close to our current position, if there are we will set a flag for our next module to use that specified that the vehicle is close to another car. We will also check if the adjacent lanes are clear by verifying that there are no cars within a specified distance window to the front and back of our car. If the lane is clear we will set a flag for the behavior module (lines 264-319).

In the behavior module section we check if the proximity flag was set, if it is not set we can assume that is is safe to proceed in the lane and to accelerate to the threshold speed or limit avoiding a jerk or acceleration violation. If the flag is set we will reduce our speed, again without a jerk or acceleration violation, and check if there is a possible lane change. If there is, we will set our target lane to the specified lane. This lane value is then used with our trajectory generation and will generate a trajectory that produces the lane change.

### Trajectory Generation

First we define a reference position for the car as a starting point for the waypoints to be generated. To smooth the path from frame to frame, the previously generated point returned from the simulator can be used, so we check if there are enough points to set our starting point. If there are not enough points we use the current state of the car and calculate x and y values tangent to the car for our previous position. After this calculation, we have 2 value pairs in our initial points array: the previous x and y values, and the current x and y values (lines 344-380).

The next step is to generate three more x and y points that are spaced apart by 30 from the current s values and that also depend on the desired lane. The next step is to do a coordinate transformation so that our points are based on the car's reference frame to simplify the math (lines383-403).

The next step is to use the Spline library that generates a polynomial fit based on specified points. Adding the points from the previous section generates a Spline along the specified trajectories that allows us to get y points at specific locations from on x values (lines 406-408). Here is where we can start creating our vector with our desired trajectory. The first step is to add any remaining points from the previous_path returned from the simulator so that we only need to use the Spline to generate remaining values (lines 410-414). For these remaining values, we use the method shown in the project walkthrough, where break up the Spline using a target x value and and obtain the y value from the Spline in this case at 30. We calculate the linear distance from the car to this point in the horizon to be used to along with our reference velocity so generate a value N for our points to be spaced apart so that the car is going at the desired velocity. From this we can generate x values based on N and use it to obtain y values from the Spline with our desired spacing between the points. We have to perform a reverse coordinate transformation so that the coordinates are back to map coordinates and we add the points to our waypoint vector that now contains previous points plus the additional generated points using the Spline (lines 416-441).

At this point, we now have our desired waypoints using previous path values and points generated by the Spline that can be passed to the simulator and that result in a smooth transition from frame to frame. Here is a sample of the result:

![Sample Run 1 in Simulator](output/path-1.gif)


## Project Information

### Simulator.
You can download the Term3 Simulator which contains the Path Planning Project from the [releases tab (https://github.com/udacity/self-driving-car-sim/releases).

### Goals
In this project your goal is to safely navigate around a virtual highway with other traffic that is driving +-10 MPH of the 50 MPH speed limit. You will be provided the car's localization and sensor fusion data, there is also a sparse map list of waypoints around the highway. The car should try to go as close as possible to the 50 MPH speed limit, which means passing slower traffic when possible, note that other cars will try to change lanes too. The car should avoid hitting other cars at all cost as well as driving inside of the marked road lanes at all times, unless going from one lane to another. The car should be able to make one complete loop around the 6946m highway. Since the car is trying to go 50 MPH, it should take a little over 5 minutes to complete 1 loop. Also the car should not experience total acceleration over 10 m/s^2 and jerk that is greater than 10 m/s^3.

#### The map of the highway is in data/highway_map.txt
Each waypoint in the list contains  [x,y,s,dx,dy] values. x and y are the waypoint's map coordinate position, the s value is the distance along the road to get to that waypoint in meters, the dx and dy values define the unit normal vector pointing outward of the highway loop.

The highway's waypoints loop around so the frenet s value, distance along the road, goes from 0 to 6945.554.

## Basic Build Instructions

1. Clone this repo.
2. Make a build directory: `mkdir build && cd build`
3. Compile: `cmake .. && make`
4. Run it: `./path_planning`.

Here is the data provided from the Simulator to the C++ Program

#### Main car's localization Data (No Noise)

["x"] The car's x position in map coordinates

["y"] The car's y position in map coordinates

["s"] The car's s position in frenet coordinates

["d"] The car's d position in frenet coordinates

["yaw"] The car's yaw angle in the map

["speed"] The car's speed in MPH

#### Previous path data given to the Planner

//Note: Return the previous list but with processed points removed, can be a nice tool to show how far along
the path has processed since last time.

["previous_path_x"] The previous list of x points previously given to the simulator

["previous_path_y"] The previous list of y points previously given to the simulator

#### Previous path's end s and d values

["end_path_s"] The previous list's last point's frenet s value

["end_path_d"] The previous list's last point's frenet d value

#### Sensor Fusion Data, a list of all other car's attributes on the same side of the road. (No Noise)

["sensor_fusion"] A 2d vector of cars and then that car's [car's unique ID, car's x position in map coordinates, car's y position in map coordinates, car's x velocity in m/s, car's y velocity in m/s, car's s position in frenet coordinates, car's d position in frenet coordinates.

## Details

1. The car uses a perfect controller and will visit every (x,y) point it recieves in the list every .02 seconds. The units for the (x,y) points are in meters and the spacing of the points determines the speed of the car. The vector going from a point to the next point in the list dictates the angle of the car. Acceleration both in the tangential and normal directions is measured along with the jerk, the rate of change of total Acceleration. The (x,y) point paths that the planner recieves should not have a total acceleration that goes over 10 m/s^2, also the jerk should not go over 50 m/s^3. (NOTE: As this is BETA, these requirements might change. Also currently jerk is over a .02 second interval, it would probably be better to average total acceleration over 1 second and measure jerk from that.

2. There will be some latency between the simulator running and the path planner returning a path, with optimized code usually its not very long maybe just 1-3 time steps. During this delay the simulator will continue using points that it was last given, because of this its a good idea to store the last points you have used so you can have a smooth transition. previous_path_x, and previous_path_y can be helpful for this transition since they show the last points given to the simulator controller with the processed points already removed. You would either return a path that extends this previous path or make sure to create a new path that has a smooth transition with this last path.

## Tips

A really helpful resource for doing this project and creating smooth trajectories was using http://kluge.in-chemnitz.de/opensource/spline/, the spline function is in a single hearder file is really easy to use.

---

## Dependencies

* cmake >= 3.5
  * All OSes: [click here for installation instructions](https://cmake.org/install/)
* make >= 4.1
  * Linux: make is installed by default on most Linux distros
  * Mac: [install Xcode command line tools to get make](https://developer.apple.com/xcode/features/)
  * Windows: [Click here for installation instructions](http://gnuwin32.sourceforge.net/packages/make.htm)
* gcc/g++ >= 5.4
  * Linux: gcc / g++ is installed by default on most Linux distros
  * Mac: same deal as make - [install Xcode command line tools]((https://developer.apple.com/xcode/features/)
  * Windows: recommend using [MinGW](http://www.mingw.org/)
* [uWebSockets](https://github.com/uWebSockets/uWebSockets)
  * Run either `install-mac.sh` or `install-ubuntu.sh`.
  * If you install from source, checkout to commit `e94b6e1`, i.e.
    ```
    git clone https://github.com/uWebSockets/uWebSockets
    cd uWebSockets
    git checkout e94b6e1
    ```

## Editor Settings

We've purposefully kept editor configuration files out of this repo in order to
keep it as simple and environment agnostic as possible. However, we recommend
using the following settings:

* indent using spaces
* set tab width to 2 spaces (keeps the matrices in source code aligned)

## Code Style

Please (do your best to) stick to [Google's C++ style guide](https://google.github.io/styleguide/cppguide.html).

## Project Instructions and Rubric

Note: regardless of the changes you make, your project must be buildable using
cmake and make!
