zabbix_web
==========

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
- name: Installation of Zabbix Web 2.4 on RHEL/CentOS 6 or 2.4/3.x on RHEL/CentOS 7
  hosts: machine1
  vars:
    # Nginx configuration
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

    # PHP configuration
    php_extensions_config:
      zabbix:
        PHP:
          post_max_size: 16M
          max_execution_time: 300
          max_input_time: 300
        date:
          date.timezone: UTC

    # PHP FPM configuration
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
          php_value[session.save_path]: /var/lib/php/session/

    # Zabbix Web configuration
    #zabbix_major_version: 3.2
    zabbix_web_server_daemon: php-fpm

    # Add nginx user into the apache group
    zabbix_web_server_user: nginx
    zabbix_web_server_group: apache
  roles:
    - postgresql
    - zabbix_server
    - nginx
    - php
    - php_fpm
    - zabbix_web

- name: Installation of Zabbix Web 3.x on RHEL/CentOS 6
  hosts: machine2
  vars:
    # Configure Nginx
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
            - fastcgi_pass unix:/opt/rh/php54/root/var/run/php-fpm/php-zabbix.socket
            - fastcgi_index index.php
            - fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name
            - fastcgi_read_timeout 600
            - include fastcgi_params

    # Use PHP 5.4
    php_scl_yumrepo_install: yes
    php_pkg: php54
    php_config_file: /opt/rh/php54/root/etc/php.ini
    php_config_dir: /opt/rh/php54/root/etc/php.d

    # Configure PHP
    php_extensions_prefix: php54-php
    php_extensions_pkgs:
      - bcmath
      - pgsql
      - mbstring
      - gd
    php_extensions_config:
      zabbix:
        PHP:
          post_max_size: 16M
          max_execution_time: 300
          max_input_time: 300
        date:
          date.timezone: UTC

    # Use PHP FPM 5.4
    php_fpm_pkg: php54-php-fpm
    php_fpm_service: php54-php-fpm
    php_fpm_config_file: /opt/rh/php54/root/etc/php-fpm.conf
    php_fpm_config_dir: /opt/rh/php54/root/etc/php-fpm.d
    php_fpm_sysconfig_file: /opt/rh/php54/root/etc/sysconfig/php-fpm

    # Configure PHP FPM
    php_fpm_pool_config:
      zabbix:
        zabbix:
          listen: /opt/rh/php54/root/var/run/php-fpm/php-zabbix.socket
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
          slowlog: /opt/rh/php54/root/var/log/php-fpm/zabbix-slow.log
          catch_workers_output: yes
          security.limit_extensions: .php .php3 .php4 .php5
          php_admin_value[error_log]: /opt/rh/php54/root/var/log/php-fpm/zabbix-error.log
          php_admin_flag[log_errors]: on
          php_value[session.save_handler]: files
          php_value[session.save_path]: /opt/rh/php54/root/usr/share/xsessions

    # For Zabbix 3.0
    zabbix_major_version: 3.0

    # For Zabbix 3.2
    #zabbix_major_version: 3.2
    #zabbix_web_yumrepo_params:
    #  name: zabbix-web
    #zabbix_web_yumrepo_url: http://repo.zabbix.com/zabbix/{{ zabbix_major_version }}/rhel/$releasever/$basearch/deprecated/
    #zabbix_server_yumrepo_params: "{{ zabbix_web_yumrepo_params }}"
    #zabbix_server_yumrepo_url: "{{ zabbix_web_yumrepo_url }}"

    # Zabbix Web configuration
    zabbix_web_server_daemon: php54-php-fpm

    # Add nginx user into the apache group
    zabbix_web_server_user: nginx
    zabbix_web_server_group: apache
  roles:
    - postgresql
    - zabbix_server
    - nginx
    - php
    - php_fpm
    - zabbix_web
```


Role variables
--------------

```
# Major Zabbix version (used for Zabbix YUM repo)
zabbix_web_major_version: "{{ zabbix_major_version | default(2.4) }}"

# Zabbix YUM repo URL
zabbix_web_yumrepo_url: "{{ zabbix_yumrepo_url | default('http://repo.zabbix.com/zabbix/' ~ zabbix_web_major_version ~ '/rhel/' ~ ansible_distribution_major_version ~ '/$basearch/') }}"

# Additional Zabbix YUM repo params
zabbix_web_yumrepo_params: "{{ zabbix_yumrepo_params | default({}) }}"

# Session directory for PHP 5.4
zabbix_web_php_session_dir: /opt/rh/php54/root/usr/share/xsessions

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

# Web server user (e.g. nginx)
zabbix_web_server_user: null

# Web server user (e.g. apache)
zabbix_web_server_group: null
```


Dependencies
------------

- [`config_encoder_filters`](https://github.com/jtyr/ansible-config_encoder_filters)
- [`nginx`](https://github.com/jtyr/ansible-nginx) (optional)
- [`php-fpm`](https://github.com/jtyr/ansible-php_fpm) (optional)
- [`php`](https://github.com/jtyr/ansible-php) (optional)
- [`postgresql`](https://github.com/jtyr/ansible-postgresql) (optional)
- [`zabbix_agent`](https://github.com/jtyr/ansible-zabbix_agent) (optional)
- [`zabbix_proxy`](https://github.com/jtyr/ansible-zabbix_proxy) (optional)
- [`zabbix_server`](https://github.com/jtyr/ansible-zabbix_server) (optional)


License
-------

MIT


Author
------

Jiri Tyr
