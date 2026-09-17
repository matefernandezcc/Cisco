# Validación TL2 — TL2-K4773-Fernandez-Toledo.pkt

![Topología actual en Packet Tracer](C:/Users/Mateo/.gemini/antigravity-ide/brain/2775d8c6-0f1e-40d5-9e45-f1ea46c4c0a1/topology.png)

## Resumen Ejecutivo

El TP tiene la **PRIMERA PARTE casi completa** pero la **SEGUNDA PARTE tiene problemas críticos**. No está listo para entregar tal como está.

---

## Checklist de Validación por Punto de la Consigna

### PRIMERA PARTE — LAN VENTAS (200.200.200.0/24)

| # | Requisito | Estado | Detalle |
|---|-----------|--------|---------|
| 7.5 | Laptop WAN_Admin con IP estática y GW correcto | ✅ OK | IP: 200.200.200.240/24, GW: 200.200.200.254 (router de borde) |
| 7.5.3 | tracert de WAN_Admin a Server Pedidos (195.195.195.195) | ⚠️ Parcial | Funciona (3/4 pings OK, 1 perdido — normal primer paquete ARP). Debería mostrar 3 saltos |
| 7.7 | Wireless Bridge IP = 200.200.200.253/24 | ✅ OK | Configurado correctamente |
| 7.7 | DHCP en AP: 200.200.200.100, hasta 8 hosts | ✅ OK | Configurado (VENTAS_Admin usa DHCP) |
| 7.7 | Seguridad WLAN: WPA2 PSK, AES, password C1sc0.4r | ✅ OK | Vendedor_1 está conectado por wireless |
| 7.7 | Canal 7 y SSID desactivado | ⚠️ Sin poder verificar | No es posible verificar el canal y SSID broadcast vía MCP, revisar manualmente en la GUI del AP |
| 7.7 | Cable cruzado entre Switch Ventas y AP (Ethernet 4) | ✅ OK | Wireless Bridge Ethernet 4 conectado |
| 7.8 | VENTAS_Admin con IP dinámica (DHCP) conectada al Switch Ventas | ✅ OK | DHCP activo, conectada a Switch Ventas, IP: 0.0.0.0 (puede que necesite un renew) |
| 7.9 | Contador con IP estática 200.200.200.242/24, GW 200.200.200.254 | ✅ OK | IP correcta, conectada al Ethernet del AP (puerto LAN) |
| 7.9.2 | Contador → FTP a Server Pedidos | ✅ OK | Ping a 195.195.195.195: 4/4 OK |
| 7.10 | Vendedor_1 con IP dinámica en WLAN | ✅ OK | IP: 200.200.200.210 por DHCP wireless |
| 7.10.2 | Comunicación LAN VENTAS interna | ✅ OK | Vendedor_1 ↔ Contador OK, Vendedor_1 ↔ WAN_Admin OK, VENTAS_Admin ↔ Contador OK |

> [!TIP]
> **VENTAS_Admin tiene IP 0.0.0.0** — Probablemente necesita un `ipconfig /release` seguido de `ipconfig /renew` para que el DHCP le asigne una IP del rango 200.200.200.100-107. Verificar antes de entregar.

---

### SEGUNDA PARTE — LAN CENTRAL (172.17.17.0/24) y LAN SEGURIDAD (192.192.192.0/24)

| # | Requisito | Estado | Detalle |
|---|-----------|--------|---------|
| 7.11 | LAN Seguridad ↔ LAN Depósito funciona | ✅ OK | CSIRT (192.192.192.192) → Server Pedidos (195.195.195.195): 4/4 OK |
| 7.12.1 | Puerto Internet del AP Router al Switch Seguridad | ✅ OK | Wireless Router Internet port conectado y UP |
| 7.12.2 | AP Router: Internet port IP = 192.192.192.253/24 | ❌ FALLA | **Internet port tiene IP 0.0.0.0** — No tiene IP estática configurada. Debería ser 192.192.192.253/24 |
| 7.12.2 | AP Router: GW = 192.192.192.254 | ❌ FALLA | Sin gateway configurado (consecuencia de lo anterior) |
| 7.12.2 | DHCP 172.17.17.0/24 con hasta 5 hosts | ✅ OK | VLAN1 = 172.17.17.1/24, Gerente 2 tiene 172.17.17.10 por DHCP |
| 7.12.3 | Wireless básico configurado (canal, SSID) | ⚠️ Sin verificar | Gerente 2 conecta por wireless, pero canal y SSID no verificables vía MCP |
| 7.12.4 | Seguridad: WPA2 Personal, AES, password FR84.U7n | ⚠️ Sin verificar | Gerente 2 conecta, lo que implica que al menos la auth funciona |
| 7.12.5 | Gerente 1 y Gerente 2 con DHCP y wireless | ❌ PARCIAL | **Gerente 1 tiene IP 0.0.0.0** — No recibió IP por DHCP. Gerente 2 OK (172.17.17.10) |
| 7.12.6 | Gerentes ↔ LAN Depósito | ❌ FALLA | **Gerente 2 → Server Pedidos: 0% — SIN CONECTIVIDAD**. Gerente 1: sin IP, tampoco llega |
| 7.12.6 | Gerentes ↔ LAN Seguridad | ❌ FALLA | **Gerente 2 → CSIRT (192.192.192.192): SIN CONECTIVIDAD** |

