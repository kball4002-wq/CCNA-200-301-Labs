# Lab 02: Inter-VLAN Routing (ROAS), Layer 3 SVI & First Hop Redundancy Protocol (HSRP)

## Opis projektu
Wdrożenie routingu pomiędzy segmentami sieci LAN (Inter-VLAN Routing) oraz zapewnienie wysokiej dostępności bramy domyślnej (FHRP) przy użyciu protokołu HSRPv2. Projekt obejmuje konfigurację subinterfejsów z enkapsulacją 802.1Q (Router-on-a-Stick) oraz mechanizmu przełączania awaryjnego (Failover & Preemption).

## Topologia sieci
* **Routery:** 2x Cisco 2911 (`R1-GW-ACT`, `R2-GW-STBY`)
* **Przełącznik L2:** 1x Cisco Catalyst 2960 (`SW-DIST-1`)
* **Hosty końcowe:** 2x PC (`PC-HR`, `PC-IT`)

## Tabela adresacji, VLAN i wirtualnych bram HSRP

| Urządzenie / Interfejs | VLAN | Adres fizyczny IP | Wirtualny IP HSRP (VIP) | Grupa HSRP | Priorytet HSRP | Rola HSRP |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **R1** `Gi0/0.10` | 10 (HR) | `10.10.10.2 /24` | `10.10.10.1` | Group 10 | 110 (Preempt) | Active |
| **R2** `Gi0/0.10` | 10 (HR) | `10.10.10.3 /24` | `10.10.10.1` | Group 10 | 100 (Default) | Standby |
| **R1** `Gi0/0.20` | 20 (IT) | `10.10.20.2 /24` | `10.10.20.1` | Group 20 | 100 (Default) | Standby |
| **R2** `Gi0/0.20` | 20 (IT) | `10.10.20.3 /24` | `10.10.20.1` | Group 20 | 110 (Preempt) | Active |
| **PC-HR** | 10 | `10.10.10.50 /24` | Brama: `10.10.10.1` | - | - | Host |
| **PC-IT** | 20 | `10.10.20.50 /24` | Brama: `10.10.20.1` | - | - | Host |

## Zaimplementowane technologie
1. **Router-on-a-Stick (802.1Q):** Podział interfejsów gigabitowych na subinterfejsy logiczne (`encapsulation dot1Q`).
2. **HSRPv2 Load Sharing:** Równomierne rozłożenie obciążenia pomiędzy routery (R1 jest bramą aktywną dla VLAN 10, R2 dla VLAN 20).
3. **Preemption & Tracking:** Wymuszenie powrotu roli Active po resecie routera (`standby preempt`).