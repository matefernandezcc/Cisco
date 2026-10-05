A continuación se presenta el listado consolidado de todos los temas expuestos en las transcripciones de las **6 clases teóricas** del profesor Ing. Víctor Raúl Alsina, detallando las pautas fijadas para el **Primer Parcial**:

---

### 📅 Definición y Alcance del Primer Parcial
* **Fecha y modalidad confirmada:** El examen es presencial en la sede Medrano.
* **Alcance explícito:** En la Clase 6, el profesor Alsina confirmó expresamente que el examen abarca **todo lo visto hasta esa clase inclusive** (Unidades 1 a 5/8 del programa académico).

---

### 📋 Listado Temático por Clase y Unidad

#### 1. Clase 1: Conceptos Básicos y Arquitectura de Redes
* **Fundamentos y evolución de redes:** Concepto de red, recursos compartidos y convergencia de voz, video y datos en redes IP únicas e integradas (ISDN). Evolución histórica desde *mainframes* con terminales hasta arquitecturas LAN/WAN, telefonía fija/celular y tecnologías de acceso por fibra óptica (FTTH, FTTN, FTTB) e híbridas (HFC).
* **Clasificación geográfica:** Redes LAN, MAN, WAN y GAN.
* **Modos de transferencia:** Redes orientadas a conexión (circuitos virtuales, entrega ordenada y control de errores) vs. redes no orientadas a conexión / datagramas (*best effort* o mejor esfuerzo, enrutamiento independiente de paquetes).
* **Topologías de red:** Topologías físicas y lógicas en estrella, malla, anillo y bus.
* **Mecanismos y clasificaciones de protocolos:**
  * **Sistemas con sondeo y selección (*Polling / Selecting*):** Control de flujo y errores mediante ARQ (*Stop-and-Wait*, *Sliding Window* o ventana deslizante).
  * **Sistemas sin sondeo:** Control en banda (Xon/Xoff), control fuera de banda (RTS/CTS) y multiplexación por división de tiempo (TDMA).
* **Acceso al medio y colisiones:** Algoritmo de postergación binaria exponencial (*Binary Exponential Backoff*) empleado en CSMA/CD.

---

#### 2. Clase 2: Capa de Enlace, Conmutación LAN y VLANs
* **Direccionamiento MAC (Capa 2):** Estructura de la dirección física de 48 bits, identificador OUI del fabricante (primeros 24 bits) y número de serie (24 bits). Dirección de difusión o *broadcast* (`FF-FF-FF-FF-FF-FF`).
* **Dispositivos de capa 1 y 2:**
  * **Hubs:** Repetidores de difusión que comparten el ancho de banda entre todos los puertos.
  * **Switches:** Conmutación de tramas, aislamiento de dominios de colisión por puerto, mantenimiento de dominios de broadcast, aprendizaje dinámico de tablas de direcciones MAC y proceso de inundación (*flooding*).
* **Subcapas IEEE 802:** Subcapa MAC (ensamblado/desensamblado de tramas, detección de errores) y Subcapa LLC (IEEE 802.2).
* **Infraestructura y cableado:** Normas de cableado estructurado (EIA/TIA 568A y 568B) y dimensionamiento físico en armarios/racks (Unidades de rack \\(1\text{U} = 1,75''\\)).
* **Segmentación con VLANs y enlaces troncales:** Creación de LANs virtuales para seguridad y organización por áreas, y operación de tramas etiquetadas mediante enlaces troncales (*Trunks*, norma IEEE 802.1Q).
* **Spanning Tree Protocol (STP / IEEE 802.1D):** Algoritmo de prevención de bucles en capa 2; selección del Puente Raíz (*Root Bridge*), Puerto Raíz (*Root Port*), Puerto Designado y Puerto Bloqueado.
* **Redes de alta velocidad en anillo:** Principios del protocolo FDDI (anillo doble de fibra óptica).

---

