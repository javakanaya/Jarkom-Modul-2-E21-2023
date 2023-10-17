# Jarkom-Modul-2-E21-2023

Laporan Resmi Praktikum Jaringan Komputer Modul 2

## Nama Anggota:

- [Yoel Mountanus Sitorus](https://github.com/zemetia) - 5025211078
- [Java Kanaya Prada](https://github.com/javakanaya) - 5025211112

## Topologi

<img width="1057" alt="Topologi" src="https://github.com/javakanaya/Jarkom-Modul-2-E21-2023/assets/87474722/a08d9785-04d2-414c-9e9e-bf14297e2537">

## Network Configuration

- Router

```
auto eth0
iface eth0 inet dhcp

auto eth1
iface eth1 inet static
	address 10.47.1.1
	netmask 255.255.255.0

auto eth2
iface eth2 inet static
	address 10.47.2.1
	netmask 255.255.255.0
```

- NakulaClient

```
auto eth0
iface eth0 inet static
    address 10.47.1.2
    netmask 255.255.255.0
    gateway 10.47.1.1
```

- SadewaClient

```
auto eth0
iface eth0 inet static
    address 10.47.1.3
    netmask 255.255.255.0
    gateway 10.47.1.1
```

- AbimanyuWebServer

```
auto eth0
iface eth0 inet static
    address 10.47.1.4
    netmask 255.255.255.0
    gateway 10.47.1.1
```

- PrabukusumaWebServer

```
auto eth0
iface eth0 inet static
    address 10.47.1.5
    netmask 255.255.255.0
    gateway 10.47.1.1
```

- WisanggeniWebServer

```
auto eth0
iface eth0 inet static
    address 10.47.1.6
    netmask 255.255.255.0
    gateway 10.47.1.1
```

- ArjunaLoadBalancer

```
auto eth0
iface eth0 inet static
    address 10.47.2.2
    netmask 255.255.255.0
    gateway 10.47.2.1
```

- WerkudaraDNSSlave

```
auto eth0
iface eth0 inet static
 	address 10.47.2.3
 	netmask 255.255.255.0
 	gateway 10.47.2.1
```

- YudhistiraDNSMaster

```
auto eth0
iface eth0 inet static
    address 10.47.2.4
 	netmask 255.255.255.0
 	gateway 10.47.2.1
```

### Sebelum memulai

Pada node **Router**, jalankan perintah berikut, atau juga dapat menuliskan perintah berikut pada file `.bashrc`, agar perintah tersebut tetap dijalankan saat kita membuka-menutup proyek.

```
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE -s 10.47.0.0/16
```

## Soal 1

> Yudhistira akan digunakan sebagai DNS Master, Werkudara sebagai DNS Slave, Arjuna merupakan Load Balancer yang terdiri dari beberapa Web Server yaitu Prabakusuma, Abimanyu, dan Wisanggeni. Buatlah topologi dengan pembagian [sebagai berikut](https://docs.google.com/spreadsheets/d/1OqwQblR_mXurPI4gEGqUe7v0LSr1yJViGVEzpMEm2e8/edit?usp=sharing). Folder topologi dapat diakses pada [drive berikut](https://drive.google.com/drive/folders/1Ij9J1HdIW4yyPEoDqU1kAwTn_iIxg3gk?usp=sharing)

### _Script_

Untuk menghubungkan semua node pada jaringan internet, kita perlu menjalankan perintah berikut pada masing-masing node.

```sh
echo 'nameserver 192.168.122.1    # IP DNS' > /etc/resolv.conf
```

IP tersebut didapatkan dari file `/etc/resolv.conf` yang ada di node Router.

Untuk pengujian, kita bisa menjalankan perintah berikut. Disini kami melakukan pengujian pada node client, yaitu **NakulaClient**, dan **SadewaClient**.

```
ping google.com -c 5
```

### Hasil

<img width="1580" alt="image" src="https://github.com/javakanaya/Jarkom-Modul-2-E21-2023/assets/87474722/8bc7a0d0-8ef2-4d1c-8cec-075421065de5">
<img width="1580" alt="image" src="https://github.com/javakanaya/Jarkom-Modul-2-E21-2023/assets/87474722/6eec7e1f-3fe6-4fd8-afe9-db3adfd5bab2">


## Soal 2

> Buatlah website utama pada node arjuna dengan akses ke **arjuna.yyy.com** dengan alias **www.arjuna.yyy.com** dengan yyy merupakan kode kelompok.

### _Script_

Pada **YudhistiraDNSMaster** Pastikan pada file `/etc/resolv.conf/` sudah terdapat nameserver seperti pada soal 1. Kemudian jalankan script berikut.

```sh
# update
apt-get update
apt-get install bind9 -y

# konfigurasi etc/bind/named.conf.local
echo 'zone "arjuna.e21.com" {
    type master;
    notify yes;
    file "/etc/bind/arjuna/arjuna.e21.com";
};' > /etc/bind/named.conf.local

mkdir /etc/bind/arjuna

# konfigurasi domain arjuna.e21.com
echo ';
; BIND data file for local loopback interface
;
$TTL  604800
@       IN    SOA       arjuna.e21.com. root.arjuna.e21.com. (
                        2023101001      ; Serial
                            604800      ; Refresh
                             86400      ; Retry
                           2419200      ; Expire
                            604800 )    ; Negative Cache TTL

;
@           IN  NS      arjuna.e21.com.
@           IN  A       10.47.2.2   ; IP Arjuna
www         IN  CNAME   arjuna.e21.com.' > /etc/bind/arjuna/arjuna.e21.com

service bind9 restart
```

Kemudian untuk melakukan pengujian, jalankan script berikut pada node client, agar node client terhubung ke **YudhistiraDNSMaster**

```sh
echo '
nameserver 10.47.2.4        # IP Yudhistira
nameserver 192.168.122.1    # IP Router' > /etc/resolv.conf
```

### Hasil

Untuk melakukan pengujian, jalankan perintah berikut pada node client, yaitu **NakulaClient** atau **SadewaClient**

```
ping arjuna.e21.com -c 5
ping www.arjuna.e21.com -c 5
```

<img width="1580" alt="image" src="https://github.com/javakanaya/Jarkom-Modul-2-E21-2023/assets/87474722/220e51e6-8670-41f4-82f1-1e6f531c855e">


## Soal 3

> Dengan cara yang sama seperti soal nomor 2, buatlah website utama dengan akses ke **abimanyu.yyy.com** dan alias **www.abimanyu.yyy.com**.

### _Script_

Melanjutkan pada soal nomor 2, maka kita perlu menambahkan konfigurasi untuk domain **abimanyu.e21.com** pada **YudhistiraDNSMaster** dengan script sebagai berikut:

```sh
# konfigurasi etc/bind/named.conf.local
echo 'zone "arjuna.e21.com" {
    type master;
    notify yes;
    file "/etc/bind/arjuna/arjuna.e21.com";
};

zone "abimanyu.e21.com" {
    type master;
    notify yes;
    file "/etc/bind/abimanyu/abimanyu.e21.com";
};' > /etc/bind/named.conf.local

mkdir /etc/bind/abimanyu

# konfigurasi domain abimanyu.e21.com
echo '
;
; BIND data file for local loopback interface
;
$TTL  604800
@       IN    SOA       abimanyu.e21.com. root.abimanyu.e21.com. (
                        2023101001      ; Serial
                            604800      ; Refresh
                             86400      ; Retry
                           2419200      ; Expire
                            604800 )    ; Negative Cache TTL

;
@               IN  NS      abimanyu.e21.com.
@               IN  A       10.47.1.4                   ; IP Abimanyu
www             IN  CNAME   abimanyu.e21.com.
@               IN  AAAA    ::1' > /etc/bind/abimanyu/abimanyu.e21.com

service bind9 restart
```

### Hasil

Lalu untuk pengujiannya, sama seperti sebelumnya, jalankan perintah berikut pada node client, yaitu **NakulaClient** atau **SadewaClient**

```
ping abimanyu.e21.com -c 5
ping www.abimanyu.e21.com -c 5
```

<img width="1580" alt="image" src="https://github.com/javakanaya/Jarkom-Modul-2-E21-2023/assets/87474722/cb6936c2-f669-4975-8e90-fd049727a2aa">


## Soal 4

> Kemudian, karena terdapat beberapa web yang harus di-deploy, buatlah subdomain **parikesit.abimanyu.yyy.com** yang diatur DNS-nya di Yudhistira dan mengarah ke Abimanyu.

### _Script_

Berikutnya untuk menambahkan subdomain **parikesit.abimanyu.e21.com**, maka dilakukan konfigurasi pada file `/etc/bind/abimanyu/abimanyu.e21.com` pada **YudhistiraDNSMaster** sebagai berikut:

```sh
# konfigurasi domain abimanyu.e21.com
echo ';
; BIND data file for local loopback interface
;
$TTL  604800
@       IN    SOA       abimanyu.e21.com. root.abimanyu.e21.com. (
                        2023101001      ; Serial
                            604800      ; Refresh
                             86400      ; Retry
                           2419200      ; Expire
                            604800 )    ; Negative Cache TTL

;
@               IN  NS      abimanyu.e21.com.
@               IN  A       10.47.1.4                   ; IP Abimanyu
www             IN  CNAME   abimanyu.e21.com.
parikesit       IN  A       10.47.1.4                   ; IP Abimanyu
www.parikesit   IN  CNAME   parikesit.abimanyu.e21.com.
@               IN  AAAA    ::1' > /etc/bind/abimanyu/abimanyu.e21.com

service bind9 restart
```

### Hasil

Lalu untuk pengujiannya, jalankan perintah berikut pada node client, yaitu **NakulaClient** atau **SadewaClient**

```
ping parikesit.abimanyu.e21.com -c 5
ping www.parikesit.abimanyu.e21.com -c 5
```

<img width="1580" alt="image" src="https://github.com/javakanaya/Jarkom-Modul-2-E21-2023/assets/87474722/2dc3c283-a421-4b89-9649-8ed1f9a97e26">


Terlihat bahwa IP-nya juga mengarah ke IP Abimanyu, yaitu `10.47.1.4`

## Soal 5

> Buat juga reverse domain untuk domain utama. (Abimanyu saja yang direverse)

### _Script_

Untuk membuat reverse domain **abimanyu.e21.com**, gunakan script berikut pada **YudhistiraDNSMaster**:

```sh
# konfigurasi etc/bind/named.conf.local
echo 'zone "arjuna.e21.com" {
    type master;
    notify yes;
    file "/etc/bind/arjuna/arjuna.e21.com";
};

zone "abimanyu.e21.com" {
    type master;
    notify yes;
    file "/etc/bind/abimanyu/abimanyu.e21.com";
};

zone "1.47.10.in-addr.arpa" {
    type master;
    notify yes;
    file "/etc/bind/abimanyu/1.47.10.in-addr.arpa";
};' > /etc/bind/named.conf.local

# konfigurasi reverse domain 1.47.10.in-addr.arpa.
echo ';
; BIND data file for local loopback interface
;
$TTL  604800
@       IN    SOA       abimanyu.e21.com. root.abimanyu.e21.com. (
                        2023101001      ; Serial
                            604800      ; Refresh
                             86400      ; Retry
                           2419200      ; Expire
                            604800 )    ; Negative Cache TTL
;
1.47.10.in-addr.arpa.   IN  NS  abimanyu.e21.com.
4                       IN  PTR abimanyu.e21.com.' > /etc/bind/abimanyu/1.47.10.in-addr.arpa

service bind9 restart
```

Untuk melakukan pengujian install dnsutils pada node client

```sh
apt-get dnsutils
```

### Hasil

Lalu untuk melakukan pengujian pada node client, jalankan perintah berikut. (`10.47.1.4` merupakan IP abimanyu)

```
host -t PTR 10.47.1.4
```

<img width="1580" alt="image" src="https://github.com/javakanaya/Jarkom-Modul-2-E21-2023/assets/87474722/bf9fd765-0a74-4941-a481-7ec1e2da749f">


## Soal 6

> Agar dapat tetap dihubungi ketika DNS Server Yudhistira bermasalah, buat juga Werkudara sebagai DNS Slave untuk domain utama.

### _Script_

Untuk membuat DNS Slave, maka ditambahkan `also-notify` dan `allow-transfer`, pada konfigurasi setiap domain di file `/etc/bind/named.conf.local`. Gunakan script berikut pada **YudhistiraDNSMaster**:

```sh
# konfigurasi etc/bind/named.conf.local
echo 'zone "arjuna.e21.com" {
    type master;
    notify yes;
    also-notify { 10.47.2.3; };     // IP Werkudara
    allow-transfer { 10.47.2.3; };  // IP Werkudara
    file "/etc/bind/arjuna/arjuna.e21.com";
};

zone "abimanyu.e21.com" {
    type master;
    notify yes;
    also-notify { 10.47.2.3; };     // IP Werkudara
    allow-transfer { 10.47.2.3; };  // IP Werkudara
    file "/etc/bind/abimanyu/abimanyu.e21.com";
};

zone "1.47.10.in-addr.arpa" {
    type master;
    notify yes;
    also-notify { 10.47.2.3; };     // IP Werkudara
    allow-transfer { 10.47.2.3; };  // IP Werkudara
    file "/etc/bind/abimanyu/1.47.10.in-addr.arpa";
};' > /etc/bind/named.conf.local

service bind9 restart
```

Kemudian pada **WerkudaraDNSSlave** jalankan script berikut:

```sh
echo '
nameserver 10.47.2.4        # IP Yudhistira
nameserver 10.47.2.3        # IP Werkudara
nameserver 192.168.122.1    # IP Router' > /etc/resolv.conf

# update
apt-get update
apt-get install bind9 -y

echo 'zone "arjuna.e21.com" {
    type slave;
    masters { 10.47.2.4; }; // IP Yudhistira
    file "/var/lib/bind/arjuna.e21.com";
};

zone "abimanyu.e21.com" {
    type slave;
    masters { 10.47.2.4; }; // IP Yudhistira
    file "/var/lib/bind/abimanyu.e21.com";
};

zone "1.47.10.in-addr.arpa" {
    type slave;
    masters { 10.47.2.4; }; // IP Yudhistira
    file "/etc/bind/1.47.10.in-addr.arpa";
};
' > /etc/bind/named.conf.local

service bind9 restart
```

Untuk melakukan pengujian hentikan service bind9 pada **YudhistiraDNSMaster**

```sh
service bind9 stop
```

### Hasil

Lalu untuk melakukan pengujian pada node client, panggil domain yang telah dibuat, dan seharusnya akan tetap terhubung, walaupun _service bind9_ pada **YudhistiraDNSMaster** sudah dimatikan.

```
ping abimanyu.e21.com -c 5
ping arjuna.e21.com -c 5
```

<img width="1580" alt="image" src="https://github.com/javakanaya/Jarkom-Modul-2-E21-2023/assets/87474722/a9e818cf-7e83-4bdf-bb7b-cc649df8bdc8">


## Soal 7

> Seperti yang kita tahu karena banyak sekali informasi yang harus diterima, buatlah subdomain khusus untuk perang yaitu **baratayuda.abimanyu.yyy.com** dengan alias **www.baratayuda.abimanyu.yyy.com** yang didelegasikan dari Yudhistira ke Werkudara dengan IP menuju ke Abimanyu dalam folder Baratayuda.

### _Script_

Untuk membuat delegasi subdomain, kita perlu menambahkan:

```
baratayuda      IN      NS      ns1.abimanyu.e21.com.
ns1             IN      A       10.47.2.3     ; IP Werkudara
```

pada file `/etc/bind/abimanyu/abimanyu.e21.com` agar mendapatkan authoritative terhadap Werkudara. Kita juga perlu mengaktifkan `allow-query { any; };` pada file `/etc/bind/named.conf.options`. Untuk melakukannya jalankan script berikut pada **YudhistiraDNSMaster**:

```sh
# konfigurasi domain abimanyu.e21.com
echo ';
; BIND data file for local loopback interface
;
$TTL  604800
@       IN    SOA       abimanyu.e21.com. root.abimanyu.e21.com. (
                        2023101001      ; Serial
                            604800      ; Refresh
                             86400      ; Retry
                           2419200      ; Expire
                            604800 )    ; Negative Cache TTL

;
@               IN  NS      abimanyu.e21.com.
@               IN  A       10.47.1.4                   ; IP Abimanyu
www             IN  CNAME   abimanyu.e21.com.
parikesit       IN  A       10.47.1.4                   ; IP Abimanyu
www.parikesit   IN  CNAME   parikesit.abimanyu.e21.com.
baratayuda      IN  NS      ns1.abimanyu.e21.com.
ns1             IN  A       10.47.2.3                   ; IP Werkudara
@               IN  AAAA    ::1' > /etc/bind/abimanyu/abimanyu.e21.com

# konfigurasi etc/bind/named.conf.options
echo 'options {
    directory "/var/cache/bind";

    allow-query { any; };

    auth-nxdomain no;   # conform to RFC1035
    listen-on-v6 {any; };
};' > /etc/bind/named.conf.options

service bind9 restart
```

Lalu pada **WerkudaraDNSSlave** jalankan script berikut:

```sh
mkdir /etc/bind/baratayuda

echo 'zone "arjuna.e21.com" {
    type slave;
    masters { 10.47.2.4; }; // IP Yudhistira
    file "/var/lib/bind/arjuna.e21.com";
};

zone "abimanyu.e21.com" {
    type slave;
    masters { 10.47.2.4; }; // IP Yudhistira
    file "/var/lib/bind/abimanyu.e21.com";
};

zone "1.47.10.in-addr.arpa" {
    type slave;
    masters { 10.47.2.4; }; // IP Yudhistira
    file "/etc/bind/1.47.10.in-addr.arpa";
};

zone "baratayuda.abimanyu.e21.com" {
    type master;
    file "/etc/bind/baratayuda/baratayuda.abimanyu.e21.com";
};
' > /etc/bind/named.conf.local

echo ';
; BIND data file for local loopback interface
;
$TTL    604800
@       IN      SOA     baratayuda.abimanyu.e21.com. root.baratayuda.abimanyu.e21.com. (
                     2023101001         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@                       IN      NS      baratayuda.abimanyu.e21.com.
@                       IN      A       10.47.1.4                       ; IP Abimanyu
www                     IN      CNAME   baratayuda.abimanyu.e21.com.
' > /etc/bind/baratayuda/baratayuda.abimanyu.e21.com

echo 'options {
    directory "/var/cache/bind";

    allow-query{any;};

    auth-nxdomain no;   # conform to RFC1035
    listen-on-v6 { any; };
};
' > /etc/bind/named.conf.options

service bind9 restart
```

### Hasil

Lalu untuk pengujiannya, jalankan perintah berikut pada node client, yaitu **NakulaClient** atau **SadewaClient**

```
ping baratayuda.abimanyu.e21.com -c 5
ping www.baratayuda.abimanyu.e21.com -c 5
```

<img width="1580" alt="image" src="https://github.com/javakanaya/Jarkom-Modul-2-E21-2023/assets/87474722/a1624095-b4d4-4771-ac69-d7863310eef4">

## Soal 8

> Untuk informasi yang lebih spesifik mengenai Ranjapan Baratayuda, buatlah subdomain melalui Werkudara dengan akses **rjp.baratayuda.abimanyu.yyy.com** dengan alias **www.rjp.baratayuda.abimanyu.yyy.com** yang mengarah ke Abimanyu.

### _Script_

Untuk membuat subdomain, jalankan script berikut pada **WerkudaraDNSSlave**(karena subdomain tersebut telah didelegasikan ke node tersebut):

```sh
echo ';
; BIND data file for local loopback interface
;
$TTL    604800
@       IN      SOA     baratayuda.abimanyu.e21.com. root.baratayuda.abimanyu.e21.com. (
                     2023101001         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@                       IN      NS      baratayuda.abimanyu.e21.com.
@                       IN      A       10.47.1.4                       ; IP Abimanyu
www                     IN      CNAME   baratayuda.abimanyu.e21.com.
rjp                     IN      A       10.47.1.4                       ; IP Abimanyu
www.rjp                 IN      CNAME   rjp.baratayuda.abimanyu.e21.com.
' > /etc/bind/baratayuda/baratayuda.abimanyu.e21.com

service bind9 restart
```

### Hasil

Lalu untuk pengujiannya, jalankan perintah berikut pada node client, yaitu **NakulaClient** atau **SadewaClient**

```
ping rjp.baratayuda.abimanyu.e21.com -c 5
ping www.rjp.baratayuda.abimanyu.e21.com -c 5
```

<img width="1580" alt="image" src="https://github.com/javakanaya/Jarkom-Modul-2-E21-2023/assets/87474722/fa13d34a-2c35-405a-8a5f-22c8a16376ec">


## Soal 9

> Arjuna merupakan suatu Load Balancer Nginx dengan tiga worker (yang juga menggunakan nginx sebagai webserver) yaitu Prabakusuma, Abimanyu, dan Wisanggeni. Lakukan deployment pada masing-masing worker.

### _Script_

Jalankan Script berikut pada **ArjunaLoadBalancer**:s

```sh
# Menghubungkan nameserver
echo '
nameserver 10.47.2.4        # IP Yudhistira
nameserver 10.47.2.3        # IP Werkudara
nameserver 192.168.122.1    # IP Router' > /etc/resolv.conf

# Update,dan instalasi
apt-get update
apt-get install dnsutils lynx -y
apt-get install nginx bind9 -y

rm /etc/nginx/sites-enabled/default

# Nginx sites-available config untuk worker loadbalancer
echo ' # Default menggunakan Round Robin
 upstream myweb  {
    server 10.47.1.5; #IP Prabukusuma
 	server 10.47.1.4; #IP Abimanyu
    server 10.47.1.6s; #IP Wisanggeni
 }

 server {
 	listen 80;
 	server_name arjuna.e21.com www.arjuna.e21.com;

 	location / {
 	proxy_pass http://myweb;
 	}
 }' > /etc/nginx/sites-available/praktikum23-loadbalancer

ln -s /etc/nginx/sites-available/praktikum23-loadbalancer /etc/nginx/sites-enabled/praktikum23-loadbalancer

service nginx restart
```

Lalu pada setiap worker (**AbimanyuWerbServer**, **PrabukusumaWebServer**, **WisanggerniWebServer**) jalankan script berikut:

```sh
echo '
nameserver 10.47.2.4        # IP Yudhistira
nameserver 10.47.2.3        # IP Werkudara
nameserver 192.168.122.1    # IP Router' > /etc/resolv.conf

apt-get update
apt-get install dnsutils lynx -y
apt-get install nginx php php-fpm -y

mkdir /var/www/praktikum23
rm /etc/nginx/sites-enabled/default

# index.php pada worker server prabukusuma
echo "<?php
\$hostname = gethostname();
\$date = date('Y-m-d H:i:s');
\$php_version = phpversion();
\$username = get_current_user();

echo \"Hello World!<br>\";
echo \"Saya adalah: \$username<br>\";
echo \"Saat ini berada di: \$hostname<br>\";
echo \"Versi PHP yang saya gunakan: \$php_version<br>\";
echo \"Tanggal saat ini: \$date<br>\";
?>
" > /var/www/praktikum23/index.php

# Nginx sites-available configuration
echo '
server {

 	listen 80;

 	root /var/www/praktikum23;

 	index index.php index.html index.htm;
 	server_name _;

 	location / {
 			try_files $uri $uri/ /index.php?$query_string;
 	}

 	# pass PHP scripts to FastCGI server
 	location ~ \.php$ {
 	include snippets/fastcgi-php.conf;
 	fastcgi_pass unix:/var/run/php/php7.2-fpm.sock;
 	}

    location ~ /\.ht {
 			deny all;
 	}

 	error_log /var/log/nginx/praktikum23_error.log;
 	access_log /var/log/nginx/praktikum23_access.log;
}
' > /etc/nginx/sites-available/praktikum23

ln -s /etc/nginx/sites-available/praktikum23 /etc/nginx/sites-enabled/praktikum23

service nginx restart
service php7.2-fpm stop
service php7.2-fpm start
```

Untuk melakukan pengujian install lynx pada node client

```sh
apt-get lynx
```

### Hasil

Lalu untuk pengujiannya, buka **arjuna.e21.com** dengan perintah berikut pada node client, yaitu **NakulaClient** atau **SadewaClient**:

```
lynx 10.47.1.5
lynx 10.47.1.4
lynx 10.47.1.6
lynx arjuna.e21.com
```

<img width="1580" alt="image" src="https://github.com/javakanaya/Jarkom-Modul-2-E21-2023/assets/87474722/b6ce704c-03b9-4b80-a1ef-90771b053471">
<img width="1580" alt="image" src="https://github.com/javakanaya/Jarkom-Modul-2-E21-2023/assets/87474722/221a5caf-ab6f-4270-a3ee-139802a95e0c">
<img width="1580" alt="image" src="https://github.com/javakanaya/Jarkom-Modul-2-E21-2023/assets/87474722/5e6d8ee1-e3c5-4b1b-8780-eefaf434eee9">


## Soal 10

> Kemudian gunakan algoritma **Round Robin** untuk Load Balancer pada **Arjuna**. Gunakan server_name pada soal nomor 1. Untuk melakukan pengecekan akses alamat web tersebut kemudian pastikan worker yang digunakan untuk menangani permintaan akan berganti ganti secara acak. Untuk webserver di masing-masing worker wajib berjalan di port 8001-8003.
> Contoh:
>
> - Prabakusuma:8001
> - Abimanyu:8002
> - Wisanggeni:8003

### _Script_

Perbarui file `/etc/nginx/sites-available/praktikum23-loadbalancer`, dengan menggunakan script berikut:

```sh
# Nginx sites-available config untuk worker loadbalancer
echo ' # Default menggunakan Round Robin
 upstream myweb  {
    server 10.47.1.5:8001; #IP Prabukusuma
 	server 10.47.1.4:8002; #IP Abimanyu
    server 10.47.1.6:8003; #IP Wisanggeni
 }

 server {
 	listen 80;
 	server_name arjuna.e21.com www.arjuna.e21.com;

 	location / {
 	proxy_pass http://myweb;
 	}
 }' > /etc/nginx/sites-available/praktikum23-loadbalancer

ln -s /etc/nginx/sites-available/praktikum23-loadbalancer /etc/nginx/sites-enabled/praktikum23-loadbalancer

service nginx restart
```

Perbarui juga file konfigurasi _sites-available_ pada masing masing worker (**AbimanyuWerbServer**, **PrabukusumaWebServer**, **WisanggerniWebServer**), agar me-listen ke port yang telah ditentukan. gunakan script berikut:

```sh
# Nginx sites-available configuration
echo '
 server {

    # [x] mengikuti digit terakhir pada port yang telah ditentukan
 	listen 800[x];

 	root /var/www/praktikum23;

 	index index.php index.html index.htm;
 	server_name _;

 	location / {
 			try_files $uri $uri/ /index.php?$query_string;
 	}

 	# pass PHP scripts to FastCGI server
 	location ~ \.php$ {
 	include snippets/fastcgi-php.conf;
 	fastcgi_pass unix:/var/run/php/php7.2-fpm.sock;
 	}

    location ~ /\.ht {
 			deny all;
 	}

 	error_log /var/log/nginx/praktikum23_error.log;
 	access_log /var/log/nginx/praktikum23_access.log;
 }
' > /etc/nginx/sites-available/praktikum23

service nginx restart
service php7.2-fpm stop
service php7.2-fpm start
```

### Hasil

Lalu untuk pengujiannya, buka **arjuna.e21.com** dengan perintah berikut pada node client, yaitu **NakulaClient** atau **SadewaClient**:

```
lynx arjuna.e21.com
```

ketika berulang-kali mengakses arjuna.e21.com, akan berjalan di web server yang berbeda-beda.
<img width="1580" alt="image" src="https://github.com/javakanaya/Jarkom-Modul-2-E21-2023/assets/87474722/b6ce704c-03b9-4b80-a1ef-90771b053471">
<img width="1580" alt="image" src="https://github.com/javakanaya/Jarkom-Modul-2-E21-2023/assets/87474722/221a5caf-ab6f-4270-a3ee-139802a95e0c">
<img width="1580" alt="image" src="https://github.com/javakanaya/Jarkom-Modul-2-E21-2023/assets/87474722/5e6d8ee1-e3c5-4b1b-8780-eefaf434eee9">

## Soal 11

> Selain menggunakan Nginx, lakukan konfigurasi Apache Web Server pada worker Abimanyu dengan web server **www.abimanyu.yyy.com**. Pertama dibutuhkan web server dengan DocumentRoot pada /var/www/abimanyu.yyy

### _Script_

Untuk konfigurasi server pertama kita perlu mendownload terlebih dahulu dan mengekstrak pada /var/www/abimanyu.e21

```bash
wget -O '/var/www/abimanyu.e21.com.zip' 'https://drive.usercontent.google.com/download?id=1a4V23hwK9S7hQEDEcv9FL14UkkrHc-Zc'

unzip -o /var/www/abimanyu.e21.com.zip -d /var/www/
mv /var/www/abimanyu.yyy.com /var/www/abimanyu.e21
rm /var/www/abimanyu.e21.com.zip
rm -rf /var/www/abimanyu.yyy.com
```

Lalu configure apache sites-available nya

```xml
<VirtualHost *:80>
        ServerName abimanyu.e21.com
        ServerAlias www.abimanyu.e21.com

        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/abimanyu.e21

        <Directory /var/www/abimanyu.e21>
            Options +FollowSymLinks -Multiviews
            AllowOverride All
        </Directory>

        ErrorLog \${APACHE_LOG_DIR}/error.log
        CustomLog \${APACHE_LOG_DIR}/access.log combined

</VirtualHost>
```

### Hasil

Folder /var/www/abimanyu.e21.com
![image](https://github.com/javakanaya/Jarkom-Modul-2-E21-2023/assets/27951856/11ac5993-cd6c-49ac-b29d-d81120ffe791)

Coba untuk membuka webnya dari salah satu client

- Ping
  ![image](https://github.com/javakanaya/Jarkom-Modul-2-E21-2023/assets/27951856/1b51de39-7f77-401e-b8a8-af3c61a49dbc)
- Lynx
  ![image](https://github.com/javakanaya/Jarkom-Modul-2-E21-2023/assets/27951856/07be5b67-44e9-4b67-964a-578d1050e2e2)

## Soal 12

> Setelah itu ubahlah agar url **www.abimanyu.yyy.com/index.php/home** menjadi **www.abimanyu.yyy.com/home**.

### _Script_

Ini dapat dilakukan dengan sedikit menambahkan code .htaccess nya

```bash
echo 'RewriteEngine On
RewriteCond %{REQUEST_FILENAME} !-d
RewriteCond %{REQUEST_FILENAME} !-f
RewriteRule ^(home)+$ index.php/$1 [NC,L]
' > /var/www/abimanyu.e21/.htaccess
```

### Hasil

![image](https://github.com/javakanaya/Jarkom-Modul-2-E21-2023/assets/27951856/465bf14d-8915-4e37-9043-7063ca9e1cba)
![image](https://github.com/javakanaya/Jarkom-Modul-2-E21-2023/assets/27951856/16838557-3d1c-4620-a58b-6f0a1d3caaca)

## Soal 13

> Selain itu, pada subdomain **www.parikesit.abimanyu.yyy.com**, DocumentRoot disimpan pada /var/www/parikesit.abimanyu.yyy

### _Script_

```bash
wget -O '/var/www/parikesit.abimanyu.e21.com.zip' 'https://drive.usercontent.google.com/download?id=1LdbYntiYVF_NVNgJis1GLCLPEGyIOreS'

unzip -o /var/www/parikesit.abimanyu.e21.com.zip -d /var/www/
mv /var/www/parikesit.abimanyu.yyy.com /var/www/parikesit.abimanyu.e21
rm /var/www/parikesit.abimanyu.e21.com.zip
rm -rf /var/www/parikesit.abimanyu.yyy.com
```

dan configuration dasar seperti berikut ini

```xml
<VirtualHost *:80>

        ServerName parikesit.abimanyu.e21.com
        ServerAlias www.parikesit.abimanyu.e21.com

        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/parikesit.abimanyu.e21

        ErrorLog \${APACHE_LOG_DIR}/error.log
        CustomLog \${APACHE_LOG_DIR}/access.log combined

</VirtualHost>
```

### Hasil

![image](https://github.com/javakanaya/Jarkom-Modul-2-E21-2023/assets/27951856/629c30a4-1d9c-4c50-be4d-61b424e1bb84)
![image](https://github.com/javakanaya/Jarkom-Modul-2-E21-2023/assets/27951856/c8fa36c9-cb8e-4f20-b90a-05aba276b994)
![image](https://github.com/javakanaya/Jarkom-Modul-2-E21-2023/assets/27951856/3db51cde-a819-46b6-9588-fee55acfd8bc)

## Soal 14

> Pada subdomain tersebut folder /public hanya dapat melakukan directory listing sedangkan pada folder /secret tidak dapat diakses (403 Forbidden).

### _Script_

tambahkan script ini pada configurasi virtual host sebelumnya pada apache sites-available

```xml
        <Directory /var/www/parikesit.abimanyu.e21/public>
            Options +Indexes
        </Directory>

        <Directory /var/www/parikesit.abimanyu.e21/secret>
            Options -Indexes
        </Directory>

        <Directory /var/www/parikesit.abimanyu.e21/public/js>
            Options +Indexes
        </Directory>

        <Directory /var/www/parikesit.abimanyu.e21/public/images>
            Options +FollowSymLinks +Indexes -Multiviews
            AllowOverride All
        </Directory>
```

### Hasil

Pada `(www.)parikesit.abimanyu.e21.com/public` dapat mengakses dan menampilkan directory listing
![image](https://github.com/javakanaya/Jarkom-Modul-2-E21-2023/assets/27951856/57f71b8c-ddbe-45d4-aa1e-dd9259a8f83c)'

Pada `(www.)parikesit.abimanyu.e21.com/secret` tidak dapat diakses dan mengeluarkan error 403: Forbidden
![image](https://github.com/javakanaya/Jarkom-Modul-2-E21-2023/assets/27951856/04dc9c18-d266-4c3e-9640-78cb779f9c55)
![image](https://github.com/javakanaya/Jarkom-Modul-2-E21-2023/assets/27951856/62da531b-d843-48f4-bad8-a2599bd51171)

## Soal 15

> Buatlah kustomisasi halaman error pada folder /error untuk mengganti error kode pada Apache. Error kode yang perlu diganti adalah 404 Not Found dan 403 Forbidden.

### _Script_

tembahkan lagi pada sites-available parikesit config

```xml
ErrorDocument 404 /error/404.html
ErrorDocument 403 /error/403.html

ErrorLog \${APACHE_LOG_DIR}/error.log
CustomLog \${APACHE_LOG_DIR}/access.log combined

```

### Hasil

- 403
  ![image](https://github.com/javakanaya/Jarkom-Modul-2-E21-2023/assets/27951856/ba0a3d29-3eeb-4350-a33a-f14dd617f067)

- 404
  ![image](https://github.com/javakanaya/Jarkom-Modul-2-E21-2023/assets/27951856/4f4db51a-32d1-4200-876a-bac873991ecf)

## Soal 16

> Buatlah suatu konfigurasi virtual host agar file asset **www.parikesit.abimanyu.yyy.com/public/js** menjadi **www.parikesit.abimanyu.yyy.com/js**.

### _Script_

pada configuration parikesit tambahkan lagi code ini

```
Alias "/js" "/var/www/parikesit.abimanyu.e21/public/js"
```

### Hasil

![image](https://github.com/javakanaya/Jarkom-Modul-2-E21-2023/assets/27951856/4d2211f2-7b40-4648-958c-5a0374a58865)
![image](https://github.com/javakanaya/Jarkom-Modul-2-E21-2023/assets/27951856/b3899031-45fc-4a24-a5a5-ca52807d50b8)

## Soal 17

> Agar aman, buatlah konfigurasi agar **www.rjp.baratayuda.abimanyu.yyy.com** hanya dapat diakses melalui port 14000 dan 14400.

### _Script_

Sebelumnya pada apache2 ports configuration file `/etc/apache2/ports.conf` tambahkan code berikut agar apache listening pada port 14000 dan 14400

```bash
Listen 14000
Listen 14400
```

Lalu pada `rjp.baratayuda.abimanyu.e21.com` configuration pada sites-available apache buat code seperti ini

```xml
<VirtualHost *:14000 *:14400>

        ServerName rjp.baratayuda.abimanyu.e21.com
        ServerAlias www.rjp.baratayuda.abimanyu.e21.com

        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/rjp.baratayuda.abimanyu.e21

        ErrorLog \${APACHE_LOG_DIR}/error.log
        CustomLog \${APACHE_LOG_DIR}/access.log combined

</VirtualHost>
```

### Hasil

![image](https://github.com/javakanaya/Jarkom-Modul-2-E21-2023/assets/27951856/9f211109-e642-42b3-aa37-d9c878d353eb)
![image](https://github.com/javakanaya/Jarkom-Modul-2-E21-2023/assets/27951856/2995ddbe-ef17-46e3-8a95-57a3bd6335aa)

## Soal 18

> Untuk mengaksesnya buatlah autentikasi username berupa “Wayang” dan password “baratayudayyy” dengan yyy merupakan kode kelompok. Letakkan DocumentRoot pada /var/www/rjp.baratayuda.abimanyu.yyy.

### _Script_

sebelumnya untuk menambahkan file pada /var/www/rjp.baratayuda.abimanyu.e21 dengan code

```bash
wget -O '/var/www/rjp.baratayuda.abimanyu.e21.com.zip' 'https://drive.usercontent.google.com/download?id=1pPSP7yIR05JhSFG67RVzgkb-VcW9vQO6'

unzip -o /var/www/rjp.baratayuda.abimanyu.e21.com.zip -d /var/www/
mv /var/www/rjp.baratayuda.abimanyu.yyy.com /var/www/rjp.baratayuda.abimanyu.e21
rm /var/www/rjp.baratayuda.abimanyu.e21.com.zip
rm -rf /var/www/rjp.baratayuda.abimanyu.yyy.com
```

Lalu untuk membuat authentication berupa username dan password kita menambahkan code

```bash
htpasswd -c -b /etc/apache2/.htpasswd Wayang baratayudae21
```

lalu menambahkan code ini untuk mengaplikasikan authentication pada virtualHost `rjp.baratayuda.abimanyu.e21.com`

```xml
<Location />
    AuthType Basic
    AuthName "Secret"
    AuthUserFile /etc/apache2/.htpasswd
    Require valid-user
</Location>
```

### Hasil

![image](https://github.com/javakanaya/Jarkom-Modul-2-E21-2023/assets/27951856/93c453bc-e676-4dde-bfb2-26eb9964ab13)
![image](https://github.com/javakanaya/Jarkom-Modul-2-E21-2023/assets/27951856/384b73ba-ca54-4682-932f-6e368edcb223)
![image](https://github.com/javakanaya/Jarkom-Modul-2-E21-2023/assets/27951856/09e03ba7-f9fb-4c9b-b9ed-8bc168c295a7)

## Soal 19

> Buatlah agar setiap kali mengakses IP dari Abimanyu akan secara otomatis dialihkan ke **www.abimanyu.yyy.com** (alias)

### _Script_

Kode ini akan mendirect ip dari abimanyu (10.47.1.4 pada topologi) ke web http://www.abimanyu.e21.com/

```bash
<VirtualHost *:80>
        ServerName 10.47.1.4
        Redirect / http://www.abimanyu.e21.com/
</VirtualHost>
```

### Hasil

![image](https://github.com/javakanaya/Jarkom-Modul-2-E21-2023/assets/27951856/7d7c0b60-2b01-481d-a456-28e6aa72f3c5)
![image](https://github.com/javakanaya/Jarkom-Modul-2-E21-2023/assets/27951856/494a934b-eb36-44e5-a160-e2ae8c6af6be)

## Soal 20

> Karena website **www.parikesit.abimanyu.yyy.com** semakin banyak pengunjung dan banyak gambar gambar random, maka ubahlah request gambar yang memiliki substring “abimanyu” akan diarahkan menuju abimanyu.png.

### _Script_

Kita perlu menambahkan htaccess pada `/var/www/parikesit.abimanyu.e21/public/images/.htaccess` dan menambahkan rewrite rule

```bash
echo '
RewriteEngine On
RewriteCond %{REQUEST_FILENAME} !-d
RewriteCond %{REQUEST_URI} abimanyu
RewriteCond %{REQUEST_URI} \.(jpg|png|jpeg)$ [NC]
RewriteRule .* /public/images/abimanyu.png [R,NC,L]
' > /var/www/parikesit.abimanyu.e21/public/images/.htaccess
```

### Hasil

## Keterangan Tambahan

- **yyy** pada url adalah **kode kelompok anda**
- File requirement dapat diakses melalui [drive berikut](https://drive.google.com/drive/folders/15Wr1eTQqn_vZzqkTXEAKF7tgULsxkxfe?usp=sharing).
