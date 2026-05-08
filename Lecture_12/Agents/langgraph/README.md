# Agentes con LangGraph

Material de lectura para el módulo de agentes del programa MCDA · Universidad EAFIT.

---

## Estructura

Este módulo está dividido en tres documentos progresivos:

| Archivo | Enfoque | Cuándo leerlo |
|---|---|---|
| [`01_react_langgraph.md`](01_react_langgraph.md) | *Qué* cambia al pasar del loop manual a LangGraph | Antes de ver el notebook `part02` |
| [`02_langgraph_internals.md`](02_langgraph_internals.md) | *Cómo* funciona LangGraph por dentro | Mientras implementas o debuggeas |
| [`03_referencia_rapida.md`](03_referencia_rapida.md) | Tablas, skeleton y patrones de ejecución | Consulta rápida mientras codeas |

---

## Contexto del módulo

En `part01` construiste el agentic loop **a mano**: un `for`, un `if finish_reason`, `messages.append(...)`.
En `part02` reescribes **exactamente el mismo agente** usando LangGraph — las herramientas son idénticas, solo cambia cómo se orquesta el loop.

El patrón implementado es **ReAct** (Reasoning + Acting):

```
HumanMessage → agent_node → [tool_calls?] → tool_node → agent_node → ... → AIMessage final
```

---

## Figuras

Cada documento referencia sus figuras SVG. Las figuras están organizadas en dos sets:

**Set `langgraph_agents_` — visión general**
```
01_langgraph_react_architecture
02_langgraph_notebook_components
03_langgraph_tool_abstraction
04_langgraph_graph_construction
05_langgraph_router_conditional_edges
06_langgraph_messages_state
07_langgraph_stream_inspection
```

**Set `langgraph_implementation_` — internals**
```
01_manual_loop_vs_langgraph
02_langgraph_agentic_loop_exact
03_nodes_as_computation_units
04_router_conditional_edges
05_messages_state_evolution
06_tool_call_lifecycle_langgraph
07_compile_graph_lifecycle
08_streaming_inspection
09_master_langgraph_notebook_architecture
```

---

## Prerequisitos

- Haber completado `part01` (loop manual con OpenAI SDK)
- Python 3.10+
- `langgraph`, `langchain-openai`, `langchain-core`

```bash
pip install langgraph langchain-openai langchain-core
```

---

## Próximos pasos

El siguiente notebook extiende este mismo grafo con:

- **`MemorySaver`** — memoria persistente entre conversaciones
- **`interrupt_before`** — human-in-the-loop: el agente pausa antes de ejecutar una herramienta y espera confirmación

---

*Módulo de Agentes · Maestría en Ciencia de Datos y Analítica · EAFIT*
