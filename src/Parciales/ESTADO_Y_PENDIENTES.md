# Estado y pendientes – Parcial Redes 1234 (Regional A)

## DIAGNÓSTICO DEL 83% Y CORRECCIONES APLICADAS

A partir de las capturas del **Assessment Items / Activity Wizard** y las indicaciones del profesor en clase, se identificaron y corrigieron todos los puntos que restaban puntaje:

---

### 1. Enlaces Troncales (Trunk VLANs) en Switches (Corregido ✅)
* **Problema en la captura:** Los puertos troncales marcaban `X Trunk VLANs 1 - 1005 Incorrect`.
* **Causa:** Al haber aplicado `switchport trunk allowed vlan 10,20`, se restringieron las demás VLANs. El Activity Wizard de la cátedra exige que los enlaces troncales operen en el modo predeterminado de Cisco permitiendo `1 - 1005` (para que también pueda fluir la VLAN 30).
* **Solución aplicada:** Se ejecutó `no switchport trunk allowed vlan` en:
  - `ACCESO 1 A`: `FastEthernet0/3`
  - `ACCESO 2 A`: `FastEthernet0/3`
  - `DISTRIBUCION A`: `FastEthernet0/1`, `FastEthernet1/1` y `GigabitEthernet6/1`.

---

### 2. Startup-Config en Switches (Corregido ✅)
* **Problema en la captura:** `ACCESO 1 A` y `DISTRIBUCION A` marcaban `X Startup Config Incorrect`.
* **Causa:** Faltaba persistir la configuración activa a la NVRAM (`startup-config`).
* **Solución aplicada:** Se ejecutó `copy running-config startup-config` (`write memory`) en todos los switches y routers.

---

### 3. Subinterfaz VLAN 30 y Default Gateways (Corregido ✅)
* **Problema en la captura:** `Admin LAN` y `Admin WLAN` marcaban `X Default Gateway Incorrect`.
* **Causa e indicación del profesor:** El profesor indicó que la VLAN 30 debe configurarse como se vio en clase:
  - Debe tener subinterfaz en el router: `FastEthernet0/0.30` con IP `192.168.30.254 255.255.255.0` (802.1Q vlan 30).
  - Los hosts de la VLAN 30 (`Admin LAN`, `Admin WLAN` y el pool DHCP del `Wireless Router A`) deben tener configurado como Default Gateway `192.168.30.254`.
* **Aislamiento exigido:**
  - En `RouterA`, se aplicó la ACL 103 en `interface fa0/0.30 in` permitiendo tráfico dentro de la propia subred (`permit ip 192.168.30.0 0.0.0.255 192.168.30.0 0.0.0.255`) y denegando cualquier salida externa o hacia otras VLANs (`deny ip 192.168.30.0 0.0.0.255 any`).
  - No se anuncia en EIGRP.
  - **Prueba realizada:** Admin LAN llega a su gateway `192.168.30.254` (0% loss), pero no tiene salida hacia VLAN 10, VLAN 20 ni hacia la WAN (100% bloqueado).

---

### 4. Numeración de ACLs en RouterA (Corregido ✅)
* **Problema en la captura:** En `Router A -> ACL` marcaba `X 101 Incorrect` y `X 102 Incorrect`.
* **Causa:** El Activity Wizard busca exactamente las ACLs extendidas numeradas como `101` (para VLAN 10) y `102` (para VLAN 20). Anteriormente estaban numeradas como 110 y 120.
* **Solución aplicada:**
  - `access-list 101` para VLAN 10 aplicada en `Fa0/0.10 in`.
  - `access-list 102` para VLAN 20 aplicada en `Fa0/0.20 in`.

---

### 5. Enable Secret en RouterA (Corregido ✅)
* **Problema en la captura:** `Router A -> Enable Secret: X Incorrect`.
* **Solución aplicada:** Se configuró `enable secret redes` en `RouterA`.

---

### 6. EIGRP en RouterA (Corregido ✅)
* **Problema en la captura:** `EIGRP -> Networks: X Route0 Incorrect`.
* **Causa:** Se había declarado `network 50.100.0.0 0.0.0.3`. La solución modelo de Cisco/cátedra utiliza la declaración classful para la red de acceso: `network 50.0.0.0`.
* **Solución aplicada:** Se configuró `network 50.0.0.0` en `router eigrp 99`.

---

### 7. Cableado en Wireless Router A (Verificado ✅)
* `Admin WLAN` conectado a `Wireless Router A (Ethernet 1)` con cable directo (straight).
* `DISTRIBUCION A (Fa3/1)` conectado a `Wireless Router A (Ethernet 2)` con cable cruzado (cross).

---

## PRUEBAS REALIZADAS Y RESULTADOS
* **HTTPS Externo:** PC1 -> `192.168.111.111` -> **0% pérdida (OK)**.
* **FTP IPsec:** PC2 -> `192.168.100.100` -> **0% pérdida (Cifrado en túnel QM_IDLE)**.
* **Segmentación:** PC1 -> PC2 -> **100% pérdida (Bloqueado por ACL 101)**.
* **Segmentación:** PC2 -> PC1 -> **100% pérdida (Bloqueado por ACL 102)**.
* **VLAN 30 Gateway:** Admin LAN -> `192.168.30.254` -> **0% pérdida (OK)**.
* **VLAN 30 Aislamiento:** Admin LAN -> `192.168.10.1` -> **100% pérdida (Bloqueado por ACL 103)**.
* **VLAN 30 Aislamiento WAN:** Admin LAN -> `192.168.111.111` -> **100% pérdida (Bloqueado por ACL 103)**.
