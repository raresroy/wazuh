# 🛡️ Wazuh SIEM — Documentación Técnica de Laboratorio

> Guía completa de instalación, configuración y análisis de seguridad con **Wazuh v4.14** sobre Ubuntu 22.04 LTS.

---

## 📋 Índice

- [¿Qué es Wazuh?](#-qué-es-wazuh)
- [Arquitectura](#-arquitectura)
- [Documentos del repositorio](#-documentos-del-repositorio)
- [Instalación rápida](#-instalación-rápida)
- [Post-instalación](#-post-instalación)
- [Dashboard y módulos](#-dashboard-y-módulos)
- [Requisitos de hardware](#-requisitos-de-hardware)
- [Cálculo de almacenamiento](#-cálculo-de-almacenamiento)
- [Recursos oficiales](#-recursos-oficiales)

---

## 🔍 ¿Qué es Wazuh?

**Wazuh** es una plataforma de seguridad **open source** que funciona como SIEM *(Security Information and Event Management)*. Recopila logs de todos los equipos y dispositivos de la red, los analiza en tiempo real y genera alertas ante comportamientos anómalos.

### ¿Qué detecta?

| Capacidad | Descripción |
|-----------|-------------|
| 🔴 Ataques activos | Intrusiones, fuerza bruta, escaneos de red |
| 📤 Exfiltración | Filtraciones o fuga de datos |
| ⚠️ Anomalías | Comportamientos irregulares en el sistema |
| 🐛 Vulnerabilidades | CVEs conocidos en software instalado |
| 🧩 Técnicas de ataque | Catálogo basado en MITRE ATT&CK |

---

## 🏗️ Arquitectura

Wazuh se compone de **tres módulos interdependientes**:

```
┌─────────────┐     logs      ┌──────────────┐    alertas    ┌─────────────┐
│   AGENTES   │ ─────────────▶│    SERVER    │ ─────────────▶│   INDEXER   │
│ (endpoints) │               │ (procesamiento│               │ (OpenSearch)│
└─────────────┘               │  + reglas)   │               └──────┬──────┘
                               └──────────────┘                      │
                                                                      ▼
                                                              ┌──────────────┐
                                                              │  DASHBOARD   │
                                                              │ (interfaz web│
                                                              └──────────────┘
```

| Componente | Función |
|------------|---------|
| **Dashboard** | Visualización, alertas, reportes y administración web |
| **Server** | Núcleo: aplica reglas de detección y genera alertas |
| **Indexer** | Almacena e indexa logs y alertas (basado en OpenSearch) |

---

## 📁 Documentos del repositorio

Este repositorio contiene cuatro guías de laboratorio:

| Archivo | Contenido |
|---------|-----------|
| `WAZUH_Introduccion.pdf` | Arquitectura, componentes, flujo de operación, requisitos de hardware y almacenamiento |
| `WAZUH_Instalacion.pdf` | Instalación del servidor con script quickstart, despliegue del primer agente Windows |
| `WAZUH_Post_Instalacion.pdf` | Post-instalación: grupos de agentes, despliegue en Linux (Kali), configuraciones por grupo, visor de logs |
| `WAZUH_Dashboard.pdf` | Panel del agente, System Inventory, detección de vulnerabilidades, SCA y módulo Discover |

---

## ⚡ Instalación rápida

### Requisitos previos (laboratorio)

| Parámetro | Valor |
|-----------|-------|
| Sistema operativo | Ubuntu 22.04.4 LTS |
| RAM | 8 GB mínimo |
| Procesadores | 2 vCPU |
| Disco | 50 GB |
| Red | NAT o bridged |

### Comando de instalación (all-in-one)

```bash
curl -sO https://packages.wazuh.com/4.14/wazuh-install.sh \
  && sudo bash ./wazuh-install.sh -a
```

> Este script instala automáticamente los tres componentes: **Indexer**, **Server** y **Dashboard**.

Al finalizar, el script muestra las credenciales de acceso:

```
INFO: You can access the web interface https://<IP>:443
User: admin
Password: <generada automáticamente>
```

> ⚠️ **Importante:** guardar la contraseña generada. También queda almacenada en `wazuh-install-files.tar`.

### Despliegue de agente en Windows

```powershell
# Desde el Dashboard: Agents > Deploy new agent > Windows
# Copiar y ejecutar el comando generado, luego:
NET START Wazuh
```

### Despliegue de agente en Linux

```bash
# Desde el Dashboard: Agents > Deploy new agent > Linux (DEB)
# Copiar y ejecutar el comando generado, luego:
systemctl start wazuh-agent
systemctl enable wazuh-agent
```

---

## 🔧 Post-instalación

### Deshabilitar actualizaciones automáticas

```bash
sed -i "s/^deb /#deb /" /etc/apt/sources.list.d/wazuh.list
apt update
```

### Ajuste de memoria del Indexer (JVM)

Editar `/etc/wazuh-indexer/jvm.options`:

```
# Por defecto (1 GB):
-Xms1024m
-Xmx1024m

# Recomendado para laboratorio (4 GB):
-Xms4g
-Xmx4g
```

```bash
systemctl restart wazuh-indexer
```

### Estructura de directorios

| Directorio | Componente |
|------------|------------|
| `/etc/wazuh-dashboard` | Dashboard (interfaz web) |
| `/etc/wazuh-indexer` | Indexer (OpenSearch) |
| `/etc/filebeat` | Server (envío de logs al Indexer) |

### Gestión de grupos de agentes

Los grupos permiten distribuir configuraciones personalizadas a conjuntos de endpoints.

**Crear un grupo:** `Agents Management > Groups > Add new group`

**Grupos recomendados para una red típica:**

| Grupo | Descripción |
|-------|-------------|
| `Windows` | Estaciones de trabajo Windows |
| `Linux` | Servidores y equipos Linux |
| `Servidores` | Equipos con rol de servidor |
| `Usuarios` | Equipos de usuarios finales |
| `Firewall` / `Switch` | Dispositivos de red |

> Un agente puede pertenecer a **múltiples grupos** simultáneamente.

**Configuración por grupo** (`agent.conf`):

```xml
<agent_config>
  <!-- Ejemplo: monitorear carpeta Descargas en Windows -->
  <localfile>
    <location>C:\Users\*\Downloads\*</location>
    <log_format>syslog</log_format>
  </localfile>
</agent_config>
```

---

## 📊 Dashboard y módulos

### Panel del agente

Al seleccionar un agente en el Dashboard se accede a:

- **Identificación:** ID, estado, IP, versión, OS, grupos
- **System Inventory:** hardware (CPU, RAM, hostname), software instalado, procesos, red, servicios
- **Métricas 24h:** evolución de eventos, tácticas MITRE ATT&CK, cumplimiento PCI DSS

### System Inventory

| Pestaña | Contenido |
|---------|-----------|
| Dashboard | Resumen de OS, paquetes, procesos y CPUs |
| System | Hardware: CPU, RAM, hostname, número de serie |
| Software | Lista de paquetes con nombre, proveedor y versión |
| Processes | Procesos activos en el momento del inventario |
| Network | Interfaces, IPs y configuración de red |
| Identity | Usuarios y grupos del sistema |
| Services | Servicios del SO y su estado |

### Detección de Vulnerabilidades

Wazuh realiza detección **pasiva**: inventaría el software instalado y lo compara contra bases de datos de CVEs.

**No lanza ataques ni sondas activas contra el equipo.**

Niveles de severidad:

```
🔴 Critical  ──  máxima prioridad, acción inmediata
🟠 High      ──  alto riesgo, atender a la brevedad
🟡 Medium    ──  riesgo moderado
🟢 Low       ──  riesgo bajo
```

### Security Configuration Assessment (SCA)

El módulo SCA compara la configuración del equipo contra benchmarks internacionales (ej: **CIS Microsoft Windows 11 Enterprise Benchmark v3.0.0**).

Cada chequeo devuelve: `Passed` | `Failed` | `Not applicable`

> Un score bajo (ej: 24%) **no indica compromiso**, sino que el equipo aún no ha sido sometido a *hardening*. Cada chequeo fallido es una oportunidad de mejora.

### Discover (Exploración de logs)

El módulo Discover permite buscar y filtrar logs en tiempo real sobre el índice `wazuh-alerts-*`.

**Métodos de filtrado:**

```
# Filtro rápido por exclusión
Clic en '-' junto al valor a excluir → aplica filtro NOT automático

# Filtro avanzado
Add filter → Field: agent.ip | Operator: is | Value: 192.168.X.X
```

Las búsquedas pueden **guardarse** (*Save search*) y vincularse a dashboards personalizados.

---

## 💾 Requisitos de hardware

### Wazuh Indexer

| | RAM mínima | CPU mín. | RAM recomendada | CPU rec. |
|---|---|---|---|---|
| Indexer | 4 GB | 2 núcleos | 16 GB | 8 núcleos |

### Wazuh Server

| | RAM mínima | CPU mín. | RAM recomendada | CPU rec. |
|---|---|---|---|---|
| Server | 2 GB | 2 núcleos | 4 GB | 8 núcleos |

### Por número de agentes

| Agentes | CPU | RAM | Disco (90 días) |
|---------|-----|-----|-----------------|
| 1 – 25 | 4 vCPU | 8 GiB | 50 GB |
| 25 – 50 | 8 vCPU | 8 GiB | 100 GB |
| 50 – 100 | 8 vCPU | 8 GiB | 200 GB |

---

## 🗄️ Cálculo de almacenamiento

### Consumo estimado en el Indexer (90 días)

| Tipo de endpoint | GB/día | GB/90 días |
|-----------------|--------|-----------|
| Servidores | 0.25 | 3.7 GB |
| Estaciones de trabajo | 0.10 | 1.5 GB |
| Dispositivos de red | 0.50 | 7.4 GB |

### Fórmula de dimensionamiento

```
HDD (90 días) = (N° Servidores × 3.8 GB)
              + (N° Estaciones × 1.54 GB)
              + (N° Dispositivos de red × 7.4 GB)
```

---

## 📚 Recursos oficiales

- 📖 [Documentación oficial de Wazuh](https://documentation.wazuh.com/current/quickstart.html)
- 🐛 [CVE Database](https://cve.mitre.org/)
- 🔐 [CIS Benchmarks](https://www.cisecurity.org/cis-benchmarks)
- 🧩 [MITRE ATT&CK Framework](https://attack.mitre.org/)

---

<div align="center">

*Documentación preparada con base en las notas de laboratorio del curso de Wazuh SIEM*  
*Laboratorios 01–03 · Wazuh v4.14 · Ubuntu 22.04.4 LTS*

</div>