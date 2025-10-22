test tugas
graph TD
    A["🎯 SATU DATA INDONESIA (SDI)"] --> B1["🔹 Prinsip Data"]
    A --> B2["🔹 Komponen Utama"]
    A --> B3["🔹 Tujuan"]

    B1 --> C1["➡️ Satu Standar Data"]
    B1 --> C2["➡️ Satu Metadata"]
    B1 --> C3["➡️ Satu Referensi Data"]
    B1 --> C4["➡️ Interoperabilitas Data"]

    B2 --> D1["📊 Walidata (BPS/Perangkat Daerah)"]
    B2 --> D2["🗃️ Produsen Data"]
    B2 --> D3["📥 Pembina Data"]
    B2 --> D4["🌍 Portal Satu Data"]

    B3 --> E1["📈 Integrasi Data Pembangunan"]
    B3 --> E2["💡 Pengambilan Keputusan Berbasis Data"]
    B3 --> E3["📚 Keterbukaan & Transparansi Informasi"]

    %% EPSS relationship
    F["🧩 EPSS (Evaluasi Penyelenggaraan Statistik Sektoral)"] --> G1["📑 Penilaian Kualitas Penyelenggaraan Statistik"]
    F --> G2["🧠 Penguatan Tata Kelola Data Sektoral"]
    F --> G3["🔍 Monitoring dan Evaluasi Pelaksanaan SDI di Daerah"]

    %% Relational links
    G1 --> C1
    G1 --> C2
    G1 --> C3
    G2 --> D1
    G2 --> D2
    G3 --> E1
    G3 --> E2

    %% Output Integration
    F --> H["🖥️ Rencana Aksi Satu Data Kabupaten/Kota"]
    H --> I["📂 Dokumen: Pedoman Teknis, SOP, dan Portal Data"]

    style A fill:#2b6cb0,stroke:#fff,color:#fff
    style F fill:#38a169,stroke:#fff,color:#fff
    style H fill:#f6ad55,stroke:#fff,color:#000
