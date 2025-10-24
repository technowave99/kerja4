```markdown
# UTS 2025 – Topologi Jaringan Tiga-Layer

Diagram ini menggambarkan rancangan **topologi jaringan tiga-layer (Core–Distribution–Access)** sesuai soal UTS, meliputi VLAN, QoS/VoIP, Dual ISP, Wireless Controller, serta server dan SAN.

```mermaid
flowchart TB
  %% ================= CORE =================
  subgraph CORE["Core Layer"]
    direction TB
    FW["Edge Router/Firewall\n(Dual WAN, NAT, ACL, QoS\nISP1 1Gbps, ISP2 500Mbps)"]
    C1["Core-1 Switch\n(40 Gbps backplane)"]
    C2["Core-2 Switch\n(40 Gbps backplane)"]
    FW -- LAG/Redundant --> C1
    FW -- LAG/Redundant --> C2
    C1 <-. MLAG/VPC .-> C2
  end

  %% ================= ISPs =================
  ISP1(("ISP1\n1 Gbps Fiber"))
  ISP2(("ISP2\n500 Mbps Fiber"))
  ISP1 --- FW
  ISP2 --- FW

  %% ================= DISTRIBUTION =================
  subgraph DIST["Distribution Layer"]
    direction TB
    subgraph D_G["Ground Floor (Sales/Admin)"]
      direction LR
      DG1["Dist-GF-1\n(10 G uplinks)"]
      DG2["Dist-GF-2\n(10 G uplinks)"]
      DG1 <-. Stack/MLAG .-> DG2
    end
    subgraph D_F["First Floor (R&D Lab)"]
      direction LR
      DF1["Dist-1F-1\n(10 G uplinks)"]
      DF2["Dist-1F-2\n(10 G uplinks)"]
      DF1 <-. Stack/MLAG .-> DF2
    end
    subgraph D_SRV["Server Room (Central)"]
      direction LR
      DS1["ToR-SRV-1\n(10/25 G to servers)"]
      DS2["ToR-SRV-2\n(10/25 G to servers)"]
      DS1 <-. MLAG .-> DS2
    end
  end

  %% Core to Distribution
  C1 == 2x10G == DG1
  C2 == 2x10G == DG2
  C1 == 2x10G == DF1
  C2 == 2x10G == DF2
  C1 == 2x25G == DS1
  C2 == 2x25G == DS2

  %% ================= ACCESS =================
  subgraph ACC["Access Layer"]
    direction TB
    subgraph A_G["Ground Floor Access (≈150 Users)"]
      direction LR
      AG1["Access-GF-1\n(48x1G PoE)"]
      AG2["Access-GF-2\n(48x1G PoE)"]
      AG3["Access-GF-3\n(48x1G PoE)"]
      AG4["Access-GF-4\n(48x1G PoE)"]
    end
    subgraph A_F["First Floor Access (≈150 Users)"]
      direction LR
      AF1["Access-1F-1\n(48x1G PoE)"]
      AF2["Access-1F-2\n(48x1G PoE)"]
      AF3["Access-1F-3\n(48x1G PoE)"]
      AF4["Access-1F-4\n(48x1G PoE)"]
    end
    subgraph WIFI["Wireless LAN"]
      direction TB
      WLC["Wireless LAN Controller"]
      APG["AP x8 (GF)\nWi-Fi 6 (802.11ax)"]
      APF["AP x7 (1F)\nWi-Fi 6 (802.11ax)"]
      WLC --- APG
      WLC --- APF
    end
  end

  DG1 == 2x10G == AG1
  DG2 == 2x10G == AG2
  DG1 == 2x10G == AG3
  DG2 == 2x10G == AG4

  DF1 == 2x10G == AF1
  DF2 == 2x10G == AF2
  DF1 == 2x10G == AF3
  DF2 == 2x10G == AF4

  DG1 --- WLC
  DF1 --- WLC

  %% ================= SERVERS =================
  subgraph SRV["Server Room (Central Facility)"]
    direction TB
    S1["App/Web Server 1"]
    S2["App/Web Server 2"]
    S3["DB Server"]
    S4["File/Collab Server"]
    S5["VoIP Call Manager"]
    SAN["Storage Area Network\n(≥10 Gbps Intra-Rack)"]
  end

  DS1 --- S1
  DS1 --- S2
  DS2 --- S3
  DS2 --- S4
  DS1 --- S5
  DS1 === SAN
  DS2 === SAN

```
```
