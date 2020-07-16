---
title: "Mengoptimalkan SSD di Sistem Operasi Linux"
date: 2020-05-23T18:21:49+07:00
lastmod: 2020-06-16T18:21:49+07:00
draft: false
author: ""
authorLink: ""
description: "Beberapa cara untuk mengoptimalkan kinerja SSD di sistem operasi linux."
license: ""
images: [optimize-ssd-linux/cover.jpg]

tags: ["Tweaks"]
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

Solid-state drive adalah perangkat penyimpanan solid-state yang menggunakan rangkaian sirkuit terintegrasi untuk menyimpan data secara terus-menerus, biasanya menggunakan memori flash dan berfungsi sebagai penyimpanan sekunder dalam hierarki penyimpanan komputer. Media penyimpanan SSD menggunakan komponen yang berbeda dengan HDD konvensional, oleh karena itu perawatan serta penggunaannya berbeda dengan HDD. Secara teori SSD memang jauh lebih cepat dibanding HDD, tetapi bukan berarti kamu bisa memperlakukannya sama dengan HDD.

Seiring berjalannya waktu, bertambahnya jumlah total penulisan data (Total Bytes Written) mempengaruhi umur SSD. Oleh karena itu, berikut ada beberapa penyesuaian yang harus dilakukan untuk mengoptimalkan solid-state drive pada sistem operasi linux agar tetap bekerja secara optimal dan panjang umur.

---

