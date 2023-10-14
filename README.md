# Jarkom-Modul-2-E21-2023

Laporan Resmi Praktikum Jaringan Komputer Modul 2

## Nama Anggota:

- [Yoel Mountanus Sitorus](https://github.com/zemetia) - 5025211078
- [Java Kanaya Prada](https://github.com/javakanaya) - 5025211112

## Topologi

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
echo '
nameserver 192.168.122.1    # IP DNS' > /etc/resolv.conf
```
IP tersebut didapatkan dari file ```/etc/resolv.conf``` yang ada di node Router.

Untuk pengujian, kita bisa menjalankan perintah berikut. Disini kami melakukan pengujian pada node client, yaitu **NakulaClient**, dan **SadewaClient**.
```
ping google.com -c 5
```
### Hasil
<img width="1580" alt="Screenshot 2023-10-14 at 10 47 02" src="https://github.com/javakanaya/Jarkom-Modul-2-E21-2023/assets/87474722/d99a67bf-af8a-4154-94c2-93160b095994">
<img width="1580" alt="Screenshot 2023-10-14 at 10 47 07" src="https://github.com/javakanaya/Jarkom-Modul-2-E21-2023/assets/87474722/dd5b1649-cafa-454c-a931-f4c524c12688">