#### 3. Clase 3: Arquitectura TCP/IP, Datagrama IP, Capa de Transporte y DHCP
* **Modelos de referencia:** Arquitectura de capas TCP/IP frente al Modelo OSI.
* **Capa de Red (IP / IPv4):**
  * **Características del protocolo IP:** Servicio no orientado a conexión, entrega no confiable de mejor esfuerzo (*best effort*).
  * **Datagrama IP y fragmentación:** Estructura del encabezado IPv4, Unidad Máxima de Transferencia (MTU) y mecanismo de fragmentación/reensamblado de datagramas (cálculo de desplazamientos/*offsets*, *flags* y longitud).
  * **Direccionamiento IPv4:** Clases de direcciones IP (A, B, C, D y E), máscaras de subred por defecto, técnicas de división en subredes (*Subnetting*), direccionamiento sin clase (CIDR) y máscaras de longitud variable (VLSM).
* **Capa de Transporte:**
  * **Protocolo TCP:** Orientado a conexión, confiabilidad, control de flujo por ventana deslizante (*sliding window*), gestión de acuses de recibo (ACK) y retransmisión por temporizadores (*timeout*), abstracción de *Sockets* (combinación de dirección IP y puerto).
  * **Protocolo UDP:** No orientado a conexión, sin acuse de recibo, adecuado para aplicaciones con bajo *overhead*.
* **Gestión dinámica de red:** Protocolo DHCP (*Dynamic Host Configuration Protocol*) para la asignación automatizada de parámetros IP y administración centralizada.

---

#### 4. Clase 4: Redes Inalámbricas (WLAN) y Tecnologías de Transmisión
* **Redes WLAN (IEEE 802.11):** Topologías de celdas únicas/múltiples, enlaces punto a punto, acceso nómade y redes *Ad-Hoc*.
* **Control de acceso inalámbrico:** Protocolo CSMA/CA (*Collision Avoidance*) y uso de tramas de reserva de canal RTS/CTS.
* **Técnicas de transmisión y modulación:** Espectro disperso por secuencia directa (DSSS), por salto de frecuencia (FHSS) y OFDM.
* **Estándares y bandas Wi-Fi:** Evolución de estándares (802.11a/b/g/n/ac/ax - Wi-Fi 6/6E) y operación en bandas de 2,4 GHz, 5 GHz y 6 GHz.
* **Seguridad inalámbrica:** Autenticación, técnicas de ocultamiento de SSID y protocolos de cifrado (WEP, WPA/WPA2, AES).
* **Telecomunicaciones móviles y WAN inalámbricas:** Arquitectura de celdas y radiobases, tecnología 4G/5G (baja latencia) y enlaces por satélites de órbita baja (LEO / Starlink) y geostacionarios (GEO).

---

#### 5. Clase 5: Protocolos Auxiliares TCP/IP y Principios de Enrutamiento
* **Traductores de direcciones (NAT / PAT):**
  * Concepto de reutilización de direcciones IP públicas mediante IP privadas.
  * Mapeo de puertos TCP/UDP en PAT y limitaciones de traducción en paquetes encapsulados (como túneles VPN).
* **Protocolos de soporte:** ICMP (mensajes de diagnóstico y comandos *ping* / *tracert*), ARP (resolución de dirección IP a MAC) y DNS (resolución de nombres de dominio).
* **Enrutamiento en redes WAN:**
  * **Funciones fundamentales del router:** Determinación de trayectorias, selección de la mejor ruta mediante métricas y reenvío de paquetes.
  * **Enrutamiento Estático vs. Enrutamiento Dinámico:** Comparación en complejidad, seguridad y adaptabilidad ante fallas.
  * **Clasificación de protocolos de enrutamiento:**
    * **IGP (*Interior Gateway Protocol*):** Protocolos intradominio dentro de un Sistema Autónomo (RIP, IGRP, EIGRP, OSPF).
    * **EGP (*Exterior Gateway Protocol*):** Protocolos interdominio entre Sistemas Autónomos (BGP).
  * **Conceptos clave:** Sistema Autónomo (AS), métrica (conteo de saltos, ancho de banda) y tablas de enrutamiento.
* **Arquitectura interna del Router:** Procesador/CPU, memoria RAM (tablas de enrutamiento activas, volátil), NVRAM, memoria Flash y ROM.

---

#### 6. Clase 6: Calidad de Servicio (QoS) e Introducción a redes IP/MPLS
* **Calidad de Servicio (QoS):** Parámetros de degradación de tráfico en redes IP (latencia/delay, variación de retardo o *jitter*, congestión de *buffers* y pérdida de paquetes).
* **Arquitectura IP/MPLS (*Multi-Protocol Label Switching*):**
  * Principio de conmutación mediante etiquetas (*Labels*) de longitud fija procesadas por hardware en lugar de inspeccionar el encabezado IP completo en cada salto.
  * **Formato del encabezado MPLS:** Campo de Etiqueta (20 bits), bits experimentales QoS (3 bits), indicador de pila *Bottom of Stack* S (1 bit) y *Time to Live* / TTL (8 bits).
  * **Componentes del dominio MPLS:** Routers LSR (*Label Switching Router*), routers de borde LER / PE (*Provider Edge*) y caminos conmutados LSP (*Label Switched Path*).
  * **Mecanismos:** Clases de equivalencia de reenvío (FEC), distribución de etiquetas (protocolo LDP) y apilamiento de etiquetas (*Label Stacking / LIFO*).
  * Balanceo de carga e integración de MPLS sobre infraestructuras subyacentes Ethernet, ATM y Frame Relay.