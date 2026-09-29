# Trabajo de Laboratorio Nº 4 — Validación y Documentación Técnica
## Configuración Avanzada de Routers y Redes Privadas Virtuales (VPN IPSec)

**Materia:** Redes de Información — UTN FRBA (Nivel 4)  
**Grupo:** Grupo 26  
**Integrantes:** Mateo Fernandez Cruz y Franco Agustin Toledo  
**Archivo PKT resultante:** `TL4-K4773-Fernandez-Toledo.pkt`  

---

## 1. Resumen de la Implementación

Se completaron con éxito todas las fases de configuración requeridas para el Trabajo de Laboratorio Nº 4 sobre la maqueta en Cisco Packet Tracer:

1. **Configuración de Conmutación (Switches Layer 2):**
   - **ACCESO 1:** Puertos Fa0/1 en VLAN 10 (Access), Fa0/2 en VLAN 20 (Access), Gi0/1-2 en modo Trunk.
   - **ACCESO 2:** Puertos Fa0/1 en VLAN 10 (Access), Fa0/2 en VLAN 20 (Access), Gi0/1-2 en modo Trunk.
   - **DISTRIBUCIÓN:** Puerto Fa0/1 en VLAN 1 (Access, LAN Admin), Gi0/1-2 y Fa0/24 en modo Trunk hacia Router1.

2. **Configuración de Endpoints y Subredes:**
   - **PC1 (VLAN 10):** IP `10.10.0.1/24`, Gateway `10.10.0.254`
   - **PC2 (VLAN 20):** IP `10.20.0.2/24`, Gateway `10.20.0.254`
   - **PC3 (VLAN 10):** IP `10.10.0.3/24`, Gateway `10.10.0.254`
   - **PC4 (VLAN 20):** IP `10.20.0.4/24`, Gateway `10.20.0.254`
   - **LAN Admin (VLAN 1):** IP `10.50.0.50/24`, Gateway `10.50.0.254`
   - **Server0:** IP `10.4.0.40/24`, Gateway `10.4.0.1`
   - **Server1:** IP `10.3.0.30/24`, Gateway `10.3.0.1`

3. **Router1 — Subinterfaces dot1Q (Router-on-a-Stick) y Enlace WAN:**
   - `FastEthernet0/0`: Interfaz física levantada (`no shutdown`).
   - `FastEthernet0/0.1`: `encapsulation dot1Q 1 native`, IP `10.50.0.254 255.255.255.0`.
   - `FastEthernet0/0.10`: `encapsulation dot1Q 10`, IP `10.10.0.254 255.255.255.0`.
   - `FastEthernet0/0.20`: `encapsulation dot1Q 20`, IP `10.20.0.254 255.255.255.0`.
   - `Serial0/0/1`: IP `10.1.0.1 255.255.255.0` hacia ISP (`10.1.0.2`).

4. **Enrutamiento Dinámico y Estático:**
   - Ruta por defecto en Router1: `ip route 0.0.0.0 0.0.0.0 10.1.0.2`
   - EIGRP AS 1 en Router1, publicando las redes LAN/VLAN y el enlace serial con auto-summary.

5. **Túnel VPN IPSec Site-to-Site (Router1 ↔ Router2):**
   - **Fase 1 (IKE / ISAKMP):** Política 10 con cifrado `AES`, autenticación `pre-share`, grupo Diffie-Hellman `5`, lifetime `900` segundos. Pre-shared key `"cisco"` vinculada a la IP pública remota `10.2.0.2`.
   - **Fase 2 (IPSec):** Transform-set 50 con `ah-sha-hmac esp-3des`. Crypto map `mymap` con peer `10.2.0.2`, SA lifetime `1800` segundos, transform-set `50` y match address `101`.
   - **Activación:** Crypto map `mymap` aplicado a la interfaz `Serial0/0/1`.
   - **Tráfico Criptográfico (ACL 101):** Permite el tráfico de origen VLAN 10 (`10.10.0.0`) hacia Server0 (`10.4.0.0`) y Server1 (`10.3.0.0`). Se actualizó de forma simétrica y coordinada en Router2 para ambos servidores.

