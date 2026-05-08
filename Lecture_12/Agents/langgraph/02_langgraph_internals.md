# LangGraph por dentro: nodos, edges, estado y compilación

> El documento anterior mostró *qué* cambia al pasar de un loop manual a LangGraph.
> Este entra en *cómo* funciona LangGraph: qué es un nodo, qué hace el compilador,
> cómo fluye el estado y qué puedes observar en ejecución.

---

## Contenido

1. [Nodos como unidades de cómputo](#1-nodos-como-unidades-de-cómputo)
2. [Router y conditional edges](#2-router-y-conditional-edges)
3. [Evolución del MessagesState](#3-evolución-del-messagesstate)
4. [Ciclo de vida de una tool call](#4-ciclo-de-vida-de-una-tool-call)
5. [Construcción y compilación del grafo](#5-construcción-y-compilación-del-grafo)
6. [Inspección con streaming](#6-inspección-con-streaming)
7. [Arquitectura completa del notebook](#7-arquitectura-completa-del-notebook)

---

## 1. Nodos como unidades de cómputo

En LangGraph un **nodo** es cualquier callable que recibe el estado actual y devuelve una actualización parcial de ese estado. Nada más.

```python
# agent_node: recibe estado, devuelve nuevo mensaje del LLM
def agent_node(state: MessagesState) -> dict:
    messages = state["messages"]
    if not any(isinstance(m, SystemMessage) for m in messages):
        messages = [SystemMessage(content=SYSTEM_PROMPT)] + messages
    response = llm_with_tools.invoke(messages)
    return {"messages": [response]}   # ← actualización parcial, no reemplazo
```

LangGraph **acumula** las actualizaciones — no reemplaza el estado. Cuando `agent_node` devuelve `{"messages": [response]}`, ese mensaje se *añade* a la lista existente. Es el equivalente del `messages.append(assistant_msg)` de `part01`, pero automático.

`ToolNode` sigue exactamente el mismo contrato: recibe el estado, extrae los `tool_calls` del último `AIMessage`, ejecuta las funciones correspondientes, y devuelve `{"messages": [ToolMessage(...), ...]}`.

![Nodos como unidades de cómputo](figs/langgraph_implementation_03_nodes_as_computation_units.svg)

| Nodo | Input (del estado) | Output (actualización) |
|---|---|---|
| `agent_node` | `state["messages"]` completo | `{"messages": [AIMessage]}` |
| `ToolNode` | último `AIMessage` con `tool_calls` | `{"messages": [ToolMessage, ...]}` |

---

## 2. Router y conditional edges

El router es la función que reemplaza el `if finish_reason == "tool_calls"` del loop manual. Recibe el estado, inspecciona el último mensaje, y devuelve el nombre del siguiente nodo.

```python
def router(state: MessagesState) -> Literal["tools", "__end__"]:
    last = state["messages"][-1]
    if hasattr(last, "tool_calls") and last.tool_calls:
        return "tools"
    return "__end__"

# Registro en el grafo:
builder.add_conditional_edges("agent", router)
```

`add_conditional_edges` toma el nodo de origen (`"agent"`) y la función de routing. LangGraph evalúa el router *después de cada ejecución de `agent_node`* y enruta el flujo según el string devuelto.

![Router y conditional edges](figs/langgraph_implementation_04_router_conditional_edges.svg)

| `part01` | `part02` |
|---|---|
| `if finish_reason == "tool_calls"` | `router` devuelve `"tools"` |
| `elif finish_reason == "stop": return text` | `router` devuelve `"__end__"` |
| `else: break` | `recursion_limit` del grafo |

> **`"__end__"` vs `END`:** `END` es el nodo especial de LangGraph que termina el grafo. `"__end__"` es su nombre como string — ambos son equivalentes en el router.

---

## 3. Evolución del MessagesState

`MessagesState` es un `TypedDict` con una sola clave: `messages`. Pero tiene una propiedad importante: usa un **reducer de acumulación**. Cuando un nodo devuelve `{"messages": [nuevo]}`, LangGraph no reemplaza la lista — la extiende.

Esto es lo que hace posible que el grafo "recuerde" el historial sin que ningún nodo lo gestione explícitamente:

```
Inicio:       [HumanMessage("precio BTC")]
Tras agent:   [Human, AIMessage(tool_call: web_search)]
Tras tools:   [Human, AI, ToolMessage("BTC: $94,200")]
Tras agent:   [Human, AI, Tool, AIMessage(tool_call: calculate)]
Tras tools:   [Human, AI, Tool, AI, ToolMessage("70,650.0")]
Tras agent:   [Human, AI, Tool, AI, Tool, AIMessage("0.75 BTC costaría $70,650")]
```

Cada nodo solo añade su propio mensaje. Ningún nodo conoce ni gestiona el historial completo — esa es responsabilidad del framework.

![Evolución del MessagesState](figs/langgraph_implementation_05_messages_state_evolution.svg)

> **El LLM sigue siendo stateless.** `MessagesState` no es memoria del modelo — es el historial completo que se envía en cada llamada a `llm_with_tools.invoke(messages)`. La diferencia con `part01` es que LangGraph gestiona ese append automáticamente.

---

## 4. Ciclo de vida de una tool call

Cuando el LLM decide usar una herramienta, emite un `AIMessage` con `tool_calls`. `ToolNode` toma ese mensaje, extrae cada tool call, despacha a la función Python correspondiente, y empaqueta los resultados como `ToolMessage` con el `tool_call_id` correcto.

Todo esto ocurre **dentro del nodo `tools`**:

```python
# part01 — manual (dentro del loop)
for tc in assistant_msg.tool_calls:
    tool_name = tc.function.name
    tool_args = json.loads(tc.function.arguments)
    fn = tool_registry.get(tool_name)
    result = fn(**tool_args)
    messages.append({"role": "tool", "tool_call_id": tc.id, "content": result})

# part02 — ToolNode hace todo esto automáticamente
tool_node = ToolNode(TOOLS)
builder.add_node("tools", tool_node)
```

`ToolNode` también maneja **tool calls paralelas**: si el LLM emite múltiples `tool_calls` en un solo `AIMessage`, las ejecuta todas y devuelve un `ToolMessage` por cada una.

![Ciclo de vida de una tool call en LangGraph](figs/langgraph_implementation_06_tool_call_lifecycle_langgraph.svg)

> **Lo que `ToolNode` abstrae:** dispatch por nombre, parseo de argumentos (`json.loads`), empaquetado del resultado (`role: tool`, `tool_call_id`), y soporte para tool calls paralelas. En `part01` esas ~8 líneas eran explícitas; aquí son una línea.

---

## 5. Construcción y compilación del grafo

LangGraph separa dos fases que en el loop manual eran una sola: **declaración** y **ejecución**.

```python
# Fase 1: declaración — describe la estructura, no ejecuta nada
builder = StateGraph(MessagesState)
builder.add_node("agent", agent_node)
builder.add_node("tools", tool_node)
builder.add_edge(START, "agent")
builder.add_conditional_edges("agent", router)
builder.add_edge("tools", "agent")

# Fase 2: compilación — valida la estructura y produce el runnable
graph = builder.compile()
```

`builder.compile()` valida que el grafo sea coherente (todos los nodos referenciados existen, hay un camino desde `START` hasta `END`, no hay edges huérfanos), genera el plan de ejecución interno, y devuelve un objeto `CompiledGraph` con los métodos `.invoke()`, `.stream()`, y `.get_graph()`.

El grafo compilado es **reutilizable e inmutable** — puedes invocarlo miles de veces. Para cambiar el comportamiento hay que declarar uno nuevo y compilarlo.

![Construcción y compilación del grafo](figs/langgraph_implementation_07_compile_graph_lifecycle.svg)

> **`recursion_limit`:** se configura en la compilación con `graph = builder.compile(recursion_limit=15)`. El valor por defecto es 25. Superar el límite lanza `GraphRecursionError`.

---

## 6. Inspección con streaming

`graph.stream()` hace visible la ejecución nodo a nodo. Dos modos útiles:

```python
# stream_mode="values" — estado completo tras cada nodo
for step in graph.stream({"messages": [HumanMessage(query)]}, stream_mode="values"):
    last = step["messages"][-1]
    if isinstance(last, AIMessage) and last.tool_calls:
        print(f"🔧 {last.tool_calls[0]['name']}(...)")
    elif isinstance(last, ToolMessage):
        print(f"📥 {last.content[:100]}")

# stream_mode="updates" — solo el delta de cada nodo (útil para enseñanza)
for node_name, update in graph.stream({"messages": [HumanMessage(query)]}, stream_mode="updates"):
    print(f"Nodo: {node_name} → {update}")
```

Para inspeccionar el historial completo al terminar:

```python
final_state = graph.invoke({"messages": [HumanMessage(query)]})
for msg in final_state["messages"]:
    print(type(msg).__name__, ":", str(msg.content)[:100])
```

Para visualizar el grafo antes de ejecutarlo:

```python
from IPython.display import Image, display
display(Image(graph.get_graph().draw_mermaid_png()))
```

![Inspección del grafo con streaming](figs/langgraph_implementation_08_streaming_inspection.svg)

> **Útil para enseñanza:** `stream_mode="updates"` muestra qué hace cada nodo y en qué orden. Ver `agent → tools → agent → __end__` en tiempo real hace el loop agéntico tangible.

---

## 7. Arquitectura completa del notebook

La figura siguiente muestra todos los componentes como un mapa de referencia: desde el cliente LLM y las herramientas decoradas con `@tool`, pasando por los nodos y el router, hasta el grafo compilado y los métodos de ejecución e inspección.

![Arquitectura completa del notebook LangGraph](figs/langgraph_implementation_09_master_langgraph_notebook_architecture.svg)

---

*← `01_react_langgraph.md` · Referencia rápida → `03_referencia_rapida.md`*
