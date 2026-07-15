# Paso 3: Test & Validate

## Objetivo
Simular la técnica T1547.001 en el laboratorio, confirmar que Sysmon la 
captura, y validar que la regla Sigma escrita en el Paso 2 detecta 
correctamente el evento generado — tanto de forma manual como automatizada.

## 1. Simulación del ataque

Se simuló la técnica ejecutando un comando `reg add` (sin privilegios de 
administrador, replicando el comportamiento típico de malware ejecutado 
con permisos de usuario estándar):
reg add HKCU\Software\Microsoft\Windows\CurrentVersion\Run /v TestPersistence /t REG_SZ /d notepad.exe /f

**Resultado:** `La operación se completó correctamente.`

## 2. Captura por Sysmon

Sysmon generó el **Event ID 13 (RegistryEvent - Value Set)** con estos datos:

| Campo | Valor |
|---|---|
| EventID | 13 |
| Image | `C:\Windows\system32\reg.exe` |
| TargetObject | `HKU\...\SOFTWARE\Microsoft\Windows\CurrentVersion\Run\TestPersistence` |
| Details | `notepad.exe` |

## 3. Validación manual (comparación campo por campo)

| Condición de la regla | Valor real del evento | ¿Coincide? |
|---|---|---|
| `EventID: 13` | `13` | ✅ |
| `TargetObject contiene 'CurrentVersion\Run'` | `...CurrentVersion\Run\TestPersistence` | ✅ |
| `condition: selection` | Ambas condiciones se cumplen | ✅ **MATCH** |

## 4. Validación automatizada con herramientas de la industria

### Herramientas usadas
- **Python 3.14** (lenguaje estándar del ecosistema de tooling de seguridad)
- **sigma-cli**: validación de sintaxis y conversión de la regla
- **Zircolite**: motor standalone que ejecuta reglas Sigma directamente 
  contra archivos `.evtx`

### Validación de sintaxis
sigma check t1547-001-registry-run-keys.yml

Resultado: `Found 0 errors, 0 condition errors and 0 issues.`

### Conversión de la regla a consulta SQL (backend SQLite, pipeline Sysmon)
sigma convert -t sqlite -p sysmon t1547-001-registry-run-keys.yml

Resultado: consulta SQL generada correctamente, confirmando que la regla 
es compatible con el pipeline de Sysmon.

### Ejecución contra el log real capturado
python zircolite.py --events sysmon_log.evtx --ruleset t1547-001-registry-run-keys.yml

**Resultado final:**

| Métrica | Valor |
|---|---|
| Eventos totales analizados | 624 |
| Reglas cargadas | 1 |
| Coverage | 1/1 rules matched (100%) |
| Eventos detectados | 7 |
| Severidad | Medium |
| Táctica MITRE ATT&CK | Persistence (T1547.001) |

## Troubleshooting documentado

- **`pip install zircolite` falló:** Zircolite no se distribuye vía PyPI; 
  se descargó directamente desde su repositorio de GitHub 
  (wagga40/Zircolite) y se ejecutó como script local.
- **Regla Sigma con 0 detecciones en el primer intento:** al inspeccionar 
  el archivo `.yml` se descubrió que solo pesaba 1 byte (contenido no se 
  guardó correctamente al copiar desde GitHub). Se recreó el archivo 
  manualmente y se confirmó su tamaño correcto (927 bytes) antes de 
  reintentar.
- **`UnicodeEncodeError` en la terminal de Windows al correr Zircolite:** 
  error cosmético relacionado con la codificación de caracteres de la 
  consola (no afectó el resultado del análisis).

## Conclusión

Se completó el ciclo de validación de extremo a extremo: ataque simulado → 
telemetría capturada por Sysmon → regla Sigma escrita → validada 
sintácticamente → convertida a consulta real → ejecutada contra el log 
real con una herramienta de la industria (Zircolite), confirmando una 
tasa de detección del 100% sobre los eventos generados.

## Próximo paso
Documentar consideraciones de despliegue y monitoreo continuo (Paso 4: 
Deploy & Monitor).

