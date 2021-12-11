# Jarkom-Modul-5-C11-2021

## Anggota Kelompok C11 : <br>
- 05111940000001 Christopher Baptista
- 05111940000093 Riki Mi’roj Achmad
- 05111940000219 M Rizqi Fiqih Thalib

### A. Tugas pertama kalian yaitu membuat topologi jaringan sesuai dengan rancangan yang diberikan Luffy dibawah ini

- Jipangu adalah DHCP Server
- Foosha, Water7, dan Guanhao adalah DHCP Relay
- Doriki adalah DNS Server
- Blueno, Cipher, Elena, dan Fukurou adalah client

![topo](https://user-images.githubusercontent.com/62735317/145663435-a56eab01-12df-47ee-935c-afb3d85bbd11.png)

### B. Karena kalian telah belajar subnetting dan routing, Luffy ingin meminta kalian untuk membuat topologi tersebut menggunakan teknik CIDR atau VLSM.
Teknik yang kami gunakan dalam subnetting dan routing adalah CIDR dengan perhitungan sebagai berikut.
![image](https://user-images.githubusercontent.com/74702068/145667980-99fa5d59-698a-4b61-b77d-8e3f3f46f3dc.png)
![image](https://user-images.githubusercontent.com/74702068/145667995-839d2867-385c-4147-8fe2-791ce876e672.png)

### C. Kalian juga diharuskan melakukan Routing agar setiap perangkat pada jaringan tersebut dapat terhubung
```
route add -net 192.189.7.128 netmask 255.255.255.248 gw 192.189.7.145
route add -net 192.189.7.0 netmask 255.255.255.128 gw 192.189.7.145
route add -net 192.189.0.0 netmask 255.255.252.0 gw 192.189.7.145

route add -net 192.189.4.0 netmask 255.255.254.0 gw 192.189.7.150
route add -net 192.189.6.0 netmask 255.255.255.0 gw 192.189.7.150
route add -net 192.189.7.136  netmask 255.255.255.248 gw 192.189.7.150
```

cek routing

```
route -n
```

### D. Tugas berikutnya adalah memberikan ip pada subnet Blueno, Cipher, Fukurou, dan Elena secara dinamis menggunakan bantuan DHCP server. Kemudian kalian ingat bahwa kalian harus setting DHCP Relay pada router yang menghubungkannya.

Jalankan ```nano /etc/network/interfaces```
Kemudian lakukan 

```
auto lo
iface lo inet loopback

auto eth0
iface eth0 inet dhcp

```

### 1. Agar topologi yang kalian buat dapat mengakses keluar, kalian diminta untuk mengkonfigurasi Foosha menggunakan iptables, tetapi Luffy tidak ingin menggunakan MASQUERADE.

Foosha
```
iptables -t nat -A POSTROUTING -s 192.189.0.0/20 -o eth0 -j SNAT --to-source 192.168.122.25
```

Keterangan:
- `-t nat`: Menggunakan tabel NAT karena akan mengubah alamat asal dari paket
- `-A POSTROUTING`: Menggunakan chain POSTROUTING karena mengubah asal paket setelah routing
- `-s 192.189.0.0/20`: Mendifinisikan alamat asal dari paket yaitu semua alamat IP dari subnet 192.189.0.0/20
- `-o eth0`: Paket keluar dari eth0 Foosha
- `-j SNAT`: Menggunakan target SNAT untuk mengubah source atau alamat asal dari paket
- `--to-s (ip eth0)`: Mendefinisikan IP source, di mana digunakan eth0 Foosha dengan IP 192.168.122.25

Testing dapat dilakukan dengan cara sebagai berikut:
- Masukkan command `ip a` pada Foosha untuk mengetahui eth0, ip eth0 akan selalu berganti ketika restart node pada foosha atau restart GNS3 dengan rentang IP yang sudah dijelaskan

### 2. Kalian diminta untuk mendrop semua akses HTTP dari luar Topologi kalian pada server yang merupakan DHCP Server dan DNS Server demi menjaga keamanan.

Jipangu dan Doriki
```
iptables -A FORWARD -d 192.189.4.128/29 -i eth0 -p tcp --dport 80 -j DROP
```

Keterangan:
- `-A FORWARD`: Menggunakan chain FORWARD
- `-d 192.189.4.128/29`: Mendefinisikan alamat tujuan dari paket (DHCP dan DNS SERVER ) berada pada subnet 192.189.4.128/29
- `-i eth0`: Paket masuk dari eth0 Foosha
- `-p tcp`: Mendefinisikan protokol yang digunakan, yaitu tcp
- `--dport 80`: Mendefinisikan port yang digunakan, yaitu 80 (HTTP)
- `-j DROP`: Paket di-drop

Testing dapat dilakukan dengan cara sebagai berikut:
- Install netcat di server Jipangu dan Doriki menggunakan command `apt-get install netcat`
- Pada Jipangu dan Doriki masukkan command `nc -l -p 80`
- Lalu, pada Foosha masukkan command berikut `nmap -p 80 192.189.4.130` atau `nmap -p 80 192.189.4.131`

### 3. Karena kelompok kalian maksimal terdiri dari 3 orang. Luffy meminta kalian untuk membatasi DHCP dan DNS Server hanya boleh menerima maksimal 3 koneksi ICMP secara bersamaan menggunakan iptables, selebihnya didrop.

Doriki dan Jipangu
```
iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
iptables -A INPUT -p icmp -m connlimit --connlimit-above 3 --connlimit-mask 0 -j DROP
```

Keterangan:
- `-A INPUT`: Menggunakan chain INPUT
- `-m state --state`: Menambahkan konfigurasi/kondisi pada iptables
- `ESTABLISHED`: kondisi dimana paket sudah berpergian antara source dan destination
- `RELATED`: kondisi dimana paket akan menginisiasi lagi koneksi baru tapi masih related dengan koneksi existing
- `-p icmp`: Mendefinisikan protokol yang digunakan, yaitu ICMP (ping)
- `-m connlimit`: Menggunakan rule connection limit
- `--connlimit-above 3`: Limit yang ditangkap paket adalah di atas 3
- `--connlimit-mask 0`: Hanya memperbolehkan 3 koneksi setiap subnet dalam satu waktu
- `-j ACCEPT`: Paket diterima
- `-j DROP`: Paket didrop

Testing dapat dilakukan dengan cara sebagai berikut:
- Masuk ke 4 node yang berbeda
- Kemudian, melakukan ping ke arah Jipangu secara bersamaan

### 4. Akses dari subnet Blueno dan Cipher hanya diperbolehkan pada pukul 07.00 - 15.00 pada hari Senin sampai Kamis.

Doriki

Paket yang berasal dari Blueno
```
iptables -A INPUT -s 192.189.0.0/22 -d 192.189.4.128/29 -m time --timestart 07:00 --timestop 15:00 --weekdays Mon,Tue,Wed,Thu -j ACCEPT
iptables -A INPUT -s 192.189.0.0/22 -j REJECT
```

Paket yang berasal dari Cipher
```
iptables -A INPUT -s 192.189.4.0/25 -d 192.189.4.128/29 -m time --timestart 07:00 --timestop 15:00 --weekdays Mon,Tue,Wed,Thu -j ACCEPT
iptables -A INPUT -s 192.189.4.0/25 -j REJECT
```

Keterangan:
- `-A INPUT`: Menggunakan chain INPUT
- `-s 192.189.0.0/22`: Mendifinisikan alamat asal dari paket yaitu IP dari subnet Blueno
- `-s 192.189.4.0/25`: Mendifinisikan alamat asal dari paket yaitu IP dari subnet Cipher
- `-d 192.189.4.128/29`: Mendifinisikan alamat tujuan dari paket yaitu IP dari subnet Doriki
- `-m time`: Menggunakan rule time
- `--timestart 07:00`: Mendefinisikan waktu mulai yaitu 07:00
- `--timestop 15:00`: Mendefinisikan waktu berhenti yaitu 15:00
- `--weekdays Mon,Tue,Wed,Thu`: Mendefinisikan hari yaitu Senin hingga Kamis
- `-j ACCEPT`: Paket diterima
- `-j REJECT`: Paket ditolak

Testing dapat dilakukan dengan cara sebagai berikut: <br />
Ubah waktu menggunakan command sebagai berikut pada Doriki. Kemudian, lakukan `ping google.com` pada client yang aksesnya diperbolehkan atau tidak diperbolehkan
- `date -s "8 nov 2021 10:00:00"`
- `date -s "8 nov 2021 17:00:00"`

### 5. Akses dari subnet Elena dan Fukuro hanya diperbolehkan pada pukul 15.01 hingga pukul 06.59 setiap harinya.

Doriki

Paket yang berasal dari Elena
```
iptables -A INPUT -s 192.189.10.0/23 -m time --timestart 15:01 --timestop 06:59 -j ACCEPT
iptables -A INPUT -s 192.189.10.0/23 -j REJECT
```

Paket yang berasal dari Fukurou
```
iptables -A INPUT -s 192.189.8.0/24 -m time --timestart 15:01 --timestop 06:59 -j ACCEPT
iptables -A INPUT -s 192.189.8.0/24 -j REJECT
```

Keterangan:
- `-A INPUT`: Menggunakan chain INPUT
- `-s 192.189.10.0/23`: Mendifinisikan alamat asal dari paket yaitu IP dari subnet Elena
- `-s 192.189.8.0/24`: Mendifinisikan alamat asal dari paket yaitu IP dari subnet Fukurou
- `-m time`: Menggunakan rule time
- `--timestart 15:01`: Mendefinisikan waktu mulai yaitu 15:01
- `--timestop 06:59`: Mendefinisikan waktu berhenti yaitu 06:59
- `--weekdays Mon,Tue,Wed,Thu`: Mendefinisikan hari yaitu Senin hingga Kamis
- `-j ACCEPT`: Paket diterima
- `-j REJECT`: Paket ditolak

Testing dapat dilakukan dengan cara sebagai berikut: <br />
Ubah waktu menggunakan command sebagai berikut pada Doriki. Kemudian, lakukan `ping google.com` pada client yang aksesnya diperbolehkan atau tidak diperbolehkan
- `date -s "8 nov 2021 10:00:00"`
- `date -s "8 nov 2021 17:00:00"`

### 6. Karena kita memiliki 2 Web Server, Luffy ingin Guanhao disetting sehingga setiap request dari client yang mengakses DNS Server akan didistribusikan secara bergantian pada Jorge dan Maingate

Guanhao

Masukkan command
```
iptables -A PREROUTING -t nat -p tcp -d 192.189.4.128 --dport 80 -m statistic --mode nth --every 2 --packet 0 -j DNAT --to-destination  192.189.9.2:80
iptables -A PREROUTING -t nat -p tcp -d 192.189.4.128 --dport 80 -j DNAT --to-destination 192.189.9.3:80
iptables -t nat -A POSTROUTING -p tcp -d 192.189.9.2 --dport 80 -j SNAT --to-source 192.189.4.128:80
iptables -t nat -A POSTROUTING -p tcp -d 192.189.9.3 --dport 80 -j SNAT --to-source 192.189.4.128:80
```

Testing dapat dilakukan dengan cara sebagai berikut:
- Pada Guanhao, Jorge, Maingate dan Elena gunakan command instal `apt-get install netcat`
- Pada Jorge masukkan command `nc -l -p 80`
- Pada Maingate masukkan command `nc -l -p 80`
- Pada client Elena masukkan command `nc 192.189.4.128 80`
- Hasil akan muncul di salah satu webserver
- Masukkan command `nc 192.189.4.128 80` pada Elena, hasil akan keluar di server kedua
