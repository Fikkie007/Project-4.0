# PRD v1.0

## Ringkasan produk

Platform web untuk menjual dan membeli barang second-hand, dengan fokus awal pada produk fashion. Produk fashion dipilih karena lebih mudah dikurasi dan memiliki pasar yang cukup besar.

## Masalah yang ingin diselesaikan

1. Seller yang tidak memiliki media sosial harus bergantung pada Facebook Marketplace.
2. Seller sering mengalami kendala dengan buyer.
3. Facebook Marketplace tidak menyediakan mekanisme keamanan transaksi yang memadai.

## Metrik keberhasilan MVP

Target untuk 90 hari pertama:

1. Minimal 100 transaksi selesai.
2. Payment success rate minimal 95%.
3. Minimal 90% seller mengirim barang sesuai batas waktu.
4. Tingkat komplain maksimal 10%.
5. Minimal 20% buyer melakukan pembelian ulang.

## Scope MVP

### Masuk MVP

- Upload listing fashion oleh seller
- Seller menentukan kondisi barang 1-100% dan menjelaskan kekurangan serta kondisi fisiknya
- Offer atau bidding dengan aturan ghosting 48 jam
- Transaksi COD dan pengiriman
- Live chat buyer dan seller
- Registrasi dan verifikasi melalui OTP WhatsApp
- Pengelolaan dana secara manual, refund, dan sistem report
- Rating, review, last login, suspend, dan ban
- Median harga pasar sederhana

### Ditunda

- Kupon dan promo
- Keranjang multi-produk
- Social login
- Integrasi otomatis status ekspedisi
- Verifikasi autentikasi brand secara otomatis
- Sistem kurator dan validasi barang oleh kurator
- Afiliasi atau komisi kurator
- Dukungan cryptocurrency

## Release gate MVP

MVP belum dirilis sebelum skenario berikut lulus pengujian end-to-end:

### 1. COD sampai selesai

- Buyer memilih DP atau pembayaran penuh.
- Admin memverifikasi pembayaran.
- Transaksi tidak aktif sebelum pembayaran diverifikasi.
- Buyer dan seller menyepakati waktu serta lokasi COD.
- Seller menandai COD selesai.
- Buyer mengonfirmasi transaksi, atau auto-release berjalan setelah 1 x 24 jam
  tanpa respons.
- Payout seller mengikuti aturan biaya platform.
- Transaksi menjadi `completed` dan payout menjadi `paid_out`.

### 2. Pengiriman dengan pembayaran penuh

- Buyer membayar penuh dan pembayaran diverifikasi.
- Seller mengunggah resi maksimal 2 x 24 jam.
- Admin mencatat status `received` berdasarkan bukti pengiriman atau konfirmasi
  buyer.
- Buyer memiliki 2 x 24 jam untuk mengonfirmasi atau mengajukan komplain.
- Jika tidak ada komplain, auto-release berjalan setelah batas waktu.
- Transaksi menjadi `completed` dan payout diproses.

### 3. Refund karena kesalahan seller

- Buyer mengajukan komplain dengan bukti.
- Dana tetap ditahan selama pemeriksaan.
- Seller diberi waktu 1 x 24 jam untuk merespons.
- Platform memutuskan maksimal dalam 3 hari kerja.
- Jika seller terbukti salah, buyer menerima 100% harga barang dan ongkos
  kirim.
- Seller menanggung ongkos retur.
- Tidak ada biaya platform yang dipotong.

### 4. Buyer atau seller tidak hadir saat COD

- Platform menyimpan waktu dan lokasi pertemuan.
- Pihak yang hadir mengirimkan bukti.
- Buyer tidak hadir: DP menjadi hak seller; pembayaran penuh dikembalikan.
- Seller tidak hadir: buyer menerima refund 100%.
- Kedua pihak tidak hadir: transaksi dibatalkan dan tidak ada payout.
- Pihak yang melanggar dapat menerima penalti reputasi atau suspend.

### 5. Pelanggaran aturan ghosting 48 jam

- Seller tidak merespons offer selama 48 jam: offer `expired`, listing kembali
  `active`.
- Buyer tidak membayar dalam 48 jam setelah offer diterima: offer `expired`,
  listing kembali `active`.
