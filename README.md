# 🛡️ Wazuh — Módulos de Seguridad

> Documentación de los módulos de seguridad del entorno Wazuh SIEM orientados a la detección y monitoreo de amenazas en endpoints.

---

## 📋 Tabla de Contenidos

- [Descripción General](#descripción-general)
- [Módulos](#módulos)
  - [Malware Detection](#-malware-detection)
  - [File Integrity Monitoring](#-file-integrity-monitoring)
  - [Vulnerability Detection](#-vulnerability-detection)
- [Requisitos](#requisitos)
- [Referencias](#referencias)

---

## Descripción General

Wazuh es una plataforma de seguridad open source que permite monitorear, detectar y responder ante amenazas en tiempo real. Este repositorio documenta tres de sus módulos principales enfocados en la seguridad de endpoints: detección de malware, integridad de archivos y detección de vulnerabilidades.

Cada módulo opera a través del agente Wazuh instalado en el endpoint, que recopila información y la envía al servidor para su análisis y generación de alertas en el Dashboard.

---

## Módulos

### 🦠 Malware Detection

**Check indicators of compromise triggered by malware infections or cyberattacks.**

Detecta indicadores de compromiso (IOCs) en el sistema. Analiza logs, eventos y comportamientos del endpoint comparándolos con reglas de detección y feeds de inteligencia de amenazas. Cuando identifica actividad sospechosa relacionada con malware o un ataque en curso, genera una alerta clasificada por nivel de severidad.

Es útil para identificar infecciones activas, movimientos laterales, accesos no autorizados y cualquier comportamiento que se desvíe del patrón normal del sistema.

---

### 📁 File Integrity Monitoring

**Alerts related to file changes, including permissions, content, ownership, and attributes.**

Monitorea en tiempo real archivos y directorios del sistema, alertando ante cualquier modificación no autorizada. Registra cambios en el contenido, permisos, propietario y atributos de los ficheros vigilados.

Es especialmente útil para detectar manipulaciones en archivos críticos del sistema operativo, cambios introducidos por atacantes tras una intrusión, o modificaciones accidentales que puedan afectar la integridad del entorno.

---

### 🔍 Vulnerability Detection

**Discover what applications in your environment are affected by well-known vulnerabilities.**

Analiza el software instalado en los endpoints y lo cruza con bases de datos de vulnerabilidades conocidas (CVE) para identificar aplicaciones desactualizadas o con fallos de seguridad. Proporciona información sobre la severidad de cada vulnerabilidad, el paquete afectado y la versión que la corrige cuando está disponible.

Permite tener visibilidad del estado de exposición del entorno y priorizar las actualizaciones según el nivel de riesgo.

---

## Requisitos

- Wazuh Server `>= 4.x` instalado y operativo
- Agente Wazuh activo en los endpoints a monitorear
- Acceso al Wazuh Dashboard

---

## 🔗 Referencias

- [Documentación oficial de Wazuh](https://documentation.wazuh.com)
- [Base de datos CVE - NVD](https://nvd.nist.gov)
- [MITRE ATT&CK Framework](https://attack.mitre.org)

---

> **Entorno:** Ubuntu 22.04.4 + Agente Windows