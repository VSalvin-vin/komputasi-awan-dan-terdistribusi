# Diagram Arsitektur Hybrid Cloud: Sistem Data Kependudukan

```mermaid
flowchart LR
    subgraph User["Pengguna"]
        W["Warga"]
        I["Instansi: Dukcapil, Polisi, Bank (KYC)"]
    end

    subgraph Public["Public Cloud (elastis, tanpa data sensitif)"]
        CDN["CDN + WAF (SaaS)"]
        P["Portal Cek Status (PaaS)"]
        C["Cache TTL pendek"]
        N["Email / Notifikasi (SaaS)"]
    end

    subgraph Private["Private Cloud Negara (data inti)"]
        GW["API Gateway Verifikasi / KYC (PaaS)"]
        DB[("Database Inti: NIK + Biometrik (IaaS)")]
        BK[("Backup dan Site DR (IaaS)")]
        LOG["Audit Log Akses"]
    end

    W --> CDN --> P --> C
    P --> N
    P -- "IPsec VPN + mutual TLS" --> GW
    I -- "Jaringan khusus pemerintah" --> GW
    GW --> DB
    DB --> BK
    GW --> LOG
```

**Keterangan:**
- Data sensitif hanya ada di Private Cloud. Public Cloud hanya menyimpan hasil minimal dan cache.
- Saat lonjakan (pemilu, pendaftaran sekolah), hanya segmen Public yang di-scale out (cloud bursting).
- Satu-satunya jalur antar segmen adalah VPN terenkripsi lewat API Gateway.
