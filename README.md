zabbix_web
=============

Ansible role which helps to install and configure Zabbix server web
interface.

The configuration of the role is done in such way that it should not be
necessary to change the role for any kind of configuration. All can be
done either by changing role parameters or by declaring completely new
configuration as a variable. That makes this role absolutely
universal. See the examples below for more details.

Please report any issues or send PR.


Usage
-----

```
- name: Recommended installation of the Zabbix Web
  hosts: machine1
  vars:
    php_extensions_config:
      zabbix:
        PHP:
          post_max_size: 16M
          max_execution_time: 300
          max_input_time: 300
        date:
          date.timezone: UTC
    nginx_vhost_config__default:
      default:
        - server:
          - listen 80 default_server
          - server_name localhost.localdomain
          - root /usr/share/zabbix
          - index index.html index.php
          - "location ~ \\.php":
            - try_files $uri =404
            - fastcgi_split_path_info ^(.+\.php)(/.+)$
            - fastcgi_pass unix:/var/run/php-fpm/php-zabbix.socket
            - fastcgi_index index.php
            - fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name
            - fastcgi_read_timeout 600
            - include fastcgi_params
    php_fpm_pool_config:
      zabbix:
        zabbix:
          listen: /var/run/php-fpm/php-zabbix.socket
          listen.owner: root
          listen.group: apache
          listen.mode: 0660
          user: nginx
          group: nginx
          pm: dynamic
          pm.max_children: 50
          pm.start_servers: 5
          pm.min_spare_servers: 5
          pm.max_spare_servers: 35
          slowlog: /var/log/php-fpm/zabbix-slow.log
          catch_workers_output: yes
          security.limit_extensions: .php .php3 .php4 .php5
          php_admin_value[error_log]: /var/log/php-fpm/zabbix-error.log
          php_admin_flag[log_errors]: on
          php_value[session.save_handler]: files
          php_value[session.save_path]:  /var/lib/php/session/
  roles:
    - nginx
    - php
    - php_fpm
    - zabbix_web
  tasks:
    - name: Add nginx user into the apache group
      user:
        name: nginx
        groups: apache
      register: nginx_add_apache_group
    - name: Restart PHP-FPM if group added
      service:
        name: php-fpm
        state: restarted
      when: >
        nginx_add_apache_group is defined and
        nginx_add_apache_group.changed
```


Role variables
--------------

```
# Major Zabbix version (used for Zabbix YUM repo)
zabbix_major_version: 2.4

# Zabbix YUM repo URL
zabbix_yumrepo_url: http://repo.zabbix.com/zabbix/{{ zabbix_major_version }}/rhel/{{ ansible_distribution_major_version }}/$basearch/

# Additional Zabbix YUM repo params
zabbix_yumrepo_params: {}

# DB engine
zabbix_web_db_engine: "{{ zabbix_server_db_engine | default('pgsql') }}"

# Package to be installed (exact version can be specified here)
zabbix_web_pkg: zabbix-web-{{ zabbix_web_db_engine  }}

# DB type
zabbix_web_db_type: "{{
  'POSTGRESQL'
    if zabbix_web_db_engine == 'pgsql'
    else
  'MYSQL' }}"

# DB server host
zabbix_web_db_host: "{{ zabbix_server_db_host | default('localhost') }}"

# DB server port
zabbix_web_db_port: "{{ zabbix_server_db_port | default(
  5432 if zabbix_web_db_engine == 'pgsql' else 3306) }}"

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
zabbix_web_server_daemon: null
```


Dependencies
------------

- [`config_encoder_filters`](https://github.com/jtyr/ansible-config_encoder_filters)
- [`nginx`](https://github.com/jtyr/ansible-nginx) (optional)
- [`php-fpm`](https://github.com/jtyr/ansible-php_fpm) (optional)
- [`php`](https://github.com/jtyr/ansible-php) (optional)
- [`zabbix_agent`](https://github.com/jtyr/ansible-zabbix_agent) (optional)
- [`zabbix_server`](https://github.com/jtyr/ansible-zabbix_server) (optional)


License
-------

MIT


Author
------

Jiri Tyr
