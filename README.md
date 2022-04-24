# Ansible Role: MySQL

[![builds.sr.ht status](https://builds.sr.ht/~tleguern/ansible-role-mysql.svg)](https://builds.sr.ht/~tleguern/ansible-role-mysql?)

Installs and configures MySQL or MariaDB on the following operating systems:

- OpenBSD (MariaDB only)
- Debian
- Fedora

This role is originally based on Jeff Geerling's with an emphasis on better portability (ie: Non-Linux unixen) and simplicity.
It is therefore not entirely compatible as some variables were renamed to prevent misunderstanding and the replication parameters have been excised and left to another role.

Automatic testing is provided using molecule's delegated driver and <https://builds.sr.ht>.

## Requirements

This role requires root access and should be run with `become=yes`.

## Role Variables

| Variable | Description | Default |
|----------|-------------|---------|
| `mysql_config_path` | Path of the main configuration file | `{{ __mysql_config_path }}` |
| `mysql_config` | Extra configuration rules for my.cnf. See bellow. | `[]` |
| `mysql_databases` | The MySQL databases to create | `[]` |
| `mysql_datadir` | Directory where MySQL data are stored | `{{ __mysql_datadir }}` |
| `mysql_db_admin_password_update` | Whether to force update the MySQL root user's password | `false` |
| `mysql_db_admin_password` | MySQL administrator password  | mandatory |
| `mysql_db_admin_user` | MySQL default administrator account | `root` |
| `mysql_packages` | A list of packages | `{{ __mysql_packages }}` |
| `mysql_python_package` | Name of the python package used by Ansible | `{{ __mysql_python_package }}` |
| `mysql_provider` | Either `mariadb` or `mysql` | `mariadb` |
| `mysql_service` | MySQL service name | `{{ __mysql_service }}` |
| `mysql_socket` | MySQL Unix domain socket used for connections | `{{ __mysql_socket }}` |
| `mysql_system_group` | MySQL system group | `{{ __mysql_system_group }}` |
| `mysql_system_user` | MySQL system user | `{{ __mysql_system_user }}` |
| `mysql_users` | The MySQL users and their privileges | `[]` |
| `mysql_enabled_on_startup` | Enable mysql service | `true` |
| `mysql_service_state` | Expected state of mysql service after configuration | `started` or `restarted` depending on context |

Most of the time `mysql_db_admin_user` is `root`, this is chosen by operating systems packagers and therefore should not be changed.

### Debian

| Variable                 | Default                            |
|--------------------------|------------------------------------|
| `__mysql_config_path`    | `/etc/mysql/{{ mysql_provider }}.conf.d/mysqld.cnf` |
| `__mysql_datadir`        | `/var/lib/mysql`                   |
| `__mysql_packages`       | `[mariadb-server]` |
| `__mysql_python_package` | `python3-pymysql`                  |
| `__mysql_service`        | `mysql`                            |
| `__mysql_socket`         | `/run/mysqld/mysqld.sock`          |
| `__mysql_system_group`   | `mysql`                            |
| `__mysql_system_user`    | `mysql`                            |

### Fedora

| Variable                 | Default                     |
|--------------------------|-----------------------------|
| `__mysql_config_path`    | `/etc/my.cnf`               |
| `__mysql_datadir`        | `/var/lib/mysql`            |
| `__mysql_packages`       | `[mariadb, mariadb-server]` |
| `__mysql_python_package` | `python3-PyMySQL`           |
| `__mysql_service`        | `mariadb`                   |
| `__mysql_socket`         | `/var/lib/mysql/mysql.sock` |
| `__mysql_system_group`   | `mysql`                     |
| `__mysql_system_user`    | `mysql`                     |

### OpenBSD

| Variable                 | Default                     |
|--------------------------|-----------------------------|
| `__mysql_config_path`    | `/etc/my.cnf`               |
| `__mysql_datadir`        | `/var/mysql`                |
| `__mysql_packages`       | `[mariadb-server]`          |
| `__mysql_python_package` | `py3-pymysql`               |
| `__mysql_service`        | `mysqld`                    |
| `__mysql_socket`         | `/var/run/mysql/mysql.sock` |
| `__mysql_system_group`   | `_mysql`                    |
| `__mysql_system_user`    | `_mysql`                    |

### `mysql_config`

`mysql_config` is a simple representation of my.cnf in YAML as a list of hashes containing only the INI section name and its content.
User provided values are combined with eventual system specific ones from this role.

Example:

```yaml
mysql_socket: /var/www/var/run/mysql/mysql.sock
mysql_config:
  - name: client-server
    content: |
      socket={{ mysql_socket }}
      port=3306
  - name: mysqld
    content: |
      bind-address=127.0.0.1
      datadir={{ mysql_datadir }}
      log-basename=mysqld
      general-log
      slow_query_log
  - name: mysqld_safe
    content: |
      syslog
```

### `mysql_provider`

This variable is used to specify the provider for `mysql`: either MariaDB (the default on most operating systems now) or Oracle's MySQL.

If you want to install the later it is mandatory to configure your package repository in advance, perhaps with a `pre_tasks` in a playbook.
Then the following variables need to be overloaded from the default value:

```yaml
# Assuming Debian Buster
mysql_packages:
  - mysql-server
mysql_service: mysql
mysql_provider: mysql
```

The variable `mysql_provider` is only used during the initial installation.

### `mysql_databases`

This variable is a list of hashes representing MySQL databases.
All of its keys are parameters to the [`mysql_db` module](https://docs.ansible.com/ansible/latest/collections/community/mysql/mysql_db_module.html).

| Variable     | Default           |
|--------------|-------------------|
| name         | mandatory         |
| collation    | `utf8_general_ci` |
| encoding     | `utf8`            |
| state        | `present`         |

### `mysql_users`

This variable is a list of hashes representing MySQL users.
All of its keys are parameters to the [`mysql_user` module](https://docs.ansible.com/ansible/latest/collections/community/mysql/mysql_user_module.html).

| Variable     | Default     |
|--------------|-------------|
| name         | mandatory   |
| host         | `localhost` |
| password     | mandatory   |
| priv         | `*.*:USAGE` |
| state        | `present`   |
| append_privs | `no`        |
| encrypted    | `no`        |

## Dependencies

None.

## Example Playbook

```yaml
- hosts: dbs
  vars:
    - mysql_db_admin_password: "{{ vaulted_mysql_db_admin_password }}"
    - mysql_users:
        - name: app_user
          password: app_password
    - mysql_databases:
        - name: app_db
  roles:
    - role: tleguern.mysql
```

## License

MIT / ISC

## Contributing

Either send [send GitHub pull requests](https://github.com/tleguern/ansible-role-mysql) or [send patches on SourceHut](https://lists.sr.ht/~tleguern/misc).

## Author Information

This role was created in 2014 by [Jeff Geerling](https://www.jeffgeerling.com/), author of [Ansible for DevOps](https://www.ansiblefordevops.com/).

Additional work by Tristan Le Guern <tleguern@bouledef.eu>.
