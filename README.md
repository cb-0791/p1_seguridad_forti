[# p1_seguridad_forti](https://youtu.be/QQkzK84si5U

# fortigate-security-lab-infra1-0791
Laboratorio de Seguridad Perimetral e Infraestructura Segura con FortiGate

# Laboratorio de Seguridad FortiGate: DMZ, Anti-Fuga, Control SSH por Roles y Filtrado Web

**Estudiante:** Cóndor Bautista  
**Matrícula:** 2025-0791  
**Materia:** Seguridad en Redes  
**Enlace al Video Demostrativo:** https://youtu.be/QQkzK84si5U

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
- **Comando desde un contenedor de la DMZ (`sysCAJA`):**
  ```bash
  ping -c 3 10.7.91.1)

  ##  4. Resultados y Pruebas de Verificación

### Prueba A: Verificación de Anti-Fuga DMZ ➔ LAN
- **Comando desde un contenedor de la DMZ (`sysCAJA`):**
  ```bash
  ping -c 3 10.7.91.1
Resultado: 100% de pérdida de paquetes (Destination Host Unreachable / Timeout). Se comprueba que las políticas de bloqueo impiden el tráfico lateral hacia la LAN.

Prueba B: Restricción de Internet en la DMZ
Comando desde sysINVENTARIO hacia internet general:

Bash
curl -I [https://google.com](https://google.com)
Resultado: Conexión rechazada / Tiempo de espera agotado. El FortiGate bloquea el tráfico web general, permitiendo únicamente el acceso a los repositorios definidos por FQDN.

Prueba C: Control de Acceso SSH por Roles
Intento desde webterm-1 (VLAN 10):

Bash
ssh user@10.7.91.196
Resultado: Conexión bloqueada y rechazada por la política de seguridad.

Intento desde webterm-2 (VLAN 20):

Bash
ssh admin@10.7.91.196
Resultado: Conexión SSH exitosa validada por el rol de administración autorizado.

Prueba D: Bloqueo de Acceso Web al Inventario (Web Filter)
Acción desde webterm-1 (VLAN 10):
Ingresar mediante navegador o curl a http://10.7.91.195.

Resultado: El FortiGate intercepta la petición y presenta al usuario la Página de Bloqueo por Violación de Política Web, cumpliendo con el requerimiento de visibilidad del bloqueo.
