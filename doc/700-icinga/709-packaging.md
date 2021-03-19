This documentation is explicitly meant for packagers and the Icinga
build infrastructure.

The following requirements need to be fulfilled in order to build the
Icinga application using a dist tarball (including notes for distributions):

* cmake >= 2.6
* GNU make (make) or ninja-build
* C++ compiler which supports C++11
    * RHEL/Fedora/SUSE: gcc-c++ >= 4.7 (extra Developer Tools on RHEL5/6 see below)
    * Debian/Ubuntu: build-essential
    * Alpine: build-base
    * you can also use clang++
* pkg-config
* OpenSSL library and header files >= 1.0.1
    * RHEL/Fedora: openssl-devel
    * SUSE: libopenssl-devel
    * Debian/Ubuntu: libssl-dev
    * Alpine: libressl-dev
* Boost library and header files >= 1.66.0
    * RHEL/Fedora: boost166-devel
    * Debian/Ubuntu: libboost-all-dev
    * Alpine: boost-dev
* GNU bison (bison)
* GNU flex (flex) >= 2.5.35
* systemd headers
    * Only required when using systemd
    * Debian/Ubuntu: libsystemd-dev
    * RHEL/Fedora: systemd-devel

### Optional features <a id="development-package-builds-optional-features"></a>

* MySQL (disable with CMake variable `ICINGA2_WITH_MYSQL` to `OFF`)
    * RHEL/Fedora: mysql-devel
    * SUSE: libmysqlclient-devel
    * Debian/Ubuntu: default-libmysqlclient-dev | libmysqlclient-dev
    * Alpine: mariadb-dev
* PostgreSQL (disable with CMake variable `ICINGA2_WITH_PGSQL` to `OFF`)
    * RHEL/Fedora: postgresql-devel
    * Debian/Ubuntu: libpq-dev
    * postgresql-dev on Alpine
* libedit (CLI console)
    * RHEL/Fedora: libedit-devel on CentOS (RHEL requires rhel-7-server-optional-rpms)
    * Debian/Ubuntu/Alpine: libedit-dev
* Termcap (only required if libedit doesn't already link against termcap/ncurses)
    * RHEL/Fedora: libtermcap-devel
    * Debian/Ubuntu: (not necessary)

### Special requirements <a id="development-package-builds-special-requirements"></a>

**FreeBSD**: libexecinfo (automatically used when Icinga 2 is installed via port or package)

**RHEL6**: Requires a newer boost version which is available on packages.icinga.com
with a version suffixed name.

### Runtime user environment <a id="development-package-builds-runtime-user-env"></a>

By default Icinga will run as user `icinga` and group `icinga`. Additionally the
external command pipe and livestatus features require a dedicated command group
`icingacmd`. You can choose your own user/group names and pass them to CMake
using the `ICINGA2_USER`, `ICINGA2_GROUP` and `ICINGA2_COMMAND_GROUP` variables.

```bash
groupadd icinga
groupadd icingacmd
useradd -c "icinga" -s /sbin/nologin -G icingacmd -g icinga icinga
```

On Alpine (which uses ash busybox) you can run:

```bash
addgroup -S icinga
addgroup -S icingacmd
adduser -S -D -H -h /var/spool/icinga2 -s /sbin/nologin -G icinga -g icinga icinga
adduser icinga icingacmd
```

Add the web server user to the icingacmd group in order to grant it write
permissions to the external command pipe and livestatus socket:

```bash
usermod -a -G icingacmd www-data
```

Make sure to replace "www-data" with the name of the user your web server
is running as.

### Building Icinga 2: Example <a id="development-package-builds-example"></a>

Once you have installed all the necessary build requirements you can build
Icinga 2 using the following commands:

```bash
mkdir release && cd release
cmake ..
cd ..
make -C release
make install -C release
```

You can specify an alternative installation prefix using `-DCMAKE_INSTALL_PREFIX`:

```bash
cmake .. -DCMAKE_INSTALL_PREFIX=/tmp/icinga2
```

### CMake Variables <a id="development-package-builds-cmake-variables"></a>

In addition to `CMAKE_INSTALL_PREFIX` here are most of the supported Icinga-specific cmake variables.

For all variables regarding defaults paths on in CMake, see
[GNUInstallDirs](https://cmake.org/cmake/help/latest/module/GNUInstallDirs.html).

Also see `CMakeLists.txt` for details.

#### System Environment

* `CMAKE_INSTALL_SYSCONFDIR`: The configuration directory; defaults to `CMAKE_INSTALL_PREFIX/etc`
* `CMAKE_INSTALL_LOCALSTATEDIR`: The state directory; defaults to `CMAKE_INSTALL_PREFIX/var`
* `ICINGA2_CONFIGDIR`: Main config directory; defaults to `CMAKE_INSTALL_SYSCONFDIR/icinga2` usually `/etc/icinga2`
* `ICINGA2_CACHEDIR`: Directory for cache files; defaults to `CMAKE_INSTALL_LOCALSTATEDIR/cache/icinga2` usually `/var/cache/icinga2`
* `ICINGA2_DATADIR`: Data directory  for the daemon; defaults to `CMAKE_INSTALL_LOCALSTATEDIR/lib/icinga2` usually `/var/lib/icinga2`
* `ICINGA2_LOGDIR`: Logfiles of the daemon; defaults to `CMAKE_INSTALL_LOCALSTATEDIR/log/icinga2 usually `/var/log/icinga2`
* `ICINGA2_SPOOLDIR`: Spooling directory ; defaults to `CMAKE_INSTALL_LOCALSTATEDIR/spool/icinga2` usually `/var/spool/icinga2`
* `ICINGA2_INITRUNDIR`: Runtime data for the init system; defaults to `CMAKE_INSTALL_LOCALSTATEDIR/run/icinga2` usually `/run/icinga2`
* `ICINGA2_GIT_VERSION_INFO`: Whether to use Git to determine the version number; defaults to `ON`
* `ICINGA2_USER`: The user Icinga 2 should run as; defaults to `icinga`
* `ICINGA2_GROUP`: The group Icinga 2 should run as; defaults to `icinga`
* `ICINGA2_COMMAND_GROUP`: The command group Icinga 2 should use; defaults to `icingacmd`
* `ICINGA2_SYSCONFIGFILE`: Where to put the config file the initscript/systemd pulls it's dirs from;
* defaults to `CMAKE_INSTALL_PREFIX/etc/sysconfig/icinga2`
* `ICINGA2_PLUGINDIR`: The path for the Monitoring Plugins project binaries; defaults to `/usr/lib/nagios/plugins`

#### Build Optimization

* `ICINGA2_UNITY_BUILD`: Whether to perform a unity build; defaults to `ON`. Note: This requires additional memory and is not advised for building VMs, Docker for Mac and embedded hardware.
* `ICINGA2_LTO_BUILD`: Whether to use link time optimization (LTO); defaults to `OFF`

#### Init System

* `USE_SYSTEMD=ON|OFF`: Use systemd or a classic SysV initscript; defaults to `OFF`
* `INSTALL_SYSTEMD_SERVICE_AND_INITSCRIPT=ON|OFF` Force install both the systemd service definition file
  and the SysV initscript in parallel, regardless of how `USE_SYSTEMD` is set.
  Only use this for special packaging purposes and if you know what you are doing.
  Defaults to `OFF`.

#### Features

* `ICINGA2_WITH_CHECKER`: Determines whether the checker module is built; defaults to `ON`
* `ICINGA2_WITH_COMPAT`: Determines whether the compat module is built; defaults to `ON`
* `ICINGA2_WITH_LIVESTATUS`: Determines whether the Livestatus module is built; defaults to `ON`
* `ICINGA2_WITH_NOTIFICATION`: Determines whether the notification module is built; defaults to `ON`
* `ICINGA2_WITH_PERFDATA`: Determines whether the perfdata module is built; defaults to `ON`
* `ICINGA2_WITH_TESTS`: Determines whether the unit tests are built; defaults to `ON`

#### MySQL or MariaDB

The following settings can be tuned for the MySQL / MariaDB IDO feature.

* `ICINGA2_WITH_MYSQL`: Determines whether the MySQL IDO module is built; defaults to `ON`
* `MYSQL_CLIENT_LIBS`: Client implementation used (mysqlclient / mariadbclient); defaults searches for `mysqlclient` and `mariadbclient`
* `MYSQL_INCLUDE_DIR`: Directory containing include files for the mysqlclient; default empty -
  checking multiple paths like `/usr/include/mysql`

See [FindMySQL.cmake](https://github.com/Icinga/icinga2/blob/master/third-party/cmake/FindMySQL.cmake)
for implementation details.

#### PostgreSQL

The following settings can be tuned for the PostgreSQL IDO feature.

* `ICINGA2_WITH_PGSQL`: Determines whether the PostgreSQL IDO module is built; defaults to `ON`
* `PostgreSQL_INCLUDE_DIR`: Top-level directory containing the PostgreSQL include directories
* `PostgreSQL_LIBRARY`: File path to PostgreSQL library : libpq.so (or libpq.so.[ver] file)

See [FindPostgreSQL.cmake](https://github.com/Icinga/icinga2/blob/master/third-party/cmake/FindPostgreSQL.cmake)
for implementation details.

#### Version detection

CMake determines the Icinga 2 version number using `git describe` if the
source directory is contained in a Git repository. Otherwise the version number
is extracted from the [ICINGA2_VERSION](ICINGA2_VERSION) file. This behavior can be
overridden by creating a file called `icinga-version.h.force` in the source
directory. Alternatively the `-DICINGA2_GIT_VERSION_INFO=OFF` option for CMake
can be used to disable the usage of `git describe`.


### Building RPMs <a id="development-package-builds-rpms"></a>

#### Build Environment on RHEL, CentOS, Fedora, Amazon Linux

Setup your build environment:

```bash
yum -y install rpmdevtools
```

#### Build Environment on SuSE/SLES

SLES:

```bash
zypper addrepo http://download.opensuse.org/repositories/devel:tools/SLE_12_SP4/devel:tools.repo
zypper refresh
zypper install rpmdevtools spectool
```

OpenSuSE:

```bash
zypper addrepo http://download.opensuse.org/repositories/devel:tools/openSUSE_Leap_15.0/devel:tools.repo
zypper refresh
zypper install rpmdevtools spectool
```

#### Package Builds <a id="development-package-builds-rpms-package-builds"></a>

Prepare the rpmbuild directory tree:

```bash
cd $HOME
rpmdev-setuptree
```

Snapshot builds:

```bash
curl https://raw.githubusercontent.com/Icinga/rpm-icinga2/master/icinga2.spec -o $HOME/rpmbuild/SPECS/icinga2.spec
```

> **Note**
>
> The above command builds snapshot packages. Change to the `release` branch
> for release package builds.

Copy the tarball to `rpmbuild/SOURCES` e.g. by using the `spectool` binary
provided with `rpmdevtools`:

```bash
cd $HOME/rpmbuild/SOURCES
spectool -g ../SPECS/icinga2.spec

cd $HOME/rpmbuild
```

Install the build dependencies. Example for CentOS 7:

```bash
yum -y install libedit-devel ncurses-devel gcc-c++ libstdc++-devel openssl-devel \
cmake flex bison boost-devel systemd mysql-devel postgresql-devel httpd \
selinux-policy-devel checkpolicy selinux-policy selinux-policy-doc
```

Note: If you are using Amazon Linux, systemd is not required.

A shorter way is available using the `yum-builddep` command on RHEL based systems:

```bash
yum-builddep SPECS/icinga2.spec
```

Build the RPM:

```bash
rpmbuild -ba SPECS/icinga2.spec
```

#### Additional Hints <a id="development-package-builds-rpms-additional-hints"></a>

##### SELinux policy module

The following packages are required to build the SELinux policy module:

* checkpolicy
* selinux-policy (selinux-policy on CentOS 6, selinux-policy-devel on CentOS 7)
* selinux-policy-doc

##### RHEL/CentOS 6

The RedHat Developer Toolset is required for building Icinga 2 beforehand.
This contains a modern version of flex and a C++ compiler which supports
C++11 features.

```bash
cat >/etc/yum.repos.d/devtools-2.repo <<REPO
[testing-devtools-2-centos-\$releasever]
name=testing 2 devtools for CentOS $releasever
baseurl=https://people.centos.org/tru/devtools-2/\$releasever/\$basearch/RPMS
gpgcheck=0
REPO
```

Dependencies to devtools-2 are used in the RPM SPEC, so the correct tools
should be used for building.

As an alternative, you can use newer Boost packages provided on
[packages.icinga.com](https://packages.icinga.com/epel).

```bash
cat >$HOME/.rpmmacros <<MACROS
%build_icinga_org 1
MACROS
```

##### Amazon Linux

If you prefer to build packages offline, a suitable Vagrant box is located
[here](https://atlas.hashicorp.com/mvbcoding/boxes/awslinux/).

### Build Debian/Ubuntu packages <a id="development-package-builds-deb"></a>

Setup your build environment on Debian/Ubuntu, copy the 'debian' directory from
the Debian packaging Git repository (https://github.com/Icinga/deb-icinga2)
into your source tree and run the following command:

```bash
dpkg-buildpackage -uc -us
```

### Build Alpine Linux packages <a id="development-package-builds-alpine"></a>

A simple way to setup a build environment is installing Alpine in a chroot.
In this way, you can set up an Alpine build environment in a chroot under a
different Linux distro.
There is a script that simplifies these steps with just two commands, and
can be found [here](https://github.com/alpinelinux/alpine-chroot-install).

Once the build environment is installed, you can setup the system to build
the packages by following [this document](https://wiki.alpinelinux.org/wiki/Creating_an_Alpine_package).

### Build Post Install Tasks <a id="development-package-builds-post-install-tasks"></a>

After building Icinga 2 yourself, your package build system should at least run the following post
install requirements:

* enable the `checker`, `notification` and `mainlog` feature by default
* run 'icinga2 api setup' in order to enable the `api` feature and generate TLS certificates for the node

### Run Icinga 2 <a id="development-package-builds-run-icinga"></a>

Icinga 2 comes with a binary that takes care of loading all the relevant
components (e.g. for check execution, notifications, etc.):

```
icinga2 daemon

[2016-12-08 16:44:24 +0100] information/cli: Icinga application loader (version: v2.5.4-231-gb10a6b7; debug)
[2016-12-08 16:44:24 +0100] information/cli: Loading configuration file(s).
[2016-12-08 16:44:25 +0100] information/ConfigItem: Committing config item(s).
...
```

#### Init Script <a id="development-package-builds-init-script"></a>

Icinga 2 can be started as a daemon using the provided init script:

```
/etc/init.d/icinga2
Usage: /etc/init.d/icinga2 {start|stop|restart|reload|checkconfig|status}
```

#### Systemd <a id="development-package-builds-systemd"></a>

If your distribution uses systemd:

```
systemctl {start|stop|reload|status|enable|disable} icinga2
```

In case the distribution is running systemd >227, you'll also
need to package and install the `etc/initsystem/icinga2.service.limits.conf`
file into `/etc/systemd/system/icinga2.service.d`.

#### openrc <a id="development-package-builds-openrc"></a>

Or if your distribution uses openrc (like Alpine):

```
rc-service icinga2
Usage: /etc/init.d/icinga2 {start|stop|restart|reload|checkconfig|status}
```

Note: the openrc's init.d is not shipped by default.
A working init.d with openrc can be found here: (https://git.alpinelinux.org/cgit/aports/plain/community/icinga2/icinga2.initd). If you have customized some path, edit the file and adjust it according with your setup.
Those few steps can be followed:

```bash
wget https://git.alpinelinux.org/cgit/aports/plain/community/icinga2/icinga2.initd
mv icinga2.initd /etc/init.d/icinga2
chmod +x /etc/init.d/icinga2
```

Icinga 2 reads a single configuration file which is used to specify all
configuration settings (global settings, hosts, services, etc.). The
configuration format is explained in detail in the [doc/](doc/) directory.

By default `make install` installs example configuration files in
`/usr/local/etc/icinga2` unless you have specified a different prefix or
sysconfdir.


### Windows Builds <a id="development-package-builds-windows"></a>

The Windows MSI packages are located at https://packages.icinga.com/windows/

The build infrastructure is based on GitLab CI and an Ansible provisioned
Windows VM running in OpenStack.

The runner uses the scripts located in `tools/win32` to configure, build
and test the packages. Uploading them to the package repository is a
separate step. For manual package creation, please refer to [this chapter](21-development.md#development-windows-dev-env).

![Windows build pipeline in GitLab](images/development/windows_builds_gitlab_pipeline.png)
