# External Monitoring & VPS

In een eerdere instantie draaide ik Uptime Kuma lokaal binnen de Proxmox-omgeving. Dit introduceerde een *Single Point of Failure*: zodra de lokale hypervisor of de internetverbinding thuis uitviel, ging het monitoringsplatform mee down. 

**Oplossing:** De monitoring is gemigreerd naar een onafhankelijke, externe cloud VPS. Hierdoor blijft de monitoring en alerting actief, ook als de complete lokale infrastructuur thuis spanningsloos is.

---

## VPS OS Configuratie

* **SSH Key-Only Authentication:** Wachtwoord-authenticatie is volledig uitgeschakeld in de SSH-daemon (`PasswordAuthentication no`). Toegang tot de VPS is uitsluitend mogelijk via cryptografische SSH-keys.
* **Automated Updates:** Het systeem is geconfigureerd voor automatische updates (`unattended-upgrades`). Kritieke security-patches voor het OS worden dagelijks zonder handmatige tussenkomst geïnstalleerd.
* **Netwerk-omgeving:** Er zijn **geen inkomende poorten geopend** in de cloud-firewall voor webverkeer. De toegang tot de Uptime Kuma GUI verloopt volledig via een uitgaande Cloudflare Tunnel.

---

## Deployment
De services zijn rechtstreeks via de Docker CLI uitgerold. Om automatische container-updates te garanderen (zodat ook de applicaties zelf up-to-date blijven), is er een update-mechanisme/Watchtower actief.

### Uptime Kuma:
```bash
docker run -d \
  --name uptime-kuma \
  -p 3001:3001 \
  -v ./uptime-kuma-data:/app/data \
  --restart unless-stopped \
  louislam/uptime-kuma:1
 
### Cloudflare tunnel:
  docker run -d \
  --name cloudflared_vps_tunnel \
  --restart unless-stopped \
  cloudflare/cloudflared:latest \
  tunnel --no-autoupdate run --token <VPS_TUNNEL_TOKEN>
  
  De echte tunnel-token is om veiligheidsredenen weggelaten uit deze publieke documentatie
  
  
 ### Watchtower:
 docker run -d   --name watchtower   --restart always   -v /var/run/docker.sock:/var/run/docker.sock   nickfedor/watchtower   --cleanup
  
