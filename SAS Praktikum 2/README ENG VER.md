# Practical Report 2 - Server Administration System
Arranged by :
1. Chintya Tribhuana Utami (1202190041)
2. Nur Wulan Maudini (1202190002)
#
The practicum is carried out based on the conditions listed in the questions and questions can be accessed [Click here.](https://github.com/aldonesia/Sistem-Administrasi-Server-2021/blob/master/modul-2/soal_praktikum.md)
#
In the implementation of working on practical questions, we made changes to the initial state of the previous practice questions with the practical questions that have been given this time.
#
### Number 1. Change focal ubuntu_landing
- Display LXC list before changing LXC
  ```
  sudo lxc -ls -f
  ```  
  ![A1](asset/1.png)

- Delete ubuntu_landing_backup first then delete ubuntu_landing. Don't forget when you want to delete, make sure LXC has stopped.
  ```
  sudo lxc-destroy ubuntu_landing_backup
  
  #check if it's been deleted
  sudo lxc -ls -f

  #stop ubuntu_landing
  sudo lxc-stop -n ubuntu_landing

  #destroy ubuntu_landing
  sudo lxc-destroy ubuntu_landing

  #check again if everything has been deleted
  sudo lxc -ls -f
  ```
  ![A1](asset/2.png)

- Create LXC ubuntu_landing with version 20
  ```
  sudo lxc-create -n ubuntu_landing -t download -- --dist ubuntu --release focal fossa --arch amd64 --foce-cache --no-validate --server images.linuxcontainers.org
  ```
  ![A1](asset/3.png)
- Setting autostart
  ```
  sudo su
  echo "lxc.start.auto = 1" >> /var/lib/lxc/ubuntu_landing/config
  exit
  ```
  ![A1](asset/4.png)
- Install nginx
  ```
  sudo lxc-start -n ubuntu_landing
  sudo lxc-attach -n ubuntu_landing
  sudo apt install nginx nginx-extras
  ```
  ![A1](asset/5.png)
- Install nano first, then edit the ip to be static
  ```
  sudo nano /etc/netplan/10-lxc.yaml
  ```
  ![A1](asset/6.png)
  ```
  ifconfig
  ```
  ![A1](asset/7.png)
- Create a file in sites-available with the name lxc_landing.dev
  ```
  cd /etc/nginx/sites-available
  touch lxc_landing.dev
  nano lxc_landing.dev
  ```
  ![A1](asset/8.png)
  ###
  ![A1](asset/9.png)
- Create a symlink in sites-enabled that points to sites-available
  ```
  cd ../sites-enabled
  ln -s /etc/nginx/sites-available/lxc_landing.dev .
  ```
  ![A1](asset/10.png)
  ```
  nginx -t
  nginx -s reload
  ```
  ![A1](asset/11.png)
- Setting etc/hosts
  ```
  nano /etc/hosts
  ```
  ![A1](asset/12.png)
- Enter ke var html
  ```
  cd /var/www/html
  mkdir lxc_landing
  cp index.nginx-debian.html lxc_landing/index.html
  cd lxc_landing
  nano index.html
  ```
  ![A1](asset/13.png)
- Check curl
  ```
  curl -i http://lxc_landing.dev
  ```
  ![A1](asset/14.png)
- Install and setting ssh
  ```
  sudo apt install openssh-server python
  nano /etc/ssh/sshd_config

  #setting config to
  PermitRootLogin yes
  RSAAuthentication yes

  #end of config
  service sshd restart

  #set password
  passwd

  exit
  ```
  ![A1](asset/15.png)
  ###
  ![A1](asset/16.png)
  ###
  ![A1](asset/17.png)
- Check ssh root if it can run
  ```
  ssh root@lxc_landing.dev
  exit
  ```
- Backup and turn off the autostart
  ```
  sudo lxc-stop -n ubuntu_landing
  sudo lxc-copy -n ubuntu_landing -N ubuntu_landing_backup -sKD
  ```
  ![A1](asset/18.png)
  ###
  ![A1](asset/19.png)
#
### Number 2. Change the focal ubuntu_php7.4
- Destroy ubuntu
  ```
  sudo su
  lxc-stop -n ubuntu_php7.4
  lxc-stop -n ubuntu_php7.4_backup
  lxc-destroy ubuntu_php7.4_backup
  lxc-destroy ubuntu_php7.4
  ```
  ![A1](asset/20.png)
- Install
  ```
  sudo lxc-create -n ubuntu_php7.4 -t download -- --dist ubuntu --release focal fossa --arch amd64 --foce-cache --no-validate --server images.linuxcontainers.org
  ```
  ![A1](asset/21.png)
- Setting autostart
  ```
  echo "lxc.start.auto = 1" >> /var/lib/lxc/ubuntu_landing/config
  exit
  ```
  ![A1](asset/22.png)
- Install nginx
  ```
  sudo lxc-start -n ubuntu_php7.4
  sudo lxc-attach -n ubuntu_php7.4
  sudo apt install nginx nginx-extras
  ```
  ![A1](asset/23.png)
- Install nano first, then edit the ip to be static
  ```
  sudo nano /etc/netplan/10-lxc.yaml
  ```
  ![A1](asset/24.png)
  ```
  ifconfig
  ```
  ![A1](asset/25.png)
- Create a file in sites-available with the name lxc_php7.4.dev
  ```
  cd /etc/nginx/sites-available
  touch lxc_php7.4.dev
  nano lxc_php7.4.dev
  ```
  ![A1](asset/26.png)
- Create a symlink in sites-enabled that points to sites-available
  ```
  cd ../sites-enabled
  ln -s /etc/nginx/sites-available/lxc_7.4.dev .
  nginx -t
  nginx -s reload
  ```
  ![A1](asset/27.png)
- Setting etc/hosts
  ```
  nano /etc/hosts
  ```
  ![A1](asset/28.png)
- Enter var html
  ```
  cd /var/www/html
  mkdir lxc_php7.4
  cp index.nginx-debian.html lxc_php7.4/index.html
  cd lxc_landing
  nano index.html
  ```
  ![A1](asset/29.png)
- Check curl
  ```
  curl -i http://lxc_php7.4.dev
  ```
  ![A1](asset/30.png)
- Install and setting ssh 
  ```
  sudo lxc-attach -n ubuntu_php7.4
  sudo apt install openssh-server python
  nano /etc/ssh/sshd_config

  #setting config menjadi
  PermitRootLogin yes
  RSAAuthentication yes

  #end of config
  service sshd restart

  #set password
  passwd

  exit
  ```
  ![A1](asset/31.png)
  ###
  ![A1](asset/32.png)
  ###
  ![A1](asset/33.png)
- Run ssh root
  ```
  ssh root@lxc_php7.dev
  ```
  ![A1](asset/34.png)
- Backup and turn off the autostart
  ```
  sudo su
  lxc-stop -n ubuntu_php7.4
  lxc-copy -n ubuntu_php7.4 -N ubuntu_php7.4_backup -sKD
  ```
  ![A1](asset/35.png)
  ###
  ![A1](asset/36.png)
#
### Number 3. vm.local/ (Laravel)
- Edit hosts in module2-ansible
  ```
  cd ~/ansible/modul2-ansible
  nano hosts
  ```
  ![A1](asset/37.png)
- Try ping
  ```
  cd ~/ansible -i hosts -m ping all -k 
  ```
  ![A1](asset/38.png)
- Update
  ```
  ssh root@ubuntu_landing
  ```
  ![A1](asset/39.png)
- Creat file deploy-laravel.yml
  ```
  cd ~/ansible/modul2-ansible
  nano deploy-laravel.yml
  ```
- Contents of deploy-laravel.yml
  ```
  - hosts: landing
  vars:
    domain: lxc_landing.dev
  roles:
    - { role: laravel }
  ```
  ![A1](asset/40.png)
- Create roles/Laravel containing:
  ```
  mkdir -p roles/laravel/tasks
  mkdir -p roles/laravel/handlers
  mkdir -p roles/laravel/templates
  ```
  ![A1](asset/41.png)
  
- Fill main.yml in tasks
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
   - php7.4
   - php7.4-fpm
   - php7.4-curl
   - php7.4-mbstring
   - php7.4-xml
   - php7.4-gd
   - php7.4-opcache
   - php7.4-zip php7.4 -y

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
  
- name: Composer create project
  become_user: root
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
     line=APP_URL=http://vm.local
    
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

- Contents of landing on templates
  ```
  server {

      listen 80;

      server_name {{ domain }};

      root /var/www/html/landing/public;
      index index.php;

      charset utf-8;

      location / {
          try_files $uri $uri/ /index.php?$query_string;
      }

      location = /favicon.ico { access_log off; log_not_found off; }
      location = /robots.txt  { access_log off; log_not_found off; }

      error_page 404 /index.php;

      location ~ \.php$ {
          fastcgi_pass unix:/var/run/php/php7.4-fpm.sock;
          fastcgi_index index.php;
          fastcgi_param SCRIPT_FILENAME $realpath_root$fastcgi_script_name;
          include fastcgi_params;
      }

      location ~ /\.(?!well-known).* {
          deny all;
      }
  }
  ```
- Isi main.yml pada handlers
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
    become_methode: su
    action: service name=php7.4-fpm state=restarted
  ```
  ![A1](asset/42.png)
- Jalankan
  ```
  cd ~/ansible/modul2-ansible
  ansible-playbook -i hosts deploy-laravel.yml -k
  ```
  ![A1](asset/43.png)
- Hasil
  ![A1](asset/44.png)
#
### Number 4. vm.local/blog (Wordpress)
- Edit deploy-wp
  ```
  cd ~/ansible/modul2-ansible
  nano deploy-wp.yml
  ```
  ![A1](asset/45.png)
- Buat roles wp
  ```
  mkdir -p roles/wp
  mkdir -p roles/wp/handlers
  mkdir -p roles/wp/tasks
  mkdir -p roles/wp/templates
  ```
  ![A1](asset/46.png)
- Isi pada /tasks/main.yml
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

- Fill in /templates/wp.conf
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

  define( 'WP_HOME', 'http://vm.local/blog' );
  define( 'WP_SITEURL', 'http://vm.local/blog' );

  // ** MySQL settings - You can get this info from your web host ** //
  /** The name of the database for WordPress */
  define( 'DB_NAME', 'blog' );

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

- Fill ini /templates/wp.local
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
- Contents of /handlers/main.yml
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
  ![A1](asset/47.png)
- Run
  ```
  cd ~/ansible/modul2-ansible
  ansible-playbook -i hosts deploy-wp.yml -k
  ```
  ![A1](asset/48.png)
- Results
  ![A1](asset/49.png)
  ###
  ![A1](asset/50.png)
  ###
  ![A1](asset/51.png)
  ###
  ![A1](asset/52.png)
#
### Number 5. Additional questions
- Create file change-socket.yml
  ```
  hosts: landing
  tasks:
    - name: www.conf
      lineinfile:
         dest: /etc/php/7.4/fpm/pool.d/www.conf
         regexp: '^listen = /run/php/php7.4-fpm.sock'
         line: listen = 127.0.0.1:9001
         state: present

    - name: rubah di sites-available
      lineinfile:
         dest: /etc/nginx/sites-available/lxc_landing.dev
         regexp: '^        fastcgi_pass'
         line:         fastcgi_pass 127.0.0.1:9001;

    - name: restart nginx
      become: yes
      become_user: root
      become_method: su
      action: service name=nginx state=restarted

  hosts: php7
  tasks:
    - name: www.conf
      lineinfile:
         dest: /etc/php/7.4/fpm/pool.d/www.conf
         regexp: '^listen = /run/php/php7.4-fpm.sock'
         line: listen = 127.0.0.1:9001
         state: present

    - name: rubah di sites-available
      lineinfile:
         dest: /etc/nginx/sites-available/lxc_php7.dev
         regexp: '^        fastcgi_pass'
         line:         fastcgi_pass 127.0.0.1:9001;

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
- Run
  ```
  cd ~/ansible/modul2-ansible
  ansible-playbook -i hosts ubah-socket.yml -k
  ``` 
  ![A1](asset/53.png)
