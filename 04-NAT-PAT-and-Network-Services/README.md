# Lab 04: NAT, PAT & Core Network Services (DHCPv4, DHCP Relay, DNS)

## Opis projektu
Wdrożenie kluczowych usług infrastrukturalnych sieci korporacyjnej: automatycznej dystrybucji adresacji IP (DHCPv4) z wykorzystaniem agenta przekazywania między podsieciami (DHCP Relay), rozwiązywania nazw domenowych (serwer DNS dla hostów oraz lokalna tablica hostów i klient DNS na routerze brzegowym) oraz translacji adresów sieciowych (Static NAT i PAT/Overload) na styku z dostawcą internetu (ISP).

## Topologia sieci
* **Router brzegowy (Enterprise Edge):** 1x Cisco 2911 (`R1-EDGE`)
* **Router dostawcy internetu (ISP):** 1x Cisco 2911 (`ISP-ROUTER`)
* **Przełącznik dostępowy:** 1x Cisco 2960 (`SW-LAN`)
* **Serwery infrastrukturalne:**
    * `SRV-INFRA` (lokalny serwer DHCP, DNS oraz wewnętrzny HTTP w sieci serwerowej)
    * `SRV-WEB-PUBLIC` (publiczny serwer WWW po stronie ISP symulujący internet)
* **Hosty końcowe:** 2x stacje robocze PC w sieci klienckiej pobierające konfigurację automatycznie

## Tabela adresacji i usług sieciowych

| Urządzenie | Interfejs | Adres IP / Maska | Rola / Konfiguracja usług |
| :--- | :--- | :--- | :--- |
| **R1-EDGE** | `Gi0/0` (LAN Clients) | `192.168.10.1 /24` | Strefa wewnętrzna NAT, DHCP Relay -> `192.168.20.10` |
| **R1-EDGE** | `Gi0/1` (LAN Servers) | `192.168.20.1 /24` | Strefa wewnętrzna NAT (Brama dla serwerów lokalnych) |
| **R1-EDGE** | `Gi0/2` (WAN -> ISP) | `203.0.113.2 /30` | Strefa zewnętrzna NAT, Trasa domyślna -> `203.0.113.1` |
| **ISP-ROUTER** | `Gi0/0` (WAN -> Edge) | `203.0.113.1 /30` | Połączenie z siecią klienta |
| **ISP-ROUTER** | `Gi0/1` (Public Net) | `198.51.100.1 /24` | Brama dla publicznego serwera w internecie |
| **SRV-INFRA** | `NIC` | `192.168.20.10 /24` | Statyczne IP; Usługi: **DHCP Server**, **DNS Server**, **HTTP** |
| **SRV-WEB-PUBLIC** | `NIC` | `198.51.100.10 /24` | Statyczne IP; Usługa: **HTTP** (strona testowa `cisco.lab`) |
| **PC-1** | `NIC` | *DHCP (`192.168.10.0/24`)* | Klient DHCP (GW: `192.168.10.1`, DNS: `192.168.20.10`) |
| **PC-2** | `NIC` | *DHCP (`192.168.10.0/24`)* | Klient DHCP (GW: `192.168.10.1`, DNS: `192.168.20.10`) |

## Plan translacji NAT / PAT oraz DNS na `R1-EDGE`
* **Static NAT (1:1):** Przekierowanie publicznego adresu IP `203.0.113.10` na wewnętrzny serwer `192.168.20.10`.
* **PAT / NAT Overload (Many:1):** Translacja całej sieci klienckiej `192.168.10.0/24` (dopasowanej przez Standard ACL `10`) na adres wyjściowy interfejsu `GigabitEthernet0/2`.
* **Lokalne tłumaczenie nazw (Router DNS):** Statyczne wpisy hostów na `R1-EDGE` dla `cisco.lab` (`198.51.100.10`) oraz `intranet.local` (`192.168.20.10`), wsparte wskazaniem serwera DNS `192.168.20.10`.

## Zaimplementowane technologie
1. **DHCPv4 & DHCP Relay:** Centralizacja serwera DHCP w odseparowanej podsieci serwerowej (`192.168.20.0/24`) i przekazywanie żądań rozgłoszeniowych klientów z sieci `192.168.10.0/24` przez router `R1-EDGE`.
2. **DNS (Serwer + Router Host Table):** Konfiguracja rekordów `A` na serwerze `SRV-INFRA` dla stacji roboczych oraz lokalnej tablicy hostów i klienta DNS na routerze `R1-EDGE`.
3. **Static NAT & PAT (Overload):** Konfiguracja stref `inside`/`outside`, statycznego mapowania 1:1 oraz dynamicznej translacji portów (PAT) opartej na liście kontroli dostępu (ACL).

## Weryfikacja poprawności działania
* **Pobranie adresacji DHCP:** Poprawne przydzielenie IP z puli `192.168.10.50–100`, bramy domyślnej oraz serwera DNS na stacjach `PC-1` i `PC-2`.
* **Tłumaczenie nazw na routerze:** Poprawne rozwiązanie nazwy `cisco.lab` bezpośrednio z CLI routera `R1-EDGE`.
* **Tablica translacji NAT:** Widoczne wpisy statyczne oraz dynamiczne translacje portów TCP/ICMP dla klientów na `R1-EDGE`.
* **Test HTTP i DNS End-to-End:** Otwarcie w przeglądarce stacji `PC-1` stron `http://intranet.local` oraz `http://cisco.lab`.

## Pliki w repozytorium
* `lab04-topology.pkt` – plik symulacji Cisco Packet Tracer
* `R1-EDGE.txt` – zrzut konfiguracji routera brzegowego (`show running-config`)
* `ISP-ROUTER.txt` – zrzut konfiguracji routera dostawcy (`show running-config`)