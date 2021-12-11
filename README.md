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
