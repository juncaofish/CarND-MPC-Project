# CarND-Controls-MPC
Self-Driving Car Engineer Nano degree Program

---
## Project Description
### Overview
Model predictive control ([MPC](https://en.wikipedia.org/wiki/Model_predictive_control)) is an advanced method of process control which relies on dynamic models of the process. 
Model predictive controllers rely on dynamic models of the process, most often linear empirical models obtained by system identification. 
The main advantage of MPC is the fact that it allows the current timeslot to be optimized, while keeping future timeslots in account. Thus MPC has the ability to anticipate future events and can take control actions accordingly, which differs from Previous PID controllers. 

The MPC controller framework consists in four main components:

1. *Trajectory* taken in consideration during optimization. This is parameterised by a number of time steps N spaced out by a time dt. Clearly, the number of variables optimized is directly proportional to N, so this must be considered in case there are computational constraints.

2. *Vehicle Model*, equations that describes system behaviour and updates across time steps. We used a simplified kinematic model (so called bicycle model) described by a state of six parameters:
    * **x** car position (x-axis)
    * **y** car position (y-axis)
    * **psi** car's heading direction
    * **v** car's velocity
    * **cte** cross-track error
    * **epsi** orientation error

3. *Constraints* in actuators' response. In this project we set these constraints as follows:
    * **steering**: bounded in range *[-25°, 25°]*
    * **acceleration**: bounded in range *[-1, 1]* from full brake to full throttle
4. *Cost Function* on whose optimization is based the whole control process. 
    * Usually cost function is made of the sum of different terms. 
    * Besides the main terms that depends on reference values (e.g. cross-track or heading error)
    * Other regular terms are present to enforce the smoothness in the controller response.
    Cost function for this project is implemented at lines 53-80 in MPC.cpp.

### Tuning Trajectory Parameters
Both N and dt are fundamental parameters in the optimization process. In particular, T = N * dt constitutes the prediction horizon considered during optimisation. 
**N** is the number of timesteps in the horizon. 
**dt** is how much time elapses between actuations.
**T** should be as large as possible, while dt should be as small as possible.
These values have to be tuned keeping in mind a couple of things:
* large dt result in less frequent actuations, which in turn could result in the difficulty in following a continuous reference trajectory (so called discreatization error)
* despite the fact that having a large T could benefit the control process, consider that predicting too far in the future does not make sense in real-world scenarios.
* large T and small dt lead to large N. As mentioned above, the number of variables optimized is directly proportional to N, so will lead to an higher computational cost.
In the project I empirically set (by visually inspecting the vehicle's behaviour in the simulator) these parameters to be N=10 and dt=0.1, for a total of T=1s in the future.

### Tuning cost function terms weights
Weights for cost function terms that need tuning include:
* cte/epsi/velocity reference terms
* actuator terms
* actuators smoothness in change

Generally, the weights are kept as 1 and only adjusted according to the actual behaviour for specific terms.

### Changing Reference System
In order to ease later computation, coordinates are converted from global reference system into car's own reference system.

### Dealing with Latency
To mimic real driving conditions where the car does actuate the commands instantly, a 100ms latency delay has been introduced before sending the data message to the simulator. In order to deal with latency, state is predicted one time step ahead before feeding it to the solver.

## Dependencies

* cmake >= 3.5
 * All OSes: [click here for installation instructions](https://cmake.org/install/)
* make >= 4.1(mac, linux), 3.81(Windows)
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
    Some function signatures have changed in v0.14.x. See [this PR](https://github.com/udacity/CarND-MPC-Project/pull/3) for more details.

* **Ipopt and CppAD:** Please refer to [this document](https://github.com/udacity/CarND-MPC-Project/blob/master/install_Ipopt_CppAD.md) for installation instructions.
* [Eigen](http://eigen.tuxfamily.org/index.php?title=Main_Page). This is already part of the repo so you shouldn't have to worry about it.
* Simulator. You can download these from the [releases tab](https://github.com/udacity/self-driving-car-sim/releases).
* Not a dependency but read the [DATA.md](./DATA.md) for a description of the data sent back from the simulator.


## Basic Build Instructions

1. Clone this repo.
2. Make a build directory: `mkdir build && cd build`
3. Compile: `cmake .. && make`
4. Run it: `./mpc`.

## Tips

1. It's recommended to test the MPC on basic examples to see if your implementation behaves as desired. One possible example
is the vehicle starting offset of a straight line (reference). If the MPC implementation is correct, after some number of timesteps
(not too many) it should find and track the reference line.
2. The `lake_track_waypoints.csv` file has the waypoints of the lake track. You could use this to fit polynomials and points and see of how well your model tracks curve. NOTE: This file might be not completely in sync with the simulator so your solution should NOT depend on it.
3. For visualization this C++ [matplotlib wrapper](https://github.com/lava/matplotlib-cpp) could be helpful.)
4.  Tips for setting up your environment are available [here](https://classroom.udacity.com/nanodegrees/nd013/parts/40f38239-66b6-46ec-ae68-03afd8a601c8/modules/0949fca6-b379-42af-a919-ee50aa304e6a/lessons/f758c44c-5e40-4e01-93b5-1a82aa4e044f/concepts/23d376c7-0195-4276-bdf0-e02f1f3c665d)
5. **VM Latency:** Some students have reported differences in behavior using VM's ostensibly a result of latency.  Please let us know if issues arise as a result of a VM environment.

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

More information is only accessible by people who are already enrolled in Term 2
of CarND. If you are enrolled, see [the project page](https://classroom.udacity.com/nanodegrees/nd013/parts/40f38239-66b6-46ec-ae68-03afd8a601c8/modules/f1820894-8322-4bb3-81aa-b26b3c6dcbaf/lessons/b1ff3be0-c904-438e-aad3-2b5379f0e0c3/concepts/1a2255a0-e23c-44cf-8d41-39b8a3c8264a)
for instructions and the project rubric.

## Hints!

* You don't have to follow this directory structure, but if you do, your work
  will span all of the .cpp files here. Keep an eye out for TODOs.

## Call for IDE Profiles Pull Requests

Help your fellow students!

We decided to create Makefiles with cmake to keep this project as platform
agnostic as possible. Similarly, we omitted IDE profiles in order to we ensure
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