- Ghosting tanpa alasan yang disetujui dapat menyebabkan suspend.
- Offer yang kedaluwarsa atau dibatalkan tidak menghasilkan payout.

### 6. Pembayaran bermasalah

Untuk pembayaran kurang, lebih, duplikat, tanpa referensi, atau tidak dapat
diverifikasi:

- Status tetap `payment_pending`.
- Transaksi tidak menjadi aktif.
- Dana tidak diteruskan kepada seller.
- Admin memasukkan kasus ke rekonsiliasi manual.
- Admin mencatat hasil dan keputusan refund atau penyesuaian.
- Setiap keputusan memiliki alasan dan bukti.

### 7. Auto-release gagal

- Dana tetap tertahan jika auto-release gagal.
- Sistem tidak membuat payout ganda.
- Kasus masuk ke alert operasional.
- Admin dapat menjalankan proses ulang.
- Payout hanya dapat dibuat satu kali untuk transaksi tersebut.
- Setelah transfer berhasil, payout menjadi `paid_out`.

## Status transaksi

Platform menyimpan lima kelompok status secara terpisah.

### Offer state

- `pending`
- `accepted`
- `rejected`
- `expired`
- `cancelled`

### Payment state

- `payment_pending`
- `partially_paid`
- `paid`
- `refunded`

### Fulfillment state

- `awaiting_cod`
- `awaiting_shipment`
- `shipped`
- `received`
- `disputed`
- `completed`
- `cancelled`

### Payout state

- `payout_pending`
- `paid_out`
- `payout_failed`

### Refund state

- `refund_pending`
- `refund_approved`
- `refunded`
- `refund_failed`

Status `partially_paid` digunakan untuk pembayaran DP. Status `paid` digunakan
untuk pembayaran penuh. Payment state dan refund state dicatat terpisah karena
refund dapat masih diproses setelah pembayaran diverifikasi.

Status pembayaran dicatat dan diperbarui oleh platform berdasarkan bukti
pembayaran serta verifikasi admin. Platform tidak meneruskan dana kepada seller
sebelum pembayaran diverifikasi.

### Pemilik perubahan status

- Buyer: membuat offer, melakukan pembayaran, memberi konfirmasi, dan mengajukan komplain
- Seller: menerima offer, mengunggah resi, dan menandai COD selesai
- Platform: memverifikasi pembayaran, mengelola dana manual, menjalankan refund,
  payout, dan penyelesaian transaksi
- Admin: melakukan override terbatas dengan audit log

### Batasan override admin

- Admin wajib memasukkan alasan dan bukti untuk setiap override.
- Semua override wajib dicatat dalam audit log.
- Admin tidak boleh mengubah `paid` tanpa bukti pembayaran yang diverifikasi.
- Admin tidak boleh mengubah `paid_out` tanpa konfirmasi pemindahan dana berhasil.
- Refund di atas Rp5.000.000 memerlukan persetujuan dua admin.

## Fitur produk

### Penawaran harga

1. Seller wajib menentukan harga jual langsung dan harga minimum penawaran.
2. Buyer dapat membeli langsung sesuai harga jual tanpa mengirim offer.
3. Buyer dapat mengajukan offer di bawah harga jual berdasarkan harga minimum.
4. Setelah seller menerima offer, listing berubah menjadi `reserved`.
5. Seller wajib menerima atau menolak offer dalam 48 jam sejak offer diajukan.
6. Buyer wajib melakukan pembayaran yang dipilih dalam 48 jam sejak offer
   diterima seller.
7. Jika seller tidak merespons offer selama 48 jam sejak offer diajukan, offer menjadi `expired` dan
   posting kembali menjadi `active`.
8. Jika seller menolak offer, offer menjadi `rejected` dan posting tetap
   `active`.
9. Jika buyer tidak melakukan pembayaran dalam 48 jam setelah offer diterima,
   offer menjadi `expired` dan posting kembali menjadi `active`.
10. Jika buyer tidak melakukan pembayaran setelah offer diterima tanpa alasan
    yang disetujui, buyer mendapat suspend dan offer tersebut menjadi `cancelled`.
