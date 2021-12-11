# Jarkom-Modul-5-B12-2021

Anggota: Faisal Reza M (05111940000009)

Spesifikasi: harus ada screenshot, cara pengerjaan, kendala

## Cara pengerjaan
### AB. Tugas pertama kalian yaitu membuat topologi jaringan sesuai dengan rancangan yang diberikan Luffy dibawah ini. Karena kalian telah belajar subnetting dan routing, Luffy ingin meminta kalian untuk membuat topologi tersebut menggunakan teknik CIDR atau VLSM. (berikut adalah setelah dilakukan subnetting VLSM):

![vlsm 2 drawio](https://user-images.githubusercontent.com/11045113/145683788-f939e79f-28dc-428f-9274-803d8ee6169c.png)
![subnet](https://user-images.githubusercontent.com/11045113/145683775-26c0b210-0d21-42fc-865e-40f0275b7f1a.png)
![image](https://user-images.githubusercontent.com/11045113/145684629-8f0778d9-fa9c-4f8c-82d7-31aaefcc7b2a.png)

### C. Kalian juga diharuskan melakukan Routing agar setiap perangkat pada jaringan tersebut dapat terhubung.
Routing hanya dilakukan di Foosha karena untuk node lain sudah dihandle oleh config gateway
fooosha:
```
route add -net 10.13.7.16 netmask 255.255.255.248 gw 10.13.7.2
route add -net 10.13.7.128 netmask 255.255.255.128 gw 10.13.7.2
route add -net 10.13.0.0 netmask 255.255.252.0 gw 10.13.7.2
route add -net 10.13.4.0 netmask 255.255.254.0 gw 10.13.7.6
route add -net 10.13.7.24 netmask 255.255.255.248 gw 10.13.7.6
route add -net 10.13.6.0 netmask 255.255.255.0 gw 10.13.7.6
```

### D. Tugas berikutnya adalah memberikan ip pada subnet Blueno, Cipher, Fukurou, dan Elena secara dinamis menggunakan bantuan DHCP server. Kemudian kalian ingat bahwa kalian harus setting DHCP Relay pada router yang menghubungkannya.
Install DHCP relay di Foosha, Guanhao, dan Water7, lalu saat ditanya IP, masukkan 10.13.7.19 (IP Jipangu) dan saat ditanyai interface, masukkan eth0 eth1 eth2 eth3

Install DHCP server di Jipangu, lalu ganti file dhcpd.conf menjadi
```
#a2
subnet 10.13.7.128 netmask 255.255.255.128{
  range 10.13.7.130 10.13.7.254;
  option routers 10.13.7.129;
  option broadcast-address 10.13.7.255;
  option domain-name-servers 10.13.7.18;
  default-lease-time 600;
  max-lease-time 7200;
}
#a5
subnet 10.13.4.0 netmask 255.255.254.0{
  range 10.13.4.2 10.13.5.254;
  option routers 10.13.4.1;
  option broadcast-address 10.13.5.255;
  option domain-name-servers 10.13.7.18;
  default-lease-time 600;
  max-lease-time 7200;
}
#a7
subnet 10.13.6.0 netmask 255.255.255.0{
  range 10.13.6.2 10.13.6.254;
  option routers 10.13.6.1;
    option broadcast-address 10.13.6.255;
  option domain-name-servers 10.13.7.18;
  default-lease-time 600;
  max-lease-time 7200;
}
#a8
subnet 10.13.0.0 netmask 255.255.252.0{
  range 10.13.0.2 10.13.3.254;
  option routers 10.13.0.1;
  option broadcast-address 10.13.3.255;
  option domain-name-servers 10.13.7.18;
  default-lease-time 600;
  max-lease-time 7200;
}
subnet 10.13.7.16 netmask 255.255.255.248 {
}
```

Potret blueno mendapatkan ip:

![image](https://user-images.githubusercontent.com/11045113/145685020-fdd50cca-ef62-48a2-b752-29e19d7f9114.png)

### 1.Agar topologi yang kalian buat dapat mengakses keluar, kalian diminta untuk mengkonfigurasi Foosha menggunakan iptables, tetapi Luffy tidak ingin menggunakan MASQUERADE!
Agar topologi yang kalian buat dapat mengakses keluar, kalian diminta untuk mengkonfigurasi Foosha menggunakan iptables, tetapi Luffy tidak ingin menggunakan MASQUERADE!
berikan ip static pada foosha
![image](https://user-images.githubusercontent.com/11045113/145685037-e13142ca-7bc9-455a-b71f-5326a6b23c75.png)

tambahkan line `iptables -t nat -A POSTROUTING -o eth0 -s 10.13.0.0/16 -j SNAT --to-s 192.168.122.2` di baris ke-2 script sh di foosha (setelah echo > resolv.conf)

### 2.Kalian diminta untuk mendrop semua akses HTTP dari luar Topologi kalian pada server yang merupakan DHCP Server dan DNS Server demi menjaga keamanan!
tambahkan line `iptables -A FORWARD -i eth0 -p tcp --dport 80 -d 10.13.7.16/29 -j DROP` di akhir script.sh foosha

### 3.Karena kelompok kalian maksimal terdiri dari 3 orang. Luffy meminta kalian untuk membatasi DHCP dan DNS Server hanya boleh menerima maksimal 3 koneksi ICMP secara bersamaan menggunakan iptables, selebihnya didrop.
doriki: tambahkan di akhir script.sh `iptables -A INPUT -p icmp -m connlimit --connlimit-above 3 --connlimit-mask 0 -j DROP`

ping bisa dilakukan pada 3 node pertama, namun setelah ping dilakukan di node ke4, maka koneksi akan di-drop

![image](https://user-images.githubusercontent.com/11045113/145685532-4163ac2e-9106-4c36-8b44-c2f69a76331f.png)
![image](https://user-images.githubusercontent.com/11045113/145685541-596961b6-d71a-4f67-b075-a8d0c7426f4a.png)
![image](https://user-images.githubusercontent.com/11045113/145685545-2432b85c-6d42-4efe-8d5f-61dc13bea113.png)
![image](https://user-images.githubusercontent.com/11045113/145685550-53507251-b96f-4c9a-bb52-b7dbe0a1e3cd.png)

### 4.Akses dari subnet Blueno dan Cipher hanya diperbolehkan pada pukul 07.00 - 15.00 pada hari Senin sampai Kamis.
doriki: tambahkan di akhir script.sh
```
iptables -A INPUT -s 10.13.7.128/25 -m time --timestart 07:00 --timestop 15:00 --weekdays Mon,Tue,Wed,Thu -j ACCEPT
iptables -A INPUT -s 10.13.7.128/25 -j REJECT

iptables -A INPUT -s 10.13.0.0/22 -m time --timestart 07:00 --timestop 15:00 --weekdays Mon,Tue,Wed,Thu -j ACCEPT
iptables -A INPUT -s 10.13.0.0/22 -j REJECT
```

![image](https://user-images.githubusercontent.com/11045113/145685321-075a9faa-a89d-4e80-8b51-6eda7f29c8fd.png)

karena di doriki sudah malam, dia tidak bisa diakses blueno

![image](https://user-images.githubusercontent.com/11045113/145685337-000d3e18-fbc9-4369-8ad6-858d9495a2cb.png)

### 5.Akses dari subnet Elena dan Fukuro hanya diperbolehkan pada pukul 15.01 hingga pukul 06.59 setiap harinya.
doriki: tambahkan di akhir script.sh
```
iptables -A INPUT -s 10.13.4.0/23 -m time --timestart 15:01 --timestop 23:59 -j ACCEPT
iptables -A INPUT -s 10.13.4.0/23 -m time --timestart 00:00 --timestop 06:59 -j ACCEPT
iptables -A INPUT -s 10.13.4.0/23 -j REJECT

iptables -A INPUT -s 10.13.6.0/24 -m time --timestart 15:01 --timestop 23:59 -j ACCEPT
iptables -A INPUT -s 10.13.6.0/24 -m time --timestart 00:00 --timestop 06:59 -j ACCEPT
iptables -A INPUT -s 10.13.6.0/24 -j REJECT
```

karena kebetulan di doriki sudah malam, elena bisa mengakses blueno

![image](https://user-images.githubusercontent.com/11045113/145685420-2888bc53-e755-45f4-a2d3-c2829ccc896d.png)

### 6.Karena kita memiliki 2 Web Server, Luffy ingin Guanhao disetting sehingga setiap request dari client yang mengakses DNS Server akan didistribusikan secara bergantian pada Jorge dan Maingate
doriki:
edit named.conf.local
```
zone "b12.com" {
        type master;
        file "/etc/bind/kaizoku/b12.com";
};
```

edit b12.com (di situs tersebut ada ip yang mengarah ke subnet a6, yang belum terassign ke node manapun yaitu 10.13.7.28)

![image](https://user-images.githubusercontent.com/11045113/145685632-a7aa2a97-d0c5-4f52-a91e-edef39ffec14.png)

guanhao: di akhir script.sh
```
iptables -t nat -A PREROUTING -d 10.13.7.28 -m state --state NEW -m statistic --mode nth --every 2 --packet 0 -j DNAT --to-destination 10.13.7.26
iptables -t nat -A PREROUTING -d 10.13.7.28 -j DNAT --to-destination 10.13.7.27
```

screenshot (mencoba membuka 10.13.7.28 menggunakan lynx, jika kita reload maka kita akan disajikan server yang berbeda2):

![image](https://user-images.githubusercontent.com/11045113/145685714-40f835c4-bba3-403f-bb3d-33f577434fe3.png)
![image](https://user-images.githubusercontent.com/11045113/145685701-678c07e8-ac75-48fa-be19-471568316092.png)
![image](https://user-images.githubusercontent.com/11045113/145685704-bbcfd222-b660-4072-b50e-824f6f603814.png)

Kendala:
- tidak ada
