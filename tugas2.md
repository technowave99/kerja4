
flowchart TB
  %% ================= CORE =================
  subgraph CORE[Core Layer]
    direction TB
    FW[Edge Router/Firewall<br/>(Dual WAN, NAT, ACL, QoS<br/>ISP1 1Gbps, ISP2 500Mbps)]
    C1[Core-1 Switch<br/>(40 Gbps backplane)]
    C2[Core-2 Switch<br/>(40 Gbps backplane)]
    FW -- LAG/Redundant --> C1
    FW -- LAG/Redundant --> C2
    C1 <-. MLAG/VPC .-> C2
  end

  %% ================= ISPs =================
  ISP1[(ISP1<br/>1 Gbps Fiber)]
  ISP2[(ISP2<br/>500 Mbps Fiber)]
  ISP1 --- FW
  ISP2 --- FW

  %% ================= DISTRIBUTION =================
  subgraph DIST[Distribution Layer]
    direction TB
    %% Ground Floor Distribution (Sales/Admin)
    subgraph D_G[Ground Floor (Sales/Admin)]
      direction LR
      DG1[Dist-GF-1<br/>(10 G uplinks)]
      DG2[Dist-GF-2<br/>(10 G uplinks)]
      DG1 <-. Stack/MLAG .-> DG2
    end

    %% First Floor Distribution (R&D Lab)
    subgraph D_F[First Floor (R&D Lab)]
      direction LR
      DF1[Dist-1F-1<br/>(10 G uplinks)]
      DF2[Dist-1F-2<br/>(10 G uplinks)]
      DF1 <-. Stack/MLAG .-> DF2
    end

    %% Server Room Distribution/TOR
    subgraph D_SRV[Server Room (Central)]
      direction LR
      DS1[ToR-SRV-1<br/>(10/25 G to servers)]
      DS2[ToR-SRV-2<br/>(10/25 G to servers)]
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
  subgraph ACC[Access Layer]
    direction TB

    %% Ground Floor Access (approx 150 users)
    subgraph A_G[Ground Floor Access (≈150 Users)]
      direction LR
      AG1[Access-GF-1<br/>(48x1G PoE)]
      AG2[Access-GF-2<br/>(48x1G PoE)]
      AG3[Access-GF-3<br/>(48x1G PoE)]
      AG4[Access-GF-4<br/>(48x1G PoE)]
    end

    %% First Floor Access (approx 150 users)
    subgraph A_F[First Floor Access (≈150 Users)]
      direction LR
      AF1[Access-1F-1<br/>(48x1G PoE)]
      AF2[Access-1F-2<br/>(48x1G PoE)]
      AF3[Access-1F-3<br/>(48x1G PoE)]
      AF4[Access-1F-4<br/>(48x1G PoE)]
    end

    %% Wireless
    subgraph WIFI[Wireless LAN]
      direction TB
      WLC[Wireless LAN Controller]
      APG[AP x8 (GF)<br/>Wi-Fi 6 (802.11ax)]
      APF[AP x7 (1F)<br/>Wi-Fi 6 (802.11ax)]
      WLC --- APG
      WLC --- APF
    end
  end

  %% Distribution to Access (redundant)
  DG1 == 2x10G == AG1
  DG2 == 2x10G == AG2
  DG1 == 2x10G == AG3
  DG2 == 2x10G == AG4

  DF1 == 2x10G == AF1
  DF2 == 2x10G == AF2
  DF1 == 2x10G == AF3
  DF2 == 2x10G == AF4

  %% WLC uplinks to both floors (for HA)
  DG1 --- WLC
  DF1 --- WLC

  %% ================= SERVERS & SAN =================
  subgraph SRV[Server Room (Central Facility)]
    direction TB
    S1[(App/Web Server 1)]
    S2[(App/Web Server 2)]
    S3[(DB Server)]
    S4[(File/Collab Server)]
    S5[(VoIP Call Manager)]
    SAN[(Storage Area Network<br/>(≥10 Gbps Intra-Rack))]
  end

  DS1 --- S1
  DS1 --- S2
  DS2 --- S3
  DS2 --- S4
  DS1 --- S5
  DS1 === SAN
  DS2 === SAN

  %% ================= LOGICAL SEGMENTATION (VLANs) =================
  classDef vlan10 fill:#e3f2fd,stroke:#1565c0,stroke-width:1px;
  classDef vlan20 fill:#fce4ec,stroke:#ad1457,stroke-width:1px;
  classDef vlan30 fill:#e8f5e9,stroke:#2e7d32,stroke-width:1px;
  classDef vlan40 fill:#fff3e0,stroke:#ef6c00,stroke-width:1px;
  classDef vlan50 fill:#ede7f6,stroke:#5e35b1,stroke-width:1px;

  %% Tagging nodes with VLAN roles
  class AG1,AG2,AG3,AG4 vlan10;        %% VLAN10 Sales/Admin (GF)
  class AF1,AF2,AF3,AF4 vlan20;        %% VLAN20 R&D (1F, isolated)
  class WLC,APG,APF vlan50;            %% VLAN50 Mgmt/WiFi-Infra
  class S5 vlan30;                     %% VLAN30 VoIP
  class S1,S2,S3,S4,SAN vlan40;        %% VLAN40 Servers/SAN

  %% Policy/Notes (annotations)
  note over DG1,DF2: Inter-VLAN via CORE firewalled policies\n- R&D (VLAN20) isolated from Sales/Admin (VLAN10)\n- Both can reach Servers (VLAN40) per ACLs\n- VoIP (VLAN30) prioritized (EF), jitter < 20ms
  note right of WLC: SSIDs mapped to VLAN10/VLAN20/VLAN30\nWi-Fi 6 coverage: ~15 AP total (1 per 20 users)
  note right of FW: Dual-WAN failover/load-balance\nEdge filtering + NAT + QoS
