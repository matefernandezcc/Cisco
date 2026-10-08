# Estado y pasos pendientes – Parcial Redes 1234 (Regional A)

Fuente de verdad: `Requerimientos.pdf` (no inventar nada; ante duda, **preguntar al usuario**).
Decisiones del usuario: SSID = `Oculta` (literal); contraseña WPA2 = `U7n+FR84` (la del PDF); contraseña global de switches = `redes` (literal).

## Cómo operar el MCP (lecciones aprendidas)
- Usar `pt_send_raw` con `configureIosDevice('<device>', 'cmd1\ncmd2...')` (en JS, `\\n` dentro del string, una sola línea).
- **Mandar lotes CORTOS** (≤ ~12 comandos, una sección por llamada). Un lote largo en RouterA dejó la config corrupta (IP de la WAN cayó en Fa0/0) y hubo que reparar.
- Verificar siempre con `pt_backup_config(device)` (muestra la config) y `pt_verify_connectivity(from_device,to_ip,timeout_s)`.
- Nombres exactos en PT: `ACCESO 1 A`, `ACCESO 2 A`, `DISTRIBUCION A`, `Router A` (hostname ya cambiado a `RouterA`), `Wireless Router A`, `Admin LAN`, `Admin WLAN`, `Smartphone1`, `Router5`, `Router6`.
- Hosts por JS: `ipc.network().getDevice(n).getPort('FastEthernet0').setIpSubnetMask(ip,mask)` / `.setDefaultGateway(gw)`.
- `pt_save_project` falló ("no se creo ...pkt"): **guardar a mano con Ctrl+S**.

## HECHO ✅
| Fase | Estado |
|---|---|
| 1 Cableado | 11 enlaces creados según tabla del PDF (cobre directo hosts/router, cruzado entre switches, serial Router6 Se0/0/1 ↔ RouterA Se0/0/0; RouterA quedó como DTE, sin clock rate) |
| 2 Switches | `enable secret redes`; VLAN 10 INVESTIGACION / 20 ACADEMICA / 30 ADMINISTRACION en los 3 switches (verificado con `pt_read_vlans`); puertos de acceso y trunks `allowed vlan 10,20` configurados |
| 2.5 IPs hosts | PC1 .10.1, PC2 .20.2, PC3 .10.3, PC4 .20.4 (GW .254 de su VLAN); Admin LAN .30.1; Admin WLAN .30.2 (sin GW) |
| 4 RouterA | hostname `RouterA`; Fa0/0.10 y Fa0/0.20 (dot1Q, .254/24); Se0/0/0 PPP 50.100.0.1/30 `no shutdown` |
| 5 EIGRP | `router eigrp 99`: 50.100.0.0 0.0.0.3, 192.168.10.0, 192.168.20.0; sin VLAN 30; `no auto-summary` |
| 6 ACL (parcial) | ACL 110 (VLAN10) y 120 (VLAN20) aplicadas `in` en Fa0/0.10 / Fa0/0.20 |
| 7 IPsec | ACL 100, isakmp policy 100, key `a.frba` peer 55.55.0.1, transform-set `Regional.a1`, crypto map `mapa-a1` 100 (lifetime 900), aplicado en Se0/0/0 |

Pruebas OK: PC1→GW, PC1→192.168.111.111 (HTTPS, directo), PC2→GW; PC1→PC2 falla (ACL, esperado).

## PENDIENTE ❌ (en orden de prioridad)

### A. IPsec NO levanta: PC2 → 192.168.100.100 sin respuesta (100% pérdida)
Diagnóstico: en **Router5** la crypto ACL del mapa `mapa-a1` (ACL 100) es
`permit ip host 192.168.100.100 192.168.2.0 0.0.0.31` → **no es el espejo** de la ACL 100 de RouterA (`192.168.20.0/24 → host 192.168.100.100`). Phase 2 no puede coincidir.
El PDF dice que Router5 "está configurado correctamente" → **preguntar al usuario** si se permite modificar Router5 (propuesta, no aplicada: `access-list 100 permit ip host 192.168.100.100 192.168.20.0 0.0.0.255`) o si hay algo que revisar de RouterA (ej.: ISAKMP, ruta a 55.55.0.1).
Otros chequeos: `show crypto isakmp sa`, `show crypto ipsec sa`, ping RouterA→55.55.0.1 (hacerlo con `ping 55.55.0.1 source 50.100.0.1` desde CLI) y `show ip route` (rutas D a 55.55.0.0, 192.168.100.0, 192.168.111.0), `show ip eigrp neighbors` (¿vecino Router6?). Router5 usa `auto-summary` en EIGRP; no debería afectar.
Prueba final del requisito: en PC2/PC4 `ftp 192.168.100.100` (docente/docente) y ver contadores `#pkts encaps/decaps` > 0.

### B. Fase 3 – Wireless Router A (solo por GUI, la API no lo permite)
1. Setup → modo **Bridge** (NO gateway/router). Dejarle IP LAN en 192.168.30.x (preguntar cuál si no es evidente).
2. Wireless: SSID `Oculta`, **SSID broadcast deshabilitado**.
3. Wireless Security: **WPA2-Personal**, **AES**, clave `U7n+FR84`.
4. DHCP: el Smartphone debe recibir IP **192.168.30.101–.110** (⚠ en modo Bridge el DHCP interno puede quedar deshabilitado; hay que **preguntar al usuario quién debe ser el servidor DHCP** — la consigna no lo dice).
5. Smartphone1 → Config → Wireless0: SSID `Oculta`, WPA2-PSK, AES, clave `U7n+FR84`, IP por DHCP.
6. Verificar: ping Smartphone ↔ Admin LAN ↔ Admin WLAN; **sin** ping a VLAN 10/20 ni a la WAN.

### C. Fase 6 – ACL de VLAN 30 (decisión pendiente)
Requerimientos: denegar VLAN30 → VLAN10, VLAN20 y todo destino externo. La VLAN 30 no tiene subinterfaz ni se transporta por el trunk (queda aislada naturalmente), así que no hay interfaz del router donde aplicarla. **Preguntar al usuario** dónde/si la quiere (ej.: ACL 130 `deny ip 192.168.30.0 0.0.0.255 any` creada sin aplicar, o con una subinterfaz VLAN 30 que el PDF no pide).
Nota contradicción: `Objetivos-Condiciones.pdf` ítem 8 dice que VLAN 30 *accede* a HTTPS/FTP; `Requerimientos.pdf` dice que se **niega**. Se sigue Requerimientos.

### D. Detalles menores
- `show run` de RouterA muestra `network 192.168.10.0` / `192.168.20.0` sin wildcard (PT los trata classful; equivalente). Si el docente exige wildcard, reaplicar `network 192.168.10.0 0.0.0.255`.
- Confirmar que Router6 s0/0/1 usa **PPP** (si el enlace no sube, `encapsulation ppp` ahí; no se tocó).
- Verificar `show ip eigrp neighbors` en RouterA (vecino 50.100.0.2) y `traceroute 192.168.111.111` desde PC1 (debe ir directo, sin túnel).
- Guardar configs: `copy running-config startup-config` en RouterA y switches, y Ctrl+S del .pka.
- Tomar el % de **Activity Wizard** (Check Results) al terminar.

## Checklist final (1.3 del PDF)
- [x] PC1 → HTTPS OK (falta tracert y probar PC3 desde navegador)
- [ ] PC2/PC4 → FTP por túnel IPsec (bloqueado por A)
- [ ] VLAN 30: ping mutuo entre Admin LAN / Admin WLAN / Smartphone; sin salida
- [x] PC1 → PC2 falla por ACL
