# UTS 2025 – Topologi Jaringan Tiga-Layer

Diagram ini menggambarkan rancangan **topologi jaringan tiga-layer (Core–Distribution–Access)** sesuai kebutuhan UTS, meliputi VLAN, QoS/VoIP, Dual ISP, Wireless Controller, serta server dan SAN.  
Mermaid ini disederhanakan agar kompatibel dengan renderer GitHub (tanpa `<br/>`, tanpa `note`, tanpa `==` edge).

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
    subgraph D_SRV[Server Room (Central)]
      direction LR
      DS1[ToR-SRV-1\n10/25G to servers]
      DS2[ToR-SRV-2\n10/25G to servers]
      DS1 -. MLAG .- DS2
    end
  end

  %% Core to Distribution (labelled links)
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
      APG[AP x8 (GF)\nWi‑Fi 6 (802.11ax)]
      APF[AP x7 (1F)\nWi‑Fi 6 (802.11ax)]
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

## Segmentasi VLAN (Logis)
- **VLAN10**: Sales/Admin (GF)
- **VLAN20**: R&D (1F, terisolasi dari VLAN10 via ACL di Core/Firewall)
- **VLAN30**: VoIP (diprioritaskan QoS – EF)
- **VLAN40**: Server/SAN (akses terbatas dari VLAN10 & VLAN20)
- **VLAN50**: Management/Wi‑Fi Infra

## Catatan Desain
- **Dual ISP** via edge firewall/router (failover/load‑balance).
- **Redundansi** core & distribution menggunakan MLAG/stack.
- **Wi‑Fi 6** total ~15 AP (8 GF, 7 1F) untuk ±300 pengguna.
- **Uplink** antar layer diberi label kapasitas (contoh `|2x10G|`).
