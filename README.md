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
@       IN      A       192.208.2.4 ;IP EniesLobby
```
