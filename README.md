# docker_platform

#### Table of Contents

1. [Overview](#overview)
2. [Module Description - What the module does and why it is useful](#module-description)
3. [Setup - The basics of getting started with docker_platform](#setup)
    * [Setup requirements](#setup-requirements)
    * [Beginning with docker_platform](#beginning-with-docker-platform])
4. [Usage - Configuration options and additional functionality](#usage)
5. [Reference - An under-the-hood peek at what the module is doing and how](#reference)
6. [Limitations - OS compatibility, etc.](#limitations)
    * [Support](#support)
    * [Known Issues](#known-issues)
7. [Development - Guide for contributing to the module](#development)

## Overview

The Puppet docker_platform module installs, configures, and manages the [Docker](https://github.com/dotcloud/docker) daemon and Docker containers.

## Description

This module lets you use Puppet to implement the Docker container system across a Puppet-managed infrastructure. It includes classes and defines to install the Docker daemon, manage images and containers across different nodesets, and run commands inside containers.

## Setup

### Setup requirements

For Enterprise Linux 7 systems, a few issues might prevent Docker from starting properly. You can learn about these issues in the [Known Issues](#known-issues) section below.

### Beginning with docker_platform

To install Docker on a node, include the class `docker`.

```puppet
include 'docker'
```

This installs [Docker](https://github.com/docker/docker) from the [official
repository](http://docs.docker.com/installation/) or alternatively from
[EPEL on RedHat](http://docs.docker.io/en/latest/installation/rhel/)
based distributions.

## Usage

### Installing Docker

You can install Docker with various parameters specified for the [`docker`](#docker) class:

```puppet
class {'docker':
  tcp_bind     => 'tcp://127.0.0.1:4243',
  socket_bind  => 'unix:///var/run/docker.sock',
  version      => '0.5.5',
  dns          => '8.8.8.8',
  docker_users => [ 'user1', 'user2' ],
}
```

This example installs Docker version 0.5.5, binds the Docker daemon to a Unix socket and a tcp socket, provides the daemon with a dns server, and adds two users to the Docker group.

Docker recently [launched new official
repositories](http://blog.docker.com/2015/07/new-apt-and-yum-repos/#comment-247448)
which are now the default for the module from version 5. If you want to
stick with the old repositories you can do so with the following:

```puppet
class { 'docker':
  package_name => 'lxc-docker',
  package_source_location => 'https://get.docker.com/ubuntu',
  package_key_source => 'https://get.docker.com/gpg',
  package_key => '36A1D7869245C8950F966E92D8576A8BA88D21E',
  package_release => 'docker',
}
```

The module also now uses the upstream repositories by default for RHEL
based distros, including Fedora. If you want to stick with the distro
packages you should use the following:

```puppet
class { 'docker':
  use_upstream_package_source => false,
  package_name => 'docker',
}
```

### Images

To install a Docker image, use the define [`docker::image`](#dockerimage):

```puppet
docker::image { 'base': }
```

This is equivalent to running `docker pull base`. This downloads a large binary, so on first run, it can take a while. For that reason, this define turns off the default five-minute timeout for exec.

```puppet
docker::image { 'ubuntu':
  ensure      => 'present',
  image_tag   => 'precise',
  docker_file => '/tmp/Dockerfile',
}
```

The above code adds an image from the listed Dockerfile. Alternatively, you can specify an image from a Docker directory, by using `docker_dir` parameter instead of `docker_file`.

### Containers

Now that you have an image, you can run commands within a container managed by Docker:

```puppet
docker::run { 'helloworld':
  image   => 'base',
  command => '/bin/sh -c "while true; do echo hello world; sleep 1; done"',
}
```

You can set ports, expose, env, dns, and volumes with either a single string or, as above, with an array of values.

Specifying `pull_on_start` pulls the image before each time it is started.

The `depends` option allows expressing containers that must be started before other containers start. This affects the generation of the init.d/systemd script.

The service file created for systemd and upstart based systems enables automatic restarting of the service on failure by default.

To use an image tag, append the tag name to the image name separated by a semicolon:

```puppet
docker::run { 'helloworld':
  image   => 'ubuntu:precise',
  command => '/bin/sh -c "while true; do echo hello world; sleep 1; done"',
}
```

If using Hiera, there's a `docker::run_instance` class you can configure, for example:

```puppet
docker::run_instance::instance:
  helloworld:
    image: 'ubuntu:precise'
    command: '/bin/sh -c "while true; do echo hello world; sleep 1; done"'
```

### Networks

As of Docker 1.9.x, Docker has official support for networks. The module
now exposes a type, `docker_network`, used to manage those. This works
like:

```puppet
docker_network { 'my-net':
  ensure   => present,
  driver   => 'overlay',
  subnet   => '192.168.1.0/24',
  gateway  => '192.168.1.1',
  ip_range => '192.168.1.4/32',
}
```

Only the name is required, along with an ensure value. If you don't pass
a driver Docker network will use the default bridge. Note that some
networks require the Docker daemon to be configured to use them, for
instance for the overlay network you'll need a cluster store configured.
You can do that on the `docker` class like so:

```puppet
extra_parameters => '--cluster-store=<backend>://172.17.8.101:<port>
--cluster-advertise=<interface>:2376'
```

### Compose

Docker Compose allows for describing a set of containers in a simple
YAML format, and then running a command to build and run those
containers. The `docker_compose` type included in the module allows for
using Puppet to run Compose. This means you can have Puppet remediate
any issues and make sure reality matches the model in your Compose
file.

Here's an example. Given the following Compose file:

```yaml
compose_test:
  image: ubuntu:14.04
  command: /bin/sh -c "while true; do echo hello world; sleep 1; done"
```

That could be added to the machine you're running Puppet using a `file`
resource or any other means.

Then define a `docker_compose` resource pointing at the Compose file
like so:

```puppet
docker_compose { '/tmp/docker-compose.yml':
  ensure  => present,
}
```

Now when Puppet runs it will automatically run Compose is required,
for example because the relevant Compose services aren't running.

You can also pass addition options (for example to enable experimental
features) as well as provide scaling rules. The following example
requests 2 containers be running for example. Puppet will now run
Compose if the number of containers for a given service don't match the
provided scale values.

```puppet
docker_compose { '/tmp/docker-compose.yml':
  ensure  => present,
  scale   => {
    'compose_test' => 2,
  },
  options => '--x-networking'
}
```

### Private registries

By default images will be pushed and pulled from [The Docker Hub](https://hub.docker.com/).
If you have your own private registry without authentication, you can fully qualify your image name.
If your private registry requires authentication you may configure a registry using the following:

```puppet
docker::registry { 'example.docker.io:5000':
  username => 'user',
  password => 'secret',
  email    => 'user@example.com',
}
```

You can logout of a registry if it is no longer required.

```puppet
docker::registry { 'example.docker.io:5000':
  ensure => 'absent',
}
```

If using Hiera, there's a `docker::registry_auth` class you can configure,
for example:

```yaml
docker::registry_auth::registries:
  'example.com:5000':
    username: 'user1'
    password: 'secret'
    email: 'user1@example.io'
```

### Exec

You can also run arbitrary commands within the context of a running container:

```puppet
docker::exec { 'helloworld-uptime':
  detach    => true,
  container => 'helloworld',
  command   => 'uptime',
  tty       => true,
}
```

### Full Basic Example

To install Docker, download a Ubuntu image, and run a Ubuntu-based container that does nothing except run the init process, you can use the following example manifest:

```puppet
class { 'docker':}

docker::image { 'ubuntu':
  require => Class['docker'],
}

docker::run { 'test_1':
  image   => 'ubuntu',
  command => 'init',
  require => Docker::Image['ubuntu'],
}
```

## Advanced Community Examples

* [Launch vNext app in Docker using Puppet](https://github.com/garethr/puppet-docker-vnext-example)

This example contains a fairly simple example using Vagrant to launch a Linux virtual machine, then Puppet to install Docker, build an image and run a container. For added spice, the container runs a ASP.NET vNext application.

* [Multihost containers connected with Consul](https://github.com/garethr/puppet-docker-example)

Launch multiple containers and connect them together using Nginx, updated by Consul and Puppet.

* [Configure Docker Swarm using Puppet](https://github.com/garethr/puppet-docker-swarm-example)

Build a cluster of hosts running Docker Swarm configured by Puppet.

## Reference

[Full API reference
documentation](http://puppetlabs.github.io/puppetlabs-docker_platform/)
is available as generated by Puppet Strings.

If you would like a local copy of the module documentation simply install
Puppet Strings as described in the [Strings
documentation](https://github.com/puppetlabs/puppetlabs-strings) and
then run the followin in the module directory.

```
puppet strings
```

This should create a directory called `doc` with all the HTML files in.

## Limitations

### Support

This module is currently supported on:

* RedHat Enterprise Linux 7.1 x86_64
* CentOS 7.1 x86_64
* Oracle Linux 7.1 x86_64
* Scientific Linux 7.1 x86_64
* Ubuntu 14.04/16.04 x86_64

### Known Issues

Depending on the initial state of your OS, you might run into issues which prevent Docker from starting properly:

#### Enterprise Linux 7

EL7 (RedHat/CentOS/Oracle/Scientific) requires at least version 1.02.93 of the device-mapper package to be installed for Docker's default configuration to work. That version is only available on EL7.1+.

You can install this package via Puppet using the following manifest:

~~~puppet
package {'device-mapper':
  ensure => latest,
}
~~~

To ensure that device-mapper is installed before the `docker` class is executed, use the `before` or `require` [metaparameters](https://docs.puppetlabs.com/references/latest/metaparameter.html).

## Development
Puppet Labs modules on the Puppet Forge are open projects, and community contributions are essential for keeping them great. We can't access the huge number of platforms and myriad hardware, software, and deployment configurations that Puppet is intended to serve. We want to keep it as easy as possible to contribute changes so that our modules work in your environment. There are a few guidelines that we need contributors to follow so that we can have a chance of keeping on top of things.

For more information, see our [module contribution guide.](https://docs.puppetlabs.com/forge/contributing.html)

To see who's already involved, see the [list of contributors.](https://github.com/puppetlabs/puppetlabs-docker_platform/graphs/contributors)

## Maintainers

This module is maintained by: Gareth Rushgrove <gareth@puppet.com>
