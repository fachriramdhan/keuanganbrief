
# Rancangan Menu Laporan Keuangan - Jemaatplus

Versi refresh PDF untuk dibagikan ke tim developer. Dokumen ini merangkum menu laporan keuangan, struktur tabel, nama kolom, tipe kolom, dan relasi antar entitas agar implementasi backend dan laporan bisa konsisten dengan web Jemaatplus yang sudah berjalan.

**Dibuat ulang:** 10-04-2026 23:04

---

## 1. Struktur Menu Laporan Keuangan
Di sidebar **Data Keuangan**, sub-menu laporan yang disarankan adalah sebagai berikut:
* **Laporan Persembahan Jemaat** - daftar persembahan per KK dalam satu periode.
* **Laporan Penerimaan dan Pengeluaran** - laporan mingguan final per periode.
* **Laporan Kas** - posisi saldo tiap kas utama.
* **Laporan Dana Titipan** - mutasi dan saldo per dompet titipan.
* **Rekap Persepuluhan Bulanan** - jumlah KK aktif memberi per bulan.
* **Rekap Sumbangan Duka Bulanan** - kewajiban dan realisasi duka per KK.
* **Daftar Tunggakan Jemaat** - filter belum bayar 1, 3, 6 bulan atau custom.
* **Riwayat Pembelian SBU** - histori pembelian SBU per KK/periode.

---

## 2. Entitas / Tabel Inti yang Dibutuhkan
Agar semua menu laporan saling terhubung dengan rapi, berikut tabel inti yang disarankan.

### 2.1 keluarga
| Nama Kolom | Tipe | Keterangan |
| :--- | :--- | :--- |
| id_keluarga | bigint unsigned | Primary key |
| no_kk | varchar(30) | Nomor KK |
| nama_kk | varchar(150) | Nama keluarga / kepala keluarga |
| sektor_id | bigint unsigned | FK ke sektor.id |
| alamat | text | Opsional |
| jumlah_jiwa | int | Total jiwa aktif dalam KK |
| status | enum('aktif', 'tidak_aktif') | Status keluarga |
| created_by | bigint unsigned | FK ke users.id |
| updated_by | bigint unsigned | FK ke users.id |
| created_at / updated_at | timestamp | Standar Laravel |

* **Relasi:** `keluarga.sektor_id -> sektor.id`
* **Relasi:** `keluarga.created_by / updated_by -> users.id`

### 2.2 keluarga_jemaat
| Nama Kolom | Tipe | Keterangan |
| :--- | :--- | :--- |
| id | bigint unsigned | Primary key |
| id_keluarga | bigint unsigned | FK ke keluarga.id_keluarga |
| id_jemaat | bigint unsigned | FK ke jemaat.id_jemaat |
| is_kepala_keluarga | tinyint(1) | 1 = kepala keluarga |
| status_anggota | enum('aktif', 'tidak_aktif') | Status anggota di KK |
| created_at / updated_at | timestamp | Standar Laravel |

* **Penting:** model Jemaat di sistem existing memakai PK `id_jemaat`, bukan `id`.

### 2.3 kas
| Nama Kolom | Tipe | Keterangan |
| :--- | :--- | :--- |
| id_kas | bigint unsigned | Primary key |
| kode_kas | varchar(50) | Contoh: dana_pelayanan |
| nama_kas | varchar(100) | Nama kas |
| jenis_kas | enum('utama', 'titipan') | Jenis kas |
| is_aktif | tinyint(1) | Aktif / tidak |
| created_at / updated_at | timestamp | Standar Laravel |

* **Isi default:** Dana Pelayanan, Sumbangan Duka, PEG, Cadangan Dana Pensiun, Dana Titipan.

### 2.4 kas_titipan
| Nama Kolom | Tipe | Keterangan |
| :--- | :--- | :--- |
| id_kas_titipan | bigint unsigned | Primary key |
| id_kas | bigint unsigned | FK ke kas.id_kas |
| kode_titipan | varchar(50) | Contoh: sektor_1, pkp_sektor_1 |
| nama_titipan | varchar(100) | Nama dompet titipan |
| sektor_id | bigint unsigned nullable | FK ke sektor.id |
| pelkat_id | bigint unsigned nullable | FK ke pelkat.id |
| is_aktif | tinyint(1) | Aktif / tidak |

