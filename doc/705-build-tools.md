
### Build Tools <a id="development-develop-builds-tools"></a>

#### CMake <a id="development-develop-builds-cmake"></a>

In its early development stages in 2012, Icinga 2 was built with autoconf/automake
and separate Windows project files. We've found this very fragile, and have changed
this into CMake as our build tool.

The most common benefits:

* Everything is described in CMakeLists.txt in each directory
* CMake only needs to know that a sub directory needs to be included.
* The global CMakeLists.txt acts as main entry point for requirement checks and library/header includes.
* Separate binary build directories, the actual source tree stays clean.
* CMake automatically generates a Visual Studio project file `icinga2.sln` on Windows.

#### Unity Builds <a id="development-develop-builds-unity-builds"></a>

Another thing you should be aware of: Unity builds on and off.

Typically, we already use caching mechanisms to reduce recompile time with ccache.
For release builds, there's always a new build needed as the difference is huge compared
to a previous (major) release.

Therefore we've invented the Unity builds, which basically concatenates all source files
into one big library source code file. The compiler then doesn't need to load the many small
files but compiles and links this huge one.

Unity builds require more memory which is why you should disable them for development
builds in small sized VMs (Linux, Windows) and also Docker containers.

There's a couple of header files which are included everywhere. If you touch/edit them,
the cache is invalidated and you need to recompile a lot more files then. `base/utility.hpp`
and `remote/zone.hpp` are good candidates for this.
