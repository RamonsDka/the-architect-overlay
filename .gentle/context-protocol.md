# Context Protocol & Standards Absorption

## 1. Contextual Skill Loading
Antes de delegar a un sub-agente SDD, el orquestador DEBE inyectar la skill correspondiente a la fase.
NO cargues skills que no correspondan a la fase actual (evitar context pollution).

| Fase SDD | Skill requerida (Pointer) |
|----------|---------------------------|
| explore  | `~/.config/opencode/prompts/sdd/sdd-explore.md` |
| propose  | `~/.config/opencode/prompts/sdd/sdd-propose.md` |
| spec     | `~/.config/opencode/prompts/sdd/sdd-spec.md`    |
| design   | `~/.config/opencode/prompts/sdd/sdd-design.md`  |
| tasks    | `~/.config/opencode/prompts/sdd/sdd-tasks.md`   |
| apply    | `~/.config/opencode/prompts/sdd/sdd-apply.md`   |
| verify   | `~/.config/opencode/prompts/sdd/sdd-verify.md`  |

## 2. Standards Absorption
Verificá el archivo `.gentle/sdd.yaml` para conocer las `conventions` activas del proyecto.
Si el sub-agente va a escribir o analizar código, inyectá el contenido de `.gentle/standards/{convention}.md` como contexto adicional en el prompt de delegación.
