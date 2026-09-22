# Validación TL3 — TL3-K4773-Fernandez-Toledo.pkt

## Resumen Ejecutivo

El trabajo de laboratorio TL3 (**Configuración básica de Routers para el funcionamiento de IP en Capa 3**) se encuentra **100% IMPLEMENTADO Y VALIDADO** sobre el simulador Packet Tracer hasta el **punto H inclusive**, cumpliendo con la consigna oficial y las instrucciones dadas por el docente en clase (el punto I de ACLs no se implementa por indicación explícita del profesor).

Todas las pruebas de conectividad punto a punto e inter-sucursal arrojan **0% de pérdida de paquetes**.

---

## 1. Topología y Cableado

- **Casa Central (Buenos Aires)**:
  - `Switch2` (Cisco 2950-24) actúa como switch central de distribución.
  - `Local1` Fa0/0 conectado a `Switch2` Fa0/1 con cable directo (straight).
  - `Local2` Fa0/0 conectado a `Switch2` Fa0/2 con cable directo (straight).
  - `Local3` Fa0/0 conectado a `Switch2` Fa0/3 con cable directo (straight).
  - `Admin0` (Laptop Admin) conectada a `Switch2` Fa0/24 con cable directo (straight).

- **LANs Sucursales**:
  - `Admin1` (PC Rosario) conectada a `Remoto1` Fa0/0 con cable cruzado (crossover).
  - `Admin2` (PC Mendoza) conectada a `Remoto2` Fa0/0 con cable cruzado (crossover).
  - `Admin3` (PC Tucumán) conectada a `Remoto3` Fa0/0 con cable cruzado (crossover).
  > **Nota conceptual:** Se usa cable cruzado entre PC y Router ya que ambos son equipos terminales (MDI) sin capacidad auto-MDIX activada en las interfaces FastEthernet de los 1841.

- **Enlaces WAN Seriales**:
  - `Local1` Serial0/1/1 <--> `Remoto1` Serial0/1/1 (cable serial DCE-DTE).
  - `Local2` Serial0/1/1 <--> `Remoto2` Serial0/1/1 (cable serial DCE-DTE).
  - `Local3` Serial0/1/1 <--> `Remoto3` Serial0/1/1 (cable serial DCE-DTE).
  > **Extremo DCE:** Los routers locales (`Local1`, `Local2`, `Local3`) tienen el extremo DCE con `clock rate 2000000`, ya que tienen la IP más alta de cada enlace /30 según la regla del enunciado (pág. 4, punto b.2).

---

## 2. Direccionamiento IP Implementado

### Casa Central – LAN `192.168.2.0/24`
| Dispositivo | Interfaz | Dirección IP | Máscara de Red | Default Gateway |
|---|---|---|---|---|
| Local1 (BsAs1) | Fa0/0 | 192.168.2.21 | 255.255.255.0 | N/C |
| Local2 (BsAs2) | Fa0/0 | 192.168.2.22 | 255.255.255.0 | N/C |
| Local3 (BsAs3) | Fa0/0 | 192.168.2.23 | 255.255.255.0 | N/C |
| Admin0 (Laptop) | FastEthernet0 | 192.168.2.254 | 255.255.255.0 | 192.168.2.22 |

### Sucursal 1 (Rosario)
| Dispositivo / Enlace | Interfaz | Dirección IP | Máscara de Red | Default Gateway |
|---|---|---|---|---|
| PC Admin1 | FastEthernet0 | 192.168.1.33 | 255.255.255.224 (/27) | 192.168.1.34 |
| Remoto1 (LAN) | Fa0/0 | 192.168.1.34 | 255.255.255.224 (/27) | N/C |
| Remoto1 (WAN - DTE) | Serial0/1/1 | 192.168.1.133 | 255.255.255.252 (/30) | N/C |
| Local1 (WAN - DCE) | Serial0/1/1 | 192.168.1.134 | 255.255.255.252 (/30) | N/C |

### Sucursal 2 (Mendoza)
| Dispositivo / Enlace | Interfaz | Dirección IP | Máscara de Red | Default Gateway |
|---|---|---|---|---|
| PC Admin2 | FastEthernet0 | 192.168.1.65 | 255.255.255.224 (/27) | 192.168.1.66 |
| Remoto2 (LAN) | Fa0/0 | 192.168.1.66 | 255.255.255.224 (/27) | N/C |
| Remoto2 (WAN - DTE) | Serial0/1/1 | 192.168.1.169 | 255.255.255.252 (/30) | N/C |
| Local2 (WAN - DCE) | Serial0/1/1 | 192.168.1.170 | 255.255.255.252 (/30) | N/C |

### Sucursal 3 (Tucumán - Integración VLSM)
La subred 3 (`192.168.1.96/27`) se divide con VLSM:
- Se extrae el primer bloque `/30` para el enlace WAN serial: `192.168.1.96/30` (IPs usables: `.97` y `.98`).
- El rango superior del bloque `/27` se utiliza para la LAN de hosts: PC Admin3 (`.100`) y Remoto3 Fa0/0 (`.101`).