11. Jika seller tidak merespons offer dalam batas waktu, offer menjadi `expired`.
    Jika seller menerima offer tetapi kemudian membatalkan tanpa alasan yang
    disetujui, seller mendapat suspend dan offer menjadi `cancelled`.
12. Seller dapat mengirim offer baru setelah offer sebelumnya gagal atau
    kedaluwarsa.
13. Seller tidak dapat menerima offer lain selama listing berstatus `reserved`.
14. Platform menolak offer di bawah harga minimum secara otomatis.
15. Buyer dapat mengirim offer baru selama listing masih berstatus `active`.
16. Harga minimum tidak ditampilkan kepada buyer.
17. Seller dapat mengubah harga minimum sebelum ada offer yang diterima.
18. Satu buyer hanya boleh memiliki satu offer aktif per listing.
19. Buyer dapat membatalkan offer sebelum seller menerimanya.
20. Buyer dapat mengirim offer baru setelah offer sebelumnya ditolak atau
    kedaluwarsa.
21. Seller hanya dapat menerima satu offer untuk satu listing.
22. Offer lain otomatis ditolak setelah satu offer diterima.
23. Harga jual langsung tidak boleh lebih rendah dari harga minimum penawaran.

### Perubahan listing

- Seller dapat mengedit listing saat status `draft` atau `active`.
- Seller tidak dapat mengubah harga, foto, kondisi, atau deskripsi saat `reserved`.
- Seller tidak dapat mengedit listing setelah pembayaran.
- Perubahan besar setelah listing aktif memerlukan pemeriksaan ulang informasi
  kondisi barang.
- Listing yang sedang dalam proses komplain tidak dapat diedit.

### Menonaktifkan listing

- Seller dapat menyembunyikan listing berstatus `draft` atau `active`.
- Listing `reserved` tidak dapat disembunyikan.
- Listing berstatus `paid` tidak dapat dihapus.
- Listing yang selesai berubah menjadi `sold`.
- Listing yang dibatalkan dapat diaktifkan kembali setelah alasan pembatalan
  selesai ditinjau.

### Pembayaran dan transaksi

1. Buyer dapat memilih pembayaran uang muka (DP) atau pembayaran penuh.
2. DP hanya dapat digunakan untuk transaksi COD. Pembayaran penuh dapat digunakan untuk COD atau pengiriman.
3. Pada Fase 1, buyer melakukan pembayaran sesuai instruksi platform dan dana
   dikelola secara manual.
4. Semua transaksi wajib dibuat dan dicatat melalui platform.
5. Platform menahan dana sampai transaksi selesai, refund diputuskan, atau
   komplain diputuskan.
6. Platform menampilkan rekening tujuan, nominal, batas waktu, dan kode referensi
   unik untuk setiap pembayaran.
7. Buyer wajib mengunggah bukti pembayaran. Admin memverifikasi nominal, rekening
   tujuan, kode referensi, dan waktu pembayaran sebelum status menjadi `paid`.
8. Pembayaran kurang, lebih, duplikat, atau tanpa referensi tetap berstatus
   `payment_pending` dan ditandai untuk rekonsiliasi manual oleh admin.
9. Ketentuan pembayaran dan penyelesaian transaksi mengikuti [SLA transaksi](SLA.md).

### Pengiriman dan bukti transaksi

1. Seller wajib mengirimkan barang dalam 2 x 24 jam setelah pembayaran penuh.
2. Untuk transaksi pengiriman, seller wajib mengunggah nomor resi ekspedisi ke platform dalam 2 x 24 jam.
3. Seller dapat mengunggah foto dan video barang sebelum dikirim.
4. Setelah menerima barang, buyer wajib mengunggah minimal tiga foto saat membuka paket. Video bersifat opsional.
5. Karena integrasi status ekspedisi ditunda, admin mencatat status `received`
   berdasarkan bukti pengiriman atau konfirmasi buyer.

### Refund

1. Buyer dapat mengajukan refund melalui platform jika barang yang diterima tidak sesuai dengan deskripsi seller.
2. Jika seller terbukti salah, buyer menerima kembali 100% harga barang dan
   ongkos kirim tanpa potongan biaya platform.
3. Jika buyer berubah pikiran tanpa kesalahan seller, buyer menerima harga
   barang dikurangi 10% sebagai biaya platform. Ongkos kirim yang sudah
   digunakan tidak dikembalikan. Jika barang belum dikirim, ongkos kirim
   dikembalikan penuh.
