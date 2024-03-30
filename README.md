## soal 2a

Pada soal ini diperintahkan untuk membuat dua program yaitu register.sh dan login.sh

![Gambar Contoh](https://github.com/Faridaqurr/lapres/blob/main/9ced4635ed159aa08306a03c8575e9a5.jpg)

`mkdir praktikum1` untuk membuat folder praktikum

`cd praktikum` untuk pindah ke directory praktikum sebelum membuat file

`nano register.sh` untuk membuat konfigurasi program register.sh

`bash register.sh` untuk menjalankan konfigurasi register.sh

`nano login.sh` untuk membuat konfigurasi program login.sh

`bash login.sh` untuk menjalankan konfigurasi login.sh

## soal 2b

Ketika penyelesaian soal, pada program register harus menampilkan email, username, pernyataan keamanan dan jawaban, serta password

    read -p "Email		  : " email
    read -p "Username	  : " username
    read -p "Security Question : " security_question
    read -p "Security Answer   : " security_answer
    read -sp "Password	  : " password
    echo
    
## soal 2c

Saat memasukkan data, program dibuat agar ketika terdeteksi kata "admin" pada email maka secara otomatis user tersebut akan dikategorikan sebagai admin

      	if [[ $user_type == "admin" ]]; then
      		echo "[ $(date +'%d%m%Y %H:%M:%S') ] [REGISTER SUCCESS] Admin $username berhasil registrasi." >> auth.log
      		echo "Admin $username berhasil regitrasi"
      	else
      		echo "[ $(date +'%d%m%Y %H:%M:%S') ] [REGISTER SUCCESS] User $username berhasil regitrasi." >> auth.log
      		echo "User $username berhasil registrasi"
      	fi

`[[ $user_type == "admin" ]]` untuk mendeteksi kata admin, sehingga akan otomatis user_type nya menjadi admin 

`>> auth.log` untuk memasukkan spesifikasi data yang masuk untuk ditampung di auth.log

## soal 2d

Dalam pembuatan password juga diminta untuk memenuhi beberapa kriteria password seperti jumlah password minimal 8 dan minimal mengandung 1 huruf kapital dan juga angka, serta untuk enkripsinya menggunakan base64

=> fungsi untuk enkripsi pasword menggunakan base64
   
    encrypt_password()
    {
    	echo -n "$1" | base64
    }

`-n "$1"` untuk menampilkan teks (password) tanpa menambahkan newline

=> fungsi untuk ketentuan password
       
    if [[ ${#password} -lt  8 || !("$password" =~ [[:lower:]]) || !("$password" =~ [[:upper:]]) || !("$password" =~ [0-9]) ]]; then
        echo -e  "!Password harus berjumlah 8 karakter atau lebih!\n!Tambahkan minimal 1 huruf kapital dan angka!"
        read -sp "Password: " password
        echo
    else
    		break
    fi

`-lt  8` untuk cek jumlah password kurang dari 8 atau tidak

`[[:lower:]]` untuk cek apakah mengandung huruf kecil

`[[:upper:]]` untuk cek apakah mengandung huruf kapital

`[0-9]` untuk cek apakah mengandung angka

## soal 2e

Semua data yang telah diinputkan akan disimpan dalam file users.txt

    echo "$email:$usename:$security_question:$security_answer:$encrypted_password:$user_type" >> users.txt

`>> users.txt` untuk menyimpan semua data yang diinputkan ke users.txt

## soal 2f

Setelah melakukan register, terdapat juga program untuk login yang cukup menampilkan email dan password

    echo "~USER LOGIN~"
    echo "1. Login"
    echo "2. Forgot Password"
    read -p "Choose an option: " option

## soal 2g

Dalam program login, harus menampilkan opsi lupa password dan ketika memilih opsi tersebut akan ditampilkan pertanyaan keamanan dan pengguna harus memasukkan jawaban dari pertanyaan keamanan tersebut agar password ditampilkan

    #case untuk forget password
    2) 
                read -p "Enter your email: " email
                reset_password "$email"
                ;;
        reset_password() 
        {
          local email=$1

`email` untuk menyimpan inputan ke alamat email

`reset_password $email` untuk memanggil fungsi alamat reset password dengan email sebagai argumen tambahannya

    #fungsi untuk reset password
    if check_registered_email "$email"; then
        security_question=$(grep "^$email" users.txt | cut -d':' -f3)
        echo "Security question: $security_question"
        read -p "Enter your answer: " security_answer

        stored_security_answer=$(grep "^$email" users.txt | cut -d':' -f4)

        if [ "$security_answer" == "$stored_security_answer" ]; then
            encrypted_password=$(grep "^$email" users.txt | cut -d':' -f5)
            decrypted_password=$(decode_base64 "$encrypted_password")
            echo "Your password is: $decrypted_password"
            echo "[ $(date +'%d/%m/%Y %H:%M:%S') ] [PASSWORD RESET] Password reset requested for user with email $email." >> auth.log
        else
            echo "Incorrect security answer. Password reset failed."
            echo "[ $(date +'%d/%m/%Y %H:%M:%S') ] [PASSWORD RESET FAILED] ERROR: Incorrect security answer for user with email $email." >> auth.log
        fi
    else
        echo "Email not found. Please register first."
        echo "[ $(date +'%d/%m/%Y %H:%M:%S') ] [PASSWORD RESET FAILED] ERROR: User with email $email not found." >> auth.log
    fi
    }

`grep` untuk mengambil alamat email di dalam users.txt

`cut -d':' -f3` untuk memilih kolom ke 3 dari file users.txt yang menunjukkan data security question

`>> auth.log` untuk memasukkan spesifikasi data yang masuk untuk ditampung di auth.log

## soal 2h

Jika login berhasil maka akan di tampilkan pesan sukse, namun jika terdeteksi seorang admin yang login maka akan muncul output tambahan yaitu admin menu yang berisikan add user, edit user, delete user, dan logout untuk memudahkan pkerjaan admin

     # case untuk login admin
     1)
            read -p "Email: " email
            read -sp "Password: " password
            echo
            if authenticate_user "$email" "$password"; then
                if [[ $email == *admin* ]]; then
                    echo "Welcome, Admin!"
                    echo "Admin menu:"
                    echo "1. Add user"
                    echo "2. Edit user"
                    echo "3. Delete user"
                    echo "4. Logout"
                    read -p "Enter your choice: " choice
    
                    case $choice in
                        1)
                            add_user
                            ;;
                        2)
                            read -p "Enter email of the user to edit: " edit_email
                            edit_user "$edit_email"
                            ;;
                        3)
                            read -p "Enter email of the user to delete: " delete_email
                            delete_user "$delete_email"
                            ;;
                        4)
                            echo "Logout successful"
                            ;;
                        *)
                            echo "Invalid choice"
                            ;;
                    esac
                else
                    echo "Login successful."
                fi
            fi
            ;;

