# Laporan Praktikum UAS - Sistem Administrasi Server 
Disusun oleh :
1. Chintya Tribhuana Utami (1202190041)
2. Nur Wulan Maudini (1202190002)
#
Praktikum dilaksanakan berdasarkan keadaan yang tertera pada FINAL PROJECT, dan dapat diakses [Klik disini.](https://github.com/aldonesia/Sistem-Administrasi-Server-2021/tree/master/Final%20Project)
#

Buat LXC yang terdiri dari :

- 6 instance LXC ubuntu 20.04 PHP 7.4
```
sudo lxc-create -n lxc_php7_1 -t download -- --dist ubuntu --release focal --arch amd64 --force-cache --no-validate --server images.linuxcontainers.org

sudo lxc-create -n lxc_php7_2 -t download -- --dist ubuntu --release focal --arch amd64 --force-cache --no-validate --server images.linuxcontainers.org

sudo lxc-create -n lxc_php7_3 -t download -- --dist ubuntu --release focal --arch amd64 --force-cache --no-validate --server images.linuxcontainers.org

sudo lxc-create -n lxc_php7_4 -t download -- --dist ubuntu --release focal --arch amd64 --force-cache --no-validate --server images.linuxcontainers.org

sudo lxc-create -n lxc_php7_5 -t download -- --dist ubuntu --release focal --arch amd64 --force-cache --no-validate --server images.linuxcontainers.org

sudo lxc-create -n lxc_php7_6 -t download -- --dist ubuntu --release focal --arch amd64 --force-cache --no-validate --server images.linuxcontainers.org
```

- 2 instance LXC debian 10 PHP 5.6
```
sudo lxc-create -n lxc_php5_1 -t download -- --dist debian --release buster --arch amd64 --force-cache --no-validate --server images.linuxcontainers.org

sudo lxc-create -n lxc_php5_2 -t download -- --dist debian --release buster --arch amd64 --force-cache --no-validate --server images.linuxcontainers.org
```

- 1 instance LXC debian 10 mariadb server
```
sudo lxc-create -n lxc_mariadb -t download -- --dist debian --release buster --arch amd64 --force-cache --no-validate --server images.linuxcontainers.org
```

Setting autostart, dan IP setiap lxc, Berikut ini adalah list lxcnya

![A1](asset/Picture1.png)

Setting sites-available pada vm menggunakan nama kelompok5.fpsas dan symlink ke sites-enabled
```
  upstream laravel {
        least_conn;
        server lxc_php7_1.dev;
        server lxc_php7_4_laravel.dev;
        server lxc_php7_6_laravel.dev;
        server lxc_php7_2_laravel.dev;
  }
  upstream yii {
        server lxc_php7_2_yii.dev weight=2;
        server lxc_php7_1_yii.dev weight=3;
        server lxc_php7_4_yii.dev weight=4;
        server lxc_php7_5_yii.dev weight=1;
        server lxc_php7_6_yii.dev weight=6;
   }
  upstream ci {
        server lxc_php5_1.dev;
        server lxc_php5_2.dev;
  }
  upstream wp {
        ip_hash;
        server lxc_php7_3_wp.dev;
        server lxc_php7_2.dev;
        server lxc_php7_4_wp.dev;
        server lxc_php7_5_wp.dev;
  }

  server {
          listen 80;
          listen [::]:80;

          server_name kelompok5.fpsas;

          root /var/www/html;
          index index.html;

          location /app {
                  rewrite /app/?(.*)$ /$1 break;
                  #proxy_pass http://lxc_php5_1.dev;
                  proxy_pass http://ci;
          }

          location /product {
                  rewrite /product/?(.*)$ /$1 break;
                  #proxy_pass http://lxc_php7_6_yii.dev;
                  proxy_pass http://yii;
          }

          location /phpmyadmin {
                  rewrite /phpmyadmin/?(.*)$ /$1 break;
                  proxy_pass http://lxc_db_server.dev;
          }

          location / {
                  #rewrite /php7/?(.*)$ /$1 break;
                  #proxy_pass http://lxc_php7_1.dev;
                  proxy_pass http://laravel;
          }

  }

  server {
         listen 80;

         server_name news.kelompok5.fpsas;

         root /var/www/html;
         index index.html;

         location / {
                  #rewrite /php7/?(.*)$ /$1 break;
                  #proxy_pass http://lxc_php7_3_wp.dev;
                  proxy_pass http://wp;
         }
  }

```

Daftar semua lxc pada /etc/hosts di vm
```
127.0.0.1 localhost
127.0.1.1 sas01
127.0.0.1 kelompok5.fpsas news.kelompok5.fpsas

#laravel
10.0.3.101  lxc_php7_1.dev
10.0.3.102  lxc_php7_2_laravel.dev
10.0.3.104  lxc_php7_4_laravel.dev
10.0.3.106  lxc_php7_6_laravel.dev

#yii
10.0.3.102  lxc_php7_2_yii.dev
10.0.3.101  lxc_php7_1_yii.dev
10.0.3.104  lxc_php7_4_yii.dev
10.0.3.105  lxc_php7_5_yii.dev
10.0.3.106  lxc_php7_6_yii.dev

#wp
10.0.3.103  lxc_php7_3_wp.dev
10.0.3.102  lxc_php7_2.dev
10.0.3.104  lxc_php7_4_wp.dev
10.0.3.105  lxc_php7_5_wp.dev

#ci
10.0.3.111  lxc_php5_1.dev
10.0.3.112  lxc_php5_2.dev

#db
10.0.3.200  lxc_db_server.dev
#10.0.3.201  lxc_db_server_coba.dev

# The following lines are desirable for IPv6 capable hosts
::1     ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
```

Deploy website menggunakan ansible:

Setting hosts ansible

![A1](asset/Picture2.png)

# Install Mariadb
Nano install-mariadb.yml
```
- hosts: database
  vars:
    username: 'admin'
    password: 'chintya'
    domain: 'lxc_db_server.dev'
  roles:
    - db
    - pma
```

# Roles db

![A1](asset/Picture3.png)
```
---
- name: restart mysql
  become: yes
  become_user: root
  become_method: su
  action: service name=mysql state=restarted 
```

![A1](asset/Picture4.png)
```
---
- name: delete apt chache
  become: yes
  become_user: root
  become_method: su
  command: rm -vf /var/lib/apt/lists/*

- name: install mariadb
  become: yes
  become_user: root
  become_method: su
  apt: name={{ item }} state=latest update_cache=true
  with_items:
   - python
   - mariadb-server
   - mariadb-client
   - python-mysqldb
   - python-pymysql

- name: Stop MySQL
  service: name=mysqld state=stopped

- name: set environment variables
  shell: systemctl set-environment MYSQLD_OPTS="--skip-grant-tables"

- name: Start MySQL
  service: name=mysqld state=started

- name: sql query
  command:  mysql -u root --execute="UPDATE mysql.user SET authentication_string = PASSWORD('{{ password }}') WHERE User = 'root' AND Host = 'localhost';"

- name: sql query flush
  command:  mysql -u root --execute="FLUSH PRIVILEGES"

- name: Stop MySQL
  service: name=mysqld state=stopped

- name: unset environment variables
  shell: systemctl unset-environment MYSQLD_OPTS

- name: Start MySQL
  service: name=mysqld state=started

- name: Create user for mysql
  command:  mysql -u root --execute="CREATE USER IF NOT EXISTS '{{ username }}'@'localhost' IDENTIFIED BY '{{ password }}';"

- name: GRANT ALL PRIVILEGES to user {{username}}
  command:  mysql -u root --execute="GRANT ALL PRIVILEGES ON * . * TO '{{ username }}'@'localhost';"

- name: Create user for remote mysql
  command:  mysql -u root --execute="CREATE USER IF NOT EXISTS '{{ username }}'@'%' IDENTIFIED BY '{{ password }}';"

- name: GRANT ALL PRIVILEGES to remote user {{username}}
  command:  mysql -u root --execute="GRANT ALL PRIVILEGES ON * . * TO '{{ username }}'@'%';"

- name: sql query flush
  command:  mysql -u root --execute="FLUSH PRIVILEGES"

- name: Create DB Landing
  command:  mysql -u root --execute="CREATE DATABASE IF NOT EXISTS `landing`;"

- name: Create DB product
  command:  mysql -u root --execute="CREATE DATABASE IF NOT EXISTS `product`;"

- name: Create DB app
  command:  mysql -u root --execute="CREATE DATABASE IF NOT EXISTS `app`;"

- name: Create DB news
  command:  mysql -u root --execute="CREATE DATABASE IF NOT EXISTS `news`;"

- name: Copy .my.cnf file with root password credentials
  template:
    src=templates/my.cnf
    dest=/etc/mysql/mariadb.conf.d/50-server.cnf
  notify: restart mysql
```

![A1](asset/Picture5.png)
```
#
# These groups are read by MariaDB server.
# Use it for options that only the server (but not clients) should see
#
# See the examples of server my.cnf files in /usr/share/mysql

# this is read by the standalone daemon and embedded servers
[server]

# this is only for the mysqld standalone daemon
[mysqld]

#
# * Basic Settings
#
user                    = mysql
pid-file                = /run/mysqld/mysqld.pid
socket                  = /run/mysqld/mysqld.sock
port                   = 3306
basedir                 = /usr
datadir                 = /var/lib/mysql
tmpdir                  = /tmp
lc-messages-dir         = /usr/share/mysql
skip-external-locking

# Instead of skip-networking the default is now to listen only on
# localhost which is more compatible and is not less secure.
bind-address            = 0.0.0.0

#
# * Fine Tuning
#
key_buffer_size        = 16M
max_allowed_packet     = 16M
thread_stack           = 192K
thread_cache_size      = 8
# This replaces the startup script and checks MyISAM tables if needed
# the first time they are touched
myisam_recover_options = BACKUP
#max_connections        = 100
#table_cache            = 64
#thread_concurrency     = 10

#
# * Query Cache Configuration
#
query_cache_limit      = 1M
query_cache_size        = 16M

#
# * Logging and Replication
#
# Both location gets rotated by the cronjob.
# Be aware that this log type is a performance killer.
# As of 5.1 you can enable the log at runtime!
#general_log_file       = /var/log/mysql/mysql.log
#general_log            = 1
#
# Error log - should be very few entries.
#
log_error = /var/log/mysql/error.log
#
# Enable the slow query log to see queries with especially long duration
#slow_query_log_file    = /var/log/mysql/mariadb-slow.log
#long_query_time        = 10
#log_slow_rate_limit    = 1000
#log_slow_verbosity     = query_plan
#log-queries-not-using-indexes
#
# The following can be used as easy to replay backup logs or for replication.
# note: if you are setting up a replication slave, see README.Debian about
#       other settings you may need to change.
#server-id              = 1
#log_bin                = /var/log/mysql/mysql-bin.log
expire_logs_days        = 10
max_binlog_size        = 100M
#binlog_do_db           = include_database_name
#binlog_ignore_db       = exclude_database_name

#
# * Security Features
#
# Read the manual, too, if you want chroot!
#chroot = /var/lib/mysql/
#
# For generating SSL certificates you can use for example the GUI tool "tinyca".
#
#ssl-ca = /etc/mysql/cacert.pem
#ssl-cert = /etc/mysql/server-cert.pem
#ssl-key = /etc/mysql/server-key.pem
#
# Accept only connections using the latest and most secure TLS protocol version.
# ..when MariaDB is compiled with OpenSSL:
#ssl-cipher = TLSv1.2
# ..when MariaDB is compiled with YaSSL (default in Debian):
#ssl = on

#
# * Character sets
#
# MySQL/MariaDB default is Latin1, but in Debian we rather default to the full
# utf8 4-byte character set. See also client.cnf
#
character-set-server  = utf8mb4
collation-server      = utf8mb4_general_ci

#
# * InnoDB
#
# InnoDB is enabled by default with a 10MB datafile in /var/lib/mysql/.
# Read the manual for more InnoDB related options. There are many!

#
# * Unix socket authentication plugin is built-in since 10.0.22-6
#
# Needed so the root database user can authenticate without a password but
# only when running as the unix root user.
#
# Also available for other users if required.
# See https://mariadb.com/kb/en/unix_socket-authentication-plugin/

# this is only for embedded server
[embedded]

# This group is only read by MariaDB servers, not by MySQL.
# If you use the same .cnf file for MySQL and MariaDB,
# you can put MariaDB-only options here
[mariadb]

# This group is only read by MariaDB-10.3 servers.
# If you use the same .cnf file for MariaDB of different versions,
# use this group for options that older servers don't understand
[mariadb-10.3]
```

# Roles pma
![A1](asset/Picture6.png)
```
---
 - name: stop apache2
   become: yes
   become_user: root
   become_method: su
   action: service name=apache2 state=stopped

 - name: restart nginx
   become: yes
   become_user: root
   become_method: su
   action: service name=nginx state=restarted

 - name: restart php
   become: yes
   become_user: root
   become_method: su
   action: service name=php7.2-fpm state=restarted
```

![A1](asset/Picture7.png)
```
---
- name: delete apt chache
  become: yes
  become_user: root
  become_method: su
  command: rm -vf /var/lib/apt/lists/*

- name: install nginx phpmyadmin
  become: yes
  become_user: root
  become_method: su
  apt: name={{ item }} state=latest update_cache=true
  with_items:
    - curl
    - nginx
    - nginx-extras
    - php7.3-fpm
    - php-mbstring
    - php-zip
    - php-gd
    - php-json
    - php-curl
    - wget
    - php7.3-xml
    - php7.3-mysqli

- name: wget phpmyadmin
  shell: wget https://files.phpmyadmin.net/phpMyAdmin/5.1.1/phpMyAdmin-5.1.1-all-languages.tar.gz

- name: tar latest.tar.gz
  shell: tar -zxvf phpMyAdmin-5.1.1-all-languages.tar.gz

- name: move phpmyadmin
  shell: sudo mv phpMyAdmin-5.1.1-all-languages /usr/share/phpmyadmin

- name: Create a tmp directory for phpMyAdmin and then change the permission.
  command: mkdir /usr/share/phpmyadmin/tmp

- name: change permission storage
  command: chmod 777 /usr/share/phpmyadmin/tmp

- name: Set the ownership of the phpMyAdmin directory.
  command: chown -R www-data:www-data /usr/share/phpmyadmin

- name: copy config
  template:
    src=templates/conf.inc.php
    dest=/usr/share/phpmyadmin/config.inc.php

- name: enable module php mbstring
  command: phpenmod mbstring
  notify:
    - restart php

- name: Copy pma.local
  template:
    src=templates/pma.local
    dest=/etc/nginx/sites-available/{{ domain }}
  vars:
    servername: '{{ domain }}'

- name: Symlink pma.local
  command: ln -sfn /etc/nginx/sites-available/{{ domain }} /etc/nginx/sites-enabled/{{ domain }}
  notify:
    - stop apache2
    - restart nginx

- name: Write {{ domain }} to /etc/hosts
  lineinfile:
    dest: /etc/hosts
    regexp: '.*{{ domain }}$'
    line: "127.0.0.1 {{ domain }}"
    state: present
```

![A1](asset/Picture8.png)
```
<?php
/**
 * phpMyAdmin sample configuration, you can use it as base for
 * manual configuration. For easier setup you can use setup/
 *
 * All directives are explained in documentation in the doc/ folder
 * or at <https://docs.phpmyadmin.net/>.
 */

declare(strict_types=1);

/**
 * This is needed for cookie based authentication to encrypt password in
 * cookie. Needs to be 32 chars long.
 */
$cfg['blowfish_secret'] = 'KELOMPOK5SISTEMADMINISTRASISERVE'; /* YOU MUST FILL IN THIS FOR COOKIE AUTH! */

/**
 * Servers configuration
 */
$i = 0;

/**
 * First server
 */
$i++;
/* Authentication type */
$cfg['Servers'][$i]['auth_type'] = 'cookie';
/* Server parameters */
$cfg['Servers'][$i]['host'] = 'localhost';
$cfg['Servers'][$i]['compress'] = false;
$cfg['Servers'][$i]['AllowNoPassword'] = false;

/**
 * phpMyAdmin configuration storage settings.
 */

/* User used to manipulate with storage */
$cfg['Servers'][$i]['controlhost'] = 'localhost';
$cfg['Servers'][$i]['controlport'] = '3306';
$cfg['Servers'][$i]['controluser'] = '{{username}}';
$cfg['Servers'][$i]['controlpass'] = '{{password}}';

/* Storage database and tables */
$cfg['Servers'][$i]['pmadb'] = 'phpmyadmin';
$cfg['Servers'][$i]['bookmarktable'] = 'pma__bookmark';
$cfg['Servers'][$i]['relation'] = 'pma__relation';
$cfg['Servers'][$i]['table_info'] = 'pma__table_info';
$cfg['Servers'][$i]['table_coords'] = 'pma__table_coords';
$cfg['Servers'][$i]['pdf_pages'] = 'pma__pdf_pages';
$cfg['Servers'][$i]['column_info'] = 'pma__column_info';
$cfg['Servers'][$i]['history'] = 'pma__history';
$cfg['Servers'][$i]['table_uiprefs'] = 'pma__table_uiprefs';
$cfg['Servers'][$i]['tracking'] = 'pma__tracking';
$cfg['Servers'][$i]['userconfig'] = 'pma__userconfig';
$cfg['Servers'][$i]['recent'] = 'pma__recent';
$cfg['Servers'][$i]['favorite'] = 'pma__favorite';
$cfg['Servers'][$i]['users'] = 'pma__users';
$cfg['Servers'][$i]['usergroups'] = 'pma__usergroups';
$cfg['Servers'][$i]['navigationhiding'] = 'pma__navigationhiding';
$cfg['Servers'][$i]['savedsearches'] = 'pma__savedsearches';
$cfg['Servers'][$i]['central_columns'] = 'pma__central_columns';
$cfg['Servers'][$i]['designer_settings'] = 'pma__designer_settings';
$cfg['Servers'][$i]['export_templates'] = 'pma__export_templates';

/**
 * End of servers configuration
 */

/**
 * Directories for saving/loading files from server
 */
$cfg['UploadDir'] = '';
$cfg['SaveDir'] = '';

/**
 * Whether to display icons or text or both icons and text in table row
 * action segment. Value can be either of 'icons', 'text' or 'both'.
 * default = 'both'
 */
//$cfg['RowActionType'] = 'icons';

/**
 * Defines whether a user should be displayed a "show all (records)"
 * button in browse mode or not.
 * default = false
 */
//$cfg['ShowAll'] = true;

/**
 * Number of rows displayed when browsing a result set. If the result
 * set contains more rows, "Previous" and "Next".
 * Possible values: 25, 50, 100, 250, 500
 * default = 25
 */
//$cfg['MaxRows'] = 50;

/**
 * Disallow editing of binary fields
 * valid values are:
 *   false    allow editing
 *   'blob'   allow editing except for BLOB fields
 *   'noblob' disallow editing except for BLOB fields
 *   'all'    disallow editing
 * default = 'blob'
 */
//$cfg['ProtectBinary'] = false;

/**
 * Default language to use, if not browser-defined or user-defined
 * (you find all languages in the locale folder)
 * uncomment the desired line:
 * default = 'en'
 */
//$cfg['DefaultLang'] = 'en';
//$cfg['DefaultLang'] = 'de';

/**
 * How many columns should be used for table display of a database?
 * (a value larger than 1 results in some information being hidden)
 * default = 1
 */
//$cfg['PropertiesNumColumns'] = 2;

/**
 * Set to true if you want DB-based query history.If false, this utilizes
 * JS-routines to display query history (lost by window close)
 *
 * This requires configuration storage enabled, see above.
 * default = false
 */
//$cfg['QueryHistoryDB'] = true;

/**
 * When using DB-based query history, how many entries should be kept?
 * default = 25
 */
//$cfg['QueryHistoryMax'] = 100;

/**
 * Whether or not to query the user before sending the error report to
 * the phpMyAdmin team when a JavaScript error occurs
 *
 * Available options
 * ('ask' | 'always' | 'never')
 * default = 'ask'
 */
//$cfg['SendErrorReports'] = 'always';

/**
 * You can find more configuration options in the documentation
 * in the doc/ folder or at <https://docs.phpmyadmin.net/>.
 */
```

![A1](asset/Picture9.png)
```
server {
    listen 80;

    server_name {{servername}};

    root /usr/share/phpmyadmin;

    index index.php;

    location / {

         try_files $uri $uri/ @phpmyadmin;

    }

    location @phpmyadmin {
            fastcgi_pass unix:/run/php/php7.3-fpm.sock;   #Sesuaikan dengan versi PHP
            fastcgi_param SCRIPT_FILENAME /usr/share/phpmyadmin/index.php;
            include /etc/nginx/fastcgi_params;
            fastcgi_param SCRIPT_NAME /index.php;

    }

    location ~ \.php$ {

            fastcgi_pass unix:/run/php/php7.3-fpm.sock;  #Sesuaikan dengan versi PHP
            fastcgi_index index.php;
            fastcgi_param SCRIPT_FILENAME /usr/share/phpmyadmin$fastcgi_script_name;
            include fastcgi_params;

    }
}
```

Jalankan install-mariadb.yml, hasilnya seperti ini:

![A1](asset/Picture10.jpg)

Install Laravel
Install-laravel.yml
```
- hosts: laravel
  vars:
    domain: 'lxc_php7_1.dev'
  roles:
    - laravel

- hosts: lxc_php7_2_laravel
  vars:
    domain: 'lxc_php7_2_laravel.dev'
  roles:
    - laravel

- hosts: lxc_php7_4_laravel
  vars:
    domain: 'lxc_php7_4_laravel.dev'
  roles:
    - laravel

- hosts: lxc_php7_6_laravel
  vars:
    domain: 'lxc_php7_6_laravel.dev'
  roles:
    - laravel
```

![A1](asset/Picture11.png)
```
---
- name: stop apache2
  become: yes
  become_user: root
  become_method: su
  action: service name=apache2 state=stopped

- name: restart nginx
  become: yes
  become_user: root
  become_method: su
  action: service name=nginx state=restarted

- name: restart php
  become: yes
  become_user: root
  become_method: su
  action: service name=php7.2-fpm state=restarted
```

![A1](asset/Picture12.png)
```
---
- name: delete apt chache
  become: yes
  become_user: root
  become_method: su
  command: rm -vf /var/lib/apt/lists/*

- name: install requirement dpkg to install php7.4
  become: yes
  become_user: root
  become_method: su
  apt: name={{ item }} state=latest update_cache=true
  with_items:
    - ca-certificates
    - apt-transport-https
    - curl
    - python-apt
    - software-properties-common

- name: Add Php Repository 7.4
  apt_repository:
    repo: "ppa:ondrej/php"
    state: present
    filename: php.list
    update_cache: true

- name: install nginx php7.4
  become: yes
  become_user: root
  become_method: su
  apt: name={{ item }} state=latest update_cache=true
  with_items:
    - git
    - nginx
    - nginx-extras
    - php7.4
    - php7.4-fpm
    - php7.4-curl
    - php7.4-mbstring
    - php7.4-xml
    - php7.4-gd
    - php7.4-opcache
    - php7.4-zip
    - php7.4-json
    - php7.4-cli

- name: Download and install Composer
  shell: curl -sS https://getcomposer.org/installer | php
  args:
    chdir: /usr/src/
    creates: /usr/local/bin/composer
    warn: false
  become: yes

- name: Add Composer to global path
  copy:
    dest: /usr/local/bin/composer
    group: root
    mode: '0755'
    owner: root
    src: /usr/src/composer.phar
    remote_src: yes
  become: yes

- name: Composer create project
  composer:
    command: create-project
    arguments: laravel/laravel landing
    working_dir: /var/www/html
    prefer_dist: yes
  environment:
    COMPOSER_NO_INTERACTION: "1"

- name: set APP_URL
  lineinfile:
    dest=/var/www/html/landing/.env
    regexp='^APP_URL='
    line=APP_URL=http://kelompok5.fpsas

- name: set DB_HOST
  lineinfile:
    dest=/var/www/html/landing/.env
    regexp='^DB_HOST='
    line=DB_HOST=10.0.3.200

- name: set DB_DATABASE
  lineinfile:
    dest=/var/www/html/landing/.env
    regexp='^DB_DATABASE='
    line=DB_DATABASE=landing

- name: set DB_USERNAME
  lineinfile:
     dest=/var/www/html/landing/.env
     regexp='^DB_USERNAME='
     line=DB_USERNAME=admin

- name: set DB_PASSWORD
  lineinfile:
    dest=/var/www/html/landing/.env
    regexp='^DB_PASSWORD='
    line=DB_PASSWORD=chintya

- name: change permission storage
  command: chmod -R 777 /var/www/html/landing/storage

- name: Copy landing.conf
  template:
    src=templates/landing
    dest=/etc/nginx/sites-available/{{ domain }}
  vars:
      servername: '{{ domain }}'

- name: www.conf
  lineinfile:
     dest: /etc/php/7.4/fpm/pool.d/www.conf
     regexp: '^listen = /run/php/php7.4-fpm.sock'
     line: listen = 127.0.0.1:9001
     state: present

- name: Delete another nginx config
  become: yes
  become_user: root
  become_method: su
  command: rm -f /etc/nginx/sites-enabled/*

- name: Symlink landing.conf
  command: ln -sfn /etc/nginx/sites-available/{{ domain }} /etc/nginx/sites-enabled/{{ domain }}
  notify:
    restart nginx

- name: Write {{ domain }} to /etc/hosts
  lineinfile:
    dest: /etc/hosts
    regexp: '.*{{ domain }}$'
    line: "127.0.0.1   {{ domain }}"
    state: present
```

![A1](asset/Picture13.png)
```
server {

    listen 80;

    server_name {{ domain }};

    root /var/www/html/landing/public;
    index index.php index.html index.htm;

    charset utf-8;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location = /favicon.ico { access_log off; log_not_found off; }
    location = /robots.txt  { access_log off; log_not_found off; }

    error_page 404 /index.php;

    location ~ \.php$ {
        #fastcgi_pass unix:/var/run/php/php7.4-fpm.sock;
        fastcgi_pass 127.0.0.1:9001;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $realpath_root$fastcgi_script_name;
        include fastcgi_params;
    }

    location ~ /\.(?!well-known).* {
        deny all;
    }
}
```
Jalankan install-laravel.yml, hasilnya seperti ini:
![A1](asset/Picture15.png)

# Install Codeigniter
Install-ci.yml
```
- hosts: ci
  vars:
    git_url: 'https://github.com/aldonesia/sas-ci'
    destdir: '/var/www/html/ci'
    domain: 'lxc_php5_1.dev'
  roles:
    - ci

- hosts: lxc_php5_2
  vars:
    git_url: 'https://github.com/aldonesia/sas-ci'
    destdir: '/var/www/html/ci'
    domain: 'lxc_php5_2.dev'
  roles:
    - ci
```
![A1](asset/Picture14.png)
```
---
- name: restart nginx
  become: yes
  become_user: root
  become_method: su
  action: service name=nginx state=restarted

- name: restart php
  become: yes
  become_user: root
  become_method: su
  action: service name=php5.6-fpm state=restarted
```

![A1](asset/Picture16.png)

```
---
- name: delete apt chache
  become: yes
  become_user: root
  become_method: su
  command: rm -vf /var/lib/apt/lists/*

- name: install requirement dpkg to install php5
  become: yes
  become_user: root
  become_method: su
  apt: name={{ item }} state=latest update_cache=true
  with_items:
    - ca-certificates
    - apt-transport-https
    - wget
    - curl
    - python-apt
    - software-properties-common
    - git

- name: Add key
  apt_key:
    url: https://packages.sury.org/php/apt.gpg
    state: present

- name: Add Php Repository
  apt_repository:
      repo: "deb https://packages.sury.org/php/ buster main"
      state: present
      filename: php.list
      update_cache: true

- name: install nginx php5
  become: yes
  become_user: root
  become_method: su
  apt: name={{ item }} state=latest update_cache=true
  with_items:
    - nginx
    - nginx-extras
    - php5.6
    - php5.6-fpm
    - php5.6-common
    - php5.6-cli
    - php5.6-curl
    - php5.6-mbstring
    - php5.6-mysqlnd
    - php5.6-xml

- name: Git clone repo sas-ci
  become: yes
  git:
    repo: '{{ git_url }}'
    dest: "{{ destdir }}"

- name: Copy app.conf
  template:
    src=templates/app.conf
    dest=/etc/nginx/sites-available/{{ domain }}
  vars:
    servername: '{{ domain }}'

- name: Delete another nginx config
  become: yes
  become_user: root
  become_method: su
  command: rm -f /etc/nginx/sites-enabled/*

- name: Symlink app.conf
  command: ln -sfn /etc/nginx/sites-available/{{ domain }} /etc/nginx/sites-enabled/{{ domain }}
  notify:
    - restart nginx

- name: Write {{ domain }} to /etc/hosts
  lineinfile:
    dest: /etc/hosts
    regexp: '.*{{ domain }}$'
    line: "127.0.0.1 {{ domain }}"
    state: present
```

![A1](asset/Picture17.png)
```
server {
  listen 80;
  server_name {{servername}};
  root {{ destdir }};
  index index.php;
  location / {
     try_files $uri $uri/ /index.php?$query_string;
  }
  location ~ \.php$ {
     fastcgi_pass unix:/run/php/php5.6-fpm.sock;  #Sesuaikan dengan versi PHP
     fastcgi_index index.php;
     fastcgi_param SCRIPT_FILENAME {{ destdir }}$fastcgi_script_name;
     include fastcgi_params;
  }
}
```
Jalankan install-ci.yml, hasilnya seperti ini :
![A1](asset/Picture19.png)

# Install Wordpress

Install-wp.yml
```
- hosts: lxc_php7_4_wp
  vars:
    username: 'admin'
    password: 'chintya'
    domain: lxc_php7_4_wp.dev
  roles:
    - wp

- hosts: lxc_php7_3_wp
  vars:
    username: 'admin'
    password: 'chintya'
    domain: lxc_php7_3_wp.dev
  roles:
    - wp

- hosts: lxc_php7_5_wp
  vars:
    username: 'admin'
    password: 'chintya'
    domain: lxc_php7_5_wp.dev
  roles:
    - wp

- hosts: wp
  vars:
username: 'admin'
    password: 'chintya'
    domain: lxc_php7_2.dev
  roles:
    - wp

```
![A1](asset/Picture18.png)
```
---
- name: restart nginx
  become: yes
  become_user: root
  become_method: su
  action: service name=nginx state=restarted

- name: restart php
  become: yes
  become_user: root
  become_method: su
  action: service name=php7.4-fpm state=restarted
```

![A1](asset/Picture20.png)
```
---
- name: delete apt chache
  become: yes
  become_user: root
  become_method: su
  command: rm -vf /var/lib/apt/lists/*

- name: install requirement dpkg to install php7.4
  become: yes
  become_user: root
  become_method: su
  apt: name={{ item }} state=latest update_cache=true
  with_items:
    - ca-certificates
    - apt-transport-https
    - curl
    - python-apt
    - software-properties-common

- name: Add Php Repository 7.4
  apt_repository:
    repo: "ppa:ondrej/php"
    state: present
    filename: php.list
    update_cache: true

- name: install nginx php7.4
  become: yes
  become_user: root
  become_method: su
  apt: name={{ item }} state=latest update_cache=true
  with_items:
    - nginx
    - nginx-extras
    - curl
    - wget
    - php7.4
    - php7.4-fpm
    - php7.4-curl
    - php7.4-xml
    - php7.4-gd
    - php7.4-opcache
    - php7.4-mbstring
    - php7.4-zip
    - php7.4-json
    - php7.4-cli
    - php7.4-mysqlnd
    - php7.4-xmlrpc

- name: wget wordpress
  shell: wget -c http://wordpress.org/latest.tar.gz

- name: tar latest.tar.gz
  shell: tar -xvzf latest.tar.gz

- name: copy folder wordpress
  shell: cp -R wordpress /var/www/html/blog

- name: chmod
  become: yes
  become_user: root
  become_method: su
  command: chmod 775 -R /var/www/html/blog/

- name: copy .wp-config.conf
  template:
    src=templates/wp.conf
    dest=/var/www/html/blog/wp-config.php

- name: copy wp.local
  template:
    src=templates/wp.local
    dest=/etc/nginx/sites-available/{{ domain }}
  vars:
    servername: '{{ domain }}'

- name: Delete another nginx config
  become: yes
  become_user: root
  become_method: su
  command: rm -f /etc/nginx/sites-enabled/*

- name: Symlink sites-available wordpress
  command: ln -sfn /etc/nginx/sites-available/{{ domain }} /etc/nginx/sites-enabled/{{ domain }}
  notify:
    - restart nginx

- name: Write {{ domain }} to /etc/hosts
  lineinfile:
    dest: /etc/hosts
    regexp: '.*{{ domain }}$'
    line: "127.0.0.1   {{ domain }}"
    state: present

- name: enable module php mbstring
  command: phpenmod mbstring
  notify:
    - restart php
```

![A1](asset/Picture21.png)
```
<?php
/**
* The base configuration for WordPress
*
* The wp-config.php creation script uses this file during the installation.
* You don't have to use the web site, you can copy this file to "wp-config.php"
* and fill in the values.
*
* This file contains the following configurations:
*
* * MySQL settings
* * Secret keys
* * Database table prefix
* * ABSPATH
*
* @link https://wordpress.org/support/article/editing-wp-config-php/
*
* @package WordPress
*/

define( 'WP_HOME', 'http://news.kelompok5.fpsas' );
define( 'WP_SITEURL', 'http://news.kelompok5.fpsas' );

// ** MySQL settings - You can get this info from your web host ** //
/** The name of the database for WordPress */
define( 'DB_NAME', 'news' );

/** MySQL database username */
define( 'DB_USER', 'admin' );

/** MySQL database password */
define( 'DB_PASSWORD', 'chintya' );

/** MySQL hostname */
define( 'DB_HOST', '10.0.3.200:3306' );

/** Database charset to use in creating database tables. */
define( 'DB_CHARSET', 'utf8' );

/** The database collate type. Don't change this if in doubt. */
define( 'DB_COLLATE', '' );

/**#@+
* Authentication unique keys and salts.
*
* Change these to different unique phrases! You can generate these using
* the {@link https://api.wordpress.org/secret-key/1.1/salt/ WordPress.org secret-key service}.
*
* You can change these at any point in time to invalidate all existing cookies.
* This will force all users to have to log in again.
*
* @since 2.6.0
*/
define( 'AUTH_KEY',         'put your unique phrase here' );
define( 'SECURE_AUTH_KEY',  'put your unique phrase here' );
define( 'LOGGED_IN_KEY',    'put your unique phrase here' );
define( 'NONCE_KEY',        'put your unique phrase here' );
define( 'AUTH_SALT',        'put your unique phrase here' );
define( 'SECURE_AUTH_SALT', 'put your unique phrase here' );
define( 'LOGGED_IN_SALT',   'put your unique phrase here' );
define( 'NONCE_SALT',       'put your unique phrase here' );

/**#@-*/

/**
* WordPress database table prefix.
*
* You can have multiple installations in one database if you give each
* a unique prefix. Only numbers, letters, and underscores please!
*/
$table_prefix = 'wp_';

/**
* For developers: WordPress debugging mode.
*
* Change this to true to enable the display of notices during development.
* It is strongly recommended that plugin and theme developers use WP_DEBUG
* in their development environments.
*
* For information on other constants that can be used for debugging,
* visit the documentation.
*
* @link https://wordpress.org/support/article/debugging-in-wordpress/
*/
define( 'WP_DEBUG', false );

/* Add any custom values between this line and the "stop editing" line. */



/* That's all, stop editing! Happy publishing. */

/** Absolute path to the WordPress directory. */
if ( ! defined( 'ABSPATH' ) ) {
      define( 'ABSPATH', __DIR__ . '/' );
}

/** Sets up WordPress vars and included files. */
require_once ABSPATH . 'wp-settings.php';

```

Isi templates/wp.local
```
server {

    listen 80;

    server_name {{ domain }};

    root /var/www/html/blog;
    index index.php;

    charset utf-8;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location ~ \.php$ {
        try_files $uri =404;
        fastcgi_split_path_info ^(.+\.php)(/.+)$;
        fastcgi_pass unix:/var/run/php/php7.4-fpm.sock;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $realpath_root$fastcgi_script_name;
        include fastcgi_params;
    }
}
```

Jalankan install-wp.yml, hasilnya:
![A1](asset/Picture22.png)

![A1](asset/Picture23.png)

![A1](asset/Picture24.png)


# Install Yii 2.0

Install-yii.yml
```
- hosts: lxc_php7_2_yii
  vars:
    domain: 'lxc_php7_2_yii.dev'
  roles:
    - yii

- hosts: lxc_php7_1_yii
  vars:
    domain: 'lxc_php7_1_yii.dev'
  roles:
    - yii

- hosts: lxc_php7_4_yii
  vars:
    domain: 'lxc_php7_4_yii.dev'
  roles:
    - yii

- hosts: lxc_php7_5_yii
  vars:
    domain: 'lxc_php7_5_yii.dev'
  roles:
    - yii

- hosts: lxc_php7_6_yii
  vars:
    domain: 'lxc_php7_6_yii.dev'
  roles:
    - yii
```

![A1](asset/Picture25.png)
```
---
- name: restart nginx
  become: yes
  become_user: root
  become_method: su
  action: service name=nginx state=restarted

- name: restart php
  become: yes
  become_user: root
  become_method: su
  action: service name=php5.6-fpm state=restarted
```
![A1](asset/Picture26.png)
```
---
- name: delete apt chache
  become: yes
  become_user: root
  become_method: su
  command: rm -vf /var/lib/apt/lists/*

- name: install requirement dpkg to install php7.4
  become: yes
  become_user: root
  become_method: su
  apt: name={{ item }} state=latest update_cache=true
  with_items:
    - ca-certificates
    - apt-transport-https
    - curl
    - python-apt
    - software-properties-common

- name: Add Php Repository 7.4
  apt_repository:
    repo: "ppa:ondrej/php"
    state: present
    filename: php.list
    update_cache: true

- name: install nginx php7.4
  become: yes
  become_user: root
  become_method: su
  apt: name={{ item }} state=latest update_cache=true
  with_items:
   - git
   - nginx
   - nginx-extras
   - php7.4
   - php7.4-fpm
   - php7.4-curl
   - php7.4-mbstring
   - php7.4-xml
   - php7.4-gd
   - php7.4-opcache
   - php7.4-zip
   - php7.4-json
   - php7.4-cli
   - php7.4-intl

- name: Download and install Composer
  shell: curl -sS https://getcomposer.org/installer | php
  args:
    chdir: /usr/src/
    creates: /usr/local/bin/composer
    warn: false
  become: yes

- name: Add Composer to global path
  copy:
    dest: /usr/local/bin/composer
    group: root
    mode: '0755'
    owner: root
    src: /usr/src/composer.phar
    remote_src: yes
  become: yes

- name: Composer create project
  composer:
    command: create-project
    arguments: yiisoft/yii2-app-basic product
    working_dir: /var/www/html
    prefer_dist: yes
 environment:
    COMPOSER_NO_INTERACTION: "1"

- name: set permission
  shell: chown -R www-data:www-data /var/www/html/product

- name: setting db
  template:
    src=templates/db.php
    dest=/var/www/html/product/config/db.php

- name: Delete another nginx config
  become: yes
  become_user: root
  become_method: su
  command: rm -f /etc/nginx/sites-enabled/*

- name: www.conf
  lineinfile:
    dest: /etc/php/7.4/fpm/pool.d/www.conf
    regexp: '^listen = /run/php/php7.4-fpm.sock'
    line: listen = 127.0.0.1:9001
    state: present

- name: Copy product.conf
  template:
    src=templates/product.config
    dest=/etc/nginx/sites-available/{{ domain }}
  vars:
    servername: '{{ domain }}'

- name: Symlink product.conf
  command: ln -sfn /etc/nginx/sites-available/{{ domain }} /etc/nginx/sites-enabled/{{domain}}
  notify:
    - restart nginx

- name: Write {{ domain }} to /etc/hosts
  lineinfile:
    dest: /etc/hosts
    regexp: '.*{{ domain }}$'
    line: "127.0.0.1   {{ domain }}"
    state: present
```

![A1](asset/Picture27.png)
```
<?php

return [
    'class' => 'yii\db\Connection',
    'dsn' => 'mysql:host=localhost;dbname=product',
    'username' => 'admin',
    'password' => 'chintya',
    'charset' => 'utf8',
];
```

![A1](asset/Picture28.png)
```
server {
        listen 80;
        listen [::]:80;

        server_name {{servername}};

        root /var/www/html/product/web;
        index index.php;

        charset utf-8;

        location / {
                index  index.html index.php;
                try_files $uri $uri/ /index.php?$args;
        }

        location ~ ^/(protected|framework|themes/\w+/views) {
                deny  all;
        }

        # pass the PHP scripts to FastCGI server listening on UNIX socket
        location ~ \.php {
                fastcgi_split_path_info  ^(.+\.php)(.*)$;
        #let yii catch the calls to unexising PHP files
                set $fsn /index.php;
                if (-f $document_root$fastcgi_script_name){
                        set $fsn $fastcgi_script_name;
                }
                fastcgi_pass   127.0.0.1:9001;
                include fastcgi_params;
                fastcgi_param  SCRIPT_FILENAME  $document_root$fsn;

       #PATH_INFO and PATH_TRANSLATED can be omitted, but RFC 3875 specifies them for CGI
                fastcgi_param  PATH_INFO        $fastcgi_path_info;
                fastcgi_param  PATH_TRANSLATED  $document_root$fsn;
        }

        # prevent nginx from serving dotfiles (.htaccess, .svn, .git, etc.)
        location ~ /\. {
                deny all;
                access_log off;
                log_not_found off;
        }
}
```
Jalankan install-yii.yml, hasilnya:

![A1](asset/Picture29.png)

# Jmeter 

## Load Testing

## kelompok5.fpsas

- 50

![A1](asset/Picture30.png)

![A1](asset/Picture31.png)

![A1](asset/Picture32.png)

- 150

![A1](asset/Picture33.png)

![A1](asset/Picture34.png)

![A1](asset/Picture35.png)

- 300

![A1](asset/Picture36.png)

![A1](asset/Picture37.png)

![A1](asset/Picture38.png)

- 500

![A1](asset/Picture39.png)

![A1](asset/Picture40.png)

![A1](asset/Picture41.png)

## news.kelompok5.fpsas

- 50

![A1](asset/Picture42.png)

![A1](asset/Picture43.png)

![A1](asset/Picture44.png)

- 150

![A1](asset/Picture45.png)

![A1](asset/Picture46.png)

![A1](asset/Picture47.png)

- 300

![A1](asset/Picture48.png)

![A1](asset/Picture49.png)

![A1](asset/Picture50.png)

- 500

![A1](asset/Picture51.png)

![A1](asset/Picture52.png)

![A1](asset/Picture53.png)

# Analisis 
1. Nilai rata – rata Throughput untuk setiap website yang dihasilkan dari load testing

- 50 user
   - kelompok5.fpsas/ : 32.2/s
   - kelompok5.fpsas/app : 34.6/s
   - kelompok5.fpsas/product : 25.2/s
   - news.kelompok5.fpsas/ : 568.2/s

- 150 user
   - kelompok5.fpsas/ : 30.1/s
   - kelompok5.fpsas/app : 31.3/s
   - kelompok5.fpsas/product : 26.1/s
   - news.kelompok5.fpsas/ : 738.9/s

- 300 user
   - kelompok5.fpsas/ : 50/s
   - kelompok5.fpsas/app : 50.7/s
   - kelompok5.fpsas/product : 35.8/s
   - news.kelompok5.fpsas/ : 547.4/s

- 500 user
   - kelompok5.fpsas/ : 63.2/s
   - kelompok5.fpsas/app : 63.3/s
   - kelompok5.fpsas/product : 42.8/s
   - news.kelompok5.fpsas/ : 591.7/s

2. Nilai rata - rata jumlah user yang dapat dilayani setiap detik untuk setiap website yang dihasilkan dari load testing
- 50 user
   - kelompok5.fpsas/ : 292        
   - kelompok5.fpsas/app : 419
   - kelompok5.fpsas/product : 219
   - news.kelompok5.fpsas/ : 52

- 150 user
   - kelompok5.fpsas/ : 1001
   - kelompok5.fpsas/app : 628
   - kelompok5.fpsas/product : 503
   - news.kelompok5.fpsas/ : 95 

- 300 user
   - kelompok5.fpsas/ : 917
   - kelompok5.fpsas/app : 221
   - kelompok5.fpsas/product : 798
   - news.kelompok5.fpsas/ : 222 

- 500 user
   - kelompok5.fpsas/ : 1129
   - kelompok5.fpsas/app : 459
   - kelompok5.fpsas/product : 1117
   - news.kelompok5.fpsas/ : 359
   
3. Cara mengurangi nilai througput dan meningkatkan nilai jumlah user yang dapat dilayani setiap detik untuk skema yang telah dibuat serta faktor-faktor yang mempengaruhinya :
- Optimasi database
- Downgrade OS
- Mengoptimalkan kinerja framework yang digunakan
