### Linux Dev Environment <a id="development-linux-dev-env"></a>

Based on CentOS 7, we have an early draft available inside the Icinga Vagrant boxes:
[centos7-dev](https://github.com/Icinga/icinga-vagrant/tree/master/centos7-dev).

If you're compiling Icinga 2 natively without any virtualization layer in between,
this usually is faster. This is also the reason why developers on macOS prefer native builds
over Linux or Windows VMs. Don't forget to test the actual code on Linux later! Socket specific
stuff like `epoll` is not available on Unix kernels.

Depending on your workstation and environment, you may either develop and run locally,
use a container deployment pipeline or put everything in a high end resource remote VM.

Fork https://github.com/Icinga/icinga2 into your own repository, e.g. `https://github.com/dnsmichi/icinga2`.

Create two build directories for different binary builds.

* `debug` contains the debug build binaries. They contain more debug information and run tremendously slower than release builds from packages. Don't use them for benchmarks.
* `release` contains the release build binaries, as you would install them on a live system. This helps comparing specific scenarios for race conditions and more.

```bash
mkdir -p release debug
```

Proceed with the specific distribution examples below. Keep in mind that these instructions
are best effort and sometimes out-of-date. Git Master may contain updates.

* [CentOS 7](21-development.md#development-linux-dev-env-centos)
* [Debian 10 Buster](21-development.md#development-linux-dev-env-debian)
* [Ubuntu 18 Bionic](21-development.md#development-linux-dev-env-ubuntu)


#### CentOS 7 <a id="development-linux-dev-env-centos"></a>

```bash
yum -y install gdb vim git bash-completion htop

yum -y install rpmdevtools ccache \
 cmake make gcc-c++ flex bison \
 openssl-devel boost169-devel systemd-devel \
 mysql-devel postgresql-devel libedit-devel \
 libstdc++-devel

groupadd icinga
groupadd icingacmd
useradd -c "icinga" -s /sbin/nologin -G icingacmd -g icinga icinga

ln -s /bin/ccache /usr/local/bin/gcc
ln -s /bin/ccache /usr/local/bin/g++

git clone https://github.com/icinga/icinga2.git && cd icinga2
```

The debug build binaries contain specific code which runs
slower but allows for better debugging insights.

For benchmarks, change `CMAKE_BUILD_TYPE` to `RelWithDebInfo` and
build inside the `release` directory.

First, off export some generics for Boost.

```bash
export I2_BOOST="-DBoost_NO_BOOST_CMAKE=TRUE -DBoost_NO_SYSTEM_PATHS=TRUE -DBOOST_LIBRARYDIR=/usr/lib64/boost169 -DBOOST_INCLUDEDIR=/usr/include/boost169 -DBoost_ADDITIONAL_VERSIONS='1.69;1.69.0'"
```

Second, add the prefix path to it.

```bash
export I2_GENERIC="$I2_BOOST -DCMAKE_INSTALL_PREFIX=/usr/local/icinga2"
```

Third, define the two build types with their specific CMake variables.

```bash
export I2_DEBUG="-DCMAKE_BUILD_TYPE=Debug -DICINGA2_UNITY_BUILD=OFF $I2_GENERIC"
export I2_RELEASE="-DCMAKE_BUILD_TYPE=RelWithDebInfo -DICINGA2_WITH_TESTS=ON -DICINGA2_UNITY_BUILD=ON $I2_GENERIC"
```

Fourth, depending on your likings, you may add a bash alias for building,
or invoke the commands inside:

```bash
alias i2_debug="cd /root/icinga2; mkdir -p debug; cd debug; cmake $I2_DEBUG ..; make -j2; sudo make -j2 install; cd .."
alias i2_release="cd /root/icinga2; mkdir -p release; cd release; cmake $I2_RELEASE ..; make -j2; sudo make -j2 install; cd .."
```

This is taken from the [centos7-dev](https://github.com/Icinga/icinga-vagrant/tree/master/centos7-dev) Vagrant box.


The source installation doesn't set proper permissions, this is
handled in the package builds which are officially supported.

```bash
chown -R icinga:icinga /usr/local/icinga2/var/

/usr/local/icinga2/lib/icinga2/prepare-dirs /usr/local/icinga2/etc/sysconfig/icinga2
/usr/local/icinga2/sbin/icinga2 api setup
vim /usr/local/icinga2/etc/icinga2/conf.d/api-users.conf

/usr/local/icinga2/lib/icinga2/sbin/icinga2 daemon
```

#### Debian 10 <a id="development-linux-dev-env-debian"></a>

Debian Buster doesn't need updated Boost packages from packages.icinga.com,
the distribution already provides 1.66+. For older versions such as Stretch,
include the release repository for packages.icinga.com as shown in the [setup instructions](02-installation.md#package-repositories-debian-ubuntu-raspbian).

```bash
docker run -ti debian:buster bash

apt-get update
apt-get -y install apt-transport-https wget gnupg

apt-get -y install gdb vim git cmake make ccache build-essential libssl-dev bison flex default-libmysqlclient-dev libpq-dev libedit-dev monitoring-plugins
apt-get -y install libboost-all-dev
```

```bash
ln -s /usr/bin/ccache /usr/local/bin/gcc
ln -s /usr/bin/ccache /usr/local/bin/g++

groupadd icinga
groupadd icingacmd
useradd -c "icinga" -s /sbin/nologin -G icingacmd -g icinga icinga

git clone https://github.com/icinga/icinga2.git && cd icinga2

mkdir debug release

export I2_DEB="-DBoost_NO_BOOST_CMAKE=TRUE -DBoost_NO_SYSTEM_PATHS=TRUE -DBOOST_LIBRARYDIR=/usr/lib/x86_64-linux-gnu -DBOOST_INCLUDEDIR=/usr/include -DCMAKE_INSTALL_RPATH=/usr/lib/x86_64-linux-gnu"
export I2_GENERIC="-DCMAKE_INSTALL_PREFIX=/usr/local/icinga2 -DICINGA2_PLUGINDIR=/usr/local/sbin"
export I2_DEBUG="$I2_DEB $I2_GENERIC -DCMAKE_BUILD_TYPE=Debug -DICINGA2_UNITY_BUILD=OFF"

cd debug
cmake .. $I2_DEBUG
cd ..

make -j2 install -C debug
```


The source installation doesn't set proper permissions, this is
handled in the package builds which are officially supported.

```bash
chown -R icinga:icinga /usr/local/icinga2/var/

/usr/local/icinga2/lib/icinga2/prepare-dirs /usr/local/icinga2/etc/sysconfig/icinga2
/usr/local/icinga2/sbin/icinga2 api setup
vim /usr/local/icinga2/etc/icinga2/conf.d/api-users.conf

/usr/local/icinga2/lib/icinga2/sbin/icinga2 daemon
```


#### Ubuntu 18 Bionic <a id="development-linux-dev-env-ubuntu"></a>

Requires Boost packages from packages.icinga.com.

```bash
docker run -ti ubuntu:bionic bash

apt-get update
apt-get -y install apt-transport-https wget gnupg

wget -O - https://packages.icinga.com/icinga.key | apt-key add -

. /etc/os-release; if [ ! -z ${UBUNTU_CODENAME+x} ]; then DIST="${UBUNTU_CODENAME}"; else DIST="$(lsb_release -c| awk '{print $2}')"; fi; \
 echo "deb https://packages.icinga.com/ubuntu icinga-${DIST} main" > \
 /etc/apt/sources.list.d/${DIST}-icinga.list
 echo "deb-src https://packages.icinga.com/ubuntu icinga-${DIST} main" >> \
 /etc/apt/sources.list.d/${DIST}-icinga.list

apt-get update
```

```bash
apt-get -y install gdb vim git cmake make ccache build-essential libssl-dev bison flex default-libmysqlclient-dev libpq-dev libedit-dev monitoring-plugins

apt-get install -y libboost1.67-icinga-all-dev

ln -s /usr/bin/ccache /usr/local/bin/gcc
ln -s /usr/bin/ccache /usr/local/bin/g++

groupadd icinga
groupadd icingacmd
useradd -c "icinga" -s /sbin/nologin -G icingacmd -g icinga icinga

git clone https://github.com/icinga/icinga2.git && cd icinga2

mkdir debug release

export I2_DEB="-DBoost_NO_BOOST_CMAKE=TRUE -DBoost_NO_SYSTEM_PATHS=TRUE -DBOOST_LIBRARYDIR=/usr/lib/x86_64-linux-gnu/icinga-boost -DBOOST_INCLUDEDIR=/usr/include/icinga-boost -DCMAKE_INSTALL_RPATH=/usr/lib/x86_64-linux-gnu/icinga-boost"
export I2_GENERIC="-DCMAKE_INSTALL_PREFIX=/usr/local/icinga2 -DICINGA2_PLUGINDIR=/usr/local/sbin"
export I2_DEBUG="$I2_DEB $I2_GENERIC -DCMAKE_BUILD_TYPE=Debug -DICINGA2_UNITY_BUILD=OFF"

cd debug
cmake .. $I2_DEBUG
cd ..
```

```bash
make -j2 install -C debug
```

The source installation doesn't set proper permissions, this is
handled in the package builds which are officially supported.

```bash
chown -R icinga:icinga /usr/local/icinga2/var/

/usr/local/icinga2/lib/icinga2/prepare-dirs /usr/local/icinga2/etc/sysconfig/icinga2
/usr/local/icinga2/sbin/icinga2 api setup
vim /usr/local/icinga2/etc/icinga2/conf.d/api-users.conf

/usr/local/icinga2/lib/icinga2/sbin/icinga2 daemon
```

### macOS Dev Environment <a id="development-macos-dev-env"></a>

It is advised to use Homebrew to install required build dependencies.
Macports have been reported to work as well, typically you'll get more help
with Homebrew from Icinga developers.

The idea is to run Icinga with the current user, avoiding root permissions.
This requires at least v2.11.

> **Note**
>
> This is a pure development setup for Icinga developers reducing the compile
> time in contrast to VMs. There are no packages, startup scripts or dependency management involved.
>
> **macOS agents are not officially supported.**
>
> macOS uses its own TLS implementation, Icinga relies on extra OpenSSL packages
> requiring updates apart from vendor security updates.

#### Requirements

Explicitly use OpenSSL 1.1.x, older versions are out of support.

```bash
brew install ccache boost cmake bison flex openssl@1.1 mysql-connector-c++ postgresql libpq
```

##### ccache

```bash
sudo mkdir /opt/ccache

sudo ln -s `which ccache` /opt/ccache/clang
sudo ln -s `which ccache` /opt/ccache/clang++

vim $HOME/.bash_profile

# ccache is managed with symlinks to avoid collision with cgo
export PATH="/opt/ccache:$PATH"

source $HOME/.bash_profile
```

#### Builds

Icinga is built as release (optimized build for packages) and debug (more symbols and details for debugging). Debug builds
typically run slower than release builds and must not be used for performance benchmarks.

The preferred installation prefix is `/usr/local/icinga/icinga2`. This allows to put e.g. Icinga Web 2 into the `/usr/local/icinga` directory as well.

```bash
mkdir -p release debug

export I2_USER=$(id -u -n)
export I2_GROUP=$(id -g -n)
export I2_GENERIC="-DCMAKE_INSTALL_PREFIX=/usr/local/icinga/icinga2 -DICINGA2_USER=$I2_USER -DICINGA2_GROUP=$I2_GROUP -DOPENSSL_INCLUDE_DIR=/usr/local/opt/openssl@1.1/include -DOPENSSL_SSL_LIBRARY=/usr/local/opt/openssl@1.1/lib/libssl.dylib -DOPENSSL_CRYPTO_LIBRARY=/usr/local/opt/openssl@1.1/lib/libcrypto.dylib -DICINGA2_PLUGINDIR=/usr/local/sbin -DICINGA2_WITH_PGSQL=OFF -DCMAKE_EXPORT_COMPILE_COMMANDS=ON"
export I2_DEBUG="-DCMAKE_BUILD_TYPE=Debug -DICINGA2_UNITY_BUILD=OFF $I2_GENERIC"
export I2_RELEASE="-DCMAKE_BUILD_TYPE=RelWithDebInfo -DICINGA2_WITH_TESTS=ON -DICINGA2_UNITY_BUILD=ON $I2_GENERIC"

cd debug
cmake $I2_DEBUG ..
cd ..

make -j4 -C debug
make -j4 install -C debug
```

In order to run Icinga without any path prefix, and also use Bash completion it is advised to source additional
things into the local dev environment.

```bash
export PATH=/usr/local/icinga/icinga2/sbin/:$PATH

test -f /usr/local/icinga/icinga2/etc/bash_completion.d/icinga2 && source /usr/local/icinga/icinga2/etc/bash_completion.d/icinga2
```

##### Build Aliases

This is derived from [dnsmichi's flavour](https://github.com/dnsmichi/dotfiles) and not generally best practice.

```bash
vim $HOME/.bash_profile

export I2_USER=$(id -u -n)
export I2_GROUP=$(id -g -n)
export I2_GENERIC="-DCMAKE_INSTALL_PREFIX=/usr/local/icinga/icinga2 -DICINGA2_USER=$I2_USER -DICINGA2_GROUP=$I2_GROUP -DOPENSSL_INCLUDE_DIR=/usr/local/opt/openssl@1.1/include -DOPENSSL_SSL_LIBRARY=/usr/local/opt/openssl@1.1/lib/libssl.dylib -DOPENSSL_CRYPTO_LIBRARY=/usr/local/opt/openssl@1.1/lib/libcrypto.dylib -DICINGA2_PLUGINDIR=/usr/local/sbin -DICINGA2_WITH_PGSQL=OFF -DCMAKE_EXPORT_COMPILE_COMMANDS=ON"

export I2_DEBUG="-DCMAKE_BUILD_TYPE=Debug -DICINGA2_UNITY_BUILD=OFF $I2_GENERIC"
export I2_RELEASE="-DCMAKE_BUILD_TYPE=RelWithDebInfo -DICINGA2_WITH_TESTS=ON -DICINGA2_UNITY_BUILD=ON $I2_GENERIC"

alias i2_debug="mkdir -p debug; cd debug; cmake $I2_DEBUG ..; make -j4; make -j4 install; cd .."
alias i2_release="mkdir -p release; cd release; cmake $I2_RELEASE ..; make -j4; make -j4 install; cd .."

export PATH=/usr/local/icinga/icinga2/sbin/:$PATH
test -f /usr/local/icinga/icinga2/etc/bash_completion.d/icinga2 && source /usr/local/icinga/icinga2/etc/bash_completion.d/icinga2


source $HOME/.bash_profile
```

#### Permissions

`make install` doesn't set all required permissions, override this.

```bash
chown -R $I2_USER:$I2_GROUP /usr/local/icinga/icinga2
```

#### Run

Start Icinga in foreground.

```bash
icinga2 daemon
```

Reloads triggered with HUP or cluster syncs just put the process into background.

#### Plugins

```bash
brew install monitoring-plugins

sudo vim /usr/local/icinga/icinga2/etc/icinga2/constants.conf
```

```
const PluginDir = "/usr/local/sbin"
```

#### Backends: Redis

```bash
brew install redis
brew services start redis
```

#### Databases: MariaDB

```bash
brew install mariadb
mkdir -p /usr/local/etc/my.cnf.d
brew services start mariadb

mysql_secure_installation
```

```
vim $HOME/.my.cnf

[client]
user = root
password = supersecurerootpassword

sudo -i
ln -s /Users/michi/.my.cnf $HOME/.my.cnf
exit
```

```bash
mysql -e 'create database icinga;'
mysql -e "grant all on icinga.* to 'icinga'@'localhost' identified by 'icinga';"
mysql icinga < $HOME/dev/icinga/icinga2/lib/db_ido_mysql/schema/mysql.sql
```

#### API

```bash
icinga2 api setup
cd /usr/local/icinga/icinga2/var/lib/icinga2/certs
HOST_NAME=mbpmif.int.netways.de
icinga2 pki new-cert --cn ${HOST_NAME} --csr ${HOST_NAME}.csr --key ${HOST_NAME}.key
icinga2 pki sign-csr --csr ${HOST_NAME}.csr --cert ${HOST_NAME}.crt
echo "const NodeName = \"${HOST_NAME}\"" >> /usr/local/icinga/icinga2/etc/icinga2/constants.conf
```

#### Web

While it is recommended to use Docker or the Icinga Web 2 development VM pointing to the shared IDO database resource/REST API, you can also install it locally on macOS.

The required steps are described in [this script](https://github.com/dnsmichi/dotfiles/blob/master/icingaweb2.sh).



### Windows Dev Environment <a id="development-windows-dev-env"></a>

The following sections explain how to setup the required build tools
and how to run and debug the code.

#### TL;DR

If you're going to setup a dev environment on a fresh Windows machine
and don't care for the details,

1. ensure there are 35 GB free space on C:
2. run the following in an administrative Powershell:
1. `Enable-WindowsOptionalFeature -FeatureName "NetFx3" -Online`
   (reboot when asked!)
2. `powershell -NoProfile -ExecutionPolicy Bypass -Command "Invoke-Expression (New-Object Net.WebClient).DownloadString('https://raw.githubusercontent.com/Icinga/icinga2/master/doc/win-dev.ps1')"`
   (will take some time)

This installs everything needed for cloning and building Icinga 2
on the command line (Powershell) as follows:

(Don't forget to open a new Powershell window
to be able to use the newly installed Git.)

```
git clone https://github.com/Icinga/icinga2.git
cd .\icinga2\
mkdir build
cd .\build\

& "C:\Program Files (x86)\Microsoft Visual Studio\2019\Community\Common7\IDE\CommonExtensions\Microsoft\CMake\CMake\bin\cmake.exe" `
  -DBoost_INCLUDE_DIR=C:\local\boost_1_71_0-Win64 `
  -DBISON_EXECUTABLE=C:\ProgramData\chocolatey\lib\winflexbison3\tools\win_bison.exe `
  -DFLEX_EXECUTABLE=C:\ProgramData\chocolatey\lib\winflexbison3\tools\win_flex.exe `
  -DICINGA2_WITH_MYSQL=OFF -DICINGA2_WITH_PGSQL=OFF ..

& "C:\Program Files (x86)\Microsoft Visual Studio\2019\BuildTools\MSBuild\Current\Bin\MSBuild.exe" .\icinga2.sln
```

Building icinga2.sln via Visual Studio itself seems to require a reboot
after installing the build tools and building once via command line.

#### Chocolatey

Open an administrative command prompt (Win key, type “cmd”, right-click and “run as administrator”) and paste the following instructions:

```
@powershell -NoProfile -ExecutionPolicy Bypass -Command "iex ((new-object net.webclient).DownloadString('https://chocolatey.org/install.ps1'))" && SET PATH=%PATH%;%ALLUSERSPROFILE%\chocolatey\bin
```

#### Git, Posh and Vim

In case you are used to `vim`, start a new administrative Powershell:

```
choco install -y vim
```

The same applies for Git integration in Powershell:

```
choco install -y poshgit
```

![Powershell Posh Git](images/development/windows_powershell_posh_git.png)

In order to fix the colors for commands like `git status` or `git diff`,
edit `$HOME/.gitconfig` in your Powershell and add the following lines:

```
vim $HOME/.gitconfig

[color "status"]
    changed = cyan bold
    untracked = yellow bold
    added = green bold
    branch = cyan bold
    unmerged = red bold

[color "diff"]
    frag = cyan
    new = green bold
    commit = yellow
    old = red white

[color "branch"]
  current = yellow reverse
  local = yellow
  remote = green bold
  remote = red bold
```

#### Visual Studio

Thanks to Microsoft they’ll now provide their Professional Edition of Visual Studio
as community version, free for use for open source projects such as Icinga.
The installation requires ~9GB disk space. [Download](https://www.visualstudio.com/downloads/)
the web installer and start the installation.

Note: Only Visual Studio 2019 is covered here. Older versions are not supported.

You need a free Microsoft account to download and also store your preferences.

Install the following complete workloads:

* C++ Desktop Development
* .NET Desktop Development

In addition also choose these individual components on Visual Studio:

* .NET
    * .NET Framework 4.x targeting packs
    * .NET Framework 4.x.y SDKs
* Code tools
    * Git for Windows
    * GitHub Extension for Visual Studio
    * NuGet package manager
* Compilers, build tools and runtimes
    * C# and Visual Basic Roslyn compilers
    * C++ 2019 Redistributable Update
    * C++ CMake tools for Windows
    * C++/CLI Support for v142 build tools (14.22)
    * MSBuild
    * MSVC v142 - VS 2019 C++ x64/x86 build tools (v14.22)
* Debugging and testing
    * .NET profiling tools
    * C++ profiling tools
    * Just-in-Time debugger
* Development activities
    * C# and Visual Basic
    * C++ core features
    * IntelliCode
    * Live Share
* Games and Graphics
    * Graphics debugger and GPU profiler for DirectX (required by C++ profiling tools)
* SDKs, libraries and frameworks
    * Windows 10 SDK (10.0.18362.0 or later)
    * Windows Universal C Runtime

![Visual Studio Installer](images/development/windows_visual_studio_installer_01.png)
![Visual Studio Installer](images/development/windows_visual_studio_installer_02.png)
![Visual Studio Installer](images/development/windows_visual_studio_installer_03.png)

After a while, Visual Studio will be ready.

##### Style Guide for Visual Studio

Navigate into `Tools > Options > Text Editor` and repeat the following for

- C++
- C#

Navigate into `Tabs` and set:

- Indenting: Smart (default)
- Tab size: 4
- Indent size: 4
- Keep tabs (instead of spaces)

![Visual Studio Tabs](images/development/windows_visual_studio_tabs_c++.png)


#### Flex and Bison

Install it using [chocolatey](https://www.wireshark.org/docs/wsdg_html_chunked/ChSetupWin32.html):

```
choco install -y winflexbison
```

Chocolatey installs these tools into the hidden directory `C:\ProgramData\chocolatey\lib\winflexbison\tools`.

#### OpenSSL

Icinga 2 requires the OpenSSL library. [Download](https://slproweb.com/products/Win32OpenSSL.html) the Win64 package
and install it into `c:\local\OpenSSL-Win64`.

Once asked for `Copy OpenSSLs DLLs to` select `The Windows system directory`. That way CMake/Visual Studio
will automatically detect them for builds and packaging.

> **Note**
>
> We cannot use the chocolatey package as this one does not provide any development headers.
>
> Choose 1.1.1 LTS from manual downloads for best compatibility.

#### Boost

Icinga needs the development header and library files from the Boost library.

Visual Studio translates into the following compiler versions:

- `msvc-14.2` = Visual Studio 2019

##### Pre-built Binaries

Prefer the pre-built package over self-compiling, if the newest version already exists.

Download the [boost-binaries](https://sourceforge.net/projects/boost/files/boost-binaries/) for

- msvc-14.2 is Visual Studio 2019
- 64 for 64 bit builds

```
https://sourceforge.net/projects/boost/files/boost-binaries/1.71.0/boost_1_71_0-msvc-14.2-64.exe/download
```

Run the installer and leave the default installation path in `C:\local\boost_1_71_0`.


##### Source & Compile

In order to use the boost development header and library files you need to [download](https://www.boost.org/users/download/)
Boost and then extract it to e.g. `C:\local\boost_1_71_0`.

> **Note**
>
> Just use `C:\local`, the zip file already contains the sub folder. Extraction takes a while,
> the archive contains more than 70k files.

In order to integrate Boost into Visual Studio, open the `Developer Command Prompt` from the start menu,
and navigate to `C:\local\boost_1_71_0`.

Execute `bootstrap.bat` first.

```
cd C:\local\boost_1_71_0
bootstrap.bat
```

Once finished, specify the required `toolset` to compile boost against Visual Studio.
This takes quite some time in a Windows VM. Boost Context uses Assembler code,
which isn't treated as exception safe by the VS compiler. Therefore set the
additional compilation flag according to [this entry](https://lists.boost.org/Archives/boost/2015/08/224570.php).

```
b2 --toolset=msvc-14.2 link=static threading=multi runtime-link=static address-model=64 asmflags=\safeseh
```

![Windows Boost Build in VS Development Console](images/development/windows_boost_build_dev_cmd.png)

#### TortoiseGit

TortoiseGit provides a graphical integration into the Windows explorer. This makes it easier to checkout, commit
and whatnot.

[Download](https://tortoisegit.org/download/) TortoiseGit on your system.

In order to clone via Git SSH you also need to create a new directory called `.ssh`
inside your user's home directory.
Therefore open a command prompt (win key, type `cmd`, enter) and run `mkdir .ssh`.
Add your `id_rsa` private key and `id_rsa.pub` public key files into that directory.

Start the setup routine and choose `OpenSSH` as default secure transport when asked.

Open a Windows Explorer window and navigate into

```
cd %HOMEPATH%\source\repos
```

Right click and select `Git Clone` from the context menu.

Use `ssh://git@github.com/icinga/icinga2.git` for SSH clones, `https://github.com/icinga/icinga2.git` otherwise.

#### Packages

CMake uses CPack and NSIS to create the setup executable including all binaries and libraries
in addition to setup dialogues and configuration. Therefore we’ll need to install [NSIS](http://nsis.sourceforge.net/Download)
first.

We also need to install the Windows Installer XML (WIX) toolset. This has .NET 3.5 as a dependency which might need a
reboot of the system which is not handled properly by Chocolatey. Therefore install it first and reboot when asked.

```
Enable-WindowsOptionalFeature -FeatureName "NetFx3" -Online
choco install -y wixtoolset
```

#### CMake

Icinga 2 uses CMake to manage the build environment. You can generate the Visual Studio project files
using CMake. [Download](https://cmake.org/download/) and install CMake. Select to add it to PATH for all users
when asked.

> **Note**
>
> In order to properly detect the Boost libraries and VS 2019, install CMake 3.15.2+.
>
> **Tip**
>
> Cheatsheet: https://www.brianlheim.com/2018/04/09/cmake-cheat-sheet.html

Once setup is completed, open a command prompt and navigate to

```
cd %HOMEPATH%\source\repos
```

Build Icinga with specific CMake variables. This generates a new Visual Studio project file called `icinga2.sln`.

Visual Studio translates into the following:

- `msvc-14.2` = Visual Studio 2019

You need to specify the previously installed component paths.

Variable              | Value                                                                | Description
----------------------|----------------------------------------------------------------------|-------------------------------------------------------
`BOOST_ROOT`          | `C:\local\boost_1_71_0`                                                    | Root path where you've extracted and compiled Boost.
`BOOST_LIBRARYDIR`    | Binary: `C:\local\boost_1_71_0\lib64-msvc-14.2`, Source: `C:\local\boost_1_71_0\stage` | Path to the static compiled Boost libraries, directory must contain `lib`.
`BISON_EXECUTABLE`    | `C:\ProgramData\chocolatey\lib\winflexbison\tools\win_bison.exe`     | Path to the Bison executable.
`FLEX_EXECUTABLE`     | `C:\ProgramData\chocolatey\lib\winflexbison\tools\win_flex.exe`      | Path to the Flex executable.
`ICINGA2_WITH_MYSQL`  | OFF                                                                  | Requires extra setup for MySQL if set to `ON`. Not supported for client setups.
`ICINGA2_WITH_PGSQL`  | OFF                                                                  | Requires extra setup for PgSQL if set to `ON`. Not supported for client setups.
`ICINGA2_UNITY_BUILD` | OFF                                                                  | Disable unity builds for development environments.

Tip: If you have previously opened a terminal, run `refreshenv` to re-read updated PATH variables.

##### Build Scripts

Icinga provides the build scripts inside the Git repository.

Open a new Powershell and navigate into the cloned Git repository. Set
specific environment variables and run the build scripts.

```
cd %HOMEPATH%\source\repos\icinga2

.\tools\win32\configure-dev.ps1
.\tools\win32\build.ps1
.\tools\win32\test.ps1
```

The debug MSI package is located in the `debug` directory.

If you did not follow the above steps with Boost binaries and OpenSSL
paths, you can still modify the environment variables.

```
$env:CMAKE_GENERATOR='Visual Studio 16 2019'
$env:CMAKE_GENERATOR_PLATFORM='x64'

$env:ICINGA2_INSTALLPATH = 'C:\Program Files\Icinga2-debug'
$env:ICINGA2_BUILDPATH='debug'
$env:CMAKE_BUILD_TYPE='Debug'
$env:OPENSSL_ROOT_DIR='C:\OpenSSL-Win64'
$env:BOOST_ROOT='C:\local\boost_1_71_0'
$env:BOOST_LIBRARYDIR='C:\local\boost_1_71_0\lib64-msvc-14.2'
```

#### Icinga 2 in Visual Studio

This requires running the configure script once.

Navigate to

```
cd %HOMEPATH%\source\repos\icinga2\debug
```

Open `icinga2.sln`. Log into Visual Studio when asked.

On the right panel, select to build the `Bin/icinga-app` solution.

The executable binaries are located in `Bin\Release\Debug` in your `icinga2`
project directory.

Navigate there and run `icinga2.exe --version`.

```
cd %HOMEPATH%\source\repos\icinga2\Bin\Release\Debug
icinga2.exe --version
```


#### Release Package

This is part of the build process script. Override the build type and pick a different
build directory.

```
cd %HOMEPATH%\source\repos\icinga2

$env:ICINGA2_BUILDPATH='release'
$env:CMAKE_BUILD_TYPE='RelWithDebInfo'

.\tools\win32\configure-dev.ps1
.\tools\win32\build.ps1
.\tools\win32\test.ps1
```

The release MSI package is located in the `release` directory.


### Embedded Dev Env: Pi <a id="development-embedded-dev-env"></a>

> **Note**
>
> This isn't officially supported yet, just a few hints how you can do it yourself.

The following examples source from armhf on Raspberry Pi.

#### ccache

```bash
apt install -y ccache

/usr/sbin/update-ccache-symlinks

echo 'export PATH="/usr/lib/ccache:$PATH"' | tee -a ~/.bashrc

source ~/.bashrc && echo $PATH
```

#### Build

Copy the icinga2 source code into `$HOME/icinga2`. Clone the `deb-icinga2` repository into `debian/`.

```bash
git clone https://github.com/Icinga/icinga2 $HOME/icinga2
git clone https://github.com/Icinga/deb-icinga2 $HOME/icinga2/debian
```

Then build a Debian package and install it like normal.

```bash
dpkg-buildpackage -uc -us
```