6. **Segmentación y Seguridad Inter-VLAN (ACLs Extendidas):**
   - Enfoque profesional implementado conforme a la clase del docente: cada VLAN permite de forma explícita únicamente el tráfico hacia sus propios recursos y los servidores remotos, dejando actuar el **implicit deny** al final de la ACL para aislar las VLANs entre sí.
   - **ACL 110** (VLAN 10) aplicada en `FastEthernet0/0.10 in`.
   - **ACL 120** (VLAN 20) aplicada en `FastEthernet0/0.20 in`.
   - **ACL 150** (VLAN 1 nativa) aplicada en `FastEthernet0/0.1 in`.

---

## 2. Configuraciones IOS Clave Aplicadas

### Router1 — Configuración VPN y ACLs

```ios
hostname Router1
!
crypto isakmp policy 10
 encr aes
 authentication pre-share
 group 5
 lifetime 900
!
crypto isakmp key cisco address 10.2.0.2
!
crypto ipsec transform-set 50 ah-sha-hmac esp-3des
!
crypto map mymap 10 ipsec-isakmp 
 set peer 10.2.0.2
 set security-association lifetime seconds 1800
 set transform-set 50 
 match address 101
!
interface FastEthernet0/0
 no ip address
 duplex auto
 speed auto
!
interface FastEthernet0/0.1
 encapsulation dot1Q 1 native
 ip address 10.50.0.254 255.255.255.0
 ip access-group 150 in
!
interface FastEthernet0/0.10
 encapsulation dot1Q 10
 ip address 10.10.0.254 255.255.255.0
 ip access-group 110 in
!
interface FastEthernet0/0.20
 encapsulation dot1Q 20
 ip address 10.20.0.254 255.255.255.0
 ip access-group 120 in
!
interface Serial0/0/1
 description hacia ISP
 ip address 10.1.0.1 255.255.255.0
 crypto map mymap
!
router eigrp 1
 network 10.10.0.0 0.0.0.255
 network 10.20.0.0 0.0.0.255
 network 10.50.0.0 0.0.0.255
 network 10.1.0.0 0.0.0.255
 auto-summary
!
ip route 0.0.0.0 0.0.0.0 10.1.0.2
!
! Tráfico VPN (Fase 2)
access-list 101 permit ip 10.10.0.0 0.0.255.255 10.4.0.0 0.0.0.255
access-list 101 permit ip 10.10.0.0 0.0.255.255 10.3.0.0 0.0.0.255
access-list 101 permit ip 10.10.0.0 0.0.255.255 10.4.0.0 0.0.255.255
access-list 101 permit ip 10.10.0.0 0.0.255.255 10.3.0.0 0.255.255
!
! ACL 110 (VLAN 10)
access-list 110 permit ip 10.10.0.0 0.0.0.255 10.3.0.0 0.0.0.255
access-list 110 permit ip 10.10.0.0 0.0.0.255 10.4.0.0 0.0.0.255
access-list 110 permit ip 10.10.0.0 0.0.0.255 10.10.0.0 0.0.0.255
!
! ACL 120 (VLAN 20)
access-list 120 permit ip 10.20.0.0 0.0.0.255 10.3.0.0 0.0.0.255
access-list 120 permit ip 10.20.0.0 0.0.0.255 10.4.0.0 0.0.0.255
access-list 120 permit ip 10.20.0.0 0.0.0.255 10.20.0.0 0.0.0.255
!
! ACL 150 (VLAN 1 / LAN Admin)
access-list 150 permit ip 10.50.0.0 0.0.0.255 10.3.0.0 0.0.0.255
access-list 150 permit ip 10.50.0.0 0.0.0.255 10.4.0.0 0.0.0.255
access-list 150 permit ip 10.50.0.0 0.0.0.255 10.50.0.0 0.0.0.255
```

