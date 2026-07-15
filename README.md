# Detection Engineering — T1547.001 (Registry Run Keys Persistence)

Proyecto de portafolio en Detection Engineering, enfocado en la técnica de 
persistencia **T1547.001 (Registry Run Keys / Startup Folder)** del framework MITRE ATT&CK.

Este proyecto sigue el ciclo completo de Detection Engineering:

**Data Source → Detection Logic → Test & Validate → Deploy & Monitor**

## 🎯 Objetivo
Construir, probar y documentar una regla de detección funcional para identificar 
intentos de persistencia vía claves de Registro (`Run`/`RunOnce`), una de las 
técnicas más comunes usadas por malware y atacantes para sobrevivir a reinicios.

## 🧪 Entorno de laboratorio
- **Hipervisor:** Oracle VirtualBox
- **VM víctima:** Windows 10 (64-bit)
- **Herramienta de telemetría:** Sysmon (configuración de SwiftOnSecurity)

## 📋 Progreso del proyecto

- [x] **Paso 1:** Data Source — Instalación y configuración de Sysmon
- [x] **Paso 2:** Detection Logic — Escritura de regla Sigma para Event ID 13
- [x] **Paso 3:** Test & Validate — Simulación de la técnica y validación de la detección
- [ ] **Paso 4:** Deploy & Monitor — Consideraciones de despliegue en un SIEM

## 📚 Documentación

| Paso | Documento |
|------|-----------|
| 1. Data Source | [docs/01-data-source-setup.md](docs/01-data-source-setup.md) |
| 3. Test & Validate | [docs/03-test-validate.md](docs/03-test-validate.md) |

## 🔗 Referencias
- [MITRE ATT&CK - T1547.001](https://attack.mitre.org/techniques/T1547/001/)
- [Sysmon - Sysinternals](https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon)
- [SwiftOnSecurity Sysmon Config](https://github.com/SwiftOnSecurity/sysmon-config)
