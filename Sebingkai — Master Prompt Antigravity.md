# Sebingkai — Master Prompt untuk Antigravity

*Beda kota, satu bingkai.* Photobooth online bareng teman atau pasangan: tanpa login, gratis, mobile first.

## 1. Peran dan Tujuan

Kamu adalah senior full-stack engineer sekaligus product designer. Bangun web app bernama **Sebingkai**: photobooth virtual real-time untuk 2 orang yang berjauhan dan memakai perangkat berbeda. Keduanya masuk ke satu room lewat link, kamera mereka digabung ke satu bingkai (seolah berada di tempat yang sama) atau dipakai sebagai kamera biasa, lalu mengambil beberapa foto dan menghasilkan photo strip yang bisa di-download.

Seluruh stack harus bisa berjalan di **free tier**. Proyek ini personal dan non-komersial (Vercel Hobby).

## 2. Aturan Kerja untuk Agent

- Kerjakan per fase (Bagian 11). Tiap fase: rencana singkat, implementasi, jalankan `lint`, `typecheck`, `build`, ringkas hasil, lalu lanjut. Berhenti hanya jika ada blocker nyata (misalnya butuh credential).
- Jangan menebak API pihak ketiga dari ingatan. Baca dokumentasi resmi terbaru untuk MediaPipe Tasks Vision, Supabase Realtime, Cloudflare Realtime TURN, Upstash Ratelimit, dan Next.js sebelum memakainya.
- Semua teks UI berbahasa Indonesia yang santai dan hangat. Kode, komentar, dan nama variabel berbahasa Inggris.
- Tidak ada login, akun, atau database foto. Foto **tidak pernah diunggah ke server mana pun**; semuanya diproses di browser dan dikirim langsung antar peserta lewat WebRTC.
- Jangan menambah dependency berat tanpa alasan. Catat keputusan penting di `docs/DECISIONS.md`.
- Bila spesifikasi ambigu, pilih opsi paling sederhana yang memenuhi acceptance criteria dan tulis asumsinya di README.
- Commit kecil dan bermakna per fitur.

## 3. Tech Stack (tetap)

- **Next.js** (App Router, TypeScript strict), di-deploy ke Vercel Hobby.
- **Tailwind CSS** dengan CSS variables untuk design tokens, **Motion** (framer-motion) untuk animasi, `next/font/google` untuk font.
- **Zustand** untuk state, **idb-keyval** untuk IndexedDB.
- **@mediapipe/tasks-vision** (ImageSegmenter, model selfie segmenter). File model dan WASM disajikan dari `/public`, bukan dari CDN pihak ketiga saat runtime.
- **WebRTC native**: RTCPeerConnection dan RTCDataChannel, memakai pola perfect negotiation.
- **Signaling: Supabase Realtime** (Broadcast dan Presence). Tanpa tabel database dan tanpa Auth; cukup anon key. Bungkus dalam interface `SignalingTransport` dan sediakan `LocalBroadcastTransport` (BroadcastChannel) agar bisa dites dua tab tanpa credential.
- **TURN: Cloudflare Realtime TURN**. Kredensial short-lived dibuat di Route Handler `/api/ice`. Jika env tidak ada, fallback ke STUN publik.
- **Rate limit** untuk `/api/ice`: Upstash Redis dengan `@upstash/ratelimit`. Opsional; no-op jika env tidak ada.
- **Test**: Vitest (unit) dan Playwright (e2e dengan fake camera).

## 4. Environment Variables

Buat `.env.example` berisi:

- `NEXT_PUBLIC_SUPABASE_URL`, `NEXT_PUBLIC_SUPABASE_ANON_KEY`
- `CLOUDFLARE_TURN_KEY_ID`, `CLOUDFLARE_TURN_API_TOKEN`
- `UPSTASH_REDIS_REST_URL`, `UPSTASH_REDIS_REST_TOKEN` (opsional)

Aplikasi harus tetap jalan (mode dev dan fallback) tanpa semuanya terisi, dengan pesan error yang jelas bila signaling tidak tersedia.

## 5. Desain dan Tema UI

**Tema: Retro Photobooth x Sticker Pop** (neo-brutalism versi lembut). Terasa seperti photobooth analog di mal tahun 2000-an: kertas krem, tinta hitam, merah booth, stiker, dan strip film.

**Design tokens**

