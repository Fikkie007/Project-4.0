# SLA transaksi

## 1. Aturan pembayaran

### Status bisnis transaksi

Platform menyimpan status offer, pembayaran, fulfillment, payout, dan refund
secara terpisah.

#### Offer state

- `pending`
- `accepted`
- `rejected`
- `expired`
- `cancelled`

#### Payment state

- `payment_pending`
- `partially_paid`
- `paid`
- `refunded`

#### Fulfillment state

- `awaiting_cod`
- `awaiting_shipment`
- `shipped`
- `received`
- `disputed`
- `completed`
- `cancelled`

#### Payout state

- `payout_pending`
- `paid_out`
- `payout_failed`

#### Refund state

- `refund_pending`
- `refund_approved`
- `refunded`
- `refund_failed`

Status `partially_paid` digunakan untuk pembayaran DP. Status `paid` digunakan
untuk pembayaran penuh. Payment state dan refund state dicatat terpisah karena
refund dapat masih diproses setelah pembayaran diverifikasi.

Buyer memilih salah satu metode pembayaran:

1. DP, hanya untuk transaksi COD
2. Pembayaran penuh, untuk COD atau pengiriman

Pada Fase 1, buyer membayar sesuai instruksi platform dan dana dikelola secara
manual. Fase 1 belum menggunakan payment gateway.

Transaksi aktif setelah platform menerima konfirmasi pembayaran. Dana tidak
diteruskan kepada seller sebelum syarat penyelesaian transaksi terpenuhi.

Setiap transaksi memiliki rekening tujuan, nominal, batas waktu, dan kode
referensi unik. Buyer wajib mengunggah bukti pembayaran. Admin memverifikasi
nominal, rekening tujuan, kode referensi, dan waktu pembayaran sebelum mengubah
status menjadi `paid`.

Pembayaran kurang, lebih, duplikat, atau tanpa referensi tetap berstatus
`payment_pending` dan masuk antrean rekonsiliasi manual. Admin mencatat hasil
rekonsiliasi serta keputusan pengembalian atau penyesuaian dana dalam audit log.
Pembayaran yang tidak dapat diverifikasi tidak mengaktifkan transaksi.

## 2. Aturan offer

1. Saat buyer mengajukan offer, status offer menjadi `pending` dan seller
   memiliki waktu 48 jam untuk menerima atau menolak offer.
2. Jika seller tidak merespons dalam 48 jam sejak offer diajukan, status offer
   menjadi `expired` dan listing kembali berstatus `active`.
3. Seller dapat menerima atau menolak offer selama periode tersebut.
4. Jika seller menolak offer, status offer menjadi `rejected` dan listing tetap
   berstatus `active`.
5. Buyer dapat membatalkan offer sebelum seller menerimanya. Status offer
   menjadi `cancelled` dan listing tetap berstatus `active`.
6. Setelah seller menerima offer, status offer menjadi `accepted` dan buyer
   memiliki waktu 48 jam untuk melakukan pembayaran yang dipilih.
7. Jika buyer tidak melakukan pembayaran dalam 48 jam setelah offer diterima,
   status offer menjadi `expired` dan listing kembali berstatus `active`.
8. Jika buyer tidak membayar tanpa alasan yang disetujui, buyer dapat dikenai
   suspend sesuai aturan penalti platform.
9. Jika seller membatalkan offer yang sudah diterima tanpa alasan yang
   disetujui, status offer menjadi `cancelled` dan seller dapat dikenai suspend.

## 3. Transaksi COD dengan DP

1. Buyer membayar DP melalui platform.
2. Buyer dan seller memilih jadwal serta lokasi COD.
3. Buyer membayar pelunasan melalui platform setelah transaksi COD dilakukan.
4. Platform memvalidasi bahwa pembayaran sudah lunas.
5. Buyer dan seller mengonfirmasi transaksi selesai.
6. Platform memproses payout kepada seller dalam 1 hari kerja.

Jika buyer membatalkan tanpa kesalahan seller, DP menjadi hak seller. Jika
seller membatalkan atau barang tidak sesuai, DP dikembalikan penuh kepada buyer.
Biaya platform tidak dikenakan jika transaksi belum selesai.

## 4. Transaksi pembayaran penuh

1. Buyer membayar penuh melalui platform.
2. Buyer memilih metode transaksi COD atau pengiriman.
3. Dana ditahan sampai transaksi selesai atau komplain diputuskan.
4. Buyer dan seller mengonfirmasi transaksi selesai.
5. Platform memproses payout kepada seller dalam 1 hari kerja.
6. Payout seller adalah harga barang dikurangi biaya platform 10%. Ongkos kirim
   tidak termasuk dalam biaya platform maupun payout seller.

