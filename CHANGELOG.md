##2015-08-07 - Supported Release 1.1.0
###Summary

A small feature release as well as a few minor fixes. Most of the new
work simply makes existing types more configurable, with the exception
of the new `docker::registry` type.

####Features
- Support for configuring docker private registries
- Repository options can be specified for RHEL package provider using `repo_opt`
- EPEL can now be disabled, for instance if you have your own Docker packages
- Environment variables can be provided in a file via `env_file` for `docker::run`
- Added the ability to run a command before a container is stopped using
  the `before_stop` parameter

#### Bugfixes
- Resolve issue enabling selinux on RHEL7 under systemd
- Change Docker repository from get.docker.io to get.docker.com


##2015-05-28 - Supported Release 1.0.1
###Summary

This release includes a few updates to the README file, as well updates to the metadata file formatting.

##2015-04-27 - Supported Release 1.0.0
###Summary

The is the initial supported release of the puppetlabs-docker_platform module which is used to install, configure and manage the docker daemon, docker images and docker containers.

####Features
- Support for Ubuntu 14.04, Red Hat Enterprise Linux 7.1 and CentOS 7.1
- Docker daemon installation and configuration
- Docker image download and management
- Docker container configuration and management