### Router2 — Actualización ACL 101 (Retorno VPN)

```ios
access-list 101 permit ip 10.4.0.0 0.0.0.255 10.10.0.0 0.0.255.255
access-list 101 permit ip 10.3.0.0 0.0.0.255 10.10.0.0 0.0.255.255
access-list 101 permit ip 10.4.0.0 0.0.255.255 10.10.0.0 0.0.255.255
access-list 101 permit ip 10.3.0.0 0.0.255.255 10.10.0.0 0.0.255.255
```

---

## 3. Matriz de Validación de Conectividad y Seguridad

Todas las pruebas fueron ejecutadas directamente sobre el motor de simulación de Packet Tracer, confirmando el comportamiento esperado al 100%:

| Test Nº | Origen | Destino | Tipo de Tráfico | Esperado | Resultado Real | Métricas / Paquetes |
|---|---|---|---|---|---|---|
| **1** | PC1 (10.10.0.1) | Server0 (10.4.0.40) | VPN IPSec | ✅ Permitido (3 saltos) | **EXITOSO** | 4/4 recibidos (0% loss) |
| **2** | PC1 (10.10.0.1) | Server1 (10.3.0.30) | VPN IPSec | ✅ Permitido (3 saltos) | **EXITOSO** | 4/4 recibidos (0% loss) |
| **3** | PC3 (10.10.0.3) | Server0 (10.4.0.40) | VPN IPSec | ✅ Permitido (3 saltos) | **EXITOSO** | 4/4 recibidos (0% loss) |
| **4** | PC3 (10.10.0.3) | Server1 (10.3.0.30) | VPN IPSec | ✅ Permitido (3 saltos) | **EXITOSO** | 4/4 recibidos (0% loss) |
| **5** | PC2 (10.20.0.2) | Server0 (10.4.0.40) | Enrutado WAN | ✅ Permitido (4 saltos) | **EXITOSO** | 4/4 recibidos (0% loss) |
| **6** | PC2 (10.20.0.2) | Server1 (10.3.0.30) | Enrutado WAN | ✅ Permitido (4 saltos) | **EXITOSO** | 4/4 recibidos (0% loss) |
| **7** | PC4 (10.20.0.4) | Server0 (10.4.0.40) | Enrutado WAN | ✅ Permitido (4 saltos) | **EXITOSO** | 4/4 recibidos (0% loss) |
| **8** | PC4 (10.20.0.4) | Server1 (10.3.0.30) | Enrutado WAN | ✅ Permitido (4 saltos) | **EXITOSO** | 4/4 recibidos (0% loss) |
| **9** | LAN Admin (10.50.0.50) | Server0 (10.4.0.40) | Enrutado WAN | ✅ Permitido (4 saltos) | **EXITOSO** | 4/4 recibidos (0% loss) |
| **10** | LAN Admin (10.50.0.50) | Server1 (10.3.0.30) | Enrutado WAN | ✅ Permitido (4 saltos) | **EXITOSO** | 4/4 recibidos (0% loss) |
| **11** | PC1 (10.10.0.1) | PC3 (10.10.0.3) | Intra-VLAN 10 | ✅ Permitido | **EXITOSO** | 4/4 recibidos (0% loss) |
| **12** | PC2 (10.20.0.2) | PC4 (10.20.0.4) | Intra-VLAN 20 | ✅ Permitido | **EXITOSO** | 4/4 recibidos (0% loss) |
| **13** | PC1 (10.10.0.1) | PC2 (10.20.0.2) | Inter-VLAN | ❌ Bloqueado | **BLOQUEADO** | 0/4 recibidos (100% loss) |
| **14** | PC2 (10.20.0.2) | PC1 (10.10.0.1) | Inter-VLAN | ❌ Bloqueado | **BLOQUEADO** | 0/4 recibidos (100% loss) |
| **15** | PC1 (10.10.0.1) | LAN Admin (10.50.0.50) | Inter-VLAN | ❌ Bloqueado | **BLOQUEADO** | 0/4 recibidos (100% loss) |
| **16** | PC2 (10.20.0.2) | LAN Admin (10.50.0.50) | Inter-VLAN | ❌ Bloqueado | **BLOQUEADO** | 0/4 recibidos (100% loss) |
| **17** | LAN Admin (10.50.0.50) | PC1 (10.10.0.1) | Inter-VLAN | ❌ Bloqueado | **BLOQUEADO** | 0/4 recibidos (100% loss) |
| **18** | LAN Admin (10.50.0.50) | PC2 (10.20.0.2) | Inter-VLAN | ❌ Bloqueado | **BLOQUEADO** | 0/4 recibidos (100% loss) |