Buyer dapat membatalkan sebelum seller mengirim barang. Refund mengikuti aturan
refund pada bagian 7. Jika transaksi selesai, biaya platform dipotong dari payout
seller.

## 5. Ketentuan pengiriman

1. Transaksi pengiriman wajib menggunakan pembayaran penuh.
2. Seller wajib mengunggah nomor resi dalam 2 x 24 jam setelah pembayaran.
3. Jika seller tidak mengunggah resi sesuai batas waktu, transaksi dibatalkan.
4. Buyer menerima kembali 100% harga barang dan ongkos kirim jika seller gagal
   mengirim barang.
5. Karena integrasi status ekspedisi ditunda pada MVP, admin mencatat status
   diterima berdasarkan bukti pengiriman atau konfirmasi buyer.
6. Buyer memiliki 2 x 24 jam sejak status diterima untuk mengonfirmasi atau
   mengajukan komplain.
7. Buyer wajib mengunggah minimal 3 foto unboxing. Video bersifat opsional.
8. Foto harus menunjukkan kondisi paket, label pengiriman, dan barang.

Jika paket belum diterima, auto-release tidak berlaku. Buyer dapat mengajukan
komplain jika status ekspedisi salah, paket hilang, atau paket rusak.

## 6. Penyelesaian transaksi dan auto-release

1. Dana diteruskan setelah buyer dan seller mengonfirmasi transaksi selesai.
2. Untuk COD, dana auto-release setelah 1 x 24 jam sejak seller menandai
   transaksi selesai jika buyer tidak merespons.
3. Untuk pengiriman, dana auto-release setelah 2 x 24 jam sejak status paket
   diterima jika buyer tidak merespons.
4. Jika buyer mengajukan komplain sebelum batas waktu, dana tetap ditahan.

Job terjadwal memeriksa batas waktu auto-release dan membuat tugas payout hanya
sekali untuk setiap transaksi. Jika job gagal, transaksi tetap tertahan dan
masuk alert operasional untuk diproses ulang oleh admin tanpa membuat payout
ganda.

## 7. Refund dan pembatalan

1. Seller terbukti salah atau barang tidak sesuai: buyer menerima kembali 100%
   harga barang dan ongkos kirim tanpa potongan biaya platform. Seller
   menanggung ongkos retur.
2. Buyer berubah pikiran tanpa kesalahan seller: buyer menerima harga barang
   dikurangi biaya platform 10%.
3. Ongkos kirim yang sudah digunakan tidak dikembalikan jika buyer berubah
   pikiran. Jika barang belum dikirim, ongkos kirim dikembalikan penuh. Buyer
   menanggung ongkos retur jika barang sudah dikirim.
4. Kesalahan platform atau payment provider: buyer menerima kembali 100% harga
   barang dan ongkos kirim tanpa potongan biaya platform.
5. Buyer tidak dapat membatalkan sepihak setelah seller mengunggah resi.
6. Seller yang membatalkan setelah pembayaran wajib mengembalikan 100% harga
   barang dan ongkos kirim.
7. Seller yang gagal mengirim sesuai batas waktu wajib mengembalikan 100% harga
   barang dan ongkos kirim.

Refund diproses maksimal 3 hari kerja setelah keputusan refund dibuat.

## 8. Komplain dan retur

Buyer dapat mengajukan komplain jika barang:

- Berbeda dari foto atau deskripsi
- Lebih buruk dari rating kurasi
- Palsu atau tidak autentik
- Rusak, kurang, atau salah ukuran karena kesalahan seller
- Berbeda dari barang pada listing

Refund tidak berlaku untuk perubahan pikiran, salah memilih ukuran, kerusakan
akibat penggunaan buyer, atau perbedaan warna kecil karena layar dan pencahayaan.

1. Buyer mengajukan komplain dengan foto atau video melalui platform.
2. Seller memiliki 1 x 24 jam untuk memberikan tanggapan dan bukti.
3. Platform memutuskan komplain dalam 3 hari kerja.
4. Dana tetap ditahan selama proses review.
5. Jika seller terbukti salah, seller menanggung ongkos retur.
6. Jika komplain dinyatakan sebagai pembatalan karena buyer berubah pikiran,
   buyer menanggung ongkos retur dan refund mengikuti aturan pada bagian 7.
7. Buyer wajib mengunggah bukti pengiriman retur.
8. Platform memproses refund sesuai keputusan komplain.
9. Jika seller tidak merespons, platform memutuskan berdasarkan bukti yang
   tersedia.
10. Setelah payout selesai, komplain baru tidak dapat diajukan kecuali untuk
    fraud atau pelanggaran serius.

## 9. COD no-show

1. Buyer tidak hadir tanpa alasan yang disetujui: DP menjadi hak seller.
2. Buyer tidak hadir pada transaksi dengan pembayaran penuh: buyer menerima
   refund 100% harga barang.
3. Seller tidak hadir: buyer menerima refund 100% harga barang dan ongkos kirim
   jika ada.
4. Kedua pihak tidak hadir: transaksi dibatalkan, buyer menerima refund 100%,
   dan tidak ada payout kepada seller.
5. Pihak yang hadir wajib mengirim bukti melalui platform.
6. Platform menyimpan bukti waktu dan lokasi pertemuan.
7. Admin memutuskan sengketa berdasarkan bukti yang tersedia dari kedua pihak.
8. Pihak yang tidak hadir dapat menerima penalti reputasi atau suspend.

## 10. Pembayaran gagal dan payout

1. Platform mencatat pembayaran sebagai `payment_pending` sampai bukti pembayaran
   diverifikasi.
2. Pembayaran yang telah diverifikasi menggunakan status `paid`.
3. Jika pembayaran tidak dapat diverifikasi, transaksi tidak diaktifkan.
4. Platform melakukan verifikasi dan rekonsiliasi secara manual.
5. Payout menggunakan status `payout_pending`, `paid_out`, atau `payout_failed`.
6. Platform mencoba ulang payout yang gagal dengan idempotency key dan memberi
   notifikasi kepada seller.

## 11. Ringkasan batas waktu

| Aktivitas | Batas waktu |
| --- | --- |
| Masa berlaku offer | 48 jam |
| Pembayaran setelah offer diterima | 48 jam |
| Seller mengunggah resi | 2 x 24 jam setelah pembayaran |
| Buyer mengajukan komplain | 2 x 24 jam setelah status diterima |
| Tanggapan seller atas komplain | 1 x 24 jam |
| Keputusan platform | 3 hari kerja |
| Pemrosesan refund | 3 hari kerja setelah keputusan |
| Payout seller | 1 hari kerja setelah transaksi selesai |

## 12. Flowchart transaksi

Diagram canonical tersedia di
[`SLA-flowchart.html`](../../diagrams/sla/SLA-flowchart.html).

## 13. Customer support

1. Platform memberikan respons awal maksimal dalam 1 hari kerja.
2. Kasus biasa diselesaikan maksimal dalam 3 hari kerja.
3. Kasus pembayaran atau dana tertahan diprioritaskan dan ditangani maksimal
   dalam 1 hari kerja.
4. Kasus fraud atau keamanan diprioritaskan segera.

## 14. Fraud dan keamanan

1. Platform dapat menahan dana dan membekukan transaksi yang mencurigakan.
2. Platform dapat meminta verifikasi tambahan dari buyer atau seller.
3. Platform dapat membatasi akun sementara selama proses investigasi.
4. Transaksi dalam investigasi fraud tidak mengikuti auto-release.
5. Platform menyimpan audit log untuk pembayaran, perubahan status, refund, dan
   keputusan admin.

## 15. Retensi data

1. Foto dan video listing serta unboxing disimpan selama transaksi aktif dan 1
   tahun setelah transaksi selesai.
2. Data pembayaran, refund, dan payout disimpan sesuai kebutuhan audit serta
   kewajiban hukum.
3. Audit log disimpan minimal 2 tahun.
4. Setelah masa retensi berakhir, media dihapus atau dianonimkan.
5. Platform memberi informasi kepada buyer dan seller tentang penggunaan serta
   penghapusan data.

## 16. Ketersediaan platform

1. Target availability bulanan platform adalah 99,5%.
2. Maintenance terjadwal tidak dihitung sebagai downtime.
3. Platform memberi pemberitahuan maintenance minimal 24 jam sebelumnya.
4. Insiden kritis mendapat respons maksimal dalam 1 jam.
5. Target recovery setelah insiden kritis adalah maksimal 4 jam.

## 17. Backup dan pemulihan

1. Platform membuat backup database otomatis setiap 24 jam.
2. Platform menyimpan minimal 7 backup terakhir.
3. Target kehilangan data maksimum adalah 24 jam.
4. Platform menguji pemulihan backup minimal 1 kali per bulan.
5. Data pembayaran dan transaksi memiliki backup terpisah dari database utama.
6. Target pemulihan mengikuti batas 4 jam pada insiden kritis.

## 18. Monitoring dan alerting

1. Platform memantau availability, error rate, latency, dan penggunaan database.
2. Platform memantau pembayaran yang belum diverifikasi, payout gagal, dan
   transaksi macet.
3. Platform memantau jumlah komplain dan refund.
4. Insiden kritis memicu alert maksimal dalam 5 menit.
5. Log aplikasi dan audit log menggunakan `transaction_id` untuk korelasi.

## 19. Kontrol akses dan keamanan akun

