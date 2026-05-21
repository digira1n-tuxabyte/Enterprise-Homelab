# Homelab
Welkom bij mijn centrale Homelab repository. Dit project fungeert als mijn persoonlijke omgeving waarin ik enterprise-grade infrastructuren ontwerp, test en beheer. 
Het doel van dit homelab is om een realistische, beveiligde bedrijfsomgeving na te bootsen. Dit stelt mij in staat om geavanceerde concepten rondom Linux Systeembeheer, Netwerkarchitectuur en Cybersecurity in de praktijk te brengen en continu te optimaliseren.

---

## Structuur

Hieronder vind je de navigatie naar de specifieke onderdelen van de infrastructuur:

* **[` netwerk-architectuur`](./netwerk-architectuur/)**
    * Bevat het visuele netwerkdiagram (topology).
    * Documentatie over de HP t730 OPNsense core router, 802.1Q VLAN-segmentatie en enterprise Wi-Fi (PPSK).
    * Uitleg over Layer 7 firewalling via Zenarmor.
* **[` docker-stacks`](./docker-stacks/)**
    * **[`reverse-proxy+crowdsec/`](./docker-stacks/reverse-proxy%2Bswag/)**: De actieve edge-beveiliging. Bevat `docker-compose.yml` voor SWAG (Nginx), Cloudflare Tunnels en de proactieve CrowdSec Multi-Agent IPS integratie.
    * **[`vps-monitoring/`](./docker-stacks/vps-monitoring/README.md)**: Bevat `docker commands`, Cloudflare Tunnels en monitoring tool UptimeKuma.
* **[` home-automation`](./home-automation/)**
    * **[`tv-proximity-sensor/`](./home-automation/tv-proximity-sensor/)**: Een tastbare IoT-integratie. Toont het gebruik van actieve 24GHz mmWave radar (ESPHome) en geavanceerde Node-RED logica voor dynamische API-aansturing van media-apparatuur.
    * **[`DIY Smart Doorbell (Analog to Digital)/`](./home-automation/smart-doorbell/)**: modificatie van een traditionele 'domme' deurbel naar een slim IoT-apparaat, direct geïntegreerd in Home Assistant. Het fungeert als de brug tussen een analoog stroomcircuit en het digitale netwerk.
      

---

## Core Tech Stack & Competenties

Binnen deze omgeving beheer en configureer ik actief de volgende technologieën:

* **Virtualisatie & Compute:** Proxmox VE, Ubuntu Server, LXC & Docker containerisatie.
* **Networking & Routing:** OPNsense, managed switching (802.1Q), WireGuard VPN, Private Pre-Shared Key (PPSK) wireless segmentatie.
* **Cybersecurity:** Layer 7 deep packet inspection (Zenarmor), CrowdSec (Behavioral Bouncer / Log Analysis), SSL/TLS management (Let's Encrypt / Reverse Proxying).
* **IoT & Event-Driven Automatisering:** Home Assistant, Node-RED (visuele logica & complexe API-calls), ESPHome (microcontrollers) en fysieke hardware/sensor-kalibratie.
* **Best Practices:** Volledige scheiding van geheimen via `.env` bestanden, strikte `.gitignore` handhaving.

---

## Toekomst & Roadmap

Dit homelab is continu in ontwikkeling. De komende perioden staan de volgende uitbreidingen gepland:

* [ ] **DIY IoT-Integratie (Ambilight):** Uitrollen van een dedicated HP T530 thin client voor het aansturen van custom omgevingsverlichting via HyperHDR en WLED (volledig geïsoleerd binnen het NoT-VLAN).
* [ ] **Centrale Logging:** Uitrollen van een centraal logging systeem (bijv. Grafana Loki of ELK-stack) om alle systeem- en containerlogs geconsolideerd binnen handbereik te hebben.
