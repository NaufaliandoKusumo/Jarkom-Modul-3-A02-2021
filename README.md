# Jarkom-Modul-3-A02-2021

## Nama Kelompok A02:
- Naufaliando Kusuma -
- Kevin Davi Samuel - 05111940000157
- Muhamad Affan - 05111840000109

#### 1. Luffy bersama Zoro berencana membuat peta tersebut dengan kriteria EniesLobby sebagai DNS Server, Jipangu sebagai DHCP Server, Water7 sebagai Proxy Server
- Membuat topologi seperti berikut:
  ![img1](https://i.postimg.cc/Cx3Bw5ZT/image.png)<br/>
- Lalu setting network masing-masing node dengan settingan:<br/>
  - Foosha (sebagai Router / DHCP Relay)
  ```
  auto eth0
  iface eth0 inet dhcp
  auto eth1
  iface eth1 inet static
    address 10.0.1.1
    netmask 255.255.255.0
  auto eth2
  iface eth2 inet static
    address 10.0.2.1
    netmask 255.255.255.0
  auto eth3
  iface eth3 inet static
    address 10.0.3.1
    netmask 255.255.255.0
  ```

  - Loguetown (sebagai Client)
  ```
  auto eth0
  iface eth0 inet static
    address 10.0.1.2
    netmask 255.255.255.0
    gateway 10.0.1.1
  ```

  - Alabasta (sebagai Client)
  ```
  auto eth0
  iface eth0 inet static
    address 10.0.1.3
    netmask 255.255.255.0
    gateway 10.0.1.1
  ```

  - EniesLobby (sebagai DNS Master)
  ```
  auto eth0
  iface eth0 inet static
    address 10.0.2.2
    netmask 255.255.255.0
    gateway 10.0.2.1
  ```

  - Water7 (sebagai Proxy Server)
  ```
  auto eth0
  iface eth0 inet static
    address 10.0.2.3
    netmask 255.255.255.0
    gateway 10.0.2.1
  ```

  - Jipangu (sebagai DHCP Server)
  ```
  auto eth0
  iface eth0 inet static
    address 10.0.2.4
    netmask 255.255.255.0
    gateway 10.0.2.1
  ```

  - Skypie (sebagai Client)
  ```
  auto eth0
  iface eth0 inet static
    address 10.0.3.2
    netmask 255.255.255.0
    gateway 10.0.3.1
  ```

  - TottoLand (sebagai Client)
  ```
  auto eth0
  iface eth0 inet static
    address 10.0.3.3
    netmask 255.255.255.0
    gateway 10.0.3.1
  ```
- Pada EniesLobby, lakukan `apt-get update` kemudian `apt-get install bind9`
- Pada Jipangu, lakukan `apt-get update` kemudian `apt-get install isc-dhcp-server`
- Pada Water7, lakukan `apt-get update` kemudian `apt-get install squid`

#### 2. Foosha sebagai DHCP Relay
- Pada Foosha, lakukan `apt-get update` kemudian `apt-get install isc-dhcp-relay`
- Edit file `/etc/default/isc-dhcp-relay` seperti berikut:
  ![img2](https://i.postimg.cc/GpschxWq/image.png)<br/>

#### 3. Client yang melalui Switch1 mendapatkan range IP dari [prefix IP].1.20 - [prefix IP].1.99 dan [prefix IP].1.150 - [prefix IP].1.169
- Pada Jipangu edit file `/etc/dhcp/dhcpd.conf` menjadi:
  ```
  subnet 10.0.2.0 netmask 255.255.255.0 {
        option routers 10.0.2.1;
  }

  subnet 10.0.1.0 netmask 255.255.255.0 {
        range 10.0.1.20 10.0.1.99;
        range 10.0.1.150 10.0.1.169;
        option routers 10.0.1.1;
        option broadcast-address 10.0.1.255;
        option domain-name-servers 10.0.2.2;
        default-lease-time 360;
        max-lease-time 7200;
  }
  ```

#### 4. Client yang melalui Switch3 mendapatkan range IP dari [prefix IP].3.30 - [prefix IP].3.50 
- Pada Jipangu edit file `/etc/dhcp/dhcpd.conf` seperti berikut:
  ```
  subnet 10.0.2.0 netmask 255.255.255.0 {
        option routers 10.0.2.1;
  }

  subnet 10.0.1.0 netmask 255.255.255.0 {
        range 10.0.1.20 10.0.1.99;
        range 10.0.1.150 10.0.1.169;
        option routers 10.0.1.1;
        option broadcast-address 10.0.1.255;
        option domain-name-servers 10.0.2.2;
        default-lease-time 360;
        max-lease-time 7200;
  }
  
  subnet 10.0.3.0 netmask 255.255.255.0 {
        range 10.0.3.30 10.0.3.50;
        option routers 10.0.3.1;
        option broadcast-address 10.0.3.255;
        option domain-name-servers 10.0.2.2;
        default-lease-time 720;
        max-lease-time 7200;
  }
  ```
- Lalu restart dengan `service isc-dhcp-server restart`

#### 5. Client mendapatkan DNS dari EniesLobby dan client dapat terhubung dengan internet melalui DNS tersebut.

#### 6. Lama waktu DHCP server meminjamkan alamat IP kepada Client yang melalui Switch1 selama 6 menit sedangkan pada client yang melalui Switch3 selama 12 menit. Dengan waktu maksimal yang dialokasikan untuk peminjaman alamat IP selama 120 menit.
Pada Jipangu edit file konfigurasi `/etc/dhcp/dhcpd.conf`:
- Edit `default-lease-time` menjadi `360`
- Edit `max-lease-time` menjadi `7200`

#### 7. Luffy dan Zoro berencana menjadikan Skypie sebagai server untuk jual beli kapal yang dimilikinya dengan alamat IP yang tetap dengan IP [prefix IP].3.69

#### 8. Pada Loguetown, proxy harus bisa diakses dengan nama jualbelikapal.yyy.com dengan port yang digunakan adalah 5000
- Lakukan pengaturan pada `/etc/squid/squid.conf` seperti berikut:
```
include /etc/squid/acl.conf
http_port 5000
visible_hostname jualbelikapal.a02.com
auth_param basic program /usr/lib/squid/basic_ncsa_auth /etc/squid/passwd
auth_param basic children 5
auth_param basic realm Proxy
auth_param basic credentialsttl 2 hours
auth_param basic casesensitive on
acl USERS proxy_auth REQUIRED
http_access allow AVAILABLE_WORKING1 USERS
http_access allow AVAILABLE_WORKING2 USERS
http_access allow AVAILABLE_WORKING3 USERS
http_access deny all
```

#### 9. Agar transaksi jual beli lebih aman dan pengguna website ada dua orang, proxy dipasang autentikasi user proxy dengan enkripsi MD5 dengan dua username, yaitu luffybelikapalyyy dengan password luffy_yyy dan zorobelikapalyyy dengan password zoro_yyy 
- Untuk menambahkan user dan pass bisa menggunakan command
```
htpasswd -b -c /etc/squid/passwd lufffybelikapala02 luffy_a02
htpasswd -b /etc/squid/passwd zorobelikapala02 zoro_a02
```

#### 10. Transaksi jual beli tidak dilakukan setiap hari, oleh karena itu akses internet dibatasi hanya dapat diakses setiap hari Senin-Kamis pukul 07.00-11.00 dan setiap hari Selasa-Jum’at pukul 17.00-03.00 keesokan harinya (sampai Sabtu pukul 03.00)
- Pada `/etc/squid/acl.conf` diisi dengan:
```
acl AVAILABLE_WORKING time MTWH 07:00-11:00
acl AVAILABLE_WORKING time TWHF 17:00-24:00
acl AVAILABLE_WORKING time TWHF 00:00-03:00
```

