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
          int len = strlen(filename);
          if (len >= 12 && strcmp(filename + len - 12, "trashcan.csv") == 0) {
              return 1; 
          } else if (len >= 14 && strcmp(filename + len - 14, "parkinglot.csv") == 0) {
              return 1; 
          }
          return 0; 
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
          } else if (pid == 0) { 
              if (remove(filepath) == 0) {
                  printf("File %s tidak memenuhi syarat autentikasi dan telah dihapus.\n", filepath);
              } else {
                  perror("remove");
              }
              exit(EXIT_SUCCESS);
          } else { 
              wait(NULL); // 
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

### soal 1b

Pada soal ini format dalam file csv nya harus sesuai dengan ketentuan soal

![Screenshot from 2024-05-11 19-13-02](https://github.com/Faridaqurr/lapres/assets/150933246/44062f90-113a-4344-8f8f-2ef39783d7be)

### soal 1c

Pada soal ini program auth.c mengirimkan nama file csv yang lolos autentikasi ke shared memory
      
      void copyFilenameToSharedMemory(char *shm_ptr, char *filename) {
          strcpy(shm_ptr, filename);
      }
      
Fungsi diatas menyalin nama file yang lolos autentikasi ke dalam shared memory

`strcpy(shm_ptr, filename);` untuk menyalin string dari _filename_ ke _shm_ptr_ 
      
      int shmid = shmget(SHARED_MEM_KEY, MAX_FILENAME_LENGTH, IPC_CREAT | 0666);
          if (shmid == -1) {
              perror("shmget");
              exit(EXIT_FAILURE);
          }
          char *shm_ptr = shmat(shmid, NULL, 0);
          if (shm_ptr == (char *)-1) {
              perror("shmat");
              exit(EXIT_FAILURE);
          }

Fungsi diatas membuat shared memory 

`0666` untuk token akses shared memory

### soal 1d

pada soal ini membuat program rate.c yang akan mengambil data csv dari shared memory dan akan memberi output tempat sampah dan tempat parkir rating terbaik dari data yang diambil

      struct Place {
          char name[MAX_FILENAME_LENGTH];
          float rating;
      };

Fungsi diatas untuk menyimpan informasi tentang tempat sampah dan tempat parkir seperti nama dan juga rating

`char name[MAX_FILENAME_LENGTH];` untuk  mendeklarasikan sebuah array karakter dengan nama _name_ yang dapat menampung string dengan panjang maksimum yang telah di define

`float rating;` untuk mendeklarasikan variabel _rating_ bertipe float yang menyimpan data rating dari file csv

      struct Place findBestPlace(FILE *file) {
          struct Place bestPlace = {"", 0.0};
          char line[MAX_FILENAME_LENGTH];
          fgets(line, sizeof(line), file);
      
          while (fgets(line, sizeof(line), file) != NULL) {
              char name[MAX_FILENAME_LENGTH];
              float rating;
              sscanf(line, "%[^,],%f", name, &rating);
      
              if (rating > bestPlace.rating) {
                  strcpy(bestPlace.name, name);
                  bestPlace.rating = rating;
              }
          }
          return bestPlace;
      }

Fungsi diatas untuk mencari tempat dengan rating tertinggi

`struct Place bestPlace = {"", 0.0};` untuk menyimpan informasi tempat dengan rating tertinggi dengan mengatur nama tempat kosong dan rating 0.0 

`fgets(line, sizeof(line), file);` untuk membaca dan membuang baris pertama atau baris header dalam file, seperti baris yang berisikan nama, rating

`sscanf(line, "%[^,],%f", name, &rating);` untuk membaca isi file dengan penjelasan:
- *%[^,]*  dan  *name* untuk membaca string hingga bertemu tanda koma dan akan menyimpannya ke dalam variabel name

- *,%f*  dan  *&rating* untuk  membaca nilai float setelah tanda koma dan akan menyimpannya ke dalam variabel rating

`while (fgets(line, sizeof(line), file) != NULL)` untuk membaca data dari file csv

      int main(int argc, char *argv[]) {
          if (argc != 2) {
              fprintf(stderr, "Usage: %s <filename.csv>\n", argv[0]);
              exit(EXIT_FAILURE);
          }
          
`if (argc != 2)` untuk menjalankan program rate.c ini harus menggunakan argumen seperti ./rate <nama filenya> 

       int shmid = shmget(SHARED_MEM_KEY, MAX_FILENAME_LENGTH, 0666);
          if (shmid == -1) {
              perror("shmget");
              exit(EXIT_FAILURE);
          }
          char *shm_ptr = shmat(shmid, NULL, 0);
          if (shm_ptr == (char *)-1) {
              perror("shmat");
              exit(EXIT_FAILURE);
          }

`int shmid = shmget(SHARED_MEM_KEY, MAX_FILENAME_LENGTH, 0666);` untuk akses shared memory

`0666` untuk token shared memory yang digunakan oleh auth.c dan db.c
      
          char filename[MAX_FILENAME_LENGTH];
          strcpy(filename, argv[1]);
      
          printf("Type: ");
          if (strstr(filename, "trashcan") != NULL) {
              printf("Trash Can\n");
          } else if (strstr(filename, "parkinglot") != NULL) {
              printf("Parking Lot\n");
          } else {
              printf("Unknown\n");
          }
          printf("Filename: %s\n", filename);
          printf("---------------------------------------\n");
      
          char filepath[MAX_FILENAME_LENGTH + 20]; 
          sprintf(filepath, "./database/%s", filename);
          FILE *file = fopen(filepath, "r");
          if (file == NULL) {
              perror("fopen");
              exit(EXIT_FAILURE);
          }
      
          struct Place bestPlace = findBestPlace(file);
          printf("Name: %s\n", bestPlace.name);
          printf("Rating: %.1f\n", bestPlace.rating);
      
          fclose(file);
          if (shmdt(shm_ptr) == -1) {
              perror("shmdt");
              exit(EXIT_FAILURE);
          }
      
          return 0;
      }

`strcpy(filename, argv[1]);` untuk membaca nama file yang ingin di proses

`printf("Type: ");` untuk penulisan output Type filenya

`printf("Filename: %s\n", filename);` untuk penulisan output Filename

`sprintf(filepath, "./database/%s", filename);` akses file pada folder database

`printf("Name: %s\n", bestPlace.name);` untuk penulisan Name dari nama tempat dengan rating terbaik

`printf("Rating: %.1f\n", bestPlace.rating);` untuk penulisan Rating dari rating tertinggi

`if (shmdt(shm_ptr) == -1)` untuk melepaskan shared memory ketika program sudah selesai

### soal 1e

Pada soal ini diminta untuk membuat prgram db.c yang dapat memindahkan file dari new-data ke folder microservices/database menggunakan shared memory
      
      #define SHARED_MEM_KEY 1234
      #define MAX_FILENAME_LENGTH 256
      #define LOG_FILE "database/db.log"

`#define SHARED_MEM_KEY 1234` untuk akses shared memory

`#define LOG_FILE "database/db.log"` untuk mendefinisikan pembuatan db.log di dalam direktori database
      
      struct FileData {
          char filename[MAX_FILENAME_LENGTH];
      };
      
`char filename[MAX_FILENAME_LENGTH];` untuk menyimpan informasi mengenai semua file agar mudah untuk dikelola

      void copyFilenameFromSharedMemory(char *shm_ptr, char *filename) {
          strcpy(filename, shm_ptr);
      }
      
`strcpy(shm_ptr, filename);` untuk menyalin string dari _filename_ ke _shm_ptr_ 

      void moveFile(char *filename) {
          char oldpath[MAX_FILENAME_LENGTH + 20]; 
          sprintf(oldpath, "../new-data/%s", filename);
          char newpath[MAX_FILENAME_LENGTH + 30];
          sprintf(newpath, "./database/%s", filename);
      
          mkdir("./database", 0777);
      
          if (rename(oldpath, newpath) == 0) {
              printf("File %s berhasil dipindahkan ke microservices/database.\n", filename);
          } else {
              perror("rename");
          }
      }
      
Fungsi diatas untuk memindahkan file ke folder database

`sprintf(oldpath, "../new-data/%s", filename);` untuk akses ke folder new-data

`sprintf(newpath, "./database/%s", filename);` untuk akses ke folder database

`mkdir("./database", 0777);` untuk membuat folder database jika belum ada di dalam microservices

      char* determineFileType(char *filename) {
          if (strstr(filename, "trashcan") != NULL) {
              return "Trash Can";
          } else if (strstr(filename, "parkinglot") != NULL) {
              return "Parking Lot";
          } else {
              return "Unknown";
          }
      }

Fungsi diatas untuk menentukan Type file berdasarkan akhiran nama file

`if (strstr(filename, "trashcan") != NULL)` untuk Type file yang berakhiran trashcan

`else if (strstr(filename, "parkinglot") != NULL)` untuk Type file yang berakhiran parking lot

`return "Unknown";` untuk kondisi jika tidak ada file yang berakhiran trashcan atau parkinglot
       
          int main() {
          int shmid = shmget(SHARED_MEM_KEY, MAX_FILENAME_LENGTH, 0666);
          if (shmid == -1) {
              perror("shmget");
              exit(EXIT_FAILURE);
          }
          char *shm_ptr = shmat(shmid, NULL, 0);
          if (shm_ptr == (char *)-1) {
              perror("shmat");
              exit(EXIT_FAILURE);
          }
Fungsi untuk mengakses shared memory dari program auth.c

`0666` token yang digunakan untuk akses shared memory

          // Melepaskan shared memory
          if (shmdt(shm_ptr) == -1) {
              perror("shmdt");
              exit(EXIT_FAILURE);
          }
Fungsi untuk melepas shared memory jika proses sudah selesai dijalankan

### soal 1f

Pada soal ini diminta untuk membuat log file yang masuk ke folder microservices/database ke dalam file db.log dengan contoh format [DD/MM/YY hh:mm:ss] [type] [filename]
      
      void writeLog(char *type, char *filename) {
          FILE *log_file = fopen(LOG_FILE, "a");
          if (log_file != NULL) {
              time_t now = time(NULL);
              struct tm *tm_info = localtime(&now);
              char timestamp[20];
              strftime(timestamp, 20, "%d/%m/%Y %H:%M:%S", tm_info);
              fprintf(log_file, "[%s] [%s] [%s]\n", timestamp, type, filename);
              fclose(log_file);
          } else {
              perror("fopen");
          }
      }

Fungsi untuk menuliskan log ke dalam db.log ketika file dari folder new-data dipindahkan ke folder database

`FILE *log_file = fopen(LOG_FILE, "a");` untuk membuka db.log dan melakukan penulisan log di dalamnya, namun jika belum ada db.log di dalam database maka akan otomatis dibuat

`fprintf(log_file, "[%s] [%s] [%s]\n", timestamp, type, filename);` untuk penulisan format dari db.log

`time_t now = time(NULL);` untuk mendapatkan waktu ketika file dipindahkan

`struct tm *tm_info = localtime(&now);` untuk mengonversi waktu dalam bentuk time_t menjadi waktu lokal (dd/mm/yyyy HH:MM:SS)

`char timestamp[20];` untuk menampung hasil dari konversi waktu

## Revisi

Pada code rate.c sebelumnya hanya bisa dijalankan untuk mengeluarkan output per file menggunakan ./rate <nama file>. Sehingga pada revisi ini code diubah agar juga dapat dijalankan untuk mengeluarkan output semua file dengan satu command yaitu ./rate

code rate.c yang diperbarui:
      
      #include <stdio.h>
      #include <stdlib.h>
      #include <string.h>
      #include <sys/ipc.h>
      #include <sys/shm.h>
      #include <dirent.h> 
      
      #define SHARED_MEM_KEY 1234
      #define MAX_FILENAME_LENGTH 256
      
      struct Place {
          char name[MAX_FILENAME_LENGTH];
          float rating;
      };
      
      struct Place findBestPlace(FILE *file) {
          struct Place bestPlace = {"", 0.0};
          // Melewati baris header
          char line[MAX_FILENAME_LENGTH];
          fgets(line, sizeof(line), file);
      
          while (fgets(line, sizeof(line), file) != NULL) {
              char name[MAX_FILENAME_LENGTH];
              float rating;
              sscanf(line, "%[^,],%f", name, &rating);
      
              if (rating > bestPlace.rating) {
                  strcpy(bestPlace.name, name);
                  bestPlace.rating = rating;
              }
          }
          return bestPlace;
      }
      
      void processCSV(char *filename) {
          // Mengakses shared memory yang telah dibuat oleh program auth.c dan db.c
          int shmid = shmget(SHARED_MEM_KEY, MAX_FILENAME_LENGTH, 0666);
          if (shmid == -1) {
              perror("shmget");
              exit(EXIT_FAILURE);
          }
          char *shm_ptr = shmat(shmid, NULL, 0);
          if (shm_ptr == (char *)-1) {
              perror("shmat");
              exit(EXIT_FAILURE);
          }
      
          // Menampilkan output
          printf("\nType: ");
          if (strstr(filename, "trashcan") != NULL) {
              printf("Trash Can\n");
          } else if (strstr(filename, "parkinglot") != NULL) {
              printf("Parking Lot\n");
          } else {
              printf("Unknown\n");
          }
          printf("Filename: %s\n", filename);
          printf("---------------------------------------\n");
      
          // Membuka file CSV yang terletak di folder database
          char filepath[MAX_FILENAME_LENGTH + 20]; // Menyertakan path folder "database"
          sprintf(filepath, "./database/%s", filename);
          FILE *file = fopen(filepath, "r");
          if (file == NULL) {
              perror("fopen");
              exit(EXIT_FAILURE);
          }
      
          // Mencari tempat dengan rating tertinggi dari file CSV dan menampilkannya
          struct Place bestPlace = findBestPlace(file);
          printf("Name: %s\n", bestPlace.name);
          printf("Rating: %.1f\n", bestPlace.rating);
      
          // Menutup file dan melepaskan shared memory
          fclose(file);
          if (shmdt(shm_ptr) == -1) {
              perror("shmdt");
              exit(EXIT_FAILURE);
          }
      }
      
      int main(int argc, char *argv[]) {
          if (argc == 1) {
              // Jika tidak ada argumen yang diberikan, tampilkan output untuk semua file CSV
              DIR *dir;
              struct dirent *ent;
              if ((dir = opendir("./database")) != NULL) {
                  while ((ent = readdir(dir)) != NULL) {
                      if (strstr(ent->d_name, ".csv") != NULL) {
                          processCSV(ent->d_name);
                      }
                  }
                  closedir(dir);
              } else {
                  perror("opendir");
                  return EXIT_FAILURE;
              }
          } else if (argc == 2) {
              // Jika diberikan satu argumen, tampilkan output untuk file CSV yang disebutkan
              processCSV(argv[1]);
          } else {
              // Jika diberikan lebih dari satu argumen, tampilkan pesan kesalahan
              fprintf(stderr, "Usage: %s [filename.csv]\n", argv[0]);
              return EXIT_FAILURE;
          }
      
          return 0;
      }

1)Penambahan fungsi _processCSV_, karena pada code sebelumnya fungsi untuk outputnya dijabarkan di main-nya berbeda pada revisi code ini, pada code ini hanya melakukan pemanggilan fungsi output pada main-nya sedangkan untuk fungsi outputnya sendiri sudah di buat pada fungsi processCSV

     void processCSV(char *filename) {
             int shmid = shmget(SHARED_MEM_KEY, MAX_FILENAME_LENGTH, 0666);
             if (shmid == -1) {
                 perror("shmget");
                 exit(EXIT_FAILURE);
             }
             char *shm_ptr = shmat(shmid, NULL, 0);
             if (shm_ptr == (char *)-1) {
                 perror("shmat");
                 exit(EXIT_FAILURE);
             }
         
             printf("\nType: ");
             if (strstr(filename, "trashcan") != NULL) {
                 printf("Trash Can\n");
             } else if (strstr(filename, "parkinglot") != NULL) {
                 printf("Parking Lot\n");
             } else {
                 printf("Unknown\n");
             }
             printf("Filename: %s\n", filename);
             printf("---------------------------------------\n");
         
             char filepath[MAX_FILENAME_LENGTH + 20]; 
             sprintf(filepath, "./database/%s", filename);
             FILE *file = fopen(filepath, "r");
             if (file == NULL) {
                 perror("fopen");
                 exit(EXIT_FAILURE);
             }
         
             struct Place bestPlace = findBestPlace(file);
             printf("Name: %s\n", bestPlace.name);
             printf("Rating: %.1f\n", bestPlace.rating);
         
             fclose(file);
             if (shmdt(shm_ptr) == -1) {
                 perror("shmdt");
                 exit(EXIT_FAILURE);
             }
         }

`int shmid = shmget(SHARED_MEM_KEY, MAX_FILENAME_LENGTH, 0666);` untuk akses shared memory

`printf("\nType: ");` untuk penulisan output Type file

`printf("Filename: %s\n", filename);` untuk penulisan output Filename

`sprintf(filepath, "./database/%s", filename);` untuk akses file di database

`printf("Name: %s\n", bestPlace.name);` untuk penulisan Name dari nama tempat dengan rating terbaik

`printf("Rating: %.1f\n", bestPlace.rating);` untuk penulisan Rating dari rating tertinggi

`if (shmdt(shm_ptr) == -1)` untuk melepaskan shared memory ketika program sudah selesai

2)Pengubahan di dalam main, karena isi main code sebelumnya adalah penjabaran untuk tampilan outputnya maka pada main code revisi ini hanya ada pemanggilan fungsi processCSV dan juga cara menjalankan code revisi ini sekarang bisa menggunakan argumen (nama file) atau tanpa argumen

      int main(int argc, char *argv[]) {
          if (argc == 1) {
              DIR *dir;
              struct dirent *ent;
              if ((dir = opendir("./database")) != NULL) {
                  while ((ent = readdir(dir)) != NULL) {
                      if (strstr(ent->d_name, ".csv") != NULL) {
                          processCSV(ent->d_name);
                      }
                  }
                  closedir(dir);
              } else {
                  perror("opendir");
                  return EXIT_FAILURE;
              }
          } else if (argc == 2) {
              processCSV(argv[1]);
          } else {
              // Jika diberikan lebih dari satu argumen, tampilkan pesan kesalahan
              fprintf(stderr, "Usage: %s [filename.csv]\n", argv[0]);
              return EXIT_FAILURE;
          }
      
          return 0;
      }

Jika pada code sebelumnya hanya ada `if (argc != 2)` untuk memunculkan output sesuai argumen yang diberikan (nama filenya), maka pada code revisinya menjadi:
   
   - `if (argc == 1)` jika tidak ada argumen yang ditambahkan maka akan menampilkan semua output file
     
   - `else if (argc == 2)` jika diberikan argumen (nama file) maka hanya akan menampilkan output dari file yang diminta
     
   - `else` jika argumen yang dimasukkan lebih, maka akan memunculkan pesan kesalahan

## Hasil Output

![Screenshot from 2024-05-10 04-32-36](https://github.com/Faridaqurr/lapres/assets/150933246/4326ede1-216d-4c38-b996-3c5f60ca54f1)

![Screenshot from 2024-05-10 04-27-58](https://github.com/Faridaqurr/lapres/assets/150933246/5492e967-977a-4561-a30b-c7b0f4b18dbc)

![Screenshot from 2024-05-10 04-29-19](https://github.com/Faridaqurr/lapres/assets/150933246/6c9d3264-2629-48c4-8624-a42aa5175f4e)

![Screenshot from 2024-05-10 04-32-36](https://github.com/Faridaqurr/lapres/assets/150933246/2c11c6ef-9b8d-4384-ba40-1898fad25339)