4. Untuk transaksi COD dengan DP, DP menjadi hak seller jika buyer membatalkan tanpa kesalahan seller.
5. Buyer membayar ongkos kirim secara terpisah dari harga barang.
6. Ongkos kirim tidak menjadi dasar perhitungan biaya platform.
7. Biaya platform sebesar 10% untuk transaksi selesai dipotong dari payout
   seller. Pengecualian hanya berlaku untuk refund karena buyer berubah pikiran,
   yang dikenai potongan 10% dari harga barang.
8. Payout seller adalah harga barang dikurangi biaya platform. Ongkos kirim
   tidak termasuk dalam biaya platform maupun payout seller.

### Komplain dan retur

1. Buyer dapat mengajukan komplain maksimal 2 x 24 jam sejak status transaksi
   menjadi `received`.
2. Buyer wajib menyertakan foto atau video sebagai bukti komplain.
3. Seller memiliki waktu 1 x 24 jam untuk memberikan tanggapan dan bukti.
4. Platform memutuskan komplain maksimal dalam 3 hari kerja.
5. Selama komplain, dana tetap ditahan dan proses auto-release dihentikan.
6. Jika seller terbukti salah, buyer menerima refund 100% harga barang dan
   ongkos kirim. Seller menanggung ongkos retur.
7. Jika buyer berubah pikiran, kasus diproses sebagai pembatalan dan refund
   mengikuti potongan 10% dari harga barang.
8. Jika seller tidak merespons, platform memutuskan berdasarkan bukti yang
   tersedia.
9. Setelah payout selesai, komplain baru tidak dapat diajukan kecuali untuk
   fraud atau pelanggaran serius.

### Informasi harga pasar

1. Platform menampilkan median harga penjualan produk berdasarkan minimal lima transaksi selesai.
2. Platform menampilkan grafik riwayat harga dalam periode mingguan, bulanan, dan tahunan.
3. Contoh: jika seller memposting Adidas Zero seharga Rp1.000.000, platform menampilkan riwayat harga mingguan, bulanan, dan tahunan untuk produk tersebut.
4. Platform menyediakan slider untuk membantu seller menentukan harga berdasarkan median harga pasar.
5. Platform menampilkan rentang harga dan indikator apakah harga produk tergolong layak atau tidak.
6. Informasi harga mempertimbangkan kondisi barang, ukuran, dan periode transaksi.
7. Jika data belum mencapai lima transaksi, platform menampilkan bahwa data belum cukup.

### Rekomendasi produk

- Setiap Product Detail Page memiliki bagian rekomendasi produk.

### Privasi foto dan video

- Foto atau video tertentu yang diunggah hanya dapat dilihat oleh user yang
  mengunggahnya dan admin.

### Kurasi dan kondisi produk

1. Seller menentukan kondisi barang dalam persentase 1-100%.
2. Seller wajib menjelaskan kekurangan, kondisi fisik, dan informasi relevan
   lainnya.
3. Pada MVP, informasi kondisi barang sepenuhnya berasal dari seller.
4. Pada Fase 2, kurator memverifikasi kondisi barang dan dapat memberi
   rekomendasi atau validasi terkait harga, brand, dan informasi produk.
5. Setiap kurator memiliki profil kurator.
6. Kurator menerima komisi dari transaksi berhasil untuk barang yang dikurasi.
7. Rating 100% berarti seperti baru, 90% sangat baik, 75% baik, dan 50% cukup.
8. Produk dengan rating di bawah 50% tidak lolos kurasi setelah sistem kurator
   diterapkan pada Fase 2.

### Barang terlarang

1. Platform melarang barang palsu, ilegal, berbahaya, atau curian.
2. Platform melarang obat, senjata, dan barang yang memerlukan izin khusus.
3. Platform melarang listing dengan deskripsi atau foto yang menyesatkan.
4. Platform dapat menghapus listing dan membatasi akun pelanggar.

### Reputasi buyer dan seller

