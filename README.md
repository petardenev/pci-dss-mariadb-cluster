# PCI-DSS MariaDB Cluster Hardened Configuration

This MariaDB Cluster Hardened service configuration provides security configurations for MariaDB. It is intended to set up production-ready mariadb instances that are configured with minimal surface for attackers. Furthermore it is intended to be compliant with the PCI-DSS v3.2.1

This MariaDB configuration is implemented as an ansible role focused on security configuration of MariaDB.
This document provides both a guide on the hardening process and a brief description of the security settings implemented.

##	Software Requirements:

* RHEL/CentOS
* Set up `mysql_root_password` variable

This hardening role installs the hardening and does install MariaDB rpm packages also.
The engineer must only ensure that the following variables are set accordingly:

- `mysql_hardening_enabled: yes` role is enabled by default and can be disabled without removing it from a playbook. You can use conditional variable, for example: `mysql_hardening_enabled: "{{ true if mysql_enabled else false }}"`
- `mysql_hardening_user: 'mysql'` The user that mysql runs as.
- `mysql_datadir: '/var/lib/mysql'` The MySQL data directory
- `mysql_hardening_mysql_hardening_conf_file: '/etc/mysql/conf.d/hardening.cnf'` The path to the configuration file where the hardening will be performed

##	Security Options

Further information is already available at [Deutsche Telekom (German)](http://www.telekom.com/static/-/155996/7/technische-sicherheitsanforderungen-si) and [Symantec](http://www.symantec.com/connect/articles/securing-mysql-step-step)

| Name           | Default Value | Description                        |
| -------------- | ------------- | -----------------------------------|
| `mysql_hardening_chroot` | "" | [chroot](http://dev.mysql.com/doc/refman/5.7/en/server-options.html#option_mysqld_chroot)|
| `mysql_hardening_options.safe-user-create` | 1 | [safe-user-create](http://dev.mysql.com/doc/refman/5.7/en/server-options.html#option_mysqld_safe-user-create)|
| `mysql_hardening_options.secure-auth` | 1 | [secure-auth](http://dev.mysql.com/doc/refman/5.7/en/server-options.html#option_mysqld_secure-auth)|
| `mysql_hardening_options.skip-symbolic-links` | 1 | [skip-symbolic-links](http://dev.mysql.com/doc/refman/5.7/en/server-options.html#option_mysqld_symbolic-links)|
| `mysql_hardening_skip_grant_tables:` | false | [skip-grant-tables](https://dev.mysql.com/doc/refman/5.7/en/server-options.html#option_mysqld_skip-grant-tables)|
| `mysql_hardening_skip_show_database` | 1 | [skip-show-database](http://dev.mysql.com/doc/refman/5.7/en/server-options.html#option_mysqld_skip-show-database)|
| `mysql_hardening_options.local-infile` | 0 | [local-infile](http://dev.mysql.com/doc/refman/5.7/en/server-system-variables.html#sysvar_local_infile)|
| `mysql_hardening_options.allow-suspicious-udfs` | 0 | [allow-suspicious-udfs](https://dev.mysql.com/doc/refman/5.7/en/server-options.html#option_mysqld_allow-suspicious-udfs)|
| `mysql_hardening_chroot.automatic-sp-privileges` | 0 | [automatic_sp_privileges](https://dev.mysql.com/doc/refman/5.7/en/server-system-variables.html#sysvar_automatic_sp_privileges)|
| `mysql_hardening_options.secure-file-priv` | /tmp | [secure-file-priv](https://dev.mysql.com/doc/refman/5.7/en/server-options.html#option_mysqld_secure-file-priv)|
| `mysql_allow_remote_root` | false | delete remote root users |
| `mysql_remove_anonymous_users` | true | remove users without authentication |
| `mysql_remove_test_database` | true | remove test database |

## Example

Assuming that you defined in the hosts.yml file the proper IP addresses

```
ansible-playbook -i hosts.yml main.yml
```


# Setup MySQL replication

This playbook will setup a MySQL replication by doing the following steps:

1. Backup the current databases from the MySQL master
2. Compress the backup
3. Download the backup
4. Upload and uncompress the backup to the MySQL slave
5. Stop MySQL on the slave
6. Delete all old MySQL databases from the MySQL slave
7. Import the backup
8. Start MySQL on the slave
9. Configute the MySQL slave process
10. Start the MySQL slave process


WARNING: This playbook will delete all previous data from the MySQL slave!

## Requirements

- A working MySQL installation on the master.
- A replication User on the master. (`GRANT REPLICATION SLAVE ON *.* TO 'SOMEIP'@'SLAVEHOST';`) You may also want to use `REQUIRE SSL`.
- [innobackupex](https://www.percona.com/doc/percona-xtrabackup/2.1/innobackupex/innobackupex_script.html) to backup the master.
- A working MySQL installation on the slave.

## Options

| Option                     | Default | Description |
|----------------------------|---------|-------------|
| master                     |         | The MySQL master from your Ansible inventory. |
| slave                      |         | The MySQL slave from your Ansible inventory. All MySQL data from this host will be deleted. |
| mysql_replication_master   | master  | The Hostname or IP that will be set as MASTER_HOST in MySQL. |
| mysql_replication_user     |         | MASTER_USER in MySQL. |
| mysql_replication_password |         | MASTER_PASSWORD in MySQL. |

## Example

Assuming that you defined `mysql_replication_user` and `mysql_replication_password` in your host_vars you can simply run:

```
ansible-playbook replication.yml -e 'master=master.exmaple.com slave=slave.example.com'
```
