## Soal 2

Masih dengan Ini Karya Kita, sang CEO ingin melakukan tes keamanan pada folder sensitif Ini Karya Kita. Karena Teknologi Informasi merupakan departemen dengan salah satu fokus di Cyber Security, maka dia kembali meminta bantuan mahasiswa Teknologi Informasi angkatan 2023 untuk menguji dan mengatur keamanan pada folder sensitif tersebut. Untuk mendapatkan folder sensitif itu, mahasiswa IT 23 harus kembali mengunjungi website Ini Karya Kita pada www.inikaryakita.id/schedule . Silahkan isi semua formnya, tapi pada form subject isi dengan nama kelompok_SISOP24 , ex: IT01_SISOP24 . Lalu untuk form Masukkan Pesanmu, ketik “Mau Foldernya” . Tunggu hingga 1x24 jam, maka folder sensitif tersebut akan dikirimkan melalui email kalian. Apabila folder tidak dikirimkan ke email kalian, maka hubungi sang CEO untuk meminta bantuan.

- Pada folder "pesan" Adfi ingin meningkatkan kemampuan sistemnya dalam mengelola berkas-berkas teks dengan menggunakan fuse.

      - Jika sebuah file memiliki prefix "base64," maka sistem akan langsung mendekode isi file tersebut dengan algoritma Base64.

      - Jika sebuah file memiliki prefix "rot13," maka isi file tersebut akan langsung di-decode dengan algoritma ROT13.

      - Jika sebuah file memiliki prefix "hex," maka isi file tersebut akan langsung di-decode dari representasi heksadesimalnya.

      - Jika sebuah file memiliki prefix "rev," maka isi file tersebut akan langsung di-decode dengan cara membalikkan teksnya.

Contoh:

File yang belum didecode/ dienkripsi rot_13:

