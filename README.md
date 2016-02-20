# Overview

This repository demonstrates how to use both pre-existing and custom ROS message
structures in C++ and Python via CMake and make. The goal is to show that,
while ROS itself is **developed** using catkin, package.xml meta-data
files, and other tools that help with managing a workspace drawn from
multiple repositories, you don't have to use those tools when developing
your software that **uses** ROS.

It should be easy to use ROS just as you would any other software
dependency, without giving consideration to the tools that are used
internal to the development of ROS (though you might also find those tools
useful for your own projects). We can do this because (thanks to catkin)
ROS packages all provide CMake configuration files that allow you to
`find_package()` and use them in your own CMake build just like most modern
software.

ROS packages also provide `pkg-config` files, which opens the door to doing
everything shown below directly with make, not CMake; [see below](#using-plain-make-ubuntu-linux-or-osx)
for that.

Some caveats:
* we're just working with messages, not libraries;
* we're not fully using the message structures (e.g., you can't call ros::Time::now() to get the current time);
* we're not using any run-time tools (rostopic, rosmsg, etc.)

These topics can all be explored further, to see what the constraints are.

# Ubuntu Linux (using binary ROS packages)

## Build and install

### Install ROS message packages
Installing on Ubuntu is easy because we have pre-packaged binaries.
~~~
# Install the minimal prerequisites: catkin plus any message packages that
# you need.  We have to explicitly install the catkin package here because
# the sensor packages don't require it but we will need to get the macros
# for doing code generation on our custom messages.
sudo apt-get install ros-indigo-catkin ros-indigo-sensor-msgs ros-indigo-geometry-msgs ros-indigo-map-msgs
~~~

### Build the example
Here's where it's the normal CMake routine, plus some initial
environment configuration to ensure that we can find the ROS packages that
are installed in `/opt/ros/indigo`.
~~~
# Get the example code
git clone https://github.com/gerkey/ros1_msg_reuse
cd ros1_msg_reuse
# Set up some environment variables. If you don't want such fine-grained
# control, you could instead do `. /opt/ros/indigo/setup.sh`
export CMAKE_PREFIX_PATH=/opt/ros/indigo:$CMAKE_PREFIX_PATH
export PYTHONPATH=/opt/ros/indigo/lib/python2.7/dist-packages:$PYTHONPATH
export CPATH=/opt/ros/indigo/include:$CPATH
# Build with cmake as usual
mkdir build
cd build
cmake -DCMAKE_INSTALL_PREFIX=/tmp/myproject ..
make install
~~~

## Run
~~~
# Extend your python path to find both your own installation and the ROS installation
export PYTHONPATH=/tmp/myproject/lib/python2.7/dist-packages:/opt/ros/indigo/lib/python2.7/dist-packages:$PYTHONPATH
# Run the executables
/tmp/myproject/bin/use_existing_msg
/tmp/myproject/bin/use_custom_msg
/tmp/myproject/bin/use_msgs.py
~~~

# Mac OSX (from source)

## Build and install

### Install prerequisites
We need to install a few system dependencies (on Ubuntu this step is done
automatically by the dependencies encoded in the binary packages). We get
those dependencies from a combination of brew and pip.
~~~
# <Install brew and configure your environment to include /usr/local (http://brew.sh)>
# Update brew and add a tap that will provide some ROS-specific dependencies
brew update
brew tap ros/deps
# brew install some stuff
brew install cmake libyaml console_bridge
# Also get pip
sudo easy_install pip
# pip install some stuff
# (Note: this step avoids using rosdep to resolve system dependencies and is therefore somewhat brittle.)
sudo -H pip install -U wstool rosinstall rosinstall_generator rospkg catkin-pkg Distribute PyYAML empy argparse
~~~

### Install ROS message packages
We don't supply binary packages of ROS for OSX, so we'll need to pull the
source and build it. The source lives in multiple repositories, so we'll use `rosinstall_generator` to dynamically
generate a custom recipe to get the source for just the packages that we
want, at the right versions.
~~~
# Make a place to work (could be anywhere)
mkdir ~/ros1_ws
cd ~/ros1_ws
# Download a custom install file for the ROS packages that we're going to use
rosinstall_generator --deps --rosdistro indigo --tar --wet-only sensor_msgs geometry_msgs map_msgs > ros1.repos
# Pull the sources described in the file into a subdirectory `src`
wstool init -j8 src ros1.repos
# Build the ROS packages, installing them into a subdirectory `install_isolated`
./src/catkin/bin/catkin_make_isolated --install -DCMAKE_BUILD_TYPE=Release
~~~

### Build the example
Like Ubuntu, this step is normal CMake, plus some environment configuration
to find the ROS packages installed in `~/ros1_ws`.
~~~
# Get the example code
git clone https://github.com/gerkey/ros1_msg_reuse
cd ros1_msg_reuse
# Set up some environment variables. If you don't want such fine-grained
# control, you could instead do `. ~/ros1_ws/setup.sh`
export CMAKE_PREFIX_PATH=$HOME/ros1_ws/install_isolated:$CMAKE_PREFIX_PATH
export PYTHONPATH=$HOME/ros1_ws/install_isolated/lib/python2.7/site-packages:$PYTHONPATH
export CPATH=$HOME/ros1_ws/install_isolated/include:$CPATH
# Build with cmake as usual
mkdir build
cd build
cmake -DCMAKE_INSTALL_PREFIX=/tmp/myproject ..
make install
~~~

## Run
~~~
# Extend your python path to find both your own installation and the ROS installation
export PYTHONPATH=/tmp/myproject/lib/python2.7/site-packages:$HOME/ros1_ws/install_isolated/lib/python2.7/site-packages:$PYTHONPATH
# Run the executables
/tmp/myproject/bin/use_existing_msg
/tmp/myproject/bin/use_custom_msg
/tmp/myproject/bin/use_msgs.py
~~~

# Ubuntu Linux (from source)
TODO

# Using plain make (Ubuntu Linux or OSX)
Because ROS packages also provide `pkg-config` files, we can run the build
from plain make. To try it out, run through the same steps as above, but
when you get to the `mkdir build` step, stop and do the following instead:

~~~
# Ubuntu, using binary debs
export PKG_CONFIG_PATH=/opt/ros/indigo/lib/pkgconfig:$PKG_CONFIG_PATH
# OSX, using local build
#export PKG_CONFIG_PATH=$HOME/ros1_ws/install_isolated/lib/pkgconfig:$PKG_CONFIG_PATH
make install
~~~

You should get the same result as with CMake, installed to /tmp/myproject.

