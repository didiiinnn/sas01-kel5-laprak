# Laporan Praktikum 2 - Sistem Administrasi Server 
Disusun oleh :
1. Chintya Tribhuana Utami (1202190041)
2. Nur Wulan Maudini (1202190002)
#
Praktikum dilaksanakan berdasarkan keadaan yang tertera pada soal dan soal dapat diakses [Klik disini.](https://github.com/aldonesia/Sistem-Administrasi-Server-2021/blob/master/modul-2/soal_praktikum.md)
#
Dalam pelaksanaan mengerjakan soal-soal praktikum, kami melakukan perubahan keadaan awal dari soal-soal latihan sebelumnya dengan soal-soal praktikum yang telah diberikan kali ini.
#
### Nomor 1. Change focal ubuntu_landing
- Menampilkan list LXC sebelum mengubah LXC
  ```
  sudo lxc -ls -f
  ```  
  ![A1](asset/1.png)

- Hapus ubuntu_landing_backup terlebih dahulu kemudian hapus ubuntu_landing. Jangan lupa ketika ingin menghapus, pastikan LXC sudah berhenti.
  ```
  sudo lxc-destroy ubuntu_landing_backup
  
  #cek apakah sudah terhapus
  sudo lxc -ls -f

  #stop ubuntu_landing
  sudo lxc-stop -n ubuntu_landing

  #hapus ubuntu_landing
  sudo lxc-destroy ubuntu_landing

  #cek lagi apakah sudah terhapus semua
  sudo lxc -ls -f
  ```
  ![A1](asset/2.png)

- Buat lxc ubuntu_landing dg versi 20
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
- Install nano terlebih dahulu, lalu edit ip nya menjadi static
  ```
  sudo nano /etc/netplan/10-lxc.yaml
  ```
  ![A1](asset/6.png)
  ```
  ifconfig
  ```
  ![A1](asset/7.png)
- Buat file di sites-available dengan nama lxc_landing.dev
  ```
  cd /etc/nginx/sites-available
  touch lxc_landing.dev
  nano lxc_landing.dev
  ```
  ![A1](asset/8.png)
  ###
  ![A1](asset/9.png)
- Buat symlink di sites-enabled yang mengarah ke sites-available
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
- Masuk ke var html
  ```
  cd /var/www/html
  mkdir lxc_landing
  cp index.nginx-debian.html lxc_landing/index.html
  cd lxc_landing
  nano index.html
  ```
  ![A1](asset/13.png)
- Cek curl
  ```
  curl -i http://lxc_landing.dev
  ```
  ![A1](asset/14.png)
- Install dan setting ssh
  ```
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
  ![A1](asset/15.png)
  ###
  ![A1](asset/16.png)
  ###
  ![A1](asset/17.png)
- cek ssh root apakah dapat berjalan
  ```
  ssh root@lxc_landing.dev
  exit
  ```
- Backup dan matikan autostart nya
  ```
  sudo lxc-stop -n ubuntu_landing
  sudo lxc-copy -n ubuntu_landing -N ubuntu_landing_backup -sKD
  ```
  ![A1](asset/18.png)
  ###
  ![A1](asset/19.png)
#
### Nomor 2. Ganti focal ubutu_php7.4
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
  echo "lxc.start.auto = 1" >> /var/lib/lxc/ubuntu_php7.4/config
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
- Install nano terlebih dahulu, lalu edit ip nya menjadi static
  ```
  sudo nano /etc/netplan/10-lxc.yaml
  ```
  ![A1](asset/24.png)
  ```
  ifconfig
  ```
  ![A1](asset/25.png)
- Buat file di sites-available dengan nama lxc_php7.4.dev
  ```
  cd /etc/nginx/sites-available
  touch lxc_php7.4.dev
  nano lxc_php7.4.dev
  ```
  ![A1](asset/26.png)
- Buat symlink di sites-enabled yang mengarah ke sites-available
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
- Masuk ke var html
  ```
  cd /var/www/html
  mkdir lxc_php7.4
  cp index.nginx-debian.html lxc_php7.4/index.html
  cd lxc_landing
  nano index.html
  ```
  ![A1](asset/29.png)
- Cek curl
  ```
  curl -i http://lxc_php7.4.dev
  ```
  ![A1](asset/30.png)
- Install dan setting ssh 
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
- Jalankan ssh root nya
  ```
  ssh root@lxc_php7.dev
  ```
  ![A1](asset/34.png)
- Backup dan matikan autostart nya
  ```
  sudo su
  lxc-stop -n ubuntu_php7.4
  lxc-copy -n ubuntu_php7.4 -N ubuntu_php7.4_backup -sKD
  ```
  ![A1](asset/35.png)
  ###
  ![A1](asset/36.png)
#
### Nomor 3. vm.local/ (Laravel)
- Edit hosts di modul2-ansible
  ```
  cd ~/ansible/modul2-ansible
  nano hosts
  ```
  ![A1](asset/37.png)
- Coba ping
  ```
  cd ~/ansible -i hosts -m ping all -k 
  ```
  ![A1](asset/38.png)
- Update
  ```
  ssh root@ubuntu_landing
  ```
  ![A1](asset/39.png)
- buat file deploy-laravel.yml
  ```
  cd ~/ansible/modul2-ansible
  nano deploy-laravel.yml
  ```
- isi dari deploy-laravel.yml
  ```
  - hosts: landing
  vars:
    domain: lxc_landing.dev
  roles:
    - { role: laravel }
  ```
  ![A1](asset/40.png)
- Buat roles/Laravel berisi:
  ```
  mkdir -p roles/laravel/tasks
  mkdir -p roles/laravel/handlers
  mkdir -p roles/laravel/templates
  ```
  ![A1](asset/41.png)
  
- Isi main.yml pada tasks
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

- Isi landing pada templates
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
### Nomor 4. vm.local/blog (Wordpress)
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

- Isi pada /templates/wp.conf
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

- Isi pada /templates/wp.local
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
- Isi /handlers/main.yml
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
- Jalankan
  ```
  cd ~/ansible/modul2-ansible
  ansible-playbook -i hosts deploy-wp.yml -k
  ```
  ![A1](asset/48.png)
- Hasil
  ![A1](asset/49.png)
  ###
  ![A1](asset/50.png)
  ###
  ![A1](asset/51.png)
  ###
  ![A1](asset/52.png)
#
### Nomor 5. Soal tambahan
- buat file ubah-socket.yml
  ```
  - hosts: landing
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

  - hosts: php7
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
  
- Jalankan
  ```
  cd ~/ansible/modul2-ansible
  ansible-playbook -i hosts ubah-socket.yml -k
  ``` 
  ![A1](asset/53.png)
  
- Hasil di lxc_landing.dev

  ![A1](asset/54.png)
  
- Hasil di lxc_php7.dev

  ![A1](asset/55.png)
