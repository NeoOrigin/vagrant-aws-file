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

An alternative use comes from using env file configurations when you want to debug your environment.  The following checks and subsequently overrides the vagrant BOX specified in the vmware-dev.env config for debugging purposes:

```shell
$ source config/vmware-dev.env
$ echo $BOX
centos/6
$ export BOX=centos/7
$ echo $BOX
centos/7
$ vagrant up --debug
```

There is also support for guest/remote proxy configuration, code deployment and multiple providers. All allowing you a streamlined workflow from development to production.

## Installation

The Vagrantfile has been designed as a standalone drop in for your vagrant enabled project. Simply download to your project root to get started.

## Overview

The Vagrantfile is a slightly opinionated yet customisable implementation of a standard Vagrantfile (essentially a ruby script) intended to work with the Hashicorp Vagrant product.

Unlike most Vagrantfiles this file has been parameterised to allow the developer to specify options via the command line, configuration files, environment variables or a combination of the above.  This provides the following benefits:

* Ability to define multiple configurations representing dev, test, prod etc
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

The following consitute the core settings that override how vagrant works or are common to most use cases

| Name     | Example             | Description                                    |
| -------- | ------------------- | ---------------------------------------------- |
| box      | --box centos/7      | Uses the downloaded box name                   |
| name     | --name etl-0        | Names the vm within the provider               |
| protocol | --protocol winrm    | Either ssh or winrm to connect to the vm       |
| sshUser  | --ssh-user mike     | The ssh user to connect with                   |
| config   | --config myconf.yml | Reads settings from a given configuration file |

### Remote Proxy

The following proxy settings can be set on the guest/remote host if the varant plugin vagrant-proxyconf is set.

```shell
$ vagrant plugin install vagrant-proxyconf
```

| Name       | Example                                      | Description                                                   |
| ---------- | -------------------------------------------- | ------------------------------------------------------------- |
| ftpProxy   | --ftp-proxy <host:port>                      | The proxy configuration to use in the guest for ftp traffic   |
| httpProxy  | --http-proxy <host:port>                     | The proxy configuration to use in the guest for http traffic  |
| httpsProxy | --https-proxy <host:port>                    | The proxy configuration to use in the guest for https traffic |
| noProxy    | --no-proxy localhost,127.0.0.1,.example.com  | Domains and IP addresses that bypass the proxy                |

### Provisioning

| Name   | Example             | Description                                            |
| ------ | ------------------- | ------------------------------------------------------ |
| run    | --run provision.sh  | Runs a provisioning script or inline shell command     |
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

For configuration, the yaml or json formats are supported.  If specifying an inline value or if the format of these files cannot be determined via the file extension you can prefix the value like so:

| Example                                      | Description                    |
| -------------------------------------------- | ------------------------------ |
| yamlfile://yaml formatted file path/         | A Yaml configuration file path |
| jsonfile://json formatted file path/         | A Json configuration file path |
| inifile://ini formatted file path/           | A Ini configuration file path  |
| envfile://env variables formatted file path/ | A Env configuration file path  |
| yaml://yaml formatted data/                  | An inline Yaml configuration   |
| json://json formatted data/                  | An inline Json configuration   |
| ini://ini formatted data/                    | An inline Ini configuration    |
| env://env variables formatted data/          | An inline Env configuration    |
| rawfile://local unformatted data path/       | A Raw file path for non configuration files |
| raw://string/                                | inline data, added for completeness, mostly not needed |
| file://local unformatted data path/          | A local file path for raw data, typically used for user_data or where content can be determined from the file extension |
| ftp://remote path/                           | A remote file path on an ftp server, standard auth settings can be included in the url |
| http://remote path/                          | A remote file path on an http server, basic auth settings can be included in the url |
| https://remote path/                         | A remote file path on an https server, basic auth settings can be included in the url |

To use the inifile format you must have the inifile gem installed, this an be achieved as so:

```shell
$ gem install inifile
```

Similarly for the env format you must have the dotenv gem installed

```shell
$ gem install dotenv
```
