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
# Soal 0 & 1 & 4
Praktikan diminta untuk melakukan register domain riegel.canyon.yyy.com untuk worker Laravel dan granz.channel.yyy.com untuk worker PHP. Selain itu semua `CLIENT` harus menggunakan Konfigurasi dari DHCP Server. Dan Client mendapatkan DNS dari Heiter dan dapat terhubung dengan internet melalui DNS tersebut.
## Jawaban
Siapkan topologi terlebih dahulu dan lakukan setup, selanjutnya lakukan setup pada DNS Server (Heiter) dengan script berikut : 
```
mkdir /etc/bind/jarkom

echo '
zone "riegel.canyon.it19.com" {
        type master;
        file "/etc/bind/jarkom/riegel.canyon.it19.com";
};

zone "granz.channel.it19.com" {
        type master;
        file "/etc/bind/jarkom/granz.channel.it19.com";
};
' > /etc/bind/named.conf.local


echo ';
; BIND data file for local loopback interface
;
$TTL    604800
@       IN      SOA     riegel.canyon.it19.com. root.riegel.canyon.it19.com. (
			            2023110101    ; Serial
                        604800        ; Refresh
                        86400         ; Retry
                        2419200       ; Expire
                        604800 )      ; Negative Cache TTL
;
@               IN      NS      riegel.canyon.it19.com.
@               IN      A       10.73.4.2 ; IP Frieren
www             IN      CNAME   riegel.canyon.it19.com.
@               IN      AAAA    ::1' > /etc/bind/jarkom/riegel.canyon.it19.com

echo '
;
; BIND data file for local loopback interface
;
$TTL    604800
@       IN      SOA     granz.channel.it19.com. root.granz.channel.it19.com. (
			    2023110101    ; Serial
                        604800        ; Refresh
                        86400         ; Retry
                        2419200       ; Expire
                        604800 )      ; Negative Cache TTL
;
@               IN      NS      granz.channel.it19.com.
@               IN      A       10.73.3.2 ; IP Lawine
www             IN      CNAME   granz.channel.it19.com.
@               IN      AAAA    ::1
' > /etc/bind/jarkom/granz.channel.it19.com

echo 'options {
        directory "/var/cache/bind";

        forwarders {
                192.168.122.1;
        };

        // dnssec-validation auto;
        allow-query{any;};
       
        listen-on-v6 { any; };
};' > /etc/bind/named.conf.options

service bind9 restart
```
Kemudian lakukan command `ping google.com` pada client untuk mengecek apakah sudah terhubung dengan internet, dan lakukan `ping riegel.canyon.it19.com` dan `ping granz.channel.it19.com` untuk mengecek apakah sudah berhasil atau tidak. 
![](https://media.discordapp.net/attachments/1025213238763327683/1176008266107596880/no_1-0.png?ex=656d4e0b&is=655ad90b&hm=49f4e2043c0c201b24a7738dbdde18f1589ffb7f2c6fc16230c918bc86483a95&=&width=1525&height=864)

# Soal 2 & 3 & 5
Client yang melalui Switch3 mendapatkan range IP dari 10.73.3.16 - 10.73.3.32 dan 10.73.3.64 - 10.73.3.80. Dan lakukan pada Switch4 untuk mendapatkan range IP dari 10.73.4.12 - 10.73.4.20 dan 10.73.4.160 - 10.73.4.168. Lama waktu DHCP server meminjamkan alamat IP kepada Client yang melalui Switch3 selama 3 menit sedangkan pada client yang melalui Switch4 selama 12 menit. Dengan waktu maksimal dialokasikan untuk peminjaman alamat IP selama 96 menit.
## Jawaban
Setelah dilakukan setup sebelumnya maka setup DHCP server dengan script sebagai berikut : 
```
echo 'INTERFACES="eth0"' > /etc/default/isc-dhcp-server

echo '
option domain-name "example.org";
option domain-name-servers nsl.example.org, ns2.example.org; 

default-lease-time 600;
max-lease-time 7200;

ddns-update-style none; 

# Switch3
subnet 10.73.1.0 netmask 255.255.255.0 {} 

subnet 10.73.3.0 netmask 255.255.255.0 {
    range 10.73.3.16 10.73.3.32;
    range 10.73.3.64 10.73.3.80;
    option routers 10.73.3.1;
    option broadcast-address 10.73.3.255;
    option domain-name-servers 10.73.1.3;
    default-lease-time 180;
    max-lease-time 5760;
}

# Switch4 
subnet 10.73.4.0 netmask 255.255.255.0 {
    range 10.73.4.12 10.73.4.20;
    range 10.73.4.160 10.73.4.168;
    option routers 10.73.4.1;
    option broadcast-address 10.73.4.255;
    option domain-name-servers 10.73.1.3;
    default-lease-time 720;
    max-lease-time 5760;
}' >  /etc/dhcp/dhcpd.conf

service isc-dhcp-server restart

# Command
service isc-dhcp-server restart → service isc-dhcp-server status
```
![](https://cdn.discordapp.com/attachments/1025213238763327683/1176011577590288444/image.png?ex=656d5121&is=655adc21&hm=d8d6b55c97cefe9cff36a7422e6fea1e1a4335488e61ff2c2aa051648d9e8d23&)
![](https://cdn.discordapp.com/attachments/1025213238763327683/1176011612751134907/image.png?ex=656d5129&is=655adc29&hm=1ff1ac1e1df18931d993020cf733955cd4b63fd4e4f31178773bd48784a22d4b&)

# Soal 6
> Pada masing-masing worker PHP, lakukan konfigurasi virtual host untuk website berikut dengan menggunakan php 7.3

## Jawaban
Sebelum menjalankan nomor 6-12 diharuskan untuk melakukan [setup](#setup) terlebih dahulu, terutama untuk bagian worker PHP.

Jika sudah, buat file konfigurasi untuk website dengan diberi nama ```granz.channel.it19.com``` di ```/etc/nginx/sites-available/```, seperti di bawah.

```
server {
    listen 80;
    server_name _;

    root /var/www/granz.channel.it19.com;
    index index.php index.html index.htm;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/run/php/php7.3-fpm.sock;  # Sesuaikan versi PHP dan socket
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }
}
```

Jangan lupa untuk ```service nginx restart``` dan menambahkan symbolic-link ke ```/etc/nginx/sites-enabled/```.

Jika dicoba untuk akses salah satu worker dengan ```lynx 10.73.3.1```, maka akan tampil seperti berikut:

![image](https://github.com/alweismiau/Jarkom-Modul-3-IT19-2023/assets/112788819/3ce3a17d-702c-4e08-8935-f22bca95b920)


# Soal 7
> Kepala suku dari Bredt Region memberikan resource server sebagai berikut -> Lawine (4GB, 2vCPU, dan 80 GB SSD), Linie (2GB, 2vCPU, dan 50 GB SSD), Lugner (1GB, 1vCPU, dan 25 GB SSD). Aturlah agar Eisen dapat bekerja dengan maksimal, lalu lakukan testing dengan 1000. request dan 100 request/second.

## Jawaban

Berganti ke Eisen, buat file ```/etc/nginx/sites-available/granz.channel.it19.com```. Setelahnya dibuat symbolic link ke ```/etc/nginx/sites-enabled/```. Berikut adalah isi file konfigurasi.

```
upstream workers {
        server 10.73.3.1;
        server 10.73.3.2;
        server 10.73.3.3;
}

server {
        listen 80;
        listen 81;
        listen 82;

        server_name granz.channel.it19.com www.granz.channel.it19.com;

        location / {
                proxy_pass http://workers;
        }
}
```

Untuk men-simulasikan spek yang ditentukan dalam soal. Dapat digunakan ```weight``` untuk menentukan skala beban kerja yang diterima salah satu atau lebih worker. Contohnya seperti di bawah:

```
upstream workers {
        server 10.73.3.1 weight=1;
        server 10.73.3.2 weight=3;
        server 10.73.3.3 weight=4;
}

server {
        listen 80;
        listen 81;
        listen 82;

        server_name granz.channel.it19.com www.granz.channel.it19.com;

        location / {
                proxy_pass http://workers;
        }
}
```

Karena Lawine memiliki spek teratas, maka untuk men-simulasikannya perlu diberikan weight yang lebih kecil untuk melihat seolah-olah Lawine memiliki sumberdaya yang lebih banyak.

Hasil testing 1000 request dan 100 request concurrent:

![image](https://github.com/alweismiau/Jarkom-Modul-3-IT19-2023/assets/112788819/99012a06-c53f-415c-aab4-220aca076bc2)


# Soal 8
> Karena diminta untuk menuliskan grimoire, buatlah analisis hasil testing dengan 200 request dan 10 request/second masing-masing algoritma Load Balancer dengan ketentuan sebagai berikut -> Nama Algoritma Load Balancer, Report hasil testing pada Apache Benchmark, Grafik request per second untuk masing masing algoritma, dan analisis.

## Jawaban
Untuk melakukan testing dengan beberapa algoritma yang berbeda, perlu dilakukan pengubahan konfigurasi beberapa kali pada node Eisen sebagai load balancer.

Berikut adalah cuplikan dari konfigurasi tambahan yang perlu ditambahkan pada file ```/etc/nginx/sites-available/granz.channel.it19.com```.

### Setup Least Connection
```
upstream workers {
	least_conn;
        server 10.73.3.1;
        server 10.73.3.2;
        server 10.73.3.3;
}
...
...
```

### Setup IP Hash
```
upstream workers {
	ip_hash;
        server 10.73.3.1;
        server 10.73.3.2;
        server 10.73.3.3;
}
...
...
```

### Setup Generic Hash
```
upstream workers {
	hash $host$request_uri;
        server 10.73.3.1;
        server 10.73.3.2;
        server 10.73.3.3;
}
...
...
```

Untuk setiap perubahan skrip, diharuskan untuk menjalankan ```service nginx restart```

Berikut adalah hasil testing dari tiap algoritma load-balancing dan analisis dari keseluruhan testing:

### Round Robin

![rr](https://github.com/alweismiau/Jarkom-Modul-3-IT19-2023/assets/112788819/9ca4dec1-cb32-4364-bc02-e9560fd5dbd9)


### Least Connection

![lc](https://github.com/alweismiau/Jarkom-Modul-3-IT19-2023/assets/112788819/ac9d6e8f-9de4-4e10-941d-e663df48b19d)


### IP Hash

![iphash](https://github.com/alweismiau/Jarkom-Modul-3-IT19-2023/assets/112788819/2f753324-7f61-47c1-860a-8cbbae8e672e)


### Generic Hash

![ghash](https://github.com/alweismiau/Jarkom-Modul-3-IT19-2023/assets/112788819/9b3359e0-7349-4280-8314-42d7bac821fb)


### Grafik Request/s

![image](https://github.com/alweismiau/Jarkom-Modul-3-IT19-2023/assets/112788819/de294e7f-6fe8-4592-ab41-dfe70c061b00)

### Analisa
Untuk hasil benchmarking, dapat dilihat bahwa Algoritma Round robin dan Least Connection memiliki performa yang cukup bagus, tetapi diungguli oleh algoritma Least Connection.
Karena dalam least connection, mengutamakan melimpahkan request ke worker yang memiliki beban paling sedikit/memiliki koneksi/request paling sedikit, dibandingkan dengan Round Robin yang membagi request secara merata.


# Soal 9
> Dengan menggunakan algoritma Round Robin, lakukan testing dengan menggunakan 3 worker, 2 worker, dan 1 worker sebanyak 100 request dengan 10 request/second, kemudian tambahkan grafiknya pada grimoire

## Jawaban

Untuk mengurangi worker yang bekerja dibawah load balancer Eisen, cukup comment row yang memiliki IP yang ingin dimatikan service nya. Disiapkan 3 konfigurasi berbeda, diikuti dengan hasil testing masing-masing pengaturan, seperti berikut:

### Setup Round Robin 3 Worker
```
upstream workers {
        server 10.73.3.1;
        server 10.73.3.2;
        server 10.73.3.3;
}
...
...
```

Hasil testing

```
Requests per second:    804.71 [#/sec] (mean)
Time per request:       12.427 [ms] (mean)
Time per request:       1.243 [ms] (mean, across all concurrent requests)
Transfer rate:          598.56 [Kbytes/sec] received
```

![image](https://github.com/alweismiau/Jarkom-Modul-3-IT19-2023/assets/112788819/d9a7b27a-9ed4-4afa-bafc-eabee7481c93)


### Setup Round Robin 2 Worker
```
upstream workers {
        server 10.73.3.1;
        server 10.73.3.2;
#        server 10.73.3.3;
}
...
...
```

Hasil testing

```
Requests per second:    878.92 [#/sec] (mean)
Time per request:       11.378 [ms] (mean)
Time per request:       1.138 [ms] (mean, across all concurrent requests)
Transfer rate:          653.61 [Kbytes/sec] received
```

![image](https://github.com/alweismiau/Jarkom-Modul-3-IT19-2023/assets/112788819/ed5dbfbb-b0dd-4f86-9d31-8f83bb5c3ed4)


### Setup Round Robin 1 Worker
```
upstream workers {
        server 10.73.3.1;
#        server 10.73.3.2;
#        server 10.73.3.3;
}
...
...
```

Hasil testing

```
Requests per second:    945.68 [#/sec] (mean)
Time per request:       10.574 [ms] (mean)
Time per request:       1.057 [ms] (mean, across all concurrent requests)
Transfer rate:          703.72 [Kbytes/sec] received
```

![image](https://github.com/alweismiau/Jarkom-Modul-3-IT19-2023/assets/112788819/c80d7572-0e8e-4bd5-8482-e11788a733e5)


Dari hasil percobaan, semakin sedikit worker yang bekerja, semakin banyak angka request per detik. Hal ini dikarenakan ketika salah satu worker mati, maka beban pekerjaan worker tersebut akan dilimpahkan ke worker lain yang sedang aktif sehingga angka request per detik menjadi lebih banyak dan berbanding terbalik dengan jumlah worker yang aktif meng-handle request.

# Soal 10 
> Selanjutnya coba tambahkan konfigurasi autentikasi di LB dengan dengan kombinasi username: “netics” dan password: “ajkyyy”, dengan yyy merupakan kode kelompok. Terakhir simpan file “htpasswd” nya di /etc/nginx/rahasisakita/
## Jawaban

Untuk menambahkan autentikasi di Load Balancer Eisen, maka perlu bantuan htpasswd yang ada dalam package apache2-utils. Maka dari itu perlu untuk install package tersebut terlebih dahulu menggunakan command ```apt-get install apache2-utils```.

Yang dapat dilakukan yaitu membuat file config autentikasi baru dengan command ```mkdir /etc/nginx/rahasiakita/ && htpasswd -c /etc/nginx/rahasiakita/.htpasswd netics```. Setelah dijalankan, di direktori yang berkaitan akan ada sebuah file .htpasswd dan berisi sbb:


Untuk dapat menggunakan file tersebut sebagai pengaturan auth Load Balancer, perlu dilakukan konfigurasi pada ```granz.channel.it19.com```, menjadi:

```
upstream workers {
        server 10.73.3.1;
        server 10.73.3.2;
        server 10.73.3.3;
}

server {
        listen 80;
        listen 81;
        listen 82;

        server_name granz.channel.it19.com www.granz.channel.it19.com;

        location / {
                proxy_pass http://workers;
                auth_basic "Restricted Access";
                auth_basic_user_file /etc/nginx/rahasiakita/.htpasswd;
        }
}
```

Save lalu restart service nginx. Hasilnya adalah sbb:

![image](https://github.com/alweismiau/Jarkom-Modul-3-IT19-2023/assets/112788819/b4d0b25a-2aa9-44d2-8f58-7aba0b2bf6f6)
![image](https://github.com/alweismiau/Jarkom-Modul-3-IT19-2023/assets/112788819/c80f705d-c9f0-4383-8356-72754e56d8c0)

Jika akses tertolak akan diberi opsi untuk mengulang proses autentikasi seperti dibawah:

![image](https://github.com/alweismiau/Jarkom-Modul-3-IT19-2023/assets/112788819/9903cdf5-933d-4675-9a6a-1fa9d7bd08c7)


# Soal 11
> Lalu buat untuk setiap request yang mengandung /its akan di proxy passing menuju halaman https://www.its.ac.id
## Jawaban

Untuk menyelesaikan soal nomor 11, diperlukan untuk menambah konfigurasi tambahan di file config nginx ```granz.channel.it19.com```, dengan menambahkan blok ```location ~ /its```, menjadi seperti berikut:

```
upstream workers {
        server 10.73.3.1;
        server 10.73.3.2;
        server 10.73.3.3;
}

server {
        listen 80;
        listen 81;
        listen 82;

        server_name granz.channel.it19.com www.granz.channel.it19.com;

        location ~ /its {
                satisfy any;
                allow all;

                proxy_pass https://www.its.ac.id;
                proxy_set_header Host www.its.ac.id;
                proxy_set_header X-Real-IP $remote_addr;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_set_header X-Forwarded-Proto $scheme;
        }

        location / {
                proxy_pass http://workers;
                auth_basic "Restricted Access";
                auth_basic_user_file /etc/nginx/rahasiakita/.htpasswd;
        }

}
```

Selanjutnya, jika kita memasukkan request dengan akhiran atau selipan /its, maka akan menampilkan page sederhana dari ```www.its.ac.id``` seperti berikut:

![image](https://github.com/alweismiau/Jarkom-Modul-3-IT19-2023/assets/112788819/b60225e4-e5b3-4283-8937-ea6aaed1e968)


# Soal 12
> Selanjutnya LB ini hanya boleh diakses oleh client dengan IP 10.73.3.69, 10.73.3.70, 10.73.4.167, dan 10.73.4.168.

## Jawaban

Untuk menyelesaikan soal nomor 12, perlu untuk kembali ke node DHCP Server Himmel untuk melakukan konfigurasi tambahan di ```/etc/dhcp/dhcpd.conf```:

```
option domain-name "example.org";
option domain-name-servers nsl.example.org, ns2.example.org;

default-lease-time 600;
max-lease-time 7200;

ddns-update-style none;

# Switch3
subnet 10.73.1.0 netmask 255.255.255.0 {}

subnet 10.73.3.0 netmask 255.255.255.0 {
    range 10.73.3.16 10.73.3.32;
    range 10.73.3.64 10.73.3.80;
    option routers 10.73.3.11;
    option broadcast-address 10.73.3.255;
    option domain-name-servers 10.73.1.3;
    default-lease-time 180;
    max-lease-time 5760;
}

# Switch4
subnet 10.73.4.0 netmask 255.255.255.0 {
    range 10.73.4.12 10.73.4.20;
    range 10.73.4.160 10.73.4.168;
    option routers 10.73.4.11;
    option broadcast-address 10.73.4.255;
    option domain-name-servers 10.73.1.3;
    default-lease-time 720;
    max-lease-time 5760;
}

host Sein {
    hardware ethernet de:d3:2d:64:0c:c1;
    fixed-address 10.73.4.167;
}

host Stark {
    hardware ethernet 82:07:ae:96:5e:ce;
    fixed-address 10.73.4.168;
}

host Revolte {
    hardware ethernet 3e:97:65:c8:a7:a8;
    fixed-address 10.73.3.69;
}

host Richter {
    hardware ethernet 96:89:4b:76:78:31;
    fixed-address 10.73.3.70;
}
```

Lalu beralih ke Load Balancer Eisen lagi untuk menambahkan beberapa line ke dalam file config ```granz.channel.it19.com```:

```
...
	location / {
                proxy_pass http://workers;

                allow 10.73.3.69;
                allow 10.73.3.70;
                allow 10.73.4.167;
                allow 10.73.4.168;
                deny all;

                auth_basic "Restricted Access";
                auth_basic_user_file /etc/nginx/rahasiakita/.htpasswd;
        }
...
```

Konfigurasi di atas hanya memperbolehkan akses yang spesifik dari 4 IP yang tertera, dan menolak selain akses dari 4 IP tersebut.

Jika diterima akan masuk ke dalam landing page ```granz.channel.it19.com```, jika tertolak akan mendapat respon 403 Forbidden, seperti berikut:

Jika akses diberikan maka akan keluar prompt username dan password seperti pada nomor 10 dan masuk ke halaman awal granz.channel.it19.com:

![image](https://github.com/alweismiau/Jarkom-Modul-3-IT19-2023/assets/112788819/b4d0b25a-2aa9-44d2-8f58-7aba0b2bf6f6)
![image](https://github.com/alweismiau/Jarkom-Modul-3-IT19-2023/assets/112788819/c80f705d-c9f0-4383-8356-72754e56d8c0)

![image](https://github.com/alweismiau/Jarkom-Modul-3-IT19-2023/assets/112788819/3ce3a17d-702c-4e08-8935-f22bca95b920)

Jika akses tertolak:

![image](https://github.com/alweismiau/Jarkom-Modul-3-IT19-2023/assets/112788819/e8dabe93-1afc-48c0-a3db-e98b831cb559)


# Soal 13
Kemudian praktikan disuruh untuk mengatur riegel.canyon.yyy.com. Semua data yang diperlukan, diatur pada Denken dan harus dapat diakses oleh Frieren, Flamme, dan Fern.
## Jawaban
Setelah melakukan setup, dibuat database dbkelompokit19 pada `Database Server` yaitu `Denken` dengan script berikut :
```
echo '[client-server]
!includedir /etc/mysql/conf.d/
!includedir /etc/mysql/mariadb.conf.d/

[mysqld]
skip-networking=0
skip-bind-address' > /etc/mysql/my.cnf

service mysql start
```
Kemudian lakukan command `mysql` untuk membuka mysqlnya, kemudian lakukan script sebagai berikut : 
```
mysql <<EOF
CREATE USER 'kelompokit19'@'%' IDENTIFIED BY 'passwordit19';
CREATE USER 'kelompokit19'@'localhost' IDENTIFIED BY 'passwordit19';
CREATE DATABASE dbkelompokit19;
GRANT ALL PRIVILEGES ON *.* TO 'kelompokit19'@'%';
GRANT ALL PRIVILEGES ON *.* TO 'kelompokit19'@'localhost';
FLUSH PRIVILEGES;
quit
EOF
```
Kemudian check terlebih dahulu pada `Denken` dengan command :
```
mysql -u kelompokit19 -p <<EOF
SHOW DATABASES;
quit
EOF
```
yang nantinya akan dimasukan password yang sudah dibuat sebelumnya. 
![](https://media.discordapp.net/attachments/1025213238763327683/1176008267743375420/no13.png?ex=656d4e0c&is=655ad90c&hm=2cd627cf826fe647760e17fb44b0ad915f6480ddafb3dd23bfa7b31383538ffb&=&width=1585&height=787)

Kemudian check di salah satu worker laravel, contohnya `Frieren` dengan command : 
```
mariadb --host=10.73.2.2 --port=3306 --user=kelompokit19 --password <<EOF
SHOW DATABASES;
quit
EOF
```
![](https://cdn.discordapp.com/attachments/1025213238763327683/1176015247237201983/image.png?ex=656d548c&is=655adf8c&hm=16ae579a3969a383ccb4ab13ad29a23343b50b556875bd9b3a0884a1aa5c7589&)

# Soal 14
Frieren, Flamme, dan Fern memiliki Riegel Channel sesuai dengan quest guide. Jangan lupa melakukan instalasi PHP8.0 dan Composer.
## Jawaban
Setelah melakukan setup, lakukan install composer dan git dengan resource yang sudah diberikan. Kemudian dilakukan clone pada resource yang diberikan pada masing-masing worker. Berikut adalah scriptnya : 
```
apt-get install -y lsb-release ca-certificates apt-transport-https software-properties-common gnupg2
curl -sSLo /usr/share/keyrings/deb.sury.org-php.gpg https://packages.sury.org/php/apt.gpg
sh -c 'echo "deb [signed-by=/usr/share/keyrings/deb.sury.org-php.gpg] https://packages.sury.org/php/ $(lsb_release -sc) main" > /etc/apt/sources.list.d/php.list'
apt-get update
apt-get install php8.0-mbstring php8.0-xml php8.0-cli php8.0-common php8.0-intl php8.0-opcache php8.0-readline php8.0-mysql php8.0-fpm php8.0-curl unzip wget -y
apt-get install nginx -y
apt-get install lynx -y

wget https://getcomposer.org/download/2.0.13/composer.phar
chmod +x composer.phar
mv composer.phar /usr/bin/composer

apt-get install git -y
cd /var/www && git clone https://github.com/martuafernando/laravel-praktikum-jarkom

cd /var/www/laravel-praktikum-jarkom && composer update
cd /var/www/laravel-praktikum-jarkom && cp /var/www/laravel-praktikum-jarkom/.env.example /var/www/laravel-praktikum-jarkom/.env

echo 'APP_NAME=Laravel
APP_ENV=local
APP_KEY=
APP_DEBUG=true
APP_URL=http://localhost

LOG_CHANNEL=stack
LOG_DEPRECATIONS_CHANNEL=null
LOG_LEVEL=debug

DB_CONNECTION=mysql
DB_HOST=10.73.2.2 #IP Danken
DB_PORT=3306
DB_DATABASE=dbkelompokit19
DB_USERNAME=kelompokit19
DB_PASSWORD=passwordit19

BROADCAST_DRIVER=log
CACHE_DRIVER=file
FILESYSTEM_DISK=local
QUEUE_CONNECTION=sync
SESSION_DRIVER=file
SESSION_LIFETIME=120

MEMCACHED_HOST=127.0.0.1

REDIS_HOST=127.0.0.1
REDIS_PASSWORD=null
REDIS_PORT=6379

MAIL_MAILER=smtp
MAIL_HOST=mailpit
MAIL_PORT=1025
MAIL_USERNAME=null
MAIL_PASSWORD=null
MAIL_ENCRYPTION=null
MAIL_FROM_ADDRESS="hello@example.com"
MAIL_FROM_NAME="${APP_NAME}"

AWS_ACCESS_KEY_ID=
AWS_SECRET_ACCESS_KEY=
AWS_DEFAULT_REGION=us-east-1
AWS_BUCKET=
AWS_USE_PATH_STYLE_ENDPOINT=false

PUSHER_APP_ID=
PUSHER_APP_KEY=
PUSHER_APP_SECRET=
PUSHER_HOST=
PUSHER_PORT=443
PUSHER_SCHEME=https
PUSHER_APP_CLUSTER=mt1

VITE_PUSHER_APP_KEY="${PUSHER_APP_KEY}"
VITE_PUSHER_HOST="${PUSHER_HOST}"
VITE_PUSHER_PORT="${PUSHER_PORT}"
VITE_PUSHER_SCHEME="${PUSHER_SCHEME}"
VITE_PUSHER_APP_CLUSTER="${PUSHER_APP_CLUSTER}"' > /var/www/laravel-praktikum-jarkom/.env

cd /var/www/laravel-praktikum-jarkom && php artisan key:generate
cd /var/www/laravel-praktikum-jarkom && php artisan config:cache
cd /var/www/laravel-praktikum-jarkom && php artisan migrate
cd /var/www/laravel-praktikum-jarkom && php artisan db:seed
cd /var/www/laravel-praktikum-jarkom && php artisan storage:link
cd /var/www/laravel-praktikum-jarkom && php artisan jwt:secret
cd /var/www/laravel-praktikum-jarkom && php artisan config:clear
```
Kemudian setup ip setiap worker sebagai berikut : 
```
10.73.4.1:8001; # Frieren 
10.73.4.2:8002; # Flamme
10.73.4.3:8003; # Fern
```
Kemudian lakukan script pada `nginx` sebagai berikut : 
```
echo 'server {
    listen [IP Worker];

    root /var/www/laravel-praktikum-jarkom/public;

    index index.php index.html index.htm;
    server_name _;

    location / {
            try_files $uri $uri/ /index.php?$query_string;
    }

    # pass PHP scripts to FastCGI server
    location ~ \.php$ {
      include snippets/fastcgi-php.conf;
      fastcgi_pass unix:/var/run/php/php8.0-fpm.sock;
    }

    location ~ /\.ht {
            deny all;
    }

    error_log /var/log/nginx/implementasi_error.log;
    access_log /var/log/nginx/implementasi_access.log;
}' > /etc/nginx/sites-available/laravel-worker


ln -s /etc/nginx/sites-available/laravel-worker /etc/nginx/sites-enabled/
chown -R www-data.www-data /var/www/laravel-praktikum-jarkom/storage
service php8.0-fpm start
service nginx restart
```
Kemudian lakukan command `lynx localhost:8001 ` berikut untuk mengecek apakah sudah berhasil atau tidak.
![](https://cdn.discordapp.com/attachments/1025213238763327683/1176020325549494363/Screenshot_2023-11-19_065412.png?ex=656d5947&is=655ae447&hm=50d2b4adc7b0ae69aba4e06b7793741ce498a7e1c52cf03f62f5a76fb76727b9&)

# Soal 15 & 16 & 17
Riegel Channel memiliki beberapa endpoint yang harus ditesting sebanyak 100 request dengan 10 request/second. Tambahkan response dan hasil testing pada grimoire. `POST /auth/register`, `POST /auth/login`, `GET /me`.
## Jawaban
Setelah dilakukan setup, lakukan script pada `.json` untuk `POST /auth/register`, `POST /auth/login`, `GET /me` pada setiap client. 
```
echo '
{
  "username": "kelompokit19",
  "password": "passwordit19"
}' > register.json
echo '
{
  "username": "kelompokit19",
  "password": "passwordit19"
}' > login.json
```
Kemudian untuk mengecek register dan login, lakukan command sebagai berikut : 
```
ab -n 100 -c 10 -p register.json -T application/json http://10.73.4.1:8001/api/auth/register

ab -n 100 -c 10 -p login.json -T application/json http://10.73.4.1:8001/api/auth/login
```
Kemudian untuk mengecek me, maka lakukan command sebagai berikut : 
```
url -X POST -H "Content-Type: application/json" -d @login.json http://10.73.4.1:8001/api/auth/login > login_it19.txt
token=$(cat login_it19.txt | jq -r '.token')
ab -n 100 -c 10 -H "Authorization: Bearer $token" http://10.73.4.1:8001/api/me
```
![](https://cdn.discordapp.com/attachments/1025213238763327683/1176022245978689656/Screenshot_2023-11-16_212230.png?ex=656d5b10&is=655ae610&hm=46605914c39e6351cd6b1b1c61bf126e4f528b6d2ab49e3824ca29cbf4c27681&)
![](https://media.discordapp.net/attachments/1025213238763327683/1176022246549094450/Screenshot_2023-11-16_212422.png?ex=656d5b11&is=655ae611&hm=c95c765b0259c783312aedfae2a7ef90b50956d9aca1ecfadaca19229508e837&=&width=931&height=864)
![](https://media.discordapp.net/attachments/1025213238763327683/1176022246255509575/Screenshot_2023-11-16_212456.png?ex=656d5b10&is=655ae610&hm=4e9e132b8525b7a36fce408d47ad8a3fe5a115722b0c8f8ab3effff6a0178bc5&=&width=1114&height=864)

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

![Screenshot (873)](https://github.com/alweismiau/Jarkom-Modul-3-IT19-2023/assets/112788819/db75d5aa-b223-45ab-80f8-5e5033cbf637)



## Analisis 19 dan 20
Dari hasil beberapa iterasi testing yang telah dilakukan, perbedaan visual dari ketiganya hampir nihil. Terdapat asumsi bahwa hal tersebut karena kapasitas spesifikasi mesin induk kurang memadai untuk menjadi "server" sehingga saat dilakukan testing, hampir setiap kali akan menunjukkan CPU Usage di angka 100%. Solusi yang dapat dilakukan adalah dengan menggunakan mesin yang memiliki spesifikasi lebih update, sehingga perbedaan dan dampak performa yang dilakukan oleh tiap worker dapat diobservasi lebih lanjut.
> Dalam kasus testing efisiensi pembagian beban kerja dengan algoritma dan spek yang telah ditentukan pada nomor 19 dan 20 seperti yang terlihat pada grafik ```htop``` tiap worker, penulis berasumsi bahwa hal yang membuat perubahan-perubahan variabel dan algoritma tidak terlihat signifikan dikarenakan keterbatasan spesifikasi perangkat praktikan. Sehingga dalam testing, hampir di tiap iterasi, terlihat lonjakan penggunaan CPU di masing-masing worker yang mencapai 100%.
