# Riset opsi WhatsApp OTP gratis untuk MVP

Tanggal akses sumber: 2026-09-24
Konteks: MVP Project-4.0, OTP untuk registrasi/login pengguna di Indonesia.
Ruang lingkup: Meta WhatsApp Cloud API, Twilio/alternatif resmi, WhatsApp Business App, dan risiko solusi unofficial.

## Kesimpulan

1. **Tidak ada jalur WhatsApp OTP produksi yang benar-benar gratis tanpa batas.** Meta Cloud API tidak mengenakan biaya platform bulanan yang terpisah, tetapi pesan authentication tetap mengikuti rate card Meta. Hosting, nomor, observability, dan engineering juga tetap berbiaya.
2. **Pilihan paling murah untuk MVP yang benar-benar ingin memakai WhatsApp:** integrasi langsung ke **Meta WhatsApp Cloud API**. Untuk Indonesia, rate card snapshot yang dipublikasikan Twilio pada saat riset menunjukkan **USD 0,025 per authentication message**, sebelum biaya infrastruktur. Konfirmasi angka terakhir di rate card Meta sebelum mengaktifkan billing.
3. **Pilihan paling cepat:** **Twilio Verify WhatsApp**. API OTP dan validasi sudah tersedia, tetapi biayanya lebih tinggi: **USD 0,05 per verifikasi sukses + biaya channel WhatsApp**, yang pada snapshot Twilio adalah **USD 0,0034 per authentication template message di AS**. Harga channel dan cakupan negara perlu dicek untuk Indonesia.
4. **WhatsApp Business App bukan API OTP.** App gratis untuk penggunaan bisnis manual, tetapi tidak memberi endpoint server-side untuk menghasilkan, mengirim, dan memvalidasi OTP dari aplikasi MVP.
5. **Hindari library unofficial yang mengotomasi WhatsApp Web atau aplikasi konsumen.** Ini dapat melanggar kebijakan WhatsApp, membuat nomor/WABA diblokir, dan tidak memberi jaminan delivery, audit, atau pemulihan yang layak untuk autentikasi.

## Keputusan untuk MVP gratis

Gunakan **mock OTP untuk development**, lalu **Twilio WhatsApp trial untuk demo tertutup** jika perlu menguji delivery sungguhan. Trial hanya cocok untuk beberapa nomor tim: penerima harus diverifikasi, jumlah penerima dibatasi, dan trial berakhir setelah masa trial.

Untuk MVP publik, pilih **Meta WhatsApp Cloud API direct** dan siapkan budget pay-as-you-go. Ini opsi resmi dengan biaya variabel paling rendah yang ditemukan, tetapi tetap bukan gratis. Jika budget benar-benar nol, ubah kanal autentikasi menjadi email OTP/magic link atau undangan admin sampai ada dana untuk WhatsApp.

Tidak ada opsi OSS yang sekaligus menjadi transport WhatsApp resmi dan gratis. OSS masih berguna untuk generator OTP, penyimpanan challenge, rate limiting, dan mock provider; pengiriman WhatsApp tetap melalui API resmi Meta atau provider berbayar.

## Apakah ada opsi benar-benar gratis?

| Arti "gratis" | Jawaban |
|---|---|
| Gratis untuk spike lokal tanpa mengirim OTP sungguhan | Ya. Gunakan fake sender, mock API, atau OTP yang dicatat di log lokal. Ini bukan pengujian delivery WhatsApp. |
| Gratis untuk mengirim OTP sungguhan tanpa batas ke pengguna | Tidak ditemukan pada sumber resmi yang ditinjau. |
| Gratis dari biaya platform provider | Meta Cloud API paling mendekati: tidak ada markup BSP atau biaya langganan provider pada jalur direct API, tetapi Meta tetap menagih pesan authentication. |
| Gratis sementara untuk proof of concept | Twilio trial menyediakan unit gratis, tetapi trial dibatasi 30 hari, penerima harus diverifikasi, dan WhatsApp trial memakai template bawaan Twilio. Ini bukan jalur produksi. |
| Gratis dengan WhatsApp Business App | App dapat dipakai manual, tetapi tidak menyelesaikan kebutuhan API OTP otomatis. |

## Perbandingan singkat

