# Del Loop Manual a LangGraph: el mismo agente, otra arquitectura

> En el notebook anterior construiste el agentic loop **a mano**: un `for`, un `if finish_reason`, `messages.append(...)`.
> En este recorrido reescribes **exactamente el mismo agente** usando LangGraph.
> Las herramientas son idénticas — solo cambia cómo se orquesta el loop.

---

## Contenido

1. [La misma lógica, otra representación](#1-la-misma-lógica-otra-representación)
2. [El loop agéntico exacto en LangGraph](#2-el-loop-agéntico-exacto-en-langgraph)
3. [Componentes del notebook](#3-componentes-del-notebook)
4. [Abstracción de herramientas con `@tool`](#4-abstracción-de-herramientas-con-tool)
5. [Construcción del grafo](#5-construcción-del-grafo)

---

## 1. La misma lógica, otra representación

El agentic loop de `part01` y el grafo de `part02` implementan exactamente el mismo patrón **ReAct** (Reasoning + Acting): el modelo razona, decide actuar llamando una herramienta, observa el resultado, y vuelve a razonar.

La diferencia no es en *qué* hacen, sino en *dónde vive el control*.

En el loop manual, el flujo está codificado en Python imperativo: un `for`, un `if/elif`, llamadas a `append`. Si quieres cambiar el comportamiento — añadir un nodo de validación, bifurcar según el tipo de tool — tienes que modificar esa función.

En LangGraph, el flujo está codificado como **edges del grafo**. Cambiar el comportamiento significa añadir o redirigir edges, sin tocar la lógica de los nodos existentes.

![Loop manual vs LangGraph](figs/langgraph_implementation_01_manual_loop_vs_langgraph.svg)

> **La intuición clave:** en el loop manual el control flow *está en el código*. En LangGraph el control flow *es la estructura del grafo*. Eso lo hace inspeccionable, visualizable y modificable sin reescribir lógica.

---

## 2. El loop agéntico exacto en LangGraph

Visto desde arriba, el loop agéntico de LangGraph tiene tres actores y dos tipos de conexión:

- **Nodos** — ejecutan cómputo: `agent_node` llama al LLM, `tool_node` ejecuta herramientas
- **Edges** — definen el flujo: `tools → agent` es el ciclo; el edge condicional desde `agent` decide si continuar o terminar
- **`MessagesState`** — el estado compartido que viaja entre todos los nodos, creciendo con cada mensaje nuevo

El ciclo ocurre porque el edge `tools → agent` vuelve incondicionalmente al nodo `agent` después de ejecutar cualquier herramienta. El grafo termina cuando el router devuelve `"__end__"`.

![LangGraph agentic loop exacto](figs/langgraph_implementation_02_langgraph_agentic_loop_exact.svg)

> **Límite de iteraciones:** en el loop manual era `range(1, 11)`. En LangGraph se configura con `graph.compile(recursion_limit=10)`. Si se supera, LangGraph lanza `GraphRecursionError` en lugar de retornar silenciosamente.

---

## 3. Componentes del notebook

Antes de construir el grafo, conviene mapear cada pieza del notebook a su rol en el sistema. Son siete componentes que trabajan juntos:

| Componente | Rol |
|---|---|
| `ChatOpenAI` | Cliente LLM, compatible con cualquier backend |
| `@tool` | Decorador que convierte una función Python en una herramienta LLM-callable |
| `MessagesState` | Estado tipado que lleva la lista de mensajes entre nodos |
| `agent_node` | Nodo que invoca al LLM con el historial completo |
| `ToolNode` | Nodo que ejecuta las tool calls emitidas por el modelo |
| `router` | Función que decide el siguiente nodo: `tools` o `END` |
| `graph` | El `StateGraph` compilado — el agente ejecutable |

![Componentes del notebook LangGraph](figs/langgraph_agents_02_langgraph_notebook_components.svg)

> **Nota pedagógica:** en `part01` estos siete componentes estaban mezclados dentro de `run_agent()`. LangGraph los separa y los nombra — eso facilita entender, debuggear y extender el sistema.

---

## 4. Abstracción de herramientas con `@tool`

En `part01` cada herramienta requería dos cosas por separado: la función Python y un JSON schema escrito a mano. Con el decorador `@tool` de LangChain, el schema se genera **automáticamente** a partir del docstring y las type annotations.

```python
# part01 — schema manual (30+ líneas por herramienta)
def web_search(query: str) -> str:
    ...

TOOLS = [{"type": "function", "function": {"name": "web_search",
           "description": "...", "parameters": {...}}}]
TOOL_REGISTRY = {"web_search": web_search}

# part02 — @tool genera el schema solo
@tool
def web_search(query: str) -> str:
    """Busca información actualizada en la web."""
    ...

TOOLS = [web_search]   # el objeto es función + schema + nombre
```

El objeto resultante expone `.name`, `.description` y el JSON schema. `ToolNode` y `llm.bind_tools()` lo consumen directamente — sin `TOOL_REGISTRY` ni schemas manuales.

![Abstracción de herramientas con @tool](figs/langgraph_agents_03_langgraph_tool_abstraction.svg)

> **Lo que desaparece:** el `TOOL_REGISTRY` dict y los JSON schemas manuales. El decorador `@tool` hace ese trabajo inferiendo el schema desde el docstring y los tipos.

---

## 5. Construcción del grafo

El grafo se construye en cinco líneas. Cada línea corresponde a un concepto:

```python
builder = StateGraph(MessagesState)      # 1. declara el tipo de estado

builder.add_node("agent", agent_node)   # 2. nodo: llama al LLM
builder.add_node("tools", tool_node)    # 3. nodo: ejecuta herramientas

builder.add_edge(START, "agent")                    # 4. edges
builder.add_conditional_edges("agent", router)      #    router decide: tools | END
builder.add_edge("tools", "agent")                  #    después de tools, vuelve al agente

graph = builder.compile()               # 5. produce el agente ejecutable
```

`StateGraph(MessagesState)` le dice a LangGraph que el estado es una lista de mensajes. Eso es suficiente para que el framework gestione la acumulación del historial en cada iteración — sin `messages.append(...)` manual.

![Construcción del grafo](figs/langgraph_agents_04_langgraph_graph_construction.svg)

> **El ciclo está en los edges, no en el código.** `tools → agent` es la línea que reemplaza el `continue` del `for` loop de `part01`. La recursión termina cuando el router devuelve `END`.

---

*Continúa en → `02_langgraph_internals.md`*
