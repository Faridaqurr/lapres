## Soal 1

Gavriel adalah seorang cyber security enthusiast. Suatu hari, ia dikontrak oleh sebuah perusahaan ternama untuk membuat sebuah program yang cukup rumit dengan bayaran jutaan rupiah. Karena tergoda dengan nominal bayaran tersebut, Gavriel langsung menerima tawaran tersebut. Setelah mencoba membuat dan mengembangkan program tersebut selama seminggu, laptop yang digunakan Gavriel mengalami overheat dan mati total sehingga harus dilarikan ke tukang servis terdekat. Karena keterbatasan waktu dalam pembuatan program dan tidak ingin mengecewakan perusahaan, Gavriel meminta bantuan kalian untuk membuat program tersebut dengan ketentuan sebagai berikut:


a. Program dapat menerima input path berupa ‘argv’ untuk mengatur folder dimana file akan dieksekusi
      
b. Program tersebut berfungsi untuk mereplace string dengan ketentuan sebagai berikut:

        String m4LwAr3 direplace dengan string [MALWARE]
        String 5pYw4R3 direplace dengan string [SPYWARE]
        String R4nS0mWaR3 direplace dengan string [RANSOMWARE]
        
c. Program harus berjalan secara daemon, dan tidak diperbolehkan menggunakan command system() dalam pembuatan program

d. Program akan secara terus menerus berjalan di background dengan jeda 15 detik

e. Catat setiap penghapusan string yang dilakukan oleh program pada sebuah file bernama virus.log dengan format: [dd-mm-YYYY][HH:MM:SS] Suspicious string at <file_name> successfully replaced!

Contoh penggunaan: ./virus /home/user/virus

