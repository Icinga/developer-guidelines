Icinga uses the integrated CI capabilities on GitHub in the development workflow.
This ensures that incoming pull requests and branches are built on create/push events.
Contributors and developers can immediately see whether builds fail or succeed and
help the final reviews.

* For Linux, we are currently using Travis CI.
* For Windows, AppVeyor has been integrated.

Future plans involve making use of GitHub Actions.

In addition to our development platform on GitHub,
we are using GitLab's CI platform to build binary packages for
all supported operating systems and distributions.
These CI pipelines provide even more detailed insights into
specific platform failures and developers can react faster.

### CI: Travis CI

[Travis CI](https://travis-ci.org/Icinga/icinga2) provides Ubuntu as base
distribution where Icinga is compiled from sources followed by running the
unit tests and a config validation check.

For details, please refer to the [.travis.yml](https://github.com/Icinga/icinga2/blob/master/.travis.yml)
configuration file.

### CI: AppVeyor

[AppVeyor](https://ci.appveyor.com/project/icinga/icinga2) provides Windows
as platform where Visual Studio and Boost libraries come pre-installed.

Icinga is built using the Powershell scripts located in `tools/win32`.
In addition to that, the unit tests are run.

Please check the [appveyor.yml](https://github.com/Icinga/icinga2/blob/master/appveyor.yml) configuration
file for details.
