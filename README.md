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

[TODO: something about what the module does and why someone would want to use it.]

##Setup

###Setup requirements

[TODO: Are there any setup requirements?]

###Beginning with docker_platform

[TODO: This is generally a simple usage example, for the most basic thing a user might want to do with the module. I used the install example, but I don't know whether that's really the right thing or not, or if I've included enough of it.]

The docker_platform module includes a single class, `docker`. To install Docker, include the class: 

```puppet
include 'docker'
```

By default, for CentOS/RHEL, this will install Docker from a pre-existing repo [TODO: does the user need to know or do anything about this repo?]. 
For Ubuntu, this will install Docker from the upstream Docker repository.

## Usage

[TODO: Many of these examples basically repeat the information I'm putting in the Reference section, so a lot of them can be cut from this part of the doc. Do you think the Basic Usage Example will be enough, or are there other examples here that are important to keep here?]

###Installing Docker

[TODO: rather than examples for each parameter, could I get one example for an install that uses some of the parameters?] 

For Ubuntu, if you want to configure your package sources independently, set `use_upstream_package_source` to 'false'. This prevents the modules from auto-including upstream sources:

```puppet
class { 'docker':
  use_upstream_package_source => false,
}
```

By default the Docker daemon will bind to a Unix socket at
/var/run/docker.sock. You can change this setting, as well as binding to a tcp
socket if required.

```puppet
class { 'docker':
  tcp_bind    => 'tcp://127.0.0.1:4243',
  socket_bind => 'unix:///var/run/docker.sock',
}
```

Unless otherwise specified, the module installs the latest version of Docker on its first run. However, if you want to specify a specific version, you can set the `version`:

```puppet
class { 'docker':
  version => '0.5.5',
}
```

Also, if you want to explicitly track the latest version you can do so as follows:

```puppet
class { 'docker':
  version => 'latest',
}
```

In some cases dns resolution won't work well in the container unless you give a dns server to the docker daemon like this:

```puppet
class { 'docker':
  dns => '8.8.8.8',
}
```

To add users to the Docker group you can pass an array like this:

```puppet
class { 'docker':
  docker_users => [ 'user1', 'user2' ],
}
```

The class contains lots of other options, please see the inline code
documentation for the full options.

### Images

[TODO: same thing here; one example using all or some parameters will be more useful than individual examples.]

The next step is probably to install a docker image; for this we have a defined type which can be used like so:

```puppet
docker::image { 'base': }
```

This is equivalent to running `docker pull base`. This is downloading a large binary so on first run can take a while. For that reason this define turns off the default 5 minute timeout for exec. Takes an optional parameter for installing image tags that is the equivalent to running `docker pull -t="precise" ubuntu`:

```puppet
docker::image { 'ubuntu':
  image_tag => 'precise'
}
```

Note: images will only install if an image of that name does not already exist.

A images can also be added/build from a Dockerfile with the `docker_file` property, this equivalent to running `docker build -t ubuntu - < /tmp/Dockerfile`

```puppet
docker::image { 'ubuntu':
  docker_file => '/tmp/Dockerfile'
}
```

Images can also be added/build from a directory containing a dockerfile with the `docker_dir` property, this is equivalent to running `docker build -t ubuntu /tmp/ubuntu_image`

```puppet
docker::image { 'ubuntu':
  docker_dir => '/tmp/ubuntu_image'
}
```

You can also remove images you no longer need with:

```puppet
docker::image { 'base':
  ensure => 'absent'
}

docker::image { 'ubuntu':
  ensure    => 'absent',
  image_tag => 'precise'
}
```

If using Hiera, there's a docker::images class you can configure, for example:

```yaml
docker::images:
  ubuntu:
    image_tag: 'precise'
```

### Containers

[TODO: same thing here; one cohesive example using all or some parameters will be more useful than individual examples.]

Now you have an image you can run commands within a container managed by docker.

```puppet
docker::run { 'helloworld':
  image   => 'base',
  command => '/bin/sh -c "while true; do echo hello world; sleep 1; done"',
}
```

This is equivalent to running the following under upstart:

    docker run -d base /bin/sh -c "while true; do echo hello world; sleep 1; done"

Run also contains a number of optional parameters:

```puppet
docker::run { 'helloworld':
  image           => 'base',
  command         => '/bin/sh -c "while true; do echo hello world; sleep 1; done"',
  ports           => ['4444', '4555'],
  expose          => ['4666', '4777'],
  links           => ['mysql:db'],
  use_name        => true,
  volumes         => ['/var/lib/couchdb', '/var/log'],
  volumes_from    => '6446ea52fbc9',
  memory_limit    => 10m, # (format: <number><unit>, where unit = b, k, m or g)
  cpuset          => ['0', '3'],
  username        => 'example',
  hostname        => 'example.com',
  env             => ['FOO=BAR', 'FOO2=BAR2'],
  dns             => ['8.8.8.8', '8.8.4.4'],
  restart_service => true,
  privileged      => false,
  pull_on_start   => false,
  depends         => [ 'container_a', 'postgres' ],
}
```

Ports, expose, env, dns and volumes can be set with either a single string or as above with an array of values.

Specifying `pull_on_start` will pull the image before each time it is started.

The `depends` option allows expressing containers that must be started before. This affects the generation of the init.d/systemd script.

The service file created for systemd and upstart based systems enables automatic restarting of the service on failure by default.

To use an image tag just append the tag name to the image name separated by a semicolon:

```puppet
docker::run { 'helloworld':
  image   => 'ubuntu:precise',
  command => '/bin/sh -c "while true; do echo hello world; sleep 1; done"',
}
```

If using hiera, there's a docker::run_instance class you can configure, for example:

```yaml
docker::run_instance:
  helloworld:
    image: 'ubuntu:precise'
    command: '/bin/sh -c "while true; do echo hello world; sleep 1; done"'
```

### Exec

[TODO: Nothing, this example looks good to me! :-)]

Docker also supports running arbitrary comments within the context of a
running container. And now so does the Puppet module.

```puppet
docker::exec { 'helloworld-uptime':
  detach    => true,
  container => 'helloworld',
  command   => 'uptime',
  tty       => true,
}
```

### Full Basic Example

To install docker, download a Ubuntu image, and run a Ubuntu-based container that does nothing except
run the init process, you can use the following example manifest:

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
  This example contains a fairly simple example using Vagrant to launch a
  Linux virtual machine, then Puppet to install Docker, build an image and
  run a container. For added spice the container runs a ASP.NET vNext
  application.
* [Multihost containers connected with
  Consul](https://github.com/garethr/puppet-docker-example)
  Launch multiple hosts running simple application containers and
  connect them together using Nginx updated by Consul and Puppet.
* [Configure Docker Swarm using
  Puppet](https://github.com/garethr/puppet-docker-swarm-example)
  Build a cluster of hosts running Docker Swarm configured by Puppet.

##Reference

###Classes

####docker

#####`use_upstream_package_source`

Optional; applies to Ubuntu only. Whether the upstream package source should be used for installation. Valid values are 'true', 'false'. Defaults to 'true'.

#####`tcp_bind`
Optional. Specify the tcp socket the Docker daemon should bind to. [TODO: Is there a default value?] Optional. 

#####`socket_bind`

Optional. Specify the Unix socket the Docker daemon should bind to. By default the Docker daemon will bind to a Unix socket at `/var/run/docker.sock`.

#####`version`

Optional. Specify a version of Docker to use. Accepts a version number or 'latest'. Default is'0.5.5',

#####`dns`

Specify a dns server for the Docker daemon to connect to. This is useful if DNS resolution isn't working properly in the container. 

#####`docker_users` => [ 'user1', 'user2' ],

Add users to the Docker group. Accepts an array like:

```puppet
class { 'docker':
  docker_users => [ 'user1', 'user2' ],
}
```

The class contains lots of other options, please see the inline code documentation for the full options. [TODO: all parameters need to be listed in the readme, so I need a list of parameters with descriptions.]


####`docker::images`

[TODO: I need to know what this class does and what the parameters are for it. Also, I'm confused, because the doc said this is a class you can use if you are using Hiera, but the docs also said that `docker` is the only class. Help?]


###Defines

####docker::image

#####base

This is equivalent to running `docker pull base`. This is downloading a large binary so on first run can take a while. For that reason this define turns off the default 5 minute timeout for exec. Takes an optional parameter for installing image tags that is the equivalent to running `docker pull -t="precise" ubuntu`:

#####image_tag

Optional. Installs image tags. Equivalent to running `docker pull -t="precise" ubuntu`

Note: images will only install if an image of that name does not already exist.

#####`docker_file`
Optional. Add images from a Dockerfile. Accepts a path to the Dockerfile.

#####`docker_dir`

Add images from a directory containing a Dockerfile with the `docker_dir` property. Accepts a path to the images.

#####`ensure`

Specify whether the image should be present or absent. Valid values are 'present', 'absent'. Default is 'present'.

####docker::run

#####`image`

The docker image to base this container off. Can also include a specific tag (e.g. ubuntu:latest).

#####`command`

The command to run inside the container once it has been started.

#####`ports`

Can be used to bind a port or ports inside a container to a different port on the host. This can be used to access a service running inside a container outside of docker.

#####`expose`

The port or ports specified will be available for incoming connections on the respective container from other docker containers.

#####`hostname`

This property will set the hostname inside the respective container.

#####`volumes`

This property allows you to mount folders on your host system inside the respective container.

#####`links`

This property allows you to create docker links between multiple containers.

#####`running`

If set to false, this will ensure that the respective container is not running. If set to true, this will ensure that the container is running.

[TODO: I need descriptions for all of the parameters below not already mentioned above]

docker::run { 'helloworld':
  image           => 'base',
  command         => '/bin/sh -c "while true; do echo hello world; sleep 1; done"',
  ports           => ['4444', '4555'],
  expose          => ['4666', '4777'],
  links           => ['mysql:db'],
  use_name        => true,
  volumes         => ['/var/lib/couchdb', '/var/log'],
  volumes_from    => '6446ea52fbc9',
  memory_limit    => 10m, # (format: <number><unit>, where unit = b, k, m or g)
  cpuset          => ['0', '3'],
  username        => 'example',
  hostname        => 'example.com',
  env             => ['FOO=BAR', 'FOO2=BAR2'],
  dns             => ['8.8.8.8', '8.8.4.4'],
  restart_service => true,
  privileged      => false,
  pull_on_start   => false,
  depends         => [ 'container_a', 'postgres' ],
  
####docker::exec [TODO: I need descriptions about each of these parameters]

#####`detach` 

#####`container`

#####`command`

#####`tty`


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

You can install this package via puppet using the following manifest:

```puppet
package {'device-mapper':
  ensure => latest,
}
```

Remember to add the appropriate metaparameters (before or require) for your environment to ensure that device-mapper is installed before the docker class is executed.

RHEL7 also has the above issue. 

Additionally, RHEL7 needs the rhel7-extras repo enabled in order to install docker. You can install docker using the following example manifest:

```puppet
package {'docker_rhel':
  name		        => 'docker',
  install_options => ['--enablerepo=rhel7-extras'],
  ensure          => latest,
}
```

Again, remember to add the appropriate metaparameters (before or require) for your environment to ensure that device-mapper is installed before the docker package is installed.

##Development

Puppet Labs modules on the Puppet Forge are open projects, and community contributions are essential for keeping them great. We can't access the huge number of platforms and myriad of hardware, software, and deployment configurations that Puppet is intended to serve.

We want to keep it as easy as possible to contribute changes so that our modules work in your environment. There are a few guidelines that we need contributors to follow so that we can have a chance of keeping on top of things.

You can read the complete module contribution guide [on the Puppet Labs wiki.](http://projects.puppetlabs.com/projects/module-site/wiki/Module_contributing)