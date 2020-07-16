---
title: "Ncmpcpp, pemutar musik CLI yang customizable"
date: 2019-11-02T23:36:51+07:00
lastmod: 2020-07-13T23:36:51+07:00
draft: false
author: ""
authorLink: ""
description: "Instalasi dan konfigurasi ncmpcpp beserta servernya, mpd."
license: ""
images: [mpd-ncmpcpp/cover.jpg]

tags: ["Applications"]
categories: ["GNU/Linux"]
featuredImage: "cover.jpg"
featuredImagePreview: "cover.jpg"

hiddenFromHomePage: false
hiddenFromSearch: false
twemoji: true
lightgallery: true
ruby: true
fraction: true
fontawesome: true
linkToMarkdown: true
rssFullText: true

toc:
  enable: true
  auto: true
code:
  copy: true
math:
  enable: true
share:
  enable: true
comment:
  enable: true
---

Ncmpcpp atau ncurses music player client plus plus adalah aplikasi client untuk mpd (music player daemon) yang dibangun dengan library ncurses, digunakan untuk manajemen musik dan dapat dikonfigurasi sesuai kebutuhan pengguna. Ncmpcpp menyediakan fitur-fitur baru yang sangat berguna dari sebelumnya yaitu ncmpc seperti library searches, extended song format, items filtering, the ability to sort playlists, dan local filesystem browser.

Untuk menggunakannya, ncmpcpp dan mpd harus terinstal satu paket karena mereka bekerja bersama dalam hubungan antara client-server.

---

## Instalasi Paket
**:(fab fa-ubuntu fa-fw): Debian dan Ubuntu**
```bash
sudo apt install mpd ncmpcpp
```

**:(fab fa-linux fa-fw): Arch Linux**
```bash
sudo pacman -S mpd ncmpcpp
```

**:(fab fa-linux fa-fw): Distro GNU/Linux lain [:(fas fa-external-link-alt fa-fw):](https://mpd.fandom.com/wiki/Install)**

---

## Konfigurasi
### Mpd
Pertama, buat direktori default untuk mpd dan playlists.
```bash
mkdir -p ~/.mpd/playlists
```

Selanjutnya, konfigurasi **mpd.conf** didalam direktori <u>~/.mpd</u>.
```bash
nano ~/.mpd/mpd.conf
```

Berikut konfigurasi default mpd.
```cfg
bind_to_address     "127.0.0.1"
port                "6600"

music_directory     "~/Music/"
playlist_directory  "~/.mpd/playlists"
db_file             "~/.mpd/mpd.db"
log_file            "~/.mpd/mpd.log"
pid_file            "~/.mpd/mpd.pid"
state_file          "~/.mpd/mpdstate"

audio_output {
    type            "pulse"
    name            "Pulse Audio"
}

audio_output {
    type            "fifo"
    name            "FIFO Visualizer"
    path            "/tmp/mpd.fifo"
    format          "44100:16:2"
}
```

### Ncmpcpp
Sekarang, buat direktori default untuk ncmpcpp.
```bash
mkdir ~/.ncmpcpp
```
{{< admonition bug "Permasalahan" true >}}
Dikarenakan terjadi konflik antara konfigurasi ncmpcpp pada ssg hugo dengan hosting di github pages, maka
unduh konfigurasi ncmpcpp saya dan letakkan di <u>~/.ncmpcpp</u>.

> [:(fas fa-download fa-fw): Download](config)
{{< /admonition >}}


---

## Penggunaan
Untuk menggunakan ncmpcpp, mpd harus dijalankan terlebih dulu.
```bash
mpd
```

{{< admonition note "Catatan" false >}}
Di debian-based distro, setelah menginstal mpd akan dijalankan sebagai layanan sistem secara otomatis. Untuk menjalankan mpd sebagai user, nonaktifkan service mpd saat startup.
```bash
sudo systemctl disable mpd.service
sudo systemctl disable mpd.socket
sudo systemctl stop mpd.service
sudo systemctl stop mpd.socket
```

Saya selalu menjalankan mpd sebagai user yang saya letakkan pada **autostart** masing-masing desktop environment atau window manager, misalnya Openbox yang terletak pada <u>~/.config/openbox/autostart</u>
```cfg
#
# These things are run when an Openbox X Session is started.
# You may place a similar script in $HOME/.config/openbox/autostart
# to run user-specific things.
#
# ---
# Apps & Others (ex:mpd)
mpd
```
{{</ admonition >}}

Cek apakah mpd sudah berjalan pada port 6600.
```bash
netstat -tulpen | grep 'mpd'
```

Kemudian jalankan ncmpcpp.
```bash
ncmpcpp
```

Setelah muncul tampilan ncmpcpp secara default masuk ke menu playlists yang kosong. Tekan ( **u** ) untuk mengupdate database, maka secara otomatis mpd akan mencari musik berada di direktori <u>~/Music</u>.

Untuk menambahkan musik ke playlist tekan ( **2** ) dan ( **enter** ) pada musik yang ingin diputar. Jika ingin menambahkan semua musik yang berada dalam folder tekan ( **v** ) untuk memilih semua lalu tekan ( **a** ) dan masukkan ke current playlist. Kembali ke playlist dengan menekan ( **1** ). Untuk menuju ke menu visualizer ncmpcpp, tekan ( **8** ). Play/pause tekan ( **p** ).

Untuk menghentikan mpd, ketikkan perintah.
```bash
killall mpd
```

---

## Default Keybindings
Selengkapnya, [disini](https://pkgbuild.com/~jelle/ncmpcpp/).

---

## Pesan Penulis
Bagaimana cukup mudah kan?  
Sebenarnya banyak aplikasi client yang digunakan untuk MPD, daftarnya bisa dilihat [disini](https://www.musicpd.org/clients/). :wink:

Cukup itu yang dapat saya berikan.

Semoga bermanfaat bagi teman-teman semua.  
Terima Kasih. :grin:

