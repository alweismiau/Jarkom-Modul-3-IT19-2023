# Jarkom-Modul-3-IT19-2023
## Kelompok IT19
- Fina Keiza Arismana       5027211028
- Rifqi Akhmad Maulana      502721035

# Topologi
![](https://cdn.discordapp.com/attachments/1025213238763327683/1174711396739596419/image.png?ex=6568963e&is=6556213e&hm=a788286db02cbc357a8e58ca9a1d88e6816419946ddc8ba44bce6a7799d39068&)

# Konfigurasi
- Aura (Router - DHCP Relay)
```
auto eth0
iface eth0 inet dhcp

auto eth1
iface eth1 inet static
	address 10.73.1.1
	netmask 255.255.255.0

auto eth2
iface eth2 inet static
	address 10.73.2.1
	netmask 255.255.255.0

auto eth3
iface eth3 inet static
	address 10.73.3.11
	netmask 255.255.255.0

auto eth4
iface eth4 inet static
	address 10.73.4.11
	netmask 255.255.255.0

```
- Himmel (DHCP Server)
```
auto eth0
iface eth0 inet static
	address 10.73.1.2
	netmask 255.255.255.0
	gateway 10.73.1.1

```
- Heiter (DNS Server)
```
auto eth0
iface eth0 inet static
	address 10.73.1.3
	netmask 255.255.255.0
	gateway 10.73.1.1
```
- Denken (Database Server)
```
auto eth0
iface eth0 inet static
	address 10.73.2.2
	netmask 255.255.255.0
	gateway 10.73.2.1
```
- Eisen (Load Balancer)
```
auto eth0
iface eth0 inet static
	address 10.73.2.3
	netmask 255.255.255.0
	gateway 10.73.2.1
```
- Frieren (Laravel Worker)
```
auto eth0
iface eth0 inet static
	address 10.73.4.1
	netmask 255.255.255.0
	gateway 10.73.4.11
```
	Untuk Flamme dan Fern diubah address belakangnya menjadi 2 dan 3
- Lawine (PHP Worker)
```
auto eth0
iface eth0 inet static
	address 10.73.3.1
	netmask 255.255.255.0
	gateway 10.73.3.11
```
	Untuk Linie dan Lugner diubah address belakangnya menjadi 2 dan 3
- Revolte, Richter, Sein, Stark (Client)
```
auto eth0
iface eth0 inet dhcp
```

# Setup
- DHCP Relay
```
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE -s 10.73.0.0/16

apt-get update
apt-get install mc -y
apt-get install isc-dhcp-relay-y
service isc-dhcp-relay restart
```
- DHCP Server
```
echo nameserver 192.168.122.1 > /etc/resolv.conf
apt-get update
apt-get install isc-dhcp-server -y
```
- DNS Server 
```
echo 'nameserver 192.168.122.1' > /etc/resolv.conf
apt-get update
apt-get install bind9 -y  
```
- Database Server 
```
echo nameserver 192.168.122.1 > /etc/resolv.conf
apt-get update
apt-get install mariadb-server -y
service mysql start
```
- Load Balancer 
```
echo nameserver 192.168.122.1 > /etc/resolv.conf
apt-get update
apt-get install apache2-utils -y
apt-get install nginx -y
apt-get install lynx -y
service nginx start
```
- Worker Laravel 
```
echo 'nameserver 10.73.1.3
nameserver 192.168.122.1' > /etc/resolv.conf

apt-get update
apt-get install lynx -y
apt-get install mariadb-client -y
```
- Worker PHP 
```

```
- Client 
```
apt update
apt install lynx -y
apt install apache2-utils -y
apt-get install jq -y
```

# Soal 0

## Jawaban
