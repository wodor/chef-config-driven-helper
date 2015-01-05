# Config driven helpers

The config driven helper cookbook is a simple catchall cookbook designd to drive other cookbooks that are not normally config driven.

Enabling this ability means that there is less duplication and therefore less scope for error across projects which utilize this cookbook.

## Apache sites
The apache sites helper enables you to create an apache vhost simply by defining it as node attributes.

Add `config-driven-helper::apache-sites` to enable it.

Here is a simple example:

```json
{
  'apache': {
    'sites': {
      'inviqa': {
        'server_name': 'inviqa.com',
        'docroot': '/var/www/inviqa.com'
      }
    }
  }
}
```

The above, when integrated in to your chef attributes will create a vhost called `inviqa` that responds to the vhost `inviqa.com` and has a document root of `/var/www/inviqa.com`.

Internally the contents of the 'inviqa' site attribute will be proxied to the apache2 cookbook `web_app` resource. This means that any configuration that is valid for `web_app` is valid here too!

As an example, consider changing the template to a custom one that you have created in a project cookbook called project.

```json
{
  'apache': {
    'sites': {
      'inviqa': {
        'server_name': 'inviqa.com',
        'docroot': '/var/www/inviqa.com',
        'template': 'my-custom-template.conf.erb',
        'cookbook': 'my-project-cookbook'
      }
    }
  }
}

```

We recommend that you use an abstract name for the key of the site (and not the hostname). This is to allow you to share site attributes across environments with minimal duplication.

### SSL

The apache sites helper also provides a means to create https vhosts. To do this you must set the 'protocols' array.

```json
{
  'apache': {
    'sites': {
      'inviqa': {
        'server_name': 'inviqa.com',
        'docroot': '/var/www/inviqa.com',
        'protocols': [ 'http', 'https' ]
      }
    }
  }
}

```

This example will create two vhosts; `inviqa` and `inviqa.ssl`. `inviqa.ssl` will be configured to respond on port 443 with some sensible SSL default settings (including limiting to secure protocols and ciphersuites).

The defaults for a site are defined here: https://github.com/inviqa/chef-config-driven-helper/blob/master/attributes/default.rb#L14

These default will be merged in to any given site.

Therefore, by default the SSL vhost template will use `/etc/pki/tls/certs/cert.pem` as the certfile and `/etc/pki/tls/private/key.pem` as the keyfile.

The apache sites helper will populate these keys for you by looking at the `node['ssl_certs']['/etc/pki/tls/certs/cert.pem']` and `node['ssl_certs']['/etc/pki/tls/private/key.pem']` attributes.

You can override any of these values by simply defining them in your site attributes. For a (contrived) example, to change the keyfile path:

```json
{
  'apache': {
    'sites': {
      'inviqa': {
        'server_name': 'inviqa.com',
        'docroot': '/var/www/inviqa.com',
        'protocols': [ 'http', 'https' ],
        'keyfile': '/tmp/my-super-insecure-keyfile.pem'
      }
    }
  }
}

```

It is assumed that projects that use this cookbook also use the data-bag-merge cookbook from https://cookbooks.opscode.com/cookbooks/data-bag-merge. This cookbook merges encrypted data bags in to your chef attributes to enabled encrypted attributes for cookbooks that do not directly support them.

## Nginx sites

The nginx sites helper is very similar to the apache sites helper with the exception that it does not proxy to any kind of `web_app` helper and uses the `nginx` top level attribute instead.

Add `config-driven-helper::nginx-sites` to enable it.

```json
{
  'nginx': {
    'sites': {
      'inviqa': {
        'server_name': 'inviqa.com',
        'docroot': '/var/www/inviqa.com',
        'protocols': [ 'http', 'https' ],
        'keyfile': '/tmp/my-super-insecure-keyfile.pem',
        "php_server_variables": {
          "FOO": "bar",
          "BAZ": "bam"
          }
      }
    }
  }
}

```

## Mysql users

The mysql users helper enables you to create mysql users from attributes. It proxies the attributes to the `mysql_database_user` resource defined by the `database` cookbook here: https://github.com/opscode-cookbooks/database#database_user. This means that any attributes valid there are valid here.

Add `config-driven-helper::mysql-users` to enable it.

The following example creates the user `my_username` with the defined password and with `GRANT ALL PRIVELEGES` applied to `database-to-grant`.

```json
{
  'mysql': {
    'users': {
      'my_username': {
        'password': 'my-password-from-data-bag-merge',
        'database_name': 'database-to-grant'
      }
    }
  }
}
```

## Mysql databases

The mysql database helper enables you to create mysql databases from attributes. It proxies the attributes to the `mysql_database` resource defined by the `database` cookbook here: https://github.com/opscode-cookbooks/database#database. This means that any attributes valid there are valid here.

Add `config-driven-helper::mysql-databases` to enable it.

The following example creates the database `my_database` with no additional options.

```json
{
  'mysql': {
    'databases': {
      'my_database': { }
    }
  }
}
```

## Packages

The packages helper simply enables you to define a list of packages to install. Due to the way that chef merges array attributes you should take care to define all packages at the same precedence level. We may in the future support hash lists to enable purges too.

Add `config-driven-helper::packages` to enable it.

The following example installs both git and java packages.

```json
{
  'packages': [ 'git', 'java' ]
}
```

## Services

The services helper enables you to perform simple actions on pre-defined services such as start / stop / enable / disable them. Note that the service name should be name used by chef, not that of the underlying OS (though they are often the same).

Add `config-driven-helper::services` to enable it.

The following example shows how to make sure that the mysql service is both enabled and started.

```json
  'services': {
    'mysql': [ 'enable', 'start' ]
  }
```

## Mysql ruby

This recipe is primarily for internal use by the mysql helpers. It forces the install of the `mysql` gem in to the chef ruby. This is provided because `chef_gem` is not an appropriate solution where mysql is installed in the same run and depends on a different version of `mysql-libs` (or your OS equivalent) to the version of MySQL being installed. A simple example of this is installing Percona on CentOS 6.5.

Add `config-driven-helper::mysql-ruby` to enable it (although it is explicitly included by the mysql helpers so this should only be necessary if used standalone).
