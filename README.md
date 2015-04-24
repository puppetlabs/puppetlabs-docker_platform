#docker_platform

####Table of Contents

1. [Overview](#overview)
2. [Module Description - What the module does and why it is useful](#module-description)
3. [Setup - The basics of getting started with docker_platform](#setup)
    * [Setup requirements](#setup-requirements)
    * [Beginning with docker_platform](#beginning-with-docker-platform])
4. [Usage - Configuration options and additional functionality](#usage)
5. [Reference - An under-the-hood peek at what the module is doing and how](#reference)
5. [Limitations - OS compatibility, etc.](#limitations)
  * [Support](#support)
  * [Known Issues](#known-issues)
6. [Development - Guide for contributing to the module](#development)

##Overview 

The Puppet docker_platform module installs, configures, and manages the [Docker](https://github.com/dotcloud/docker) daemon and Docker containers.

##Description

This module allows the implementation of the Docker container system across all Puppet-managed infrastructure. It allows the installation of the Docker daemon, as well as management of images and containers accross different nodesets. Additionally, commands running inside already running containers can be run if required.

##Setup

###Setup requirements

For RHEL and CentOS systems, a few issues might prevent Docker from starting properly. You can learn about these issues in the [Known Issues](#known-issues) section below.

###Beginning with docker_platform

To install Docker on a node, include the class `docker`.

~~~
include 'docker'
~~~

By default, for CentOS/RHEL, this installs Docker from your operating system's repo. For Ubuntu, this installs Docker from the upstream Docker repository.

## Usage

###Installing Docker

You can install Docker with various parameters specified for the [`docker`](#docker) class:

~~~
class {'docker':
  tcp_bind    => 'tcp://127.0.0.1:4243',
  socket_bind => 'unix:///var/run/docker.sock',
  version => '0.5.5',
  dns => '8.8.8.8',
  docker_users => [ 'user1', 'user2' ],
}
~~~

This example installs Docker version 0.5.5, binds the Docker daemon to a Unix socket and a tcp socket, provides the daemon with a dns server, and adds a set of two users to the Docker group.

### Images

To install a Docker image, use the define [`docker::image`](#dockerimage):

~~~
docker::image { 'base': }
~~~

This is equivalent to running `docker pull base`. This downloads a large binary, so on first run, it can take a while. For that reason, this define turns off the default five-minute timeout for exec. 

~~~
docker::image { 'ubuntu':
  ensure => 'present'
  image_tag => 'precise',
  docker_file => '/tmp/Dockerfile',
}
~~~

The above code adds an image from the listed Dockerfile. Alternatively, you can specify an image from a Docker directory, by using `docker_dir` parameter instead of `docker_file`:

### Containers

Now that you have an image, you can run commands within a container managed by Docker:

~~~
docker::run { 'helloworld':
  image   => 'base',
  command => '/bin/sh -c "while true; do echo hello world; sleep 1; done"',
}
~~~

You can set ports, expose, env, dns, and volumes either a single string or as above with an array of values.

Specifying `pull_on_start` pulls the image before each time it is started.

The `depends` option allows expressing containers that must be started before. This affects the generation of the init.d/systemd script.

The service file created for systemd and upstart based systems enables automatic restarting of the service on failure by default.

To use an image tag just append the tag name to the image name separated by a semicolon:

~~~
docker::run { 'helloworld':
  image   => 'ubuntu:precise',
  command => '/bin/sh -c "while true; do echo hello world; sleep 1; done"',
}
~~~

If using Hiera, there's a `docker::run_instance` class you can configure, for example:

~~~
docker::run_instance:
  helloworld:
    image: 'ubuntu:precise'
    command: '/bin/sh -c "while true; do echo hello world; sleep 1; done"'
~~~

### Exec

You can also run arbitrary comments within the context of a running container:

~~~
docker::exec { 'helloworld-uptime':
  detach    => true,
  container => 'helloworld',
  command   => 'uptime',
  tty       => true,
}
~~~

### Full Basic Example

To install Docker, download a Ubuntu image, and run a Ubuntu-based container that does nothing except
run the init process, you can use the following example manifest:

~~~
class { 'docker':}

docker::image { 'ubuntu':
  require => Class['docker'],
}

docker::run { 'test_1':
  image   => 'ubuntu',
  command => 'init', 
  require => Docker::Image['ubuntu'],
}
~~~

## Advanced Community Examples

* [Launch vNext app in Docker using Puppet](https://github.com/garethr/puppet-docker-vnext-example)

This example contains a fairly simple example using Vagrant to launch a Linux virtual machine, then Puppet to install Docker, build an image and run a container. For added spice, the container runs a ASP.NET vNext application.

* [Multihost containers connected with Consul](https://github.com/garethr/puppet-docker-example)

Launch multiple connect them together using Nginx updated by Consul and Puppet.

* [Configure Docker Swarm using Puppet](https://github.com/garethr/puppet-docker-swarm-example)

Build a cluster of hosts running Docker Swarm configured by Puppet.

##Reference

###Classes

####docker

Installs, configures, and manages your Docker installation.

#####`use_upstream_package_source`

Optional; applies to Ubuntu only. Whether the upstream package source should be used for installation. Valid values are 'true', 'false'. Defaults to 'true'.

#####`tcp_bind`

Optional. Specify the tcp socket the Docker daemon should bind to.

#####`socket_bind`

Optional. Specify the Unix socket the Docker daemon should bind to. By default the Docker daemon will bind to a Unix socket at `/var/run/docker.sock`.

#####`version`

Optional. Specify a version of Docker to use. Accepts a version number or 'latest'. Default is'0.5.5'.

#####`dns`

Specify a dns server for the Docker daemon to connect to. This is useful if DNS resolution isn't working properly in the container. 

#####`docker_users` => [ 'user1', 'user2' ],

Add users to the Docker group. Accepts an array like:

~~~
class { 'docker':
  docker_users => [ 'user1', 'user2' ],
}
~~~

#####`ensure`
Passed to the Docker package. Defaults to present.

#####`prerequired_packages`

An array of additional packages that need to be installed to support Docker. Defaults change depending on the operating system.

#####`tcp_bind`

The tcp socket to bind to in the format 'tcp://127.0.0.1:4243' Defaults to undefined.

#####`socket_bind`

The unix socket to bind to. Defaults to 'unix:///var/run/docker.sock'.

#####`log_level`

Set the logging level. Defaults to undef: Docker defaults to 'info' if no value specified. Valid values: 'debug', 'info', 'warn', 'error', 'fatal'.

#####`selinux_enabled`

Enable selinux support. Default is false. SELinux does not  presently support the BTRFS storage driver. Valid values: 'true', 'false'.

#####`use_upstream_package_source` 

Whether or not to use the upstream package source. If you run your own package mirror, you may set this to 'false'.

#####`package_source_location`

The location of your upstream package source, if you're using one. Defaults to 'https://get.docker.io/ubuntu' on Debian.

#####`service_state`

Whether you want to the Docker daemon to start up. Valid values: 'running', 'stopped'. Defaults to 'running'.

#####`service_enable`

Whether you want to Docker daemon to start up at boot. Valid values: 'true', 'false'. Defaults to 'true'.

#####`root_dir`

Custom root directory for containers. Valid values: a directory path. Defaults to undefined.

#####`manage_kernel`

Attempt to install the correct Kernel required by Docker. Valid values: 'true', 'false'. Defaults to 'true'.

#####`dns`

Custom dns server address. Defaults to undefined.

#####`dns_search`

Custom dns search domains. Defaults to undefined.

#####`socket_group`

Group ownership of the Unix control socket. Accepts a Unix user group. Defaults to undefined.

#####`extra_parameters`

Any extra parameters that should be passed to the Docker daemon. Defaults to undefined.

#####`shell_values`

Array of shell values to pass into init script config files.

#####`proxy`

Sets the http_proxy and https_proxy env variables in `/etc/sysconfig/docker` (Red Hat/CentOS) or `/etc/default/docker` (Debian).

#####`no_proxy`

Sets the no_proxy variable in `/etc/sysconfig/docker` (Red Hat/CentOS) or `/etc/default/docker` (Debian).

#####`storage_driver`

Specify a storage driver to use. Default is undef: this lets Docker choose the correct driver. Valid values: 'aufs', 'devicemapper', 'btrfs', 'overlayfs', 'vfs'.

#####`dm_basesize`

The size to use when creating the base device, which limits the size of images and containers. Accepts a size such as 5G, 10G, 11G. Default value is 10G.

#####`dm_fs`

The filesystem to use for the base image. Valid values: 'xfs' or 'ext4'. Defaults to 'ext4'.

#####`dm_mkfsarg`

Specifies extra mkfs arguments to be used when creating the base device.

#####`dm_mountopt`

Specifies extra mount options used when mounting the thin devices.

#####`dm_blocksize`

A custom blocksize to use for the thin pool. Default blocksize is 64K.

Warning: **DO NOT** change this parameter after the lvm devices have been initialized.

#####`dm_loopdatasize`

Specifies the size to use when creating the loopback file for the "data" device which is used for the thin pool. Default size is 100G.

#####`dm_loopmetadatasize`

Specifies the size to use when creating the loopback file for the "metadata" device which is used for the thin pool. Default size is 2G.

#####`dm_datadev`

A custom blockdevice to use for data for the thin pool.

#####`dm_metadatadev`

A custom blockdevice to use for metadata for the thin pool.

#####`manage_package`

Whether to manage the package. If set to 'false',the module doesn't define or install the package. Useful if you want to use your own package. Valid values: 'true', 'false'. Defaults to 'true'.

#####`package_name`

Specify custom package name. Default is set on a per system basis.

#####`service_name`

Specify custom service name. Default is set on a per system basis in `docker::params`.

#####`docker_command`

Specify a custom Docker command name. Default is set on a per system basis.

#####`docker_users`

Specifies an array of users to add to the Docker group. Default is empty.

####`docker::images`

This class is for using Hiera for image management. You can use the same parameters as the `docker::image` define, and those values will be passed through from Hiera into the `docker::image` define.

####`docker::run_instance`

This class is for using Hiera for instance management. You can use the same parameters as the `docker::run` define, and those values will be passed through from Hiera into the `docker::image` define.


###Defines

####docker::image

#####`base`

This is equivalent to running `docker pull base`. This is downloading a large binary so on first run can take a while. For that reason this define turns off the default 5 minute timeout for exec. Takes an optional parameter for installing image tags that is the equivalent to running `docker pull -t="precise" ubuntu`:

#####`image_tag`

Optional. Installs image tags. Equivalent to running `docker pull -t="precise" ubuntu`.

Note: images will only install if an image of that name does not already exist.

#####`docker_file`
Optional. Add images from a Dockerfile. Cannot be used with the `docker_dir` parameter. Accepts a path to the Dockerfile.

#####`docker_dir`

Add images from a directory containing a Dockerfile with the `docker_dir` property. Cannot be used with the `docker_file` parameter. Accepts a path to the images.

#####`ensure`

Specify whether the image should be present or absent. Valid values are 'present', 'absent'. Default is 'present'.

####docker::run
  
#####`volumes_from`

Used to mount a volume from another container on the current container.

#####`memory_limit`

Specifies the memory limit for the container.

#####`cpuset`

Binds the containers to a specific CPU core on the host.

#####`username`

Runs the Docker container as a paticular user on the host system.

#####`env`

Allows the setting of environment variables within the container.
  
#####`dns`

Sets a custom DNS server for the containers.
  
#####`restart_service`

If set to 'true', restarts the containers if any other properties change.
  
#####`privileged`

If set to 'true', the containers become a Docker "privileged container". [See Docker documentation for further details.](TODO: link to relevant docs?)
  
#####`pull_on_start`

This pulls a fresh copy of the image from the upstream repo every time the container is started. Valid values: 'true', 'false'. Default is 'TODO'.

#####`depends`

Allows specifying containers that must be started first. This affects the generation of the init.d/systemd script. Valid values: 'true', 'false'. Defaults to 'false'.

#####`image`

The Docker image to base this container off. Can also include a specific tag (e.g., `ubuntu:latest`).

#####`command`

The command to run inside the container once it has been started.

#####`ports`

Can be used to bind a port or ports inside a container to a different port on the host. This can be used to access a service running inside a container outside of Docker.

#####`expose`

The port or ports specified will be available for incoming connections on the respective container from other Docker containers.

#####`hostname`

This property will set the hostname inside the respective container.

#####`volumes`

This property allows you to mount folders on your host system inside the respective container.

#####`links`

This property allows you to create Docker links between multiple containers.

#####`running`

If set to false, this will ensure that the respective container is not running. If set to true, this will ensure that the container is running.
  
####docker::exec

#####`detach`

If set to true, this will run the command in the background

#####`container`

The name of the container to run the specified command in

#####`command`

The command to run inside the container

#####`tty`

Allocate a pseudo-TTY

## Limitations

### Support

This module is currently supported on:

* Ubuntu 14.04
* Red Hat Enterprise Linux 7.1
* CentOS 7.1

### Known Issues

Depending on the initial state of your OS, you may run into issues which may mean that Docker fails to start properly.

#### RHEL7 and CentOS 7

RHEL7/CentOS requires at least version 1.02.93 of the device-mapper package to be installed in order for Docker's default configuration to work. This is only available on RHEL/CentOS 7.1+.

You can install this package via Puppet using the following manifest:

~~~
package {'device-mapper':
  ensure => latest,
}
~~~

Remember to add the appropriate metaparameters (`before` or `require`) for your environment to ensure that device-mapper is installed before the `docker` class is executed.

RHEL7 also has the above issue. 

Additionally, RHEL7 needs the rhel7-extras repo enabled in order to install Docker. You can install Docker using the following example manifest:

~~~
package {'docker_rhel':
  name		        => 'docker',
  install_options => ['--enablerepo=rhel7-extras'],
  ensure          => latest,
}
~~~

Again, remember to add the appropriate metaparameters (before or require) for your environment to ensure that device-mapper is installed before the Docker package is installed.

##Development

Puppet Labs modules on the Puppet Forge are open projects, and community contributions are essential for keeping them great. We can't access the huge number of platforms and myriad of hardware, software, and deployment configurations that Puppet is intended to serve.

We want to keep it as easy as possible to contribute changes so that our modules work in your environment. There are a few guidelines that we need contributors to follow so that we can have a chance of keeping on top of things.

You can read the complete module contribution guide [on the Puppet Labs wiki.](http://projects.puppetlabs.com/projects/module-site/wiki/Module_contributing)