| Opsi | Biaya pesan OTP | Free tier / trial | Template dan OTP | Syarat utama | Kesesuaian MVP |
|---|---|---|---|---|---|
| **Meta WhatsApp Cloud API** | Meta authentication rate per negara. Snapshot Indonesia: **USD 0,025/pesan** melalui rate card yang direferensikan provider. Tidak ada biaya markup BSP. | Tidak ada free tier production yang dijanjikan untuk authentication pada sumber yang ditinjau. | OTP harus dikirim sebagai authentication template yang disetujui; template memiliki format dan batasan keamanan Meta, termasuk tombol copy code pada format yang didukung. | Meta Business Portfolio, WABA, business phone number, Meta app, access token/system user, webhook, dan template approval. Messaging limit serta verifikasi bisnis dapat membatasi kenaikan skala. | **Terbaik untuk biaya** bila tim siap mengurus onboarding, token, webhook, template, retry, dan monitoring sendiri. |
| **Twilio Programmable Messaging for WhatsApp** | **USD 0,005/pesan** biaya Twilio, ditambah biaya template Meta. Snapshot Twilio untuk Indonesia: Meta authentication **USD 0,025/pesan**, sehingga indikasi total **USD 0,030/pesan** sebelum pajak/add-on. | Trial: 100 WhatsApp messages, berlaku 30 hari; maksimal 5 penerima terverifikasi dan pembatasan trial lain. | Custom template perlu dibuat dan dikirim untuk approval. Authentication template memiliki body yang ditentukan WhatsApp dan wajib Copy Code button. | Akun Twilio, WABA, sender WhatsApp, Messaging Service, template approved, dan webhook status. | Baik bila tim ingin API messaging umum dan tidak membutuhkan lifecycle OTP khusus. Lebih mahal dari direct Meta. |
| **Twilio Verify WhatsApp** | **USD 0,05 per successful verification** + biaya channel. Halaman harga Twilio menampilkan **USD 0,0034 per authentication template message di AS**; harga/rute Indonesia harus dikonfirmasi di akun. | Trial Twilio tetap dibatasi oleh free units dan aturan trial. Tidak sama dengan free production tier. | Verify memakai authentication template yang dikelola/diotomatisasi Twilio; tidak mendukung PSD2, pre-approved, atau custom templates menurut dokumentasi Verify WhatsApp. | Verify Service, sender WhatsApp milik sendiri, WABA, dan proses Bring Your Own Sender. Twilio menyebut generic sender tidak boleh dipakai sejak 1 Maret 2024. | **Terbaik untuk time-to-market OTP**, terutama bila nanti perlu fallback ke SMS atau channel lain. Biaya dasar lebih tinggi. |
| **360dialog WhatsApp API** | **EUR 49/nomor/bulan** untuk plan Regular, ditambah Meta WhatsApp Messaging Fees. Tidak ada markup pesan Meta menurut halaman harga. | Tidak ada free tier production yang tercantum. | Mendukung semua jenis pesan API sesuai compliance Meta; template tetap tunduk pada approval Meta. | Nomor/WABA, onboarding 360dialog, dan Meta compliance. | Alternatif resmi bila ingin BSP dengan API langsung, tetapi biaya tetap bulanan membuatnya kurang cocok untuk MVP ber-volume sangat kecil. |
| **Vonage Messages API** | Harga WhatsApp dihitung per delivered message, ditambah Vonage platform fee dan biaya provider/Meta. Halaman harga publik yang ditinjau tidak menampilkan angka WhatsApp spesifik untuk Indonesia. | Ada tombol free trial, tetapi tidak ditemukan kuota WhatsApp produksi gratis yang dapat dijadikan dasar perencanaan. | Template dan aturan WhatsApp tetap berlaku; gunakan API Messages dan approved template untuk pesan yang dimulai bisnis. | Akun Vonage, sender/WABA, onboarding, dan konfigurasi channel. | Layak sebagai alternatif enterprise/omnichannel, tetapi perlu quotation tertulis sebelum dibandingkan secara biaya. |
| **WhatsApp Business App** | App manual, tanpa biaya API per pesan. Biaya nomor, perangkat, dan operasi tetap ada. | App gratis untuk penggunaan manual, bukan free API tier. | Pesan manual tidak menjadi authentication template API. Tidak ada alur resmi server-side untuk template OTP otomatis. | Nomor WhatsApp Business App dan perangkat/akun bisnis. | Cocok untuk customer support manual skala kecil; **tidak cocok sebagai backend OTP**. |
| **Unofficial WhatsApp Web automation** | Sering tampak gratis selain hosting. | Tidak ada SLA, quota, atau free tier resmi. | Tidak ada jaminan template/authentication policy atau approval. | Biasanya memakai sesi WhatsApp Web, QR login, reverse-engineered protocol, atau akun konsumen. | **Jangan dipakai untuk OTP produksi.** Risiko pemblokiran dan kegagalan delivery lebih besar daripada penghematan biaya. |

