# TKA-Praktikum-Modul1

## 1 Inisialisasi Sistem NERV

### 1. Asuka memulai dengan menyiapkan struktur proyek. Buat sebuah folder bernama eva-project dengan struktur sebagai berikut:
```
eva-project/
├── v1/
│   ├── Dockerfile
│   └── index.html
└── v2/
    ├── Dockerfile
    └── index.html
```

### 2. Masing-masing folder v1 dan v2 memiliki file index.html dengan isi yang berbeda. Untuk v1, isinya adalah:
```
EVANGELION STATUS SYSTEM
Unit-02 Pilot: Asuka Langley Soryu
Version: v1 - Lightweight
```
Sedangkan untuk v2, isinya adalah:
```
EVANGELION STATUS SYSTEM
Unit-02 Pilot: Asuka Langley Soryu
Version: v2 - Full System
```
### 3. Inilah perbedaan utama kedua versi. Buat Dockerfile untuk v1 menggunakan base image nginx:latest — versi ringan yang langsung siap pakai. Salin index.html ke direktori default nginx di dalam container dan expose port 80.
di Dockerfile:
```bash
# Materi Poin G - FROM: Menentukan base image.
# Menggunakan base image nginx:latest (ringan, sesuai permintaan Asuka).
FROM nginx:latest

# Materi Poin G - COPY: Menyalin file/folder ke dalam image.
# Salin index.html dari folder v1 (host) ke direktori default web server nginx di container.
COPY index.html /usr/share/nginx/html/index.html

# Materi Poin G - EXPOSE: Mendokumentasikan port yang akan digunakan container.
# Memberi tahu bahwa container ini akan listening di port 80.
EXPOSE 80
```

### 4. Untuk Dockerfile v2, Asuka ingin pendekatan yang berbeda. Gunakan base image ubuntu:latest, lalu lakukan instalasi nginx secara manual menggunakan perintah RUN di dalam Dockerfile. Setelah nginx terinstall, salin index.html ke direktori web server nginx, dan definisikan port 80. Pastikan nginx berjalan di foreground saat container dijalankan.
di Dockerfile:
```bash
# Materi Poin G - FROM: Base image Ubuntu (lebih berat, sesuai permintaan).
FROM ubuntu:latest

# Materi Poin G - RUN: Menjalankan perintah saat proses build image.
# 1. Update daftar paket (apt-get update).
# 2. Install Nginx secara otomatis (apt-get install -y nginx).
RUN apt-get update && apt-get install -y nginx

# Salin file index.html ke direktori web server Ubuntu (biasanya di /var/www/html/).
COPY index.html /var/www/html/index.html

# Ekspos port 80.
EXPOSE 80

# Materi Poin G - CMD: Perintah yang dijalankan saat container dimulai.
CMD ["nginx", "-g", "daemon off;"]  # Jalankan Nginx sebagai proses utama di container
                                     # - nginx       : perintah untuk menjalankan web server Nginx
                                     # - -g          : memberikan global directive untuk Nginx
                                     # - "daemon off;": agar Nginx berjalan di foreground (tidak di background)
                                     #                  penting supaya container tidak berhenti otomatis
```

### 5. Build kedua image dengan format nama berikut:
```
eva-web-[nomor_kelompok]:v1 dari folder v1  (contoh: eva-web-03:v1)
eva-web-[nomor_kelompok]:v2 dari folder v2  (contoh: eva-web-03:v2)
```
Tampilkan daftar image untuk memastikan kedua image berhasil dibuat.
Jalankan:
```bash
docker build -t eva-web-06:v1 ./v1
docker build -t eva-web-06:v2 ./v2
```
Cek daftar image:
```bash
docker images
```

### 6. Jalankan kedua container secara bersamaan dalam mode detached dengan ketentuan berikut:
```
Container dari v1 bernama eva-container-[nomor_kelompok]-v1, dapat diakses di http://localhost:8080
Container dari v2 bernama eva-container-[nomor_kelompok]-v2, dapat diakses di http://localhost:8081
```
Tampilkan daftar container yang sedang berjalan dan pastikan keduanya aktif serentak.
Jalankan:
```bash
docker run -d --name eva-container-06-v1 -p 8080:80 eva-web-06:v1

docker run -d --name eva-container-06-v2 -p 8081:80 eva-web-06:v2 
```
### 7. Buka browser dan akses
`http://localhost:8080`dan `http://localhost:8081` 
secara bergantian. Pastikan masing-masing menampilkan halaman yang sesuai dengan versinya (v1 dan v2). 

### 8. Asuka ingin membuktikan Docker Volume bekerja pada salah satu container. Tambahkan volume pada container v1: hentikan dan hapus container v1 yang sedang berjalan, lalu jalankan ulang dengan menghubungkan folder eva-project/v1 di host ke direktori web server nginx di dalam container. Port mapping dan mode detached tetap sama. Container v2 dibiarkan tetap berjalan selama proses ini.
```
docker stop eva-container-06-v1              
docker rm eva-container-06-v1

# Cek posisi sekarang
pwd

# Jalankan container (dari folder v1)
docker run -d --name eva-container-06-v1 -p 8080:80 -v "${PWD}:/usr/share/nginx/html" eva-web-06:v1
```

### 9. Untuk membuktikan volume bekerja, ubah isi file index.html pada folder v1 di host: ganti baris status menjadi 
`“<p>Status: Unit-02 Launch Successful</p>”`
Refresh browser di `http://localhost:8080` 
dan pastikan perubahan langsung terlihat tanpa rebuild image.
Ubah file `index.html`
```bash
EVANGELION STATUS SYSTEM
Unit-02 Pilot: Asuka Langley Soryu
Version: v1 - Lightweight
<p>Status: Unit-02 Launch Successful</p> # Tambah baris ini
```

### 10. Masuk ke dalam container v2 menggunakan perintah exec. Di dalam container, tampilkan isi file index.html dan verifikasi bahwa nginx terinstall dengan benar menggunakan perintah nginx -v
```bash
# Masuk ke dalam container dan jalankan shell interaktif:
docker exec -it eva-container-06-v2 /bin/bash

# Tampilkan isi file index.html
cat /var/www/html/index.html

# Verifikasi versi Nginx
nginx -v

exit
```
