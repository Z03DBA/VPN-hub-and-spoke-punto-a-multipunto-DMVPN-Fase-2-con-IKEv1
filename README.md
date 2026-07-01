
# Laboratorio de Conectividad Segura: Arquitectura DMVPN Fase 2 (IKEv1)

Este repositorio contiene las configuraciones nativas y la documentación técnica para el despliegue de una red privada virtual multipunto dinámica (**DMVPN Fase 2**) utilizando **IKEv1 (ISAKMP)** para la protección criptográfica y **EIGRP** como protocolo de enrutamiento dinámico interno.

## 🗺️ Topología y Mapeo Lógico

El escenario está montado sobre un entorno virtualizado utilizando nodos de Cisco operando bajo las siguientes interconexiones de red:

```text
                      [ R1 (HUB / Central) ]
                                |
                     Tunnel0: 172.16.0.1/24
                                |
             +------------------+------------------+
             |                                     |
    [ R2 (SPOKE 1) ]                      [ R3 (SPOKE 2) ]
    Tunnel0: 172.16.0.2                   Tunnel0: 172.16.0.3
    LAN: 192.8.39.16/28                   LAN: 192.8.39.32/28
             |                                     |
          [ PC1 ]                               [ PC2 ]

```

### Direccionamiento Físico de Interfaces

* **R1 (Hub):** `Gi1/0` (Hacia R2 - `.1`), `Gi2/0` (Hacia R3 - `.5`).
* **R2 (Spoke 1):** `Gi1/0` (WAN - `.2`), `Gi2/0` (LAN - `.17`).
* **R3 (Spoke 2):** `Gi1/0` (WAN - `.6`), `Gi2/0` (LAN - `.33`).

## 🛠️ Tecnologías Utilizadas

* **Next Hop Resolution Protocol (NHRP):** Base de resolución de direcciones NBMA a IPs lógicas.
* **Multipoint GRE (mGRE):** Una sola interfaz de túnel para múltiples destinos.
* **IPsec (Modo Transporte):** Minimiza el overhead encapsulando directamente el payload GRE.
* **EIGRP AS 1:** Intercambio automático de prefijos con deshabilitación de *Next-Hop-Self* para flujo directo Spoke-to-Spoke.

## 🚀 Instrucciones de Despliegue

1. Inicializar la infraestructura de red en el emulador.
2. Cargar los scripts provistos en el directorio `/config` de manera secuencial (Hub -> Spokes).
3. Validar la adyacencia de enrutamiento mediante `show ip eigrp neighbors`.
4. Ejecutar tráfico ICMP continuo desde las VPCS para disparar la Phase 2 dinámica.


## 📑 2. Contenido del Documento Técnico Profesional (Para tu PDF)
*Copia este texto y utilízalo para armar el cuerpo de tu reporte final.*

---

# INFORME DE INGENIERÍA DE REDES: DESPLIEGUE DE DMVPN FASE 2
**Preparado por:** Zoe Daniela Bobonagua Acevedo  

---

## 1. OBJETIVO TÉCNICO DE LA VPN
El propósito fundamental de este proyecto es implementar una arquitectura **DMVPN (Dynamic Multipoint VPN) Fase 2** sobre un entorno empresarial distribuido. 

A diferencia de los túneles tradicionales punto a punto (VTI) que saturan el procesamiento del router central y exigen configuraciones masivas a medida que la red crece, DMVPN permite:
* **Escalabilidad óptima:** El nodo central (Hub) requiere una única interfaz de túnel (`Tunnel0`) sin importar cuántas sucursales se agreguen.
* **Optimización de caminos de datos (Spoke-to-Spoke):** Al implementar la Fase 2, las sucursales pueden establecer túneles IPsec dinámicos directos entre ellas para intercambiar datos de voz, video o archivos pesados, reduciendo la latencia de red a la mitad al no tener que enviar el tráfico obligatoriamente a través del Hub.
* **Seguridad criptográfica integrada:** Toda la información que transita por la red pública (ISP) es blindada mediante políticas de cifrado robustas con IKEv1/IPsec.

---

## 2. PARÁMETROS TÉCNICOS ADOPTADOS

Para cumplir con las normativas corporativas de seguridad y rendimiento, se definieron y aplicaron los siguientes parámetros de configuración unificados:

### Fase 1: ISAKMP (IKEv1)
* **Algoritmo de Cifrado:** AES (Advanced Encryption Standard) con clave de 256 bits.
* **Función de Hash:** SHA-256 (Algoritmo de Hash Seguro).
* **Método de Autenticación:** Pre-shared Key (Clave compartida: `ClaveDMVPN123`).
* **Grupo Diffie-Hellman:** Grupo 14 (Seguridad de clave de 2048 bits para prevenir ataques de fuerza bruta).

