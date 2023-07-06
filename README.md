# Building the Code

This code was intentionally written with minimal dependencies,
requiring only CMake (as a build system), your favorite
compiler (tested with Visual Studio 2017 and 2019 under Windows, and GCC under
Linux), and the OptiX 7 SDK (including CUDA 10.1 and NVIDIA driver recent
enough to support OptiX)

## Dependencies

- a compiler
    - On Windows, tested with Visual Studio 2017 and 2019 community editions
    - On Linux, tested with Ubuntu 18 and Ubuntu 19 default gcc installs
- CUDA 10.1
    - Download from developer.nvidia.com
    - on Linux, suggest to put `/usr/local/cuda/bin` into your `PATH`
- latest NVIDIA developer driver that comes with the SDK
    - download from http://developer.nvidia.com/optix and click "Get OptiX"
- OptiX 7 SDK
    - download from http://developer.nvidia.com/optix and click "Get OptiX"
    - on linux, set the environment variable `OptiX_INSTALL_DIR` to wherever you installed the SDK.
    `export OptiX_INSTALL_DIR=<wherever you installed OptiX 7 SDK>`
    - on windows, the installer should automatically put it into the right directory

The only *external* library we use is GLFW for windowing, and
even this one we build on the fly under Windows, so installing
it is required only under Linux. 

Detailed steps below:

## Building under Linux

- Install required packages

    - on Debian/Ubuntu: `sudo apt install libglfw3-dev cmake-curses-gui`
    - on RedHat/CentOS/Fedora (tested CentOS 7.7): `sudo yum install cmake3 glfw-devel freeglut-devel`

- Clone the code
```
    git clone https://gitee.com/games-assignment/optix7course.git
    cd optix7course
```

- create (and enter) a build directory
```
    mkdir build
    cd build
```

- configure with cmake
    - Ubuntu: `cmake ..`
    - CentOS 7: `cmake3 ..`

- and build
```
    make
```

## Building under Windows

- Install Required Packages
	- see above: CUDA 10.1, OptiX 7 SDK, latest driver, and cmake
- download or clone the source repository
- Open `CMake GUI` from your start menu
	- point "source directory" to the downloaded source directory
	- point "build directory" to <source directory>/build (agree to create this directory when prompted)
	- click 'configure', then specify the generator as Visual Studio 2017 or 2019, and the Optional platform as x64. If CUDA, SDK, and compiler are all properly installed this should enable the 'generate' button. If not, make sure all dependencies are properly installed, "clear cache", and re-configure.
	- click 'generate' (this creates a Visual Studio project and solutions)
	- click 'open project' (this should open the project in Visual Studio)

## 交互方式

- 按'C'在场景中添加方块
    - 需要输入方块的中心坐标，边长和颜色
    - 实现方式：预先设定最大可添加方块数量MAX_CUBE_NUM，默认为16，在buildAccel函数建立Acceleration Structure时新增MAX_CUBE_NUM个build input，顶点坐标都设置为0使其不可见，在buildflag中添加OPTIX_BUILD_FLAG_ALLOW_UPDATE来允许后续对Acceleration Structure的更新；在buildSBT函数中同样新增MAX_CUBE_NUM * RAY_TYPE_COUNT个HitgroupRecord
    然后添加函数updateAccel，这个函数首先接受输入的方块中心坐标，边长和颜色，然后以坐标和边长建立方块的TriangleMesh，更新对应的buffer和buildinput，然后调用updateSBT函数更新方块颜色，最后将OptixAccelBuildOptions::operation设置为OPTIX_BUILD_OPERATION_UPDATE后调用optixBuildAccel完成更新
    - 添加函数updateSBT来更新对应方块的HitgroupRecord中的color，然后上传到sbt中

- 按'L'在场景中添加光源
    - 需要输入光源的