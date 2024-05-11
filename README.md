## Soal 1

Pada zaman dahulu pada galaksi yang jauh-jauh sekali, hiduplah seorang Stelle. Stelle adalah seseorang yang sangat tertarik dengan Tempat Sampah dan Parkiran Luar Angkasa. Stelle memulai untuk mencari Tempat Sampah dan Parkiran yang terbaik di angkasa. Dia memerlukan program untuk bisa secara otomatis mengetahui Tempat Sampah dan Parkiran dengan rating terbaik di angkasa. Programnya berbentuk microservice sebagai berikut:

a) Dalam auth.c pastikan file yang masuk ke folder new-entry adalah file csv dan berakhiran  trashcan dan parkinglot. Jika bukan, program akan secara langsung akan delete file tersebut. Contoh dari nama file yang akan diautentikasi:

   - belobog_trashcan.csv
   - osaka_parkinglot.csv

b) Format data (Kolom)  yang berada dalam file csv adalah seperti berikut:

![Screenshot from 2024-05-11 17-00-30](https://github.com/Faridaqurr/lapres/assets/150933246/39dd4511-abb0-4bd0-bbe4-ecce56dc44cd)

atau

![Screenshot from 2024-05-11 17-00-50](https://github.com/Faridaqurr/lapres/assets/150933246/3ff99191-f9eb-4c27-ad6a-71be18a31e65)

c) File csv yang lolos tahap autentikasi akan dikirim ke shared memory. 

d) Dalam rate.c, proses akan mengambil data csv dari shared memory dan akan memberikan output Tempat Sampah dan Parkiran dengan Rating Terbaik dari data tersebut.

![Screenshot from 2024-05-11 17-02-35](https://github.com/Faridaqurr/lapres/assets/150933246/201843ae-69c0-4daf-80ef-dc9ac92c97bc)

e) Pada db.c, proses bisa memindahkan file dari new-data ke folder microservices/database, WAJIB MENGGUNAKAN SHARED MEMORY.

f) Log semua file yang masuk ke folder microservices/database ke dalam file db.log dengan contoh format sebagai berikut:
[DD/MM/YY hh:mm:ss] [type] [filename]

ex : `[07/04/2024 08:34:50] [Trash Can] [belobog_trashcan.csv]

Contoh direktori awal:


Contoh direktori akhir setelah dijalankan auth.c dan db.c:


