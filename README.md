# 📘 Cowork Cookbook

> Guía práctica para construir, configurar y operar workflows automatizados con **Claude Cowork** — el módulo de Claude Desktop que corre tareas de forma agéntica mientras trabajas en otra cosa.

**Autor:** Felix M. Figueroa · [@FMFigueroa](https://github.com/FMFigueroa)

---

## 🗺️ Estructura del cookbook

El contenido sigue una progresión pedagógica tipo **receta de cocina** — primero los fundamentos, después las herramientas, y al final las recetas que las combinan:

```
cowork-cookbook/
│
├── 00-foundations/              ← FUNDAMENTOS
│   ├── 01-introduccion.md       ← qué es Cowork + mapa del cookbook
│   ├── 02-projects-y-tasks.md   ← Project + Task Loop (núcleo de cómo trabaja)
│   └── 03-customize/            ← herramientas transversales del agente
│       ├── 01-skills.md
│       ├── 02-conectores.md
│       └── 03-plugins.md
│
├── 01-patterns/                 ← PATRONES reutilizables
│   └── mcp-secrets-management.md
│
├── 02-recipes/                  ← RECETAS concretas end-to-end
│   └── 01-linkedin-content-factory/
│
└── 03-advanced/                 ← AVANZADO (roadmap)
```

## 📚 Qué contiene cada módulo

| Módulo | Propósito | Cuándo leerlo |
|---|---|---|
| **[`00-foundations/`](./00-foundations/)** | Núcleo conceptual: qué es Cowork, cómo trabaja un Project + Tasks, y las herramientas que puedes agregarle (Skills, Conectores, Plugins). | Primero. Es lo que hay que entender antes de construir. |
| **[`01-patterns/`](./01-patterns/)** | Patrones reutilizables que aplican transversalmente a distintos Projects (ej: manejo de secrets de MCPs). | Cuando ya dominas los fundamentos y necesitas resolver problemas recurrentes. |
| **[`02-recipes/`](./02-recipes/)** | Implementaciones concretas end-to-end — un Project real con sus Instructions, Tasks, Skills y Conectores configurados. | Cuando quieres inspirarte con casos reales o adaptar uno a tu escenario. |
| **[`03-advanced/`](./03-advanced/)** | Contenido avanzado (multi-agent orchestration, subagent design patterns, integraciones custom). | Cuando ya tienes varios Projects en producción. |

## 🎯 Por dónde empezar

### Si nunca usaste Cowork

1. Empieza con [`00-foundations/01-introduccion.md`](./00-foundations/01-introduccion.md)
2. Sigue con [`02-projects-y-tasks.md`](./00-foundations/02-projects-y-tasks.md) — entender **Projects** y el **Task Loop** hace clic a todo lo demás
3. Después, explora [`03-customize/`](./00-foundations/03-customize/) para ver las herramientas transversales

### Si ya usaste Cowork y quieres profundizar

- [`02-projects-y-tasks.md`](./00-foundations/02-projects-y-tasks.md) — Task Loop bajo el capó (subagentes, VM, steering)
- [`03-customize/02-conectores.md`](./00-foundations/03-customize/02-conectores.md) — detalle del MCP (transport, LocalDev vs Desktop Extensions)

### Si buscas patrones/recetas específicas

- **Manejo seguro de secrets de MCPs** → [`01-patterns/mcp-secrets-management.md`](./01-patterns/mcp-secrets-management.md)
- **Projects reales end-to-end** → [`02-recipes/`](./02-recipes/)

---

## 🔗 Referencias externas

- [Anthropic Academy — curso oficial de Cowork](https://www.anthropic.com/learn/cowork)
- [Model Context Protocol spec](https://spec.modelcontextprotocol.io/) — el protocolo detrás de los Conectores

---

## 📜 Licencia

Ver [`LICENSE`](./LICENSE).
