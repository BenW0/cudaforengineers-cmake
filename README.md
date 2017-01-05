This repository contains a modified form of a few of the sample codes from [Cuda for Engineers](http://www.cudaforengineers.com), modified to use CMake for generating build environments.

To run, you will need the NVIDIA CUDA Toolkit and an appropriate (CUDA-compatible) compiler for your OS.

Windows binaries (maybe only VS 2013 binaries), libraries, and headers for [FreeGLUT](http://www.transmissionzero.co.uk/software/freeglut-devel/), and [GLEW](glew.sourceforge.net) are also included for your convenience. Those files are covered by the license agreements of their respective projects.

##To Build
First, build GLUI, unless your platform has a package already present for it. Use CMake on (libs/glui/CMakeLists.txt) to create a build environment for GLUI in libs/glui/bin and compile. If on Windows, be sure to build both debug and release variants.

Next, create a build environment for flashlight using CMake (flashlight/CMakeLists.txt). It should automatically find the GLUI libraries, and if on Windows it can retrieve FreeGLUT and GLEW from the included binaries as well.

##Installation and Compilation on Ubuntu

###Build Tools
install git - `sudo apt-get install git` (already on Ubuntu)

install cmake - `sudo apt-get install cmake` (already on Ubuntu) and `sudo apt-get install cmake-curses-gui`

###Libraries
`sudo apt-get install freeglut3-dev`

`sudo apt-get install libglew-dev`

`sudo apt-get install libxi-dev`

`sudo apt-get install libxmu-dev`

`sudo apt-get install libglew1.13`

###Code
* switch to the directory you want to work in
* clone this repository - `git clone https://github.com/BenW0/flashlight-GLUI-example.git`
* switch to cmake_build branch - `git checkout cmake_build`

###Compiling GLUI
GLUI isn't available on most platforms as a package, so we'll build it from source using a CMakeList I put together for it.

* Configure the glui library using CMake - `cd libs/glui && mkdir bin && ccmake ../`
 * Press "c" to configure. Hopefully no errors occur
 * Press "Enter" on the `CMAKE_BUILD_TYPE` field and type Release to specify a release build
 * Press "g" to generate the build file
* Build the project - `make`. Afterwards there should be a `libglui32.a` file in libs/glui/bin

Now we have the static library for GLUI and can use it to build the Flashlight application

###Flashlight
We will switch back to the flashlight project folder, create a "bin" folder to hold our build instructions and binaries,
and use CMake to populate the build instructions.
~~~
cd flashlight-GLUI-example/flashlight
mkdir bin && cd bin
ccmake ../
~~~

The CMake screen should appear. Press "c" to configure. As before, we want to specify the build type. Press "Enter" on the `CMAKE_BUILD_TYPE` field and type Debug or Release, as you wish. Then use "g" to generate the makefile and exit.

Now we'll run make to compile the source into an executable, linked with GLEW, GLUT, and GLUI.

~~~
make
~~~

If it worked, an executable file should appear.

## Working with Eclipse
References: [official wiki](https://cmake.org/Wiki/Eclipse_CDT4_Generator), [tip I needed](http://stackoverflow.com/questions/11645575/importing-a-cmake-project-into-eclipse-cdt)

First, Eclipse doesn't like your CMake Project Name and Binary Name to be the same, so in the first few lines of CMakeLists.txt, change either the project() or set(EXE_NAME ...) calls so they are not the same.

Build a CMake project for Eclipse using

~~~
path/to/flashlight$ cmake -G "Eclipse CDT4 - Unix Makefiles" .
~~~

Note two things: First, Eclipse really wants its project files to be at the root of the source tree, so we *don't* use a bin subfolder this time. Second, the indication that this worked is the presence of *hidden* ".project" and ".cproject" files in the working directory; use `ls -a` to check whether they were created.

Now, open Eclipse and go to File->Import. You want General->Existing Projects Into Workspace. On the next screen, navigate to the source directory and *important* Uncheck the Copy Projects Into Workspace checkbox (otherwise, building within Eclipse fails).

This seems to work with Eclipse NSight Edition version 8.

## Converting other samples (or your own project) to CMake

### Prerequisites

[CMake](https://cmake.org/) is a cross-platform tool for managing platform-specific builds for your code. The same CMake configuration file can be used to create a makefile on Linux and a Visual Studio project on Windows, for example. These instructions assume you already have CMake installed on your platform. On Linux, this tutorial assumes you have also installed the command line GUI, ccmake (`apt-get install cmake-curses-gui`)

### Files and Directories

The provided CMake configuration files assume the source files are separate from the generated build files and output binary files. This makes everything cleaner, but requires a bit of reorganization of the sample code provided in the original repository.

First, move all the source code into a new subdirectory named "src". Then, delete the Visual Studio solution (.sln) and project (.vcxproj) as well as the Makefile. After this step, there should be only one folder in the project, named "src" and no other files.

The CMake build scripts we have created rely explicitly on two directories external to the project: `libs` and `include`. `libs` contains several useful CMake scripts for locating and configuring various libraries, as well as binaries for FreeGLUT and GLEW on Windows (provided because of Windows' lack of a package manager). `include` contains a few header files which should be included in all projects.

### Selecting and Setting Up a CMakeLists.txt

The CMake build rules are stored in a special file called CMakeLists.txt. We have provided several template CMakeLists.txt, depending on which dependencies are required for your project. Go select a CMakeLists.txt file from one of the subfolders in this repository according to your needs:

* CUDA development only: `dist_v1_cuda/CMakeLists.txt`
* CUDA + OpenGL Interop (GLEW, GLUT): `vis_3d/CMakeLists.txt`

Copy the appropriate CMakeLists.txt file into the root folder of your current project. Open it in your favorite text editor and find the lines near the top which read:

~~~
project(dist_v1_cuda)
set(EXE_NAME dist_v1_cuda)
~~~

Change "dist_v1_cuda" to match the name of your current project on both lines and save the file.

### Configuring and Generating using CMake

This process varies somewhat depending on whether a terminal or the Windows GUI for CMake is used.

#### Using a terminal (on any OS)

Start a terminal in the project folder (this folder contains your CMakeLists.txt) and create a subdirectory named `bin`. Change to this new directory and execute `ccmake ../` to enter the terminal-based CMake GUI.
* Press "c" to configure. Hopefully no errors occur
* Press "Enter" on the `CMAKE_BUILD_TYPE` field and type Release or Configure to specify the build type you wish.
* Press "g" to generate the build files

To compile, use `make` in the `bin` directory.

If you don't want to use the terminal GUI, you can specify the build type, configure, and generate the build files using the following command line:

~~~
path/to/project/bin$cmake -DCMAKE_BUILD_TYPE=Release ..
~~~

#### Using Windows GUI

Open the CMake GUI from the Start Menu, then follow these steps:
* Click the "Browse Source..." button and navigate to the folder containing your CMakeLists.txt
* Copy the "source code" path from the top line and paste it into the second line ("where to build the binaries"), then append "/bin"
* Click the Configure button (in the bottom left)
 * In the dialog that appears, select your compiler of choice. For CUDA compatibility on Windows, you must use a version of Visual Studio, and Win64 is preferred if not required by the tool chain. For example, I use "Visual Studio 12 2013 Win64"
 * Click Finish to accept all other options
* It will think for a bit; if any errors appear, a dialog will notify you and the log at the bottom of the window will give details. Red highlighted lines in the center of the window are NOT ERRORS.
* Click the Generate button to create the Visual Studio project.
* Click the Open Project button to open Visual Studio and use it to compile your code.

Once you have done this once, you do NOT need to use the CMake GUI to re-generate the project if you edit your CMakeLists.txt unless you want to change versions of Visual Studio or do other fun things not discussed here.

__Note:__ When working with Visual Studio solutions generated by CMake, there are three virtual projects created, all opened simultaneously. `ZERO_CHECK` is a dummy project which checks if CMakeLists has been changed and rebuilds the project as needed. It should never contain code files and doesn't compile anything. `ALL_BUILD` is a dummy project which builds all other projects (similar to `make all`) and is generally not used. The third project has a name specified in your CMakeLists.txt file and contains your code. You must make this project the "Start Up" project so Visual Studio knows which one to run. To do this, right click on your project in the Solution Explorer window and select "Set as Start Up Project" from the menu. This will turn the project title bold-face

Use Build->Build All to compile your code. Occasionally, especially at first or after you have changed your CMakeLists.txt file, a dialog may appear asking you if you want to reload the project because it has been changed by another program. Click "Reload All" when this happens.

##Errata
Tweaks by Ben Weiss; original source by Duane Storti and Mete Yyurtoglu.