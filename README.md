# Jarkom-Modul-3-E08-2021

Lapres Praktikum Jaringan Komputer Modul 3
-  (05111940000005)
- Zahra Dyah Meilani (05111940000069)
-  (05111940000)

## **Soal 1**

Luffy yang sudah menjadi Raja Bajak Laut ingin mengembangkan daerah kekuasaannya dengan membuat peta seperti berikut:

![topologi](images/topologi.jpeg)

Luffy bersama Zoro berencana membuat peta tersebut dengan kriteria EniesLobby sebagai DNS Server, Jipangu sebagai DHCP Server, Water7 sebagai Proxy Server.

### **Pembahasan**

Pertama, kita buat terlebih dahulu topologi sesuai gambar pada soal. Kemudian edit network configuration di setiap node menjadi sebagai berikut:

- Foosha

```
auto eth0
iface eth0 inet dhcp

auto eth1
iface eth1 inet static
	address 10.33.1.1
	netmask 255.255.255.0

auto eth2
iface eth2 inet static
	address 10.33.2.1
	netmask 255.255.255.0

auto eth3
iface eth3 inet static
	address 10.33.3.1
	netmask 255.255.255.0
```

- Jipangu

```
auto eth0
iface eth0 inet static
	address 10.33.2.4
	netmask 255.255.255.0
	gateway 10.33.2.1
```

- EniesLobby
  
```
auto eth0
iface eth0 inet static
	address 10.33.2.2
	netmask 255.255.255.0
	gateway 10.33.2.1
```

- Water7

```
auto eth0
iface eth0 inet static
	address 10.33.2.3
	netmask 255.255.255.0
	gateway 10.33.2.1
```

- Loguetown

```
auto eth0
iface eth0 inet static
	address 10.33.1.2
	netmask 255.255.255.0
	gateway 10.33.1.1
```

- Alabasta

```
auto eth0
iface eth0 inet static
	address 10.33.1.3
	netmask 255.255.255.0
	gateway 10.33.1.1
```

- Skypie

```
auto eth0
iface eth0 inet static
	address 10.33.3.2
	netmask 255.255.255.0
	gateway 10.33.3.1
```

- Tottoland

```
auto eth0
iface eth0 inet static
	address 10.33.3.3
	netmask 255.255.255.0
	gateway 10.33.3.1
```

Setelah itu, kita buat node supaya dapat tersambung ke internet dengan menjalankan perintah di bawah pada Foosha:

`iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE -s 10.33.0.0/16`

Kemudian pada node lainnya:

`echo nameserver 192.168.122.1 > /etc/resolv.conf`

## **Soal 2**

..., dan Foosha sebagai DHCP Relay.

### **Pembahasan**

Pertama, install DHCP relay pada Foosha:

`apt-get update`

`apt-get install isc-dhcp-relay -y`

Edit isi `/etc/default/isc-dhcp-relay` dengan menggunakan nano menjadi seperti berikut:

![2a](images/2a.png)

ubah nilai SERVERS dengan IP Jipangu (DHCP server) yaitu "10.33.2.4" supaya terhubung dengan DHCP server dan INTERFACES nya menjadi "eth1 eth2 eth3" supaya terhubung dengan switch1, switch2, dan switch3.

Selanjutnya, pada Jipangu yang merupakan DHCP Server kita lakukan beberapa hal:

Pertama, install DHCP server:

```
apt-get update
apt-get install isc-dhcp-server -y
```

Edit isi `/etc/default/isc-dhcp-server` dengan menggunakan nano menjadi seperti berikut:

![2b](images/2b.png)

ubah nilai INTERFACES nya menjadi "eth0" karena pada Jipangu yang digunakan adalah eth0.

Edit isi `/etc/sysctl.conf` dengan menggunakan nano menjadi seperti berikut:

![2c](images/2c.png)

uncomment `net.ipv4.ip_forward=1` supaya dapat 

Pada EniesLobby, install bind9: 

```
apt-get update
apt-get install bind9 -y
```

Dan pada Water7, install squid dan apache:

```
apt-get update
apt-get install squid -y
apt-get install apache2-utils -y
```
Start DHCP server pada Jipangu dan DHCP relay pada Foosha:

```service isc-dhcp-server restart```

```service isc-dhcp-relay restart```

Untuk mengecek status:

```service isc-dhcp-server status```

## **Soal 3**

Ada beberapa kriteria yang ingin dibuat oleh Luffy dan Zoro, yaitu:

### **Pembahasan**
  
1. Semua client yang ada HARUS menggunakan konfigurasi IP dari DHCP Server.

    Agar semua client menggunakan konfigurasi IP dari DHCP Server, maka edit network configuration menjadi sebagai berikut:

    ```
    auto eth0
    iface eth0 inet dhcp
    ```

