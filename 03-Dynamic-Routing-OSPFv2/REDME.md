# Lab 03: Dynamic Routing – Single-Area OSPFv2 & Security

## Opis projektu
Wdrożenie protokołu routingu dynamicznego OSPFv2 (Open Shortest Path First) w pojedynczym obszarze szkieletowym (`Area 0`) dla sieci wielooddziałowej połączonej w topologii pełnego trójkąta (Full Mesh). Projekt obejmuje optymalizację konwergencji poprzez wymuszenie trybu Point-to-Point na łączach międzyrouterowych, zabezpieczenie interfejsów lokalnych (Passive Interfaces), kryptograficzne uwierzytelnianie sąsiedztwa algorytmem MD5 oraz dynamiczną propagację trasy domyślnej do internetu.

## Topologia sieci
* **Routery brzegowe i oddziałowe:** 3x Cisco 2911 (`R1-HQ`, `R2-BRANCH-1`, `R3-BRANCH-2`)
* **Przełączniki dostępowe:** 3x Cisco 2960 (`SW-HQ`, `SW-B1`, `SW-B2`)
* **Symulacja ISP / Internet:** 1x Interfejs `Loopback0` na routerze `R1-HQ`
* **Hosty końcowe:** 3x stacje robocze PC przypisane do poszczególnych oddziałów

## Tabela adresacji i parametrów OSPF

| Urządzenie | Interfejs | Adres IP / Maska | Obszar OSPF | Typ sieci / Rola |
| :--- | :--- | :--- | :--- | :--- |
| **R1-HQ** | `Loopback0` (ISP) | `8.8.8.8 /32` | - | Trasa statyczna domyślna (`0.0.0.0/0`) |
| **R1-HQ** | `Gi0/0` (LAN HQ) | `192.168.10.1 /24` | Area 0 | Passive Interface (Brama dla PC-HQ) |
| **R1-HQ** | `Gi0/1` (-> R2) | `10.0.12.1 /30` | Area 0 | Point-to-Point, MD5 Auth |
| **R1-HQ** | `Gi0/2` (-> R3) | `10.0.13.1 /30` | Area 0 | Point-to-Point, MD5 Auth |
| **R2-BRANCH-1** | `Gi0/0` (LAN B1) | `192.168.20.1 /24` | Area 0 | Passive Interface (Brama dla PC-B1) |
| **R2-BRANCH-1** | `Gi0/1` (-> R1) | `10.0.12.2 /30` | Area 0 | Point-to-Point, MD5 Auth |
| **R2-BRANCH-1** | `Gi0/2` (-> R3) | `10.0.23.1 /30` | Area 0 | Point-to-Point, MD5 Auth |
| **R3-BRANCH-2** | `Gi0/0` (LAN B2) | `192.168.30.1 /24` | Area 0 | Passive Interface (Brama dla PC-B2) |
| **R3-BRANCH-2** | `Gi0/1` (-> R1) | `10.0.13.2 /30` | Area 0 | Point-to-Point, MD5 Auth |
| **R3-BRANCH-2** | `Gi0/2` (-> R2) | `10.0.23.2 /30` | Area 0 | Point-to-Point, MD5 Auth |
| **PC-HQ** | `NIC` | `192.168.10.10 /24` | - | Host w sieci centrali (GW: `192.168.10.1`) |
| **PC-B1** | `NIC` | `192.168.20.10 /24` | - | Host w oddziale 1 (GW: `192.168.20.1`) |
| **PC-B2** | `NIC` | `192.168.30.10 /24` | - | Host w oddziale 2 (GW: `192.168.30.1`) |

## Zaimplementowane technologie
1. **Single-Area OSPFv2 (Process ID 1):** Dynamiczna wymiana tablic routingu w obszarze `Area 0` z jawnym zdefiniowaniem identyfikatorów Router-ID (`1.1.1.1`, `2.2.2.2`, `3.3.3.3`).
2. **Optymalizacja konwergencji (Point-to-Point):** Wymuszenie typu sieci `point-to-point` na łączach `/30`, eliminujące proces elekcji DR/BDR i skracające czas nawiązania pełnego sąsiedztwa.
3. **Hardening & MD5 Authentication:** Blokada wysyłania pakietów OSPF Hello w stronę sieci LAN (`passive-interface`) oraz uwierzytelnianie sąsiedztwa kluczem MD5 na wszystkich łączach międzyrouterowych.
4. **Default Route Propagation:** Konfiguracja trasy domyślnej na routerze brzegowym `R1-HQ` w stronę internetu (`Loopback0`) i jej automatyczne rozgłoszenie w domenie OSPF (`default-information originate`).

## Weryfikacja poprawności działania
* **Sąsiedztwo OSPF:** `show ip ospf neighbor` – wszystkie relacje w stanie `FULL/-` (brak wyboru DR/BDR).
* **Tablica routingu:** `show ip route ospf` – obecność tras z kodem `O` do wszystkich sieci oddziałowych oraz trasy zewnętrznej `O*E2 0.0.0.0/0` na routerach `R2` i `R3`.
* **Test łączności i redundancji:** Pomyślny `ping 8.8.8.8` ze stacji roboczych `PC-B1` i `PC-B2` oraz automatyczne przełączenie ruchu trasą zapasową przez drugi oddział po wyłączeniu łącza głównego do `R1-HQ`.
