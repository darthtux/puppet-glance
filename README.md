Team and repository tags
========================

[![Team and repository tags](https://governance.openstack.org/tc/badges/puppet-glance.svg)](https://governance.openstack.org/tc/reference/tags/index.html)

<!-- Change things from this point on -->

glance
=======

#### Table of Contents

1. [Overview - What is the glance module?](#overview)
2. [Module Description - What does the module do?](#module-description)
3. [Setup - The basics of getting started with glance](#setup)
4. [Implementation - An under-the-hood peek at what the module is doing](#implementation)
5. [Limitations - OS compatibility, etc.](#limitations)
6. [Beaker-Rspec - Beaker-rspec tests for the project](#beaker-rpsec)
7. [Development - Guide for contributing to the module](#development)
8. [Release Notes - Release notes for the project](#release-notes)
9. [Contributors - Those with commits](#contributors)
10. [Repository - The project source code repository](#repository)

Overview
--------

The glance module is a part of [OpenStack](https://opendev.org/openstack), an effort by the OpenStack infrastructure team to provide continuous integration testing and code review for OpenStack and OpenStack community projects as part of the core software. The module its self is used to flexibly configure and manage the image service for OpenStack.

Module Description
------------------

The glance module is a thorough attempt to make Puppet capable of managing the entirety of glance.  This includes manifests to provision such things as keystone endpoints, RPC configurations specific to glance, and database connections.  Types are shipped as part of the glance module to assist in manipulation of configuration files.

This module is tested in combination with other modules needed to build and leverage an entire OpenStack software stack.

Setup
-----

**What the glance module affects**

* [Glance](https://docs.openstack.org/glance/latest/), the image service for OpenStack.

### Installing glance

    puppet module install openstack/glance

### Beginning with glance

To utilize the glance module's functionality you will need to declare multiple resources. This is not an exhaustive list of all the components needed, we recommend you consult and understand the [core openstack](https://docs.openstack.org) documentation.

**Define a glance node**

```puppet
class { 'glance::api::authtoken':
  password => '12345',
  auth_url => 'http://172.17.0.3:5000',
  auth_uri => 'http://172.17.0.3:5000',
}

class { 'glance::registry::authtoken':
  password => '12345',
  auth_url => 'http://172.17.0.3:5000',
  auth_uri => 'http://172.17.0.3:5000',
}

class { 'glance::api':
  database_connection => 'mysql+pymysql://glance:12345@127.0.0.1/glance',
  stores              => ['file', 'http'],
  default_store       => 'file',
}

class { 'glance::registry':
  database_connection => 'mysql+pymysql://glance:12345@127.0.0.1/glance',
}

class { 'glance::backend::file': }
```

**Setup postgres node glance**

```puppet
class { 'glance::db::postgresql':
  password => '12345',
}
```

**Setup mysql node for glance**

```puppet
class { 'glance::db::mysql':
  password      => '12345',
  allowed_hosts => '%',
}
```

**Setup up keystone endpoints for glance on keystone node**

```puppet
class { 'glance::keystone::auth':
  password     => '12345'
  email        => 'glance@example.com',
  public_url   => 'http://172.17.0.3:9292',
  admin_url    => 'http://172.17.0.3:9292',
  internal_url => 'http://172.17.1.3:9292',
  region       => 'example-west-1',
}
```

**Setup up notifications for multiple RabbitMQ nodes**

```puppet
class { 'glance::notify::rabbitmq':
  default_transport_url      => os_transport_url({
    'transport'    => 'rabbit',
    'hosts'        => ['host1', 'host2'],
    'username'     => 'glance',
    'password'     => 'secret',
    'virtual_host' => 'glance',
  )},
  notification_transport_url => os_transport_url({
    'transport'    => 'rabbit',
    'hosts'        => ['host1', 'host2'],
    'username'     => 'notify',
    'password'     => 'secret',
    'virtual_host' => 'notify',
  )},
}
```

### Types

#### glance_api_config

The `glance_api_config` provider is a children of the ini_setting provider. It allows one to write an entry in the `/etc/glance/glance-api.conf` file.

```puppet
glance_api_config { 'DEFAULT/image_cache_dir' :
  value => /var/lib/glance/image-cache,
}
```

This will write `image_cache_dir=/var/lib/glance/image-cache` in the `[DEFAULT]` section.

##### name

Section/setting name to manage from `glance-api.conf`

##### value

The value of the setting to be defined.

##### secret

Whether to hide the value from Puppet logs. Defaults to `false`.

##### ensure_absent_val

If value is equal to ensure_absent_val then the resource will behave as if `ensure => absent` was specified. Defaults to `<SERVICE DEFAULT>`

#### glance_registry_config

The `glance_registry_config` provider is a children of the ini_setting provider. It allows one to write an entry in the `/etc/glance/glance-registry.conf` file.

```puppet
glance_registry_config { 'DEFAULT/workers' :
  value => 1,
}
```

This will write `workers=1` in the `[DEFAULT]` section.

##### name

Section/setting name to manage from `glance-registry.conf`

##### value

The value of the setting to be defined.

##### secret

Whether to hide the value from Puppet logs. Defaults to `false`.

##### ensure_absent_val

If value is equal to ensure_absent_val then the resource will behave as if `ensure => absent` was specified. Defaults to `<SERVICE DEFAULT>`

#### glance_cache_config

The `glance_cache_config` provider is a children of the ini_setting provider. It allows one to write an entry in the `/etc/glance/glance-cache.conf` file.

```puppet
glance_cache_config { 'DEFAULT/image_cache_stall_time' :
  value => 86400,
}
```

This will write `image_cache_stall_time=86400` in the `[DEFAULT]` section.

##### name

Section/setting name to manage from `glance-cache.conf`

##### value

The value of the setting to be defined.

##### secret

Whether to hide the value from Puppet logs. Defaults to `false`.

##### ensure_absent_val

If value is equal to ensure_absent_val then the resource will behave as if `ensure => absent` was specified. Defaults to `<SERVICE DEFAULT>`

Implementation
--------------

### glance

glance is a combination of Puppet manifest and ruby code to deliver configuration and extra functionality through types and providers.

Limitations
------------

* Only supports configuring the file, swift and rbd storage backends.

Beaker-Rspec
------------

This module has beaker-rspec tests

To run the tests on the default vagrant node:

To run:

```shell
bundle install
bundle exec rspec spec/acceptance
```

For more information on writing and running beaker-rspec tests visit the documentation:

* https://github.com/puppetlabs/beaker-rspec/blob/master/README.md

Development
-----------

Developer documentation for the entire puppet-openstack project.

* https://docs.openstack.org/puppet-openstack-guide/latest/

Release Notes
-------------

* https://docs.openstack.org/releasenotes/puppet-glance

Contributors
------------

* https://github.com/openstack/puppet-glance/graphs/contributors

Repository
----------

* https://opendev.org/openstack/puppet-glance