2. Client yang melalui Switch1 mendapatkan range IP dari [prefix IP].1.20 - [prefix IP].1.99 dan [prefix IP].1.150 - [prefix IP].1.169.
   
   Pada node Jipangu buka file `/etc/dhcp/dhcpd.conf`, dan tambahkan:

   ```
   subnet 10.33.2.0 netmask 255.255.255.0 {
    option routers 10.33.2.1;
    }
   ```
   supaya DHCP relay (Foosha) dapat terhubung dengan subnet 10.33.2.0.

   Kemudian tambahkan pula:

   ```
   subnet 10.33.1.0 netmask 255.255.255.0 {
    range 10.33.1.20 10.33.1.99;
    range 10.33.1.150 10.33.1.169;
    option routers 10.33.1.1;
    option broadcast-address 10.33.1.255;
    option domain-name-servers 10.33.2.2;
    default-lease-time 360;
    max-lease-time 7200;
    }
    ```

    ![3a](images/3a.png)

    Periksa pada client node switch 3 dengan `ip a`,

    - Loguetown

      ![3b](image/3b.png)

    - Alabasta

      ![3c](image/3c.png)

    Dari hasil tersebut, nilai IP kedua node telah sesuai dengan range di soal.

## **Soal 4**

Client yang melalui Switch3 mendapatkan range IP dari [prefix IP].3.30 - [prefix IP].3.50.

### **Pembahasan**

Pada node Jipangu buka file `/etc/dhcp/dhcpd.conf`, dan tambahkan:

```
subnet 10.33.3.0 netmask 255.255.255.0 {
  range 10.33.3.30 10.33.3.50;
  option subnet-mask 255.255.255.0;
  option broadcast-address 10.33.3.255;
  option domain-name-servers 10.33.2.2;
  option routers 10.33.3.1;
  default-lease-time 720;
  max-lease-time 7200;
}
```

![4a](images/4a.png)

Periksa pada client node switch 3 dengan `ip a`,

- Skypie

![4b](image/4b.png)

- Tottoland

![4c](image/4c.png)

Dari hasil tersebut, nilai IP kedua node telah sesuai dengan range di soal.

## **Soal 5**

Client mendapatkan DNS dari EniesLobby dan client dapat terhubung dengan internet melalui DNS tersebut.

### **Pembahasan**

Pada Jipangu, tambahkan  `option domain-name-servers 10.33.2.2;` di konfigurasi dhcpd.conf.

Pada EniesLobby, buka file `/etc/bind/named.conf.options`, tambahkan:

```
forwarders {
  192.168.122.1;
};
```

Kemudian comment pada bagian:

`dnssec-validation auto;`

Dan tambahkan:

`allow-query{any;};`

![5a](image/5a.png)

Lalu, restart bind9:

`service bind9 restart`

Tes koneksi internet di setiap node

- Loguetown

    ![5b](image/5b.png)

- Alabasta

    ![5c](image/5c.png)

- Skypie

    ![5d](image/5d.png)

- Tottoland

    ![5e](image/5e.png)


## **Soal 6**
Lama waktu DHCP server meminjamkan alamat IP kepada Client yang melalui Switch1 selama 6 menit sedangkan pada client yang melalui Switch3 selama 12 menit. Dengan waktu maksimal yang dialokasikan untuk peminjaman alamat IP selama 120 menit.

### **Pembahasan**

Pada subnet switch pertama, digunakan default lease time 360 dan max lease time 7200.
```
default-lease-time 360;
max-lease-time 7200;
```

Pada subnet switch ketiga, digunakan default lease time 720 dan max lease time 7200.

```
default-lease-time 720;
max-lease-time 7200;
```


## **Soal 7**

Luffy dan Zoro berencana menjadikan Skypie sebagai server untuk jual beli kapal yang dimilikinya dengan alamat IP yang tetap dengan IP [prefix IP].3.69

### **Pembahasan**

Agar node Skypie memiliki alamat IP tetap, maka digunakan fixed address.

Pertama, periksa ethernet pada Skypie dengan command `ip a`, ambil nilai link/ether:

![7a](image/7a.png)

Pada node Jipangu buka file `/etc/dhcp/dhcpd.conf`, dan tambahkan:

```
host Skypie {
    hardware ethernet da:92:b0:a5:84:6c;
    fixed-address 10.33.3.69;
}
```

hardware ethernet merupakan nilai link/ether dari Skypie dan fixed-address merupakan IP yang diminta, yaitu 10.33.3.69.

Edit network configuration pada Skypie:

```
auto eth0
iface eth0 inet dhcp
hwaddress ether da:92:b0:a5:84:6c
```

![7b](images/7b.png)

Restart DHCP server pada Jipangu:

```service isc-dhcp-server restart```

Hasil `ip a` di node Skypie:

![7c](image/7c.png)












