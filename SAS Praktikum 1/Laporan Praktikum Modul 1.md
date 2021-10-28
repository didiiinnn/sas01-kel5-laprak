# Laporan Praktikum 1 - Sistem Administrasi Server 
Disusun oleh :
1. Chintya Tribhuana Utami (1202190041)
2. Nur Wulan Maudini (1202190002)


# ______________________________________

Praktikum dikerjakan berdasarkan skema yang tertera di soal dan soal dapat diakses [Klik Disini.](https://github.com/aldonesia/Sistem-Administrasi-Server-2021/blob/master/modul-1/soal_praktikum.md)
#  ______________________________________
Pada pelaksanaan pengerjaan soal praktikum , kita melakukan perubahan dengan keadaan awal latihan soal praktikum sebelumnya dengan soal praktikum yang sudah diberikan kali ini.
#
#
### Soal 1. Rename ubuntu_php5.6 menjadi ubuntu_landing, serta rubah IP mengikuti skema yang baru
___
- Menampilkan list LXC untuk mengecek nama LXC sebelum direname
    ```
    sudo lxc -ls -f
    ```  
    ![B1](asset/Picture1.png)

- Stop terlebih dahulu service pada ubuntu_php5.6

  ![B1](asset/Picture2.png)

- Ubah nama ubuntu_php5.6 menjadi ubuntu_landing

  ![B1](asset/Picture3.png)

- Start service ubuntu_landing lalu attach ubuntu_landing
  ```
    sudo lxc-start -n ubuntu_landing
    sudo lxc-attach -n ubuntu_landing
  ```  

  ![B1](asset/Picture4.png)

- Ganti IP pada LXC ubuntu_landing menjadi 10.0.3.103

  ![B1](asset/Picture5.png)

  ![B1](asset/Picture6.png)

### Soal 2. Install lxc debian 9 dengan nama debian_php5.6
- Cek setting IP di virtual machine

  ![B1](asset/Picture7.png)

- Cek apakah bisa tersambung ke internet

  ![B1](asset/Picture8.png)

- Tambahkan kode berikut pada sources list ubuntu
    ```
    sudo nano /etc/apt/sources.list
    ```
  ![B1](asset/Picture10.png)
  
- Install lxc Debian 9
  ```
  sudo lxc-create -n debian_php5.5 -t download -- --dist debian --release stretch --arch amd64 --force-cache --no-validate --server images.linuxcontainers.org
  ```

  ![B1](asset/Picture11.png)
  
  - Update debian_php5.6

  ![B1](asset/Picture13.png)
  
  - Informasi debian release

  ![B1](asset/Picture14.png)

### Soal 3. setup nginx pada debian_php5.6 untuk domain http://lxc_php5.dev , buat halaman index.html yang menerangkan informasi nama lxc
- Exit ubuntu landing
  ```
  Exit
  ```
  Start dan attach debian_php5.6 lalu install nginx dan nginx extra pada debian_php5.6
  ```
  sudo lxc-start -n debian_php5.6
  sudo lxc-attach -n debian_php5.6
  sudo apt install nginx nginx-extras
  ```
  ![B1](asset/Picture15.png)

- Install curl dan setting ip
   ```
  apt install nano net-tools curl
  nano /etc/network/interfaces
  ```

  ![B1](asset/Picture16.png)

  ![B1](asset/Picture17.png)
  
  Restart service networking
  ```
  systemctl restart networking.service
  ```
  Alternatif apabila tidak terganti IP nya :
  ```
  shutdown now
  sudo lxc-start -n debian_php5.6
  sudo lxc-attach -n debian_php5.6
  ```
  
  Cek IP apakah sudah ganti ataukah belum menggunakan ifconfig

  ![B1](asset/Picture18.png)
  
- Masuk ke direktori sites-available pada nginx debian_php5.6
```
cd /etc/nginx/sites-available
```
- Buat file baru bernama lxc_php5.6.dev
```
touch lxc_php5.6.dev
```
- Edit lxc_php5.6
```
nano lxc_php5.6.dev
```

  ![B1](asset/Picture19.png)

- Masuk ke direktori sites-enabled pada nginx debian_php5.6 untuk membuat symlink ke sites-available/lxc_php5.6
  ```
  kode
  ```
  
- Tes nginx dan restart service nginx
  ```
  kode
  ```

  ![B1](asset/Picture20.png)

- Setting hosts dengan masuk ke direktori /etc/hosts
  ```
  kode
  ```
  
- Tambahkan ip lxc_php5.dev sama seperti localhost yaitu 127.0.0.1
  ```
  kode
  ```

  ![B1](asset/Picture21.png)

- Masuk direktori var/www/html lalu buat folder baru bernama lxc_php5.6
  ```
  kode
  ```
- Copy index.nginx-debian.html ke file baru di folder lxc_php5.6 dengan nama index.html
  ```
  kode
  ```
  
  ![B1](asset/Picture22.png)
  
- Edit index di folder lxc_php5.6 beri keterangan bahwa 'Halaman ini dari lxc_debian5.6' dan simpan
  ```
  kode
  ```
  
  ![B1](asset/Picture23.png)

- Cek isi http lxc_php5.dev menggunakan curl dari localhost lxc debian_php5.6
  ```
  kode
  ```

  ![B1](asset/Picture24.png)

### Soal 4. setup nginx pada ubuntu_landing untuk domain http://lxc_landing.dev , buat halaman index.html yang menerangkan informasi nama lxc
- Keluar direktori debian_php5.6 lalu masuk direktori ubuntu_landing
  ```
  kode
  ```
  
- Masuk sites-available dan edit lxc_php5.6.dev
  ```
  kode
  ```

  ![B1](asset/Picture25.png)

- Edit server name di sites-available/lxc_php5.6.dev menjadi lxc_landing.dev
  Server name adalah nama server yang nantinya akan dipanggil

  ![B1](asset/Picture26.png)

- Masuk direktori sites-enabled tampilkan isi dari folder dan symlink menggunakan:
  ```
  ls -la
  ```
  
- Tes nginx dan reload nginx
  ```
  kode
  ```
  
  ![B1](asset/Picture27.png)
  
- Setting hosts dengan masuk ke direktori /etc/hosts lalu ubah ip server name menjadi 127.0.0.1 lxc_landing.dev

  ![B1](asset/Picture28.png)

- Masuk ke direktori html namun cek terlebih dahulu isi folder menggunakan:
  ```
  ls
  ```
  
  ![B1](asset/Picture29.png)
  
  Diketahui sudah ada folder bernama lxc_php5.6 sebelumnya lalu masuk ke direktori folder tersebut dan edit fie index.html didalamnya
  ```
  nano index.html
  ```

- Ubah isi index.html menjadi informasi nama lxc 'Halaman ini dari lxc ubuntu_landing'

  ![B1](asset/Picture30.png)

- Cek isi (http://lxc_landing.dev) pada localhost lxc ubuntu_landing menggunakan curl
  ```
  kode
  ```

  ![B1](asset/Picture31.png)

### Soal 5. LXC ubuntu_landing harus auto start ketika vm dinyalakan, hal ini digunakan untuk menjaga agar website company profile tidak mengalami downtime
- Keluar terlebih dahulu dari direktori lxc ubuntu_landing lalu stop service ubuntu landing
  ```
  kode
  ```
  Cek service status ubuntu_landing menjadi stopped
  ```
  lxc-ls -f
  ``
  ![B1](asset/Picture32.png)

- Masuk root lalu ke direktori /var/lib/lxc dan masuk ke direktori ubuntu_landing/config
  ```
  kode
  ```

  ![B1](asset/Picture33.png)
  
- Tambahkan kode autostart 1
  ```
  kode
  ```
 
 ![B1](asset/Picture34.png)

- Cek apakah sudah benar autostart menjadi 1
  ```
  lxc-ls -f
  ```

  ![B1](asset/Picture35.png)

### Soal 6. setup nginx pada vm.local untuk mengatur proxy_pass dimana :
1. mengakses http://vm.local akan diarahkan ke http://lxc_landing.dev
2. mengakses http://vm.local/blog akan diarahkan ke http://lxc_php7.dev
3. mengakses http://vm.local/app akan diartahkan ke http://lxc_php5.dev

- Setting hosts pada vm
  Tambahkan ip dari debian_php5.6
  
  ![B1](asset/Picture36.png)

- Edit vm.local di sites-available
  ```
  kode
  ```

  ![B1](asset/Picture37.png)
  
- Ubah proxy_pass
  - mengakses http://vm.local akan diarahkan ke http://lxc_landing.dev :
  ```
  kode
  ```
  Fungsi rewrite ialah untuk menghapus bagian belakang dari /
  proxy_pass diganti ke http://lxc_landing.dev
  
  - mengakses http://vm.local/blog akan diarahkan ke http://lxc_php7.dev :
   ```
  kode
  ```
  Fungsi rewrite ialah untuk menghapus bagian belakang dari /blog
  proxy_pass diganti ke http://lxc_php7.dev
  
  - mengakses http://vm.local/app akan diartahkan ke http://lxc_php5.dev :
  ```
  kode
  ```
  Fungsi rewrite ialah untuk menghapus bagian belakang dari /app
  proxy_pass diganti ke http://lxc_php5.dev
  
  ![B1](asset/Picture38.png)

- Masuk sites-enabled reset nginx
  ```
  kode
  ```

  ![B1](asset/Picture39.png)

### Soal 7. untuk kebutuhan presentasi mereka, browser di laptop mereka harus dapat mengakses ketiga url tersebut.

- Cek secara local di vm apakah link berjalan dengan baik menggunakan curl
  ```
  curl -i http://vm.local/
  curl -i http://vm.local/app
  curl -i http://vm.local/blog
  ```
  
- Konfigurasi ip hosts pada laptop agar terhubung dengan virtual box dengan cara buka notepad sebagai admin lalu buka hosts pada C://windows/system32/driver/etc/hosts dan tambahkan ip virtual box yaitu 192.168.43.100 dengan server name vm.local

    ![B1](asset/Picture40.png)

- Coba pada browser
  - http://vm.local/

  ![B1](asset/Picture41.png)
  
  - http://vm.local/app

  ![B1](asset/Picture42.png)
  
  - http://vm.local/blog

  ![B1](asset/Picture43.png)


### Soal 8. Menyiapkan analisa untuk diserahkan ke CTO
 - mengapa untuk kebutuhan php5.6 tidak bisa menggunakan ubuntu 16.04, sehingga perlu diganti os ke debian 9?
    - Karena Ubuntu 16.04 xenial sudah mencapai akhir versi dan tidak support untuk php 5.6 pada bulan april tahun 2021
- kenapa harus menggunakan virtualisasi LXC pada skema website yang akan didevelop?
    - Karena skema website menggunakan beberapa sistem linux yang berbeda. Oleh karena itu, menggunakan virtualisasi LXC memudahkan untuk membuat server
- apa yang dimaksud dengan proxy server? kenapa vm.local bisa kita anggap sebagai proxy server?
    - server proxy adalah aplikasi server yang bertindak sebagai perantara antara klien yang meminta sumber daya dan server yang menyediakan sumber daya itu.
    - Dari kasus tersebut, vm.local sebagai server proxy dan halaman arahan web. Artinya vm.localis jembatan untuk menghubungkan internet
