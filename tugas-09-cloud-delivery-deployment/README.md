# Tugas 9 (Pekan 9) — Cloud Delivery & Deployment Model

**Materi terkait:** Karakteristik esensial cloud computing (NIST SP 800-145), Deployment Model (Public/Private/Hybrid/Community), Service Model (IaaS/PaaS/SaaS).

> Bagian 2 mata kuliah ini murni **analisis & desain di atas kertas** — tidak ada biaya cloud sama sekali, karena tugas berhenti di tahap rekomendasi, bukan implementasi nyata (implementasi baru mulai Tugas 11).

## Studi Kasus

Pemerintah ingin membangun **Sistem Data Kependudukan** (nama, NIK, alamat, data biometrik) yang:
- Sangat **rahasia** (data pribadi warga negara, diatur UU Perlindungan Data Pribadi).
- Diakses oleh **ribuan instansi** (dukcapil daerah, kepolisian, bank untuk verifikasi KYC) dengan pola akses yang **tidak rata** (lonjakan saat musim pemilu/pendaftaran sekolah).
- Butuh **skalabilitas** agar tidak downtime saat lonjakan akses, tapi **tidak boleh** datanya berada di infrastruktur yang tidak terkendali penuh oleh negara.

## Tugas Kelompok

1. **Rekomendasikan Deployment Model** (Public / Private / Hybrid / Community Cloud) yang paling tepat untuk sistem ini. Jelaskan alasannya berdasarkan **karakteristik NIST** (on-demand self-service, broad network access, resource pooling, rapid elasticity, measured service) dikaitkan dengan kebutuhan kerahasiaan vs skalabilitas.
2. **Rekomendasikan Service Model** (IaaS / PaaS / SaaS) yang tepat untuk komponen-komponen berbeda dari sistem ini (boleh berbeda service model untuk komponen berbeda, mis. database inti vs portal publik untuk pengecekan status).
3. Jelaskan **minimal 2 risiko spesifik** dari pilihan kalian dan mitigasinya (mis. jika Hybrid Cloud dipilih: risiko sinkronisasi data antara private & public segment).
4. Bandingkan singkat dengan **satu alternatif yang kalian tolak** — kenapa alternatif itu kurang tepat (mis. kenapa Public Cloud murni ditolak, atau kenapa Private Cloud murni dianggap kurang skalabel).

## Jawaban

### 1. Rekomendasi Deployment Model: Hybrid Cloud

Menurut NIST SP 800-145, **Hybrid Cloud** adalah gabungan dua atau lebih infrastruktur cloud yang berbeda (private, community, atau public) yang tetap berdiri sebagai entitas terpisah, tetapi dihubungkan oleh teknologi standar atau proprietary yang memungkinkan portabilitas data dan aplikasi.

Data inti (NIK, biometrik, alamat) disimpan di **Private Cloud** milik negara. Komponen yang tidak memuat data sensitif (portal publik, cek status) berjalan di **Public Cloud** yang bisa dibesarkan saat lonjakan. Keduanya dihubungkan lewat jalur privat terenkripsi.

| Karakteristik NIST | Kerahasiaan | Skalabilitas | Penerapan di Hybrid |
|---|---|---|---|
| On-demand self-service | Provisioning di private cloud diatur kebijakan ketat | Tim instansi bisa menambah resource tanpa tiket manual | Portal self-service dengan akses berbasis peran |
| Broad network access | Akses data inti dibatasi lewat jaringan pemerintah (VPN/jaringan khusus) | Ribuan instansi bisa mengakses dari berbagai perangkat | API gateway di public segment untuk layanan publik, jalur privat untuk data sensitif |
| Resource pooling | Pool private cloud tidak dibagi dengan tenant lain | Pool public cloud dipakai bersama untuk komponen non-sensitif | Isolasi untuk data inti, multi-tenant hanya untuk front-end |
| Rapid elasticity | Private cloud sulit di-scale cepat karena hardware terbatas | Public cloud bisa scale out otomatis saat pemilu/pendaftaran sekolah | Cloud bursting khusus beban non-sensitif (portal dan baca data minimal); data inti tidak pernah diproses di public cloud |
| Measured service | Audit log akses data inti diperlukan untuk akuntabilitas dan kepatuhan UU PDP (pencatatan pemrosesan dan pengamanan data) | Pemakaian terukur untuk perencanaan kapasitas | Metering di kedua segmen, dipantau terpusat |

Kebutuhan kerahasiaan dan skalabilitas saling bertabrakan. Hybrid memisahkan keduanya: kerahasiaan dijaga di private cloud, skalabilitas diambil dari public cloud, sehingga negara tetap mengendalikan penuh data sensitif.

### 2. Rekomendasi Service Model per Komponen

| Komponen | Service Model | Lokasi | Alasan |
|---|---|---|---|
| Database inti (NIK, biometrik) | IaaS | Private cloud | Butuh kontrol penuh atas OS, enkripsi, dan manajemen kunci |
| API verifikasi instansi/KYC | PaaS | Private cloud | Tim fokus ke logika aplikasi dan keamanan API, runtime dikelola platform |
| Portal publik cek status | PaaS | Public cloud | Tidak memuat data sensitif lengkap, butuh elastis saat lonjakan |
| Email/notifikasi, helpdesk, WAF/CDN | SaaS | Public cloud | Bukan fungsi inti, tidak menyimpan data kependudukan |
| Backup & disaster recovery | IaaS | Private (site DR milik negara) | Salinan harus tetap di bawah kendali negara dan terenkripsi |

