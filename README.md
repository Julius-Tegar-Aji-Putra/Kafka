# Apache Kafka Docker Demo - Streaming Data Realtime (Studi Kasus: Data KTP)

Proyek ini adalah demonstrasi penggunaan **Apache Kafka** menggunakan **Docker Container** dengan mode modern **KRaft** (tanpa Zookeeper). Proyek ini dibuat sebagai simulasi aliran data (*streaming*) sistem pendataan KTP secara *realtime*.

Demonstrasi ini menunjukkan konsep **Publisher-Subscriber**, di mana data yang diinputkan dari satu terminal (Producer/Kecamatan) akan langsung diterima oleh terminal lain (Consumer/Disdukcapil) dengan latensi yang sangat rendah.

## Anggota Kelompok KTP Kelas C

* **24060123120008** - Irvino Kent Setiawan
* **24060123120012** - Syafiq Abiyyu Taqi
* **24060123130088** - Muhamad Sahal Annabil
* **24060123130117** - Julius Tegar Aji Putra
* **24060123140204** - Mohammad Imron Rosyadi

## Struktur File

* `docker-compose.yml`: File konfigurasi utama untuk membangun infrastruktur Kafka Server (Broker & Controller) dalam container.
* `kafka-data`: (Folder otomatis) Volume docker yang menyimpan data log pesan agar tidak hilang saat container dimatikan.

## Prasyarat

* **Docker Desktop** (atau Docker Engine) sudah terinstal dan berjalan.
* **Terminal / PowerShell / Command Prompt** (Disarankan menggunakan terminal yang mendukung *Split Screen* seperti VS Code Terminal atau Windows Terminal).

## Cara Menjalankan

Ikuti langkah-langkah berikut untuk menjalankan simulasi streaming data KTP:

### 1. Jalankan Server Kafka
Buka terminal di direktori proyek, lalu jalankan perintah berikut untuk mengunduh image dan menyalakan server di background.

```powershell
docker-compose up -d
```

Pastikan status server sudah **Up** dengan mengecek menggunakan `docker-compose ps`.

### 2. Buat Topik Data (Wadah)
Kita perlu membuat "folder" khusus untuk menampung data KTP agar terisolasi.

```powershell
docker exec kafka /opt/kafka/bin/kafka-topics.sh --create --topic ktp --bootstrap-server localhost:9092
```

### 3. Jalankan Listener (Consumer)
Buka **Terminal Baru** (Split Kiri). Terminal ini bertindak sebagai **Server Pusat (Disdukcapil)** yang memantau data masuk.

```powershell
docker exec -it kafka /opt/kafka/bin/kafka-console-consumer.sh --topic ktp --bootstrap-server localhost:9092
```

Terminal ini akan diam (blinking cursor), menunggu data dikirim.

### 4. Jalankan Pengirim (Producer)
Buka **Terminal Baru** (Split Kanan). Terminal ini bertindak sebagai **Operator Input (Kecamatan)**.

```powershell
docker exec -it kafka /opt/kafka/bin/kafka-console-producer.sh --topic ktp --bootstrap-server localhost:9092
```

### 5. Simulasi Input Data
Ketik pesan simulasi di terminal Producer (Kanan), lalu tekan Enter:

```
Halo Dunia!!!
```

Lihat ke terminal Consumer (Kiri), pesan akan muncul secara instan!

---

## Penjelasan Konsep & Perintah

Berikut adalah penjelasan teknis mengenai komponen yang digunakan dalam demo ini:

### 1. KRaft Mode (Tanpa Zookeeper)
Pada `docker-compose.yml`, kita menggunakan konfigurasi:

```yaml
KAFKA_PROCESS_ROLES: broker,controller
```

Ini menandakan Kafka berjalan dalam mode mandiri (**Self-Managed Metadata**), menggantikan arsitektur lama yang membutuhkan Zookeeper. Ini membuat setup lebih ringan dan modern (sesuai standar Kafka 4.0+).

### 2. Topik (--topic ktp)
Perintah `kafka-topics.sh` digunakan untuk manajemen log.

* `--create`: Menginstruksikan Controller untuk mengalokasikan partisi log baru di disk.
* `ktp`: Nama unik saluran komunikasi. Producer dan Consumer harus sepakat menggunakan nama topik yang sama.

### 3. Producer (kafka-console-producer.sh)
Komponen ini bertanggung jawab untuk **Publishing**.

* Data yang kita ketik diubah menjadi **Byte Array**.
* Dikirim ke **Leader Partition** di dalam Broker Kafka.

### 4. Consumer (kafka-console-consumer.sh)
Komponen ini bertanggung jawab untuk **Subscribing**.

* `--bootstrap-server localhost:9092`: Memberitahu consumer alamat server Kafka.
* Consumer melakukan **polling** terus menerus untuk mengambil data terbaru yang belum dibaca.

---

## Catatan Troubleshooting

**Error Topic already exists**: Jika topik `ktp` sudah ada dan ingin dihapus bersih, gunakan perintah:

```powershell
docker exec kafka /opt/kafka/bin/kafka-topics.sh --delete --topic ktp --bootstrap-server localhost:9092
```

**Note**: Jika di Windows penghapusan gagal, restart container dengan `docker-compose restart`.

**Error Connection Refused**: Jika muncul saat baru menyalakan docker, tunggu sekitar 15-30 detik. Kafka membutuhkan waktu untuk bootstrapping sebelum siap menerima koneksi.

**Reset Total**: Jika ingin mengulang dari nol (menghapus semua data dan topik), jalankan:

```powershell
docker-compose down -v
docker-compose up -d
```