| Dispositivo / Enlace | Interfaz | Dirección IP | Máscara de Red | Default Gateway |
|---|---|---|---|---|
| PC Admin3 | FastEthernet0 | 192.168.1.100 | 255.255.255.224 (/27) | 192.168.1.101 |
| Remoto3 (LAN) | Fa0/0 | 192.168.1.101 | 255.255.255.224 (/27) | N/C |
| Remoto3 (WAN - DTE) | Serial0/1/1 | 192.168.1.97 | 255.255.255.252 (/30) | N/C |
| Local3 (WAN - DCE) | Serial0/1/1 | 192.168.1.98 | 255.255.255.252 (/30) | N/C |

---

## 3. Enrutamiento Dinámico RIPv2

- **Routers Remotos (`Remoto1`, `Remoto2`, `Remoto3`)**:
  ```ios
  router rip
   version 2
   network 192.168.1.0
   passive-interface FastEthernet0/0
  ```
  - **Passive-interface:** Se aplica sobre `FastEthernet0/0` para evitar enviar broadcasts/multicasts de RIP hacia las PCs locales por motivos de seguridad y ancho de banda.

- **Routers Locales (`Local1`, `Local2`, `Local3`)**:
  ```ios
  router rip
   version 2
   network 192.168.1.0
   network 192.168.2.0
  ```
  - **Sin passive-interface en Fa0/0:** No se pasiviza `FastEthernet0/0` en los locales porque necesitan intercambiar las actualizaciones de tablas RIP a través del switch central. Si se pasivizara, las sucursales quedarían incomunicadas entre sí.

---

## 4. Hardening y Seguridad Cisco IOS

Aplicado en los 6 routers según la consigna:
- `enable secret utn`
- `service password-encryption`
- `banner motd #Acceso no autorizado prohibido#`
- `ip domain-name tl3.com`
- Generación de claves RSA 1024 bits (`crypto key generate rsa modulus 1024`)
- `ip ssh version 2`
- Usuario administrador: `username redes privilege 15 password cisco`
- Línea VTY 0: `transport input ssh` y `login local`
- Líneas VTY 1 a 4: `transport input none`
- Encapsulación WAN: `encapsulation ppp` en todas las interfaces seriales.

---

## 5. Resultados de Validación Real en Packet Tracer

### Verificación de Topología y Enlaces
- `pt_health_check`: **100% Saludable** (0 enlaces caídos, 0 IPs duplicadas, 10 enlaces operativos).

### Pruebas de Conectividad (Ping Real)
| Origen | Destino | IP Destino | Resultado | Pérdida |
|---|---|---|---|---|
| Admin1 | Gateway Remoto1 | 192.168.1.34 | CONECTIVIDAD OK | 0% (4/4) |
| Admin2 | Gateway Remoto2 | 192.168.1.66 | CONECTIVIDAD OK | 0% (4/4) |
| Admin3 | Gateway Remoto3 | 192.168.1.101 | CONECTIVIDAD OK | 0% (4/4) |
| Admin0 | Gateway Local2 | 192.168.2.22 | CONECTIVIDAD OK | 0% (4/4) |
| Local1 | Remoto1 Serial | 192.168.1.133 | CONECTIVIDAD OK | 0% (5/5) |
| Local2 | Remoto2 Serial | 192.168.1.169 | CONECTIVIDAD OK | 0% (5/5) |
| Local3 | Remoto3 Serial | 192.168.1.97 | CONECTIVIDAD OK | 0% (5/5) |
| **Admin1 (Rosario)** | **Admin2 (Mendoza)** | **192.168.1.65** | **CONECTIVIDAD OK** | **0% (4/4)** |
| **Admin1 (Rosario)** | **Admin3 (Tucumán)** | **192.168.1.100** | **CONECTIVIDAD OK** | **0% (4/4)** |
| **Admin2 (Mendoza)** | **Admin3 (Tucumán)** | **192.168.1.100** | **CONECTIVIDAD OK** | **0% (4/4)** |
| **Admin0 (Laptop)** | **Admin1 (Rosario)** | **192.168.1.33** | **CONECTIVIDAD OK** | **0% (4/4)** |
| **Admin0 (Laptop)** | **Admin2 (Mendoza)** | **192.168.1.65** | **CONECTIVIDAD OK** | **0% (4/4)** |
| **Admin0 (Laptop)** | **Admin3 (Tucumán)** | **192.168.1.100** | **CONECTIVIDAD OK** | **0% (4/4)** |

---

## 6. Archivo Final Generado
- **Ruta:** `c:/Users/Mateo/Desktop/UTN/Redes/Cisco/src/PKTs/TL3-K4773-Fernandez-Toledo.pkt`
- **Tamaño:** 66,480 bytes
- **Estado:** Listo para entregar.