`*admin*` untuk konfirmasi masukan yang terdapat kata adminnya

`edit_email` untuk memanggil fungsi edit_email

`delete_email` untuk memanggil fungsi delete_email

## soal 2i

Ketika memilih menu admin edit dan delete user, maka akan memunculkan tampilan untuk inputan email atau username

    # Fungsi untuk mengedit
    edit_user() 
    {
        local email=$1
        read -p "Enter new username: " new_username
        read -p "Enter new security question: " new_security_question
        read -p "Enter new security answer: " new_security_answer
        read -sp "Enter new password: " new_password
        echo
    
        # Update user dan dimasukkan ke  users.txt
        sed -i "s/^$email:.*/$email:$new_username:$new_security_question:$new_security_answer:$(echo -n $new_password | base64):user/g" users.txt
    
        echo "User $email updated successfully."
        echo "[ $(date +'%d/%m/%Y %H:%M:%S') ] [USER EDIT] User with email $email edited successfully." >> auth.log
    }

`local email=$1` untuk memanggil fungsi local email pada register.sh

`>> auth.log` untuk memasukkan spesifikasi data yang masuk untuk ditampung di auth.log

    # Fungsi untuk menghapus user
    delete_user() 
    {
        local email=$1
    
    #hapus user dari users.txt
        sed -i "/^$email:/d" users.txt
        echo "User $email deleted successfully"
        echo "[ $(date +'%d/%m/%Y %H:%M:%S') ] [USER DELETE] User with email $email deleted successfully" >> auth.log
    }

`local email=$1` untuk memanggil fungsi local email pada register.sh

`-i` untuk melakukan perubahan (delete) langsung

`"/^$email:/d"` untuk menghapus seluruh data dari alamat email yang ditentukan

`users.txt` untuk melakukan perubahan (delete) langsung di dalam users.txt

## soal 2j

Seluruh data log yang diinputkan akan disimpan ke dalam auth.log baik login maupun register, dengan format [DATE][TYPE][MASSAGE]

![Gambar Contoh](https://github.com/Faridaqurr/lapres/blob/main/9ced4635ed159aa08306a03c8575e9a5.jpg)



