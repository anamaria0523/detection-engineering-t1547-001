# Paso 1: Configuración del Data Source (Sysmon)

## Objetivo
Configurar telemetría suficiente en un endpoint Windows para poder detectar 
la técnica **T1547.001 - Boot or Logon Autostart Execution: Registry Run Keys**.

## Entorno de laboratorio
- **Host:** Windows 11 Pro (8GB RAM)
- **Hipervisor:** Oracle VirtualBox
- **VM víctima:** Windows 10 Home (64-bit), 2GB RAM asignada, 40GB disco dinámico
- **Herramienta de telemetría:** Sysmon v15.21 (Sysinternals)
- **Configuración usada:** [sysmon-config de SwiftOnSecurity](https://github.com/SwiftOnSecurity/sysmon-config)

## ¿Por qué Sysmon?
Los logs nativos de Windows no registran de forma granular eventos como 
modificaciones al Registro, creación de procesos con línea de comandos completa, 
o conexiones de red por proceso. Sysmon cierra esta brecha de visibilidad, 
siendo el estándar de facto en la industria para detection engineering.

Para T1547.001 específicamente, el evento clave es:
- **Event ID 13 (RegistryEvent - Value Set):** se dispara cuando se escribe 
  un valor en el Registro, incluyendo las claves `Run`/`RunOnce` usadas para 
  persistencia.

## Pasos realizados
1. Descarga de Sysmon desde Microsoft Sysinternals
2. Descarga de archivo de configuración `sysmonconfig.xml` (SwiftOnSecurity)
3. Instalación vía símbolo del sistema (con privilegios de administrador):
sysmon64.exe -accepteula -i sysmonconfig.xml
4. Verificación y corrección de configuración:
   sysmon64.exe -c sysmonconfig.xml
5. Validación de eventos en el Visor de Eventos de Windows:
   `Registros de aplicaciones y servicios > Microsoft > Windows > Sysmon > Operational`

## Troubleshooting documentado
- **RAM limitada (8GB host):** se ajustó la VM a 2GB para viabilidad del lab.
- **Instalación desatendida colgada:** resuelto liberando RAM del host antes 
  de crear la VM.
- **Error "You need to launch Sysmon as an Administrator":** la terminal 
  abierta desde la barra de direcciones del Explorador no tiene privilegios 
  elevados; se resolvió abriendo `cmd.exe` con "Ejecutar como administrador".
- **Configuración no aplicada en primer intento** (comando cortado sin el 
  argumento del XML): corregido con el flag `-c` para actualizar la 
  configuración sin reinstalar el servicio.

## Resultado
✅ Sysmon instalado y corriendo como servicio  
✅ Configuración de SwiftOnSecurity aplicada y validada  
✅ Eventos confirmados en el Visor de Eventos (Event ID 1, 5, 22, entre otros)

## Próximo paso
Simular la técnica T1547.001 (crear una entrada en Registry Run Keys) y 
verificar que Sysmon capture el Event ID 13 correspondiente.