### Fase 2: IPsec y mGRE
* **Modo de Encapsulamiento:** `Transport Mode` *(Mandatorio para DMVPN; ahorra 20 bytes por paquete al omitir la cabecera IP externa duplicada, ya que GRE proporciona el direccionamiento IP de los extremos).*
* **Transform-Set:** ESP-AES-256 y ESP-SHA256-HMAC.
* **Red del Túnel Lógico:** Subred `172.16.0.0/24`.

---

## 3. ESQUEMA DE DIRECCIONAMIENTO E INTERFACES

La topología implementa una segmentación estricta dividida en el segmento de transporte (WAN), el direccionamiento interno criptográfico (VPN) y las redes de usuarios finales (LAN):

| Dispositivo | Interfaz | Propósito / Conexión | Dirección IP / Máscara | Gateway |
| :--- | :--- | :--- | :--- | :--- |
| **R1 (Hub)** | `Gi1/0` <br> `Gi2/0` <br> `Tunnel0` | WAN (Enlace R2)<br>WAN (Enlace R3)<br>Interfaz Multipunto mGRE | `192.8.39.1 /30`<br>`192.8.39.5 /30`<br>`172.16.0.1 /24` | N/A<br>N/A<br>N/A |
| **R2 (Spoke 1)** | `Gi1/0` <br> `Gi2/0` <br> `Tunnel0` | WAN (Hacia el ISP)<br>LAN (Usuarios PC1)<br>Interfaz Multipunto mGRE | `192.8.39.2 /30`<br>`192.8.39.17 /28`<br>`172.16.0.2 /24` | `192.8.39.1`<br>N/A<br>`172.16.0.1` |
| **R3 (Spoke 2)** | `Gi1/0` <br> `Gi2/0` <br> `Tunnel0` | WAN (Hacia el ISP)<br>LAN (Usuarios PC2)<br>Interfaz Multipunto mGRE | `192.8.39.6 /30`<br>`192.8.39.33 /28`<br>`172.16.0.3 /24` | `192.8.39.5`<br>N/A<br>`172.16.0.1` |

---

## 4. AUDITORÍA, CAPTURAS REQUERIDAS Y JUSTIFICACIÓN TÉCNICA
*(En esta sección debes adjuntar las capturas de tu terminal basándote en la siguiente guía explicativa)*

### Punto A: Validación del Registro NHRP en el Hub
* **Comando a capturar:** `R1# show ip nhrp` o `R1# show dmvpn`
* **Explicación técnica:** Esta captura demuestra la correcta operación del protocolo NHRP (Next Hop Resolution Protocol). El Hub actúa como el servidor de resolución de nombres (NHS), y se evidencia cómo los Spokes registran dinámicamente sus direcciones IP públicas cambiantes vinculándolas a su IP del túnel fija. El estado debe reflejarse como **`up`** con atributo **`D`** (Dinámico).

### Punto B: Tabla de Enrutamiento EIGRP en los Spokes
* **Comando a capturar:** `R2# show ip route eigrp`
* **Explicación técnica:** Muestra que las subredes internas (`192.8.39.16/28` y `192.8.39.32/28`) se están propagando de manera automatizada a través del túnel. Se debe comprobar visualmente que la LAN remota apunta al túnel con el siguiente salto del Spoke vecino y **no** con la IP del Hub. Esto confirma que los comandos `no ip next-hop-self` introducidos en el Hub están funcionando, permitiendo la Fase 2.

### Punto C: Evidencia del Túnel Spoke-to-Spoke Dinámico Directo
* **Comando a capturar:** `R2# show dmvpn` *(mientras se ejecuta un `ping` desde PC1 a PC2)*
* **Explicación técnica:** Esta es la captura crucial del proyecto. Inicialmente, R2 solo tiene el túnel hacia el Hub (`172.16.0.1`). Al iniciar el ping hacia PC2, NHRP intercepta la petición, le pregunta al Hub la IP pública de R3, y R2 levanta un túnel directo hacia R3. La captura mostrará una nueva línea hacia la IP `172.16.0.3` con el atributo **`D`**, demostrando el éxito de la arquitectura multipunto.

### Punto D: Estado de las Asociaciones de Seguridad (Criptografía)
* **Comando a capturar:** `R2# show crypto isakmp sa` y `R2# show crypto ipsec sa | include pkts`
* **Explicación técnica:** Certifica la seguridad perimetral. El primer comando debe devolver de manera inequívoca el estado **`QM_IDLE`**, indicando que la Fase 1 se negoció sin discrepancias de políticas. El segundo comando demuestra el incremento constante de paquetes encapsulados y desencapsulados, garantizando que el tráfico no va en texto plano.


**Desarrollado por Zoe Daniela Bobonagua Acevedo** 🚀
