# RM Manipulator SDK
wr_arm_sdk provides C++ interface to control the arm and get the joint angle status. For the complete interface function description, refer to the SDK manual.

## Supported Platforms
- Architecture: x86_64/arm64
- OS: Ubuntu (18.04, 20.04)

## Build SDK
### Install Dependencies
```
$ sudo apt-get update
$ sudo apt-get install build-essential git cmake libasio-dev
```

### Install the SDK
```
$ git clone https://github.com/westonrobot/wr_arm_sdk.git
$ cd wr_arm_sdk
$ mkdir build
$ cd build
```

### Select Build Type
There are two build types available:
- Debug: Debugging information, no optimization
- Release: No debugging information, full optimization

**Debug** is selected as the default build type, to continue with debug, run:
```
$ cmake ..
$ make
```
or
```
$ cmake -DCMAKE_BUILD_TYPE=Release ..
$ make
```

**Note**: The **main.cpp** (wr_arm_sdk/demo/) contains simple robot arm operations. Build the project using the 'make' command whenever there are changes to this file.

### Run Demo
```
$ ./bin/demo