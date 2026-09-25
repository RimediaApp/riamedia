# PANDUAN SETUP RIMEDIA — Database, PWA, dan APK

## 1. Setup Database Supabase (WAJIB)

1. Buat akun di https://supabase.com → **New Project** (gratis).
2. Buka **Project Settings → API**, salin **Project URL** dan **anon public key**.
3. Buka file `index.html`, cari bagian paling atas class `AppConfig` (di dalam `<script>`), lalu ganti:
   - `SUPABASE_URL` → Project URL Anda
   - `SUPABASE_ANON_KEY` → anon public key Anda
4. Buka **SQL Editor** di dashboard Supabase, jalankan SQL ini:

```sql
-- Tabel data utama (pesan realtime, media, file portal)
create table if not exists nexus_data (
  id bigint generated always as identity primary key,
  tipe_data text,
  json_data jsonb,
  created_at timestamptz default now()
);
alter table nexus_data enable row level security;
create policy "auth_all" on nexus_data
  for all to authenticated using (true) with check (true);

-- Tabel branding aplikasi (nama, logo, warna — bisa diubah tanpa edit kode)
create table if not exists app_settings (
  id int primary key,
  app_name text,
  logo_url text,
  theme_color text,
  updated_at timestamptz default now()
);
alter table app_settings enable row level security;
create policy "read_all" on app_settings for select using (true);
create policy "auth_write" on app_settings
  for all to authenticated using (true) with check (true);
insert into app_settings (id, app_name, logo_url, theme_color)
values (1, 'Rimedia', 'icons/icon-192.png', '#050511')
on conflict (id) do nothing;
```

5. Aktifkan **Realtime**: buka **Database → Replication** → aktifkan untuk tabel `nexus_data`
   (atau jalankan: `alter publication supabase_realtime add table nexus_data;`)
6. Buat **Storage**: buka **Storage → New Bucket** → nama: `attachments` → centang **Public bucket**.
   Tambahkan policy: di bucket `attachments` → Policies → New Policy → allow INSERT & SELECT untuk `authenticated`.

## 2. Menjalankan sebagai PWA (Online + Offline)

Upload folder ini (`index.html`, `manifest.webmanifest`, `sw.js`, folder `icons/`)
ke hosting HTTPS apa pun, misalnya **GitHub Pages, Netlify, Vercel, atau Firebase Hosting**
(gratis, cukup drag & drop folder). HTTPS wajib untuk kamera, mikrofon, dan sidik jari.

- Buka URL-nya di Chrome Android → menu ⋮ → **Instal Aplikasi / Add to Home Screen**.
- Offline: chat, CRM, dan riwayat tetap bisa dibuka; pesan yang dikirim saat offline
  otomatis terkirim saat online kembali (sinkronisasi tiap 10 detik).

## 3. Mengubah Logo / Nama / Warna Aplikasi

Login → **Pengaturan → Branding Aplikasi** → isi nama, URL logo, warna → Simpan.
Ikon PWA memakai file di folder `icons/`; untuk mengganti ikon instalasi,
ganti file `icons/icon-192.png` dan `icons/icon-512.png` dengan logo Anda.

## 4. Membuat APK (2 cara mudah, tanpa coding Android)

### Cara A — Bubblewrap (disarankan, resmi dari Google)
```bash
npm install -g @bubblewrap/cli
bubblewrap init --manifest https://URL-APLIKASI-ANDA/manifest.webmanifest
bubblewrap build
```
Hasilnya: `app-release-signed.apk` — langsung kirim ke HP dan install.

### Cara B — PWABuilder (paling gampang, tanpa install apa pun)
1. Buka https://www.pwabuilder.com
2. Masukkan URL aplikasi Anda → **Start** → pilih **Android** → **Download Package**.
3. Di dalam ZIP ada file `.apk` — kirim ke HP, install, selesai.

> Catatan: APK hasil TWA membuka aplikasi web Anda full-screen. Semua fitur
> (kamera, mikrofon, video call, sidik jari, notifikasi) berfungsi normal karena
> tetap memakai engine Chrome.

## 5. Fitur & Cara Pakai

| Fitur | Cara pakai |
|---|---|
| Telepon suara | Buka chat kontak → tombol ☎ di header |
| Video call | Buka chat kontak → tombol 📹 di header |
| Pesan suara | Tombol 🎤 di kolom ketik (tekan sekali = rekam, tekan lagi = kirim) |
| Kirim file/gambar di chat | Tombol 📎 di kolom ketik |
| Kirim data HP ⇄ Laptop | Tab **Portal** → unggah file → buka tab Portal yang sama di laptop → Unduh |
| Sidik jari | Pengaturan → **Aktifkan Sidik Jari** (HP Android/Chrome). Setiap buka aplikasi akan diminta sidik jari. |
| Status online kontak | Titik hijau di daftar kontak & header chat |
| Online/Offline | Lencana di header daftar pesan & Pengaturan |
| WhatsApp / FB / IG / TikTok | Tab masing-masing; dibuka di tab browser baru karena kebijakan keamanan situs tersebut melarang iframe |

## 6. Catatan Panggilan

- Panggilan suara/video memakai **WebRTC + Supabase Realtime** (server STUN Google gratis).
- Kedua pihak harus **online** dan menggunakan akun Rimedia masing-masing.
- Jika panggilan sering gagal di jaringan seluler/NAT ketat, tambahkan server **TURN**
  (mis. gratis dari metered.ca) ke `AppConfig.TURN` di `index.html`.
- "Telepon" antar HP dilakukan lewat akun kontak Rimedia (seperti WhatsApp),
  bukan nomor GSM.
