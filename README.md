# Jarkom-Modul-2-E17-2021

```IP Prefix: 192.208```

## 1. EniesLobby akan dijadikan sebagai DNS Master, Water7 akan dijadikan DNS Slave, dan Skypie akan digunakan sebagai Web Server. Terdapat 2 Client yaitu Loguetown, dan Alabasta. Semua node terhubung pada router Foosha, sehingga dapat mengakses internet.

Topologi:<br>
![image](https://user-images.githubusercontent.com/49693862/139535627-00ed755c-0818-4c7b-ab00-53f0c0d3ce52.png)<br>

- *Foosha*
Network Config
```
auto eth0
iface eth0 inet dhcp

auto eth1
iface eth1 inet static
	address 192.208.1.1
	netmask 255.255.255.0

auto eth2
iface eth2 inet static
	address 192.208.2.1
	netmask 255.255.255.0
  ```
Set iptables untuk route paket ke NAT (internet)
```
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE -s 192.208.0.0/16
```

- *Loguetown*
Network Config
```
auto eth0
iface eth0 inet static
	address 192.208.1.2
	netmask 255.255.255.0
	gateway 192.208.1.1
```

Edit file `/etc/resolv.conf` agar dapat meresolve domain dari internet
```
nameserver 192.168.122.1
```

- *Alabasta*
```
auto eth0
iface eth0 inet static
	address 192.208.1.3
	netmask 255.255.255.0
	gateway 192.208.1.1
```

Edit file `/etc/resolv.conf` agar dapat meresolve domain dari internet
```
nameserver 192.168.122.1
```

- *EniesLobby*
```
auto eth0
iface eth0 inet static
	address 192.208.2.2
	netmask 255.255.255.0
	gateway 192.208.2.1
```

Edit file `/etc/resolv.conf` agar dapat meresolve domain dari internet
```
nameserver 192.168.122.1
```

- Water7
```
auto eth0
iface eth0 inet static
	address 192.208.2.3
	netmask 255.255.255.0
	gateway 192.208.2.1
```

Edit file `/etc/resolv.conf` agar dapat meresolve domain dari internet
```
nameserver 192.168.122.1
```

- Skypie
```
auto eth0
iface eth0 inet static
	address 192.208.2.4
	netmask 255.255.255.0
	gateway 192.208.2.1
```

Edit file `/etc/resolv.conf` agar dapat meresolve domain dari internet
```
nameserver 192.168.122.1
```

## 2. Luffy ingin menghubungi Franky yang berada di EniesLobby dengan denden mushi. Kalian diminta Luffy untuk membuat website utama dengan mengakses franky.e17.com dengan alias www.franky.e17.com pada folder kaizoku

- Install bind9 di EniesLobby
```
apt-get update
apt-get install bind9 -y
```

- Buat folder kaizoku
```
mkdir /etc/bind/kaizoku
```

- Buat file zone untuk franky.e17.com di `/etc/bind/named.conf.local` berisi
```
zone "franky.e17.com" {
    type master
    file "/etc/bind/kaizoku/franky.e17.com";
};
```

- Buat file config DNS untuk franky.e17.com di `/etc/bind/kaizoku/franky.e17.com`
```
;
; BIND data file for local loopback interface
;
$TTL    604800
@       IN      SOA     franky.e17.com. root.franky.e17.com. (
                        2021102401      ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@       IN      NS      franky.e17.com.
@       IN      A       192.208.2.4
www     IN      CNAME   franky.e17.com.
```

## 3. Setelah itu buat subdomain super.franky.e17.com dengan alias www.super.franky.e17.com yang diatur DNS nya di EniesLobby dan mengarah ke Skypie

- Edit file config DNS untuk super.franky.e17.com di `/etc/bind/kaizoku/franky.e17.com`. Untuk membuat subdomain gunakan CNAME.
```
;
; BIND data file for local loopback interface
;
$TTL    604800
@       IN      SOA     franky.e17.com. root.franky.e17.com. (
                        2021102401      ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@       IN      NS      franky.e17.com.
@       IN      A       192.208.2.4
www     IN      CNAME   franky.e17.com.
super   IN      A       192.208.2.4
www.super       IN      CNAME   super
```

## 4. Buat juga reverse domain untuk domain utama
- Edit file zona di `/etc/bind/named.conf.local` untuk menambahkan zone untuk reverse domain
```
zone "franky.e17.com" {
    type master;
    file "/etc/bind/kaizoku/franky.e17.com";
};
zone "2.208.192.in-addr.arpa" {
    type master;
    file "/etc/bind/kaizoku/2.208.192.in-addr.arpa";
};
```

- Berikut file config DNS untuk reverse domain terletak di `/etc/bind/kaizoku/2.208.192.in-addr.arpa`
```
;
; BIND data file for local loopback interface
;
$TTL    604800
@       IN      SOA     franky.e17.com. root.franky.e17.com. (
                        2021102401      ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
2.208.192.in-addr.arpa.   IN      NS      franky.e17.com.
2       IN      PTR     franky.e17.com.
```

## 5. Supaya tetap bisa menghubungi Franky jika server EniesLobby rusak, maka buat Water7 sebagai DNS Slave untuk domain utama
- Edit file zona di `/etc/bind/named.conf.local` untuk mengatur Water7 sebagai slave.
```
zone "franky.e17.com" {
    type master;
    notify yes;
    also-notify { 192.208.2.3; };
    allow-transfer { 192.208.2.3; };
    file "/etc/bind/kaizoku/franky.e17.com";
};
zone "2.208.192.in-addr.arpa" {
    type master;
    file "/etc/bind/kaizoku/2.208.192.in-addr.arpa";
};
```

## 6. Setelah itu terdapat subdomain mecha.franky.e17.com dengan alias www.mecha.franky.e17.com yang didelegasikan dari EniesLobby ke Water7 dengan IP menuju ke Skypie dalam folder sunnygo
- Edit file config DNS untuk pendelegasian mecha.franky.e17.com di `/etc/bind/kaizoku/franky.e17.com`. Untuk mentransfer, gunakan rule NS. 
```
;
; BIND data file for local loopback interface
;
$TTL    604800
@       IN      SOA     franky.e17.com. root.franky.e17.com. (
                        2021102401      ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@       IN      NS      franky.e17.com.
@       IN      A       192.208.2.4 ;IP EniesLobby
www     IN      CNAME   franky.e17.com.
super   IN      A       192.208.2.4 ;IP Skypie
www.super       IN      CNAME   super
ns1     IN      A       192.208.2.3 ;IP Water7
mecha   IN      NS      ns1
```

- Water7 belum mengenali subdomain mecha.franky.e17.com, oleh karena itu perlu menambahkan zona pada Water7.
```
zone "franky.e17.com" {
    type slave;
    masters { 192.208.2.2; };
    file "/var/lib/bind/franky.e17.com";
};
zone "mecha.franky.e17.com" {
    type master;
    file "/etc/bind/sunnygo/mecha.franky.e17.com";
};
```

Zone yang pertama untuk set bahwa Water7 adalah slave dari EniesLobby, zone kedua untuk menjawab query atas domain mecha.franky.e17.com. Dimana file config terletak di `/etc/bind/sunnygo/mecha.franky.e17.com`.
- Untuk itu, buat file untuk menjawab query atas mecha.franky.e17.com di `/etc/bind/sunnygo/mecha.franky.e17.com`
```
;
; BIND data file for local loopback interface
;
$TTL    604800
@       IN      SOA     mecha.franky.e17.com. root.mecha.franky.e17.com. (
                        2021102400      ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@       IN      NS      mecha.franky.e17.com.
@       IN      A       192.208.2.4
www       IN      CNAME   mecha.franky.e17.com.
```

## 7. Untuk memperlancar komunikasi Luffy dan rekannya, dibuatkan subdomain melalui Water7 dengan nama general.mecha.franky.e17.com dengan alias www.general.mecha.franky.e17.com yang mengarah ke Skypie
- Untuk menjawab soal nomor 7, tambahkan CNAME general di `/etc/bind/sunnygo/mecha.franky.e17.com`
```
;
; BIND data file for local loopback interface
;
$TTL    604800
@       IN      SOA     mecha.franky.e17.com. root.mecha.franky.e17.com. (
                        2021102400      ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@       IN      NS      mecha.franky.e17.com.
@       IN      A       192.208.2.4
www       IN      CNAME   mecha.franky.e17.com.
general IN      A       192.208.2.4
www.general     IN      CNAME   general
```

## 8. Setelah melakukan konfigurasi server, maka dilakukan konfigurasi Webserver. Pertama dengan webserver www.franky.e17.com. Pertama, luffy membutuhkan webserver dengan DocumentRoot pada /var/www/franky.e17.com.
- Instalasi apache dan modul php
```
apt-get update
apt-get install apache2 -y
apt-get install php -y
apt-get install libapache2-mod-php7.0 -y
```

- Buat file config di `/etc/apache2/sites-available` dengan nama `/etc/apache2/sites-available/www.franky.e17.com.conf` 
```
<VirtualHost *:80>
        ServerAdmin webmaster@localhost
        ServerName franky.e17.com
        ServerAlias www.franky.e17.com
        DocumentRoot /var/www/franky.e17.com

        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined

</VirtualHost>
```

- Enable config dan jangan lupa restart
```
a2ensite www.franky.e17.com.conf
service apache2 restart
```

## 9. Setelah itu, Luffy juga membutuhkan agar url www.franky.e17.com/index.php/home dapat menjadi menjadi www.franky.e17.com/home. '
- Untuk menjawab soal nomor 9, gunakan alias pada config `www.franky.e17.com.conf`. Sehingga file tersebut sekarang berisi:
```
<VirtualHost *:80>
        ServerAdmin webmaster@localhost
        ServerName franky.e17.com
        ServerAlias www.franky.e17.com
        DocumentRoot /var/www/franky.e17.com

        Alias "/home" "/var/www/franky.e17.com/index.php/home"

        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined

</VirtualHost>
```

## 10. Setelah itu, pada subdomain www.super.franky.yyy.com, Luffy membutuhkan penyimpanan aset yang memiliki DocumentRoot pada /var/www/super.franky.yyy.com
- Buat file config di `/etc/apache2/sites-available/www.super.franky.e17.com` berisi
```
<VirtualHost *:80>
    ServerAdmin webmaster@localhost
    ServerName super.franky.e17.com
    ServerAlias www.super.franky.e17.com
    DocumentRoot /var/www/super.franky.e17.com
</VirtualHost>
```

- Enable config dan jangan lupa restart
```
a2ensite www.super.franky.e17.com.conf
service apache2 restart
```

## 11. Akan tetapi, pada folder /public, Luffy ingin hanya dapat melakukan directory listing saja.
- Edit file config di `/etc/apache2/sites-available/www.super.franky.e17.com` sehingga berisi sebagai berikut. `Options +Indexes` memberikan izin untuk melakukan directory listing di directory tersebut. Jangan lupa restart apache2 setelah membuat perubahan
```
<VirtualHost *:80>
    ServerAdmin webmaster@localhost
    ServerName super.franky.e17.com
    ServerAlias www.super.franky.e17.com
    DocumentRoot /var/www/super.franky.e17.com
    
    <Directory ~ /var/www/super.franky.e17.com/public/.*>
        Options -Indexes
    </Directory>

    <Directory ~ /var/www/super.franky.e17.com/public/$>
        Options +Indexes
    </Directory>
</VirtualHost>
```

## 12. Tidak hanya itu, Luffy juga menyiapkan error file 404.html pada folder /error untuk mengganti error kode pada apache .
- Ubah lagi config `/etc/apache2/sites-available/www.super.franky.e17.com`dengan menambahkan ErrorDocument sebagai berikut disertai dengan kode errornya.
```
<VirtualHost *:80>
    ServerAdmin webmaster@localhost
    ServerName super.franky.e17.com
    ServerAlias www.super.franky.e17.com
    DocumentRoot /var/www/super.franky.e17.com
    
    <Directory ~ /var/www/super.franky.e17.com/public/.*>
        Options -Indexes
    </Directory>

    <Directory ~ /var/www/super.franky.e17.com/public/$>
        Options +Indexes
    </Directory>
    
    # Set error page
    ErrorDocument 404 /error/404.html
</VirtualHost>
```

## 13. Luffy juga meminta Nami untuk dibuatkan konfigurasi virtual host. Virtual host ini bertujuan untuk dapat mengakses file asset www.super.franky.yyy.com/public/js menjadi www.super.franky.yyy.com/js.
- Untuk menjawab soal ini, gunakan alias untuk mengaliaskan /js dengan folder /public/js. Setelah itu allow untuk listing directory dengan menggunakan LocationMatch (LocationMatch diambil dari URL, sedangkan Directory merujuk aturan untuk target directory)
```
<VirtualHost *:80>
    ServerAdmin webmaster@localhost
    ServerName super.franky.e17.com
    ServerAlias www.super.franky.e17.com
    DocumentRoot /var/www/super.franky.e17.com

    Alias "/js" "/var/www/super.franky.e17.com/public/js"

    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined

    <Directory /var/www/super.franky.e17.com>
        Options +FollowSymLinks -Multiviews
        AllowOverride All
    </Directory>
    
    <Directory ~ /var/www/super.franky.e17.com/public/.*>
        Options -Indexes
    </Directory>

    <Directory ~ /var/www/super.franky.e17.com/public/$>
        Options +Indexes
    </Directory>

    <Directory ~ /var/www/super.franky.e17.com/public/images>
        Options +Indexes
    </Directory>
    
    <LocationMatch "^/js">
        Options +Indexes
    </LocationMatch>    
    
    # Set error page
    ErrorDocument 404 /error/404.html
</VirtualHost>
```

## 14. Dan Luffy meminta untuk web www.general.mecha.franky.yyy.com hanya bisa diakses dengan port 15000 dan port 15500 
- Apache secara default tidak LISTEN ke port tersebut, ubah file `/etc/apache2/ports.conf` untuk membuat apache LISTEN di 2 port tersebut.
```
Listen 80
Listen 15000
Listen 15500

<IfModule ssl_module>
        Listen 443
</IfModule>

<IfModule mod_gnutls.c>
        Listen 443
</IfModule>
```

- Buat config untuk akses dengan port 15000 dan 15500.
> File `/etc/apache2/sites-available/www.general.mecha.franky.e17.com-15000.conf`
```
<VirtualHost *:15000>
        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/general.mecha.franky.e17.com
        ServerName general.mecha.franky.e17.com
        ServerAlias www.general.mecha.franky.e17.com

        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined

        <Directory "var/www/general.mecha.franky.e17.com">
            AuthType Basic
            AuthName "Restricted Content"
            AuthUserFile /etc/apache2/.htpasswd
            Require valid-user
        </Directory>
</VirtualHost>
```

> File `/etc/apache2/sites-available/www.general.mecha.franky.e17.com-15500.conf`
```
<VirtualHost *:15500>
        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/general.mecha.franky.e17.com
        ServerName general.mecha.franky.e17.com
        ServerAlias www.general.mecha.franky.e17.com

        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined

        <Directory "var/www/general.mecha.franky.e17.com">
            AuthType Basic
            AuthName "Restricted Content"
            AuthUserFile /etc/apache2/.htpasswd
            Require valid-user
        </Directory>
</VirtualHost>
```

- Jangan lupa enable dan restart apache
```
a2ensite www.general.mecha.franky.e17.com-15000.conf
a2ensite www.general.mecha.franky.e17.com-15500.conf
service apache2 restart
```

## 15. dengan autentikasi username luffy dan password onepiece dan file di /var/www/general.mecha.franky.yyy
- Pertama, buat dulu file berisi username dan password dengan command `htpasswd -c <lokasi_file> <nama_user>`. Setelah file jadi, letakkan di `/etc/apache/htpasswd`. Lalu edit config untuk kedua port, tambahkan line config berikut. Setelah di set seperti ini, maka domain akan memerlukan http basic auth dengan credential yang sudah dibuat tadi. Jangan lupa untuk restart apache setelah mengubah.
```
...
<Directory "var/www/general.mecha.franky.e17.com">
    AuthType Basic
    AuthName "Restricted Content"
    AuthUserFile /etc/apache2/.htpasswd
    Require valid-user
</Directory>
</VirtualHost>
```

## 16. Dan setiap kali mengakses IP Skypie akan dialihkan secara otomatis ke www.franky.yyy.com
- Edit file `/etc/apache2/sites-available/default` dengan isi dibawah. Redirect berguna untuk mengalihkan ke www.franky.e17.com
```
<VirtualHost *:80>
        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/franky.e17.com

        Redirect / http://franky.e17.com
        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined

</VirtualHost>
```

## 17. Dikarenakan Franky juga ingin mengajak temannya untuk dapat menghubunginya melalui website www.super.franky.yyy.com, dan dikarenakan pengunjung web server pasti akan bingung dengan randomnya images yang ada, maka Franky juga meminta untuk mengganti request gambar yang memiliki substring “franky” akan diarahkan menuju franky.png. 
- Aktifkan apache2 rewrite module mod
```
a2enmod rewrite
```

- Untuk memenuhi soal nomor 17, gunakan .htaccess dan letakkan pada `/var/www/super.franky.e17.com/.htaccess`. Berisi:
```
RewriteEngine On
RewriteBase /
RewriteRule .*franky.*jpg$ http://super.franky.e17.com/public/images/franky.png$1 [L,R=301]
```
> Parameter pertama RewriteRule adalah regex kapan rewrite akan dilakukan. Tertulis bila di url terdapat substring franky, dan berujung jpg.
> Parameter kedua adalah lokasi tujuan bila parameter satu matched.

- Jangan lupa restart apache
