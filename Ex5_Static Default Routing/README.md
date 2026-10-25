# Lab 5: Static and Default Routing

**File:** `Lab5_Static_Default_Routing.pkt`

## Aim
To configure static and default routing between two routers in Cisco Packet Tracer and verify connectivity between computers with `ping`.

## Topology
```
PC1 ─┐
     ├─ Switch0 ── R1 ══ Serial (DCE/DTE) ══ R2 ── PC3
PC2 ─┘
```

| Device | Interface | IP Address | Mask | Gateway |
|--------|-----------|------------|------|---------|
| PC1 | Fa0 | 192.168.1.10 | 255.255.255.0 | 192.168.1.1 |
| PC2 | Fa0 | 192.168.1.20 | 255.255.255.0 | 192.168.1.1 |
| PC3 | Fa0 | 192.168.2.10 | 255.255.255.0 | 192.168.2.1 |
| R1 | Gig0/0 | 192.168.1.1 | 255.255.255.0 | – |
| R1 | Serial0/0/0 (DCE) | 10.0.0.1 | 255.255.255.252 | – |
| R2 | Serial0/0/0 (DTE) | 10.0.0.2 | 255.255.255.252 | – |
| R2 | Gig0/0 | 192.168.2.1 | 255.255.255.0 | – |

Design notes:
- A switch (Switch0) joins PC1 and PC2 because R1 has one LAN port for that network.
- The 2911 router has no serial ports by default. I powered each router off, added an **HWIC-2T** module in the Physical tab, and powered it on.
- Cables: PC ↔ Switch and Switch ↔ Router are straight-through; PC3 ↔ R2 is crossover (PC ↔ Router); R1 ↔ R2 is a serial DCE cable.

## Steps followed
1. **Built the topology** with the devices and cables above, and added the HWIC-2T modules.
2. **Configured the PCs** with **Desktop → IP Configuration** using the table.
3. **Configured R1 and R2** in the CLI tab (full commands in [`configs/R1.txt`](configs/R1.txt) and [`configs/R2.txt`](configs/R2.txt)). `clock rate 64000` goes on the DCE side only.
4. **Checked interfaces:** `show ip interface brief` must show both interfaces `up/up`.
5. **Tested the serial link:** `ping 10.0.0.2` from R1 and `ping 10.0.0.1` from R2.
6. **Static routes:**
   ```
   R1(config)# ip route 192.168.2.0 255.255.255.0 10.0.0.2
   R2(config)# ip route 192.168.1.0 255.255.255.0 10.0.0.1
   ```
   `show ip route` shows `S 192.168.2.0/24 [1/0] via 10.0.0.2` on R1 and `S 192.168.1.0/24 [1/0] via 10.0.0.1` on R2.
7. **Tested PC to PC:** `ping 192.168.2.10` from PC1 and `ping 192.168.1.10` from PC3. (The first packet may time out while ARP resolves.)
8. **Default routes:**
   ```
   R1(config)# ip route 0.0.0.0 0.0.0.0 10.0.0.2
   R2(config)# ip route 0.0.0.0 0.0.0.0 10.0.0.1
   ```
   `show ip route` now also shows `S* 0.0.0.0/0`.
9. **Re-tested** with `ping` and `tracert 192.168.2.10` (expected hops: 192.168.1.1, 10.0.0.2, 192.168.2.10).
10. **Saved:** `copy running-config startup-config` on both routers.

Screenshots: `images/01_topology.png`, `images/02_ip_interface_brief.png`, `images/03_serial_ping.png`, `images/04_static_routes.png`, `images/05_default_routes.png`, `images/06_pc_ping.png`, `images/07_tracert.png`

## Troubleshooting
- `administratively down`: missing `no shutdown`
- Serial `up/down`: missing `clock rate` on the DCE side, or wrong IP/mask
- No serial interface: HWIC-2T not installed
- Cross-router ping fails: check next-hop addresses and PC default gateways

## Result
Static and default routing were configured between R1 and R2, and connectivity between 192.168.1.0/24 and 192.168.2.0/24 was verified with `ping` and `tracert`.
