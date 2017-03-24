## 2016-04-12 - Supported Release 2.2.0

Several minor improvements to the Docker Compose support including:

* Support for v2 of the Compose file syntax
* Support refreshing the docker_compose resource
* The ability to pass an install_path for custom installations
* Passing arguments to docker-compose up
* Ensuring curl is available when using it to install Compose

New parameters for docker::run including stop_wait_time to allow
containers time to stop when killed

New parameters for the docker class, including icc, storage_setup_file

Support for the overlay2 storage driver and the splunk log driver.

Improvements to management when not using the upstream repository,
including doing less to manage potentially unneeded dependencies.

Support multiple registry authentications on the same host.

Fix an issue with using docker::run on Swarm.

Fix a number of issues if running the module with strict variables
enabled, and add this to the tested conbinations.

## 2016-04-12 - Supported Release 2.1.0

Note that changes in Docker 1.10 changed the flag used to start the
docker daemon. If you are using a version of docker prior to 1.8 you
will need to pass the docker_subcommand parameter with the '-d' option.

This release includes a few minor bug-fixes along with several new
features:

* The module now allows for installing, and running, Docker Compose from
  Puppet, using both the docker::compose class the the docker_compose
  type.
* The module also now allows for the creation and management of Docker
  Network using the new docker_network type
* And the docker::run type now supports ensure => absent
* Lots of options to configure the docker deaemon network
* Support for installing Docker CS, the commercially supported Docker
  engine
* Disable managing the docker service in Puppet

Fixes include:

* Ensuring idempotence of docker::run using deprecated params
* Properly escaping variables in unless for docker::exec
* Explicitly specify systemd as the service provider for recent versions
  of Ubuntu and Debian
* Fix issue with Amazon Linux support

## 2015-12-18 - Supported Release 2.0.0

Note that this is a major release and in particular changes the default
repository behaviour so all supported operating systems use the new
Docker upstream repos.

This release includes:

- Full docker label support
- Support for CentOS 7 repository options
- Support for Docker's built-in restart policy
- Docker storage setup options support for systemd
- The ability to configure log drivers
- Support unless for docker exec
- Full datamapper property support, and deprecation of old property
  names
- Allow arbitrary parameters to be passed to systemd
- Add ZFS storage driver support
- Allow docker image resources to be refreshed, pulling the latest
- Deprecates use_name, all containers are now named for the resource
- Support for Puppet 4.3 with the stricter parser


As well as fixes for:

- Fix running=false to not start the docker image on docker restart
  under systemd
- Fixed README to document correct Valid Options for `storage_driver`
- Prevent timeouts for docker run
- Ensure docker is running before attempting to use docker run
- Obsfucate registry password from Puppet logs

## 2015-08-07 - Supported Release 1.1.0
### Summary

A small feature release as well as a few minor fixes. Most of the new
work simply makes existing types more configurable, with the exception
of the new `docker::registry` type.

#### Features
- Support for configuring docker private registries
- Repository options can be specified for RHEL package provider using `repo_opt`
- EPEL can now be disabled, for instance if you have your own Docker packages
- Environment variables can be provided in a file via `env_file` for `docker::run`
- Added the ability to run a command before a container is stopped using
  the `before_stop` parameter

#### Bugfixes
- Resolve issue enabling selinux on RHEL7 under systemd
- Change Docker repository from get.docker.io to get.docker.com

## 2015-07-28 - Supported Release 1.0.2
### Summary

This release includes official support for Puppet 4.x and Puppet Enterprise 2015.2.x

## 2015-05-28 - Supported Release 1.0.1
### Summary

This release includes a few updates to the README file, as well updates to the metadata file formatting.

## 2015-04-27 - Supported Release 1.0.0
### Summary

The is the initial supported release of the puppetlabs-docker_platform module which is used to install, configure and manage the docker daemon, docker images and docker containers.

#### Features
- Support for Ubuntu 14.04/16.04, Red Hat Enterprise Linux 7.1 and CentOS 7.1
- Docker daemon installation and configuration
- Docker image download and management
- Docker container configuration and management
