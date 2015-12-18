##2015-12-18 - Supported Release 2.0

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
- Prevent timeouts for docker run
- Ensure docker is running before attempting to use docker run
- Obsfucate registry password from Puppet logs


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

##2015-07-28 - Supported Release 1.0.2
###Summary

This release includes official support for Puppet 4.x and Puppet Enterprise 2015.2.x

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

