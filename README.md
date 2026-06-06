Ataque-DoS-mediante-CDP# Ataque DoS mediante CDP

![Python](https://img.shields.io/badge/Python-3.x-blue)
![Platform](https://img.shields.io/badge/Platform-Kali%20Linux-red)
![Environment](https://img.shields.io/badge/Environment-GNS3%20%7C%20IOSvL2-orange)
![Use](https://img.shields.io/badge/Use-Controlled%20Lab-yellow)
![Topic](https://img.shields.io/badge/Topic-Network%20Security-purple)

## Información del proyecto

Este repositorio documenta y demuestra un ataque de **Denegación de Servicio (DoS) mediante el protocolo CDP (Cisco Discovery Protocol)** en un entorno controlado de GNS3. El objetivo es saturar la tabla de vecinos CDP de un switch Cisco IOSvL2 enviando anuncios CDP falsos de manera masiva, causando degradación en el rendimiento del dispositivo.

La práctica también incluye contramedidas para mitigar el ataque, deshabilitando CDP globalmente o por interfaz.

| Dato                  | Información                                     |
| --------------------- | ----------------------------------------------- |
| Autor                 | Darwing                                         |
| Matrícula             | 2024-2690                                       |
| Docente               | Jonathan Rondon                                 |
| Repositorio           | https://github.com/TuUsuario/CDP-DoS-Attack     |
| Video demostrativo    | https://youtu.be/JLB376PKTwk       |
| Documentación técnica | docs/documentacion-tecnica-profesional.pdf      |
| Red de laboratorio    | 20.24.26.0/24                                   |

---

## Aviso de uso responsable

Este proyecto fue desarrollado únicamente con fines educativos, académicos y de laboratorio controlado.

Los scripts incluidos deben ejecutarse solamente en entornos propios o autorizados como GNS3, EVE-NG o PNETLab. No debe utilizarse en redes públicas, empresariales o de terceros sin autorización explícita.

---

## Objetivo del laboratorio

Demostrar cómo un atacante conectado a un puerto de acceso puede abusar del protocolo **CDP** para generar cientos de anuncios falsos hacia un switch Cisco, provocando:

- Saturación de la tabla de vecinos CDP.
- Incremento en el uso de CPU del switch.
- Consumo excesivo de memoria del dispositivo.
- Posible degradación del rendimiento general de la red.

---

## Objetivo del script

El script `cdp_dos.py` genera y envía múltiples tramas CDP falsificadas desde Kali Linux hacia el switch. Cada trama incluye un Device ID, Port ID y Platform diferentes, haciendo creer al switch que existen cientos de vecinos distintos conectados.

---

## Archivos del repositorio

| Archivo                                        | Descripción                                                      |
| ---------------------------------------------- | ---------------------------------------------------------------- |
| `cdp_dos.py`                                   | Script principal para ejecutar el ataque DoS mediante CDP.       |
| `mitigacion-cdp.md`                            | Documento con las contramedidas aplicadas contra el ataque CDP.  |
| `README.md`                                    | Guía principal del laboratorio.                                  |
| `docs/documentacion-tecnica-profesional.pdf`   | Documentación técnica profesional detallada del laboratorio.     |
| `images/`                                      | Capturas de pantalla de evidencia del laboratorio.               |

---

## Topología del laboratorio

La topología está compuesta por una máquina Kali Linux atacante, un switch Cisco IOSvL2 y una PC víctima (VPCS).

```
[Kali Linux]──eth0──Gi0/0──[SW-1]──Gi0/1──[PC1]
                              │
                           Gi0/2
                              │
                            [R1]
```

| Dispositivo | Rol               | Interfaz | Dirección IP    | Descripción                   |
| ----------- | ----------------- | -------- | --------------- | ----------------------------- |
| R1          | Gateway           | F0/0     | 20.24.26.91/24  | Router principal de la red    |
| SW-1        | Switch capa 2     | Gi0/0~2  | N/A             | Switch objetivo del ataque    |
| Kali Linux  | Atacante          | eth0     | 20.24.26.90/24  | Ejecuta el flood de CDP       |
| PC1         | Víctima / Cliente | e0       | 20.24.26.92/24  | Equipo en la misma red        |

---

## Requisitos previos

- GNS3 o entorno de virtualización equivalente.
- Switch Cisco IOSvL2 con CDP habilitado.
- Kali Linux con Python 3 instalado.
- Librería Scapy instalada (`pip install scapy`).
- Permisos de superusuario (`sudo`).
- Conectividad de capa 2 entre Kali y el switch.

Instalar dependencias:

```bash
pip install scapy
```

---

## Preparación del laboratorio

### 1. Clonar el repositorio

```bash
git clone https://github.com/TuUsuario/CDP-DoS-Attack.git
cd CDP-DoS-Attack
```

### 2. Verificar la interfaz de Kali

```bash
ip -br addr
```

### 3. Verificar CDP en el switch antes del ataque

```
SW-1# show cdp neighbors
SW-1# show cdp traffic
SW-1# show processes cpu sorted
```

---

## Parámetros del script

| Parámetro          | Descripción                                              | Ejemplo          |
| ------------------ | -------------------------------------------------------- | ---------------- |
| `-i` / `--iface`   | Interfaz de red usada para enviar paquetes CDP           | `-i eth0`        |
| `-c` / `--count`   | Cantidad de paquetes a enviar (0 = infinito)             | `-c 1000`        |
| `-d` / `--delay`   | Tiempo de espera entre paquetes en segundos              | `-d 0.01`        |
| `-v` / `--verbose` | Muestra cada paquete enviado en pantalla                 | `-v`             |

---

## Ejecución del ataque

```bash
sudo python3 cdp_dos.py -i eth0 -c 1000 -v
```

Durante la ejecución, la salida mostrará cada paquete CDP falso enviado:

```
╔══════════════════════════════════════════════╗
║         CDP DoS Flood Attack                 ║
║  Interfaz : eth0                            ║
║  Paquetes : 1000                            ║
╚══════════════════════════════════════════════╝

[   1] Enviado → MAC=02:a1:b2:c3:d4:e5  DeviceID=SW-4F3A2B
[   2] Enviado → MAC=02:f1:e2:d3:c4:b5  DeviceID=RT-8C1D4E
...
[+] Total de paquetes CDP enviados: 1000
```

Para detener el ataque en modo infinito:
```
Ctrl + C
```

---

## Verificación del impacto en el switch

Durante el ataque, ejecutar en el switch:

```
SW-1# show cdp neighbors
SW-1# show processes cpu sorted
SW-1# show cdp traffic
```

Se observará:
- Decenas o cientos de vecinos CDP falsos en la tabla.
- Incremento notable en el uso de CPU.
- Contadores CDP disparados.

---

## Contramedidas aplicadas

### Opción 1 — Deshabilitar CDP globalmente

Si CDP no es necesario en el switch:

```
SW-1(config)# no cdp run
```

Verificar:
```
SW-1# show cdp neighbors
% CDP is not enabled
```

### Opción 2 — Deshabilitar CDP solo en el puerto del atacante (recomendado)

```
SW-1(config)# interface GigabitEthernet0/0
SW-1(config-if)# no cdp enable
SW-1(config-if)# exit
SW-1(config)# end
SW-1# write memory
```

Esta opción mantiene CDP activo en otros puertos donde puede ser útil para administración.

### Limpiar la tabla CDP después de mitigar

```
SW-1# clear cdp table
SW-1# clear cdp counters
SW-1# show cdp neighbors
```

---

## Verificación posterior a la mitigación

Luego de aplicar la contramedida, relanzar el script en Kali y verificar en el switch que ya no aparecen nuevas entradas CDP falsas y que el CPU se mantiene estable.

```
SW-1# show processes cpu sorted
SW-1# show cdp neighbors
SW-1# show cdp traffic
```

---

## Video demostrativo

La demostración práctica del ataque y su mitigación está disponible en YouTube:

[Ver video del laboratorio en YouTube](https://www.youtube.com/watch?v=PENDIENTE)

---

## Conclusión

Este laboratorio demuestra cómo el protocolo CDP, diseñado para descubrimiento de vecinos, puede ser abusado para generar una condición de Denegación de Servicio en switches Cisco. El ataque incrementa el uso de CPU y llena la tabla de vecinos con entradas falsas.

La contramedida más efectiva es deshabilitar CDP en interfaces no confiables o globalmente cuando no sea necesario, evitando que el dispositivo procese anuncios CDP desde equipos externos.

---

## Autor

**Darwing**  
Matrícula: **2024-2690**  
Repositorio: https://github.com/TuUsuario/CDP-DoS-Attack
