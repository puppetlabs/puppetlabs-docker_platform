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

This module lets you use Puppet to implement the Docker container system across a Puppet-managed infrastructure. It includes classes and defines to install the Docker daemon, manage images and containers across different nodesets, and run commands inside containers.

##Setup

###Setup requirements

For Enterprise Linux 7 systems, a few issues might prevent Docker from starting properly. You can learn about these issues in the [Known Issues](#known-issues) section below.

###Beginning with docker_platform

To install Docker on a node, include the class `docker`.

~~~
include 'docker'
~~~

This installs [Docker](https://github.com/docker/docker) from the [official
repository](http://docs.docker.com/installation/) or alternatively from
[EPEL on RedHat](http://docs.docker.io/en/latest/installation/rhel/)
based distributions.

## Usage

###Installing Docker

You can install Docker with various parameters specified for the [`docker`](#docker) class:

~~~
class {'docker':
  tcp_bind     => 'tcp://127.0.0.1:4243',
  socket_bind  => 'unix:///var/run/docker.sock',
  version      => '0.5.5',
  dns          => '8.8.8.8',
  docker_users => [ 'user1', 'user2' ],
}
~~~

This example installs Docker version 0.5.5, binds the Docker daemon to a Unix socket and a tcp socket, provides the daemon with a dns server, and adds two users to the Docker group.

Docker recently [launched new official
repositories](http://blog.docker.com/2015/07/new-apt-and-yum-repos/#comment-247448)
which are now the default for the module from version 5. If you want to
stick with the old repositories you can do so with the following:

~~~
puppet
class { 'docker':
  package_name => 'lxc-docker',
  package_source_location => 'https://get.docker.com/ubuntu',
  package_key_source => 'https://get.docker.com/gpg',
  package_key => '36A1D7869245C8950F966E92D8576A8BA88D21E',
  package_release => 'docker',
}
~~~

The module also now uses the upstream repositories by default for RHEL
based distros, including Fedora. If you want to stick with the distro
packages you should use the following:

~~~
puppet
class { 'docker':
  use_upstream_package_source => false,
  package_name => 'docker',
}
~~~

### Images

To install a Docker image, use the define [`docker::image`](#dockerimage):

~~~
docker::image { 'base': }
~~~

This is equivalent to running `docker pull base`. This downloads a large binary, so on first run, it can take a while. For that reason, this define turns off the default five-minute timeout for exec.

~~~
docker::image { 'ubuntu':
  ensure      => 'present',
  image_tag   => 'precise',
  docker_file => '/tmp/Dockerfile',
}
~~~

The above code adds an image from the listed Dockerfile. Alternatively, you can specify an image from a Docker directory, by using `docker_dir` parameter instead of `docker_file`.

### Containers

Now that you have an image, you can run commands within a container managed by Docker:

~~~
docker::run { 'helloworld':
  image   => 'base',
  command => '/bin/sh -c "while true; do echo hello world; sleep 1; done"',
}
~~~

You can set ports, expose, env, dns, and volumes with either a single string or, as above, with an array of values.

Specifying `pull_on_start` pulls the image before each time it is started.

The `depends` option allows expressing containers that must be started before other containers start. This affects the generation of the init.d/systemd script.

The service file created for systemd and upstart based systems enables automatic restarting of the service on failure by default.

To use an image tag, append the tag name to the image name separated by a semicolon:

~~~
docker::run { 'helloworld':
  image   => 'ubuntu:precise',
  command => '/bin/sh -c "while true; do echo hello world; sleep 1; done"',
}
~~~

If using Hiera, there's a `docker::run_instance` class you can configure, for example:

~~~
docker::run_instance::instance:
  helloworld:
    image: 'ubuntu:precise'
    command: '/bin/sh -c "while true; do echo hello world; sleep 1; done"'
~~~

### Private registries

By default images will be pushed and pulled from [The Docker Hub](https://hub.docker.com/).
If you have your own private registry without authentication, you can fully qualify your image name.
If your private registry requires authentication you may configure a registry using the following:

~~~
docker::registry { 'example.docker.io:5000':
  username => 'user',
  password => 'secret',
  email    => 'user@example.com',
}
~~~

You can logout of a registry if it is no longer required.

~~~
docker::registry { 'example.docker.io:5000':
  ensure => 'absent',
}
~~~

If using Hiera, there's a `docker::registry_auth` class you can configure,
for example:

~~~
docker::registry_auth::registries:
  'example.com:5000':
    username: 'user1'
    password: 'secret'
    email: 'user1@example.io'
~~~

### Exec

You can also run arbitrary commands within the context of a running container:

~~~
docker::exec { 'helloworld-uptime':
  detach    => true,
  container => 'helloworld',
  command   => 'uptime',
  tty       => true,
}
~~~

### Full Basic Example

To install Docker, download a Ubuntu image, and run a Ubuntu-based container that does nothing except run the init process, you can use the following example manifest:

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

Launch multiple containers and connect them together using Nginx, updated by Consul and Puppet.

* [Configure Docker Swarm using Puppet](https://github.com/garethr/puppet-docker-swarm-example)

Build a cluster of hosts running Docker Swarm configured by Puppet.

##Reference

###Classes

* [`docker`](#class-docker)
* [`docker::images`](#class-dockerimages)
* [`docker::run_instance`](#class-dockerrun_instance)

###Defines

* [`docker::image`](#define-dockerimage)
* [`docker::run`](#define-dockerrun)
* [`docker::exec`](#define-dockerexec)

###Parameters

####Class: docker

Installs, configures, and manages your Docker installation. All parameters are optional unless otherwise specified.

#####`dm_basesize`

Specifies a size for the base device, which limits the size of images and containers. Valid options: a string containing a size (e.g., '5G', '10G', '11G'). Default: '10G'.

#####`dm_blocksize`

Specifies a custom blocksize for the thin pool. Valid options: a string containing a size (e.g., '5G', '10G', '11G'). Default: '64K'.

Warning: **DO NOT** change this parameter after the lvm devices have been initialized.

#####`dm_datadev`

**Deprecated** This parameter is no longer in use, `dm_thinpooldev` should be used instead.

#####`dm_fs`

Specifies a filesystem for the base image. Valid options: 'xfs' and 'ext4'. Default: 'ext4'.

#####`dm_loopdatasize`

Specifies the size of the loopback file for the "data" device used for the thin pool. Valid options: a string containing a size (e.g., '5G', '10G', '11G').Default: '2G'.

#####`dm_loopmetadatasize`

Specifies the size of the loopback file for the "metadata" device used for the thin pool. Valid options: a string containing a size (e.g., '5G', '10G', '11G'). Default: '2G'.

#####`dm_metadatadev`

**Deprecated** This parameter is no longer in use, `dm_thinpooldev` should be used instead.

#####`dm_mkfsarg`

Specifies extra mkfs arguments for creating the base device. Accepts a string.

#####`dm_mountopt`

Specifies extra mount options for mounting thin devices. Accepts a string.

#####`dm_thinpooldev`

Specifies a custom block storage device to use for the thin pool.

#####`dm_use_deferred_removal`

Enables use of deferred device removal if libdm and the kernel driver support the mechanism.

#####`dm_blkdiscard`

Enables or disables the use of blkdiscard when removing devicemapper devices. Defaults to false

#####`dm_override_udev_sync_check`

By default, the devicemapper backend attempts to synchronize with the udev device manager for the Linux kernel. This option allows disabling that synchronization, to continue even though the configuration may be buggy. Defaults to true.

#####`dns`

Specifies a DNS server for the Docker daemon to connect to. This is useful if DNS resolution isn't working properly in the container. Accepts a valid IP address. Default: undefined.

#####`dns_search`

Specifies one or more dns search domain for the Docker daemon to use. Accepts a string containing a domain. Default: undefined.

#####`docker_command`

Specifies a custom Docker command. Accepts a string. Default [varies by operating system](https://github.com/garethr/garethr-docker/blob/master/manifests/params.pp).

#####`docker_users`

Valid options: an array of usernames formatted as strings. For example:

~~~
class { 'docker':
  docker_users => [ 'user1', 'user2' ],
}
~~~

#####`ensure`

Specifies the desired state of the Docker package; passed to Puppet's native `package` type. Valid options: 'present', 'absent'. Default: 'present'.

#####`extra_parameters`

Passes one or more extra parameters to the Docker daemon. Accepts an array of strings. Default: undefined.

#####`log_level`

Sets the logging level. Valid options: 'debug', 'info', 'warn', 'error', and 'fatal'. Default: undef. If not specified, Docker uses 'info'.

#####`manage_epel`

Specified whether or not to include the EPEL repositories if they are
required to locate the docker package. Valid options: 'true' and 'false'. Default: 'true'.

#####`manage_package`

Specifies whether the docker module should manage the Docker package. If set to 'false', Puppet doesn't define or install the package. Useful if you want to use your own package. Valid options: 'true' and 'false'. Default: 'true'.

#####`no_proxy`

Passes a value to the no_proxy variable in `/etc/sysconfig/docker` (on RedHat and CentOS) or `/etc/default/docker` (on Debian). Accepts a string containing a network range or domain name.

#####`package_name`

Specifies a custom Docker package. Valid options: a string containing the name of the package to manage.

#####`package_source_location`

*Required, except on Ubuntu.* Specifies the location of your upstream package source, if you're using one. Accepts a URL. Default on Ubuntu: 'https://get.docker.io/ubuntu'.

#####`prerequired_packages`

An array of additional packages that need to be installed to support Docker. Accepts an array of one or more strings containing package names. Default: varies by operating system.

#####`proxy`

Passes a value to the http_proxy and https_proxy env variables in `/etc/sysconfig/docker` (on RedHat and CentOS) or `/etc/default/docker` (on Debian). Accepts a string containing the URL to a proxy server.

#####`root_dir`

Specifies a custom root directory for Docker containers. Valid options: a string containing the path to a directory. Default: undefined.

#####`selinux_enabled`

Specifies whether to enable selinux support. Note: SELinux does not  presently support the BTRFS storage driver. Valid options: 'true' and 'false'. Default: 'false'.

#####`service_state`

Specifies whether the Docker daemon should be running. Valid options: 'running' and 'stopped'. Default: 'running'. 

#####`service_enable`

Specifies whether the Docker daemon should start up at boot. Valid options: 'true' and 'false'. Default: 'true'.

#####`service_name`

Specifies a custom Docker service. Valid options: a string containing the name of the service to manage.

#####`shell_values`

Passes one or more shell values into init script config files. Accepts an array of strings.

#####`socket_bind`

Specifies the Unix socket the Docker daemon should bind to. Accepts a string containing the path to a Unix socket. Default: if not specified, Docker uses `/var/run/docker.sock`.

#####`socket_group`

Specifies the group ownership of the Unix control socket. Valid options: a string containing a Unix user group. Default: undefined.

#####`storage_driver`

Specifies a storage driver. Valid options: 'aufs', 'devicemapper', 'btrfs', 'overlay', and 'vfs'. Default: undef. If not specified, Docker chooses a driver automatically.

#####`tcp_bind`

Specifies the tcp socket the Docker daemon should bind to. Accepts a string containing a port number.

#####`use_upstream_package_source`

Specifies whether the upstream package source should be used for installation. If you run your own package mirror, you can set this to 'false'. Valid options: 'true' and 'false'. Default: 'true'.

#####`version`

Specify a version of Docker to use. Valid options: a string containing a version number or 'latest'. Default: '0.5.5'.

#####`log_driver`
Set the log driver. Defaults to undef. Docker default is json-file.
Valid values: none, json-file, syslog, journald, gelf, fluentd
Valid values description:

~~~
     none     : Disables any logging for the container.
                Docker logs won't be available with this driver.
     json-file: Default logging driver for Docker.
                Writes JSON messages to file.
     syslog   : Syslog logging driver for Docker.
                Writes log messages to syslog.
     journald : Journald logging driver for Docker.
                Writes log messages to journald.
     gelf     : Graylog Extended Log Format (GELF) logging driver for Docker.
                Writes log messages to a GELF endpoint: Graylog or Logstash.
     fluentd  : Fluentd logging driver for Docker.
                Writes log messages to fluentd (forward input).
~~~

#####`log_opt`
Set the log driver specific options. Defaults to undef. Valid values per log driver:

~~~
     none     : undef
     json-file: max-size=[0-9+][k|m|g]
                max-file=[0-9+]
     syslog   : syslog-address=[tcp|udp]://host:port
                syslog-address=unix://path
                syslog-facility=daemon|kern|user|mail|auth|
                                syslog|lpr|news|uucp|cron|
                                authpriv|ftp|
                                local0|local1|local2|local3|
                                local4|local5|local6|local7
                syslog-tag="some_tag"
     journald : undef
     gelf     : gelf-address=udp://host:port
                gelf-tag="some_tag"
     fluentd  : fluentd-address=host:port
                fluentd-tag={{.ID}} - short container id (12 characters)|
                            {{.FullID}} - full container id
                            {{.Name}} - container name
~~~

#####`storage_devs`
A quoted, space-separated list of devices to be used.

#####`storage_vg`
The volume group to use for docker storage.

#####`storage_root_size`
The size to which the root filesystem should be grown.

#####`storage_data_size`
The desired size for the docker data LV.

#####`storage_chunk_size`
Controls the chunk size/block size of thin pool.

#####`storage_growpart`
Enable resizing partition table backing root volume group.

#####`storage_auto_extend_pool`
Enable/disable automatic pool extension using lvm.

#####`storage_pool_autoextend_threshold`
Auto pool extension threshold (in % of pool size).

#####`storage_pool_autoextend_percent`
Extend the pool by specified percentage when threshold is hit.

####Class: docker::images

This class lets you use Hiera for image management. It accepts the same parameters as the [`docker::image`](#define-dockerimage) define, and passes their values from Hiera into the `docker::image` define.

####Class: docker::run_instance

This class lets you use Hiera for instance management. It accepts the same parameters as the [`docker::run`](#define-dockerrun) define, and passes their values from Hiera into the `docker::run` define.

####Define: docker::image

Parameters are optional, except where noted.

#####`base`

Pulls the base container image from the Docker Hub registry. This is equivalent to running `docker pull base`, which involves downloading a large binary. That can take awhile, so this define turns off the `exec` resource's default 5-minute timeout.

#####`docker_dir`

Adds images from a directory containing a Dockerfile with the `docker_dir` property. Cannot be used with the `docker_dir` parameter. Valid options: a string containing an absolute path to the directory.

#####`docker_file`

Adds images from a Dockerfile. Cannot be used with the `docker_dir` parameter. Valid options: a string containing an absolute path to the Dockerfile.

#####`ensure`

Specifies whether the image should exist. Valid options: 'present' and 'absent'. Default: 'present'.

#####`image_tag`

Installs image tags. Equivalent to running `docker pull -t="precise" ubuntu`.

####Define: docker::run

Parameters are optional, except where noted.

#####`before_stop`

Provide a command to run whenever the service running the container
recieves a stop command. Valid options: a string containing a shell
command. Default: false.

#####`command`

Specifies a command to run inside the container once it has been started. Valid options: a string.

#####`cpuset`

Binds the containers to a specific CPU core on the host. Valid options: a string containing a CPU core ID number. Default: undefined.

#####`depends`

Specifies containers that must be started before this one. Valid options: a string containing the name of another container.

#####`dns`

Specifies a DNS server for the Docker daemon to connect to. This is useful if DNS resolution isn't working properly in the container. Valid options: a string containing an IP address.

#####`env`

Lets you set environment variables within the container. Valid options: an array of strings containing `<key>=<value>` pairs. Default: undefined.

#####`env_file`

Lets you set environment variables within the container based on environment files. Valid options: an array of strings containing the path to the file. Default: undefined.

#####`expose`

Specifies whether the port(s) specified in the `ports` parameter should be available for incoming connections on the respective container from other Docker containers. Valid options: an array of port numbers.

#####`hostname`

Specifies the hostname inside the respective container. Accepts a valid hostname.

#####`image`

Specifies a Docker image on which to base this container. You can also include a specific tag (e.g., `ubuntu:latest`). Valid options: a string containing the name of an image.

#####`links`

Creates Docker links between multiple containers. Valid options: a string containing the Docker linking syntax. For more information, see the Docker [linking system](https://docs.docker.com/userguide/dockerlinks/#connect-with-the-linking-system) documentation.

#####`memory_limit`

Specifies the memory limit for the container. Valid options: a string containing a size (e.g., '5G', '10G', '11G').

#####`ports`

Binds one or more ports inside the container to a different port on the host. This lets you access a service running inside a container from outside of Docker. Valid options: an array of port numbers.

#####`privileged`

Specifies whether to make the container a "privileged container". Valid options: 'true' and 'false'. For further details, see the [Docker documentation](https://docs.docker.com/reference/run/#runtime-privilege-linux-capabilities-and-lxc-configuration).

#####`pull_on_start`

Specifies whether to pull a fresh copy of the image from the upstream repository every time the container starts up. Valid options: 'true' and 'false'. Default: 'false'.

#####`restart_service`

Specifies whether to restart the containers if any other properties change. Valid options: 'true' and 'false'.

#####`running`

Specifies whether the container should be running. Valid options: 'true' and 'false'. Default: 'true'.

#####`username`

Runs the Docker container as a paticular user on the host system. Valid options: a string containing a username. Default: undefined.

#####`volumes`

Mounts directories from the host filesystem inside the respective container. Valid options: an array of strings, where each string comprises a path on the host system, a colon, and a mount location within the container. For example, 'root:/root/mnt:rw'.

#####`volumes_from`

Mounts a volume from another container on the current container. Valid values: a string or an array of strings, where each string contains the name of a source container. Default: undefined.

####Define: docker::exec

Parameters are optional, except where noted.

#####`command`

*Required* Specifies a command to run inside the container. Accepts a string.

#####`container`

*Required*. Specifies the name of the container in which to run the specified command. Accepts a string.

#####`detach`

Specifies whether to run the command in the background. Valid options: 'true', false'. Default: 'false'.

#####`tty`

Specifies whether to allocate a pseudo-TTY. Valid options: 'true', false'. Default: 'false'.


#####`unless`

A command to run in the context of the container; if it returns a non-zero exit code, the command in the `command` is run.

####Class: docker::registry

#####`name`

*Required* The full address of the private registry you would like to
authenticate with.

#####`username`

*Required* The username of the user you would like to authenticate with
the specified private registry.

#####`password`

*Required* The private registry password for the specified user.

#####`email`

*Required* The email address associated with the private registry
account.

##Limitations

###Support

This module is currently supported on:

* RedHat Enterprise Linux 7.1 x86_64
* CentOS 7.1 x86_64
* Oracle Linux 7.1 x86_64
* Scientific Linux 7.1 x86_64
* Ubuntu 14.04 x86_64

###Known Issues

Depending on the initial state of your OS, you might run into issues which prevent Docker from starting properly:

####Enterprise Linux 7

EL7 (RedHat/CentOS/Oracle/Scientific) requires at least version 1.02.93 of the device-mapper package to be installed for Docker's default configuration to work. That version is only available on EL7.1+.

You can install this package via Puppet using the following manifest:

~~~puppet
package {'device-mapper':
  ensure => latest,
}
~~~

To ensure that device-mapper is installed before the `docker` class is executed, use the `before` or `require` [metaparameters](https://docs.puppetlabs.com/references/latest/metaparameter.html).

##Development
Puppet Labs modules on the Puppet Forge are open projects, and community contributions are essential for keeping them great. We can't access the huge number of platforms and myriad hardware, software, and deployment configurations that Puppet is intended to serve. We want to keep it as easy as possible to contribute changes so that our modules work in your environment. There are a few guidelines that we need contributors to follow so that we can have a chance of keeping on top of things.

For more information, see our [module contribution guide.](https://docs.puppetlabs.com/forge/contributing.html)

To see who's already involved, see the [list of contributors.](https://github.com/puppetlabs/puppetlabs-docker_platform/graphs/contributors)
