## (Local-Only Services)

Hoewel alle subdomeinen via DNS naar deze SWAG-container worden gerouteerd, zijn een aantal kritieke interne services (zoals management interfaces) **uitsluitend bereikbaar vanaf het lokale netwerk**.

Dit is op proxy-niveau dichtgetimmerd via Nginx Access Control Lists (ACLs).

## Web Application Security (WAF) & Log Parsing

Naast netwerksegmentatie wordt inkomend webverkeer actief gemonitord en gefilterd door een  **CrowdSec** implementatie. Deze setup fungeert niet alleen als een gedragsgebaseerde IPS, maar biedt volwaardige **Web Application Firewall (WAF)** functionaliteit.

### 1. Actieve Log Parsing
De CrowdSec agent parseert in real-time de logs van verschillende vitale services binnen het homelab (aangeboden via read-only Docker volumes). Afwijkend gedrag, zoals poortscans, brute-force pogingen of ongebruikelijke HTTP-requests, resulteert direct in netwerkblokkades. Gemonitorde logbronnen zijn onder andere:
* SWAG (Nginx) access- en error-logs
* Onderliggende applicatielogs (zoals de media streaming engine)

### 2. OWASP Top 10 & Exploit bescherming

De achterliggende webdiensten zijn actief beschermd tegen moderne aanvalsvectoren en de **OWASP Top 10 vulnerabilities**. Dit wordt gerealiseerd via de volgende actieve CrowdSec collections:

* **AppSec & WAF (Layer 7 Filtering):** * `appsec-crs` (Core Rule Set - Bescherming tegen o.a. SQLi, XSS, LFI).
  * `appsec-virtual-patching` (Directe mitigatie van opkomende zero-days en CVE's zonder de applicatie te hoeven updaten).
  * `appsec-generic-rules`
* **HTTP & CVE Preventie:** * `http-cve` (Voorkomt misbruik van bekende kwetsbaarheden).
  * `base-http-scenarios` (Detecteert agressieve crawlers en path-traversal pogingen).
* **Service Protectie & OS Hardening:** * `nginx`, `linux`, `sshd` (Beveiliging tegen protocol-specifieke aanvallen).
* **False-Positive Reductie:** * `whitelist-good-actors` (Garandeert ongehinderde toegang voor legitieme diensten en zoekmachines).

*(Een visuele demonstratie van deze bescherming vindt u aan het einde van dit document).*

### Implementatie:
In de specifieke Nginx proxy-configuraties (`.subdomain.conf`) van lokale services is een custom beveiligingsblok ingeladen via een include-bestand (bijv. `ssl.conf` of een losse `local-only.conf`):

**Werking:** Wanneer een externe gebruiker via het internet (of de Cloudflare Tunnel) probeert te navigeren naar een lokale service, zal Nginx de HTTP-aanvraag direct weigeren en een 403 Forbidden statuscode teruggeven. Alleen verkeer afkomstig uit de vertrouwde `10.10.0.0/16` VLANs krijgt een succesvolle verbinding.

```nginx
# /config/nginx/site-confs/local-only.conf

# Toegang uitsluitend toestaan voor de interne netwerkranges
allow 10.10.0.0/16;

# Blokkeer al het overige (externe) verkeer
deny all;
```
</p>

---

## Visuele Demonstratie: CrowdSec AppSec in Actie

De onderstaande GIF toont hoe de CrowdSec AppSec agent een gesimuleerde aanval detecteert en onmiddellijk blokkeert, wat resulteert in het 403 Forbidden scherm. Deze bescherming wordt mogelijk gemaakt door de geconfigureerde OWASP Core Rule Set (`appsec-crs`).

<p align="center">
  <img src="https://github.com/user-attachments/assets/ca38c4d3-2147-4ae8-8126-d20671c6c629" width="240" alt="CrowdSec AppSec Blokkade Demonstratie" />
</p>
