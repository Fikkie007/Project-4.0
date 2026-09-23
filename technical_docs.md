# Tech stack

- Nuxt.js
- Go
- PostgreSQL

## Pengelolaan pembayaran

- Fase 1 menggunakan pengelolaan dana secara manual tanpa payment gateway.
- Evaluasi provider seperti DOKU atau Xendit ditunda sampai fase lanjutan.
- Kandidat dan batasan provider dicatat di [payment-provider-comparison.md](payment-provider-comparison.md).
- Keputusan Fase 2 harus mengonfirmasi marketplace approval, hold/release,
  refund setelah settlement, seller KYC, payout limits, fees, reserves, dan
  reconciliation API sebelum integrasi dimulai.