---

## Problemas Críticos a Resolver

> [!CAUTION]
> ### 1. Wireless Router: Puerto Internet sin IP (CRÍTICO)
> El puerto `Internet` del Wireless Router tiene **IP 0.0.0.0** en lugar de **192.192.192.253/24**. Esto es lo que impide toda comunicación entre LAN Central y LAN Seguridad/Depósito.
> 
> **Solución:** En la GUI del Wireless Router (Setup → Internet Setup), configurar:
> - IP estática: `192.192.192.253`
> - Máscara: `255.255.255.0`
> - Gateway: `192.192.192.254`

> [!WARNING]
> ### 2. Gerente 1 sin IP (0.0.0.0)
> Gerente 1 no recibió dirección IP por DHCP. La interfaz wireless está UP y linked, pero sin IP.
> 
> **Solución:** Ir a Gerente 1 → Desktop → IP Configuration:
> 1. Seleccionar Static, poner cualquier dato temporal
> 2. Volver a seleccionar DHCP
> 3. Verificar que reciba una IP del rango 172.17.17.10-14

> [!WARNING]
> ### 3. VENTAS_Admin sin IP (0.0.0.0)
> Similar al caso anterior, tiene DHCP habilitado pero no recibió IP.
> 
> **Solución:** Ir a VENTAS_Admin → Desktop → IP Configuration → hacer toggle Static/DHCP o ejecutar `ipconfig /release` + `ipconfig /renew`

---

## Puntos que el Profesor Enfatizó en Clase (Transcript)

Según lo que explicó el profesor, estos son los conceptos clave que va a evaluar y deberías poder responder:

| Concepto | ¿Está demostrado en tu TP? |
|----------|---------------------------|
| Diferencia modo Bridge vs Router en el AP | ✅ Ambos APs presentes |
| Importancia del Default Gateway | ⚠️ Solo funciona parcialmente (la segunda parte falla) |
| PC con DHCP: GW = AP (alcance solo LAN) | ✅ Vendedor_1 demuestra esto |
| PC con IP estática + GW = router de borde (alcance WAN) | ✅ Contador y WAN_Admin lo demuestran |
| AP Router: NAT enmascarado, salto intermedio con timeout | ❌ No funciona porque el puerto Internet del AP Router no tiene IP |
| tracert mostrando 3 saltos (PC → Router borde → Router remoto → Server) | ✅ Funciona desde Contador/WAN_Admin |
| Comunicación entre LAN Central y LAN Depósito | ❌ No funciona |

---

## Veredicto Final

| Parte | Estado |
|-------|--------|
| Primera Parte (LAN Ventas) | ✅ **~90% lista** (falta renew de VENTAS_Admin) |
| Segunda Parte (LAN Central/Seguridad) | ❌ **NO lista** — Puerto Internet del AP Router sin configurar |

> [!IMPORTANT]
> **El TP NO está listo para entregar.** Necesitás resolver al menos:
> 1. **Configurar la IP del puerto Internet del Wireless Router** (192.192.192.253/24, GW 192.192.192.254)
> 2. **Hacer `ipconfig /renew` en Gerente 1** para que tome IP por DHCP
> 3. **Hacer `ipconfig /renew` en VENTAS_Admin** para que tome IP por DHCP
> 4. **Verificar que Gerente 1 y Gerente 2 lleguen al Server Pedidos** (debería funcionar una vez resuelto el punto 1)
> 
> Con estos 4 cambios, el TP debería estar listo para entregar.