- Paper (background): `#FFF4E0`
- Ink (teks, border): `#1B1A17`
- Booth Red (primary): `#E63B2E`
- Butter Yellow (accent): `#FFC93C`
- Bubblegum Pink (accent): `#FF9EBB`
- Mint (secondary): `#8FD6BC`
- Booth Dark (layar kamera): `#14110F`

**Tipografi**: Fraunces (judul, chunky retro serif), Plus Jakarta Sans (body), DM Mono (kode room, timestamp, label kecil ala tiket).

**Gaya komponen**

- Border tinta 2px, radius 16 sampai 24px, bayangan keras offset `4px 4px 0 var(--ink)`. Efek tekan: bayangan mengecil dan elemen bergeser saat di-tap.
- Tombol utama berbentuk stiker (merah, teks krem). Chip filter, timer, dan background berbentuk stiker kecil.
- Motif: perforasi film di tepi, strip foto miring 2 sampai 3 derajat, pita tape di sudut kartu, tekstur grain kertas sangat halus (SVG noise, opacity rendah).
- Layar booth memakai warna gelap supaya fokus ke kamera; UI di sekitarnya tetap bergaya stiker.
- Animasi: spring ringan, countdown besar dengan efek pop, flash putih saat jepret, strip yang keluar seperti dari slot mesin. Hormati `prefers-reduced-motion`.

## 6. Landing Page

Mobile first, menarik, dan ringan (animasi tidak boleh menghambat LCP). Bagian-bagian:

1. **Hero**: judul *Beda kota, satu bingkai.* dengan subjudul singkat (photobooth online bareng teman atau pasangan, tanpa login, tanpa install). Dua CTA: **Buat Room** (utama) dan **Gabung pakai Kode**. Link kecil **Coba sendiri dulu** menuju mode solo. Visual: mockup mesin booth dan strip foto beranimasi, dibuat dengan SVG dan CSS tanpa foto stok.
2. **Cara kerja** dalam 3 langkah: Buat room dan bagikan link; Nyalakan kamera, pilih background dan timer; Jepret, pilih strip, download.
3. **Fitur**: satu bingkai tanpa harus sama tempat, timer 3/5/10 detik, filter, strip bisa ganti kapan saja, retake, tanpa login.
4. **Contoh hasil**: carousel strip contoh (placeholder dibuat dengan SVG).
5. **Privasi**: foto tidak diunggah ke server, diproses di perangkatmu.
6. **FAQ** (accordion): perlu install? aman? kenapa kamera tidak muncul? berapa orang? cocok di HP?
7. **Footer** sederhana.

SEO dasar: metadata, Open Graph image, `lang=id`.

## 7. Booth UI dan Responsif

Pendekatan **mobile first** (mulai dari 360px), lalu tablet dan desktop.

- Gunakan `dvh` dan `env(safe-area-inset-*)`; tidak boleh ada scroll horizontal; target sentuh minimal 44px.
- **Mobile (portrait)**: top bar (kode room, tombol salin link, status koneksi), canvas preview di tengah dengan rasio mengikuti slot strip, kontrol di bawah: tombol shutter besar, chip timer, tombol background, filter, strip, dan balik kamera. Pengaturan dibuka sebagai **bottom sheet** dengan thumbnail.
- **Desktop**: preview besar di kiri, panel pengaturan di kanan.
- **Landscape di HP**: kontrol dipindah ke sisi.
- Layar **Lobby**: kode room, tombol salin dan bagikan link (Web Share API), indikator menunggu teman, preview kamera sendiri.
- Layar **Izin kamera**: penjelasan singkat sebelum meminta izin, serta penanganan error (ditolak, tidak ada kamera, kamera dipakai aplikasi lain).
- Deteksi **in-app browser** (Instagram, WhatsApp, dan sejenisnya) yang sering memblokir kamera; tampilkan arahan untuk membuka di Chrome atau Safari.
- Pakai Screen Wake Lock agar layar tidak mati saat sesi berlangsung.

## 8. Fitur

**F1. Tanpa login.** Room dibuat dengan kode acak 6 karakter (base32, tanpa karakter yang mirip). Pembuat room menjadi host, yang bergabung menjadi guest. Maksimal 2 orang; orang ketiga melihat pesan *Room penuh*.

**F2. Mode tampilan (switch kapan saja, tersinkron).**

