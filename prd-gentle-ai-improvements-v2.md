# PRD: gentle-ai + OpenCode + SDD Ecosystem Improvements

> **Version**: 2.0 (Corregida y Simplificada)  
> **Author**: Senior Architect Review  
> **Date**: 2026-04-19  
> **Status**: Draft - Ready for Implementation

---

## Executive Summary

Este documento analiza el ecosistema actual de gentle-ai + OpenCode + SDD y propone mejoras **simplificadas** para resolver los problemas reales sin caer en overengineering.

### Problemas Reales Identificados

1. **Acoplamiento de contexto** - AGENTS.md mezcla identidad + orchestrator + config (224 líneas)
2. **Scope mixing** - No hay separación clara entre configuración global vs por-proyecto
3. **Falta de carga contextual** - Los skills se cargan manualmente, no dinámicamente según fase

### Solución: 2 Fases Simplificadas

**Fase 1 (Quick Wins):** Separar responsabilidades, clarificar scope  
**Fase 2 (Automatización):** Carga contextual de skills según fase SDD

**NO vamos a:**
- Crear registry YAML innecesario
- Auto-load masivo de skills (context pollution)
- Eliminar prompts/sdd/ (son pointers necesarios)

---

## Estado Actual (As-Is)

### Arquitectura General

```
~/.config/opencode/ (GLOBAL - para todos los proyectos)
├── AGENTS.md (224 líneas - TODO mezclado)
├── opencode.json (563 líneas - config de agentes)
├── skills/ (50+ skills)
│   ├── sdd-*/ (10 SDD skills - 186 líneas cada uno)
│   └── [40+ general skills]
├── commands/ (8 slash commands)
├── prompts/sdd/ (10 archivos - POINTERS a skills)
├── .atl/skill-registry.md (auto-generado)
└── openspec/ (artifact storage)

MCP Servers:
├── context7 (remote)
└── engram (local)
```

### Relación prompts/ vs skills/ (CORREGIDO)

**Antes pensábamos:** Duplicación  
**Realidad arquitectónica:**

```
prompts/sdd/sdd-explore.md (1 línea):
"Read your skill file at ~/.config/opencode/skills/sdd-explore/SKILL.md"

skills/sdd-explore/SKILL.md (186 líneas):
[TODO el contenido de cómo ejecutar la fase]
```

**Conclusión:** `prompts/sdd/` son **POINTERS**, NO duplicación. Se mantienen.

### Componente: AGENTS.md (224 líneas)

**Contenido actual:**
- Líneas 1-59: Identidad (Rules, Personality, Philosophy, Expertise)
- Líneas 60-224: SDD Orchestrator Rules (Delegation, Artifact Store, Engram Protocol)

**Problema:** Mezcla de responsabilidades - "quién soy" vs "cómo orquesto SDD"

### Componente: Scope Mixing

**Problema detectado:**
- `~/.config/opencode/` es GLOBAL (para todos los proyectos)
- Pero el SDD workflow es POR-PROYECTO (cada repo tiene su propio flujo)

**Consecuencia:** Si ponés orchestrator rules en global, todos los proyectos usan las mismas reglas. Esto es bajo acoplamiento incorrecto.

---

## Propuesta (To-Be)

### Arquitectura Objetivo

```
~/.config/opencode/ (GLOBAL)
├── AGENTS.md (60 líneas - SOLO identidad)
├── opencode.json (config de agentes)
├── skills/ (50+ skills)
├── prompts/sdd/ (pointers - SE MANTIENEN)
└── ...

{project-root}/.gentle/ (LOCAL - por proyecto)
├── orchestrator.md (SDD workflow rules)
├── context-protocol.md (cómo pasar contexto)
├── sdd.yaml (config de artifact store)
└── standards/
    ├── hexagonal-architecture.md
    ├── clean-architecture.md
    └── testing-patterns.md
```

### Separación de Responsabilidades (CORREGIDA)

| Archivo | Scope | Qué contiene | Quién lo lee |
|---------|-------|---------------|---------------|
| `~/.config/opencode/AGENTS.md` | Global | Identidad, philosophy, reglas globales | Todos los agentes |
| `{project}/.gentle/orchestrator.md` | Local | SDD workflow, delegation rules | Solo orchestrator en ese proyecto |
| `{project}/.gentle/context-protocol.md` | Local | Cómo pasar contexto a sub-agentes | Solo orchestrator |
| `~/.config/opencode/skills/sdd-*/SKILL.md` | Global | Cómo ejecutar cada fase | Sub-agentes SDD |
| `~/.config/opencode/prompts/sdd/*.md` | Global | Pointers a skills | opencode.json |
| `{project}/.gentle/standards/*.md` | Local | Convenciones arquitectónicas del proyecto | Sub-agentes según contexto |

---

## Plan de Implementación (SIMPLIFICADO)

### Fase 1: Separación de Responsabilidades (Semana 1)

**Objetivo:** Separar identidad de workflow, clarificar scope global vs local

#### Tarea 1.1: Extraer SDD Orchestrator de AGENTS.md

