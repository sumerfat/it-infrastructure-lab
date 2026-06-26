# Lab 2: Enterprise Network Topology (Cisco CCNA)

Dieses Labor demonstriert den Entwurf und die Implementierung eines sicheren, redundanten und skalierten Unternehmensnetzwerks für zwei Standorte (Essen und Dortmund). Es beinhaltet modernes Inter-VLAN-Routing (Router-on-a-Stick), dynamisches Routing via OSPF, eine simulierte ISP-Internet-Anbindung sowie erweiterte Sicherheitsrichtlinien (ACLs) zur Isolierung von Gast-Netzwerken.

---

## 🗺️ Netzwerk-Topologie & Design

Das Netzwerk ist in die Hauptzentrale (Essen) und eine remote Filiale (Dortmund) unterteilt. Um die Ausfallsicherheit und Strukturierung zu optimieren, werden die Standorte über ein Point-to-Point-Subnetz verbunden und die Zentrale in logische VLANs segmentiert.

### IP-Adressierung & CIDR-Bereiche

| Standort / Segment | VLAN | Netzadresse (CIDR) | Standard-Gateway | Zweck / Abteilung |
| :--- | :---: | :--- | :--- | :--- |
| **Essen (Zentrale)** | 10 | `192.168.10.0/24` | `192.168.10.1` | Interne Verwaltung / HR |
| **Essen (Zentrale)** | 20 | `192.168.20.0/24` | `192.168.20.1` | IT-Infrastruktur & Server |
| **Essen (Zentrale)** | 30 | `172.16.30.0/24` | `172.16.30.1` | Isoliertes Gäste-Netzwerk |
| **WAN-Strecke** | — | `10.0.0.0/30` | — | Point-to-Point (Essen <-> Dortmund) |
| **Dortmund (Filiale)**| — | `192.168.50.0/24` | `192.168.50.1` | Lokales LAN der Außenstelle |
| **Internet-Verbindung**| — | `198.51.100.0/30` | — | Point-to-Point (Essen <-> ISP-Router) |
| **Public Internet** | — | `8.8.8.0/24` | `8.8.8.1` | Simulierter Google-DNS Server (`8.8.8.8`) |

---

## 🛠️ Konfigurations-Details (CLI Core)

### 1. Inter-VLAN Routing (Router-on-a-Stick)
Auf dem `Router-Essen` terminiert ein Trunk-Port vom Switch. Virtuelle Subinterfaces leiten den Verkehr zwischen den internen VLANs weiter:

```text
interface GigabitEthernet0/0.10
 encapsulation dot1Q 10
 ip address 192.168.10.1 255.255.255.0

interface GigabitEthernet0/0.20
 encapsulation dot1Q 20
 ip address 192.168.20.1 255.255.255.0

interface GigabitEthernet0/0.30
 encapsulation dot1Q 30
 ip address 172.16.30.1 255.255.255.0

2. Dynamisches Routing (OSPFv2)
Die internen Firmennetze und die WAN-Verbindung tauschen Routing-Informationen dynamisch in Area 0 aus. Wildcard-Masken werden präzise angewendet:

router ospf 1
 router-id 1.1.1.1
 network 192.168.10.0 0.0.0.255 area 0
 network 192.168.20.0 0.0.0.255 area 0
 network 172.16.30.0 0.0.0.255 area 0
 network 10.0.0.0 0.0.0.3 area 0
 default-information originate

IT-Security & Netzwerk-Härtung
Gäste-Isolierung via Extended Access Control List (ACL)
Aus Compliance- und Sicherheitsgründen darf das Gäste-VLAN 30 unter keinen Umständen auf interne Unternehmensressourcen zugreifen. Es wird eine strikte Firewall-Regel (First-Match-Prinzip) am Inbound-Interface angewendet. Dies verhindert laterale Bewegungen (Lateral Movement) von potenziell infizierten Gäste-Geräten:

ip access-list extended GAESTE_SECURITY
 deny ip 172.16.30.0 0.0.0.255 192.168.10.0 0.0.0.255
 deny ip 172.16.30.0 0.0.0.255 192.168.20.0 0.0.0.255
 deny ip 172.16.30.0 0.0.0.255 192.168.50.0 0.0.0.255
 permit ip any any

interface GigabitEthernet0/0.30
 ip access-group GAESTE_SECURITY in

🌐 ISP-Anbindung & Internet-Simulation
Zur Simulation des realen Internetverkehrs ist der Router-Essen an einen simulierten Provider (ISP) angebunden.

Default Route & Propagation
Eine statische Standardroute leitet jeglichen unbekannten Traffic ins Internet weiter:

ip route 0.0.0.0 0.0.0.0 198.51.100.2

Durch den Befehl default-information originate innerhalb des OSPF-Prozesses injiziert der Hauptrouter diese Route automatisch in das gesamte Firmennetzwerk, sodass auch die Filiale Dortmund ohne manuelle Routen das Internet erreicht.

Provider-Rückrouting (Lab-Spezifisch)
Da im Labor-Szenario kein NAT-Overload betrieben wird, wurden auf dem ISP-Router statische Rückrouten definiert, damit Antworten aus dem Internet den Weg zurück in die privaten Subnetze (RFC 1918) des Unternehmens finden:

ip route 192.168.10.0 255.255.255.0 198.51.100.1
ip route 192.168.20.0 255.255.255.0 198.51.100.1
ip route 172.16.30.0 255.255.255.0 198.51.100.1
ip route 192.168.50.0 255.255.255.0 198.51.100.1

Simulation & Validierung
Die vollständige, interaktive Netzwerktopologie ist in der Datei unternehmensnetzwerk_ruhrgebiet.pkt hinterlegt und kann direkt in Cisco Packet Tracer geladen und getestet werden.

Test-Ergebnisse (Verifiziert):
Konnektivität zu Google (8.8.8.8): Erfolgreich von allen Endgeräten (Verwaltung, IT, Gäste, Dortmund).

Inter-VLAN Kommunikation: Verwaltung (192.168.10.10) kann IT-Infrastruktur (192.168.20.10) und Dortmund (192.168.50.10) erfolgreich pingen.

Sicherheits-Filterprüfung: Pings vom Gäste-PC (172.16.30.10) zu internen Firmen-IPs schlagen vereinbarungsgemäß fehl (Request timed out), während das Surfen im Internet (ping 8.8.8.8) einwandfrei funktioniert.
