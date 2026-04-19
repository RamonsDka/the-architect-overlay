# Gentle-AI Template

Este template contiene la estructura base para usar el ecosistema Gentle-AI + OpenCode + SDD en cualquier proyecto.

## Estructura

```
.gentle/
├── orchestrator.md          # Reglas de orquestación SDD
├── context-protocol.md      # Carga contextual de skills y standards
├── sdd.yaml.template        # Configuración del proyecto (renombrar a sdd.yaml)
└── standards/               # Convenciones arquitectónicas
    ├── README.md
    ├── hexagonal-architecture.md
    ├── clean-architecture.md
    └── testing-patterns.md
```

## Cómo usar este template

### 1. Copiar a tu proyecto

```bash
cp -r ~/.config/opencode/templates/.gentle/ {tu-proyecto}/.gentle/
```

### 2. Configurar sdd.yaml

Renombrar `sdd.yaml.template` a `sdd.yaml` y ajustar:

```yaml
artifact_store: engram  # o openspec, hybrid, none
project: tu-proyecto-nombre
conventions:
  - hexagonal-architecture  # Activar/desactivar según necesites
  - clean-architecture
  - testing-patterns
```

### 3. Ajustar standards (opcional)

Si tu proyecto tiene convenciones específicas, editá los archivos en `standards/` o agregá nuevos.

### 4. Verificar wiring en opencode.json

Asegurate de que el `sdd-orchestrator` lea ambos archivos:

```json
"sdd-orchestrator": {
  "prompt": "{file:./AGENTS.md}\n\n{file:.gentle/orchestrator.md}"
}
```

## Arquitectura

- **Global** (`~/.config/opencode/`): Identidad del agente (AGENTS.md), skills, prompts
- **Local** (`{proyecto}/.gentle/`): Workflow SDD, standards específicos del proyecto

## Beneficios

- ✅ Separación de responsabilidades (identidad vs workflow)
- ✅ Carga contextual de skills (sin context pollution)
- ✅ Standards absorption automática
- ✅ Configuración por proyecto
- ✅ Replicable en cualquier repo