![image](https://github.com/Faridaqurr/lapres/assets/150933246/e3b669db-b875-49b9-85de-bddc4c46ab7f)

File yang sudah didecode/ dienkripsi rot_13:

![image](https://github.com/Faridaqurr/lapres/assets/150933246/a8848789-c08e-4c5a-a74b-cb78ec1829c0)


- Pada folder “rahasia-berkas”, Adfi dan timnya memutuskan untuk menerapkan kebijakan khusus. Mereka ingin memastikan bahwa folder dengan prefix "rahasia" tidak dapat diakses tanpa izin khusus.

   - Jika seseorang ingin mengakses folder dan file pada “rahasia”, mereka harus memasukkan sebuah password terlebih dahulu (password bebas)

- Setiap proses yang dilakukan akan tercatat pada logs-fuse.log dengan format :

  [SUCCESS/FAILED]::dd/mm/yyyy-hh:mm:ss::[tag]::[information]

  Ex:

  [SUCCESS]::01/11/2023-10:43:43::[moveFile]::[File moved successfully]

## Jawab

### Pesan

Pada soal ini, diminta untuk mendekrip semua file sesuai tipe nya pada folder sensitif/pesan

      static void rot13(char *str) {
          for (char *c = str; *c; ++c) {
              if ((*c >= 'a' && *c <= 'z') || (*c >= 'A' && *c <= 'Z')) {
                  if ((*c >= 'a' && *c <= 'm') || (*c >= 'A' && *c <= 'M'))
                      *c += 13;
                  else
                      *c -= 13;
              }
          }
      }

`static void rot13(char *str)` untuk deklarasi fungsi dari ROT13

`if ((*c >= 'a' && *c <= 'z')...` untuk memeriksa apakah ada huruf a-z kecil maupun kapital

` if ((*c >= 'a' && *c <= 'm')...` untuk proses penggeseran huruf sesuai rentangnya yang kemudian ditambahkan 13 ke nilai ASCII-nya
      
      static void hex_decode(const char *input, char *output) {
          size_t len = strlen(input) / 2;
          for (size_t i = 0; i < len; i++) {
              sscanf(input + 2 * i, "%2hhx", &output[i]);
          }
          output[len] = '\0';
      }

`static void hex_decode(const char *input, char *output)` untuk deklarasi fungsi heksadecimal

`size_t len = strlen(input) / 2;` untuk menghitung panjang outputnya

`for (size_t i = 0;...` untuk proses perulangan konversi heksadesimal ke biner
      
      static void reverse_string(char *str) {
          int len = strlen(str);
          for (int i = 0; i < len / 2; i++) {
              char temp = str[i];
              str[i] = str[len - i - 1];
              str[len - i - 1] = temp;
          }
      }

`static void reverse_string(char *str)` untuk deklarasi fungsi reverse

`int len = strlen(str);` untuk menghitung panjang stringnya

`for (int i = 0; i < ...` untuk perulangan ketika membalikkan/reverse kata
      
      static void base64_decode(const char *input, char *output) {
          BIO *bio, *b64;
          int input_len = strlen(input);
          int output_len = (input_len * 3) / 4 + 1;
          char *buffer = malloc(output_len);
          if (!buffer) return;
          
          bio = BIO_new_mem_buf(input, input_len);
          b64 = BIO_new(BIO_f_base64());
          bio = BIO_push(b64, bio);
          BIO_set_flags(bio, BIO_FLAGS_BASE64_NO_NL);
          int decoded_size = BIO_read(bio, buffer, input_len);
          buffer[decoded_size] = '\0';
          
          BIO_free_all(bio);
          strcpy(output, buffer);
          free(buffer);
      }

`static void base64_decode(const char *input, char *output)` untuk deklarasi fungsi base64

`int input_len = strlen(input);` untuk hitung panjang input

`int output_len = (input_len * 3) / 4 + 1;` untuk hitung panjang output

`char *buffer = malloc(output_len);` untuk alokasi memori buffer-nya

`int decoded_size = BIO_read(bio, buffer, input_len);` untuk membaca dan dekode base64-nya

`BIO_free_all(bio);` untuk membersihkan BIO dan menyalin ke outputnya

      static void decrypt_message(const char *path, char *content) {
          if (strstr(path, "rev") != NULL) {
              reverse_string(content);
          } else if (strstr(path, "hex") != NULL) {
              char *decoded = malloc(strlen(content) / 2 + 1);
              if (decoded) {
                  hex_decode(content, decoded);
                  strcpy(content, decoded);
                  free(decoded);
              }
          } else if (strstr(path, "base64") != NULL) {
              char *decoded = malloc(strlen(content) * 3 / 4 + 1);
              if (decoded) {
                  base64_decode(content, decoded);
                  strcpy(content, decoded);
                  free(decoded);
              }
          } else if (strstr(path, "rot13") != NULL) {
              rot13(content);
          }
          log_message("decryptMessage", path, 1);
      }

`static void decrypt_message(const char *path, char *content)` untuk deklarasi fungsi dekripnya

`if (strstr(path, "rev")...` untuk periksa fungsi reverse

`else if (strstr(path, "hex")...` untuk periksa fungsi heksadesimal

`else if (strstr(path, "base64")...` untuk periksa fungsi base64

`else if (strstr(path, "rot13")...` untuk periksa fungsi rot13

`log_message("decryptMessage", path, 1);` untuk fungsi log massage

      static int xmp_getattr(const char *path, struct stat *stbuf) {
          int res;
          char fpath[PATH_MAX];
      
          snprintf(fpath, sizeof(fpath), "%s%s", SENSITIVE_DIR, path);
          res = lstat(fpath, stbuf);
      
          if (res == -1)
              return -errno;
      
          return 0;
      }

`static int xmp_getattr(const char *path, struct stat *stbuf)` untuk deklarasi fungsi 
      
      static int xmp_readdir(const char *path, void *buf, fuse_fill_dir_t filler, off_t offset, struct fuse_file_info *fi) {
          DIR *dp;
          struct dirent *de;
          char fpath[PATH_MAX];
      
          (void) offset;
          (void) fi;
      
          snprintf(fpath, sizeof(fpath), "%s%s", SENSITIVE_DIR, path);
          dp = opendir(fpath);
          if (dp == NULL)
              return -errno;
      
          while ((de = readdir(dp)) != NULL) {
              struct stat st;
              memset(&st, 0, sizeof(st));
              st.st_ino = de->d_ino;
              st.st_mode = de->d_type << 12;
              if (filler(buf, de->d_name, &st, 0))
                  break;
          }
      
          closedir(dp);
          return 0;
      }
