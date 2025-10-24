# UTS 2025 – Topologi Jaringan: Fisik vs Logis (VLAN)

Dokumen ini memecah rancangan jaringan menjadi **dua diagram** agar jelas antara struktur **fisik (3-layer)** dan **logis (segmentasi VLAN, kebijakan akses, QoS)**.  
Mermaid telah disederhanakan agar kompatibel dengan renderer GitHub (tanpa `<br/>`, tanpa `note`, tanpa `==` edge).

---

## 1) Diagram Topologi **Fisik** (Core–Distribution–Access)

```mermaid
flowchart TB
  %% ===== CORE =====
  subgraph CORE[Core Layer]
    direction TB
    FW[Edge Router/Firewall\nDual WAN, NAT, ACL, QoS]
    C1[Core-1 Switch\n40 Gbps backplane]
    C2[Core-2 Switch\n40 Gbps backplane]
    FW --- C1
    FW --- C2
    C1 -. Redundancy .- C2
  end

  %% ===== ISPs =====
  ISP1[(ISP1\n1 Gbps Fiber)]
  ISP2[(ISP2\n500 Mbps Fiber)]
  ISP1 --- FW
  ISP2 --- FW

  %% ===== DISTRIBUTION =====
  subgraph DIST[Distribution Layer]
    direction TB
    subgraph D_G[Ground Floor (Sales/Admin)]
      direction LR
      DG1[Dist-GF-1\n10G uplinks]
      DG2[Dist-GF-2\n10G uplinks]
      DG1 -. Stack/MLAG .- DG2
    end
    subgraph D_F[First Floor (R&D Lab)]
      direction LR
      DF1[Dist-1F-1\n10G uplinks]
      DF2[Dist-1F-2\n10G uplinks]
      DF1 -. Stack/MLAG .- DF2
    end
    subgraph D_SRV[Server Room (TOR)]
      direction LR
      DS1[ToR-SRV-1\n10/25G to servers]
      DS2[ToR-SRV-2\n10/25G to servers]
      DS1 -. MLAG .- DS2
    end
  end

  %% Core to Distribution (labeled links)
  C1 ---|2x10G| DG1
  C2 ---|2x10G| DG2
  C1 ---|2x10G| DF1
  C2 ---|2x10G| DF2
  C1 ---|2x25G| DS1
  C2 ---|2x25G| DS2

  %% ===== ACCESS =====
  subgraph ACC[Access Layer]
    direction TB
    subgraph A_G[Ground Floor Access (~150 Users)]
      direction LR
      AG1[Access-GF-1\n48x1G PoE]
      AG2[Access-GF-2\n48x1G PoE]
      AG3[Access-GF-3\n48x1G PoE]
      AG4[Access-GF-4\n48x1G PoE]
    end
    subgraph A_F[First Floor Access (~150 Users)]
      direction LR
      AF1[Access-1F-1\n48x1G PoE]
      AF2[Access-1F-2\n48x1G PoE]
      AF3[Access-1F-3\n48x1G PoE]
      AF4[Access-1F-4\n48x1G PoE]
    end
    subgraph WIFI[Wireless LAN]
      direction TB
      WLC[Wireless LAN Controller]
      APG[AP x8 (GF)\nWi-Fi 6 (802.11ax)]
      APF[AP x7 (1F)\nWi-Fi 6 (802.11ax)]
      WLC --- APG
      WLC --- APF
    end
  end

  %% Distribution to Access
  DG1 ---|2x10G| AG1
  DG2 ---|2x10G| AG2
  DG1 ---|2x10G| AG3
  DG2 ---|2x10G| AG4

  DF1 ---|2x10G| AF1
  DF2 ---|2x10G| AF2
  DF1 ---|2x10G| AF3
  DF2 ---|2x10G| AF4

  %% WLC uplinks
  DG1 --- WLC
  DF1 --- WLC

  %% ===== SERVERS & SAN =====
  subgraph SRV[Server Room (Central Facility)]
    direction TB
    S1[(App/Web Server 1)]
    S2[(App/Web Server 2)]
    S3[(DB Server)]
    S4[(File/Collab Server)]
    S5[(VoIP Call Manager)]
    SAN[(Storage Area Network)]
  end

  DS1 --- S1
  DS1 --- S2
  DS2 --- S3
  DS2 --- S4
  DS1 --- S5
  DS1 --- SAN
  DS2 --- SAN
```

