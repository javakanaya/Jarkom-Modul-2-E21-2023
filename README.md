# Jarkom-Modul-2-E21-2023

Laporan Resmi Praktikum Jaringan Komputer Modul 2

## Nama Anggota:

- [Yoel Mountanus Sitorus](https://github.com/zemetia) - 5025211078
- [Java Kanaya Prada](https://github.com/javakanaya) - 5025211112

## Topologi
<img width="1057" alt="Topologi" src="https://github.com/javakanaya/Jarkom-Modul-2-E21-2023/assets/87474722/c7b68ddc-95c8-4208-aa75-1edc92eae57f">

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
Pada node **Router**, jalankan perintah berikut, atau juga dapat menuliskan perintah berikut pada file ```.bashrc```, agar perintah tersebut tetap dijalankan saat kita membuka-menutup proyek.
```
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE -s 10.47.0.0/16
```
## Soal 1
> Yudhistira akan digunakan sebagai DNS Master, Werkudara sebagai DNS Slave, Arjuna merupakan Load Balancer yang terdiri dari beberapa Web Server yaitu Prabakusuma, Abimanyu, dan Wisanggeni. Buatlah topologi dengan pembagian [sebagai berikut](https://docs.google.com/spreadsheets/d/1OqwQblR_mXurPI4gEGqUe7v0LSr1yJViGVEzpMEm2e8/edit?usp=sharing). Folder topologi dapat diakses pada [drive berikut](https://drive.google.com/drive/folders/1Ij9J1HdIW4yyPEoDqU1kAwTn_iIxg3gk?usp=sharing)

### *Script*

Untuk menghubungkan semua node pada jaringan internet, kita perlu menjalankan perintah berikut pada masing-masing node. 
```sh
echo 'nameserver 192.168.122.1    # IP DNS' > /etc/resolv.conf
```
IP tersebut didapatkan dari file ```/etc/resolv.conf``` yang ada di node Router.

Untuk pengujian, kita bisa menjalankan perintah berikut. Disini kami melakukan pengujian pada node client, yaitu **NakulaClient**, dan **SadewaClient**.
```
ping google.com -c 5
```
### Hasil
<img width="1580" alt="Screenshot 2023-10-14 at 10 47 02" src="https://github.com/javakanaya/Jarkom-Modul-2-E21-2023/assets/87474722/d99a67bf-af8a-4154-94c2-93160b095994">
<img width="1580" alt="Screenshot 2023-10-14 at 10 47 07" src="https://github.com/javakanaya/Jarkom-Modul-2-E21-2023/assets/87474722/dd5b1649-cafa-454c-a931-f4c524c12688">


## Soal 2
> Buatlah website utama pada node arjuna dengan akses ke **arjuna.yyy.com** dengan alias **www.arjuna.yyy.com** dengan yyy merupakan kode kelompok.
### *Script*
Pada **YudhistiraDNSMaster** Pastikan pada file ```/etc/resolv.conf/``` sudah terdapat nameserver seperti pada soal 1. Kemudian jalankan script berikut. 
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
<img width="1580" alt="Screenshot 2023-10-14 at 11 36 53" src="https://github.com/javakanaya/Jarkom-Modul-2-E21-2023/assets/87474722/4f775e24-514d-4462-a2fa-cf4d7c7137e8">

<img width="1580" alt="Screenshot 2023-10-14 at 11 39 42" src="https://github.com/javakanaya/Jarkom-Modul-2-E21-2023/assets/87474722/9f1c6a67-eb06-4ecd-af60-19bf4f6acdf3">

## Soal 3
> Dengan cara yang sama seperti soal nomor 2, buatlah website utama dengan akses ke **abimanyu.yyy.com** dan alias **www.abimanyu.yyy.com**.
### *Script*
Melanjutkan pada soal nomor 2, maka kita perlu menambahkan konfigurasi untuk domain **abimanyu.e21.com** dengan script sebagai berikut.

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
```
### Hasil
Lalu untuk pengujiannya, sama seperti sebelumnya, jalankan perintah berikut pada node client, yaitu **NakulaClient** atau **SadewaClient**
```
ping abimanyu.e21.com -c 5
ping www.abimanyu.e21.com -c 5
```
<img width="1580" alt="image" src="https://github.com/javakanaya/Jarkom-Modul-2-E21-2023/assets/87474722/8214b0f5-ab0a-4fe4-829c-2b715a9ab96a">

## Soal 4
> Kemudian, karena terdapat beberapa web yang harus di-deploy, buatlah subdomain **parikesit.abimanyu.yyy.com** yang diatur DNS-nya di Yudhistira dan mengarah ke Abimanyu.
### *Script*
### Hasil

## Soal 5
> Buat juga reverse domain untuk domain utama. (Abimanyu saja yang direverse)
### *Script*
### Hasil

## Soal 6
> Agar dapat tetap dihubungi ketika DNS Server Yudhistira bermasalah, buat juga Werkudara sebagai DNS Slave untuk domain utama.
### *Script*
### Hasil

## Soal 7
> Seperti yang kita tahu karena banyak sekali informasi yang harus diterima, buatlah subdomain khusus untuk perang yaitu **baratayuda.abimanyu.yyy.com** dengan alias **www.baratayuda.abimanyu.yyy.com** yang didelegasikan dari Yudhistira ke Werkudara dengan IP menuju ke Abimanyu dalam folder Baratayuda.
### *Script*
### Hasil

## Soal 8
> Untuk informasi yang lebih spesifik mengenai Ranjapan Baratayuda, buatlah subdomain melalui Werkudara dengan akses **rjp.baratayuda.abimanyu.yyy.com** dengan alias **www.rjp.baratayuda.abimanyu.yyy.com** yang mengarah ke Abimanyu.
### *Script*
### Hasil

## Soal 9
> Arjuna merupakan suatu Load Balancer Nginx dengan tiga worker (yang juga menggunakan nginx sebagai webserver) yaitu Prabakusuma, Abimanyu, dan Wisanggeni. Lakukan deployment pada masing-masing worker.
### *Script*
### Hasil

## Soal 10
> Kemudian gunakan algoritma **Round Robin** untuk Load Balancer pada **Arjuna**. Gunakan server_name pada soal nomor 1. Untuk melakukan pengecekan akses alamat web tersebut kemudian pastikan worker yang digunakan untuk menangani permintaan akan berganti ganti secara acak. Untuk webserver di masing-masing worker wajib berjalan di port 8001-8003. 
Contoh:
>   - Prabakusuma:8001
>   - Abimanyu:8002
>   - Wisanggeni:8003
### *Script*
### Hasil

## Soal 11
> Selain menggunakan Nginx, lakukan konfigurasi Apache Web Server pada worker Abimanyu dengan web server **www.abimanyu.yyy.com**. Pertama dibutuhkan web server dengan DocumentRoot pada /var/www/abimanyu.yyy
### *Script*
### Hasil

## Soal 12
> Setelah itu ubahlah agar url **www.abimanyu.yyy.com/index.php/home** menjadi **www.abimanyu.yyy.com/home**.
### *Script*
### Hasil

## Soal 13
> Selain itu, pada subdomain **www.parikesit.abimanyu.yyy.com**, DocumentRoot disimpan pada /var/www/parikesit.abimanyu.yyy
### *Script*
### Hasil

## Soal 14
> Pada subdomain tersebut folder /public hanya dapat melakukan directory listing sedangkan pada folder /secret tidak dapat diakses (403 Forbidden).
### *Script*
### Hasil

## Soal 15
> Buatlah kustomisasi halaman error pada folder /error untuk mengganti error kode pada Apache. Error kode yang perlu diganti adalah 404 Not Found dan 403 Forbidden. 
### *Script*
### Hasil

## Soal 16
> Buatlah suatu konfigurasi virtual host agar file asset **www.parikesit.abimanyu.yyy.com/public/js** menjadi **www.parikesit.abimanyu.yyy.com/js**. 
### *Script*
### Hasil

## Soal 17
> Agar aman, buatlah konfigurasi agar **www.rjp.baratayuda.abimanyu.yyy.com** hanya dapat diakses melalui port 14000 dan 14400.
### *Script*
### Hasil

## Soal 18
> Untuk mengaksesnya buatlah autentikasi username berupa “Wayang” dan password “baratayudayyy” dengan yyy merupakan kode kelompok. Letakkan DocumentRoot pada /var/www/rjp.baratayuda.abimanyu.yyy.
### *Script*
### Hasil

## Soal 19
> Buatlah agar setiap kali mengakses IP dari Abimanyu akan secara otomatis dialihkan ke **www.abimanyu.yyy.com** (alias)
### *Script*
### Hasil

## Soal 20
> Karena website **www.parikesit.abimanyu.yyy.com** semakin banyak pengunjung dan banyak gambar gambar random, maka ubahlah request gambar yang memiliki substring “abimanyu” akan diarahkan menuju abimanyu.png.
### *Script*
### Hasil

## Keterangan Tambahan
- **yyy** pada url adalah **kode kelompok anda**
- File requirement dapat diakses melalui [drive berikut](https://drive.google.com/drive/folders/15Wr1eTQqn_vZzqkTXEAKF7tgULsxkxfe?usp=sharing).
