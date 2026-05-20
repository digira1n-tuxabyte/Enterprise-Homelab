<img width="1883" height="897" alt="Screenshot 2026-05-18 141113" src="https://github.com/user-attachments/assets/3568e8d6-316c-42b9-a963-769da409b301" />
<img width="3654" height="1120" alt="network-topology" src="https://github.com/user-attachments/assets/48a50d1a-f3d4-4c9b-bcde-392e66ba2bb2" />
# Netwerk Architectuur & VLAN Segmentatie

## Core Technologieën
* **Hypervisor:** Proxmox VE
* **Firewall / Router:** OPNsense
* **Deep Packet Inspection:** Zenarmor
* **Switching/Access Points:** EAP225/EAP245 & TP-Link TL-SG105E managed switches (TP-Link Omada software controller)

## Hardware Specificaties
* **Firewall/Router Node:** HP t730 Thin Client
  * **Network Interface Card:** Fenvi 2.5Gb/s (Intel i226-V/i225-V gebaseerd)
  * **Uplink:** Nokia ONT (Fiber optic)
* **Core Switching:** 2x TP-Link TL-SG105E (Easy Smart Switches met 802.1Q VLAN ondersteuning)
  * *Locatie Begane Grond :* Gekoppeld via 2.5Gb/s backbone.
  * *Locatie Zolder :* Gekoppeld via High-Speed trunk link.
* **Wireless Infrastructure:** TP-Link Omada EAP225 (Begane Grond) & EAP245 (Zolder)

## VLAN Topologie
Om de veiligheid te waarborgen en broadcastverkeer te beperken, is het netwerk opgedeeld in de volgende geïsoleerde VLANs:

* **VLAN 10 - Main:** Vertrouwde apparaten en management interfaces (Toegang tot alle interne subnetten, gefilterd internetverkeer).
* **VLAN 20 - IoT (Internet of Things):** Slimme apparaten die externe internettoegang vereisen (mediaspelers, smart home hubs). Geen toegang tot Main.
* **VLAN 30 - NoT (Network of Things):** Apparaten die uitsluitend lokaal moeten communiceren. Externe internettoegang is op firewall-niveau volledig geblokkeerd.
* **VLAN 40 - DMZ:** Publiek toegankelijke webdiensten (via een reverse proxy). Volledig geïsoleerd van interne VLANs ter preventie van lateral movement.
* **VLAN 50 - Guest:** Geïsoleerd netwerk voor bezoekers met uitsluitend toegang tot het internet via een specifiek subnet.

## Draadloze Architectuur & PPSK
In plaats van het uitzenden van meerdere SSID's, maakt deze infrastructuur gebruik van **PPSK (Private Pre-Shared Key)** via de Omada Access Points:
* Er wordt één primair draadloos netwerk uitgezonden.
* Op basis van de unieke vooraf gedeelde sleutel (wachtwoord) wijst de controller het apparaat dynamisch toe aan **VLAN 30 (IoT)**, **VLAN 50 (Guest)** of **VLAN 10 (Main)**.
* Dit minimaliseert 'wireless airtime fairness' problemen en verhoogt de operationele veiligheid.

## DNS & Interne Resolutie
Om te voorkomen dat intern netwerkverkeer onnodig via het publieke internet (Cloudflare Tunnels/WAN) reist om bij lokale services te komen, is er **Split DNS** ingericht:

* **DNS Resolver (Unbound DNS):** Draait rechtstreeks op OPNsense en is voorzien van actieve ad- en telemetry-blokkades.
* **Wildcard DNS Override:** In Unbound is een host override ingesteld voor `*.domein.nl` die direct verwijst naar het interne IP-adres van de SWAG reverse proxy container. 
* **Resultaat:** Intern verkeer blijft volledig binnen het LAN en raakt de WAN-poort niet, wat zorgt voor maximale doorvoersnelheid en lagere latency.

## Geavanceerde Beveiliging & Dreigingsanalyse
* **Layer 7 Firewalling (Zenarmor):** Actief geconfigureerd op de IoT- en Guest-subnetten. Dit filtert netwerkverkeer op applicatie- en webniveau (zoals het blokkeren van bekende malafide servers en tracking), waardoor potentieel gecompromitteerde IoT-apparaten geen kwaad kunnen.
* **Gedistribueerde CrowdSec Multi-Agent Setup:** * Een centrale CrowdSec-omgeving analyseert in realtime de logs van zowel de **OPNsense-firewall** als de **SWAG reverse proxy container**.
  * Zodra er kwaadaardig verkeer (zoals poortscans op de firewall of brute-force pogingen op de webserver) wordt gedetecteerd, wordt de aanvaller direct op netwerkniveau geblokkeerd via een centrale blocklist.
* **Beheer & Toegang (Anti-Lockout):** Strikte scheiding van management-verkeer. SSH- en GUI-toegang tot core-systemen is uitsluitend toegestaan vanuit geverifieerde endpoints binnen het Main-VLAN, beveiligd met specifieke netwerk-uitsluitingen.
* **Remote Access:** Veilige toegang van buitenaf via een afgeschermde **WireGuard VPN-tunnel**, rechtstreeks beëindigd op de OPNsense-gateway.

## Edge Security & Geautomatiseerde Drop-Rules (OPNsense)

Om de *attack surface* te minimaliseren en interne resources te sparen, wordt de eerste verdedigingslinie al op de WAN-poort (de Edge) van de OPNsense firewall gehandhaafd. Verkeer van bekende *bad actors* bereikt hierdoor nooit de achterliggende reverse proxy of interne VLANs.

* **CrowdSec L3/L4 Bouncer:** De OPNsense firewall is uitgerust met een actieve CrowdSec bouncer. IP-adressen die lokaal (via SWAG logs) opvallen, óf die mondiaal op de CrowdSec blocklist staan, worden direct op Layer 3/Layer 4 netwerkniveau gedropt.
* **Resource Optimalisatie:** Door kwaadaardig verkeer aan de poort te blokkeren, wordt voorkomen dat de webservers CPU en opslag (logs) verspillen aan het verwerken van botnet-scans en brute-force pogingen.
* **Default Deny Extensie:** Naast de standaard inkomende drop-rules, zorgen deze threat-intelligence feeds voor een dynamische, real-time bijgewerkte blokkade van kwaadaardige infrastructuren.

## Compute & Virtualisatie (Attic Stack)
Al de interne applicaties en services draaien gevirtualiseerd op een dedicated **Proxmox VE hypervisor** node op de zolderverdieping:
* **Ubuntu Media Server:** Draait onder andere een mediaserver. De logs hiervan worden via een read-only mount direct doorgestuurd naar CrowdSec voor anomaliedetectie.
* **Netwerkaansluiting:** De hypervisor is via een trunk-verbinding aangesloten op de Attic Switch, waardoor virtuele machines direct in hun respectievelijke VLANs (zoals de DMZ voor publieke webdiensten) kunnen worden geplaatst.
