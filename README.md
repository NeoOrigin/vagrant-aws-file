# vagrant-aws-file

[![GitHub license](https://img.shields.io/badge/license-GPLv3-blue.svg)](https://raw.githubusercontent.com/NeoOrigin/vagrant-aws-file/master/LICENSE)
[![GitHub release](https://img.shields.io/github/release/NeoOrigin/vagrant-aws-file.svg)](https://github.com/NeoOrigin/vagrant-aws-file/releases/latest)

Provides a completely parameterised Vagrantfile capable of working with multiple providers, allowing developers to concentrate on configuration and adopt shared best practices.

Essentially you can do this:

```shell
$ vagrant --config vmware-dev.json up
$ vagrant --config aws-prod.yml --sudo true halt
$ vagrant --config "http://myhost/dev" up
```

Or if config.yml is present, simply:

```shell
$ vagrant up
```

There is also support for guest/remote proxy configuration, code deployment and multiple providers. All allowing you a streamlined workflow from development to production.

## Installation

The Vagrantfile has been designed as a standalone drop in for your vagrant enabled project. Simply download to your project root to get started.

## Overview

The Vagrantfile is a slightly opinionated yet customisable implementation of a standard Vagrantfile (essentially a ruby script) intended to work with the Hashicorp Vagrant product.

Unlike most Vagrantfiles this file has been parameterised to allow the developer to specify options via the command line, configuration files, environment variables or a combination of the above.  This provides the following benefits:

* Ability to define multiple configurations representing dev, test, prod
* Ability to download configuration dynamically e.g from a GitHub URL 
* By separating configuration from code you can ignore sensitive passwords from version control
* Follow 12 factor best practices, defining configuration in environment variables
* Multiple configurations in the same place allowing you to define local vm's alongside aws, retaining the same provisioning steps
* Config files are generally easier to interrogate/modify programmatically than scripts
* Code re-use. Your developers might not know Ruby, how to write Vagrantfiles or configure proxy settings etc
* Ability to override settings at run time for debugging purposes

## Usage

This implementation uses the getoptlong library.  All user arguments must be specified before the vagrant action keyword such as 'up' or 'destroy'. I.e

```shell
$ vagrant --options-go-here-- <action>
```

The Vagrantfile isn't executed directly but copied into your project alongside a configuration file of your own(optional, you can pass options via the command line purely). It will be picked up in the usual vagrant way.

## Supported Options

### Core

| Name     | Example             | Description |
| -------- | ------------------- | ----------- |
| box      | --box centos/7      | Uses the downloaded box name |
| name     | --name etl-0        | Names the vm within the provider |
| protocol | --protocol winrm    | Either ssh or winrm to connect to the vm |
| sshUser  | --ssh-user mike     | The user to connect with |
| config   | --config myconf.yml | Reads settings |

### Remote Proxy

| Name     | Example                 | Description |
| -------- | ----------------------- | ----------- |
| ftpProxy | --ftp-proxy <host:port> | The proxy configuration to use in the guest |

### Provisioning

| Name   | Example             | Description |
| ------ | ------------------- | ----------- |
| run    | --run <script_path> | Runs a provisioning script or inline shell command |
| sudo   | --sudo true         | Elevates privileges during provisioning if set to true |

### Deployments

### Providers

#### AWS

| Name          | Example                  | Description |
| ------------- | ------------------------ | ----------- |
| ami           | --ami i-52677338786      | The ami id to use |
| connectWith   | --connect-with privateIP | Connect to privateIP, publicIP or dns |
| instanceType  | --instance-type t2.micro | The instance type dictating the size of the vm |
| monitor       | --monitor true           | Sets detailed monitoring if true |
| profile       | --profile pre-prod       | The AWS connection profile set in the ~/.aws/ configuration |
| tags          | --tags tagfile.yml       | Name value pairs to tag the instance |

#### Virtualbox

To do

#### VMware 

To do

#### Common Conventions

For those options that accept filepaths they also accept remote ftp, http, https or sftp paths. This may be useful for example if you wish to store your provisioning scripts together in their own global repository rather than local to the project. 

For configuration, the yaml or json formats are supported.  If specifying an inline value or if the format of these files cannot be determined via the file extension you can prefix the value

| Example | Description |
| ------- | ----------- |
| yamlfile://yaml formatted file path/ ||
| jsonfile://json formatted file path/ ||
| yaml://yaml formatted data/ ||
| json://json formatted data/ ||
| rawfile://local unformatted data path/ ||
| raw://string/ ||
| file://local unformatted data path/ ||
| ftp://remote path/ ||
| http://remote path/ ||
| https://remote path/ ||