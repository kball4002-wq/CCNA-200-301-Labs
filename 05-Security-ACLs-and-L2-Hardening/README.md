# Lab 05: Network Security – ACLs & Layer 2 Hardening

## Opis projektu
Wdrożenie wielowarstwowych mechanizmów bezpieczeństwa w sieci przedsiębiorstwa: kontrola dostępu do zdalnego zarządzania urządzeniami (SSHv2 + Standard ACL), segmentacja ruchu między podsieciami za pomocą rozszerzonych list kontroli dostępu (Extended ACL) oraz zabezpieczenie warstwy dostępowej przełącznika przed nieautoryzowanym podłączeniem urządzeń i atakami w warstwie 2 (Port Security, DHCP Snooping, Dynamic ARP Inspection).

## Topologia sieci
* **Router centralny:** 1x Cisco 2911 (`R1-SEC`)
* **Przełącznik dostępowy:** 1x Cisco 2960 (`SW-ACCESS`)
* **Serwer infrastrukturalny:** 1x Server (`SRV-ADMIN` – dostępny tylko dla IT)
* **Hosty końcowe:**
    * `PC-IT` (stacja administratora)
    * `PC-SALES` (stacja działu sprzedaży)
    * `PC-ROGUE` (nieautoryzowany laptop do testów naruszeń Port Security)

## Tabela adresacji i przypisania VLAN

| Urządzenie | Interfejs | Adres IP / Maska | VLAN / Rola |
| :--- | :--- | :--- | :--- |
| **R1-SEC** | `Gi0/0.10` (Subinterface) | `192.168.10.1 /24` | VLAN 10 (IT & Management) |
| **R1-SEC** | `Gi0/0.20` (Subinterface) | `192.168.20.1 /24` | VLAN 20 (Sales) |
| **R1-SEC** | `Gi0/1` (Server LAN) | `10.10.10.1 /24` | Sieć serwerowa (strefa chroniona) |
| **SW-ACCESS** | `SVI Vlan 10` | `192.168.10.2 /24` | Zarządzanie przełącznikiem (GW: `192.168.10.1`) |
| **SW-ACCESS** | `Gi0/1` | *Trunk (VLAN 10, 20)* | Uplink do `R1-SEC` (`ip dhcp snooping trust`, `ip arp inspection trust`) |
| **SW-ACCESS** | `Fa0/1` | *Access VLAN 10* | Port podpięty do `PC-IT` (Port Security: max 1 MAC, Sticky) |
| **SW-ACCESS** | `Fa0/2` | *Access VLAN 20* | Port podpięty do `PC-SALES` (Port Security: max 1 MAC, Sticky) |
| **SRV-ADMIN** | `NIC` | `10.10.10.10 /24` | Serwer wewnętrzny (HTTP + ICMP, GW: `10.10.10.1`) |
| **PC-IT** | `NIC` | `192.168.10.10 /24` | Stacja administratora (GW: `192.168.10.1`) |
| **PC-SALES** | `NIC` | `192.168.20.10 /24` | Stacja użytkownika (GW: `192.168.20.1`) |

## Polityka bezpieczeństwa (Reguły ACL)
* **Ochrona dostępu VTY (Standard ACL `ADMIN_SSH`):** Dostęp przez SSH do `R1-SEC` oraz `SW-ACCESS` dozwolony wyłącznie z sieci `192.168.10.0/24` (VLAN 10).
* **Segmentacja ruchu (Extended ACL `SALES-POLICY`):**
    1. Dział Sprzedaży (`192.168.20.0/24`) **może** pingować serwer `10.10.10.10` (`ICMP`).
    2. Dział Sprzedaży **nie może** otwierać zasobów WWW (`TCP 80`, `TCP 443`) na serwerze `10.10.10.10`.
    3. Dział Sprzedaży **nie może** inicjować żadnego ruchu do sieci administratorów (`192.168.10.0/24`).
    4. Cały pozostały ruch z działu Sprzedaży jest dozwolony.

## Zaimplementowane technologie
1. **Hardening dostępu zdalnego (SSHv2 + Standard ACL):** Wyłączenie nieszyfrowanego protokołu Telnet (`transport input ssh`), wymuszenie kluczy RSA 1024-bit oraz ograniczenie logowania na liniach VTY za pomocą `access-class`.
2. **Extended ACL (Filtrowanie w warstwie 3 i 4):** Granularna kontrola dostępu na podinterfejsie routera (`Gi0/0.20 in`) z zachowaniem właściwej kolejności reguł od szczegółowych do ogólnych.
3. **Port Security:** Ograniczenie liczby adresów MAC na portach dostępowych (`maximum 1`), dynamiczne uczenie się adresu z zapisem w konfiguracji (`mac-address sticky`) oraz blokada portu (`shutdown`) w przypadku podłączenia obcego urządzenia.
4. **DHCP Snooping & Dynamic ARP Inspection (DAI):** Ochrona przed fałszywymi serwerami DHCP oraz zatruwaniem tablic ARP (ARP Spoofing) poprzez oznaczenie portu uplink jako zaufanego (`trust`).

## Weryfikacja poprawności działania
* **Test ACL i SSH:** Próba nawiązania połączenia SSH z `PC-SALES` do `192.168.10.1` oraz otwarcia `http://10.10.10.10` zostaje odrzucona; te same operacje z `PC-IT` kończą się sukcesem.
* **Test Port Security:** Podłączenie `PC-ROGUE` do portu `Fa0/2` przełącza interfejs w stan `err-disabled` (`show port-security interface Fa0/2`).
* **Stan DHCP Snooping i DAI:** `show ip dhcp snooping` oraz `show ip arp inspection` potwierdzają aktywną ochronę dla VLAN 10 i 20 oraz zaufany status interfejsu `Gi0/1`.

## Pliki w repozytorium
* `lab05-topology.pkt` – plik symulacji Cisco Packet Tracer
* `R1-SEC.txt` – zrzut konfiguracji routera centralnego (`show running-config`)
* `SW-ACCESS.txt` – zrzut konfiguracji przełącznika dostępowego (`show running-config`)