# Labor 2: Unternehmens-Netzwerkdesign & Routing-Architektur (Cisco CCNA)

## 📋 Szenario-Übersicht
In diesem Labor wird die Netzwerk-Infrastruktur für ein expandierendes mittelständisches Unternehmen im Ruhrgebiet simuliert. Das Unternehmen verfügt über eine Hauptzentrale in **Essen** und eine neu angebundene Filiale in **Dortmund**. Ziel ist es, ein hochverfügbares, segmentiertes und sicheres Netzwerk zu entwerfen und zu konfigurieren.

Die Umsetzung und Validierung dieses Szenarios erfolgt mittels **Cisco Packet Tracer / GNS3**.

---

## 🏗️ Netzwerk-Architektur & Segmentierung

### 1. LAN-Segmentierung via VLANs (Zentrale Essen)
Um Broadcast-Domänen zu verkleinern und die Sicherheit zu erhöhen, wird das lokale Netzwerk in der Zentrale strukturiert aufgeteilt:
* **VLAN 10 (Verwaltung/Finance):** Subnetz `192.168.10.0/24`
* **VLAN 20 (IT-Infrastruktur):** Subnetz `192.168.20.0/24`
* **VLAN 30 (Gäste-Netzwerk):** Subnetz `172.16.30.0/24` (Isoliert)
* **VLAN 99 (Native/Management):** Subnetz `10.99.99.0/24` (Für SSH-Zugriff auf Switches/Router)

### 2. Inter-VLAN Routing & Trunking
* Die Switches sind über **802.1Q Trunk-Links** miteinander verbunden. Nicht benötigte Ports werden aus Sicherheitsgründen abgeschaltet und dem ungenutzten VLAN 999 zugewiesen.
* Das Routing zwischen den VLANs wird über ein **Router-on-a-Stick (RoaS)** Design am zentralen Core-Router realisiert (Subinterfaces `Gig0/0.10`, `Gig0/0.20`, etc. mit `encapsulation dot1Q`).

---

## 🌐 Routing & WAN-Anbindung (Essen <-> Dortmund)

### 1. Dynamisches Routing mit OSPFv2
Für den Routing-Informationsaustausch zwischen dem Hauptstandort Essen und der Filiale Dortmund wird **OSPF (Open Shortest Path First)** in Area 0 konfiguriert:
* **Router-IDs:** Essen-Router (`1.1.1.1`), Dortmund-Router (`2.2.2.2`)
* **Netzwerk-Ankündigungen:** Es werden nur die produktiven internen Subnetze via OSPF transportiert. Die Point-to-Point-Verbindung nutzt das hocheffiziente `/30`-Netzwerk (`10.0.0.0/30`).

### 2. Internet-Anbindung & NAT/PAT
* **Static Default Route:** Der Edge-Router in Essen leitet jeglichen unbekannten Traffic über eine statische Standard-Route (`0.0.0.0 0.0.0.0`) an den ISP-Router weiter.
* **Port Address Translation (PAT):** Um IP-Adressen zu sparen, wird **NAT Overload** konfiguriert. Alle internen Mitarbeiter teilen sich beim Surfen im Internet die einzige öffentliche IP-Adresse des Außen-Interfaces.

---

## 🔒 Netzwerksicherheit (Security Hardening)

* **Sicherer Geräte-Zugriff:** Konsolen- und VTY-Linien werden mit starken Passwörtern geschützt. Unverschlüsselte Passwörter in der Konfiguration werden mittels `service password-encryption` unlesbar gemacht. SSH (Version 2) wird erzwungen (`crypto key generate rsa 2048`), Telnet ist komplett deaktiviert.
* **Access Control Lists (ACLs):** Eine erweiterte Access-Liste (Extended ACL) blockiert jeglichen Zugriff aus dem Gäste-VLAN 30 auf die internen Server im IT-Infrastruktur-VLAN 20, erlaubt den Gästen aber den Zugriff auf das Internet.

---

## 💻 Beispiel-Konfigurationsauszug (Cisco CLI)

Hier ist ein Auszug der grundlegenden Router-Konfiguration für das Inter-VLAN-Routing und OSPF:

```text
! --- INTER-VLAN ROUTING (ROUTER-ON-A-STICK) ---
interface GigabitEthernet0/0
 no shutdown
!
interface GigabitEthernet0/0.10
 encapsulation dot1Q 10
 ip address 192.168.10.1 255.255.255.0
!
interface GigabitEthernet0/0.20
 encapsulation dot1Q 20
 ip address 192.168.20.1 255.255.255.0

! --- OSPF ROUTING CONFIGURATION ---
router ospf 1
 router-id 1.1.1.1
 network 192.168.10.0 0.0.0.255 area 0
 network 192.168.20.0 0.0.0.255 area 0
 network 10.0.0.0 0.0.0.3 area 0
 log-adjacency-changes

Die vollständige interaktive Netzwerktopologie kann über die Datei `unternehmensnetzwerk_ruhrgebiet.pkt` oben im Ordner direkt in Cisco Packet Tracer geladen und getestet werden.
