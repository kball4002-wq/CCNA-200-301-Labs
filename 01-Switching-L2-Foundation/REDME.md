# Lab 01: Switching, VLANs, EtherChannel & STP Hardening

## Opis projektu
Wdrożenie bezpiecznej i redundantnej architektury przełączania (Warstwa 2) dla sieci kampusowej. Projekt obejmuje segmentację ruchu na sieci VLAN, agregację łącz w standardach LACP oraz PAgP, deterministyczny wybór mostu głównego w Rapid PVST+ oraz zabezpieczenie portów dostępowych i zarządzania (SSHv2).

## Topologia sieci
* **Urządzenia:** 3x Cisco Catalyst 2960 (`SW-CORE-1`, `SW-ACC-1`, `SW-ACC-2`), 4x PC
* **Agregacja łączy:**
    * `Po1` (`SW-CORE-1` <-> `SW-ACC-1`): Porty `Fa0/1 - 2` (Protokół **LACP**)
    * `Po2` (`SW-CORE-1` <-> `SW-ACC-2`): Porty `Fa0/3 - 4` (Protokół **PAgP**)
* **Łącze nadmiarowe:** `Gi0/1` (`SW-ACC-1` <-> `SW-ACC-2`)

## Tabela adresacji i VLAN

| VLAN ID | Nazwa VLAN | Podsieć | Urządzenie / Interfejs | Adres IP / Maska |
| :--- | :--- | :--- | :--- | :--- |
| **10** | `HR` | `10.10.10.0/24` | `PC-HR-1` (`Fa0/10`), `PC-HR-2` (`Fa0/10`) | `.11/24`, `.12/24` |
| **20** | `IT` | `10.10.20.0/24` | `PC-IT-1` (`Fa0/20`), `PC-IT-2` (`Fa0/20`) | `.11/24`, `.12/24` |
| **99** | `MGMT` | `10.10.99.0/24` | `SW-CORE-1` (SVI VLAN 99) | `10.10.99.1/24` |
| **99** | `MGMT` | `10.10.99.0/24` | `SW-ACC-1` (SVI VLAN 99) | `10.10.99.2/24` |
| **99** | `MGMT` | `10.10.99.0/24` | `SW-ACC-2` (SVI VLAN 99) | `10.10.99.3/24` |
| **88** | `BLACKHOLE` | *Brak* | Nieużywane porty fizyczne | *Administratively Down* |
| **999** | `NATIVE` | *Brak* | Wszystkie łącza Trunk | *Brak IP (Untagged)* |

## Zaimplementowane technologie
1. **Device Hardening & Management:** Konfiguracja SSHv2 (RSA 2048-bit), lokalne uwierzytelnianie (privilege 15), blokada Telnetu na liniach VTY, szyfrowanie haseł (`service password-encryption`, `enable secret`).
2. **VLAN & Trunking Security:** Wyłączenie negocjacji DTP (`nonegotiate`), zmiana domyślnego Native VLAN na `999`, przycięcie dozwolonych sieci VLAN na łączach trunk, izolacja nieużywanych portów w `VLAN 88`.
3. **EtherChannel:** Konfiguracja kanałów portów w trybach LACP (`active`/`passive`) oraz PAgP (`desirable`/`auto`).
4. **Spanning Tree Protocol (Rapid PVST+):** Wymuszenie roli Root Bridge Primary (`SW-CORE-1`) oraz Secondary (`SW-ACC-1`), aktywacja `PortFast` oraz `BPDU Guard` na portach krawędziowych.

## Pliki w repozytorium
* `lab01-topology.pkt` – plik symulacji Cisco Packet Tracer
* `SW-CORE-1.txt` – pełna konfiguracja przełącznika rdzeniowego
* `SW-ACC-1.txt` – pełna konfiguracja przełącznika dostępowego 1
* `SW-ACC-2.txt` – pełna konfiguracja przełącznika dostępowego 2