### 2.5 keuangan_transaksi
| Nama Kolom | Tipe | Keterangan |
| :--- | :--- | :--- |
| id_transaksi | bigint unsigned | Primary key |
| tanggal_transaksi | date | Tanggal diterima / dikeluarkan |
| tanggal_input | datetime | Kapan diinput |
| jenis_transaksi | enum('penerimaan', 'pengeluaran', 'transfer') | Jenis transaksi |
| sumber_transaksi | enum('persembahan_jemaat', 'kas_umum', 'dana_titipan', 'transfer_kas') | Asal dana |
| nomor_bukti | varchar(50) nullable | Opsional |
| keterangan | text nullable | Catatan umum |
| status_posting | enum('draft', 'posted', 'void') | Status posting |
| periode_mulai | date nullable | Untuk laporan mingguan |
| periode_selesai | date nullable | Untuk laporan mingguan |
| created_by / updated_by | bigint unsigned | FK ke users.id |

### 2.6 keuangan_transaksi_detail
| Nama Kolom | Tipe | Keterangan |
| :--- | :--- | :--- |
| id_detail | bigint unsigned | Primary key |
| id_transaksi | bigint unsigned | FK ke keuangan_transaksi.id_transaksi |
| id_kas | bigint unsigned | FK ke kas.id_kas |
| id_kas_titipan | bigint unsigned nullable | FK ke kas_titipan.id_kas_titipan |
| arah_dana | enum('debit', 'kredit') | Arah pencatatan |
| nominal | decimal(18,2) | Nominal |
| kategori | varchar(100) | Persepuluhan / duka / syukur / SBU / operasional |
| subkategori | varchar(100) nullable | Detail tambahan |
| keterangan | text nullable | Catatan detail |

### 2.7 persembahan_jemaat
| Nama Kolom | Tipe | Keterangan |
| :--- | :--- | :--- |
| id_persembahan | bigint unsigned | Primary key |
| id_transaksi | bigint unsigned | FK ke keuangan_transaksi.id_transaksi |
| id_keluarga | bigint unsigned | FK ke keluarga.id_keluarga |
| nama_pemberi | varchar(150) nullable | Nama individu pemberi |
| diterima_melalui | varchar(100) | Contoh: Ibadah Minggu 07.00 |
| tanggal_terima | date | Tanggal terima persembahan |
| catatan | text nullable | Catatan umum |
| created_by / updated_by | bigint unsigned | FK ke users.id |

### 2.8 persembahan_jemaat_detail
| Nama Kolom | Tipe | Keterangan |
| :--- | :--- | :--- |
| id_persembahan_detail | bigint unsigned | Primary key |
| id_persembahan | bigint unsigned | FK ke persembahan_jemaat.id_persembahan |
| jenis_persembahan | enum('persepuluhan', 'sumbangan_duka', 'sbu', 'syukur') | Jenis persembahan |
| id_kas | bigint unsigned | FK ke kas.id_kas |
| nominal | decimal(18,2) | Nominal transaksi |
| jumlah_eksemplar | int nullable | Khusus SBU |
| harga_satuan | decimal(18,2) nullable | Khusus SBU |
| jumlah_jiwa | int nullable | Khusus duka |
| tarif_per_jiwa | decimal(18,2) nullable | Khusus duka |
| nominal_tagihan | decimal(18,2) nullable | Khusus duka |
| status_pembayaran | enum('lunas', 'kurang_bayar', 'lebih_bayar') nullable | Khusus duka |
| sisa_kelebihan | decimal(18,2) nullable | Khusus duka |
| keterangan / catatan | varchar + text nullable | Catatan: ulang tahun |

### 2.9 persembahan_periode_alokasi
| Nama Kolom | Tipe | Keterangan |
| :--- | :--- | :--- |
| id_alokasi | bigint unsigned | Primary key |
| id_persembahan_detail | bigint unsigned | FK ke persembahan_jemaat_detail.id_persembahan_detail |
| bulan | tinyint | 1-12 |
| tahun | smallint | Contoh 2026 |
| nominal_alokasi | decimal(18,2) nullable | Opsional |
| qty_alokasi | int nullable | Opsional, khusus SBU |

