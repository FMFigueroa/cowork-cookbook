# Introducción a Claude Cowork

> Punto de entrada al cookbook. Define qué es Cowork y cómo está organizado el resto del contenido.

**Última actualización:** 2026-04-22
**Autor:** Felix M. Figueroa · [@FMFigueroa](https://github.com/FMFigueroa)

---

## 📋 Índice

- [¿Qué es Cowork?](#-qué-es-cowork)
- [Cómo está organizado este cookbook](#-cómo-está-organizado-este-cookbook)
- [Por dónde empezar](#-por-dónde-empezar)

---

## 💡 ¿Qué es Cowork?

**Cowork es el módulo de Claude Desktop que te permite automatizar un trabajo con Claude mientras trabajas en otra cosa.**

Su arquitectura es similar al sistema agéntico de Claude Code.

**Básicamente, Cowork es un agente que vive en tu ordenador.** Corre en un entorno virtual y ejecuta trabajos de forma automatizada, sin que tengas que intervenir.

### Cómo trabaja (visión rápida)

Piénsalo como crear un proyecto:

1. **Planificas** lo que quieres obtener.
2. **Ejecutas** el plan.
3. **Obtienes** el resultado.

Puedes ver el proceso de ejecución y modificarlo en tiempo real si hace falta.

**Cowork es el project manager de tu proyecto.** Distribuye el trabajo en diferentes tareas y las ejecuta en paralelo a través de una orquestación de agentes.

> 🔎 **El detalle profundo** de cómo Cowork trabaja (Projects, Tasks, Task Loop, subagentes, entorno aislado) vive en [`02-projects-y-tasks.md`](./02-projects-y-tasks.md).

---

## 🗺️ Cómo está organizado este cookbook

El cookbook sigue una progresión pedagógica tipo receta de cocina — primero los fundamentos, después las herramientas, y al final las recetas que las combinan:

```
cowork-cookbook/
│
├── 00-foundations/              ← FUNDAMENTOS (estás aquí)
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
├── 02-recipes/                  ← RECETAS específicas end-to-end
│   └── 01-linkedin-content-factory/
│
└── 03-advanced/                 ← AVANZADO (roadmap)
```

### Qué contiene cada módulo

| Módulo | Propósito | Cuándo leerlo |
|---|---|---|
| **`00-foundations/`** | Núcleo conceptual: qué es Cowork, cómo trabaja un Project + Tasks, y las herramientas que puedes agregarle (Skills, Conectores, Plugins). | Primero. Es lo que hay que entender antes de construir. |
| **`01-patterns/`** | Patrones reutilizables que aplican transversalmente a distintos Projects (ej: manejo de secrets de MCPs). | Cuando ya dominas los fundamentos y necesitas resolver problemas recurrentes. |
| **`02-recipes/`** | Implementaciones concretas end-to-end — un Project real con sus Instructions, Tasks, Skills y Conectores configurados. | Cuando quieres inspirarte con casos reales o adaptar uno a tu escenario. |
| **`03-advanced/`** | Contenido avanzado (multi-agent orchestration, subagent design patterns, integraciones custom). | Cuando ya tienes varios Projects en producción. |

---

## 🎯 Por dónde empezar

### Si nunca usaste Cowork

1. Lee este doc (ya estás aquí ✅)
2. Sigue con [`02-projects-y-tasks.md`](./02-projects-y-tasks.md) — entender **Projects** y el **Task Loop** es lo que hace clic a todo lo demás.
3. Después, explora [`03-customize/`](./03-customize/) para ver las herramientas transversales que le das al agente.

### Si ya usaste Cowork y quieres profundizar

- Ve directo a [`02-projects-y-tasks.md`](./02-projects-y-tasks.md) para entender el Task Loop bajo el capó (subagentes, VM, steering).
- Revisa [`03-customize/02-conectores.md`](./03-customize/02-conectores.md) si te interesa el detalle del MCP (transport, LocalDev vs Desktop Extensions, etc.).

### Si estás buscando patrones/recetas específicas

- **Manejo seguro de secrets de MCPs** → [`01-patterns/mcp-secrets-management.md`](../01-patterns/mcp-secrets-management.md)
- **Projects reales end-to-end** → [`02-recipes/`](../02-recipes/)

---

## 🔗 Referencias externas

- [Anthropic Academy — curso oficial de Cowork](https://www.anthropic.com/learn/cowork)
- [Model Context Protocol spec](https://spec.modelcontextprotocol.io/) — el protocolo detrás de los Conectores
