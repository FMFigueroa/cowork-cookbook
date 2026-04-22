# 00-foundations

Núcleo conceptual del cookbook. Cubre qué es Cowork, cómo trabaja internamente (Projects + Task Loop) y las herramientas transversales que se le agregan al agente (Skills, Conectores, Plugins).

Orden de lectura sugerido:

| # | Archivo | Qué contiene |
|---|---|---|
| 1 | [`01-introduccion.md`](./01-introduccion.md) | Qué es Cowork + mapa del cookbook completo |
| 2 | [`02-projects-y-tasks.md`](./02-projects-y-tasks.md) | **Núcleo.** Project + Task Loop de 4 pasos + bajo el capó (subagentes, VM, steering) |
| 3 | [`03-customize/`](./03-customize/) | Herramientas transversales: **Skills**, **Conectores**, **Plugins** |

## Por qué este orden

1. **Primero los fundamentos** — qué es Cowork y cómo ejecuta cada Project (el Task Loop).
2. **Después las herramientas** — Skills, Conectores y Plugins son componentes que se le agregan al agente, y aplican **transversalmente a todos los Projects**. Por eso vienen en segundo plano, después del núcleo.

Los patterns reutilizables y las recetas concretas end-to-end viven en `01-patterns/` y `02-recipes/`, respectivamente.