**Antes:** AGENTS.md tiene 224 líneas (identidad + orchestrator)  
**Después:** 
- AGENTS.md: 60 líneas (SOLO identidad)
- `.gentle/orchestrator.md`: Reglas de SDD workflow (extraídas)

```
Archivos afectados:
- ~/.config/opencode/AGENTS.md (reducir a 60 líneas)

Archivos a crear (TEMPLATE para proyectos):
- ~/.config/opencode/templates/.gentle/orchestrator.md
- ~/.config/opencode/templates/.gentle/context-protocol.md
- ~/.config/opencode/templates/.gentle/sdd.yaml
```

**Criterios de éxito:**
- AGENTS.md tiene SOLO: Rules, Personality, Language, Tone, Philosophy, Expertise, Behavior
- NO tiene: Delegation Rules, Artifact Store Policy, Engram Protocol
- Template `.gentle/` existe para copiar a proyectos

#### Tarea 1.2: Crear estructura .gentle/ en proyecto actual

**Acción:** Copiar template a `~/.config/opencode/.gentle/` (para usar OpenCode mismo como proyecto)

```
Archivos a crear:
- ~/.config/opencode/.gentle/orchestrator.md (copiado de template)
- ~/.config/opencode/.gentle/context-protocol.md
- ~/.config/opencode/.gentle/sdd.yaml
- ~/.config/opencode/.gentle/standards/README.md
```

**Contenido de sdd.yaml:**
```yaml
artifact_store: engram
project: opencode
conventions:
  - hexagonal-architecture
  - clean-architecture
  - testing-patterns
```

#### Tarea 1.3: Actualizar opencode.json para leer .gentle/

**Cambio:** El agente `sdd-orchestrator` debe leer `.gentle/orchestrator.md` del proyecto actual

```json
"sdd-orchestrator": {
  "model": "kiro/claude-sonnet-4-5-1m",
  "prompt": "{file:./AGENTS.md}\n\n{file:.gentle/orchestrator.md}",
  "tools": { ... }
}
```

```
Archivos afectados:
- ~/.config/opencode/opencode.json
```

**Criterios de éxito:**
- Orchestrator lee AGENTS.md (identidad) + .gentle/orchestrator.md (workflow)
- Funcionalidad SDD intacta

**Impacto esperado después de Fase 1:**
- AGENTS.md legible (60 líneas)
- Separación clara: global (identidad) vs local (workflow)
- Template `.gentle/` disponible para otros proyectos

---

### Fase 2: Carga Contextual de Skills (Semana 2)

**Objetivo:** Cargar skills dinámicamente según fase SDD, NO auto-load masivo

#### Tarea 2.1: Implementar Contextual Loader en orchestrator

**Problema:** Actualmente los sub-agentes leen su skill manualmente  
**Solución:** El orchestrator detecta la fase y carga el skill automáticamente

**Antes:**
```
Usuario: "/sdd-new feature"
Orchestrator: lanza sdd-explore
Sub-agent sdd-explore: lee skills/sdd-explore/SKILL.md manualmente
```

**Después:**
```
Usuario: "/sdd-new feature"
Orchestrator: detecta fase "explore"
Orchestrator: carga skills/sdd-explore/SKILL.md
Orchestrator: lanza sdd-explore CON el skill ya cargado
```

**Implementación en `.gentle/orchestrator.md`:**
```markdown
## Contextual Skill Loading

Antes de lanzar un sub-agente SDD, carga su skill:

| Fase | Skill a cargar |
|------|----------------|
| init | skills/sdd-init/SKILL.md |
| explore | skills/sdd-explore/SKILL.md |
| propose | skills/sdd-propose/SKILL.md |
| spec | skills/sdd-spec/SKILL.md |
| design | skills/sdd-design/SKILL.md |
| tasks | skills/sdd-tasks/SKILL.md |
| apply | skills/sdd-apply/SKILL.md |
| verify | skills/sdd-verify/SKILL.md |
| archive | skills/sdd-archive/SKILL.md |

Patrón de delegación:
1. Detectar fase actual
2. Leer skill correspondiente
3. Pasar skill content en el prompt del sub-agente
4. Lanzar sub-agente
```

```
Archivos afectados:
- ~/.config/opencode/.gentle/orchestrator.md
```

#### Tarea 2.2: Implementar Standards Absorption

**Problema mencionado:** "Si tengo un nuevo estándar (ej. Arquitectura Hexagonal), me frustra si tengo que inyectarlo manualmente"

**Solución:** Sistema de "standards absorption" basado en contexto

**Implementación en `.gentle/context-protocol.md`:**
```markdown
## Standards Absorption

Cuando lances un sub-agente, detecta el contexto y carga standards relevantes:

### Detección de Contexto
- Si el archivo siendo editado está en `src/domain/` → cargar `standards/hexagonal-architecture.md`
- Si el archivo tiene `describe()` o `test()` → cargar `standards/testing-patterns.md`
- Si el archivo tiene `interface` o `abstract class` → cargar `standards/clean-architecture.md`

### Carga de Standards
1. Leer `.gentle/sdd.yaml` para ver qué standards están activos
2. Detectar contexto del trabajo actual
3. Cargar standards relevantes
4. Pasar en el prompt del sub-agente

Ejemplo:
```
SKILL: {contenido de skills/sdd-apply/SKILL.md}