Harga di atas adalah snapshot, bukan penawaran komersial. Provider dapat mengubah rate, pajak, minimum spend, currency, dan eligibility tanpa perubahan pada kode aplikasi.

## Detail opsi

### 1. Meta WhatsApp Cloud API

#### Model biaya

- Meta membedakan kategori pesan, termasuk authentication, utility, marketing, dan customer service/free-form.
- OTP login/registrasi adalah use case authentication, bukan utility atau marketing.
- Biaya Meta bergantung pada negara tujuan dan kategori. Rate card harus dibaca berdasarkan country code penerima, bukan lokasi perusahaan.
- Rate card snapshot dari CSV harga Twilio yang ditautkan pada halaman resmi Twilio mencantumkan baris Indonesia: marketing USD 0,0411, utility USD 0,025, authentication USD 0,025. Untuk OTP, angka yang relevan adalah authentication.
- Direct Cloud API menghilangkan markup per pesan BSP seperti biaya Twilio USD 0,005 atau langganan channel 360dialog. Direct API tetap membutuhkan hosting dan pengelolaan operasional sendiri.

#### Template/authentication

- Pesan yang dimulai bisnis di luar customer service window harus menggunakan template yang disetujui.
- Authentication template memiliki format khusus untuk one-time passcode. Dokumentasi provider yang menggunakan Meta template mensyaratkan Copy Code button dan membatasi body template.
- Jangan mengirim OTP sebagai free-form text hanya untuk menghindari approval. Itu bukan jalur yang didukung untuk pesan bisnis yang dimulai aplikasi.
- Pantau quality rating, template rejection, template pause, messaging limit, dan delivery status. OTP yang gagal terkirim harus memiliki fallback yang aman, misalnya email atau SMS, tanpa memperpanjang masa berlaku OTP secara sembarangan.

#### Syarat akun dan implementasi

Minimum yang perlu disiapkan:

- Meta Business Portfolio dan WhatsApp Business Account (WABA).
- Nomor telepon bisnis yang dapat didaftarkan sebagai sender API. Nomor yang sedang dipakai pada WhatsApp consumer/app perlu mengikuti proses migrasi atau memakai nomor terpisah sesuai aturan Meta.
- Meta app, access token dengan permission yang tepat, phone number ID, dan webhook HTTPS.
- Authentication template yang disubmit/approved serta bahasa yang dibutuhkan pengguna.
- Rate limit, idempotency key internal, OTP TTL, batas percobaan, audit event, dan webhook verification.

Business verification, messaging limit, display name, dan eligibility dapat memengaruhi kapan sender boleh mengirim lebih banyak pesan. Jangan menganggap pembuatan WABA otomatis berarti kapasitas produksi tanpa batas.

### 2. Twilio

Ada dua produk yang perlu dibedakan.

#### Programmable Messaging API

Twilio mengenakan biaya platform **USD 0,005 untuk setiap pesan WhatsApp**, inbound maupun outbound, lalu meneruskan biaya template Meta. Untuk Indonesia, snapshot Twilio yang diakses pada 2026-09-24 menampilkan Meta authentication USD 0,025/pesan, sehingga biaya nominal per OTP outbound sekitar USD 0,030 sebelum pajak dan biaya lain.

Twilio menyatakan bahwa free-form hanya boleh selama customer service window 24 jam setelah user menginisiasi percakapan. OTP login yang dimulai aplikasi biasanya berada di luar kondisi itu dan perlu authentication template.

#### Verify WhatsApp

Verify menghasilkan dan memvalidasi OTP, menyediakan lifecycle verification, dan dapat dipakai dengan `channel=whatsapp`. Harga yang ditampilkan Twilio pada saat riset adalah:

- USD 0,05 per successful verification.
- Ditambah channel fee.
- WhatsApp: USD 0,0034 per authentication template message untuk AS pada halaman harga yang ditinjau.

Verify mengurangi pekerjaan backend, tetapi biaya fixed per verification membuatnya perlu dibandingkan dengan direct Cloud API. Twilio merekomendasikan Verify untuk use case OTP, terutama bila ingin orkestrasi WhatsApp, SMS, RCS, atau channel lain.

#### Trial

Dokumentasi trial Twilio menyebut:

- Tanpa kartu kredit untuk pendaftaran.
- 100 WhatsApp messages sebagai product-specific free units.
- Trial berakhir setelah 30 hari.
- Penerima harus diverifikasi, maksimal lima nomor menurut batasan trial yang didokumentasikan.
- Trial memakai template bawaan Twilio; custom WhatsApp templates tidak tersedia selama trial.