**Keterangan Fisik Singkat**  
- Dual ISP ke firewall, uplink redundant ke Core-1 dan Core-2.  
- Redundansi core dan distribution via MLAG/stack.  
- Uplink antar layer dilabel kapasitas (2x10G, 2x25G).  
- Wi-Fi 6 total ~15 AP (8 di GF, 7 di 1F).  
- Server Room berisi App/Web, DB, File, VoIP, dan SAN.

---

## 2) Diagram Topologi **Logis (VLAN, Akses, QoS)**

```mermaid
flowchart TB
  %% VLAN domains
  subgraph V10[VLAN10 – Sales/Admin (GF)]
    SA1[User Sales/Admin\nAccess-GF]
  end

  subgraph V20[VLAN20 – R&D (1F)]
    RD1[User R&D\nAccess-1F]
  end

  subgraph V30[VLAN30 – VoIP]
    VOIP[IP Phones\nCall Manager]
  end

  subgraph V40[VLAN40 – Server/SAN]
    APP[App/Web]
    DB[Database]
    FILE[File/Collab]
    CM[VoIP Call Manager]
    STG[SAN]
  end

  subgraph V50[VLAN50 – Management/WiFi Infra]
    WLC[WLC]
    APs[AP Controller Plane]
    NMS[Network Mgmt/Monitoring]
  end

  %% Internet and Edge
  EDGE[Edge FW/Router\nNAT, ACL, QoS]
  INET[(Internet)]
  INET --- EDGE

  %% Inter-VLAN policies (logical)
  SA1 --- EDGE
  RD1 --- EDGE
  VOIP --- EDGE

  %% Access to servers (allowed via ACL)
  SA1 -. ACL allow .- APP
  SA1 -. ACL allow .- FILE
  RD1 -. ACL allow .- APP
  RD1 -. ACL allow .- DB
  RD1 -. ACL allow .- FILE

  %% VoIP priority path
  VOIP -. QoS EF .- CM
  VOIP -. QoS EF .- EDGE

  %% Management/Infra access
  WLC --- APs
  WLC -. manage SSID/VLAN mapping .- SA1
  WLC -. manage SSID/VLAN mapping .- RD1
  NMS -. mgmt only .- APP
  NMS -. mgmt only .- DB
  NMS -. mgmt only .- FILE
  NMS -. mgmt only .- EDGE

  %% Server internal links
  APP --- DB
  FILE --- STG
```

**Aturan Logis Singkat**  
- **Segmentasi**:  
  - VLAN10 Sales/Admin dan **VLAN20 R&D terisolasi** satu sama lain; komunikasi lintas VLAN hanya via **ACL di Core/Firewall**.  
  - **VLAN30 VoIP** diberi **QoS EF** menuju Call Manager dan Internet.  
  - **VLAN40 Server/SAN** diakses terbatas dari VLAN10/VLAN20 sesuai kebutuhan (APP/FILE untuk Sales, APP/DB/FILE untuk R&D).  
  - **VLAN50** untuk manajemen perangkat (WLC, APs, NMS).  
- **Internet**: Semua akses internet keluar melalui **Edge FW/Router** (NAT, ACL, QoS).  
- **Wi-Fi**: SSID dipetakan ke VLAN10/VLAN20/VLAN30 melalui WLC.  

---

### Catatan Implementasi
- Terapkan **ACL** yang eksplisit (deny by default) lalu allow khusus ke layanan yang diperlukan.  
- Terapkan **QoS** dengan kelas **EF** untuk VoIP; pastikan jitter rendah dan prioritas antrean.  
- Pastikan link kritikal menggunakan **LACP** untuk redundansi (di Core–Dist–Access dan ToR–Server).  
- Dokumentasikan **IP plan** per VLAN dan **gateway SVI** di Core/Firewall.
