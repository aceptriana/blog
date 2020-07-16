---
title: "Metode mounting partisi di Sistem Operasi Linux"
date: 2019-11-03T14:11:13+07:00
lastmod: 2020-06-16T14:11:13+07:00
draft: false
author: ""
authorLink: ""
description: "Beberapa cara melakukan mounting partisi di dalam sistem operasi linux."
license: ""
images: [mounting-partitions-linux/cover.jpg]

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

Dalam sistem operasi linux setiap partisi seperti ntfs, fat32, dan exfat harus dimount terlebih dahulu atau didefinisikan dalam filesystem table agar dapat diakses. Jika kamu pengguna [desktop environment](https://id.wikipedia.org/wiki/Lingkungan_desktop) contohnya GNOME yang menggunakan **gvfs** sebagai volume manajemen. Jadi saat pertama kali kamu mencoba mengakses partisi tersebut di files/nautilus (GNOME) maka gvfs akan melakukan mounting partisi tersebut. Itu karena pada dasarnya linux menggunakan extended partition.

---

## Metode Mounting
Ada 2 cara mounting partisi yaitu secara sementara dan permanen.

### Sementara
Pertama kita harus mengetahui lokasi partisi yang akan dimount menggunakan perintah **fdisk**.
```bash
sudo fdisk -l
```

Sehingga muncul seperti berikut.
```bash
Disk /dev/sda: 111.81 GiB, 120034123776 bytes, 234441648 sectors
Disk model: ADATA SU650     
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: 3FDC54A4-E17A-11E9-8D3E-CFA8BEF8E9F7

Device         Start       End   Sectors  Size Type
/dev/sda1       2048    206847    204800  100M EFI System
/dev/sda2     206848    239615     32768   16M Microsoft reserved
/dev/sda3     239616 111562751 111323136 53.1G Microsoft basic data
/dev/sda4  111562752 226131967 114569216 54.6G Linux filesystem
/dev/sda5  226131968 234441614   8309647    4G Linux swap


Disk /dev/sdb: 465.78 GiB, 500107862016 bytes, 976773168 sectors
Disk model: TOSHIBA MQ01ABD0
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 4096 bytes
I/O size (minimum/optimal): 4096 bytes / 4096 bytes
Disklabel type: gpt
Disk identifier: 424A89D0-B711-4B15-B9BA-5896AED39ABA

Device         Start       End   Sectors   Size Type
/dev/sdb1       2048 347582343 347580296 165.8G Microsoft basic data
/dev/sdb2  347582344 976773119 629190776   300G Microsoft basic data


Disk /dev/zram0: 676.97 MiB, 709832704 bytes, 173299 sectors
Units: sectors of 1 * 4096 = 4096 bytes
Sector size (logical/physical): 4096 bytes / 4096 bytes
I/O size (minimum/optimal): 4096 bytes / 4096 bytes 
```

{{< admonition note "Catatan" true >}}
Disini saya menggunakan 2 storage:
- SSD (Storage 1) untuk sistem operasi yaitu <u>/dev/sda</u>
- HDD (Storage 2) untuk dokumen/file pribadi yaitu <u>/dev/sdb</u>
        
Letak partisi saya terdapat pada HDD yaitu <u>/dev/sdb</u>.  
Sehingga lokasinya seperti berikut:  
- Partisi 1 Storage 2 : <u>/dev/sdb1</u>
- Partisi 2 Storage 2 : <u>/dev/sdb2</u>
{{< /admonition >}}
        
Setelah mengetahui lokasi partisi, buat direktori dimana partisi tersebut akan dimount.  
Misalnya:
```bash
sudo mkdir /media/localdisk1
sudo mkdir /media/localdisk2
```

Karena pada tutorial ini saya mounting partisi ntfs, maka perintah yang digunakan adalah **mount.ntfs-3g**.
```bash
sudo mount.ntfs-3g /dev/sdb1 /media/localdisk1
sudo mount.ntfs-3g /dev/sdb2 /media/localdisk2
```

Untuk unmount partisi ketikkan perintah **umount**.
```bash
sudo umount /dev/sdb1
sudo umount /dev/sdb2
```

### Permanen (Auto Mount)
Sebenarnya cara melakukan mounting secara otomatis menggunakan cara yang sama dengan mounting sementara, yaitu mengecek lokasi partisi dan membuat direktori. Perbedaannya adalah disini kita juga membuat baris baru pada file <u>/etc/fstab</u> seperti berikut.
```bash
sudo nano /etc/fstab
```

Lalu tambahkan baris dibawah ini menggunakan pola seperti berikut.
```cfg
#<device><location>          <linux type> <options> <dump> <pass>

/dev/sdb1 /media/localdisk1   ntfs-3g      defaults    0      0
/dev/sdb2 /media/localdisk2   ntfs-3g      defaults    0      0
```

Jika mendapati masalah saat booting (biasanya terjadi pertukaran blok pada beberapa distro linux), gunakan UUID sebagai gantinya <u>/dev/sdX</u>.
```bash
sudo blkid
```

Sehingga muncul seperti berikut.
```bash
/dev/sda1: UUID="F9DE-02AB" BLOCK_SIZE="512" TYPE="vfat" PARTUUID="71a07d93-cf3e-c548-8616-875016335794"
/dev/sda2: PARTLABEL="Microsoft reserved partition" PARTUUID="cc0aa241-fd0b-4a72-a0f7-c48093870afa"
/dev/sda3: LABEL="Windows 10" BLOCK_SIZE="512" UUID="BCBC3B75BC3B2972" TYPE="ntfs" PARTLABEL="Basic data partition" PARTUUID="beffbdf6-c308-4d5b-8560-5f42f125b6e1"
/dev/sda4: UUID="933d7893-4b69-4488-a657-3b0f7a223bf3" BLOCK_SIZE="4096" TYPE="ext4" PARTUUID="09a2a416-4fa0-b640-8380-5ba606902850"
/dev/sda5: UUID="64db9f99-a1d9-4e20-945f-e8c2be70df38" TYPE="swap" PARTUUID="4905d0d3-ebef-5e40-800e-908bc64852a5"
/dev/sdb1: LABEL="AllMedia" BLOCK_SIZE="512" UUID="0876114D0876114D" TYPE="ntfs" PARTLABEL="Microsoft basic data" PARTUUID="fb031833-4d26-40c1-92d4-cfe697ca0c1c"
/dev/sdb2: LABEL="AllData" BLOCK_SIZE="512" UUID="0E3C176C0E3C176C" TYPE="ntfs" PARTLABEL="Microsoft basic data" PARTUUID="0831fb44-3bb9-476b-9403-ff58d5c653e0"
/dev/zram0: LABEL="zram0" UUID="f45740ba-a959-48ff-8d75-c9ba1b1c2bcc" TYPE="swap"
```

Lalu sesuaikan dengan UUID yang ada seperti pada output perintah **blkid**.
```cfg
#<device>            <location>          <linux type> <options> <dump> <pass>

UUID=0876114D0876114D /media/localdisk1   ntfs-3g      defaults    0      0
UUID=0E3C176C0E3C176C /media/localdisk2   ntfs-3g      defaults    0      0
```

Terakhir, simpan konfigurasi dan reboot.

---

## Pesan Penulis
Bagaimana cukup mudah kan?  
Mengapa tadi menggunakan ntfs-3g dan bukan menggunakan ntfs? :thinking:

Sebagai tambahan referensi, berikut merupakan tabel tipe partisi yang digunakan dalam konfigurasi di atas.

| System Name     | English Name                          | Linux Type |
|-----------------|---------------------------------------|------------|
| W95 FAT32       | Microsoft FAT32                       | vfat       |
| NTFS Volume Set | Microsoft NTFS                        | ntfs       |
| NTFS Volume Set | Microsoft NTFS with read-write access | ntfs-3g    |
| Apple_HFS       | Apple HFS                             | hfsplus    |

Cukup itu yang dapat saya berikan.

Semoga bermanfaat bagi teman-teman semua.  
Terima Kasih. :grin:
