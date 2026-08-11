# SIPERDIN BAZNAS — Desain Antarmuka Perjalanan Dinas

Prototipe UI/UX untuk aplikasi perjalanan dinas BAZNAS, mencakup seluruh siklus
berkas dari pengajuan sampai pembayaran sisa. Satu berkas HTML mandiri, tanpa
dependensi eksternal — cukup dibuka di peramban.

```
open index.html
```

## Cakupan — 12 tahap

Rel proses adalah tulang punggung aplikasi. Setiap tahap punya layar kerjanya
sendiri, penanggung jawab yang jelas, dan syarat yang harus dipenuhi sebelum
berkas boleh berpindah.

| # | Tahap | Penanggung jawab | Syarat berpindah |
|---|---|---|---|
| 01 | Pengajuan Perdin | Pemohon | Lampiran dasar penugasan terunggah |
| 02 | Persetujuan Pimpinan | Pimpinan | Keputusan disertai catatan |
| 03 | Eksekusi Kabag | Kepala Bagian | Empat butir daftar periksa tercentang |
| 04 | Surat Tugas | Kepala Bagian | Tanda tangan elektronik terbubuh |
| 05 | Rincian Uang Muka | Pemohon (verifikasi Kabag & Bendahara) | Seluruh baris sesuai SBM atau beralasan |
| 06 | Dokumen SPPD | Kepala Bagian | SPPD terbit untuk tiap pelaksana |
| 07 | Pembayaran Uang Muka | Bendahara | Bukti transfer terunggah |
| 08 | Pertanggungjawaban Perdin | Pemohon | Tiap komponen punya bukti sah |
| 09 | Upload SPJ Rampung | Bendahara | Tidak ada dokumen bermasalah |
| 10 | Dokumen Laporan Perdin | Pemohon | Ringkasan, hasil, dan tindak lanjut terisi |
| 11 | Perhitungan Selisih Uang Muka | Bendahara | Selisih disahkan bendahara |
| 12 | Pembayaran Sisa Perdin | Bendahara | Tiga syarat penutupan terpenuhi |

## Delapan layar

- **Beranda** — KPI, antrean tugas berdasarkan tenggat, serapan anggaran, perdin terbaru
- **Daftar Perdin** — tabel dengan kolom posisi berkas pada rel 12 tahap
- **Kotak Persetujuan** — layar keputusan pimpinan, lengkap dengan cek pagu dan SLA
- **Detail Berkas** — rel 12 tahap + panel kerja tiap tahap + linimasa audit
- **Arsip Dokumen** — semua dokumen terikat nomor berkas dan tahap penerbitnya
- **Kas & Pembayaran** — layar kerja bendahara: uang muka, pelunasan, pengembalian
- **Alur & Peran** — matriks tanggung jawab dan kamus status
- **Panduan Desain** — token, komponen, aturan penulisan, catatan aksesibilitas

## Yang bisa dicoba

- **Ganti peran** di kanan atas (Pemohon / Pimpinan / Kabag / Bendahara) — tombol
  di luar wewenang menjadi nonaktif, bukan hilang, sehingga pengguna tetap tahu
  langkah berikutnya dan siapa yang berwenang.
- **Tahap 05** — ubah volume, harga satuan, atau porsi uang muka; total rencana
  biaya, nilai uang muka, dan nilai tertahan terhitung ulang seketika.
- **Tahap 08** — ubah nilai realisasi; kolom selisih berganti warna dan arah, dan
  kotak hitung berpindah antara "BAZNAS masih harus membayar" dan "wajib
  dikembalikan pelaksana".
- **Klik tahap mana pun** pada rel proses untuk membuka layar kerjanya.

## Bahasa desain

**Warna.** Satu aksen — biru amanah `#0F5FA6`. Kuningan `#8C6420` dipakai khusus
untuk penanda dokumen resmi (nomor surat, meterai elektronik, uang muka yang
menahan pagu), bukan sebagai aksen kedua. Warna semantik berdiri sendiri: hijau
sah/lunas, kuning perlu tindakan, merah telat/ditolak. Netral berbias biru dingin
agar sebidang dengan aksen.

**Tipografi.** Sans sistem untuk antarmuka; **serif untuk pratinjau dokumen**
(Surat Tugas dan SPPD) karena begitulah surat dinas terlihat di atas kertas —
isyarat bahwa berkas itu akan dicetak dan ditandatangani; mono tabular untuk
seluruh nominal, nomor berkas, dan tanggal agar sejajar dalam kolom.

**Enam keputusan yang membentuk antarmuka ini**

1. Rel 12 tahap sebagai tulang punggung — penomoran dipakai karena ini memang
   urutan wajib, bukan hiasan.
2. Uang selalu ditampilkan berpasangan: rencana ↔ realisasi, uang muka ↔ selisih.
3. Serif hanya di dalam pratinjau dokumen resmi.
4. Selisih punya dua arah, dan keduanya punya warna serta alur yang berbeda.
5. Tombol tak pernah hilang, hanya nonaktif dengan penjelasan wewenang.
6. Laporan substansi terpisah dari SPJ keuangan — satu struk hilang tidak boleh
   menahan laporan yang sudah selesai.

## Teknis

- Satu berkas, tanpa CDN, tanpa pustaka pihak ketiga.
- Dua tema penuh: seluruh warna berasal dari token CSS; mode gelap mengikuti
  setelan perangkat dan menghormati stempel `data-theme` eksplisit.
- Status tak pernah disampaikan hanya lewat warna — tiap label memuat titik
  bentuk dan teks, sehingga tetap terbaca pada cetakan hitam-putih.
- Tabel lebar menggulir di dalam wadahnya; halaman tidak pernah bergeser ke
  samping — penting karena SPJ sering diisi dari lapangan lewat ponsel.
- Fokus papan tik terlihat di seluruh kontrol; `prefers-reduced-motion` dihormati.
- Diuji pada lebar 390 / 1024 / 1440 piksel, tema terang dan gelap, seluruh
  delapan layar dan dua belas tahap.

## Catatan

Seluruh nama orang, nomor surat, nominal, dan dokumen adalah **data contoh**
untuk keperluan perancangan. Pratinjau Surat Tugas dan SPPD memperagakan tata
letak, bukan dokumen sungguhan.
