zabbix_web
=============

Ansible role which helps to install and configure Zabbix server web
interface.

The configuraton of the role is done in such way that it should not be
necessary to change the role for any kind of configuration. All can be
done either by changing role parameters or by declaring completely new
configuration as a variable. That makes this role absolutely
universal. See the examples below for more details.

Please report any issues or send PR.


Usage
-----

```
# Default usage with no configuration changes
- name: Example 1
  hosts: machine1
  roles:
    - zabbix_web
```

This role requires [Config
Encoders](https://github.com/jtyr/ansible/blob/jtyr-config_encoders/lib/ansible/plugins/filter/config_encoders.py)
which must be configured in the `ansible.cfg` file like this:

```
[defaults]

filter_plugins = ./plugins/filter/
```

Where the `./plugins/filter/` containes the `config_encoders.py` file.


Role variables
--------------

```
# DB engine
zabbix_web_db_engine: "{{ zabbix_server_db_engine | default('pgsql') }}"

# DB type
zabbix_web_db_type: "{{ 'POSTGRESQL' if zabbix_web_db_engine == 'pgsql' else 'MYSQL' }}"

# DB server host
zabbix_web_db_host: "{{ zabbix_server_db_host | default('localhost') }}"

# DB server port
zabbix_web_db_port: "{{ zabbix_server_db_port | default('0') }}"

# DB name
zabbix_web_db_name: "{{ zabbix_server_db_name | default('zabbix') }}"

# DB user
zabbix_web_db_user: "{{ zabbix_server_db_user | default('zabbix') }}"

# DB password
zabbix_web_db_password: "{{ zabbix_server_db_password | default('zabbix') }}"

# Used only for PostgreSQL
zabbix_web_db_schema: ''

# Zabbix server host
zabbix_web_server_host: localhost

# Zabbix server port
zabbix_web_server_port: 10051

# Zabbix server name
zabbix_web_server_name: ''

# Enable maintenance mode
zabbix_web_maintenance_mode: false

# Maintenance access IP range
zabbix_web_maintenance_range:
  - 127.0.0.1

# Maintenance warning message
zabbix_web_maintenance_message: Zabbix is under maintenance.

# Web server daemon (e.g. apache, httpd, nginx)
zabbix_web_server_daemon: none
```


Dependencies
------------

- [`yumrepo`](https://raw.githubusercontent.com/jtyr/ansible-modules-extras/jtyr-yumrepo_params/packaging/os/yumrepo.py)
  module (put it into the `library` directory)
- [Config Encoders](https://github.com/jtyr/ansible/blob/jtyr-config_encoders/lib/ansible/plugins/filter/config_encoders.py)


License
-------

MIT


Author
------

Jiri Tyr
