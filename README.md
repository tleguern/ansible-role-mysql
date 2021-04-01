# Ansible Role: MySQL

[![builds.sr.ht status](https://builds.sr.ht/~tleguern/ansible-role-mysql.svg)](https://builds.sr.ht/~tleguern/ansible-role-mysql?)

Yet another MySQL role, this time focussed on OpenBSD and based on geerlingguy's.
The focus is on simplicity: some variables are renamed to prevent misunderstanding and the replication parameters are left for an additional role.

Automatic testing is provided using molecule's delegated driver and <https://builds.sr.ht>.

## Requirements

None.

## Role Variables

| Variable | Description | Default |
|----------|-------------|---------|
| `mysql_databases` | The MySQL databases to create | `[]` |
| `mysql_config_path` | Path of the main configuration file | `{{ __mysql_config_path }}` |
| `mysql_db_admin_password_update` | Whether to force update the MySQL root user's password | `false` |
| `mysql_db_admin_user` | MySQL default administrator account | `root` |
| `mysql_db_admin_password` | MySQL administrator password  | mandatory |
| `mysql_users` | The MySQL users and their privileges | `[]` |
| `mysql_extra_config` | Extra configuration rules for my.cnf. See bellow. | `[]` |
| `mysql_datadir` | Directory where MySQL data are stored | `{{ __mysql_datadir }}` |
| `mysql_packages` | A list of packages | `{{ __mysql_packages }}` |
| `mysql_service` | MySQL service name | `{{ __mysql_service }}` |
| `mysql_system_group` | MySQL system group | `{{ __mysql_system_group }}` |
| `mysql_system_user` | MySQL system user | `{{ __mysql_system_user }}` |

Most of the time `mysql_db_admin_user` is `root`, this is chosen by operating systems packagers and therefore should not be changed.

### Debian

| Variable | Default |
|----------|---------|
| `__mysql_config_path` | `/etc/mysql/my.cnf` |
| `__mysql_datadir` | `/var/lib/mysql` |
| `__mysql_packages` | `[mariadb-client, mariadb-server, python3-pymysql]` |
| `__mysql_service` | `mysql` |
| `__mysql_system_group` | `mysql` |
| `__mysql_system_user` | `mysql` |

### OpenBSD

| Variable | Default |
|----------|---------|
| `__mysql_config_path` | `/etc/my.cnf` |
| `__mysql_datadir` | `/var/mysql` |
| `__mysql_packages` | `[mariadb-client, mariadb-server, py3-pymysql]` |
| `__mysql_service` | `mysqld` |
| `__mysql_system_group` | `_mysql` |
| `__mysql_system_user` | `_mysql` |

### `mysql_extra_config`

`mysql_extra_config` is YAML representation of my.cnf, as hash of hashes.
Its value is combined with the default minimal configuration provided for each supported operating system.

Example:

```yml
mysql_extra_config:
  client-server:
    socket: /var/www/var/run/mysql/mysql.sock
```

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
    - role: ansible-role-mysql
```

## License

MIT / BSD

## Contributing

Either send [send GitHub pull requests](https://github.com/tleguern/ansible-role-mysql) or [send patches on SourceHut](https://lists.sr.ht/~tleguern/misc).

## Author Information

This role was created in 2014 by [Jeff Geerling](https://www.jeffgeerling.com/), author of [Ansible for DevOps](https://www.ansiblefordevops.com/).

Additional work by Tristan Le Guern <tleguern@bouledef.eu>.