STANDARDS:
{contenido de .gentle/standards/hexagonal-architecture.md}
{contenido de .gentle/standards/testing-patterns.md}
```
```

```
Archivos afectados:
- ~/.config/opencode/.gentle/context-protocol.md

Archivos a crear:
- ~/.config/opencode/.gentle/standards/hexagonal-architecture.md
- ~/.config/opencode/.gentle/standards/clean-architecture.md
- ~/.config/opencode/.gentle/standards/testing-patterns.md
```

**Impacto esperado después de Fase 2:**
- Skills se cargan dinámicamente según fase (NO auto-load global)
- Standards se absorben automáticamente según contexto
- Menos intervención manual del usuario

---

## Priorización (SIMPLIFICADA)

### HIGH Priority (Must Have - Fase 1)

| Tarea | Impacto | Esfuerzo | Justificación |
|-------|---------|----------|---------------|
| 1.1 Extraer orchestrator | Alto | Bajo | Reduce complejidad de AGENTS.md |
| 1.2 Crear .gentle/ | Alto | Bajo | Clarifica scope global vs local |
| 1.3 Actualizar opencode.json | Alto | Bajo | Conecta identidad + workflow |

### MEDIUM Priority (Should Have - Fase 2)

| Tarea | Impacto | Esfuerzo | Justificación |
|-------|---------|----------|---------------|
| 2.1 Contextual Loader | Alto | Medio | Elimina carga manual de skills |
| 2.2 Standards Absorption | Medio | Medio | Resuelve frustración principal |

---

## Tareas ELIMINADAS (Overengineering)

| Tarea Original | Por qué se eliminó |
|----------------|---------------------|
| Auto-load masivo de skills | Context pollution - carga 10+ skills en cada chat |
| Registry YAML | OpenCode ya tiene sistema de skills implícito |
| Eliminar prompts/sdd/ | Son pointers necesarios, NO duplicación |
| Fase 3 completa | Engram ya funciona, solo necesita enforcement en context-protocol |

---

## Tradeoffs

### Global vs Local

**Elegido:** Hybrid (identidad global + workflow local) por:
- Cada proyecto puede tener su propio flujo SDD
- La identidad del agente es consistente
- Bajo acoplamiento correcto

**Alternativa:** Todo global  
**Descartado porque:** Proyectos diferentes necesitan workflows diferentes

### Contextual Load vs Auto-Load

**Elegido:** Contextual (carga según fase) por:
- No contamina contexto con skills irrelevantes
- Performance mejor
- Más inteligente

**Alternativa:** Auto-load masivo  
**Descartado porque:** Context pollution

---

## Riesgos y Mitigaciones

| Riesgo | Likelihood | Impacto | Mitigación |
|--------|------------|---------|------------|
| Romper SDD workflow | Low | High | Verificar manualmente después de cada cambio |
| .gentle/ no se lee correctamente | Medium | High | Test con comando `/sdd-new test` |
| Standards no se cargan | Low | Medium | Logging en context-protocol |

---

## Métricas de Éxito

### Fase 1 (Separación)
- [ ] AGENTS.md reduce de 224 a ~60 líneas
- [ ] `.gentle/` directory existe con 4 archivos
- [ ] opencode.json lee `.gentle/orchestrator.md`
- [ ] Comando `/sdd-new test` funciona

### Fase 2 (Automatización)
- [ ] Skills se cargan dinámicamente (verificar logs)
- [ ] Standards se absorben según contexto
- [ ] Usuario NO tiene que cargar skills manualmente

---

## Comparación v1 vs v2

| Aspecto | v1 (Original) | v2 (Simplificada) |
|---------|---------------|-------------------|
| Fases | 3 (6 semanas) | 2 (2 semanas) |
| Tareas | 10 | 5 |
| Registry YAML | Sí | NO (overengineering) |
| Auto-load | Masivo (10+ skills) | NO (context pollution) |
| prompts/sdd/ | Eliminar | Mantener (son pointers) |
| Scope | Mixing | Clarificado (global vs local) |

---

## Preguntas Resueltas

1. **¿Mantener prompts/sdd/?** → SÍ, son pointers necesarios
2. **¿Qué estándares absorber?** → Hexagonal, Clean, Testing (en `.gentle/standards/`)
3. **¿YAML o JSON?** → YAML solo para `sdd.yaml` (config simple)
4. **¿Auto-load masivo?** → NO, carga contextual por fase

---

## Referencias

- OpenCode configuration: `~/.config/opencode/opencode.json`
- AGENTS.md actual: `~/.config/opencode/AGENTS.md`
- Skills: `~/.config/opencode/skills/sdd-*/SKILL.md`
- Prompts (pointers): `~/.config/opencode/prompts/sdd/*.md`
- PRD v1: `~/.config/opencode/prd-gentle-ai-improvements.md`