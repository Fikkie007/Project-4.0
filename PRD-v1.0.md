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

- Listing fashion dengan minimal 3 foto
- Kurasi dan rating kondisi
- Offer harga
- Pembayaran melalui virtual account (VA) dan e-wallet
- Transaksi COD dan pengiriman
- Hold, payout, refund, dan komplain
- Median harga pasar sederhana

### Ditunda

- Chat buyer dan seller
- Kupon dan promo
- Keranjang multi-produk
- Social login
- Integrasi otomatis status ekspedisi
- Verifikasi autentikasi brand secara otomatis

## Release gate MVP

MVP belum dirilis sebelum skenario berikut lulus pengujian end-to-end:

1. COD dengan DP sampai payout seller.
2. Pengiriman dengan pembayaran penuh sampai auto-release.
3. Refund karena kesalahan seller.
4. Buyer atau seller tidak hadir saat COD.
5. Pembayaran gagal dan rekonsiliasi dana.

## Status transaksi

Platform menyimpan tiga kelompok status secara terpisah.

### Payment state

- `payment_pending`
- `paid`
- `payment_failed`
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

`payment_failed` hanya dapat diubah oleh webhook payment provider atau proses
rekonsiliasi admin. Buyer dapat mencoba pembayaran ulang tanpa membuat transaksi
baru. Platform tidak melakukan payout sebelum `payment_state` menjadi `paid`.

### Pemilik perubahan status

- Buyer: membuat offer, melakukan pembayaran, memberi konfirmasi, dan mengajukan komplain
- Seller: menerima offer, mengunggah resi, dan menandai COD selesai
- Payment provider: mengirim payment webhook
- Platform: menjalankan auto-release, refund, payout, dan rekonsiliasi
- Admin: melakukan override terbatas dengan audit log

### Batasan override admin

- Admin wajib memasukkan alasan dan bukti untuk setiap override.
- Semua override wajib dicatat dalam audit log.
- Admin tidak boleh mengubah `paid` tanpa hasil rekonsiliasi payment provider.
- Admin tidak boleh mengubah `paid_out` tanpa konfirmasi payout berhasil.
- Refund di atas Rp5.000.000 memerlukan persetujuan dua admin.

### Payment webhook dan rekonsiliasi

- Platform memverifikasi signature setiap webhook.
- Platform menolak webhook duplikat atau replay.
- Platform menyimpan payload webhook mentah untuk audit.
- Platform mencoba ulang pemrosesan webhook maksimal 5 kali.
- Platform menjalankan rekonsiliasi otomatis setiap 15 menit.
- Perbedaan status ditandai sebagai `reconciliation_required`.

## Fitur produk

### Penawaran harga

1. Seller wajib menentukan harga jual langsung dan harga minimum penawaran.
2. Buyer dapat membeli langsung sesuai harga jual tanpa mengirim offer.
3. Buyer dapat mengajukan offer di bawah harga jual berdasarkan harga minimum.
3. Setelah seller menerima offer, listing berubah menjadi `reserved`.
4. Buyer memiliki 1 x 24 jam untuk menyelesaikan pembayaran.
5. Buyer lain tidak dapat membeli atau mengajukan offer selama listing berstatus `reserved`.
6. Jika buyer tidak membayar dalam batas waktu, listing kembali berstatus `active`.
7. Seller tidak dapat menerima offer lain selama listing berstatus `reserved`.
8. Platform menolak offer di bawah harga minimum secara otomatis.
9. Buyer dapat mengirim offer baru selama listing masih berstatus `active`.
10. Harga minimum tidak ditampilkan kepada buyer.
11. Seller dapat mengubah harga minimum sebelum ada offer yang diterima.
12. Satu buyer hanya boleh memiliki satu offer aktif per listing.
13. Buyer dapat membatalkan offer sebelum seller menerimanya.
14. Buyer dapat mengirim offer baru setelah offer sebelumnya ditolak atau
    kedaluwarsa.
15. Seller hanya dapat menerima satu offer untuk satu listing.
16. Offer lain otomatis ditolak setelah satu offer diterima.
17. Harga jual langsung tidak boleh lebih rendah dari harga minimum penawaran.

### Perubahan listing