1. Platform memisahkan role buyer, seller, support, kurator, dan admin.
2. Admin dan support wajib menggunakan MFA.
3. Setiap role hanya dapat mengakses data dan tindakan yang diperlukan.
4. Platform menyimpan password menggunakan hashing yang aman.
5. Platform mencatat login, perubahan rekening payout, refund, dan tindakan
   admin.
6. Platform memvalidasi tipe dan ukuran file foto serta video sebelum disimpan.
7. Platform mengakhiri sesi aktif setelah password atau data keamanan berubah.

## 20. Notifikasi transaksi

1. Platform memberi notifikasi saat offer diterima, ditolak, atau kedaluwarsa.
2. Platform memberi notifikasi saat pembayaran berhasil atau gagal.
3. Platform memberi pengingat sebelum batas pengiriman dan konfirmasi berakhir.
4. Platform memberi notifikasi saat resi diunggah, paket diterima, refund,
   komplain, dan payout.
5. Notifikasi penting dikirim melalui aplikasi dan email.
6. Platform menyimpan waktu pengiriman serta status berhasil atau gagal untuk
   setiap notifikasi.

## 21. Kurasi listing

1. Pada Fase 2, kurator meninjau listing maksimal dalam 2 hari kerja.
2. Listing yang ditolak wajib memiliki alasan penolakan.
3. Seller dapat memperbaiki dan mengirim ulang listing yang ditolak.
4. Pada Fase 2, kurator meninjau ulang revisi maksimal dalam 1 hari kerja.
5. Listing tanpa minimal 3 foto tidak masuk antrean kurasi.
6. Platform dapat menyembunyikan atau menghapus listing yang melanggar aturan.

## 22. Informasi harga pasar

1. Median hanya ditampilkan jika terdapat minimal 5 transaksi selesai.
2. Perhitungan menggunakan transaksi selesai, bukan harga listing.
3. Data dikelompokkan berdasarkan produk, kondisi, ukuran, dan periode.
4. Data harga diperbarui minimal 1 kali per hari.
5. Jika data kurang dari 5 transaksi, platform menampilkan bahwa data belum
   cukup.
6. Platform tidak menampilkan identitas transaksi sumber.

## 23. Rating kondisi barang

1. Rating 100% berarti seperti baru tanpa cacat berarti.
2. Rating 90% berarti sangat baik dengan cacat kecil.
3. Rating 75% berarti baik dengan tanda pemakaian yang terlihat.
4. Rating 50% berarti cukup dengan cacat atau kerusakan yang dijelaskan.
5. Barang dengan rating di bawah 50% tidak lolos kurasi.
6. Kurator wajib menjelaskan dasar rating, kelebihan, dan kekurangan barang.

## 24. Barang terlarang

1. Platform melarang barang palsu atau hasil pemalsuan.
2. Platform melarang barang ilegal, berbahaya, atau curian.
3. Platform melarang obat, senjata, dan barang yang memerlukan izin khusus.
4. Platform melarang deskripsi atau foto yang menyesatkan.
5. Platform melarang barang yang melanggar hak cipta atau merek.
6. Platform dapat menghapus listing dan membatasi akun pelanggar.

## 25. Verifikasi identitas

1. Buyer dan seller wajib memverifikasi akun menggunakan OTP WhatsApp.
2. Seller wajib memberikan nomor rekening saat transaksi akan diselesaikan.
3. Transaksi senilai Rp5.000.000 atau lebih wajib melalui verifikasi identitas
   tambahan.
4. Satu rekening payout tidak boleh digunakan oleh banyak akun tanpa verifikasi.
5. Platform dapat menahan payout sampai proses verifikasi selesai.

## 26. Biaya platform

1. Biaya platform adalah 10% dari harga barang.
2. Ongkos kirim tidak termasuk dasar perhitungan biaya platform.
3. Biaya pemindahan dana, jika ada, tidak termasuk dasar perhitungan biaya
   platform.
4. Biaya platform untuk transaksi selesai hanya dikenakan jika transaksi selesai
   dan dipotong dari payout seller. Pengecualian hanya berlaku untuk refund
   karena buyer berubah pikiran, yang dikenai potongan 10% dari harga barang.
5. Payout seller adalah harga barang dikurangi biaya platform. Ongkos kirim
   tidak termasuk dalam payout seller.
6. Refund karena kesalahan seller atau platform tidak dipotong biaya platform.

## 27. Ongkos kirim

1. Buyer membayar ongkos kirim saat checkout.
2. Ongkos kirim ditampilkan terpisah dari harga barang.
3. Seller menanggung ongkos retur jika seller terbukti salah.
4. Buyer menanggung ongkos retur jika buyer berubah pikiran.
5. Platform tidak menghitung ongkos kirim sebagai dasar biaya platform.

## 28. Release gate MVP

MVP hanya dapat dirilis jika seluruh skenario berikut lulus pengujian end-to-end.

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
