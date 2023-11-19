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


# Soal 18

## Jawaban
Dalam soal 18, DNS riegel.canyon diubah, diarahkan ke node Eisen dengan port 83 (80-82 dipakai worker PHP), untuk memudahkan testing menggunakan salah satu endpoint backend. Adapun konfigurasinya adalah sebagai berikut:

```
upstream backend {
        server 10.73.4.1:8001;
        server 10.73.4.2:8002;
        server 10.73.4.3:8003;
}

server {
        listen 83;

        server_name riegel.canyon.it19.com www.riegel.canyon.it19.com;

        location / {
                proxy_pass http://backend;
        }
}
```
Setelahnya, restart service nginx dengan ```service nginx restart```.
Untuk memastikan apakah konfigurasi berhasil, dapat dilakukan testing sederhana, dalam case ini menggunakan ```ab -n 300 -c 10 http://riegel.canyon.it19.com:83/```

#### Hasil testing, POV Load Balancer

![image](https://github.com/alweismiau/Jarkom-Modul-3-IT19-2023/assets/112788819/d095a86c-d4ec-4931-ace3-86bc8229eae1)


Log akses pada ```Frieren```

![image](https://github.com/alweismiau/Jarkom-Modul-3-IT19-2023/assets/112788819/0e681ee4-46ae-4d13-b1be-59d15b7ddf85)


Log akses pada ```Fern```

![image](https://github.com/alweismiau/Jarkom-Modul-3-IT19-2023/assets/112788819/0415e320-bbfe-4214-9ad8-56000b6ecd60)


Log akses pada ```Flamme```

![image](https://github.com/alweismiau/Jarkom-Modul-3-IT19-2023/assets/112788819/d62bd49f-5a95-4895-9d35-f4900cbc6c94)


Gambar grafik htop dan log akses di atas menunjukkan bahwa terdapat requests yang masuk, menandakan konfigurasi ter-atur dengan benar.

# Soal 19
> Untuk meningkatkan performa dari Worker, coba implementasikan PHP-FPM pada Frieren, Flamme, dan Fern. Untuk testing kinerja naikkan 
> - pm.max_children
> - pm.start_servers
> - pm.min_spare_servers
> - pm.max_spare_servers
> sebanyak tiga percobaan dan lakukan testing sebanyak 100 request dengan 10 request/second kemudian berikan hasil analisisnya pada Grimoire.

## Jawaban
Dalam soal 19, diharuskan untuk mengubah konfigurasi php yang terletak di ```/etc/php/8.0/fpm/pool./www.conf```. Untuk pengubahan config disediakan 4 bash script sebagai berikut:

#### Bash script konfigurasi pm awal
```
echo '[www]
user = www-data
group = www-data
listen = /run/php/php8.0-fpm.sock
listen.owner = www-data
listen.group = www-data
php_admin_value[disable_functions] = exec,passthru,shell_exec,system
php_admin_flag[allow_url_fopen] = off

; Choose how the process manager will control the number of child processes.

pm = dynamic
pm.max_children = 5
pm.start_servers = 2
pm.min_spare_servers = 1
pm.max_spare_servers = 3' > /etc/php/8.0/fpm/pool.d/www.conf
```

#### Hasil testing

![Screenshot (873)](https://github.com/alweismiau/Jarkom-Modul-3-IT19-2023/assets/112788819/db75d5aa-b223-45ab-80f8-5e5033cbf637)

```
Server Software:        nginx/1.14.2
Server Hostname:        riegel.canyon.it19.com
Server Port:            83

Document Path:          /api/auth/login
Document Length:        31 bytes

Concurrency Level:      10
Time taken for tests:   11.153 seconds
Complete requests:      100
Failed requests:        0
Non-2xx responses:      100
Total transferred:      32300 bytes
Total body sent:        22100
HTML transferred:       3100 bytes
Requests per second:    8.97 [#/sec] (mean)
Time per request:       1115.345 [ms] (mean)
Time per request:       111.535 [ms] (mean, across all concurrent requests)
Transfer rate:          2.83 [Kbytes/sec] received
                        1.94 kb/s sent
                        4.76 kb/s total

Connection Times (ms)
              min  mean[+/-sd] median   max
Connect:        0    2   5.0      0      38
Processing:   152  802 862.1    557    3415
Waiting:      152  801 860.8    557    3415
Total:        153  805 863.1    558    3417

Percentage of the requests served within a certain time (ms)
  50%    558
  66%    651
  75%    722
  80%    819
  90%   2903
  95%   3323
  98%   3409
  99%   3417
 100%   3417 (longest request)
Diatas merupakan test load-balancing http://riegel.canyon.it19.com:83/api/auth/login dengan konfig pm init
```


#### Bash script konfigurasi pm 2
```
echo '[www]
user = www-data
group = www-data
listen = /run/php/php8.0-fpm.sock
listen.owner = www-data
listen.group = www-data
php_admin_value[disable_functions] = exec,passthru,shell_exec,system
php_admin_flag[allow_url_fopen] = off

; Choose how the process manager will control the number of child processes.

pm = dynamic
pm.max_children = 25
pm.start_servers = 5
pm.min_spare_servers = 3
pm.max_spare_servers = 10' > /etc/php/8.0/fpm/pool.d/www.conf
```

#### Hasil testing

![Screenshot (875)](https://github.com/alweismiau/Jarkom-Modul-3-IT19-2023/assets/112788819/70c0a721-a675-41a4-b47b-3e2385f10bf5)

```
Server Software:        nginx/1.14.2
Server Hostname:        riegel.canyon.it19.com
Server Port:            83

Document Path:          /api/auth/login
Document Length:        31 bytes

Concurrency Level:      10
Time taken for tests:   9.278 seconds
Complete requests:      100
Failed requests:        0
Non-2xx responses:      100
Total transferred:      32300 bytes
Total body sent:        22100
HTML transferred:       3100 bytes
Requests per second:    10.78 [#/sec] (mean)
Time per request:       927.847 [ms] (mean)
Time per request:       92.785 [ms] (mean, across all concurrent requests)
Transfer rate:          3.40 [Kbytes/sec] received
                        2.33 kb/s sent
                        5.73 kb/s total

Connection Times (ms)
              min  mean[+/-sd] median   max
Connect:        0    2   3.9      0      20
Processing:   165  851 866.0    528    3764
Waiting:      165  850 866.1    527    3763
Total:        168  853 866.0    528    3764

Percentage of the requests served within a certain time (ms)
  50%    528
  66%    717
  75%    843
  80%   1001
  90%   2773
  95%   3315
  98%   3440
  99%   3764
 100%   3764 (longest request)
Diatas merupakan test load-balancing http://riegel.canyon.it19.com:83/api/auth/login dengan konfig pm 2
```

#### Bash script konfigurasi pm 3
```
echo '[www]
user = www-data
group = www-data
listen = /run/php/php8.0-fpm.sock
listen.owner = www-data
listen.group = www-data
php_admin_value[disable_functions] = exec,passthru,shell_exec,system
php_admin_flag[allow_url_fopen] = off

; Choose how the process manager will control the number of child processes.

pm = dynamic
pm.max_children = 50
pm.start_servers = 8
pm.min_spare_servers = 5
pm.max_spare_servers = 15' > /etc/php/8.0/fpm/pool.d/www.conf
```

#### Hasil testing

![Screenshot (877)](https://github.com/alweismiau/Jarkom-Modul-3-IT19-2023/assets/112788819/e67659f1-e10a-4fce-a63f-768d106d7f10)

```
Server Software:        nginx/1.14.2
Server Hostname:        riegel.canyon.it19.com
Server Port:            83

Document Path:          /api/auth/login
Document Length:        31 bytes

Concurrency Level:      10
Time taken for tests:   10.097 seconds
Complete requests:      100
Failed requests:        0
Non-2xx responses:      100
Total transferred:      32300 bytes
Total body sent:        22100
HTML transferred:       3100 bytes
Requests per second:    9.90 [#/sec] (mean)
Time per request:       1009.747 [ms] (mean)
Time per request:       100.975 [ms] (mean, across all concurrent requests)
Transfer rate:          3.12 [Kbytes/sec] received
                        2.14 kb/s sent
                        5.26 kb/s total

Connection Times (ms)
              min  mean[+/-sd] median   max
Connect:        0    2   4.2      1      25
Processing:   145  919 891.6    621    3814
Waiting:      143  918 891.7    620    3814
Total:        146  921 891.8    622    3817

Percentage of the requests served within a certain time (ms)
  50%    622
  66%    871
  75%    966
  80%   1118
  90%   2843
  95%   3440
  98%   3706
  99%   3817
 100%   3817 (longest request)
Diatas merupakan test load-balancing http://riegel.canyon.it19.com:83/api/auth/login dengan konfig pm 3
```

#### Bash script konfigurasi pm 4
```
echo '[www]
user = www-data
group = www-data
listen = /run/php/php8.0-fpm.sock
listen.owner = www-data
listen.group = www-data
php_admin_value[disable_functions] = exec,passthru,shell_exec,system
php_admin_flag[allow_url_fopen] = off

; Choose how the process manager will control the number of child processes.

pm = dynamic
pm.max_children = 75
pm.start_servers = 10
pm.min_spare_servers = 5
pm.max_spare_servers = 20' > /etc/php/8.0/fpm/pool.d/www.conf
```

#### Hasil testing

![Screenshot (879)](https://github.com/alweismiau/Jarkom-Modul-3-IT19-2023/assets/112788819/333c694f-fa4c-45c3-929e-78cf86b823f3)

```
Server Software:        nginx/1.14.2
Server Hostname:        riegel.canyon.it19.com
Server Port:            83

Document Path:          /api/auth/login
Document Length:        31 bytes

Concurrency Level:      10
Time taken for tests:   9.900 seconds
Complete requests:      100
Failed requests:        0
Non-2xx responses:      100
Total transferred:      32300 bytes
Total body sent:        22100
HTML transferred:       3100 bytes
Requests per second:    10.10 [#/sec] (mean)
Time per request:       989.992 [ms] (mean)
Time per request:       98.999 [ms] (mean, across all concurrent requests)
Transfer rate:          3.19 [Kbytes/sec] received
                        2.18 kb/s sent
                        5.37 kb/s total

Connection Times (ms)
              min  mean[+/-sd] median   max
Connect:        0    5   9.5      1      45
Processing:   152  908 829.2    621    3677
Waiting:      152  907 829.4    620    3677
Total:        184  913 829.6    625    3679

Percentage of the requests served within a certain time (ms)
  50%    625
  66%    819
  75%    940
  80%   1034
  90%   2529
  95%   3395
  98%   3580
  99%   3679
 100%   3679 (longest request)
Diatas merupakan test load-balancing http://riegel.canyon.it19.com:83/api/auth/login dengan konfig pm 4
```


# Soal 20
> Nampaknya hanya menggunakan PHP-FPM tidak cukup untuk meningkatkan performa dari worker maka implementasikan Least-Conn pada Eisen. Untuk testing kinerja dari worker tersebut dilakukan sebanyak 100 request dengan 10 request/second

Untuk menambahkan algoritma Least Conn, maka perlu ditambahkan ```least_conn;``` pada file config nginx untuk riegel.canyon.it19.com, seperti berikut:

![Screenshot 2023-11-19 220718](https://github.com/alweismiau/Jarkom-Modul-3-IT19-2023/assets/112788819/6b69234e-62ec-4d20-856f-86733a7a7c54)

Setelahnya, dilakukan testing dengan menggunakan command:
```ab -n 100 -c 10 -p login.json -T application/json http://www.riegel.canyon.it19.com/api/auth/login```

Gambaran htop saat melakukan testing:
