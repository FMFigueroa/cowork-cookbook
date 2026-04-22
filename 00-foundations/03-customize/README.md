# 03-customize

**Customize** es la sección de Cowork donde viven las extensiones que potencian y personalizan al agente. Es el área de mayor valor del entorno: todo lo que hace a Cowork útil para tu caso específico se configura aquí.

Customize agrupa **tres tipos de extensiones**, cada una con su propio doc:

| # | Tipo | Qué es | Doc |
|---|---|---|---|
| 1 | **🧠 Skills** | Habilidades especializadas que el agente adopta para ejecutar tareas dentro de su entorno local. | [`01-skills.md`](./01-skills.md) |
| 2 | **🔌 Conectores** | MCP servers que conectan al agente con aplicaciones y servicios externos. | [`02-conectores.md`](./02-conectores.md) |
| 3 | **🧩 Plugins** | Kits curados del marketplace que combinan Skills + Conectores + Subagents bajo un mismo nombre. | [`03-plugins.md`](./03-plugins.md) |

## Por qué estas herramientas van en "segundo plano"

Porque **aplican transversalmente a todos los Projects**. No son el núcleo de cómo Cowork trabaja (eso es el [Project + Task Loop](../02-projects-y-tasks.md)) — son los componentes que le das al agente para que pueda hacer más cosas **dentro** de cualquier Project.

Piénsalo así:

- **Project + Tasks** = el motor de ejecución (dónde vive y cómo trabaja).
- **Customize** = el toolkit que extiende lo que el motor puede hacer (qué habilidades y qué integraciones tiene a mano).

Primero entendemos el motor, después el toolkit.
