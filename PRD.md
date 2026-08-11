# PRD — SIPERDIN BAZNAS
### Sistem Informasi Perjalanan Dinas Badan Amil Zakat Nasional

| | |
|---|---|
| **Versi dokumen** | 1.2 |
| **Tanggal** | 11 Agustus 2026 |
| **Status** | Draf untuk ditinjau pemilik proses dan tim teknis |
| **Pemilik produk** | Bagian Umum — BAZNAS RI |
| **Pemangku kepentingan** | Pimpinan/Ketua Bidang, Kepala Bagian Umum, Bendahara Pengeluaran, Admin Anggaran, Satuan Audit Internal |
| **Tumpukan teknologi** | Laravel (PHP) · MySQL 8 · Livewire · Tailwind CSS |
| **Prototipe rujukan** | `index.html` pada repositori ini |

---

## Daftar isi

**Bagian I — Produk**
1. [Ringkasan eksekutif](#1-ringkasan-eksekutif)
2. [Latar belakang dan masalah](#2-latar-belakang-dan-masalah)
3. [Sasaran dan ukuran keberhasilan](#3-sasaran-dan-ukuran-keberhasilan)
4. [Pengguna dan peran](#4-pengguna-dan-peran)
5. [Glosarium](#5-glosarium)
6. [Alur proses — tiga belas tahap](#6-alur-proses--tiga-belas-tahap)
7. [Kebutuhan fungsional](#7-kebutuhan-fungsional)
8. [Aturan bisnis](#8-aturan-bisnis)
9. [Status berkas](#9-status-berkas)

**Bagian II — Teknis**
10. [Arsitektur](#10-arsitektur)
11. [Keputusan teknis yang mengikat](#11-keputusan-teknis-yang-mengikat)
12. [Skema basis data MySQL](#12-skema-basis-data-mysql)
13. [Struktur aplikasi Laravel](#13-struktur-aplikasi-laravel)
14. [Rute dan halaman](#14-rute-dan-halaman)
15. [Integrasi](#15-integrasi)
16. [Keamanan dan jejak audit](#16-keamanan-dan-jejak-audit)
17. [Kebutuhan non-fungsional](#17-kebutuhan-non-fungsional)
18. [Pengujian dan kriteria penerimaan](#18-pengujian-dan-kriteria-penerimaan)
19. [Penyebaran dan lingkungan](#19-penyebaran-dan-lingkungan)

**Bagian III — Pelaksanaan**
20. [Rencana rilis](#20-rencana-rilis)
21. [Migrasi data awal](#21-migrasi-data-awal)
22. [Risiko dan mitigasi](#22-risiko-dan-mitigasi)
23. [Asumsi](#23-asumsi)
24. [Pertanyaan terbuka](#24-pertanyaan-terbuka)
25. [Rujukan](#25-rujukan)
26. [Lampiran — matriks tanggung jawab](#26-lampiran--matriks-tanggung-jawab)

---
---

# Bagian I — Produk

## 1. Ringkasan eksekutif

SIPERDIN mengelola satu berkas perjalanan dinas dari pengajuan sampai pelunasan
sisa dalam **tiga belas tahap yang berurutan dan tidak dapat dilompati**. Sistem
menegakkan dua gerbang kepatuhan yang selama ini dikerjakan manual:

1. **Gerbang anggaran (DIPA).** Tidak ada perjalanan dinas di luar mata anggaran
   yang terdaftar dalam DIPA yang disahkan Kementerian Keuangan. Persetujuan
   pimpinan diambil dengan sisa pagu efektif tampil di hadapannya, dan dikunci
   sistem bila pagu tidak tersedia.
2. **Gerbang tarif (SBM).** Setiap komponen biaya dibandingkan dengan Standar
   Biaya Masukan tahun anggaran berjalan. Melampaui batas tetap dimungkinkan,
   tetapi wajib beralasan tertulis yang ikut menjadi bagian berkas.

Keluaran sistem adalah berkas yang lengkap dan siap diperiksa: Surat Tugas,
SPPD, rincian uang muka, bukti bayar, SPJ rampung, laporan perdin, kertas kerja
selisih, dan bukti pelunasan — seluruhnya terikat pada satu nomor berkas dan
satu jejak audit.

---

## 2. Latar belakang dan masalah

Pengelolaan perjalanan dinas saat ini tersebar pada lembar kerja, surel, dan
berkas cetak. Empat masalah berulang:

| Masalah | Akibat |
|---|---|
| Ketersediaan pagu diperiksa manual setelah persetujuan | Perjalanan sudah disetujui padahal akun tidak lagi memiliki pagu, atau diblokir tanda bintang |
| Tarif SBM disalin dari dokumen tahun lalu | Rincian ditolak verifikator, berkas bolak-balik, pembayaran terlambat |
| Posisi berkas tidak diketahui bersama | Pelaksana menalangi biaya lebih dulu; pertanyaan status membebani Bagian Umum |
| SPJ menyusul lama setelah perjalanan | Uang muka menggantung, laporan realisasi tidak mencerminkan keadaan |

**Ruang lingkup dokumen ini** adalah perjalanan dinas jabatan bagi amil dan
pegawai BAZNAS RI yang dibiayai APBN melalui DIPA satker BAZNAS.

---

## 3. Sasaran dan ukuran keberhasilan

| # | Sasaran | Ukuran | Target 6 bulan setelah rilis |
|---|---|---|---|
| S-1 | Tidak ada perdin melampaui pagu | Jumlah berkas disetujui yang melebihi sisa efektif | **0** |
| S-2 | Persetujuan lebih cepat | Median waktu tahap 01→02 | ≤ 1 hari kerja (SLA 2×24 jam) |
| S-3 | SPJ tepat waktu | Persentase SPJ rampung ≤ 5 hari kerja setelah kembali | ≥ 90% |
| S-4 | Uang muka tidak menggantung | Nilai uang muka belum di-SPJ pada akhir bulan | ≤ 5% pagu perdin |
| S-5 | Rincian benar sejak awal | Persentase berkas dikembalikan karena tarif tidak sesuai SBM | ≤ 5% |
| S-6 | Berkas siap audit | Persentase berkas selesai dengan dokumen wajib lengkap | 100% |

**Bukan sasaran rilis pertama:** perencanaan perjalanan tahunan, integrasi
pemesanan tiket, reimbursement non-perjalanan dinas, dan perjalanan dinas
pindah/mutasi.

---

## 4. Pengguna dan peran

| Peran | Siapa | Yang dikerjakan | Yang tidak boleh |
|---|---|---|---|
| **Pemohon** | Amil/pegawai pelaksana; ketua rombongan | Mengajukan, merinci biaya, mempertanggungjawabkan, menyusun laporan | Menyetujui, menerbitkan surat, membayar |
| **Pimpinan** | Ketua Bidang / Ketua BAZNAS | Menyetujui, mengembalikan, menolak, mendelegasikan | Mengubah rincian biaya, membayar |
| **Kepala Bagian** | Kabag Umum | Menerbitkan berkas, Surat Tugas, SPPD; menyetujui alasan melampaui SBM; menyetujui laporan perdin | Menyetujui pengajuan, membayar |
| **Verifikator Anggaran** | Staf verifikasi pada Bagian Keuangan | Mencocokkan usulan pembebanan dengan data DIPA, menetapkan akun definitif, memindahkan akun bila usulan keliru, menerbitkan register anggaran | Menyetujui substansi perjalanan, menerbitkan surat, membayar |
| **Bendahara** | Bendahara Pengeluaran | Membayar uang muka, memverifikasi SPJ, menghitung dan mengesahkan selisih, melunasi sisa | Mengubah maksud perjalanan, menyetujui pengajuan |
| **Admin Anggaran** | Pengelola data induk | Mengunggah dan memelihara SBM per tahun anggaran; menyinkronkan DIPA dan revisinya | Menyentuh berkas perjalanan dinas |
| **Auditor** *(baca saja)* | Satuan Audit Internal, BPK | Membaca berkas, dokumen, dan jejak audit; mengunduh paket audit | Semua aksi ubah |
| **Administrator Sistem** | Tim TI | Mengelola pengguna, peran, dan konfigurasi | Menyetujui atau membayar |

**Prinsip hak akses.** Kontrol di luar wewenang ditampilkan **nonaktif disertai
alasan**, bukan disembunyikan — pengguna tetap mengetahui langkah berikutnya dan
siapa yang berwenang.

---

## 5. Glosarium

| Istilah | Arti dalam sistem ini |
|---|---|
| **DIPA** | Daftar Isian Pelaksanaan Anggaran yang disahkan Kementerian Keuangan; sumber kebenaran pagu |
| **Pagu** | Nilai anggaran yang tersedia pada satu akun belanja |
| **Blokir** | Pagu bertanda bintang pada Halaman IV DIPA; tidak dapat dipakai sampai revisi disahkan |
| **Realisasi** | Nilai yang SP2D-nya sudah terbit |
| **Cadangan** | Nilai berkas yang sudah disetujui tetapi belum menjadi realisasi (*encumbrance*) |
| **Sisa efektif** | `pagu − blokir − realisasi − cadangan`; satu-satunya angka yang boleh dipakai menyetujui perdin baru |
| **Halaman III** | Rencana penarikan dana bulanan pada DIPA |
| **SBM** | Standar Biaya Masukan; tarif batas yang ditetapkan PMK tiap tahun anggaran |
| **UP / GUP** | Uang Persediaan; Ganti Uang Persediaan |
| **SPP / SPM / SP2D** | Surat Permintaan Pembayaran → Surat Perintah Membayar → Surat Perintah Pencairan Dana (KPPN) |
| **SPPD** | Surat Perintah Perjalanan Dinas; lembar II dibubuhi cap tiba dan berangkat |
| **SPJ** | Surat Pertanggungjawaban beserta seluruh bukti pengeluaran |
| **Berkas** | Satu perjalanan dinas beserta seluruh dokumen dan jejaknya; bernomor `PD/TTTT/BB/NNNN` |

---

## 6. Alur proses — tiga belas tahap

Rel tiga belas tahap adalah tulang punggung sistem. Setiap tahap memiliki satu
penanggung jawab, satu syarat perpindahan, dan satu layar kerja.

| # | Tahap | Penanggung jawab | Syarat berpindah |
|---|---|---|---|
| 01 | Pengajuan Perdin | Pemohon | Usulan pembebanan terisi dan pagu mencukupi |
| 02 | Persetujuan Pimpinan | Pimpinan | Cek pagu DIPA lulus; keputusan bercatatan |
| 03 | **Verifikasi & Alokasi Anggaran** | **Verifikator Anggaran** | **Akun definitif ditetapkan; cadangan terbentuk pada akun tersebut** |
| 04 | Eksekusi Kabag | Kepala Bagian | Empat butir daftar periksa tercentang |
| 05 | Surat Tugas | Kepala Bagian | Tanda tangan elektronik terbubuh |
| 06 | Rincian Uang Muka | Pemohon (verifikasi Kabag & Bendahara) | Sesuai SBM TA berjalan, atau beralasan tertulis |
| 07 | Dokumen SPPD | Kepala Bagian | SPPD terbit untuk tiap pelaksana |
| 08 | Pembayaran Uang Muka | Bendahara | Bukti transfer terunggah |
| 09 | Pertanggungjawaban Perdin | Pemohon | Tiap komponen punya bukti sah |
| 10 | Upload SPJ Rampung | Bendahara | Tidak ada dokumen bermasalah |
| 11 | Dokumen Laporan Perdin | Pemohon | Ringkasan, hasil, dan tindak lanjut terisi |
| 12 | Perhitungan Selisih Uang Muka | Bendahara | Selisih disahkan bendahara |
| 13 | Pembayaran Sisa Perdin | Bendahara | SP2D terbit; tiga syarat penutupan terpenuhi |

**Mengapa verifikasi berdiri sendiri.** Pemohon tahu ke mana ia pergi dan untuk
apa; ia tidak dituntut menguasai struktur DIPA. Karena itu pembebanan yang
diisinya berstatus **usulan**. Verifikator Anggaran-lah yang mencocokkannya
dengan DIPA dan menetapkan akun definitif — dan Surat Tugas tidak dapat terbit
sebelum penetapan itu ada, karena dasar pembebanan ikut tercetak di dalamnya.

**Perpindahan mundur.** Berkas dapat dikembalikan ke tahap sebelumnya oleh
pemegang wewenang tahap berjalan, disertai alasan wajib yang tampil di riwayat.
Pengembalian tidak menghapus dokumen yang sudah terbit; dokumen tersebut diberi
versi baru bila diterbitkan ulang.

---

## 7. Kebutuhan fungsional

Penomoran mengikuti pengelompokan: `F-1xx` pengajuan, `F-2xx` persetujuan,
`F-3xx` dokumen, `F-4xx` anggaran, `F-5xx` standar biaya dan rincian,
`F-6xx` pembayaran, `F-7xx` pertanggungjawaban, `F-8xx` pelaporan dan arsip.

### 7.1 Pengajuan (tahap 01)

| ID | Kebutuhan | Prioritas |
|---|---|---|
| F-101 | Pemohon mengisi jenis perjalanan (dalam kota / luar kota / luar negeri), maksud, dasar penugasan, tempat tujuan, tanggal berangkat dan kembali, serta daftar pelaksana | Wajib |
| F-102 | Formulir menyesuaikan jenis perjalanan: luar negeri menambah kolom paspor, visa, dan izin Setneg; dalam kota menyembunyikan penginapan dan tiket | Wajib |
| F-103 | Lama perjalanan dihitung sistem dari tanggal berangkat dan kembali; tidak dapat diketik manual | Wajib |
| F-104 | Pemohon memilih **kegiatan pada DIPA** sebagai **usulan pembebanan**; sistem menurunkan KRO, RO, komponen, dan akun belanja sesuai jenis perjalanan. Pemohon tidak mengetik kode akun, dan usulannya belum mengikat | Wajib |
| F-105 | Hanya kegiatan yang boleh diakses unit kerja pemohon yang ditampilkan | Wajib |
| F-106 | Sistem menampilkan cek pagu seketika: sisa efektif akun, nilai pengajuan, dan sisa setelah disetujui | Wajib |
| F-107 | Pengajuan **tidak dapat dikirim** bila usulan mata anggaran kosong, akun diblokir seluruhnya, atau nilai melebihi sisa efektif akun yang diusulkan | Wajib |
| F-108 | Tingkat jabatan tiap pelaksana ditarik dari data kepegawaian dan menentukan golongan tarif SBM pada tahap 05 | Wajib |
| F-109 | Lampiran dasar penugasan (undangan, TOR) wajib diunggah sebelum kirim | Wajib |
| F-110 | Pemohon dapat menyimpan draf tanpa validasi penuh | Wajib |
| F-111 | Sistem menahan pengajuan baru dari pelaksana yang memiliki SPJ lewat tenggat | Wajib |
| F-112 | Pemohon dapat menyalin rincian dari perdin sebelumnya sebagai titik awal | Sebaiknya |

### 7.2 Persetujuan (tahap 02)

| ID | Kebutuhan | Prioritas |
|---|---|---|
| F-201 | Layar keputusan menampilkan **lembar cek pagu DIPA**: pagu, blokir, realisasi, cadangan, sisa efektif, nilai pengajuan, dan sisa setelah disetujui | Wajib |
| F-202 | Lembar cek ditutup satu putusan tegas: *pagu tersedia* atau *pagu tidak tersedia* disertai alasan | Wajib |
| F-203 | Tombol **Setujui** dikunci sistem bila pagu tidak tersedia; tombol *Kembalikan untuk revisi* tetap terbuka | Wajib |
| F-204 | Keputusan wajib disertai catatan yang tampil di riwayat berkas dan terkirim ke pemohon | Wajib |
| F-205 | Persetujuan **mencadangkan pagu** pada akun yang **diusulkan**, seketika, dalam satu transaksi basis data dengan penguncian baris | Wajib |
| F-206 | Angka yang mendasari keputusan **dibekukan** sebagai bukti audit; berkas menyimpan salinan sisa efektif pada saat keputusan | Wajib |
| F-207 | Nilai di atas ambang tertentu atau perjalanan luar negeri menambah satu lapis persetujuan secara otomatis | Wajib |
| F-208 | Pimpinan dapat mendelegasikan wewenang persetujuan untuk rentang tanggal tertentu | Wajib |
| F-209 | Antrean persetujuan diurutkan berdasarkan sisa SLA, bukan tanggal masuk | Wajib |
| F-210 | Sistem menampilkan riwayat kepatuhan SPJ pemohon pada layar keputusan | Sebaiknya |
| F-211 | Persetujuan massal untuk berkas yang seluruh cek pagunya lulus | Bisa nanti |

### 7.3 Verifikasi dan alokasi anggaran (tahap 03)

| ID | Kebutuhan | Prioritas |
|---|---|---|
| F-321 | Verifikator melihat **usulan pemohon dan penetapannya bersanding**, lengkap dengan kegiatan, KRO, RO, komponen, dan akun | Wajib |
| F-322 | Verifikator **menetapkan akun definitif**; ia berwenang memilih akun yang berbeda dari usulan | Wajib |
| F-323 | Pemindahan akun **memindahkan cadangan pagu**: satu mutasi pelepasan pada akun usulan dan satu mutasi pencadangan pada akun definitif, dalam satu transaksi | Wajib |
| F-324 | Verifikator wajib mengisi **alasan** bila akun yang ditetapkan berbeda dari usulan; alasan tampil di riwayat dan pada kertas kerja | Wajib |
| F-325 | Daftar periksa verifikasi minimal: mata anggaran terdaftar pada DIPA revisi berlaku; akun sesuai jenis perjalanan; tidak ada blokir; sisa efektif mencukupi; sesuai rencana Halaman III bulan berjalan | Wajib |
| F-326 | Sistem menolak penetapan pada akun yang **tidak sesuai jenis perjalanan** (mis. paket meeting untuk perjalanan biasa) | Wajib |
| F-327 | Penetapan alokasi menerbitkan **nomor register anggaran** yang menjadi rujukan pembebanan pada Surat Tugas dan SPPD | Wajib |
| F-328 | Verifikator dapat mengembalikan berkas ke pemohon atau pimpinan disertai alasan | Wajib |
| F-329 | **Surat Tugas tidak dapat diterbitkan** sebelum alokasi definitif ditetapkan | Wajib |
| F-330 | Antrean verifikasi memiliki SLA tersendiri dan tampil sebagai layar kerja Verifikator Anggaran | Wajib |
| F-331 | Verifikator tidak berwenang mengubah maksud, tanggal, pelaksana, atau nilai pengajuan | Wajib |

### 7.4 Eksekusi dan dokumen (tahap 04, 05, 07)

| ID | Kebutuhan | Prioritas |
|---|---|---|
| F-301 | Kabag menjalankan daftar periksa: kelengkapan lampiran, **alokasi anggaran definitif diterima dari verifikator**, alokasi nomor Surat Tugas, penunjukan bendahara pembayar | Wajib |
| F-302 | Kabag menunjuk pemegang uang muka (satu rekening per rombongan) | Wajib |
| F-303 | Penerbitan berkas **mengunci komponen biaya**; perubahan setelahnya harus melalui adendum bernomor dan menerbitkan ulang Surat Tugas | Wajib |
| F-304 | Nomor Surat Tugas dan SPPD dibangkitkan sistem menurut format resmi, berurutan, dan tidak dapat diketik manual | Wajib |
| F-305 | Surat Tugas dan SPPD ditandatangani dengan tanda tangan elektronik tersertifikasi | Wajib |
| F-310 | Surat Tugas dan SPPD **mencantumkan dasar pembebanan**: nomor DIPA, kegiatan, komponen, akun, dan nomor register anggaran | Wajib |
| F-306 | SPPD terbit dua lembar: lembar penugasan dan lembar pengesahan yang dicetak untuk dibubuhi cap tiba dan berangkat di tempat tujuan | Wajib |
| F-307 | SPPD dapat diterbitkan sekaligus untuk seluruh pelaksana dalam satu rombongan | Wajib |
| F-308 | Setiap dokumen memiliki versi; versi lama tetap tersimpan dan dapat dibaca | Wajib |
| F-309 | Dokumen memuat penanda keaslian (kode QR menuju halaman verifikasi publik) | Sebaiknya |

### 7.5 Anggaran dan DIPA

| ID | Kebutuhan | Prioritas |
|---|---|---|
| F-401 | Sistem menyimpan DIPA satker beserta nomor, tahun anggaran, revisi berjalan, tanggal dan pejabat pengesah | Wajib |
| F-402 | Struktur anggaran mengikuti Program → Kegiatan → KRO → RO → Komponen → Akun | Wajib |
| F-403 | Tiap akun menyimpan pagu, blokir, realisasi, dan cadangan; sisa efektif dihitung sistem | Wajib |
| F-404 | Siklus status nilai: **cadangan** saat disetujui → tetap cadangan saat dibayar dari UP → **realisasi** saat SP2D terbit | Wajib |
| F-405 | Pembatalan atau penolakan berkas **melepas cadangan** kembali ke sisa efektif | Wajib |
| F-406 | Sistem menyimpan Halaman III (rencana penarikan dana bulanan) dan membandingkannya dengan pengajuan bulan berjalan | Wajib |
| F-407 | Pengajuan melebihi rencana bulan berjalan tetap dapat disetujui, dengan peringatan dan alasan, serta menandai perlunya usulan revisi Halaman III | Wajib |
| F-408 | Riwayat revisi DIPA tersimpan; berkas yang sedang berjalan **dinilai ulang otomatis** saat revisi disahkan | Wajib |
| F-409 | Usulan revisi yang sedang berjalan ditampilkan beserta berkas dan nilai yang tertahan karenanya | Wajib |
| F-410 | Berkas yang tertahan karena blokir **tidak dihapus**; berkas dinilai ulang begitu revisi disahkan tanpa pengajuan ulang | Wajib |
| F-411 | Data DIPA disinkronkan dari sistem keuangan negara; unggahan berkas menjadi jalur cadangan yang wajib tersedia sejak Fase 1 | Wajib |

### 7.6 Standar Biaya Masukan dan rincian (tahap 06)

| ID | Kebutuhan | Prioritas |
|---|---|---|
| F-501 | SBM dikelola sebagai **data induk berversi tahun anggaran**, bukan angka yang diketik ulang tiap berkas | Wajib |
| F-502 | Admin Anggaran mengunggah SBM tahun baru beserta dasar hukumnya (nomor PMK) dan tanggal berlaku | Wajib |
| F-503 | Kategori tarif minimal: uang harian, penginapan, transpor lokal, dan batas tiket pesawat | Wajib |
| F-504 | Tarif dibedakan menurut wilayah (provinsi atau rute) dan golongan/tingkat jabatan bila berlaku | Wajib |
| F-505 | Daftar tarif menampilkan pembanding tahun anggaran sebelumnya beserta persentase perubahan | Sebaiknya |
| F-506 | Impor massal dari XLSX dengan templat resmi, disertai pratinjau perubahan sebelum diberlakukan | Wajib |
| F-507 | Versi yang dipakai sebuah berkas ditentukan **tanggal berangkat**, bukan tanggal pengajuan | Wajib |
| F-508 | Versi SBM **dibekukan pada berkas saat Surat Tugas terbit**; pembaruan berikutnya tidak mengubah rincian yang sudah disetujui | Wajib |
| F-509 | Saat SBM baru diberlakukan, sistem menandai berkas berstatus draf yang tarifnya menjadi tidak sesuai dan memberi tahu pemohonnya | Wajib |
| F-510 | Versi lama tidak dihapus; berkas lama tetap dapat dibaca dengan tarif yang dipakainya | Wajib |
| F-511 | Sistem memperingatkan bila SBM tahun anggaran berikutnya belum diunggah menjelang akhir tahun | Wajib |
| F-512 | Seluruh aksi pengelolaan SBM terbatas pada peran Admin Anggaran dan tercatat pada jejak audit | Wajib |
| F-513 | Rincian disusun per komponen dengan volume, satuan, dan harga satuan; jumlah dihitung sistem | Wajib |
| F-514 | Tiap baris menampilkan **batas SBM** yang berlaku beserta putusan kesesuaiannya, dihitung ulang seketika saat nilai berubah | Wajib |
| F-515 | Baris yang melampaui batas ditandai jelas dan **membuka kolom alasan wajib** | Wajib |
| F-516 | Pengiriman rincian **dikunci** selama masih ada baris melampaui batas tanpa alasan | Wajib |
| F-517 | Alasan melampaui SBM disetujui Kepala Bagian dan ikut tercetak pada kertas kerja rincian biaya | Wajib |
| F-518 | Porsi uang muka dapat diatur sampai batas kebijakan (maksimal 80% rencana biaya); sisanya ditahan sampai SPJ rampung | Wajib |
| F-519 | Seluruh komponen dibebankan pada akun DIPA yang disetujui pimpinan dan tidak dapat diubah pada tahap ini | Wajib |
| F-520 | Rekening penerima uang muka diverifikasi terhadap data kepegawaian | Wajib |

### 7.7 Pembayaran (tahap 08 dan 13)

| ID | Kebutuhan | Prioritas |
|---|---|---|
| F-601 | Uang muka dibayarkan ke satu rekening ketua rombongan melalui voucher bernomor | Wajib |
| F-602 | Bendahara mengunggah bukti transfer; berkas berpindah tahap otomatis setelah bukti terverifikasi | Wajib |
| F-603 | Sistem membedakan sifat pembayaran: **uang muka**, **pelunasan sisa**, dan **tagihan pengembalian** | Wajib |
| F-604 | Antrean pembayaran mendukung pemrosesan berkelompok dan ekspor daftar transfer bank | Wajib |
| F-605 | Pembayaran mencatat mekanisme (UP atau LS) serta nomor SPM dan SP2D yang menihilkannya | Wajib |
| F-606 | Tagihan pengembalian memiliki jatuh tempo, kode setor, dan penanda keterlambatan | Wajib |
| F-607 | Pelunasan sisa hanya dapat dieksekusi setelah SPJ rampung, laporan disetujui, dan selisih disahkan | Wajib |
| F-608 | Integrasi transfer bank *host-to-host* dengan persetujuan berjenjang | Bisa nanti |

### 7.8 Pertanggungjawaban dan selisih (tahap 09, 10, 12)

| ID | Kebutuhan | Prioritas |
|---|---|---|
| F-701 | Realisasi diisi per komponen bersanding dengan rencana; selisih per baris dihitung dan diberi arah | Wajib |
| F-702 | Tiap komponen memerlukan bukti; sistem membedakan komponen lumpsum (uang harian) dari at-cost (penginapan, transpor, tiket) | Wajib |
| F-703 | Realisasi diuji dengan **versi SBM yang dibekukan pada berkas**, bukan versi terbaru | Wajib |
| F-704 | Kelebihan di atas batas SBM menjadi beban pribadi pelaksana kecuali sudah beralasan dan disetujui pada tahap 05 | Wajib |
| F-705 | Unggahan SPJ mendukung banyak berkas, penandaan jenis dokumen, versi, dan status verifikasi per dokumen | Wajib |
| F-706 | Berkas **tidak dapat dinyatakan rampung** selama masih ada dokumen berstatus perlu perbaikan | Wajib |
| F-707 | Perhitungan selisih menghasilkan salah satu dari dua arah: **kurang bayar** (hak pelaksana) atau **kelebihan** (wajib dikembalikan), masing-masing memicu alur berbeda | Wajib |
| F-708 | Efisiensi terhadap rencana dikembalikan ke pagu unit, bukan menjadi hak pelaksana | Wajib |
| F-709 | Kertas kerja selisih dapat diunduh sebagai dokumen tersendiri | Wajib |
| F-710 | Tenggat SPJ 5 hari kerja setelah kembali; pelanggaran menahan pengajuan baru pelaksana | Wajib |
| F-711 | Pemindaian bukti dengan pembacaan nominal otomatis | Bisa nanti |

### 7.9 Laporan, arsip, dan penutupan (tahap 11, 13)

| ID | Kebutuhan | Prioritas |
|---|---|---|
| F-801 | Laporan substansi **terpisah** dari pertanggungjawaban keuangan; keduanya diverifikasi pihak berbeda dan tidak saling menahan | Wajib |
| F-802 | Laporan memuat ringkasan pelaksanaan, hasil yang dicapai, tindak lanjut, dokumentasi kegiatan, dan lampiran | Wajib |
| F-803 | Berkas ditutup hanya bila empat syarat terpenuhi: SPJ rampung, laporan disetujui, selisih disahkan, pembayaran sisa dieksekusi | Wajib |
| F-804 | Setelah ditutup, seluruh dokumen **dikunci** — hanya dapat dibaca dan diunduh | Wajib |
| F-805 | Arsip dokumen dapat dicari lintas tahun berdasarkan nomor berkas, jenis dokumen, kota, unit kerja, dan kata kunci | Wajib |
| F-806 | Paket audit satu berkas dapat diunduh sebagai satu arsip terkompresi berisi seluruh dokumen dan jejak audit | Wajib |
| F-807 | Laporan pengelola: serapan per akun, uang muka beredar, kepatuhan SPJ, dan berkas melewati SLA | Wajib |
| F-808 | Ekspor tabular (XLSX) untuk seluruh daftar dan laporan | Wajib |

---

## 8. Aturan bisnis

| ID | Aturan | Ditegakkan di |
|---|---|---|
| BR-01 | `sisa efektif = pagu − blokir − realisasi − cadangan`. Hanya sisa efektif yang boleh dipakai menyetujui pengajuan baru | `AnggaranService::sisaEfektif()` |
| BR-02 | Persetujuan mencadangkan pagu pada akun usulan. Verifikasi memindahkannya ke akun definitif. Cadangan dilepas bila berkas dibatalkan atau ditolak | `AnggaranService::cadangkan()` · `pindahkanCadangan()` · `lepasCadangan()` |
| BR-03 | Nilai menjadi realisasi hanya setelah SP2D terbit — bukan saat uang muka ditransfer dari UP | `AnggaranService::realisasikan()` |
| BR-04 | Versi SBM ditentukan tanggal berangkat, dan dibekukan pada berkas saat Surat Tugas terbit | `SbmResolver::untukBerkas()` |
| BR-05 | Melampaui SBM diperbolehkan dengan alasan tertulis yang disetujui Kepala Bagian; tanpa alasan, rincian tidak dapat dikirim | `RincianService::validasiSbm()` |
| BR-06 | Uang muka maksimal 80% rencana biaya | `config('perdin.uang_muka.persen_maks')` |
| BR-07 | Satu rombongan memiliki satu pemegang uang muka dan satu rekening penerima | Basis data: indeks unik parsial |
| BR-08 | Tenggat SPJ 5 hari kerja setelah tanggal kembali | `TenggatService` + hari libur nasional |
| BR-09 | Pelaksana dengan SPJ lewat tenggat tidak dapat mengajukan perdin baru | `PerdinPolicy::create()` |
| BR-10 | SLA persetujuan 2×24 jam kerja; pelanggaran dieskalasi ke atasan pemberi persetujuan | Penjadwal harian |
| BR-11 | Nomor berkas, Surat Tugas, SPPD, dan voucher dibangkitkan sistem, berurutan, tanpa lompatan, dan tidak dapat digunakan ulang | `NomorService` dengan penguncian baris |
| BR-12 | Perubahan setelah Surat Tugas terbit memerlukan adendum bernomor dan penerbitan ulang dokumen | `PerdinState` |
| BR-15 | Surat Tugas hanya dapat terbit bila `perdin.mata_anggaran_id` (akun definitif) dan nomor register anggaran sudah terisi | `PenerbitSuratTugas` |
| BR-13 | Efisiensi realisasi terhadap rencana kembali ke pagu, bukan menjadi hak pelaksana | `SelisihService` |
| BR-14 | Berkas selesai bersifat hanya-baca; koreksi dilakukan melalui berkas koreksi yang merujuk berkas asal | `PerdinPolicy::update()` |

---

## 9. Status berkas

```
Draf ──► Menunggu persetujuan ──► Disetujui ──► Terbit ──► Dibayar di muka
                    │                                            │
                    ├─► Dikembalikan ──┐                         ▼
                    └─► Ditolak        │              Pertanggungjawaban
                                       │                         │
                                       └──── (kembali ke Draf)   ▼
                                                          SPJ rampung
                                                                 │
                                                                 ▼
                                                   Selisih disahkan ──► Selesai
```

| Status (`enum`) | Arti | Siapa yang harus bertindak |
|---|---|---|
| `draf` | Hanya terlihat pemohon; belum menyentuh anggaran | Pemohon |
| `menunggu_persetujuan` | SLA berjalan | Pimpinan |
| `dikembalikan` | Bola di pemohon; alasan wajib dan tampil di riwayat | Pemohon |
| `ditolak` | Berakhir; cadangan tidak pernah terbentuk | — |
| `pagu_terkunci` | Akun diblokir atau sisa efektif tidak cukup | Admin Anggaran |
| `disetujui` | Cadangan terbentuk pada akun usulan | Verifikator Anggaran |
| `menunggu_verifikasi` | Menunggu penetapan akun definitif | Verifikator Anggaran |
| `teralokasi` | Akun definitif ditetapkan; register anggaran terbit | Kepala Bagian |
| `terbit` | Surat Tugas & SPPD terbit; komponen biaya terkunci | Bendahara |
| `dibayar_di_muka` | Uang muka ditransfer dari UP | Pemohon |
| `pertanggungjawaban` | Realisasi sedang diisi | Pemohon |
| `spj_rampung` | Seluruh bukti terverifikasi | Bendahara |
| `selisih_disahkan` | Kertas kerja selisih terbit | Bendahara |
| `selesai` | Dokumen terkunci, hanya dapat dibaca dan diunduh | — |
| `dibatalkan` | Cadangan dilepas kembali ke pagu | — |

---
---

# Bagian II — Teknis

## 10. Arsitektur

```
┌──────────────────────────────────────────────────────────────┐
│  Peramban — Blade + Livewire 3 + Alpine.js + Tailwind CSS    │
│  Layar kerja stateful: cek pagu langsung, validasi SBM       │
└───────────────────────────┬──────────────────────────────────┘
                            │ HTTP (sesi, CSRF)
┌───────────────────────────▼──────────────────────────────────┐
│  Laravel                                                     │
│  ┌─────────────┬──────────────┬───────────────────────────┐  │
│  │ Http        │ Domain       │ Support                   │  │
│  │ Controllers │ Services     │ Policies · FormRequests   │  │
│  │ Livewire    │ Actions      │ Enums · Casts · Rules     │  │
│  │ Middleware  │ States       │ Notifications · Exports   │  │
│  └─────────────┴──────────────┴───────────────────────────┘  │
│  Queue (Redis + Horizon)  ·  Scheduler  ·  Events/Listeners  │
└──────┬───────────────────────┬────────────────────┬──────────┘
       │                       │                    │
┌──────▼──────┐        ┌───────▼───────┐   ┌────────▼─────────┐
│  MySQL 8    │        │ Penyimpanan   │   │ Layanan luar     │
│  InnoDB     │        │ objek (S3/    │   │ SAKTI · BSrE ·   │
│  utf8mb4    │        │ MinIO)        │   │ HRIS · SMTP      │
└─────────────┘        └───────────────┘   └──────────────────┘
```

**Mengapa Livewire, bukan SPA terpisah.** Layar yang paling menentukan — cek pagu
DIPA dan validasi SBM — memerlukan perhitungan yang **tidak boleh berbeda antara
peramban dan peladen**. Dengan Livewire, aturan `sisa efektif` dan `batas SBM`
hidup di satu tempat (PHP) dan tidak perlu ditulis ulang dalam JavaScript, sehingga
tidak ada risiko validasi peramban dan peladen berbeda hasil. Alpine.js dipakai
untuk interaksi murni tampilan (buka-tutup, tab, fokus).

---

## 11. Keputusan teknis yang mengikat

Sepuluh keputusan berikut bukan preferensi gaya. Melanggarnya menimbulkan
kesalahan keuangan atau temuan audit.

### 11.1 Uang disimpan sebagai bilangan bulat rupiah

```php
// migrasi
$table->unsignedBigInteger('nilai'); // rupiah penuh, tanpa sen

// model
protected function casts(): array {
    return ['nilai' => 'integer'];
}
```

**Jangan pernah** memakai `FLOAT` atau `DOUBLE` untuk uang. APBN tidak mengenal
pecahan sen pada tingkat ini; `BIGINT` rupiah penuh menghilangkan seluruh
kesalahan pembulatan. Perhitungan persentase (porsi uang muka) memakai
pembulatan eksplisit ke rupiah terdekat, dan hasilnya disimpan — bukan dihitung
ulang saat ditampilkan.

### 11.2 Pagu adalah buku besar, bukan angka yang ditimpa

Kolom `cadangan` dan `realisasi` **tidak diperbarui dengan `UPDATE ... SET
cadangan = cadangan + x`**. Setiap perubahan dicatat sebagai baris pada
`anggaran_mutasi` (append-only), dan saldo diperoleh dengan penjumlahan.
Alasannya: auditor selalu dapat menanyakan "mengapa angkanya sekian", dan
jawabannya harus dapat ditelusuri baris per baris.

Untuk kinerja, saldo diringkas pada `pagu_akun_saldo` yang diperbarui dalam
transaksi yang sama — ringkasan boleh dibangun ulang kapan saja dari buku besar.

### 11.3 Pencadangan pagu wajib memakai penguncian baris

Dua pimpinan menyetujui dua berkas pada akun yang sama, pada detik yang sama,
dengan sisa yang hanya cukup untuk satu. Tanpa penguncian, keduanya lolos.

```php
DB::transaction(function () use ($perdin) {
    $saldo = PaguAkunSaldo::where('pagu_akun_id', $perdin->pagu_akun_id)
        ->lockForUpdate()          // SELECT ... FOR UPDATE
        ->firstOrFail();

    $sisaEfektif = $saldo->pagu - $saldo->blokir - $saldo->realisasi - $saldo->cadangan;

    if ($perdin->nilai_rencana > $sisaEfektif) {
        throw new PaguTidakCukup($perdin, $sisaEfektif);
    }

    AnggaranMutasi::create([...]);   // jenis: cadangan
    $saldo->increment('cadangan', $perdin->nilai_rencana);
    $perdin->state->transitionTo(Disetujui::class);
});
```

Tingkat isolasi transaksi: **REPEATABLE READ** (bawaan InnoDB) sudah memadai
karena penguncian dilakukan eksplisit.

### 11.4 Angka keputusan dibekukan

Tabel `perdin_persetujuan` menyimpan salinan `pagu`, `blokir`, `realisasi`,
`cadangan`, dan `sisa_efektif` **pada saat keputusan diambil**. Angka ini tidak
pernah dihitung ulang. Keputusan dinilai dengan pagu yang berlaku saat itu.

### 11.5 Versi SBM dibekukan pada berkas

`perdin.sbm_versi_id` diisi saat Surat Tugas terbit dan tidak pernah berubah.
Seluruh pengujian tarif pada tahap 05 dan 08 memakai kolom ini, bukan versi
yang sedang berlaku.

### 11.6 Penomoran berurutan tanpa lompatan

```php
// nomor_urut: (jenis, tahun, bulan) → unique, dikunci baris saat diambil
$urut = DB::transaction(function () use ($jenis, $tahun, $bulan) {
    $baris = NomorUrut::where(compact('jenis','tahun','bulan'))
        ->lockForUpdate()->firstOrCreate([...], ['terakhir' => 0]);
    $baris->increment('terakhir');
    return $baris->terakhir;
});
```

Nomor **diambil saat dokumen benar-benar terbit**, bukan saat formulir dibuka —
agar tidak ada nomor terbuang. Format diatur pada `config/perdin.php` sehingga
dapat diubah tanpa menyentuh kode.

### 11.7 Dokumen bersifat *immutable* dan berversi

Berkas PDF yang sudah ditandatangani tidak pernah ditimpa. Penerbitan ulang
membuat baris baru pada `perdin_dokumen` dengan `versi` bertambah; versi lama
tetap dapat diunduh.

### 11.8 Jejak audit *append-only*

Tabel `audit_log` tidak memiliki rute pembaruan maupun penghapusan pada tingkat
aplikasi. Hak `DELETE` dan `UPDATE` untuk tabel ini **dicabut pada pengguna basis
data aplikasi**; hanya `INSERT` dan `SELECT` yang diberikan.

### 11.9 Penghapusan lunak, tanpa penghapusan keras

Seluruh entitas transaksional memakai `softDeletes()`. Retensi 10 tahun berarti
tidak ada data yang benar-benar dihapus dalam masa itu.

### 11.10 Waktu dan bahasa

`APP_TIMEZONE=Asia/Jakarta`, `APP_LOCALE=id`, kolom waktu bertipe `TIMESTAMP`.
Seluruh tampilan tanggal memakai format Indonesia; seluruh angka memakai
pemisah ribuan titik.

---

## 12. Skema basis data MySQL

**Konvensi.** Mesin `InnoDB`, karakter `utf8mb4`, kolasi `utf8mb4_unicode_ci`.
Nama tabel jamak dalam bahasa Indonesia. Kunci utama `BIGINT UNSIGNED AUTO_INCREMENT`.
Seluruh kunci asing memakai `RESTRICT` pada penghapusan, kecuali disebut lain.
Kolom uang bertipe `BIGINT UNSIGNED` dalam rupiah penuh.

### 12.1 Peta tabel

| Kelompok | Tabel |
|---|---|
| Pengguna & organisasi | `users`, `pegawai`, `unit_kerja`, `roles`, `permissions`, `model_has_roles` |
| Anggaran | `dipa`, `dipa_revisi`, `mata_anggaran`, `pagu_akun`, `pagu_akun_saldo`, `anggaran_mutasi`, `rpd_bulanan` |
| Standar biaya | `sbm_versi`, `sbm_tarif` |
| Berkas perdin | `perdin`, `perdin_pelaksana`, `perdin_persetujuan`, `perdin_rincian`, `perdin_realisasi`, `perdin_laporan` |
| Dokumen & bayar | `perdin_dokumen`, `pembayaran`, `lampiran` |
| Sistem | `audit_log`, `nomor_urut`, `notifications`, `jobs`, `failed_jobs`, `hari_libur` |

### 12.2 Anggaran

**`dipa`**

| Kolom | Tipe | Keterangan |
|---|---|---|
| `id` | BIGINT UNSIGNED PK | |
| `nomor` | VARCHAR(64) | mis. `SP DIPA-025.03.1.426871/2026` |
| `tahun_anggaran` | SMALLINT UNSIGNED | |
| `kode_satker` | VARCHAR(16) | |
| `nama_satker` | VARCHAR(160) | |
| `bagian_anggaran` | VARCHAR(16) | |
| `revisi_berlaku` | TINYINT UNSIGNED | nomor revisi yang sedang berlaku |
| `tanggal_sah` | DATE | |
| `pejabat_pengesah` | VARCHAR(160) | |
| `status` | ENUM('berlaku','arsip') | |
| `timestamps`, `deleted_at` | | |

> Indeks unik: `(kode_satker, tahun_anggaran)`.

**`dipa_revisi`**

| Kolom | Tipe | Keterangan |
|---|---|---|
| `id` | BIGINT UNSIGNED PK | |
| `dipa_id` | FK → `dipa` | |
| `nomor_revisi` | TINYINT UNSIGNED | 0 = DIPA awal |
| `perihal` | VARCHAR(255) | |
| `pengesah` | VARCHAR(160) | Ditjen Anggaran / Kanwil DJPb |
| `tanggal_sah` | DATE | nullable saat masih usulan |
| `status` | ENUM('usulan','berlaku','digantikan') | |
| `dampak_nilai` | BIGINT SIGNED | dapat negatif |

**`mata_anggaran`**

| Kolom | Tipe | Keterangan |
|---|---|---|
| `id` | BIGINT UNSIGNED PK | |
| `dipa_id` | FK → `dipa` | |
| `program` | VARCHAR(24) | mis. `025.03.WA` |
| `kegiatan` | VARCHAR(16) | mis. `2104` |
| `kro` | VARCHAR(24) | mis. `2104.EBA` |
| `ro` | VARCHAR(32) | mis. `2104.EBA.994` |
| `komponen` | VARCHAR(8) | mis. `051` |
| `akun` | VARCHAR(8) | mis. `524111` |
| `uraian` | VARCHAR(255) | |
| `jenis_perjalanan` | SET('dalam_kota','luar_kota','luar_negeri') | akun mana untuk jenis apa |

> Indeks unik: `(dipa_id, kegiatan, kro, ro, komponen, akun)`.
> Indeks: `(dipa_id, akun)`.

**`pagu_akun`** — pagu yang ditetapkan per revisi

| Kolom | Tipe |
|---|---|
| `id` | BIGINT UNSIGNED PK |
| `mata_anggaran_id` | FK → `mata_anggaran` |
| `dipa_revisi_id` | FK → `dipa_revisi` |
| `pagu` | BIGINT UNSIGNED |
| `blokir` | BIGINT UNSIGNED |
| `berlaku_sejak` | DATE |

**`pagu_akun_saldo`** — ringkasan berjalan (dibangun ulang dari buku besar)

| Kolom | Tipe | Keterangan |
|---|---|---|
| `id` | BIGINT UNSIGNED PK | |
| `mata_anggaran_id` | FK unik → `mata_anggaran` | satu baris per akun |
| `pagu` | BIGINT UNSIGNED | |
| `blokir` | BIGINT UNSIGNED | |
| `realisasi` | BIGINT UNSIGNED | |
| `cadangan` | BIGINT UNSIGNED | |
| `dihitung_pada` | TIMESTAMP | |

> `sisa_efektif` **tidak disimpan** sebagai kolom biasa; gunakan kolom hasil
> perhitungan agar tidak pernah berbeda dari komponennya:
> ```sql
> sisa_efektif BIGINT AS (pagu - blokir - realisasi - cadangan) STORED
> ```

**`anggaran_mutasi`** — buku besar, *append-only*

| Kolom | Tipe | Keterangan |
|---|---|---|
| `id` | BIGINT UNSIGNED PK | |
| `mata_anggaran_usulan_id` | FK → `mata_anggaran` | diisi pemohon pada tahap 01 |
| `mata_anggaran_id` | FK → `mata_anggaran` nullable | **akun definitif**, diisi verifikator pada tahap 03 |
| `nomor_register_anggaran` | VARCHAR(32) UNIQUE nullable | terbit saat alokasi ditetapkan |
| `perdin_id` | FK → `perdin` (nullable) | |
| `jenis` | ENUM('cadangan','lepas_cadangan','pindah_keluar','pindah_masuk','realisasi','koreksi_pagu','blokir','buka_blokir') | `pindah_keluar` dan `pindah_masuk` selalu berpasangan dalam satu transaksi |
| `nilai` | BIGINT SIGNED | positif menambah beban, negatif melepas |
| `referensi` | VARCHAR(64) | nomor SP2D / SPM / revisi |
| `keterangan` | VARCHAR(255) | |
| `dibuat_oleh` | FK → `users` | |
| `created_at` | TIMESTAMP | tanpa `updated_at` — baris tidak pernah diubah |

> Indeks: `(mata_anggaran_id, jenis)`, `(perdin_id)`.

**`rpd_bulanan`** — Halaman III

| Kolom | Tipe |
|---|---|
| `mata_anggaran_id` | FK |
| `bulan` | TINYINT UNSIGNED (1–12) |
| `nilai_rencana` | BIGINT UNSIGNED |

> Indeks unik: `(mata_anggaran_id, bulan)`.

### 12.3 Standar biaya

**`sbm_versi`**

| Kolom | Tipe | Keterangan |
|---|---|---|
| `id` | BIGINT UNSIGNED PK | |
| `tahun_anggaran` | SMALLINT UNSIGNED UNIQUE | |
| `dasar_hukum` | VARCHAR(128) | nomor PMK |
| `berlaku_sejak` | DATE | |
| `berlaku_sampai` | DATE nullable | |
| `status` | ENUM('draf','berlaku','arsip') | |
| `diunggah_oleh` | FK → `users` | |
| `catatan` | TEXT nullable | |

**`sbm_tarif`**

| Kolom | Tipe | Keterangan |
|---|---|---|
| `id` | BIGINT UNSIGNED PK | |
| `sbm_versi_id` | FK → `sbm_versi` | |
| `kategori` | ENUM('uang_harian','penginapan','transpor_lokal','tiket_pesawat','representasi') | |
| `wilayah` | VARCHAR(64) nullable | provinsi |
| `rute` | VARCHAR(32) nullable | mis. `CGK-UPG` |
| `golongan` | VARCHAR(32) nullable | mis. `III` / `Eselon II` |
| `satuan` | VARCHAR(24) | `orang/hari`, `orang/malam`, `orang/kali`, `orang/PP` |
| `nilai` | BIGINT UNSIGNED | |

> Indeks unik: `(sbm_versi_id, kategori, wilayah, rute, golongan)`.
> Indeks pencarian: `(sbm_versi_id, kategori, wilayah)`.

### 12.4 Berkas perjalanan dinas

**`perdin`**

| Kolom | Tipe | Keterangan |
|---|---|---|
| `id` | BIGINT UNSIGNED PK | |
| `nomor` | VARCHAR(32) UNIQUE nullable | `PD/2026/08/0147`, diisi saat kirim |
| `jenis` | ENUM('dalam_kota','luar_kota','luar_negeri') | |
| `maksud` | VARCHAR(500) | disalin apa adanya ke Surat Tugas |
| `dasar_penugasan` | VARCHAR(500) | |
| `kota_tujuan` | VARCHAR(128) | |
| `provinsi` | VARCHAR(64) | penentu tarif SBM |
| `negara` | VARCHAR(64) nullable | |
| `tanggal_berangkat` | DATE | |
| `tanggal_kembali` | DATE | |
| `lama_hari` | SMALLINT UNSIGNED | dihitung sistem |
| `unit_kerja_id` | FK → `unit_kerja` | |
| `mata_anggaran_id` | FK → `mata_anggaran` | |
| `sbm_versi_id` | FK → `sbm_versi` nullable | **dibekukan saat Surat Tugas terbit** |
| `pemohon_id` | FK → `users` | |
| `pemegang_uang_muka_id` | FK → `pegawai` nullable | |
| `bendahara_id` | FK → `users` nullable | |
| `nilai_rencana` | BIGINT UNSIGNED | jumlah rincian |
| `persen_uang_muka` | TINYINT UNSIGNED | 0–80 |
| `nilai_uang_muka` | BIGINT UNSIGNED | dibulatkan dan disimpan |
| `nilai_realisasi` | BIGINT UNSIGNED nullable | |
| `nilai_selisih` | BIGINT SIGNED nullable | positif = kurang bayar |
| `status` | VARCHAR(32) | nilai `PerdinState` |
| `tahap` | TINYINT UNSIGNED | 1–12 |
| `tenggat_spj` | DATE nullable | |
| `dikunci_pada` | TIMESTAMP nullable | saat berkas selesai |
| `timestamps`, `deleted_at` | | |

> Indeks: `(status, tahap)`, `(unit_kerja_id, tanggal_berangkat)`,
> `(mata_anggaran_id)`, `(tenggat_spj)`, `(pemohon_id, status)`.
> Indeks *fulltext*: `(maksud, kota_tujuan)` untuk pencarian arsip.

**`perdin_pelaksana`**

| Kolom | Tipe |
|---|---|
| `perdin_id` | FK → `perdin` (CASCADE) |
| `pegawai_id` | FK → `pegawai` |
| `golongan` | VARCHAR(32) — disalin saat pengajuan |
| `ketua_rombongan` | BOOLEAN |
| `nomor_sppd` | VARCHAR(64) nullable |

> Indeks unik: `(perdin_id, pegawai_id)`.
> Indeks unik parsial ketua rombongan ditegakkan di aplikasi (MySQL tidak
> mendukung indeks unik parsial); gunakan kolom bantu
> `ketua_key = IF(ketua_rombongan, perdin_id, NULL)` dengan indeks unik.

**`perdin_persetujuan`** — keputusan beserta angka yang dibekukan

| Kolom | Tipe |
|---|---|
| `perdin_id` | FK → `perdin` |
| `lapis` | TINYINT UNSIGNED — 1, 2, … |
| `pemutus_id` | FK → `users` |
| `keputusan` | ENUM('setuju','setuju_dengan_catatan','kembalikan','tolak') |
| `catatan` | TEXT |
| `snapshot_pagu` | BIGINT UNSIGNED |
| `snapshot_blokir` | BIGINT UNSIGNED |
| `snapshot_realisasi` | BIGINT UNSIGNED |
| `snapshot_cadangan` | BIGINT UNSIGNED |
| `snapshot_sisa_efektif` | BIGINT UNSIGNED |
| `nilai_diajukan` | BIGINT UNSIGNED |
| `diputus_pada` | TIMESTAMP |

**`perdin_verifikasi`** — penetapan alokasi anggaran

| Kolom | Tipe | Keterangan |
|---|---|---|
| `id` | BIGINT UNSIGNED PK | |
| `perdin_id` | FK unik → `perdin` | satu penetapan berlaku per berkas |
| `verifikator_id` | FK → `users` | |
| `mata_anggaran_usulan_id` | FK → `mata_anggaran` | disalin saat verifikasi |
| `mata_anggaran_final_id` | FK → `mata_anggaran` | |
| `dipindahkan` | BOOLEAN | `true` bila akun final berbeda dari usulan |
| `alasan_pemindahan` | VARCHAR(500) nullable | wajib bila `dipindahkan` |
| `periksa_terdaftar_dipa` | BOOLEAN | |
| `periksa_sesuai_jenis` | BOOLEAN | |
| `periksa_tanpa_blokir` | BOOLEAN | |
| `periksa_pagu_cukup` | BOOLEAN | |
| `periksa_sesuai_rpd` | BOOLEAN | |
| `nomor_register_anggaran` | VARCHAR(32) | |
| `ditetapkan_pada` | TIMESTAMP | |

> Indeks: `(verifikator_id, ditetapkan_pada)`.

**`perdin_rincian`** — rencana biaya

| Kolom | Tipe | Keterangan |
|---|---|---|
| `perdin_id` | FK → `perdin` (CASCADE) | |
| `urutan` | SMALLINT UNSIGNED | |
| `komponen` | VARCHAR(160) | |
| `keterangan` | VARCHAR(255) nullable | |
| `sbm_tarif_id` | FK → `sbm_tarif` nullable | tarif acuan yang dipakai |
| `batas_sbm` | BIGINT UNSIGNED nullable | **disalin**, bukan dirujuk saat tampil |
| `volume` | DECIMAL(10,2) | |
| `satuan` | VARCHAR(24) | |
| `harga_satuan` | BIGINT UNSIGNED | |
| `jumlah` | BIGINT UNSIGNED | `ROUND(volume × harga_satuan)` |
| `melampaui_sbm` | BOOLEAN | |
| `alasan_melampaui` | VARCHAR(500) nullable | wajib bila `melampaui_sbm` |
| `disetujui_kabag_pada` | TIMESTAMP nullable | |

**`perdin_realisasi`**

| Kolom | Tipe |
|---|---|
| `perdin_rincian_id` | FK → `perdin_rincian` |
| `nilai_realisasi` | BIGINT UNSIGNED |
| `sifat` | ENUM('lumpsum','at_cost') |
| `status_bukti` | ENUM('belum','terlampir','perlu_ganti','sah') |
| `catatan_verifikasi` | VARCHAR(500) nullable |
| `diverifikasi_oleh` | FK → `users` nullable |

**`perdin_laporan`**

| Kolom | Tipe |
|---|---|
| `perdin_id` | FK unik → `perdin` |
| `ringkasan_pelaksanaan` | TEXT |
| `hasil_dicapai` | TEXT |
| `tindak_lanjut` | TEXT |
| `status` | ENUM('draf','diajukan','disetujui','dikembalikan') |
| `disetujui_oleh` | FK → `users` nullable |

### 12.5 Dokumen dan pembayaran

**`perdin_dokumen`**

| Kolom | Tipe | Keterangan |
|---|---|---|
| `perdin_id` | FK → `perdin` | |
| `jenis` | ENUM('surat_tugas','sppd','rincian_biaya','bukti_bayar','spj','laporan','kertas_kerja_selisih','lainnya') | |
| `nomor` | VARCHAR(64) nullable | |
| `versi` | TINYINT UNSIGNED | bertambah saat terbit ulang |
| `path` | VARCHAR(512) | pada penyimpanan objek |
| `hash_sha256` | CHAR(64) | penjamin keutuhan berkas |
| `status` | ENUM('draf','terbit','ditandatangani','perlu_perbaikan','terverifikasi') | |
| `ditandatangani_oleh` | FK → `users` nullable | |
| `ditandatangani_pada` | TIMESTAMP nullable | |
| `tte_referensi` | VARCHAR(128) nullable | identitas transaksi TTE |

> Indeks unik: `(perdin_id, jenis, versi)`.

**`pembayaran`**

| Kolom | Tipe | Keterangan |
|---|---|---|
| `perdin_id` | FK → `perdin` | |
| `nomor_voucher` | VARCHAR(32) UNIQUE | mis. `BKK/2026/08/0219` |
| `sifat` | ENUM('uang_muka','pelunasan_sisa','pengembalian') | |
| `mekanisme` | ENUM('up','tup','ls') | |
| `nilai` | BIGINT UNSIGNED | |
| `rekening_nama` | VARCHAR(160) | |
| `rekening_bank` | VARCHAR(64) | |
| `rekening_nomor` | VARCHAR(40) | disimpan terenkripsi |
| `jadwal_transfer` | DATE | |
| `jatuh_tempo` | DATE nullable | untuk pengembalian |
| `nomor_spm` | VARCHAR(32) nullable | |
| `nomor_sp2d` | VARCHAR(32) nullable | |
| `tanggal_sp2d` | DATE nullable | pemicu realisasi |
| `status` | ENUM('draf','siap_bayar','dibayar','gagal','lunas','telat') | |
| `dieksekusi_oleh` | FK → `users` nullable | |

### 12.6 Sistem

**`audit_log`** — *append-only*, tanpa `updated_at`

| Kolom | Tipe |
|---|---|
| `id` | BIGINT UNSIGNED PK |
| `auditable_type` | VARCHAR(160) |
| `auditable_id` | BIGINT UNSIGNED |
| `user_id` | FK → `users` nullable |
| `peran` | VARCHAR(32) |
| `aksi` | VARCHAR(64) |
| `nilai_lama` | JSON nullable |
| `nilai_baru` | JSON nullable |
| `catatan` | VARCHAR(500) nullable |
| `ip` | VARBINARY(16) |
| `user_agent` | VARCHAR(255) |
| `created_at` | TIMESTAMP |

> Indeks: `(auditable_type, auditable_id)`, `(user_id, created_at)`.

**`nomor_urut`**

| Kolom | Tipe |
|---|---|
| `jenis` | ENUM('perdin','surat_tugas','sppd','voucher') |
| `tahun` | SMALLINT UNSIGNED |
| `bulan` | TINYINT UNSIGNED |
| `terakhir` | INT UNSIGNED |

> Indeks unik: `(jenis, tahun, bulan)`.

**`hari_libur`** — dasar perhitungan hari kerja untuk SLA dan tenggat SPJ

| Kolom | Tipe |
|---|---|
| `tanggal` | DATE UNIQUE |
| `keterangan` | VARCHAR(160) |

---

## 13. Struktur aplikasi Laravel

### 13.1 Susunan direktori

```
app/
├── Console/Commands/
│   ├── SinkronDipa.php                 # tarik pagu & realisasi dari sistem keuangan
│   ├── IngatkanSlaPersetujuan.php
│   ├── IngatkanTenggatSpj.php
│   ├── PeringatanSbmBelumDiunggah.php
│   └── BangunUlangSaldoAnggaran.php    # rekonsiliasi dari buku besar
├── Domain/
│   ├── Anggaran/
│   │   ├── AnggaranService.php         # cadangkan, pindahkanCadangan, lepasCadangan, realisasikan
│   │   ├── SisaEfektif.php             # objek nilai
│   │   └── Exceptions/PaguTidakCukup.php
│   ├── Sbm/
│   │   ├── SbmResolver.php             # versi berlaku untuk tanggal berangkat
│   │   ├── PemeriksaTarif.php          # batas & putusan kesesuaian per baris
│   │   └── ImporSbm.php
│   ├── Perdin/
│   │   ├── States/                     # PerdinState + 13 kelas status
│   │   ├── Actions/
│   │   │   ├── KirimPengajuan.php
│   │   │   ├── PutuskanPersetujuan.php
│   │   │   ├── TetapkanAlokasiAnggaran.php
│   │   │   ├── TerbitkanBerkas.php
│   │   │   ├── SimpanRincian.php
│   │   │   ├── SimpanRealisasi.php
│   │   │   ├── HitungSelisih.php
│   │   │   └── TutupBerkas.php
│   │   └── TenggatService.php
│   ├── Dokumen/
│   │   ├── PenerbitSuratTugas.php
│   │   ├── PenerbitSppd.php
│   │   ├── PenandatanganElektronik.php
│   │   └── PaketAudit.php
│   └── Nomor/NomorService.php
├── Http/
│   ├── Controllers/                    # tipis: hanya merutekan ke Action
│   ├── Requests/                       # validasi masukan
│   └── Middleware/
├── Livewire/
│   ├── Perdin/FormPengajuan.php        # cek pagu langsung
│   ├── Perdin/RelTahap.php
│   ├── Perdin/TabelRincian.php         # validasi SBM langsung
│   ├── Perdin/TabelRealisasi.php
│   ├── Persetujuan/LembarCekPagu.php
│   ├── Verifikasi/PenetapanAlokasi.php  # banding usulan vs penetapan
│   ├── Sbm/KelolaTarif.php
│   └── Keuangan/AntreanPembayaran.php
├── Models/
├── Policies/
│   ├── PerdinPolicy.php
│   ├── PersetujuanPolicy.php
│   ├── VerifikasiPolicy.php
│   ├── PembayaranPolicy.php
│   └── SbmPolicy.php
├── Enums/
│   ├── JenisPerjalanan.php
│   ├── KategoriSbm.php
│   ├── SifatPembayaran.php
│   └── JenisMutasi.php
├── Notifications/
├── Exports/  Imports/                  # maatwebsite/excel
└── Observers/AuditObserver.php
```

### 13.2 Perpustakaan pihak ketiga

| Kebutuhan | Paket | Alasan |
|---|---|---|
| Peran dan izin | `spatie/laravel-permission` | Enam peran dengan izin berbutir halus |
| Mesin status | `spatie/laravel-model-states` | Dua belas tahap dengan transisi yang harus ditegakkan |
| Berkas lampiran | `spatie/laravel-medialibrary` | Konversi, versi, penyimpanan objek |
| PDF | `barryvdh/laravel-dompdf` | Surat Tugas & SPPD ditulis sebagai Blade, mudah dirawat Bagian Umum |
| XLSX | `maatwebsite/excel` | Impor SBM, ekspor seluruh daftar dan laporan |
| Antrean | `laravel/horizon` | Pemantauan tugas latar: PDF, notifikasi, sinkronisasi |
| Audit | `owen-it/laravel-auditing` | Dasar; ditambah `audit_log` khusus untuk aksi bisnis |
| Pengujian | `pestphp/pest` | Uji fitur per kriteria penerimaan |
| Mutu kode | `laravel/pint`, `larastan/larastan` | Gaya seragam dan analisis statis tingkat 6+ |

### 13.3 Peristiwa dan pendengarnya

| Peristiwa | Pendengar |
|---|---|
| `PengajuanDikirim` | Beri tahu pimpinan · mulai penghitung SLA |
| `PengajuanDisetujui` | Cadangkan pagu pada akun usulan · beri tahu Verifikator Anggaran |
| `AlokasiAnggaranDitetapkan` | Pindahkan cadangan bila akun berubah · terbitkan register anggaran · beri tahu Kabag |
| `PengajuanDitolak` / `PerdinDibatalkan` | Lepas cadangan · beri tahu pemohon |
| `BerkasDiterbitkan` | Bekukan versi SBM · bangkitkan Surat Tugas & SPPD · kunci rincian |
| `UangMukaDibayar` | Perbarui status · jadwalkan pengingat tenggat SPJ |
| `SpjDinyatakanRampung` | Hitung selisih · terbitkan instruksi bayar atau tagihan |
| `Sp2dDicatat` | Pindahkan cadangan → realisasi pada buku besar |
| `BerkasDitutup` | Kunci dokumen · masukkan ke arsip |
| `SbmVersiDiberlakukan` | Tandai draf yang tarifnya tidak lagi sesuai · beri tahu pemohonnya |
| `DipaRevisiDisahkan` | Nilai ulang berkas berstatus `pagu_terkunci` |

### 13.4 Tugas terjadwal

| Perintah | Jadwal | Tujuan |
|---|---|---|
| `perdin:sinkron-dipa` | tiap 6 jam | Tarik pagu, blokir, dan realisasi terbaru |
| `perdin:ingatkan-sla` | tiap jam kerja | Peringatan dan eskalasi persetujuan lewat SLA |
| `perdin:ingatkan-spj` | harian 07.00 | H-2, H-0, dan lewat tenggat |
| `perdin:peringatan-sbm` | harian, sejak 1 Des | SBM tahun berikutnya belum diunggah |
| `perdin:bangun-saldo` | harian 01.00 | Rekonsiliasi ringkasan saldo dengan buku besar; laporkan selisih |

---

## 14. Rute dan halaman

| Rute | Halaman | Peran |
|---|---|---|
| `GET /` | Beranda — KPI, antrean tugas, pagu DIPA, perdin terbaru | Semua |
| `GET /perdin` | Daftar perdin dengan saringan | Semua |
| `GET /perdin/create` · `POST /perdin` | Formulir pengajuan | Pemohon |
| `GET /perdin/{perdin}` | Detail berkas — rel 13 tahap | Semua (sesuai lingkup) |
| `POST /perdin/{perdin}/kirim` | Kirim ke pimpinan | Pemohon |
| `GET /persetujuan` | Kotak persetujuan | Pimpinan |
| `POST /perdin/{perdin}/putuskan` | Setujui/kembalikan/tolak | Pimpinan |
| `GET /verifikasi` | Antrean verifikasi anggaran | Verifikator |
| `POST /perdin/{perdin}/alokasi` | Tetapkan akun definitif & register anggaran | Verifikator |
| `POST /perdin/{perdin}/terbitkan` | Eksekusi Kabag | Kabag |
| `GET/POST /perdin/{perdin}/rincian` | Rincian uang muka | Pemohon, Kabag |
| `GET /perdin/{perdin}/dokumen/{jenis}` | Unduh dokumen | Sesuai lingkup |
| `GET/POST /perdin/{perdin}/realisasi` | Pertanggungjawaban | Pemohon |
| `POST /perdin/{perdin}/spj-rampung` | Nyatakan rampung | Bendahara |
| `GET/POST /perdin/{perdin}/laporan` | Laporan perdin | Pemohon, Kabag |
| `POST /perdin/{perdin}/selisih` | Sahkan selisih | Bendahara |
| `GET /keuangan/pembayaran` | Antrean pembayaran | Bendahara |
| `POST /pembayaran/{pembayaran}/eksekusi` | Bayar & catat SP2D | Bendahara |
| `GET /anggaran/dipa` | Pagu & DIPA | Semua (baca) |
| `POST /anggaran/dipa/impor` | Unggah DIPA/revisi | Admin Anggaran |
| `GET /referensi/sbm` | Standar Biaya | Semua (baca) |
| `POST /referensi/sbm` | Unggah SBM tahun baru | Admin Anggaran |
| `GET /arsip` | Arsip dokumen | Semua, Auditor |
| `GET /arsip/{perdin}/paket-audit` | Unduh paket audit | Auditor, Kabag |
| `GET /laporan/*` | Laporan pengelola | Pimpinan, Kabag, Bendahara |
| `GET /verifikasi/{kode}` | Verifikasi keaslian dokumen | Publik |

Antarmuka mesin-ke-mesin (Fase 3) memakai `routes/api.php` dengan
`laravel/sanctum` dan pembatasan laju permintaan.

---

## 15. Integrasi

| Sistem | Arah | Data | Prioritas |
|---|---|---|---|
| Sistem keuangan negara (SAKTI/OM-SPAN) | Masuk | Pagu DIPA per akun, revisi, realisasi SP2D | Wajib |
| Sistem keuangan negara | Keluar | Data SPP/SPM perjalanan dinas | Sebaiknya |
| Tanda tangan elektronik tersertifikasi (BSrE) | Dua arah | Penandatanganan Surat Tugas, SPPD, dokumen bayar | Wajib |
| Sistem kepegawaian | Masuk | Pegawai, jabatan, golongan, unit kerja, rekening | Wajib |
| Perbankan | Keluar | Daftar transfer; *host-to-host* pada fase lanjutan | Sebaiknya |
| Surel dan pesan instan | Keluar | Pemberitahuan tenggat, keputusan, permintaan tindakan | Wajib |
| Penyimpanan objek | Dua arah | Dokumen dan lampiran, terenkripsi saat diam | Wajib |

**Aturan integrasi.** Setiap integrasi wajib memiliki **jalur cadangan manual**
berupa unggahan berkas, agar sistem tetap dapat dioperasikan sejak hari pertama
dan tidak berhenti bila layanan luar tidak tersedia. Seluruh panggilan keluar
dijalankan sebagai tugas antrean dengan percobaan ulang bertahap.

---

## 16. Keamanan dan jejak audit

| ID | Kebutuhan |
|---|---|
| SEC-01 | Otentikasi terpusat; kata sandi mengikuti kebijakan lembaga; otentikasi dua langkah wajib bagi Pimpinan, Bendahara, dan Admin Anggaran |
| SEC-02 | Otorisasi berbasis peran dan kebijakan (`Policy`) pada setiap aksi; tidak ada pemeriksaan wewenang yang hanya di antarmuka |
| SEC-03 | Nomor rekening dan data pribadi disimpan terenkripsi (`encrypted` cast) |
| SEC-04 | Berkas dokumen tidak dapat diakses langsung melalui URL publik; unduhan melalui rute bertanda tangan berumur pendek |
| SEC-05 | Pengguna basis data aplikasi tidak memiliki hak `UPDATE`/`DELETE` pada `audit_log` |
| SEC-06 | Setiap aksi bisnis mencatat aktor, peran, waktu, nilai sebelum dan sesudah, serta alamat IP |
| SEC-07 | Pembatasan laju pada rute masuk dan unggah |
| SEC-08 | Unggahan diperiksa jenis MIME sebenarnya, bukan ekstensi; dipindai antivirus bila tersedia |
| SEC-09 | Kebijakan Keamanan Konten diaktifkan; seluruh keluaran Blade lolos *escape* secara bawaan |
| SEC-10 | Rahasia disimpan pada berkas lingkungan, tidak pernah masuk repositori |
| SEC-11 | Setiap perubahan peran pengguna dicatat pada jejak audit |
| SEC-12 | Uji penetrasi sebelum rilis produksi |

---

## 17. Kebutuhan non-fungsional

| ID | Kebutuhan | Ukuran |
|---|---|---|
| NF-01 | Waktu muat layar kerja | ≤ 2 detik pada jaringan kantor |
| NF-02 | Cek pagu dan validasi SBM terasa seketika | ≤ 300 ms setelah nilai berubah |
| NF-03 | Ketersediaan pada jam kerja | ≥ 99,5% |
| NF-04 | Dapat dioperasikan dari telepon genggam | Seluruh layar pemohon, terutama pengisian SPJ dari lapangan |
| NF-05 | Aksesibilitas | Kontras memadai; status tidak disampaikan hanya lewat warna; seluruh alur dapat dijalankan dengan papan tik |
| NF-06 | Jejak audit | Seluruh aksi ubah tercatat lengkap |
| NF-07 | Retensi | Berkas dan dokumen disimpan sekurang-kurangnya 10 tahun |
| NF-08 | Skala | 2.000 berkas per tahun, 200 pengguna, 60 pengguna serentak pada puncak |
| NF-09 | Pencadangan | Basis data harian dengan `binlog` untuk pemulihan titik waktu; penyimpanan objek direplikasi; uji pemulihan triwulanan |
| NF-10 | Bahasa | Antarmuka berbahasa Indonesia; istilah keuangan negara sesuai peraturan |
| NF-11 | Waktu | Seluruh waktu dalam WIB; tanggal dalam format Indonesia |
| NF-12 | Ukuran unggahan | Maksimal 10 MB per berkas; PDF, JPG, PNG |
| NF-13 | Peramban | Dua versi terakhir Chrome, Edge, Firefox, dan Safari |

---

## 18. Pengujian dan kriteria penerimaan

### 18.1 Lapisan pengujian

| Lapisan | Cakupan | Alat |
|---|---|---|
| Unit | `AnggaranService`, `PemeriksaTarif`, `SelisihService`, `TenggatService`, `NomorService` | Pest |
| Fitur | Setiap kriteria penerimaan di bawah, melalui HTTP dan Livewire | Pest + `Livewire::test()` |
| Kebijakan | Matriks peran × aksi lengkap | Pest |
| Serentak | Dua persetujuan bersamaan pada akun yang sama | Pest + transaksi paralel |
| Ujung ke ujung | Alur 13 tahap satu berkas | Laravel Dusk |

**Ambang mutu.** Cakupan pengujian pada `app/Domain` ≥ 90%. Analisis statis
Larastan tingkat 6 tanpa galat. Setiap kebutuhan berprioritas *Wajib* memiliki
sekurang-kurangnya satu uji fitur yang menyebut ID kebutuhannya.

### 18.2 Kriteria penerimaan kritis

**AC-01 — Persetujuan terkunci saat pagu tidak tersedia** *(F-203)*
> **Diberikan** akun `524211` diblokir seluruhnya pada Halaman IV DIPA,
> **ketika** Pimpinan membuka pengajuan yang membebani akun tersebut,
> **maka** lembar cek pagu menampilkan putusan *pagu tidak tersedia* beserta
> alasannya, tombol **Setujui** nonaktif disertai keterangan, dan tombol
> *Kembalikan untuk revisi* tetap dapat digunakan.

**AC-02 — Cadangan terbentuk saat persetujuan** *(F-205, BR-02)*
> **Diberikan** sisa efektif akun `524111` sebesar Rp 776.620.000,
> **ketika** pengajuan senilai Rp 9.020.000 disetujui,
> **maka** satu baris `anggaran_mutasi` berjenis `cadangan` terbentuk, ringkasan
> saldo bertambah Rp 9.020.000, sisa efektif menjadi Rp 767.600.000, dan
> `perdin_persetujuan` menyimpan salinan angka tersebut.

**AC-03 — Dua persetujuan serentak tidak boleh melampaui pagu** *(11.3)*
> **Diberikan** sisa efektif Rp 10.000.000 dan dua pengajuan masing-masing
> Rp 8.000.000 pada akun yang sama,
> **ketika** keduanya disetujui pada saat yang sama,
> **maka** tepat satu berhasil dan satu lagi gagal dengan `PaguTidakCukup`;
> sisa efektif akhir tidak pernah negatif.

**AC-04 — Rincian melampaui SBM mengunci pengiriman** *(F-515, F-516)*
> **Diberikan** batas SBM penginapan Prov. Sulawesi Selatan Rp 1.000.000,
> **ketika** pemohon mengisi harga satuan Rp 1.400.000,
> **maka** baris ditandai melampaui batas sebesar Rp 400.000, kolom alasan
> terbuka dan wajib diisi, dan aksi *Ajukan ke Bendahara* ditolak peladen
> selama alasan kosong — bukan hanya dinonaktifkan di antarmuka.

**AC-11 — Pemindahan akun memindahkan cadangan** *(F-323)*
> **Diberikan** berkas disetujui dengan usulan akun `524119` sehingga cadangannya
> terbentuk di sana,
> **ketika** verifikator menetapkan akun definitif `524111` disertai alasan,
> **maka** terbentuk sepasang mutasi `pindah_keluar` pada `524119` dan
> `pindah_masuk` pada `524111` dalam satu transaksi, sisa efektif `524119`
> bertambah kembali sebesar nilai berkas, sisa efektif `524111` berkurang sebesar
> nilai yang sama, dan jumlah beban keseluruhan tidak berubah.

**AC-12 — Surat Tugas terkunci sebelum alokasi** *(F-329, BR-15)*
> **Diberikan** berkas yang sudah disetujui pimpinan tetapi belum diverifikasi,
> **ketika** Kabag mencoba menerbitkan Surat Tugas,
> **maka** aksi ditolak kebijakan dengan alasan *alokasi anggaran belum
> ditetapkan*, dan tidak ada nomor Surat Tugas yang terpakai.

**AC-13 — Pemindahan akun wajib beralasan** *(F-324)*
> **Diberikan** verifikator memilih akun yang berbeda dari usulan,
> **ketika** ia menetapkan alokasi tanpa mengisi alasan,
> **maka** peladen menolak dengan galat validasi pada kolom alasan.

**AC-05 — Versi SBM dibekukan** *(F-508)*
> **Diberikan** berkas yang Surat Tugasnya terbit dengan SBM TA 2026,
> **ketika** Admin Anggaran memberlakukan SBM TA 2027,
> **maka** `perdin.sbm_versi_id` tidak berubah, pengujian rincian dan realisasi
> tetap memakai tarif TA 2026, sedangkan berkas berstatus `draf` ditandai dan
> pemohonnya menerima pemberitahuan.

**AC-06 — Selisih dua arah** *(F-707)*
> **Diberikan** uang muka Rp 6.765.000,
> **ketika** realisasi disahkan sebesar Rp 8.675.000,
> **maka** terbit `pembayaran` bersifat `pelunasan_sisa` senilai Rp 1.910.000;
> **dan ketika** realisasi disahkan sebesar Rp 5.900.000,
> **maka** terbit `pembayaran` bersifat `pengembalian` senilai Rp 865.000
> beserta jatuh tempo dan kode setornya.

**AC-07 — Realisasi hanya setelah SP2D** *(BR-03)*
> **Diberikan** uang muka sudah ditransfer dari UP,
> **ketika** saldo akun diperiksa,
> **maka** nilainya masih tercatat sebagai `cadangan`; nilai berpindah ke
> `realisasi` hanya setelah nomor dan tanggal SP2D dicatat.

**AC-08 — Penutupan berkas** *(F-803, F-804)*
> **Diberikan** berkas dengan SPJ rampung, laporan disetujui, dan selisih disahkan,
> **ketika** pembayaran sisa dieksekusi dan SP2D dicatat,
> **maka** berkas berstatus `selesai`, `dikunci_pada` terisi, seluruh upaya
> pengubahan ditolak kebijakan, dan dokumen tetap dapat diunduh.

**AC-09 — Nomor berurutan tanpa lompatan** *(BR-11)*
> **Diberikan** sepuluh dokumen diterbitkan bersamaan,
> **ketika** nomor dibangkitkan,
> **maka** sepuluh nomor berurutan tanpa duplikat dan tanpa nomor terlewat.

**AC-10 — Jejak audit tidak dapat diubah** *(SEC-05)*
> **Diberikan** satu baris `audit_log`,
> **ketika** aplikasi mencoba memperbarui atau menghapusnya,
> **maka** basis data menolak karena hak akses tidak diberikan.

---

## 19. Penyebaran dan lingkungan

### 19.1 Kebutuhan peladen

| Komponen | Spesifikasi minimum |
|---|---|
| Aplikasi | PHP 8.3+, Nginx + PHP-FPM, 4 vCPU, 8 GB |
| Basis data | MySQL 8.0+, 4 vCPU, 16 GB, penyimpanan SSD, `binlog` aktif |
| Antrean & tembolok | Redis 7+ |
| Penyimpanan objek | S3 atau MinIO, kapasitas awal 500 GB |
| Pekerja latar | Supervisor menjalankan Horizon |
| Penjadwal | Cron memanggil `schedule:run` tiap menit |

### 19.2 Lingkungan

| Lingkungan | Tujuan | Data |
|---|---|---|
| Lokal | Pengembangan | Data benih |
| Uji | Pengujian otomatis dan tinjauan | Data benih |
| Pratayang | Penerimaan pengguna | Salinan produksi yang disamarkan |
| Produksi | Operasional | Sebenarnya |

### 19.3 Kunci lingkungan utama

```dotenv
APP_TIMEZONE=Asia/Jakarta
APP_LOCALE=id
DB_CONNECTION=mysql
DB_CHARSET=utf8mb4
DB_COLLATION=utf8mb4_unicode_ci
QUEUE_CONNECTION=redis
FILESYSTEM_DISK=s3
PERDIN_UANG_MUKA_PERSEN_MAKS=80
PERDIN_TENGGAT_SPJ_HARI_KERJA=5
PERDIN_SLA_PERSETUJUAN_JAM=48
PERDIN_SLA_VERIFIKASI_JAM=24
PERDIN_AMBANG_PERSETUJUAN_LAPIS_2=25000000
SAKTI_BASE_URL=
BSRE_BASE_URL=
HRIS_BASE_URL=
```

Seluruh ambang kebijakan berada pada `config/perdin.php` dan dapat diubah tanpa
menyentuh kode — karena angka-angka ini memang berubah mengikuti peraturan.

### 19.4 Alur rilis

1. Gabungan ke cabang utama memicu jalur otomasi: Pint → Larastan → Pest.
2. Penyebaran tanpa henti; `php artisan migrate --force` dijalankan pada langkah
   terpisah yang dapat ditinjau.
3. Migrasi yang memuat perubahan struktur tabel besar dijalankan di luar jam kerja.
4. `php artisan perdin:bangun-saldo --periksa` dijalankan setelah rilis untuk
   memastikan ringkasan saldo cocok dengan buku besar.

---
---

# Bagian III — Pelaksanaan

## 20. Rencana rilis

| Fase | Cakupan | Hasil yang bisa dipakai |
|---|---|---|
| **Fase 1 — Inti** | Tahap 01–08 · DIPA unggah manual · SBM · peran & kebijakan · Surat Tugas & SPPD · jejak audit | Perdin dapat diajukan, disetujui dengan cek pagu, diterbitkan surat, dan dibayar uang mukanya |
| **Fase 2 — Penutupan siklus** | Tahap 09–13 · arsip dokumen · laporan pengelola · ekspor | Siklus penuh sampai pelunasan dan penutupan berkas |
| **Fase 3 — Integrasi** | Sinkronisasi sistem keuangan negara, kepegawaian, TTE penuh, *host-to-host* perbankan | Pengurangan entri ganda dan rekonsiliasi manual |
| **Fase 4 — Penajaman** | Persetujuan massal · pembacaan bukti otomatis · papan analitik lanjutan | Percepatan pekerjaan berulang |

---

## 21. Migrasi data awal

| Data | Sumber | Cara | Penanggung jawab |
|---|---|---|---|
| Pegawai, jabatan, golongan, unit kerja | Sistem kepegawaian | Impor XLSX pada Fase 1, antarmuka pada Fase 3 | Bagian Umum |
| DIPA tahun berjalan beserta revisinya | Cetakan DIPA / RKA-K/L | Impor XLSX dengan templat resmi | Admin Anggaran |
| SBM tahun berjalan dan tahun sebelumnya | PMK terkait | Impor XLSX; tahun sebelumnya diperlukan untuk kolom pembanding | Admin Anggaran |
| Hari libur nasional | Surat Keputusan Bersama | Benih tahunan | Admin Sistem |
| Perdin berjalan yang belum selesai | Berkas kertas | Entri manual pada tahap yang sesuai, disertai penanda *migrasi* | Bagian Umum |
| Perdin selesai tahun-tahun sebelumnya | Arsip | **Tidak dimigrasikan**; arsip lama tetap di tempatnya | — |

---

## 22. Risiko dan mitigasi

| Risiko | Dampak | Mitigasi |
|---|---|---|
| Integrasi sistem keuangan negara tertunda | Pagu tidak mutakhir; keputusan salah | Unggahan manual DIPA sejak Fase 1, dengan penanda waktu sinkronisasi terakhir yang tampil di layar keputusan |
| Admin terlambat mengunggah SBM tahun baru | Pengajuan awal tahun tertahan | Peringatan otomatis sejak 1 Desember; peran cadangan admin |
| Dua persetujuan serentak melampaui pagu | Temuan audit | Penguncian baris pada transaksi pencadangan; uji serentak wajib |
| Verifikasi menjadi leher botol baru | Surat Tugas terlambat, perjalanan mendesak terhambat | SLA verifikasi 1×24 jam dengan eskalasi; verifikator cadangan; penetapan massal untuk berkas yang usulannya sudah benar |
| Ringkasan saldo menyimpang dari buku besar | Angka keputusan salah | Rekonsiliasi harian otomatis dengan laporan selisih |
| Pengguna menghindari sistem saat mendesak | Berkas tidak lengkap | Jalur perdin mendesak yang tetap tercatat, dengan pengesahan menyusul berbatas waktu |
| Kualitas bukti pindai buruk | SPJ bolak-balik | Pemeriksaan keterbacaan saat unggah, contoh bukti yang baik, opsi surat pernyataan riil |
| Perubahan peraturan perjalanan dinas | Aturan usang | Tarif dan ambang disimpan sebagai data dan konfigurasi, bukan ditanam dalam kode |
| Perpindahan dari cara lama | Adopsi lambat | Pendampingan per unit kerja; berjalan paralel satu bulan |

---

## 23. Asumsi

1. Seluruh perjalanan dinas dalam ruang lingkup ini dibiayai APBN melalui DIPA
   satker BAZNAS; tidak ada jalur pembiayaan lain pada rilis pertama.
2. Uang muka dibayarkan dari **Uang Persediaan** oleh Bendahara Pengeluaran,
   bukan melalui pembayaran langsung ke rekening pelaksana.
3. Satu rombongan memiliki satu pemegang uang muka.
4. Tanda tangan elektronik tersertifikasi sudah tersedia bagi pejabat penanda tangan.
5. Data pegawai, jabatan, dan golongan tersedia dari sistem kepegawaian.
6. Kode satker, Bagian Anggaran, struktur kegiatan, dan seluruh tarif pada
   prototipe adalah **contoh**, dan diganti dengan data sebenarnya saat implementasi.
7. Tim pengembang menguasai Laravel; karena itu Livewire dipilih daripada
   kerangka kerja antarmuka terpisah.

---

## 24. Pertanyaan terbuka

| # | Pertanyaan | Kepada | Dampak bila belum terjawab |
|---|---|---|---|
| Q-1 | Apakah perjalanan dinas yang dibiayai **Hak Amil** (di luar APBN) juga masuk ruang lingkup? Bila ya, apa gerbang anggarannya, karena tidak ada DIPA | Pimpinan, Bagian Keuangan | Menentukan apakah model anggaran perlu dua jalur |
| Q-2 | Berapa ambang nilai yang memicu lapis persetujuan tambahan Ketua BAZNAS? | Pimpinan | `PERDIN_AMBANG_PERSETUJUAN_LAPIS_2` belum dapat diisi |
| Q-3 | Apakah uang muka benar dibayar dari UP, atau ada skema LS untuk nilai besar? | Bendahara | Memengaruhi F-605 dan pelaporan realisasi |
| Q-4 | Berapa tenggat pengembalian kelebihan uang muka, dan apa konsekuensinya bila lewat? | Bagian Keuangan, SAI | F-606 belum lengkap |
| Q-5 | Apakah SPPD lembar II tetap wajib dicap basah, atau dapat digantikan pengesahan elektronik? | SAI, Bagian Umum | Menentukan apakah alur cetak masih diperlukan |
| Q-6 | Siapa peran cadangan Admin Anggaran saat pemegangnya berhalangan? | Bagian Umum | Risiko SBM tidak terbarui |
| Q-7 | Apakah tersedia antarmuka pertukaran data resmi untuk pagu dan realisasi dari sistem keuangan negara? | Tim TI, Kemenkeu | Menentukan lingkup Fase 3 |
| Q-8 | Bagaimana perlakuan perjalanan dinas yang dibatalkan setelah tiket terlanjur dibeli? | Bagian Keuangan | Aturan pelepasan cadangan dan penggantian biaya hangus |
| Q-9 | Apakah aplikasi ditempatkan di pusat data lembaga atau layanan awan? | Tim TI | Menentukan rancangan penyebaran dan pencadangan |
| Q-10 | Apakah sudah ada penyedia SSO lembaga yang harus dipakai? | Tim TI | Menentukan rancangan otentikasi |
| Q-11 | Siapa yang memegang peran Verifikator Anggaran — staf Bagian Keuangan, PPK, atau PPSPM? | Bagian Keuangan | Menentukan pemisahan wewenang dan siapa cadangannya |
| Q-12 | Berapa SLA verifikasi anggaran yang wajar, dan apa yang terjadi bila terlampaui? | Bagian Keuangan | `PERDIN_SLA_VERIFIKASI_JAM` belum dapat diisi |
| Q-13 | Apakah verifikator boleh mengubah nilai pengajuan, atau hanya akunnya? Dokumen ini mengasumsikan hanya akun | Bagian Keuangan, SAI | Menentukan lingkup F-331 |

---

## 25. Rujukan

- Undang-Undang Nomor 23 Tahun 2011 tentang Pengelolaan Zakat
- Peraturan Menteri Keuangan tentang Perjalanan Dinas Dalam Negeri bagi Pejabat
  Negara, Pegawai Negeri Sipil, dan Pegawai Tidak Tetap
- Peraturan Menteri Keuangan tentang Perjalanan Dinas Luar Negeri
- Peraturan Menteri Keuangan tentang Standar Biaya Masukan (terbit tiap tahun anggaran)
- Peraturan Direktur Jenderal Perbendaharaan tentang Mekanisme UP, TUP, dan GUP

> Nomor peraturan sengaja tidak dicantumkan di sini karena berubah tiap tahun.
> Nomor yang berlaku dicatat pada data induk SBM dan DIPA di dalam sistem, bukan
> ditanam dalam dokumen ini maupun dalam kode.

---

## 26. Lampiran — matriks tanggung jawab

**E** = eksekutor · **P** = pemutus · **V** = verifikator · **L** = dilihat saja

| # | Tahap | Pemohon | Pimpinan | Verifikator | Kabag | Bendahara | Admin |
|---|---|:---:|:---:|:---:|:---:|:---:|:---:|
| 01 | Pengajuan Perdin | E | L | — | L | — | — |
| 02 | Persetujuan Pimpinan | L | P | L | L | V | — |
| 03 | Verifikasi & Alokasi Anggaran | L | L | E | L | V | L |
| 04 | Eksekusi Kabag | L | L | L | E | L | — |
| 05 | Surat Tugas | L | L | L | E | L | — |
| 06 | Rincian Uang Muka | E | L | V | V | V | — |
| 07 | Dokumen SPPD | L | L | — | E | L | — |
| 08 | Pembayaran Uang Muka | L | — | L | L | E | — |
| 09 | Pertanggungjawaban | E | — | L | L | V | — |
| 10 | Upload SPJ Rampung | E | — | L | L | P | — |
| 11 | Laporan Perdin | E | L | — | P | — | — |
| 12 | Perhitungan Selisih | L | — | V | L | E | — |
| 13 | Pembayaran Sisa | L | — | L | L | E | — |
| — | Data induk SBM | L | L | L | L | L | E |
| — | Data induk DIPA | L | L | V | L | V | E |
