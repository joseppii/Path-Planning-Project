# CarND-Path-Planning-Project
Self-Driving Car Engineer Nanodegree Program

### Project Requirements
The requirements for this project are the following:
* The car must be able to drive around the track for at least 4.32 miles without incident
* The car must not exceed the speed limit
* The car must not exceed the maximum acceleration & jerk limits
* The car must avoid collisions
* The car must stay within the lanes except for the time between changing lanes
* The car must be able to change lanes
* The code must compile without errors using cmake

### File structure
The project is implemented mainly in one file, `src/main.cpp`. This file contains the bulk of the path planning code. Some helper function that where provided as part of the project are located within `src/helpers.h`. This file has not been modified. In order to utilize splines for the trajectory calculations, we imported an external library. The library is contained within `src/splines.h` and was downloaded from this location: https://kluge.in-chemnitz.de/opensource/spline/ . 

### Code Analysis

The data supplied by the simulator can be split into two categories. The actual car data, which are stored in the corresponding variables within lines 77-93, and the sensor fusion data, which are stored in the the sensor fusion pointer on line 97. The sensor fusion data are related to the other cars present in the simulation. 
The code is split in three main parts. The first part (lines 110-130) iterates over the sensor fusion data and determines if there is a car to the left/right lane or if there is a car in the same lane. This is done using the s and d coordinates (Frenet) of each reported car and our lane as a reference. Since the boundaries of our lane is +/- 2m, it is possible to figure out a nearby car's location and set an appropriate variable used for decision making.
The second part of the code is the decision making part (lanes 132-151). Based on the flags set by processing the sensor fusion data and the current lane of our car, we turn reft/right or decelerate if there is a car in close proximity. In case we are not in the center late, we return if it is safe. The code makes sure that we always travel below the maximum allowed limit of 50mph (lines 148-150). Speed is incremented by 0.224 mph at each iteration, making it impossible to go over the speed limit. 
In the third part of the code, we calculate the trajectory of the car. We start by retrieving the size of the previous path size in line 98. We then check the size of the previous path. If the size is empty (or contains a single point), we need to use the car's state and generate two path points, one based on the current and one on the previous state. In order to calculate the previous position of the car, we need to have a path that is tangent to the angle of the car, therefore we use its angle in order to calculate it (lines 161-162). If we have enough previous path points, we need to do something similar, but in this case we do not need to calculate the value of the previous path point as it is available (lines 169-181). Then we create three points that will be ahead of the car, spacing them quite far apart (30m), in the s dimension (Frenet). For this calculation we use the lane as relative reference in the d dimension (lines 184/186).
We then transform the coordinates of the points we created so far, to use the car's reference frame (lines 196-202), in order to simplify the calculations and pass them to the spline we created (lines 204-206). At this point, we get any previous path points that were still available and add them to our future path points (lines 211-213). Finally, we need to calculate some more future path points so that the total is 50. We start by calculating the break up of spline points so that we travel at the desired velocity. This is done using the equation in line 223, where we calculate the spacing between the desired points on the spline and use it obtain the x,y values for each point over a distance of 30m. We then revert back to the original reference frame and add the point to list of the previously remaining points. This set of points will be the future trajectory of the vehicle. 

### Results

The car was able to navigate itself around the track for more than 4.32 miles, without exceeding the speed limit, avoiding collisions with other cars, while maintaining a smooth ride, as acceleration and jerk was kept within comfortable limits. This can be verified on the attached video.

[![Path Planning](http://img.youtube.com/vi/tEJZy5zXtog/0.jpg)](http://www.youtube.com/watch?v=tEJZy5zXtog)

### Reflection & improvements

While the generated trajectories where quite smooth, the lane changing strategy could be improved. Currently, it is based on absolute distances, that where tuned by trial and error. Despite the fact that there are no collisions during lane changes, sometimes the car takes a lot of time to overtake other cars in situations where there are several vehicles driving at approximately the same speed. A cost function could be used for deciding when to change lane. In a similar manner, acceleration and deceleration could be decided using a cost function, as currently the car does not change its speed according to the distance of the car in front of it, responding slowly in cases of danger.

### Simulator.
You can download the Term3 Simulator which contains the Path Planning Project from the [releases tab (https://github.com/udacity/self-driving-car-sim/releases/tag/T3_v1.2).  

To run the simulator on Mac/Linux, first make the binary file executable with the following command:
```shell
sudo chmod u+x {simulator_file_name}
```

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

A really helpful resource for doing this project and creating smooth trajectories was using http://kluge.in-chemnitz.de/opensource/spline/, the spline function is in a single header file is really easy to use.

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


## Call for IDE Profiles Pull Requests

Help your fellow students!

We decided to create Makefiles with cmake to keep this project as platform
agnostic as possible. Similarly, we omitted IDE profiles in order to ensure
that students don't feel pressured to use one IDE or another.

However! I'd love to help people get up and running with their IDEs of choice.
If you've created a profile for an IDE that you think other students would
appreciate, we'd love to have you add the requisite profile files and
instructions to ide_profiles/. For example if you wanted to add a VS Code
profile, you'd add:

* /ide_profiles/vscode/.vscode
* /ide_profiles/vscode/README.md

The README should explain what the profile does, how to take advantage of it,
and how to install it.

Frankly, I've never been involved in a project with multiple IDE profiles
before. I believe the best way to handle this would be to keep them out of the
repo root to avoid clutter. My expectation is that most profiles will include
instructions to copy files to a new location to get picked up by the IDE, but
that's just a guess.

One last note here: regardless of the IDE used, every submitted project must
still be compilable with cmake and make./

## How to write a README
A well written README file can enhance your project and portfolio.  Develop your abilities to create professional README files by completing [this free course](https://www.udacity.com/course/writing-readmes--ud777).

