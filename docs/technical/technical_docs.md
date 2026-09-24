# Tech stack

- Nuxt.js
- Go
- PostgreSQL

## Pengelolaan pembayaran

- Fase 1 menggunakan pengelolaan dana secara manual tanpa payment gateway.
- Evaluasi provider seperti DOKU atau Xendit ditunda sampai fase lanjutan.
- Kandidat dan batasan provider dicatat di [payment-provider-comparison.md](../research/payment-provider-comparison.md).
- Keputusan Fase 2 harus mengonfirmasi marketplace approval, hold/release,
  refund setelah settlement, seller KYC, payout limits, fees, reserves, dan
  reconciliation API sebelum integrasi dimulai.

## Live chat MVP

- MVP menggunakan Centrifugo OSS sebagai transport realtime self-hosted.
- Go menangani autentikasi, izin akses berdasarkan transaksi, aturan chat,
  laporan pengguna, dan penutupan percakapan.
- PostgreSQL menyimpan pesan, riwayat percakapan, audit event, dan status
  penanganan laporan.
- Pesan ditulis ke PostgreSQL sebelum dipublikasikan ke Centrifugo.
- Centrifugo tidak menjadi arsip utama; history/recovery realtime hanya membantu
  koneksi ulang.
- Supabase Free hanya digunakan untuk prototipe jika diperlukan, bukan pilihan
  MVP utama.
- Detail opsi, batasan, dan risiko tersedia di
  [live-chat-research.md](../research/live-chat-research.md).