---

## 4. Evidencia de Trazado de Rutas (Tracert)

### A. Tráfico por el Túnel VPN (VLAN 10 → Server0 / Server1)
El tráfico de la VLAN 10 entra en el túnel IPSec en Router1 y sale descifrado en Router2. La nube pública del ISP queda **enmascarada**, produciendo exactamente **3 saltos**:

```text
C:\>tracert 10.4.0.40

Tracing route to 10.4.0.40 over a maximum of 30 hops: 

  1   0 ms      0 ms      0 ms      10.10.0.254  (Router1 - Gateway VLAN 10)
  2   13 ms     20 ms     15 ms     10.2.0.2     (Router2 - Extremo Túnel VPN)
  3   24 ms     48 ms     2 ms      10.4.0.40    (Server0)

Trace complete.
```

```text
C:\>tracert 10.3.0.30

Tracing route to 10.3.0.30 over a maximum of 30 hops: 

  1   0 ms      0 ms      0 ms      10.10.0.254  (Router1 - Gateway VLAN 10)
  2   26 ms     2 ms      1 ms      10.2.0.2     (Router2 - Extremo Túnel VPN)
  3   33 ms     4 ms      35 ms     10.3.0.30    (Server1)

Trace complete.
```

### B. Tráfico Fuera del Túnel VPN (VLAN 20 / LAN Admin → Servidores)
El tráfico de la VLAN 20 y de LAN Admin viaja por enrutamiento convencional EIGRP/Default a través del ISP sin encapsulamiento IPSec, produciendo exactamente **4 saltos**:

```text
C:\>tracert 10.4.0.40 (desde PC2)

Tracing route to 10.4.0.40 over a maximum of 30 hops: 

  1   1 ms      0 ms      0 ms      10.20.0.254  (Router1 - Gateway VLAN 20)
  2   10 ms     1 ms      0 ms      10.1.0.2     (ISP Router)
  3   0 ms      17 ms     14 ms     10.2.0.2     (Router2)
  4   1 ms      0 ms      14 ms     10.4.0.40    (Server0)

Trace complete.
```

```text
C:\>tracert 10.4.0.40 (desde LAN Admin)

Tracing route to 10.4.0.40 over a maximum of 30 hops: 

  1   0 ms      0 ms      0 ms      10.50.0.254  (Router1 - Gateway VLAN 1)
  2   9 ms      4 ms      5 ms      10.1.0.2     (ISP Router)
  3   1 ms      1 ms      13 ms     10.2.0.2     (Router2)
  4   12 ms     4 ms      13 ms     10.4.0.40    (Server0)

Trace complete.
```

---

## 5. Estado de Archivos y Entrega

- Archivo de salida: `C:\Users\Mateo\Desktop\UTN\Redes\Cisco\src\PKTs\TL4-K4773-Fernandez-Toledo.pkt`
- Todas las configuraciones (`running-config`) fueron consolidadas en `startup-config` (`write memory`) en **Router1**, **Router2**, **ISP**, **ACCESO 1**, **ACCESO 2** y **DISTRIBUCIÓN**.
- **Perfil de usuario para entrega:** "Grupo 26" (según indicación del docente para la ventana inicial de perfil/Guest en Cisco Packet Tracer).