- **Satu Bingkai**: orang dipisahkan dari background lalu ditempatkan di satu background yang sama.
- **Kamera Biasa**: tanpa segmentasi; dua kamera tampil berdampingan atau atas-bawah dalam bingkai.
- **Background**: warna polos, gradient, beberapa scene SVG buatan sendiri (langit senja, pantai minimalis, kamar, bintang, dan sejenisnya), serta **upload gambar sendiri**. Sediakan registry `src/config/backgrounds.ts` dan folder `public/backgrounds/` supaya gambar baru mudah ditambah. Upload custom di-downscale (maksimal 1280px, JPEG) lalu dikirim ke peer agar kedua sisi memakai gambar yang sama.
- Opsional (nice-to-have): geser dan cubit untuk mengatur posisi serta skala tiap orang.

**F3. Timer 3, 5, dan 10 detik.** Default 5 detik. Countdown tampil serempak di kedua perangkat (lihat sinkronisasi di 9.5). Setelah tiap jepretan, tampilkan preview singkat (sekitar 1,5 detik) lalu otomatis lanjut ke foto berikutnya (opsi **Auto lanjut** default aktif). Jumlah foto per sesi mengikuti strip terpilih, default 4.

**F4. Filter** (di booth dan setelah strip jadi). Normal, Hitam Putih, Sepia, Vintage, Film (grain dan vignette), Pastel, Cool, Warm, Dramatic, Glow. Filter tampil langsung di preview booth, dan bisa diganti lagi di layar hasil.

**F5. Ganti strip kapan saja.** Strip dibuat sendiri oleh pemilik di Canva (lihat 9.8 dan `docs/STRIP_GUIDE.md`). Di layar hasil, user bisa memilih strip lain lewat carousel dan hasilnya dirender ulang dari foto yang sama, tanpa foto ulang. Jika strip punya slot lebih sedikit dari jumlah foto, user memilih foto mana mengisi slot mana (tap slot lalu tap foto). Strip dengan slot lebih banyak dari jumlah foto dinonaktifkan dengan keterangan *butuh X foto*.

**F6. Retake.** Setelah semua foto diambil, tampilkan grid foto. Tap sebuah foto lalu **Ulangi** untuk mengambil ulang hanya foto itu (timer dan filter tetap), atau **Ulangi semua**. Retake juga bisa dari layar hasil. Status retake tersinkron di kedua sisi.

**F7. Hasil dan ekspor.** Preview strip besar, pilihan filter, pilihan strip, tombol **Download** (PNG, JPG sebagai opsi), **Bagikan** (Web Share API bila tersedia), dan **Foto lagi**. Masing-masing peserta bisa mengunduh sendiri.

**F8. Mode Solo.** Rute `/solo`: seluruh alur yang sama untuk satu orang tanpa jaringan. Berguna untuk latihan dan dasar pengembangan bertahap.

**F9. Ketahanan sesi.** Foto mentah disimpan di IndexedDB sehingga refresh halaman tidak menghilangkan hasil. Bila koneksi putus, tampilkan banner, lakukan ICE restart otomatis, dan izinkan bergabung lagi dengan kode yang sama.

## 9. Arsitektur Teknis

### 9.1 Rute

`/` landing, `/solo` booth solo, `/r/[code]` booth multiplayer (rute `/new` membuat kode lalu redirect), `/api/ice` kredensial TURN, `/api/keepalive` ping ringan ke Supabase.

### 9.2 Signaling

Interface `SignalingTransport` dengan `join(room)`, `send(msg)`, `onMessage`, dan `presence`. Signaling hanya dipakai untuk SDP, ICE, dan hello awal. Setelah DataChannel terbuka, semua pesan aplikasi lewat DataChannel.

### 9.3 WebRTC

- Perfect negotiation, ICE server dari `/api/ice` (di-cache sampai kedaluwarsa), ICE restart saat putus.
- Satu video track kirim dan satu DataChannel reliable (`ctrl`). Transfer blob (foto, background) memakai chunk dengan framing dan ack sendiri.
- Batasi resolusi maksimal 720p dan bitrate sekitar 1,2 sampai 2 Mbps lewat `RTCRtpSender.setParameters`.
- Tampilkan indikator koneksi (direct atau relay, RTT).

### 9.4 Pipeline Mode Satu Bingkai

Video WebRTC tidak mendukung alpha, jadi gunakan paket **side-by-side**.

- **Sender**: kamera, mirror opsional, MediaPipe ImageSegmenter (input sekitar 256 sampai 384px, 20 sampai 30 fps, `requestVideoFrameCallback`), haluskan mask (temporal smoothing dan feather ringan), lalu gambar ke satu canvas selebar 2W: kiri RGB, kanan mask grayscale. Kirim lewat `canvas.captureStream(24)` dan `replaceTrack`.
- **Receiver dan preview lokal**: WebGL shader membaca texture paket dan menghasilkan RGBA (alpha dari mask dengan smoothstep agar tepi tajam), lalu menggambarnya di atas background pada canvas komposit. Tidak ada operasi per-pixel di CPU pada jalur render. Peserta lokal memakai jalur yang sama supaya preview identik.
- **Mode Kamera Biasa**: tanpa segmentasi, sender mengirim track kamera mentah.
- **Adaptif**: ukur waktu per frame; jika melebihi budget, turunkan fps dan resolusi segmentasi; jika tetap berat, sarankan Mode Kamera Biasa.
- Muat MediaPipe secara lazy hanya saat Mode Satu Bingkai dipakai.
- Layout default: host di kiri, guest di kanan, posisi kaki sejajar bawah, skala otomatis agar kepala tidak terpotong, overlap tipis diperbolehkan. Rasio canvas mengikuti rasio slot strip terpilih.

### 9.5 Sinkronisasi State (host-authoritative)

State: `mode`, `backgroundId`, `filterId`, `timerSec`, `stripId`, `shotCount`, `phase`, meta `shots[]`, `countdownEndsAt` (jam host), dan `version`. Guest mengirim `intent` (misalnya setTimer, requestShot, requestRetake(i)); host memvalidasi, memperbarui, lalu menyiarkan snapshot. Estimasi selisih jam dengan ping-pong agar countdown tampil serempak.

State machine: `lobby` > `ready` > `countdown` > `capturing` > `review` > (ulang sampai jumlah foto cukup) > `result`, dengan cabang `retaking(i)`.

### 9.6 Pengambilan Foto

Saat countdown mencapai 0 pada host: efek flash, capture komposit beresolusi penuh (lebar 1080px, tinggi mengikuti rasio) **tanpa filter**, simpan sebagai JPEG kualitas sekitar 0,92, kirim ke guest dengan chunk dan ack. Kedua sisi menyimpan `shots[]` di IndexedDB. Pengembangan lanjutan: guest mengirim frame resolusi penuh saat jepret agar kualitas sisi guest tidak terkompresi.

### 9.7 Filter

Satu implementasi **WebGL** (fragment shader: color matrix, contrast, saturation, tint, vignette, grain, soft glow) dipakai untuk preview langsung **dan** render akhir supaya hasilnya identik. **Jangan bergantung pada `ctx.filter`** karena tidak konsisten di Safari. Foto disimpan tanpa filter dan filter diterapkan saat render, sehingga bisa diganti kapan saja. Fallback bila WebGL gagal: color matrix pada ImageData.

### 9.8 Sistem Strip

Strip didesain pemilik di Canva sebagai gambar overlay dengan area foto kosong. Skema:

```ts
type StripTemplate = {
  id: string;          // slug unik
  name: string;
  src: string;         // /strips/xxx.png (overlay dengan lubang transparan)
  width: number;       // ukuran asli px
  height: number;
  slots: { x: number; y: number; w: number; h: number }[]; // urut atas ke bawah
  thumb?: string;
};
```

**Script `pnpm strips:scan`** (dev tool, pakai sharp atau pngjs): baca semua PNG di `public/strips-src/`, deteksi area lubang secara otomatis, hasilkan `public/strips/*.png`, thumbnail, dan `src/config/strips.generated.ts`. Aturan deteksi: piksel dianggap lubang bila alpha rendah **atau** warna mendekati **magenta `#FF00FF`** (toleransi kecil). Piksel magenta diubah menjadi transparan pada PNG hasil. Ini mengakomodasi desain Canva tanpa fitur background transparan (yang setahu saya butuh Canva Pro): cukup isi area foto dengan kotak magenta solid. Cari connected component, buang noise, urutkan atas ke bawah lalu kiri ke kanan, dan beri error jelas bila slot tidak persegi panjang.

**Renderer**: canvas seukuran template asli; gambar tiap foto ke slotnya (cover-fit, crop tengah) dengan filter; gambar overlay PNG di atasnya; ekspor lewat `toBlob`. Filter hanya diterapkan pada foto, bukan pada desain template. Sediakan 2 strip contoh buatan sendiri sebagai placeholder (3 slot dan 4 slot).

Karena rasio foto mengikuti slot strip terpilih saat pengambilan, tampilkan saran di `docs/STRIP_GUIDE.md`: gunakan **rasio slot yang sama** di semua strip agar bisa ditukar bebas. Bila rasio berbeda, lakukan crop tengah dan tampilkan peringatan.

