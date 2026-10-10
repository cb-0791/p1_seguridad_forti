[[# tarea4_fuga_20250791](https://youtu.be/v0IvYmou8vQ)

# fortigate-security-lab-infra1-0791
Laboratorio de Seguridad Perimetral e Infraestructura Segura con FortiGate

# Laboratorio de Seguridad FortiGate: DMZ, Anti-Fuga, Control SSH por Roles y Filtrado Web

**Estudiante:** Cóndor Bautista  
**Matrícula:** 2025-0791  
**Materia:** Seguridad en Redes  
**Enlace al Video Demostrativo:** https://youtu.be/v0IvYmou8vQ

---

##  1. Resumen del Proyecto
Este proyecto documenta e implementa una topología de red segura en GNS3 administrada íntegramente mediante la interfaz gráfica (GUI) de un Firewall FortiGate. Se implementa el aislamiento de servidores críticos ubicados en una zona DMZ, políticas estrictas para prevenir la fuga de tráfico lateral hacia las redes locales (LAN), restricción de acceso a Internet en la DMZ limitada exclusivamente a repositorios de actualización autorizados, control granular de acceso SSH basado en roles de VLAN, y un sistema de filtrado de contenido Web para restringir el acceso al servidor de inventario con visualización obligatoria de página de advertencia ante violaciones de política.

---

##  2. Arquitectura y Esquema de Direccionamiento

### Topología General
*(Inserta aquí la captura o diagrama de tu topología de GNS3 de la Infraestructura 1)*

### Tabla de Subredes e Interfaces
| Dispositivo / Zona | Interfaz / VLAN | Dirección IP / Subred | Función / Descripción |
| :--- | :--- | :--- | :--- |
| **FortiGate Gateway** | `port1.10` (VLAN 10) | `10.7.91.1/25` | Gateway de la VLAN de Usuarios Generales |
| **FortiGate Gateway** | `port1.20` (VLAN 20) | `10.7.91.129/26` | Gateway de la VLAN de Administradores |
| **FortiGate Gateway** | `port3` (DMZ) | `10.7.91.192/28` | Gateway de la Zona de Servidores |
| **webterm-1** | VLAN 10 | `10.7.91.10/25` | Host Cliente / Usuario General |
| **sysCAJA** | DMZ | `10.7.91.193/28` | Servidor Web: Sistema de Caja |
| **sysINVENTARIO** | DMZ | `10.7.91.195/28` | Servidor Web: Sistema de Inventario |
| **db-server** | DMZ | `10.7.91.196/28` | Servidor de Base de Datos |

---

##  3. Políticas de Seguridad e Implementaciones en la GUI

### 3.1. Aislamiento de la DMZ y Políticas Anti-Fuga (`DMZ_Bloquear_Trafico_LAN`)
Para garantizar que los servidores expuestos en la zona DMZ no representen un riesgo de compromiso lateral en caso de vulneración, se configuraron reglas de Firewall de tipo **DENY** desde la interfaz `port3` (DMZ) hacia las interfaces de la red interna `port1.10` y `port1.20`. Cualquier intento de comunicación iniciada desde los servidores hacia la red de usuarios es interceptado y descartado de manera inmediata por el firewall.

### 3.2. Restricción de Acceso a Internet en la DMZ (`Repos_Actualizacion`)
Se implementó un esquema de salida a Internet restrictivo para la DMZ:
- **Objeto FQDN:** Se creó un objeto de tipo FQDN (`archive.ubuntu.com`) para acotar el destino permitido exclusivamente a los repositorios oficiales de parches y actualizaciones.
- **Regla de Permiso (`DMZ_Permitir_Updates`):** Permite el tráfico HTTP/HTTPS únicamente hacia el FQDN autorizado.
- **Regla de Bloqueo General (`DMZ_Bloquear_Internet_General`):** Colocada inmediatamente debajo de la anterior, deniega todo el tráfico restante hacia `destination: all` mediante una acción **DENY**.

### 3.3. Control de Acceso SSH por Roles de VLAN
Se segmentó el acceso administrativo remoto hacia la DMZ para cumplir con el principio de mínimo privilegio:
- **Acceso Permitido (`VLAN20_SSH_Admin_DMZ`):** Autoriza el tráfico con servicio **SSH** desde la interfaz `port1.20` (VLAN 20 - Administradores) hacia la DMZ.
- **Acceso Denegado (`VLAN10_Bloquear_SSH_DMZ`):** Bloquea de forma explícita el servicio **SSH** originado desde la interfaz `port1.10` (VLAN 10 - Usuarios generales) hacia la zona de servidores.

### 3.4. Filtrado Web y Página de Advertencia para el Sistema de Inventario
Para restringir el acceso al servidor de inventario (`10.7.91.195`) desde la VLAN de usuarios generales:
- **Perfil de Web Filter:** Se configuró un perfil personalizado (`Perfil_Inventario_Bloqueado`) aplicando un filtro de URL mediante comodines (`*10.7.91.195*`) con la acción configurada en **Block**.
- **Comportamiento:** Al intentar acceder a la aplicación desde la VLAN 10, el FortiGate intercede en la sesión HTTP/HTTPS y despliega en el navegador del usuario la página oficial de advertencia de violación de política de seguridad.

### 3.5. Asignación Dinámica de Direccionamiento (DHCP Server)
Se habilitó el servicio de asignación automática de direcciones IP directamente desde la GUI en el menú `Network > Interfaces` sobre la subinterfaz `port1.10`:
- **Rango de IPs:** `10.7.91.2` - `10.7.91.126`
- **Máscara de Subred:** `255.255.255.128` (/25)
- **Gateway Predeterminado:** `10.7.91.1`

---

##  4. Resultados y Pruebas de Verificación


### Prueba A: Verificación de Anti-Fuga DMZ ➔ LAN
> Captura de la regla de Firewall `DMZ_Bloquear_Trafico_LAN` habilitada en la GUI del FortiGate 
<img width="448" height="505" alt="imagen_2026-10-10_172759720" src="https://github.com/user-attachments/assets/78c8dd51-58d0-4f60-a90b-a63b8b86ce54" />


---

### Prueba B: Restricción de Internet en la DMZ (FQDN)
>  Captura de pantalla de la sección **Policy & Objects > Firewall Policy** mostrando la regla de FQDN para actualizaciones y la regla de bloqueo general debajo.
<img width="540" height="509" alt="imagen_2026-10-10_173512091" src="https://github.com/user-attachments/assets/7abaad27-c3c5-4731-8a8d-aaaa05f31217" />
<img width="763" height="123" alt="imagen_2026-10-10_173615914" src="https://github.com/user-attachments/assets/fbbea71a-20cf-4411-ac80-53bc76afa8f4" />


---

### Prueba C: Control de Acceso SSH por Roles de VLAN
>  Captura de los detalles de la política de Firewall que permite SSH desde la VLAN 20 y la regla que lo bloquea desde la VLAN 10.
<img width="756" height="220" alt="imagen_2026-10-10_173721775" src="https://github.com/user-attachments/assets/28965713-9dc4-4345-bc75-07a3e8995884" />


---

### Prueba D: Bloqueo de Acceso Web al Inventario (Web Filter)
>  Captura de pantalla del navegador del usuario (VLAN 10) mostrando la **Página de Bloqueo por Web Filter (FortiGuard Security Warning)** al intentar acceder al servidor de inventario.
<img width="647" height="519" alt="imagen_2026-10-10_173855003" src="https://github.com/user-attachments/assets/f5e780f0-d882-4a68-875c-0b1fdad80b83" />
