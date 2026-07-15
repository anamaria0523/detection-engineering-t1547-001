# Paso 2: Detection Logic

## Objetivo
Traducir el comportamiento observado de la técnica T1547.001 en una regla 
de detección escrita en formato **Sigma**, el estándar de la industria 
para reglas de detección agnósticas de SIEM.

## ¿Por qué Sigma?
Cada SIEM (Splunk, Elastic, Sentinel, QRadar) tiene su propio lenguaje de 
búsqueda. Sigma permite escribir la lógica de detección una sola vez y 
traducirla automáticamente al motor de cualquier plataforma, evitando 
reescribir la misma detección varias veces.

## La regla

Archivo completo: [t1547-001-registry-run-keys.yml](../detection-rules/t1547-001-registry-run-keys.yml)

```yaml
title: Persistencia vía Registry Run Keys (T1547.001)
id: 7f3a2e1c-8b4d-4e5f-9a1b-2c3d4e5f6a7b
status: experimental
logsource:
    category: registry_event
    product: windows
detection:
    selection:
        EventID: 13
        TargetObject|contains:
            - 'CurrentVersion\Run'
            - 'CurrentVersion\RunOnce'
    condition: selection
level: medium
tags:
    - attack.persistence
    - attack.t1547.001
```

## Decisiones de diseño

| Decisión | Justificación |
|---|---|
| `EventID: 13` | Es el evento de Sysmon "RegistryEvent (Value Set)" — se dispara cuando se **escribe** un valor en el Registro, no cuando solo se lee. |
| `TargetObject\|contains` con lista (`Run`, `RunOnce`) | Ambas claves son variantes válidas del mismo mecanismo de persistencia; una sola regla cubre ambos casos en lugar de duplicar lógica. |
| `level: medium` (no "high") | El mismo comportamiento técnico (escribir en Run keys) lo realizan también programas legítimos (instaladores, actualizadores). La severidad real depende de contexto adicional (qué proceso, qué ruta del ejecutable, etc.), por lo que no se marca como crítica por sí sola. |
| `falsepositives` documentados | Un analista de SOC necesita saber que esta alerta puede generarse por software legítimo, para priorizar correctamente su investigación. |

## Validación de sintaxis

La regla fue validada con `sigma-cli` antes de continuar al siguiente paso:
sigma check t1547-001-registry-run-keys.yml

Resultado: `Found 0 errors, 0 condition errors and 0 issues.`

## Próximo paso
Simular el ataque real y validar que esta regla detecta correctamente el 
evento generado (Paso 3: Test & Validate).


