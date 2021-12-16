# Laporan Praktikum 3 - Sistem Administrasi Server 
Disusun by :
1. Chintya Tribhuana Utami (1202190041)
2. Nur Wulan Maudini (1202190002)
#
Praktikum dilaksanakan berdasarkan keadaan yang tertera pada soal dan soal dapat diakses [Klik disini.](https://github.com/aldonesia/Sistem-Administrasi-Server-2021/blob/master/modul-3/silabus.md)
#
Dalam pelaksanaan mengerjakan soal-soal praktikum, kami melakukan perubahan keadaan awal dari soal-soal latihan sebelumnya dengan soal-soal praktikum yang telah diberikan kali ini.
#
Masuk ke ~/ansible/modul2-ansible , kemudian membuat folder roles/laravel yang berisi tasks, template, dan handlers
```
cd ~/ansible/modul2-ansible
sudo mkdir -p roles/laravel/tasks
sudo mkdir -p roles/laravel/template
sudo mkdir -p roles/laravel/handlers
```
![A1](asset/Picture1.png)
###
Pada ansible buat setting-landing.yml berisi:
###
![A1](asset/Picture2.png)
###
Pada Laravel/tasks/main.yml isi :
###
![A1](asset/Picture3.png)
```
---
- name: delete apt chache
  become: yes
  become_user: root
  become_method: su
  command: rm -vf /var/lib/apt/lists/*

- name: install package tambahan
  become: yes
  become_user: root
  become_method: su
  apt: name={{ item }} state=latest update_cache=true
  with_items:
    - gtkhash
    - bind9
    - dnsutils
    - crack-md5
    - git
    - curl

- name: buat directori dev/landing
  file:
    path: /var/www/html/dev/landing
    state: directory

- name: copy named.conf.local
  template:
    src=template/named.conf.local
    dest=/var/www/html/dev/landing
  notify:
    - restart bind

- name: copy vm.local
  template:
    src=template/vm.local
    dest=/var/www/html/dev/landing
  notify:
    - restart bind

- name: copy 1.168.192 .in-addr.arpa
  template:
    src=template/1.168.192 .in-addr.arpa
    dest=/var/www/html/dev/landing
  notify:
    - restart bind

- name: copy resolv.conf
  template:
    src=template/resolv.conf
    dest=/etc/resolv.conf

- name: copy named.conf.options
  template:
    src=template/named.conf.options
    dest=/var/www/html/dev/landing
  notify:
    - restart bind
```
Pada Laravel/handlers/main.yml berisi:
###
![A1](asset/Picture4.png)
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

- name: restart bind
  become: yes
  become_user: root
  become_method: su
  action: service name=bind9 state=restarted
```
Pada Laravel/template/named.conf.local
###
![A1](asset/Picture5.png)
```
//
// Do any local configuration here
//

// Consider adding the 1918 zones here, if they are not used in your
// organization
//include "/etc/bind/zones.rfc1918";

zone "vm.local" {
        type master;
        file "/etc/bind/vm/vm.local";
};

zone "1.168.192 .in-addr.arpa" {
        type master;
        file "/etc/bind/vm/1.168.192 .in-addr.arpa";
};
```
Pada Laravel/template/vm.local berisi :
###
![A1](asset/Picture6.png)
```
;
; BIND data file for local loopback interface
;
$TTL    604800
@       IN      SOA     vm.local. root.vm.local. (
                              2         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@       IN      NS      vm.local.
@       IN      A       192.168.1.100 
dev     IN      CNAME   vm.local.
```
Pada Laravel/template/ 1.168.192 .in-addr.arpa
###
![A1](asset/Picture7.png) 
```
;
; BIND data file for local loopback interface
;
$TTL    604800
@       IN      SOA     vm.local. root.vm.local. (
                              2         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
1.168.192.in-addr.arpa.  IN      NS      vm.local. ; IP VM dibalik tanpa byte ke 4
100                       IN      PTR     vm.local. ; byte ke 4 IP VM
```
Pada Laravel/template/resolv.conf
###
![A1](asset/Picture8.png)
```
# This file is managed by man:systemd-resolved(8). Do not edit.
#
# This is a dynamic resolv.conf file for connecting local clients to the
# internal DNS stub resolver of systemd-resolved. This file lists all
# configured search domains.
#
# Run "resolvectl status" to see details about the uplink DNS servers
# currently in use.
#
# Third party programs must not access this file directly, but only through the
# symlink at /etc/resolv.conf. To manage man:resolv.conf(5) in a different way,
# replace this symlink by a static file or a different symlink.
#
# See man:systemd-resolved.service(8) for details about the supported modes of
# operation for /etc/resolv.conf.

#nameserver 127.0.0.53
#options edns0 trust-ad

nameserver 192.168.1.100 
```
Pada Laravel/template/named.conf.options
###
![A1](asset/Picture9.png)
```    
options {
        directory "/var/cache/bind";

        // If there is a firewall between you and nameservers you want
        // to talk to, you may need to fix the firewall to allow multiple
        // ports to talk.  See http://www.kb.cert.org/vuls/id/800113

        // If your ISP provided one or more IP addresses for stable
        // nameservers, you probably want to use them as forwarders.
        // Uncomment the following block, and insert the addresses replacing
        // the all-0's placeholder.

        forwarders {
                8.8.8.8;
        };

        //========================================================================
        // If BIND logs error messages about the root key being expired,
        // you will need to update your keys.  See https://www.isc.org/bind-keys
        //========================================================================
        //dnssec-validation auto;
        allow-query{any;};

        listen-on-v6 { any; };
};
```
Jalankan
###
![A1](asset/Picture10.png)
###
Tambahkan dev.vm.local di /etc/hosts
###
![A1](asset/Picture11.png)
###
![A1](asset/Picture12.png)
###
Masuk ke /var/www/html/dev/landing pada lxc_landing, lalu edit vm.local menjadi
###
![A1](asset/Picture13.png)
###
![A1](asset/Picture14.png)
###
Lalu restart service bind9
###
![A1](asset/Picture15.png)
###
Edit vm.local di sites available
###
![A1](asset/Picture16.png)
###
Jangan lupa rstart nginx
Ganti dns di control panel
###
![A1](asset/Picture17.png)
###
![A1](asset/Picture18.png)
###
![A1](asset/Picture19.png)
###
![A1](asset/Picture20.png)