### 2.10 laporan_keuangan_periode
| Nama Kolom | Tipe | Keterangan |
| :--- | :--- | :--- |
| id_periode_laporan | bigint unsigned | Primary key |
| nama_periode | varchar(100) | Contoh: Periode 13 s.d. 18 Maret 2026 |
| tanggal_mulai | date | Awal periode |
| tanggal_selesai | date | Akhir periode |
| status_periode | enum('draft', 'final') | Status periode |
| created_by | bigint unsigned | FK ke users.id |
| finalized_by | bigint unsigned nullable | FK ke users.id |
| finalized_at | datetime nullable | Waktu finalisasi |

---

## 3. Rincian Menu Laporan
Bagian ini menjelaskan kolom tampilan setiap menu laporan, sumber tabelnya, dan join utamanya.

### 3.1 Laporan Persembahan Jemaat
**Tujuan:** menampilkan daftar persembahan per KK untuk satu periode.

| Nama Kolom UI | Sumber | Tipe |
| :--- | :--- | :--- |
| No KK | keluarga.no_kk | varchar |
| Nama KK | keluarga.nama_kk | varchar |
| Nama Pemberi | persembahan_jemaat.nama_pemberi | varchar |
| Sektor | sektor.nama / nama_sektor | varchar |
| Tanggal Terima | persembahan_jemaat.tanggal_terima | date |
| Diterima Melalui | persembahan_jemaat.diterima_melalui | varchar |
| Persepuluhan | sum(detail.nominal) filter persepuluhan | decimal |
| Sumb. Duka | sum(detail.nominal) filter sumbangan_duka | decimal |
| Sumb. SBU | sum(detail.nominal) filter sbu | decimal |
| Syukur | sum(detail.nominal) filter syukur | decimal |
| Jumlah | total semua jenis | decimal |
| Catatan | persembahan_jemaat.catatan | text |

* **Join:** `persembahan_jemaat.id_keluarga = keluarga.id_keluarga`
* **Join:** `keluarga.sektor_id = sektor.id`
* **Join:** `persembahan_jemaat.id_persembahan = persembahan_jemaat_detail.id_persembahan`

### 3.2 Laporan Penerimaan dan Pengeluaran
| Nama Kolom UI | Sumber | Tipe |
| :--- | :--- | :--- |
| Tanggal | keuangan_transaksi.tanggal_transaksi | date |
| Jenis | keuangan_transaksi.jenis_transaksi | enum |
| Kas | kas.nama_kas | varchar |
| Kategori | keuangan_transaksi_detail.kategori | varchar |
| Subkategori | keuangan_transaksi_detail.subkategori | varchar |
| Uraian | transaksi.keterangan / detail.keterangan | text |
| Penerimaan | nominal bila jenis = penerimaan | decimal |
| Pengeluaran | nominal bila jenis = pengeluaran | decimal |
| Status Posting | keuangan_transaksi.status_posting | enum |

* **Join:** `keuangan_transaksi.id_transaksi = keuangan_transaksi_detail.id_transaksi`
* **Join:** `keuangan_transaksi_detail.id_kas = kas.id_kas`

### 3.3 Laporan Kas
| Nama Kolom UI | Sumber | Tipe |
| :--- | :--- | :--- |
| Nama Kas | kas.nama_kas | varchar |
| Jenis Kas | kas.jenis_kas | enum |
| Total Penerimaan | agregat detail masuk | decimal |
| Total Pengeluaran | agregat detail keluar | decimal |
| Saldo Akhir | penerimaan - pengeluaran | decimal |
| Status | kas.is_aktif | boolean |

### 3.4 Laporan Dana Titipan
| Nama Kolom UI | Sumber | Tipe |
| :--- | :--- | :--- |
| Nama Dompet Titipan | kas_titipan.nama_titipan | varchar |
| Kas Induk | kas.nama_kas | varchar |
| Sektor | sektor.nama | varchar nullable |
| Pelkat | pelkat.nama_pelkat | varchar nullable |
| Total Penerimaan | agregat | decimal |
| Total Pengeluaran | agregat | decimal |
| Saldo Akhir | hitungan | decimal |

### 3.5 Rekap Persepuluhan Bulanan
| Nama Kolom UI | Sumber | Tipe |
| :--- | :--- | :--- |
| Sektor | sektor.nama | varchar |
| Jumlah KK | count keluarga.id_keluarga | int |
| Bulan | persembahan_periode_alokasi.bulan | tinyint |
| Tahun | persembahan_periode_alokasi.tahun | smallint |
| KK Aktif Memberi | count distinct id_keluarga dengan alokasi persepuluhan | int |

