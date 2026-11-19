# Jurnal Proses — Tugas 9

## Diskusi deployment model
- Opsi yang dipertimbangkan dan kenapa akhirnya memilih salah satu:     
    Kami telah mempertimbangkan keempat model cloud deployment, yaitu Public Cloud, Private Cloud, Community Cloud, dan Hybrid Cloud. Berdasarkan evaluasi kami, Public Cloud tidak dipilih karena infrastrukturnya dikelola oleh pihak ketiga dan digunakan bersama oleh banyak penyewa sehingga tidak memberikan kendali penuh pada negara serta menimbulkan risiko keamanan terhadap data kependudukan yang bersifat rahasia. Private Cloud juga tidak dipilih karena kapasitasnya terbatas pada infrastuktur yang dimiliki, sehingga sistem berisiko mengalami penurunan kinerja saat terjadi lonjakan akses yang ekstrem. Sementara itu, Community Cloud kurang sesuai untuk kebutuhan akses lintas sektor seperti perbankan karena setiap sektor membutuhkan keamanan, regulasi dan pola penggunaan yang berbeda. Oleh karena itu, kami memutuskan untuk memilih **Hybrid Cloud**, yaitu model yang menggabungkan Private Cloud dan Public Cloud dalam satu arsitektur terpadu. Model ini dipilih karena Private Cloud mampu menjaga keamanan, kendali penuh atas data sensitif, serta memenuhi regulasi perlindungan data pribadi. Selain itu, Public Cloud dimanfaatkan untuk layanan seperti portal publik yang mampu melakukan auto-scaling ketika terjadi lonjakan akses pada musim pemilu maupun pendaftaran sekolah.

## Diskusi service model per komponen
- Kami membandingkan tiga service model (IaaS, PaaS, SaaS) untuk setiap komponen sistem. Pertimbangannya adalah tingkat kerahasiaan data, kebutuhan kontrol infrastruktur, serta kemudahan pengelolaan layanan.

| Komponen | Service Model | Lokasi | Kenapa dipilih | Yang ditolak |
|---|---|---|---|---|
| Database inti kependudukan | IaaS | Private Cloud | Data NIK, alamat, dan biometrik sangat rahasia. Enkripsi dan pengelolaan kunci harus tetap berada dalam kendali negara. | PaaS/SaaS: sebagian kontrol pengelolaan diserahkan kepada penyedia layanan. |
| API verifikasi instansi/KYC | PaaS | Private Cloud | Tim pengembang dapat fokus pada kode dan logika aplikasi, sedangkan server dan runtime dikelola platform. | IaaS: membutuhkan pengelolaan server sendiri. SaaS: tidak tersedia layanan jadi yang sesuai dengan kebutuhan API. |
| Portal publik cek status | PaaS | Public Cloud | Lebih mudah ditingkatkan kapasitasnya ketika terjadi lonjakan akses, seperti saat pemilu dan pendaftaran sekolah. | IaaS: pengelolaan dan peningkatan kapasitas lebih rumit. SaaS: sulit disesuaikan dengan kebutuhan aplikasi. |
| Email, notifikasi, helpdesk, WAF/CDN | SaaS | Public Cloud | Termasuk layanan pendukung sehingga lebih praktis menggunakan layanan yang sudah tersedia. | IaaS/PaaS: membutuhkan pengelolaan dan pengembangan tambahan. |
| Backup dan Disaster Recovery | IaaS | Private Cloud (site DR milik negara) | Salinan data kependudukan harus tetap berada dalam kendali negara. | SaaS backup: lokasi dan pengelolaan salinan data bergantung pada penyedia layanan. |

Beberapa komponen sempat kami pertimbangkan lebih lanjut:
- **Database inti.** Kami sempat mempertimbangkan managed database (PaaS) karena menyediakan backup dan pembaruan otomatis. Namun, data biometrik sangat sensitif dan membutuhkan pengendalian ketat terhadap enkripsi serta kunci. Oleh karena itu, kami memilih IaaS meskipun pengelolaannya lebih kompleks.
- **API verifikasi.** Penempatan API di Public Cloud sebenarnya dapat mempermudah peningkatan kapasitas. Namun, API ini berhubungan langsung dengan database kependudukan sehingga kami memilih menempatkannya di Private Cloud untuk menjaga keamanan akses data.
- **Portal publik.** SaaS menjadi pilihan yang lebih praktis, tetapi portal membutuhkan penyesuaian khusus, seperti masking data dan cache dengan TTL pendek. Karena itu, kami memilih PaaS agar pengembangan dan pengaturan aplikasi lebih fleksibel.

**Keputusan akhir:** Kami menggunakan kombinasi IaaS, PaaS, dan SaaS karena setiap komponen memiliki kebutuhan yang berbeda. IaaS digunakan untuk komponen yang membutuhkan kontrol tinggi, PaaS untuk pengembangan aplikasi, dan SaaS untuk layanan pendukung.

## Log Penggunaan AI (Level 2)

> Wajib diisi sesuai kebijakan Level 2 di [`../RUBRIK-UMUM.md`](../RUBRIK-UMUM.md). Tulis "Tidak memakai AI" pada baris pertama jika memang tidak dipakai. Hanya untuk brainstorming ide/outline — bukan untuk kode/analisis/teks akhir.

| Tanggal | Tool AI | Prompt yang diberikan | Ringkasan saran/ide AI | Bagaimana diolah jadi tulisan/kode sendiri |
|---|---|---|---|---|
| 19 November 2026 | Gemini | Jelaskan Cloud Deployment Model (public, private, community, dan hybrid) beserta kelebihan dan kekurangannya pada masing-masing model. | Memetakan definisi empat model cloud deployment, poin kekurangan dan kelebihan, serta tabel perbandingan masing-masing model. | Memahami karakteristik tiap model, menyaring opsi yang relevan dengan studi kasus, dan memahami poin kelebihan dan kekurangan masing-masing model cloud deployment sebagai kerangka awal menyusun argumen. |
| 20 November 2026 | Claude | Jelaskan beda IaaS, PaaS, dan SaaS, dan contoh komponen sistem yang cocok untuk masing-masing. | AI menjelaskan pembagian tanggung jawab di tiap model: makin ke arah SaaS, makin sedikit yang harus dikelola sendiri, beserta contoh umumnya. | Kami cocokkan dengan materi kuliah, lalu jadikan patokan: makin rahasia datanya makin butuh kontrol penuh, makin umum fungsinya makin cocok pakai layanan jadi. |
| ... | ... | ... | ... | ... |
