# Patrones

Plantillas reusables de cómo estructurar workflows en Cowork.

Cada pattern describe un tipo de problema recurrente y la estructura de Project (Instructions + Schedule + Contexto) que lo resuelve. Sirven como blueprint para armar tus propios Projects sin partir de cero.

## Roadmap de patterns

| Pattern                     | Para qué                                                                                       | Complejidad |
| --------------------------- | ---------------------------------------------------------------------------------------------- | ----------- |
| **Single-shot automation**  | Project simple: Instructions + 1 tarea `Manual`. Ejemplo: "resume mi inbox"                    | ⭐          |
| **Recurring delivery**      | Tarea con `Frequency: Daily/Weekly` + conector externo (Slack, email) para reportes recurrentes | ⭐⭐        |
| **Data → Document pipeline**| Lee contexto (CSV, sheets) → procesa con Skill → escribe output (`.docx`, `.pdf`, `.xlsx`)     | ⭐⭐        |
| **Knowledge base as context**| Project con Instructions que referencian una carpeta local + links/Drive como contexto read-only | ⭐⭐        |
| **Multi-agent research**    | Orquestador + subagentes paralelos + agregación final                                          | ⭐⭐⭐      |
| **Custom MCP integration**  | Conectar un servicio propio vía conector MCP custom y usarlo desde un Project                  | ⭐⭐⭐      |

## Estructura de cada pattern

Cada pattern se documentará en su propio archivo con las siguientes secciones:

1. **Cuándo usarlo** — el problema recurrente que resuelve.
2. **Estructura del Project** — cómo configurar Instructions, Schedule y Contexto.
3. **Ejemplo concreto** — un caso aterrizado end-to-end.
4. **Variantes y extensiones** — cómo adaptarlo a casos relacionados.

## Estado

🚧 **En desarrollo.** Los patterns se irán agregando uno por uno conforme se validen en uso real.
