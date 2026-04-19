---
name: opencode-threat-modeling
description: >
  Metodología de modelado de amenazas Maestro adaptada a Opencode.
  Trigger: Cuando se requiera evaluar riesgos de seguridad, revisar fronteras de confianza
  o analizar amenazas en repositorios y entornos de Opencode.
license: Apache-2.0
metadata:
  author: gentleman-programming
  version: "1.0"
---

## Propósito en Opencode

Esta skill implementa el framework **Maestro** para el análisis de seguridad preventivo. En el ecosistema Opencode, la seguridad no es un parche, es una **frontera de confianza** (Trust Boundary) que debemos definir antes de codear.

## Flujo de Análisis Opencode (Boundary-First)

1.  **Mapa del Sistema**: Identificar actores (agentes/usuarios), almacenamiento, herramientas/APIs y dependencias externas.
2.  **Fronteras de Confianza**: ¿Dónde cambia la confianza? ¿Dónde un input no confiable se convierte en una acción privilegiada?
3.  **Amenazas por Frontera**: 
    -   *Spoofing, Tampering, Disclosure*.
    -   *Prompt Injection* (Crítico en agentes de IA).
    -   *Tool Abuse* y automatización insegura.
4.  **Capas Maestro (Opcional)**: Mapear a las 7 capas (Foundation Models, Data Ops, Agent Frameworks, Infrastructure, Observability, Compliance, Ecosystem).

## Entregables Opencode

Los reportes deben seguir el formato de nombrado de Opencode para ser fácilmente indexables por Engram:

`YYYY-MM-DD_opencode_<target>_<artifact>.md`

- **Artifacts**: `summary`, `full-report`, `security-note`.

## Reglas de Oro de Seguridad

- **Least Privilege**: El agente solo debe tener los permisos mínimos necesarios para la tarea.
- **Validation/Normalization**: Todo input externo es hostil hasta que se demuestre lo contrario.
- **Auditability**: Cada acción crítica debe dejar un rastro claro en los logs o el historial de git.

## Comandos de Auditoría

```bash
# Inspeccionar estado y logs recientes
git status --short && git log --oneline -n 5

# Revisar configuración del repo (GitHub)
gh repo view --json rulesets,visibility
```

## Recursos

- **Templates**: Ver `skills/opencode-threat-modeling/assets/` para reportes técnicos.
- **Taxonomía**: Basado en OWASP para Agentes de IA.