1. Buyer dapat memberi rating seller setelah transaksi selesai.
2. Seller dapat memberi rating buyer setelah transaksi selesai.
3. Setiap pihak hanya dapat memberi satu rating per transaksi.
4. Rating hanya aktif setelah tidak ada komplain.
5. Review terhadap buyer dan seller bersifat publik.
6. Pengguna dapat melaporkan ulasan yang menyesatkan atau mengandung pelecehan.
7. Platform dapat menyembunyikan ulasan yang melanggar aturan.
8. Akun user menampilkan informasi last login untuk keperluan informatif.

### Pencarian dan filter listing

1. Buyer dapat mencari berdasarkan nama produk, brand, dan kategori.
2. Buyer dapat memfilter berdasarkan harga, ukuran, kondisi, metode transaksi,
   dan lokasi.
3. Buyer dapat mengurutkan berdasarkan harga, listing terbaru, dan rating kondisi.
4. Listing `draft`, `reserved`, `sold`, atau `hidden` tidak muncul sebagai listing aktif.
5. Hasil pencarian menampilkan status ketersediaan dengan jelas.

### Lokasi COD

1. Buyer dan seller hanya melihat area atau kota, bukan alamat pribadi.
2. Platform menyediakan pilihan lokasi pertemuan umum.
3. Alamat lengkap hanya dibagikan setelah kedua pihak menyetujui transaksi COD.
4. Platform menyimpan waktu dan lokasi COD untuk bukti no-show.
5. Buyer dan seller dapat mengubah jadwal sebelum waktu pertemuan.

### Live chat

1. Platform menyediakan live chat di dalam aplikasi.
2. Untuk MVP, live chat dapat menggunakan API atau service gratis jika memenuhi
   kebutuhan.
3. Buyer dan seller dapat menggunakan live chat untuk membahas transaksi,
   termasuk pengaturan lokasi, tanggal, dan jam COD.

Jika seller tidak hadir pada waktu COD yang disepakati:

1. Buyer menerima refund 100% harga barang dan ongkos kirim jika ada.
2. Buyer dapat melaporkan seller.
3. Pelanggaran dapat memengaruhi rating atau reputasi seller.
4. Seller dapat dikenai suspend atau ban berdasarkan tingkat atau frekuensi
   pelanggaran.

Jika buyer tidak hadir pada waktu COD yang disepakati:

1. Untuk transaksi dengan DP, DP menjadi hak seller.
2. Untuk transaksi dengan pembayaran penuh, buyer menerima refund 100% harga
   barang.
3. Buyer dapat dikenai penalti reputasi atau suspend.

Jika buyer dan seller sama-sama tidak hadir:

1. Transaksi dibatalkan.
2. Buyer menerima refund 100%.
3. Tidak ada payout kepada seller.

Pihak yang hadir wajib mengirim bukti melalui platform. Admin memutuskan
sengketa berdasarkan bukti yang tersedia dari kedua pihak.

Status penalti:

- **Suspend:** Akun dinonaktifkan sementara selama durasi tertentu.
- **Ban:** Akun dinonaktifkan secara permanen.

### Onboarding seller

1. Seller wajib memverifikasi akun menggunakan OTP WhatsApp sebelum membuat
   listing.
2. Seller dapat membuat listing tanpa mengisi nomor rekening saat registrasi.
3. Seller wajib memberikan nomor rekening ketika transaksi akan diselesaikan.
4. Akun yang dibatasi tidak dapat membuat listing atau menerima offer.

### Onboarding buyer

1. Buyer wajib memverifikasi akun menggunakan OTP WhatsApp sebelum mengirim
   offer.
2. Buyer dapat menjelajah listing tanpa verifikasi.
3. Buyer dengan akun yang dibatasi tidak dapat mengirim offer atau melakukan
   pembayaran.
4. Satu buyer hanya boleh memiliki satu akun aktif.

### Banding akun

1. Buyer atau seller dapat mengajukan banding melalui platform.
2. Banding wajib menyertakan alasan dan bukti.
3. Platform meninjau banding maksimal dalam 3 hari kerja.
4. Akun tetap dibatasi selama proses review.
5. Platform memberi alasan tertulis atas keputusan akhir.

## Flowchart transaksi

Flow transaksi pembayaran, pengiriman, auto-release, refund, dan komplain
terdapat di [SLA transaksi](SLA.md#12-flowchart-transaksi).