* **Filter penting:** `persembahan_jemaat_detail.jenis_persembahan = 'persepuluhan'`

### 3.6 Rekap Sumbangan Duka Bulanan
| Nama Kolom UI | Sumber | Tipe |
| :--- | :--- | :--- |
| No KK | keluarga.no_kk | varchar |
| Nama KK | keluarga.nama_kk | varchar |
| Sektor | sektor.nama | varchar |
| Jumlah Jiwa | keluarga.jumlah_jiwa / detail.jumlah_jiwa | int |
| Bulan | alokasi.bulan | tinyint |
| Tahun | alokasi.tahun | smallint |
| Tarif per Jiwa | detail.tarif_per_jiwa | decimal |
| Tagihan | detail.nominal_tagihan / hitungan | decimal |
| Dibayar | detail.nominal | decimal |
| Status | detail.status_pembayaran | enum |
| Sisa / Lebih Bayar | detail.sisa_kelebihan | decimal |

* **Filter penting:** `persembahan_jemaat_detail.jenis_persembahan = 'sumbangan_duka'`

### 3.7 Daftar Tunggakan Jemaat
| Nama Kolom UI | Sumber / Logika | Tipe |
| :--- | :--- | :--- |
| No KK | keluarga.no_kk | varchar |
| Nama KK | keluarga.nama_kk | varchar |
| Sektor | sektor.nama | varchar |
| Jenis | hasil filter: persepuluhan / sumbangan_duka | varchar |
| Bulan Belum Terpenuhi| hasil kalkulasi | int |
| Periode Terakhir Bayar| max bulan/tahun alokasi | varchar |
| Status | hasil kalkulasi | varchar |

### 3.8 Riwayat Pembelian SBU
| Nama Kolom UI | Sumber | Tipe |
| :--- | :--- | :--- |
| Tanggal | persembahan_jemaat.tanggal_terima | date |
| Nama KK | keluarga.nama_kk | varchar |
| Nama Pemberi | persembahan_jemaat.nama_pemberi | varchar |
| Sektor | sektor.nama | varchar |
| Bulan SBU | bulan + tahun alokasi | varchar |
| Jumlah Eksemplar | detail.jumlah_eksemplar | int |
| Harga Satuan | detail.harga_satuan | decimal |
| Total Bayar | detail.nominal | decimal |
| Catatan | detail.catatan | text |

* **Filter penting:** `persembahan_jemaat_detail.jenis_persembahan = 'sbu'`

---

## 4. Mapping dengan Struktur Existing Jemaatplus
* Tabel existing yang langsung dipakai: **users, jemaat, sektor, pelkat.**
* PK existing penting: **jemaat.id_jemaat, users.id, sektor.id, pelkat.id.**
* Gunakan **snake_case** agar konsisten dengan Laravel dan style tabel existing.
* Nama kolom yang direkomendasikan: `id_keluarga`, `nama_kk`, `id_transaksi`, `id_persembahan`, `id_kas`, `tanggal_transaksi`, `tanggal_terima`, `jenis_persembahan`, `status_posting`.

---

## 5. Alur Relasi Antar Tabel
* **Persembahan jemaat masuk:** `keluarga -> persembahan_jemaat -> persembahan_jemaat_detail -> persembahan_periode_alokasi`.
* **Posting ke kas:** `persembahan_jemaat -> keuangan_transaksi -> keuangan_transaksi_detail -> kas`.
* **Dana titipan:** `keuangan_transaksi_detail -> kas_titipan`.
* **Anggota keluarga:** `keluarga -> keluarga_jemaat -> jemaat`.

---

## 6. Keputusan Bisnis yang Sudah Fix
* Basis pelaporan persembahan adalah **KK**.
* Nama KK tampil utama, nama pemberi ikut tampil.
* Persepuluhan dipantau bulanan.
* Sumbangan duka dipantau per jiwa.
* Admin wajib pilih periode alokasi untuk persepuluhan, sumbangan duka, dan SBU.
* Syukur masuk ke Kas Dana Pelayanan.
* Persepuluhan masuk ke Kas Dana Pelayanan.
* Sumbangan duka masuk ke Kas Sumbangan Duka.
* Dana titipan dipisah dari form persembahan jemaat.
* Laporan final menggunakan periode mingguan.