## Aktifkan TRIM dan kurangi WRITE
TRIM memastikan bahwa ketika sistem operasi ingin menulis di sektor yang sama, data lama akan dihapus sepenuhnya tanpa sampah. Ini berjalan pada minimal kernel linux 3.8 atau lebih baru, dan menggunakan ext4 atau filesystem yang mendukung [TRIM](https://wiki.archlinux.org/index.php/Solid_state_drive#TRIM) lainnya.

Pertama, edit file <u>/etc/fstab</u> menggunakan text editor, misalnya nano.
```bash
sudo nano /etc/fstab
```

Tambahkan flag **discard** ke partisi root dan flag **noatime** ke masing-masing partisi yang berada di SSD.
```cfg
# Static information about the filesystems.
# See fstab(5) for details.

# <file system>                             <dir>       <type>  <options>                                                                                                 <dump> <pass>
# /dev/sda5
UUID=933d7893-4b69-4488-a657-3b0f7a223bf3    /           ext4    noatime,discard,errors=remount-ro                                                                           0      1

# /dev/sda1
UUID=F9DE-02AB                               /boot/efi   vfat    rw,noatime,fmask=0022,dmask=0022,codepage=437,iocharset=iso8859-1,shortname=mixed,utf8,errors=remount-ro    0      2

# /dev/sda6
UUID=64db9f99-a1d9-4e20-945f-e8c2be70df38    none        swap    defaults,noatime                                                                                            0      0
```

Selanjutnya, tambahkan 4 baris berikut sehingga menggunakan RAM sebagai penyimpanan untuk file **temp** dan **log**.
```cfg
# Use RAM instead of SSD for temp and log files
tmpfs   /tmp            tmpfs   defaults,noatime,mode=1777      0 0
tmpfs   /var/log        tmpfs   defaults,noatime,mode=0755      0 0
tmpfs   /var/spool      tmpfs   defaults,noatime,mode=1777      0 0
tmpfs   /var/tmp        tmpfs   defaults,noatime,mode=1777      0 0
```

Simpan konfigurasi dan reboot.

Setelah reboot, jalankan TRIM secara berkala.
```bash
sudo fstrim -va
```

---

## Mengubah Nilai Swappiness
Parameter sysctl swappiness mewakili preferensi (penghindaran) kernel dari ruang swap. Swappiness dapat memiliki nilai antara 0 - 100, nilai standarnya adalah 60. Nilai yang rendah menyebabkan kernel menghindari penulisan swap, nilai yang lebih tinggi menyebabkan kernel mencoba menggunakan lebih banyak ruang swap. Menggunakan nilai rendah pada memori yang cukup diakui dapat meningkatkan daya tanggap (responsiveness) pada banyak sistem.

**:(fab fa-ubuntu fa-fw): Debian dan Ubuntu**
```bash
sudo nano /etc/sysctl.conf
```

**:(fab fa-linux fa-fw): Arch Linux**
```bash
sudo nano /etc/sysctl.d/99-sysctl.conf
```

Kemudian, tambahkan dua baris ini pada kernel parameter sysctl.
```cfg
vm.swappiness=1
vm.vfs_cache_pressure=50
```

Simpan konfigurasi dan reboot.

---

## Mengubah I/O Scheduler (Advanced Preference)
Linux memberi opsi untuk memilih I/O scheduler. Secara default biasanya menggunakan **mq-deadline** atau **bfq**.

I/O scheduler berupaya meningkatkan throughput dengan menyusun ulang akses permintaan ke dalam urutan linear berdasarkan alamat logis dari data dan mencoba untuk mengelompokkannya bersama-sama. Meskipun hal ini dapat meningkatkan throughput keseluruhan, hal itu dapat menyebabkan beberapa permintaan I/O menunggu terlalu lama sehingga menyebabkan masalah latensi. I/O scheduler berupaya menyeimbangkan kebutuhan akan throughput tinggi dan mencoba membagi secara adil permintaan I/O diantara proses.

Berbagai pendekatan telah diambil untuk berbagai I/O scheduler dan masing-masing memiliki kekuatan dan kelemahan masing-masing dan aturan umumnya adalah bahwa tidak ada I/O scheduler default yang sempurna untuk semua rentang tuntutan I/O yang mungkin dialami oleh suatu sistem.

{{< admonition info "Informasi dari Wiki" false >}}
**[:(fab fa-ubuntu fa-fw):](https://wiki.ubuntu.com/Kernel/Reference/IOSchedulers) Ubuntu Wiki**  
Best I/O scheduler to use
Different I/O requirements may benefit from changing from the Ubuntu distro default. A quick start guide to select a suitable I/O scheduler is below. The results are based on running 25 different synthetic I/O patterns generated using fio on ext4, xfs and btrfs with the various I/O schedulers using the 5.3 kernel.

**SSD or NVME drives**  
It is worth noting that there is little difference in throughput between the mq-deadline/none/bfq I/O schedulers when using fast multi-queue SSD configurations or fast NVME devices. In these cases it may be preferable to use the 'none' I/O scheduler to reduce CPU overhead.

**HDD**  
Avoid using the none/noop I/O schedulers for a HDD as sorting requests on block addresses reduce the seek time latencies and neither of these I/O schedulers support this feature. mq-deadline has been shown to be advantageous for the more demanding server related I/O, however, desktop users may like to experiment with bfq as has been shown to load some applications faster.

Of course, your use-case may differ, the above are just suggestions to start with based on some synthetic tests. You may find other choices with adjustments to the I/O scheduler tunables produce better results.

---

**[:(fab fa-linux fa-fw):](https://wiki.archlinux.org/index.php/Improving_performance#Input/output_schedulers) Arch Wiki**  
The best choice of scheduler depends on both the device and the exact nature of the workload. Also, the throughput in MB/s is not the only measure of performance: deadline or fairness deteriorate the overall throughput but may improve system responsiveness. [Benchmarking](https://wiki.archlinux.org/index.php/Benchmarking) may be useful to indicate each I/O scheduler performance.
{{</ admonition >}}

### Rekomendasi
Untuk perangkat komputer yang menggunakan SSD saja disarankan untuk mengganti I/O scheduler menjadi **none** sedangkan untuk perangkat komputer yang menggunakan SSD bersama HDD atau perangkat komputer yang hanya menggunakan HDD saja gunakan **mq-deadline** atau **bfq**.

{{< admonition info "Hasil Benchmark" false >}}
{{< image type="image/svg+xml" src="https://openbenchmarking.org/embed.php?i=2003265-NI-LINUX56IO44&sha=432aff7&p=2" caption="Benchmark Summary" height="auto" width="100%" >}}
{{< image type="image/svg+xml" src="https://openbenchmarking.org/embed.php?i=2003265-NI-LINUX56IO44&sha=8909f1b&p=2" caption="Benchmark Result" height="auto" width="100%" >}}

[:(fas fa-external-link-alt fa-fw): Sumber](https://www.phoronix.com/scan.php?page=article&item=linux-56-nvme&num=1)
{{</ admonition >}}

Pertama, cek I/O scheduler yang aktif sekarang ini.
```bash
cat /sys/block/sd*/queue/scheduler
```

Berikut contoh I/O scheduler yang aktif ditandai dengan kurung siku **[bfq]**.
```cfg
mq-deadline kyber [bfq] none
mq-deadline kyber [bfq] none
```

Kemudian, ubah I/O scheduler menjadi **none** pada langkah selanjutnya.

**:(fab fa-ubuntu fa-fw): Debian dan Ubuntu**  
Menggunakan kernel boot parameters.
```bash
sudo nano /etc/default/grub
```

Tambahkan **elevator=none** ke baris <u>GRUB_CMDLINE_LINUX_DEFAULT</u>.
```cfg
GRUB_CMDLINE_LINUX_DEFAULT="elevator=none quiet"
```

Update konfigurasi GRUB:
```bash
sudo update-grub
```

**:(fab fa-linux fa-fw): Arch Linux**  
Menggunakan udev rules.
```bash
sudo nano /etc/udev/rules.d/60-ioschedulers.rules
```

Tambahkan baris berikut.
```cfg
ACTION=="add|change", KERNEL=="sd[a-z]|mmcblk[0-9]*", ATTR{queue/rotational}=="0", ATTR{queue/scheduler}="none"
```

Simpan konfigurasi dan reboot.

---

## Pesan Penulis
Bagaimana cukup mudah kan?

Cukup itu yang dapat saya berikan.

Semoga bermanfaat bagi teman-teman semua.  
Terima Kasih. :grin:
