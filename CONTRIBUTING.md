# Panduan Kontribusi

Terima kasih atas minat Anda untuk berkontribusi pada proyek Second Brain! Kontribusi Anda sangat berarti dan akan membantu kami membuat proyek ini lebih baik.

Berikut adalah beberapa panduan untuk membantu Anda memulai.

## Kode Etik

Proyek ini dan semua orang yang berpartisipasi di dalamnya diatur oleh Kode Etik Second Brain. Dengan berpartisipasi, Anda diharapkan untuk menjunjung tinggi kode ini.

## Bagaimana Berkontribusi

### Melaporkan Bug

Jika Anda menemukan bug, harap buka issue baru di GitHub. Pastikan untuk menyertakan:
*   Deskripsi yang jelas dan ringkas mengenai bug tersebut.
*   Langkah-langkah untuk mereproduksi perilaku tersebut.
*   Perilaku yang diharapkan.
*   Tangkapan layar atau log jika relevan.
*   Versi proyek dan lingkungan Anda (Node.js, pnpm, OS).

### Menyarankan Fitur Baru

Kami sangat senang mendengar ide-ide Anda! Jika Anda memiliki saran untuk fitur baru:
*   Buka issue baru di GitHub.
*   Jelaskan fitur yang Anda inginkan dan mengapa itu akan bermanfaat bagi proyek.
*   Berikan contoh penggunaan atau kasus penggunaan jika memungkinkan.

### Mengajukan Pull Request (PR)

1.  **Fork** repositori ini ke akun GitHub Anda.
2.  **Kloning** fork Anda ke mesin lokal: `git clone git@github.com:agvst/second-brain.git` (Ganti `agvst` dengan nama pengguna GitHub Anda).
3.  **Buat cabang baru** untuk pekerjaan Anda: `git checkout -b feature/nama-fitur-anda` atau `git checkout -b bugfix/deskripsi-bug`.
4.  **Lakukan perubahan** pada kode Anda. Pastikan untuk mengikuti standar kode yang ada.
5.  **Jalankan tes** untuk memastikan semua yang ada berfungsi dengan baik: `pnpm test` (atau perintah tes spesifik seperti yang ada di `README.md`).
6.  **Commit perubahan Anda** dengan pesan commit yang jelas dan deskriptif. (Misalnya: `feat: Menambahkan fitur X`, `fix: Memperbaiki bug Y`).
7.  **Push cabang Anda** ke fork GitHub Anda: `git push origin feature/nama-fitur-anda`.
8.  **Buka Pull Request** baru dari fork Anda ke repositori utama. Jelaskan perubahan Anda secara rinci, mengapa perubahan itu diperlukan, dan masalah apa yang dipecahkan.

## Lingkungan Pengembangan

Lihat bagian "Setup" dan "Run" di `README.md` untuk informasi tentang cara menyiapkan lingkungan pengembangan lokal Anda.

## Standar Kode

*   Kami menggunakan TypeScript, jadi pastikan kode Anda ditulis dengan tipe yang kuat.
*   Ikuti gaya penulisan kode yang sudah ada dalam proyek. Linting dan pemformatan otomatis mungkin akan diberlakukan.
*   Pastikan semua tes lolos sebelum mengajukan PR.