### 9.9 Penyimpanan

IndexedDB: foto mentah, background custom, pengaturan terakhir. Tidak ada cookie, tidak ada analytics.

## 10. Struktur Folder (saran)

```
src/app/            rute (landing, solo, r/[code], api/ice, api/keepalive)
src/components/     ui (stiker, sheet, chip), booth, landing, result
src/lib/rtc/        peer, signaling (supabase, local), datachannel, blob transfer
src/lib/vision/     segmenter, packer, shader komposit
src/lib/render/     filter shader, strip renderer, capture
src/lib/store/      zustand, idb
src/config/         backgrounds.ts, filters.ts, strips.generated.ts
scripts/            strips-scan
public/             models, wasm, strips-src, strips, backgrounds
docs/               DECISIONS.md, STRIP_GUIDE.md
```

## 11. Fase Pengerjaan dan Acceptance Criteria

**Fase 0 — Setup dan design system.** Project Next.js, Tailwind tokens, font, komponen dasar bergaya stiker, lint dan typecheck. *Selesai bila* halaman contoh menampilkan semua komponen dasar dan `build` lolos.

**Fase 1 — Landing page.** Semua bagian di Bagian 6. *Selesai bila* tampil rapi dari 360px sampai desktop, Lighthouse Mobile Performance minimal 90, tidak ada scroll horizontal.

**Fase 2 — Booth Solo.** Kamera, balik kamera, Mode Satu Bingkai dengan background, Kamera Biasa, timer, filter live, pengambilan foto, retake, dan strip sederhana dengan download. *Selesai bila* seluruh alur `/solo` bekerja di Chrome Android dan Safari iOS tanpa jaringan.

**Fase 3 — Sistem strip.** Script scan, template picker, slot assignment, ganti strip setelah foto jadi, filter di layar hasil, ekspor dan share. *Selesai bila* strip magenta dan strip alpha sama-sama terdeteksi benar dan strip bisa diganti tanpa foto ulang.

**Fase 4 — Multiplayer.** Signaling (local dan Supabase), `/api/ice`, WebRTC, paket side-by-side, state sync, countdown serempak, transfer foto dan background. *Selesai bila* dua tab (transport lokal) dan dua perangkat berbeda lewat internet bisa menyelesaikan satu sesi penuh dengan hasil strip identik di kedua sisi.

**Fase 5 — Hardening.** Reconnect, error state, in-app browser warning, performa adaptif, aksesibilitas, wake lock, rate limit. *Selesai bila* putus-sambung koneksi tidak menghilangkan foto dan UI tetap responsif di HP kelas menengah ke bawah.

**Fase 6 — Deploy dan dokumentasi.** README dengan langkah setup Supabase, Cloudflare TURN, Upstash, dan Vercel; `.env.example`; `docs/STRIP_GUIDE.md`; `vercel.json` dengan cron harian untuk `/api/keepalive` (cek batasan cron Hobby terbaru) agar project Supabase free tidak di-pause karena tidak aktif.

## 12. Non-Functional dan Pengujian

- **Performa**: halaman landing tidak memuat MediaPipe; JS awal landing sekecil mungkin; booth memuat model secara lazy.
- **Aksesibilitas**: kontras cukup, label pada semua kontrol, countdown memakai `aria-live`, navigasi keyboard di desktop, dukung `prefers-reduced-motion`.
- **Privasi**: tidak ada unggah foto, tidak ada analytics; halaman Privasi singkat menjelaskan hal ini.
- **Keamanan**: kode room acak dan sulit ditebak; rate limit pada `/api/ice`; kredensial TURN hanya dibuat server-side.
- **Unit test (Vitest)**: deteksi slot, penempatan foto ke slot, reducer state machine, chunking transfer.
- **E2E (Playwright)**: dua browser context dengan fake media device (`--use-fake-device-for-media-stream --use-fake-ui-for-media-stream`) menyelesaikan sesi penuh, termasuk retake dan ganti strip.
- Uji manual di Chrome Android, Safari iOS, dan Chrome desktop; catat hasilnya di README.

## 13. Deliverables

Repo siap deploy ke Vercel; README lengkap; `.env.example`; `docs/DECISIONS.md`; `docs/STRIP_GUIDE.md` (ukuran rekomendasi, cara memakai kotak magenta di Canva, rasio slot yang sama, cara menjalankan `pnpm strips:scan`); dua strip contoh dan minimal 8 background bawaan.
