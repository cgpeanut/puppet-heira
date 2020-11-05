# puppet-heira
- Two different methods of storing variables that can be used in place of any attribute:
- The params pattern, and Hiera
- Refactor MySQL module that's been set up to use params pattern to Hiera.

# Create OS-specific Hiera data
1. From the puppet master, find the mysql module:
- $ cd /etc/puppetlabs/code/environments/production/modules/mysql

2. Review all files, focusing specifically on the params.pp file. Make note of all parameters:
- install_name
- install_ensure
- config_path
- config_ensure
- service_name
- service_ensure
- service_enable
- service_hasrestart

- Notice how some variables are OS-family-specific while others are not.

3. Set up Hiera to filter by OS family:

$ sudo vim hiera.yaml

hierarchy:
 - name: 'Operating System Family'
   path: '%{facts.os.family}-family.yaml'

 - name: 'common'
   path: 'common.yaml'

Save and Exit

4. Add the RedHat-family.yaml values:

$ sudo vim data/RedHat-family.yaml

---
mysql::install_name: 'mariadb-server'
mysql::config_path: '/etc/my.cnf.d/server.cnf'
mysql::service_name: 'mariadb'

5. Do the same for Debian-family.yaml:

$ sudo vim data/Debian-family.yaml
---
mysql::install_name: 'mysql-server-5.7'
mysql::config_path: '/etc/mysql/mysql.conf.d/mysqld.cnf'
mysql::service_name: 'mysql'

6. Add the defaults to common.yaml:

---
mysql::install_name: 'mysql-server-5.7'
mysql::install_ensure: 'present'
mysql::config_path: '/etc/mysql/mysql.conf.d/mysqld.cnf'
mysql::config_ensure: 'file'
mysql::service_ensure: 'running'
mysql::service_name: 'mysql'
mysql::service_enable: true
mysql::service_hasrestart: true

# Update the manifest to use them: 

1. Next, we want to refactor the manifests themselves. Let's start with the install class:

$ sudo vim manifests/install.pp

# @summary
#   Installs MySQL
class mysql::install {
  package { "${mysql::install_name}":
    ensure => $mysql::install_ensure,
  }
}

2. Then, update the config class:

$ sudo vim manifests/config.pp

# @summary A short summary of the purpose of this class
#  Configures MySQL daemon file
class mysql::config {
  file { "${mysql::config_path}":
    ensure => $mysql::config_ensure,
    source => "puppet:///modules/mysql/${osfamily}.cnf",
    mode   => '0644',
    owner  => 'root',
    group  => 'root',
  }
}

3. Edit service:

$ sudo vim manifests/service.pp

# @summary A short summary of the purpose of this class
#   Ensures service is started and enabled and allow for restarts
class mysql::service {
  service { "${mysql::service_name}":
    ensure     => $mysql::service_ensure,
    enable     => $mysql::service_enable,
    hasrestart => $mysql::service_hasrestart,
  }
}

4. Finally, we have to rework the init.pp file:

$ sudo vim manifests/init.pp

# @summary
# Manages the basic functions of a MySQL server
class mysql (
  String $install_name,
  String $install_ensure,
  String $config_path,
  String $config_ensure,
  String $service_name,
  String $service_ensure,
  Boolean $service_enable,
  Boolean $service_hasrestart,
) {
  contain mysql::install
  contain mysql::config
  contain mysql::service

  Class['::mysql::install']
  -> Class['::mysql::config']
  ~> Class['::mysql::service']
}

# test the module against both servers

Add the module to our main manifest:

$ sudo vim ../../manifests/site.pp
node default {
  include mysql
}
Test the module on the Puppet master:

$ sudo puppet agent -t
We also want to test the module on the Ubuntu node. Let's first ensure Puppet is up and running on that additional server:

$ curl -k https://puppet.ec2.internal:8140/packages/current/install.bash | sudo bash
Approve the cert on the master:

$ sudo puppetserver ca sign --all
Test the module against the additional node:

$ sudo puppet agent -t
Finally, remove the params.pp file:

$ sudo rm manifests/params.pp