- Seller dapat mengedit listing saat status `draft` atau `active`.
- Seller tidak dapat mengubah harga, foto, kondisi, atau deskripsi saat `reserved`.
- Seller tidak dapat mengedit listing setelah pembayaran.
- Perubahan besar setelah listing aktif memerlukan kurasi ulang.
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
3. Platform MVP mendukung pembayaran melalui virtual account (VA) dan e-wallet.
4. Semua transaksi wajib dibuat dan diproses melalui platform.
5. Setelah buyer melakukan pembayaran, platform menahan dana sampai transaksi selesai, auto-release, atau komplain diputuskan.
6. Ketentuan pembayaran dan penyelesaian transaksi mengikuti [SLA transaksi](SLA.md).

### Pengiriman dan bukti transaksi

1. Seller wajib mengirimkan barang dalam 2 x 24 jam setelah pembayaran penuh.
2. Untuk transaksi pengiriman, seller wajib mengunggah nomor resi ekspedisi ke platform dalam 2 x 24 jam.
3. Seller dapat mengunggah foto dan video barang sebelum dikirim.
4. Setelah menerima barang, buyer wajib mengunggah minimal tiga foto saat membuka paket. Video bersifat opsional.

### Refund

1. Buyer dapat mengajukan refund melalui platform jika barang yang diterima tidak sesuai dengan deskripsi seller.
2. Jika seller terbukti salah, buyer menerima refund 100% tanpa potongan biaya platform.
3. Jika buyer berubah pikiran tanpa kesalahan seller, refund dipotong 10% dari harga barang sebagai biaya platform.
4. Untuk transaksi COD dengan DP, DP menjadi hak seller jika buyer membatalkan tanpa kesalahan seller.
5. Buyer membayar ongkos kirim secara terpisah dari harga barang.
6. Ongkos kirim tidak menjadi dasar perhitungan biaya platform.

### Informasi harga pasar

1. Platform menampilkan median harga penjualan produk berdasarkan minimal lima transaksi selesai.
2. Platform menampilkan grafik riwayat harga dalam periode mingguan, bulanan, dan tahunan.
3. Contoh: jika seller memposting Adidas Zero seharga Rp1.000.000, platform menampilkan riwayat harga mingguan, bulanan, dan tahunan untuk produk tersebut.
4. Platform menyediakan slider untuk membantu seller menentukan harga berdasarkan median harga pasar.
5. Platform menampilkan rentang harga dan indikator apakah harga produk tergolong layak atau tidak.
6. Informasi harga mempertimbangkan kondisi barang, ukuran, dan periode transaksi.
7. Jika data belum mencapai lima transaksi, platform menampilkan bahwa data belum cukup.

### Kurasi dan kondisi produk

1. Seller wajib mengunggah minimal tiga foto untuk setiap produk.
2. Platform memberikan rating untuk produk yang telah dikurasi.
3. Rating mencakup persentase kondisi barang serta penjelasan mengenai kelebihan dan kekurangannya.
4. Rating 100% berarti seperti baru, 90% sangat baik, 75% baik, dan 50% cukup.
5. Produk dengan rating di bawah 50% tidak lolos kurasi.

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
5. Pengguna dapat melaporkan ulasan yang menyesatkan atau mengandung pelecehan.
6. Platform dapat menyembunyikan ulasan yang melanggar aturan.

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

### Onboarding seller

1. Seller wajib memverifikasi email dan nomor telepon sebelum membuat listing.
2. Seller dapat membuat listing tanpa rekening payout terverifikasi.
3. Seller wajib memverifikasi rekening payout sebelum menerima dana.
4. Seller dengan verifikasi gagal tidak dapat membuat listing aktif.
5. Akun yang dibatasi tidak dapat membuat listing atau menerima offer.

### Onboarding buyer

1. Buyer wajib memverifikasi email dan nomor telepon sebelum mengirim offer.
2. Buyer wajib memverifikasi data pembayaran sebelum checkout.
3. Buyer dapat menjelajah listing tanpa verifikasi.
4. Buyer dengan akun yang dibatasi tidak dapat mengirim offer atau melakukan
   pembayaran.
5. Satu buyer hanya boleh memiliki satu akun aktif.

### Banding akun

1. Buyer atau seller dapat mengajukan banding melalui platform.
2. Banding wajib menyertakan alasan dan bukti.
3. Platform meninjau banding maksimal dalam 3 hari kerja.
4. Akun tetap dibatasi selama proses review.
5. Platform memberi alasan tertulis atas keputusan akhir.

## Flowchart transaksi

Flow transaksi pembayaran, pengiriman, auto-release, refund, dan komplain
terdapat di [SLA transaksi](SLA.md#12-flowchart-transaksi).