Contoh isi file sebelum program dijalankan:

        pU=-JWQ$5$)D-[??%AVh]$cB6bm4LwAr3jEQC2p3R{HV]=-AUaxj:Qe+h
        !aNX,i:!z3W=2;.tHc3&S+}6F)CFf%tfZLP1*w5m1PAzZJUux(
        Pd&f8$F5=E?@#[6jd{TJKj]5pYw4R3{KK1?hz384$ge@iba5GAj$gqB41
        #C&&a}M9C#f64Eb.?%c)dGbCvJXtU[?SE4h]BY4e1PR4nS0mWaR3{]S/{w?*

Contoh isi file setelah setelah program dijalankan:

        pU=-JWQ$5$)D-[??%AVh]$cB6b[MALWARE]jEQC2p3R{HV]=-AUaxj:Qe+h
        !aNX,i:!z3W=2;.tHc3&S+}6F)CFf%tfZLP1*w5m1PAzZJUux(
        Pd&f8$F5=E?@#[6jd{TJKj][SPYWARE]{KK1?hz384$ge@iba5GAj$gqB41
        #C&&a}M9C#f64Eb.?%c)dGbCvJXtU[?SE4h]BY4e1P[RANSOMWARE]{]S/{w?*

## ! Penyelesaian !

### soal 2a

Pada soal ini diminta agar dapat menerima input path berupa ‘argv’ untuk mengatur folder dimana file akan dieksekusi

        int main(int argc, char *argv[]) {
                if (argc != 2) {
                printf("Usage: %s <path>\n", argv[0]);
                exit(EXIT_FAILURE);
        }

`if (argc != 2)` digunakan untuk memberikan keterangan jika format untuk menjalankan program berjumlah dua argumen yaitu argumen pertama (argv[0]) adalah nama program dan  argumen kedua (argv[1]) adalah path direktori yang akan dieksekusi

`printf("Usage: %s <path>\n", argv[0]);` digunakan untuk format argumen yang digunakan untuk menjalankan program

### soal 2b

pada soal ini diminta agar program berfungsi untuk mereplace string sesuai permintaan 

                replaceString(filename, "m4LwAr3", "[MALWARE]", log);
                replaceString(filename, "5pYw4R3", "[SPYWARE]", log);
                replaceString(filename, "R4nS0mWaR3", "[RANSOMWARE]", log);
            
`replaceString(...` digunakan untuk memanggil fungsi replaceString

`...filename,...` digunakan untuk nama file yang ingin diganti stringnya

`..."m4LwAr3",...` digunakan untuk string yang ingin diganti

`..."[MALWARE]",...` digunakan untuk string pengganti (menggantikan string yang ingin diganti)

`...log);` digunakan untuk mencatat semua perubahan yang terjadi selama proses (penggantian string) berjalan

### soal 2c 

pada soal ini diminta agar berjalan secara daemon, dan tidak diperbolehkan menggunakan command system() dalam pembuatan program

        // Forking to daemon
        pid_t pid = fork();
        if (pid < 0) {
                perror("Error forking");
                exit(EXIT_FAILURE);
        }
        if (pid > 0) {
                // Exiting from parent process
                exit(EXIT_SUCCESS);
        }
        
        // Changing working directory
        if (chdir("/") < 0) {
                perror("Error changing directory");
                exit(EXIT_FAILURE);
        }
        
        // Closing standard file descriptors
        close(STDIN_FILENO);
        close(STDOUT_FILENO);
        close(STDERR_FILENO);
        
`pid_t pid = fork();` digunakan untuk membuat proses baru (child process)

`if (pid < 0) {...` digunakan untuk kondisi ketika fork gagal maka akan mencetak pesan yang ada pada fungsi tersebut

`if (pid > 0) {...` digunakan untuk kondisi ketika fork berhasil maka akan mencetak pesan yang ada pada fungsi tersebut

`if (chdir("/") < 0) {...` digunakan untuk kondisi ketika daemon  gagal memastikan bahwa tidak tidak ada ketergantungan pada direktori spesifik saat berjalan sekaligus mencegah terjadinya masalah ketika direktori asal mungkin dihapus atau diganti

`close(STDIN_FILENO);` digunakan untuk menutup standar file deskriptor agar daemon tidak menerima input atau memunculkan output ke terminal

### soal 2d

pada soal ini diminta agar program dapat berjalan terus menerus di background dengan jeda 15 detik

        // Daemon main loop
        while (1) {
            checkAndReplace(path, log);
            fflush(log);
            sleep(15); // Delay for 15 seconds
        }

`while (1) {...` digunakan untuk perulangan tanpa henti

`checkAndReplace(path, log);` digunakan untuk memanggil fungsi checkAndReplace setiap perulangan untuk mengecek apakah ada pola string (seperti string malware) dalam direktori, jika iya maka akan menjalankan program sesuai ketentuan

`fflush(log);` digunakan untuk memastikan bahwa data ditulis ke log secara benar

`sleep(15);}` digunakan untuk menidurkan program setelah melakukan iterasi tiap 15 detik

### soal 2e

pada soal ini diminta agar setiap penggantian string yang dilakukan oleh program akan disimpan pada file virus.log dengan format: [dd-mm-YYYY][HH:MM:SS] Suspicious string at <file_name> successfully replaced!

        // Adding timestamp to the log message
        time_t now = time(NULL);
        struct tm *tm_info = localtime(&now);
        char timestamp[MAX_STRING_LENGTH];
        strftime(timestamp, sizeof(timestamp), "[%d-%m-%Y][%H:%M:%S]", tm_info);
        printf("%s Suspicious string replaced at %s\n", timestamp, filename);
        fprintf(log, "%s Suspicious string at %s successfully replaced!\n", timestamp, filename);

`struct tm *tm_info = localtime(&now);` digunakan untuk mengonversi nilai waktu ke dalam struktur tm yang menyimpan informasi waktu terperinci, seperti tahun, bulan, tanggal, jam, menit, dan detik yang kemudian disimpan dalam variabel tm_info

`char timestamp[MAX_STRING_LENGTH];` digunakan untuk menyimpan string format waktu yang akan dicetak

`strftime(timestamp, sizeof(timestamp), "[%d-%m-%Y][%H:%M:%S]", tm_info);` digunakan untuk mengambil informasi waktu dari tm_info dan mengonversinya menjadi string dengan format yang ditentukan, lalu disimpan ke dalam array timestamp

`printf("%s Suspicious string replaced at %s\n", timestamp, filename);` digunakan untuk mencetak informasi waktu dan nama file di mana string yang ingin diganti telah berhasil diganti

`fprintf(log, "%s Suspicious string at %s successfully replaced!\n", timestamp, filename);` digunakan untuk mencetak informasi yang tersimpan dalam file.log berisikan format sesuai yang diminta dalam soal

### Revisi

pada code di github belum terdapat fungsi untuk menjalankan daemon nya di background setiap 15 detik, sehingga saya memperbaiki kode program ini dengan menambahkan code ini setelah fungsi yang digunakan untuk membuka virus.log

            // Forking to daemon
            pid_t pid = fork();
            if (pid < 0) {
                perror("Error forking");
                exit(EXIT_FAILURE);
            }
            if (pid > 0) {
                // Exiting from parent process
                exit(EXIT_SUCCESS);
            }
        
            // Changing working directory
            if (chdir("/") < 0) {
                perror("Error changing directory");
                exit(EXIT_FAILURE);
            }
        
            // Closing standard file descriptors
            close(STDIN_FILENO);
            close(STDOUT_FILENO);
            close(STDERR_FILENO);
        
            // Daemon main loop
            while (1) {
                checkAndReplace(path, log);
                fflush(log);
                sleep(15); // Delay for 15 seconds
            }
        
            fclose(log);
        
            exit(EXIT_SUCCESS);
            }

code diatas bertujuan agar jika program dijalankan maka daemon akan terus berjalan tiap 15 detik di backgroun hal ini dapat dicek dengan membuat stringfile.txt kedua dan string pada file tersebut juga tetap akan ikut diganti seperti tampilan dibawah ini

gambar stringfile1.txt

### Hasil output

gambar

gambar

gambar

`gcc virus.c -o virus` digunakan untuk mengcompiler file virus.c dan menyimpannya ke dalam file virus

`./virus /home/farida/virus` digunakan untuk menjalankan file virus dan memeriksa string aneh yang terdapat di file pada direktori yang ditunjuk (virus)

`cat virus.log` digunakan untuk menampilkan format jika pengubahan string berhasil dijalankan dan disimpan dalam virus.log

`cat string1.txt` digunakan untuk menampilkan isi file yang telah diubah stringnya

`cat string2.txt` digunakan untuk menunjukkan bahwa program daemon terus berjalan tiap 15 detik