Prinsipnya: semakin sensitif datanya, semakin rendah level abstraksi (kontrol maksimal). Semakin umum fungsinya, semakin tinggi level abstraksi (efisiensi maksimal).

### 3. Risiko Spesifik dan Mitigasi

**Risiko 1: Kebocoran data di titik sambung private-public.**
Jika portal publik diretas, penyerang bisa mencoba masuk ke data inti.
Mitigasi:
- Portal publik tidak menyimpan data sensitif lengkap, hanya hasil minimal (mis. status "terverifikasi") atau data yang di-tokenisasi/masking.
- Koneksi lewat IPsec VPN/dedicated link dengan mutual TLS, ditambah API gateway dan WAF di perbatasan.
- Prinsip least privilege, segmentasi jaringan (zero trust), dan audit log akses.

**Risiko 2: Inkonsistensi data antara private dan public segment.**
Cache/replika di public cloud bisa basi, mis. status penduduk sudah berubah tapi portal masih menampilkan data lama. Yang di-cache di public segment hanya data minimal atau data yang sudah di-masking/tokenisasi (bukan data sensitif lengkap), sehingga konsisten dengan prinsip pemisahan di Bagian 1.
Mitigasi:
- Single source of truth di private cloud, public hanya menyimpan cache data minimal dengan TTL pendek.
- Sinkronisasi berbasis event/message queue dengan retry dan invalidasi cache.
- Rekonsiliasi data secara berkala.

**Risiko 3: Ketergantungan pada penyedia public cloud dan kedaulatan data.**
Gangguan atau perubahan kebijakan penyedia bisa mengganggu layanan publik, dan pemrosesan di luar negeri bisa melanggar aturan lokasi data. Menurut PP No. 71 Tahun 2019 tentang Penyelenggaraan Sistem dan Transaksi Elektronik, penyelenggara sistem elektronik lingkup publik (termasuk instansi pemerintah) pada prinsipnya wajib mengelola, memproses, dan/atau menyimpan sistem dan data elektroniknya di wilayah Indonesia.
Mitigasi:
- Pilih penyedia dengan region di Indonesia dan patuh UU PDP serta PP 71/2019.
- Pakai teknologi portabel (container/Kubernetes, standar terbuka) agar mudah dipindah.
- Siapkan failover ke private cloud untuk fungsi esensial (mode terbatas).

### 4. Perbandingan dengan Alternatif yang Ditolak: Public Cloud Murni

| Aspek | Public Cloud Murni | Hybrid Cloud (dipilih) |
|---|---|---|
| Kontrol data | Data di infrastruktur penyedia, negara tidak punya kendali penuh | Data inti tetap di infrastruktur negara |
| Kerahasiaan | Resource pooling multi-tenant meningkatkan risiko akses tidak sah | Data sensitif terisolasi di private cloud |
| Kepatuhan UU PDP | Sulit menjamin lokasi dan akses data | Mudah diaudit |
| Elastisitas | Sangat baik | Baik (untuk komponen non-sensitif) |
| Biaya | Rendah di awal, risiko hukum/reputasi tinggi | Investasi awal lebih besar, risiko lebih terkendali |

Public Cloud murni unggul di rapid elasticity dan on-demand self-service, tetapi melanggar syarat utama studi kasus: data tidak boleh berada di infrastruktur yang tidak dikendalikan penuh oleh negara. Kebocoran data biometrik berdampak permanen karena tidak bisa "diganti" seperti password, sehingga keunggulan skalabilitas tidak sebanding dengan risikonya.
## Struktur Submission

```
tugas-09-cloud-delivery-deployment/
├── README.md      # Analisis lengkap (isi dengan poin 1-4 di atas)
├── JURNAL.md
└── bukti/          # (opsional) diagram pendukung
```

## Rubrik Penilaian (Tugas 9)

| Komponen | Bobot | Kriteria |
|---|---|---|
| Ketepatan rekomendasi deployment model | 30% | Argumen berbasis karakteristik NIST, bukan sekadar "private cloud lebih aman" tanpa detail |
| Ketepatan rekomendasi service model | 25% | Pemetaan komponen sistem ke IaaS/PaaS/SaaS masuk akal |
| Analisis risiko & mitigasi | 25% | Risiko spesifik ke konteks data kependudukan (bukan risiko generik) |
| Perbandingan dengan alternatif yang ditolak | 20% | Argumen penolakan jelas dan berdasar |

## Batasan Penggunaan AI (Level 2)

Kebijakan **Level 2 (AI Assisted Idea Generation & Structuring)** berlaku — lihat [`../RUBRIK-UMUM.md`](../RUBRIK-UMUM.md). Boleh bertanya ke AI gambaran umum karakteristik NIST/deployment model untuk brainstorming; **tidak boleh** meminta AI menuliskan rekomendasi/analisis akhir yang tinggal ditempel ke `README.md`. Catat pemakaian AI di "Log Penggunaan AI" pada `JURNAL.md`.

- Jawaban yang terasa seperti "template" (bisa dipakai untuk sistem cloud apa saja tanpa menyebut spesifik kependudukan/data pribadi/regulasi) akan dinilai rendah pada komponen ketepatan rekomendasi.
- `README.md` wajib membahas minimal satu skenario tandingan yang kalian buat sendiri (mis. "bagaimana jika instansi daerah tidak punya infrastruktur private cloud sendiri?") beserta jawabannya.