Kesimpulannya, trial cocok untuk menguji wiring dan delivery ke nomor tim, bukan untuk pilot pengguna publik.

### 3. WhatsApp Business App

WhatsApp Business App adalah produk untuk bisnis kecil yang menangani chat melalui aplikasi. Ia dapat dipakai tanpa biaya API per pesan, tetapi model operasinya manual.

Batasan yang relevan untuk OTP:

- Tidak ada endpoint resmi untuk backend menghasilkan dan mengirim OTP secara programmatic.
- Tidak ada webhook API yang setara Cloud API untuk menyimpan status pengiriman dan delivery evidence.
- Tidak ada template authentication API atau Copy Code flow yang dapat dikontrol aplikasi MVP.
- Otomasi melalui accessibility automation, emulator, WhatsApp Web scripting, atau scraping mengubah solusi menjadi unofficial dan membawa risiko kebijakan.

Business App dapat menjadi kanal support manual di samping OTP provider. Ia bukan pengganti Cloud API.

### 4. Alternatif resmi selain Twilio

#### 360dialog

360dialog adalah BSP/API provider yang menampilkan harga publik **EUR 49 per nomor per bulan** untuk Regular, di luar Meta WhatsApp Messaging Fees. Ini berarti volume sangat kecil dapat lebih mahal daripada direct Meta Cloud API karena ada subscription fee. Nilainya ada pada support, escalation path, dan API access tanpa markup pesan Meta yang dinyatakan pada halaman pricing.

#### Vonage

Vonage Messages API menyatakan bahwa WhatsApp dihitung per delivered message dan semua channel memiliki Vonage platform fee di atas biaya provider/channel. Halaman harga publik yang ditinjau tidak memperlihatkan angka WhatsApp Indonesia. Minta quotation yang memisahkan:

- Meta fee authentication Indonesia.
- Vonage platform fee.
- Setup atau monthly minimum.
- Sender/WABA onboarding.
- Fallback SMS dan delivery report.

Tanpa angka tertulis itu, Vonage tidak dapat disebut lebih murah dari direct Meta atau Twilio.

## Risiko solusi unofficial

Risiko utama bukan hanya kualitas kode:

1. **Pelanggaran kebijakan.** WhatsApp Business Messaging Policy mengharuskan bisnis menggunakan jalur bisnis yang sah dan melarang perilaku spam, abuse, atau aktivitas yang dapat merugikan pengguna/platform. Reverse-engineering WhatsApp Web, scraping, dan automasi akun konsumen tidak mendapat status resmi dari Meta.
2. **Nomor atau akun diblokir.** OTP adalah alur kritis. Jika sesi Web logout, device session invalid, atau nomor terkena enforcement, seluruh login baru dapat gagal tanpa SLA pemulihan.
3. **Tidak ada kontrak delivery.** Library unofficial biasanya tidak memberi jaminan bahwa status sent/ delivered/ read berarti sama dengan bukti API resmi. Retry yang salah dapat menggandakan OTP atau membuat user menerima kode kedaluwarsa.
4. **Kebocoran kredensial.** Session token, QR session, dan database auth state menjadi secret bernilai tinggi. Kebocoran dapat mengambil alih akun bisnis atau mengirim pesan atas nama bisnis.
5. **Sulit diaudit.** Tidak ada standar resmi untuk template approval, webhook signature, billing record, data retention, atau dispute dengan provider.
6. **Biaya tersembunyi.** Penghematan fee bisa hilang karena akun diblokir, OTP fallback meningkat, operator harus memulihkan sesi, atau tim harus mengganti nomor yang sudah digunakan pengguna.

Untuk OTP, risiko availability dan account recovery cukup untuk menolak unofficial sebagai pilihan MVP publik.

## Rekomendasi implementasi MVP

### Rekomendasi utama

Pilih **Meta WhatsApp Cloud API direct** jika:

- Tim dapat mengalokasikan waktu untuk onboarding Meta dan template approval.
- MVP dapat menerima fallback email/SMS saat WhatsApp gagal.
- Volume awal rendah dan biaya per pesan harus minimum.
- Backend siap menyimpan state OTP sendiri.

Pilih **Twilio Verify WhatsApp** jika:

- Waktu implementasi lebih penting daripada biaya per verifikasi.
- Tim ingin generator/verifier OTP dan lifecycle verification yang dikelola provider.
- Fallback multi-channel akan segera dibutuhkan.

Jangan memilih WhatsApp Business App atau unofficial automation sebagai backend OTP.

### Kontrak internal yang tetap harus dimiliki aplikasi

Provider tidak boleh menjadi satu-satunya sumber state OTP. Simpan minimal:

- `otp_challenge_id`, user/phone hash, purpose, provider, provider message/verification ID.
- Hash OTP, `expires_at`, `attempt_count`, `last_sent_at`, dan status (`pending`, `verified`, `expired`, `locked`).
- Idempotency key untuk request kirim ulang.
- Rate limit per nomor, IP, device, dan account.
- Audit event tanpa menyimpan OTP plaintext di log.
- Webhook signature verification dan deduplication.
- Fallback provider dengan aturan agar satu challenge tidak menerima kode berbeda tanpa invalidasi yang jelas.

Batas awal yang aman untuk MVP: OTP berlaku singkat, percobaan verifikasi dibatasi, resend memakai backoff, dan status provider tidak langsung dianggap sebagai bukti bahwa user telah memverifikasi.

## Checklist sebelum memilih provider

- Minta konfirmasi tertulis rate authentication Indonesia terbaru dan apakah ada pajak/biaya platform tambahan.
- Pastikan sender WhatsApp dapat menggunakan brand dan nomor yang akan dipakai produksi.
- Uji approval authentication template dalam bahasa Indonesia dan Inggris.
- Uji webhook delivery, timeout, duplicate event, rate limit, dan retry.
- Uji nomor yang sudah memakai WhatsApp Business App serta proses migrasinya.
- Tetapkan fallback ketika user tidak memiliki WhatsApp, nomor tidak aktif, atau template/message gagal.
- Pasang budget alert dan usage alert sebelum membuka pendaftaran publik.

## Sumber resmi dan tanggal akses

Semua URL berikut adalah sumber resmi Meta, WhatsApp, Twilio, 360dialog, atau Vonage. Diakses 2026-09-24.

### Meta dan WhatsApp

- [WhatsApp Cloud API overview](https://developers.facebook.com/docs/whatsapp/cloud-api/overview)
- [Cloud API get started](https://developers.facebook.com/docs/whatsapp/cloud-api/get-started)
- [WhatsApp pricing and rate cards](https://developers.facebook.com/docs/whatsapp/pricing)
- [Authentication templates](https://developers.facebook.com/docs/whatsapp/business-management-api/authentication-templates/)
- [WhatsApp messaging limits](https://developers.facebook.com/docs/whatsapp/messaging-limits/)
- [WhatsApp Business Platform](https://business.whatsapp.com/products/business-platform)
- [WhatsApp Business App](https://business.whatsapp.com/products/business-app)
- [WhatsApp Business download](https://www.whatsapp.com/business/download)
- [WhatsApp Business Messaging Policy](https://business.whatsapp.com/policy)

Catatan: beberapa halaman Meta/WhatsApp menggunakan rendering atau proteksi yang tidak mengembalikan isi melalui fetch teks anonim. URL tetap menunjuk ke dokumentasi resmi pemilik produk; angka harga provider di bawah dipakai sebagai snapshot yang dapat dibaca dan dikutip langsung.

### Twilio

- [Twilio WhatsApp pricing](https://www.twilio.com/en-us/whatsapp/pricing)
- [Twilio WhatsApp pricing CSV](https://www.twilio.com/content/dam/twilio-com/pricing-data/en/WhatsAppPricing-pricing-details.csv)
- [Twilio Verify pricing](https://www.twilio.com/en-us/verify/pricing)
- [Verify WhatsApp overview](https://www.twilio.com/docs/verify/whatsapp)
- [WhatsApp template tutorial and authentication requirements](https://www.twilio.com/docs/whatsapp/tutorial/send-whatsapp-notification-messages-templates)
- [Twilio free trial account](https://www.twilio.com/docs/usage/tutorials/how-to-use-your-free-trial-account)
- [Twilio WhatsApp trial](https://www.twilio.com/docs/usage/trials/try-out-whatsapp)

### Alternatif BSP/API resmi

- [360dialog pricing](https://www.360dialog.com/pricing/)
- [Vonage Messages API pricing](https://www.vonage.com/communications-apis/messages/pricing/)
- [Vonage WhatsApp pricing](https://www.vonage.com/communications-apis/messages/features/whatsapp/pricing/)

### Cara membaca angka

- Angka Meta yang muncul pada laporan ini adalah per pesan authentication, bukan harga bulanan.
- Angka Twilio Programmable Messaging adalah Twilio fee + Meta fee.
- Angka Twilio Verify adalah per successful verification + channel fee.
- Angka 360dialog adalah subscription per nomor + Meta fee.
- Angka Vonage tidak diberi nominal Indonesia pada halaman publik yang ditinjau; jangan membuat proyeksi tanpa quotation.
