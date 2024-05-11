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

![Screenshot from 2024-05-11 17-09-58](https://github.com/Faridaqurr/lapres/assets/150933246/a9e9954d-63c4-402c-8ebb-7568502047bd)

Contoh direktori akhir setelah dijalankan auth.c dan db.c:

![Screenshot from 2024-05-11 17-10-26](https://github.com/Faridaqurr/lapres/assets/150933246/b003a07b-c4ff-4f51-af9c-05efd2baac8a)

## Jawab

### soal 1a

Pada soal ini diminta untuk membuat sebuah program yang mengautentikasi file csv dengan syarat file berakhiran _parkinglot_ atau _trashacan_ dan menghapus file yang tidak memenuhi syarat autentikasi
      
      #define SHARED_MEM_KEY 1234
      #define MAX_FILENAME_LENGTH 256

`#define SHARED_MEM_KEY 1234` untuk mendefinisikan kunci/token dari shared memory yang dipakai program ini

`#define MAX_FILENAME_LENGTH 256` untuk mendefinisikan panjang karakter nama file adalah 256 dan menghindari terjadinya perulangan angka 256 di tempat yang berbeda
      
      struct FileData {
          char filename[MAX_FILENAME_LENGTH];
      };

`char filename[MAX_FILENAME_LENGTH];` untuk menyimpan informasi mengenai semua file agar mudah untuk dikelola

      int isFileAuthenticated(char *filename) {
          // Memeriksa apakah nama file berakhir dengan "trashcan.csv" atau "parkinglot.csv"
          int len = strlen(filename);
          if (len >= 12 && strcmp(filename + len - 12, "trashcan.csv") == 0) {
              return 1; // File lolos autentikasi
          } else if (len >= 14 && strcmp(filename + len - 14, "parkinglot.csv") == 0) {
              return 1; // File lolos autentikasi
          }
          return 0; // File tidak lolos autentikasi
      }

`if (len >= 12 && strcmp(filename + len - 12, "trashcan.csv") == 0)` untuk syarat autentikasi file jika berakhiran trashcan.csv dimana berjumlah 12 karakter

`else if (len >= 14 && strcmp(filename + len - 14, "parkinglot.csv") == 0)` untuk syarat autentikasi file jika berakhiran parkinglot.csv dimana berjumlah 14 karakter

`return 1` untuk menunjukkan indikasi bahwa file memenuhi syarat autentukasi

`return 0` untuk menunjukkan indikasi bahwa file tidak memenuhi syarat autentikasi

      void deleteFile(char *filepath) {
          pid_t pid = fork();
          if (pid < 0) {
              perror("fork");
              exit(EXIT_FAILURE);
          } else if (pid == 0) { // Proses anak
              if (remove(filepath) == 0) {
                  printf("File %s tidak memenuhi syarat autentikasi dan telah dihapus.\n", filepath);
              } else {
                  perror("remove");
              }
              exit(EXIT_SUCCESS);
          } else { // Proses induk
              wait(NULL); // Memanggil wait di sini
          }
      }

Fungsi diatas digunakan untuk menghapus file yang tidak memenuhi syarat autentikasi

`pid_t pid = fork();` untuk memanggil fungsi sistem fork

`if (pid < 0)` untuk kondisi jika gagal membuat proses baru

`else if (pid == 0)` untuk kondisi pembuatan proses baru berhasil dibuat

`wait(NULL);` untuk eksekusi proses induk agar menunggu proses childnya selesai berproses dahulu

          if (shmdt(shm_ptr) == -1) {
              perror("shmdt");
              exit(EXIT_FAILURE);
          }
          
 Fungsi diatas untuk melepaskan shared memory setelah proses program selesai agar tidak terjadi kebocoran data 
