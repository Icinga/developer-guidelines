### Snapshot Packages (Nightly Builds) <a id="development-tests-snapshot-packages"></a>

Icinga provides snapshot packages as nightly builds from [Git master](https://github.com/icinga/icinga2).

These packages contain development code which should be considered "work in progress".
While developers ensure that tests are running fine with CI actions on PRs,
things might break, or changes are not yet documented in the changelog.

You can help the developers and test the snapshot packages, e.g. when larger
changes or rewrites are taking place for a new major version. Your feedback
is very much appreciated.

Snapshot packages are available for all supported platforms including
Linux and Windows and can be obtained from [https://packages.icinga.com](https://packages.icinga.com).

The [Vagrant boxes](https://github.com/Icinga/icinga-vagrant) also use
the Icinga snapshot packages to allow easier integration tests. It is also
possible to use Docker with base OS images and installing the snapshot
packages.

If you encounter a problem, please [open a new issue](https://github.com/Icinga/icinga2/issues/new/choose)
on GitHub and mention that you're testing the snapshot packages.

#### RHEL/CentOS <a id="development-tests-snapshot-packages-rhel"></a>

2.11+ requires the [EPEL repository](02-installation.md#package-repositories-rhel-epel) for Boost 1.66+.

In addition to that, the `icinga-rpm-release` package already provides the `icinga-snapshot-builds`
repository but it is disabled by default.

```bash
yum -y install https://packages.icinga.com/epel/icinga-rpm-release-7-latest.noarch.rpm
yum -y install epel-release
yum makecache

yum install --enablerepo=icinga-snapshot-builds icinga2
```

#### Debian <a id="development-tests-snapshot-packages-debian"></a>

2.11+ requires Boost 1.66+ which either is provided by the OS, backports or Icinga stable repositories.
It is advised to configure both Icinga repositories, stable and snapshot and selectively
choose the repository with the `-t` flag on `apt-get install`.

```bash
apt-get update
apt-get -y install apt-transport-https wget gnupg

wget -O - https://packages.icinga.com/icinga.key | apt-key add -

DIST=$(awk -F"[)(]+" '/VERSION=/ {print $2}' /etc/os-release); \
 echo "deb https://packages.icinga.com/debian icinga-${DIST} main" > \
 /etc/apt/sources.list.d/${DIST}-icinga.list
 echo "deb-src https://packages.icinga.com/debian icinga-${DIST} main" >> \
 /etc/apt/sources.list.d/${DIST}-icinga.list

DIST=$(awk -F"[)(]+" '/VERSION=/ {print $2}' /etc/os-release); \
 echo "deb http://packages.icinga.com/debian icinga-${DIST}-snapshots main" > \
 /etc/apt/sources.list.d/${DIST}-icinga-snapshots.list
 echo "deb-src http://packages.icinga.com/debian icinga-${DIST}-snapshots main" >> \
 /etc/apt/sources.list.d/${DIST}-icinga-snapshots.list

apt-get update
```

On Debian Stretch, you'll also need to add Debian Backports.

```bash
DIST=$(awk -F"[)(]+" '/VERSION=/ {print $2}' /etc/os-release); \
 echo "deb https://deb.debian.org/debian ${DIST}-backports main" > \
 /etc/apt/sources.list.d/${DIST}-backports.list

apt-get update
```

Then install the snapshot packages.

```bash
DIST=$(awk -F"[)(]+" '/VERSION=/ {print $2}' /etc/os-release); \
apt-get install -t icinga-${DIST}-snapshots icinga2
```

#### Ubuntu <a id="development-tests-snapshot-packages-ubuntu"></a>

```bash
apt-get update
apt-get -y install apt-transport-https wget gnupg

wget -O - https://packages.icinga.com/icinga.key | apt-key add -

. /etc/os-release; if [ ! -z ${UBUNTU_CODENAME+x} ]; then DIST="${UBUNTU_CODENAME}"; else DIST="$(lsb_release -c| awk '{print $2}')"; fi; \
 echo "deb https://packages.icinga.com/ubuntu icinga-${DIST} main" > \
 /etc/apt/sources.list.d/${DIST}-icinga.list
 echo "deb-src https://packages.icinga.com/ubuntu icinga-${DIST} main" >> \
 /etc/apt/sources.list.d/${DIST}-icinga.list

. /etc/os-release; if [ ! -z ${UBUNTU_CODENAME+x} ]; then DIST="${UBUNTU_CODENAME}"; else DIST="$(lsb_release -c| awk '{print $2}')"; fi; \
 echo "deb https://packages.icinga.com/ubuntu icinga-${DIST}-snapshots main" > \
 /etc/apt/sources.list.d/${DIST}-icinga-snapshots.list
 echo "deb-src https://packages.icinga.com/ubuntu icinga-${DIST}-snapshots main" >> \
 /etc/apt/sources.list.d/${DIST}-icinga-snapshots.list

apt-get update
```

Then install the snapshot packages.

```bash
. /etc/os-release; if [ ! -z ${UBUNTU_CODENAME+x} ]; then DIST="${UBUNTU_CODENAME}"; else DIST="$(lsb_release -c| awk '{print $2}')"; fi; \
apt-get install -t icinga-${DIST}-snapshots icinga2
```

#### SLES <a id="development-tests-snapshot-packages-sles"></a>

The required Boost packages are provided with the stable release repository.

```bash
rpm --import https://packages.icinga.com/icinga.key

zypper ar https://packages.icinga.com/SUSE/ICINGA-release.repo
zypper ref

zypper ar https://packages.icinga.com/SUSE/ICINGA-snapshot.repo
zypper ref
```

Selectively install the snapshot packages using the `-r` parameter.

```bash
zypper in -r icinga-snapshot-builds icinga2
```


### Unit Tests <a id="development-tests-unit"></a>

Build the binaries and run the tests.


```bash
make -j4 -C debug
make test -C debug
```

Run a specific boost test:

```bash
debug/Bin/Debug/boosttest-test-base --run_test=remote_url
```
