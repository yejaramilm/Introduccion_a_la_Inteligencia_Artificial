# Referencia rápida: LangGraph

> Consulta rápida mientras codeas. Sin prosa — solo tablas, snippets y patrones.

---

## part01 vs part02: equivalencias exactas

| Concepto LangGraph | Equivalente en el loop manual |
|---|---|
| `StateGraph(MessagesState)` | La lista `messages` + el `for` loop |
| `agent_node` | El bloque `client.chat.completions.create(...)` |
| `ToolNode(TOOLS)` | El bloque `for tc in tool_calls: fn(**args); append(...)` |
| `router` | El `if finish_reason == "tool_calls"` |
| `add_conditional_edges` | La bifurcación `if/elif/else` |
| `add_edge("tools", "agent")` | El `continue` implícito del loop |
| `builder.compile()` | No existe — el loop siempre está "compilado" |
| `graph.stream(...)` | Instrumentación manual con `print` en cada paso |
| `recursion_limit` | El `range(1, 11)` del `for` |

---

## part01 vs part02: dimensiones del sistema

| Aspecto | `part01` — from scratch | `part02` — LangGraph |
|---|---|---|
| **Loop** | `for iteration in range(10)` explícito | `StateGraph` con ciclo implícito |
| **Tools** | JSON schema manual + `TOOL_REGISTRY` dict | `@tool` genera schema; `ToolNode` ejecuta |
| **Estado** | `messages.append(...)` manual | `MessagesState` gestionado automáticamente |
| **Router** | `if finish_reason == "tool_calls"` | función `router` + `add_conditional_edges` |
| **Max iteraciones** | `range(1, 11)` | `graph.compile(recursion_limit=10)` |
| **Streaming** | No nativo | `graph.stream(...)` |
| **Visualización** | No | `graph.get_graph().draw_mermaid_png()` |
| **Extensibilidad** | Modificar el loop | Añadir nodos y edges |

---

## Contratos de los nodos

| Nodo | Input (del estado) | Output (actualización) |
|---|---|---|
| `agent_node` | `state["messages"]` completo | `{"messages": [AIMessage]}` |
| `ToolNode` | último `AIMessage` con `tool_calls` | `{"messages": [ToolMessage, ...]}` |

---

## Skeleton completo

```python
from langgraph.graph import StateGraph, MessagesState, START, END
from langgraph.prebuilt import ToolNode
from langchain_core.tools import tool
from langchain_openai import ChatOpenAI
from langchain_core.messages import SystemMessage, AIMessage, ToolMessage
from typing import Literal

# 1. Herramientas
@tool
def web_search(query: str) -> str:
    """Busca información actualizada en la web."""
    ...

@tool
def calculate(expression: str) -> str:
    """Evalúa una expresión matemática."""
    ...

TOOLS = [web_search, calculate]

# 2. LLM con tools
llm = ChatOpenAI(model="gpt-4o-mini")
llm_with_tools = llm.bind_tools(TOOLS)

SYSTEM_PROMPT = "Eres un asistente útil con acceso a herramientas."

# 3. Nodos
def agent_node(state: MessagesState) -> dict:
    messages = state["messages"]
    if not any(isinstance(m, SystemMessage) for m in messages):
        messages = [SystemMessage(content=SYSTEM_PROMPT)] + messages
    response = llm_with_tools.invoke(messages)
    return {"messages": [response]}

tool_node = ToolNode(TOOLS)

# 4. Router
def router(state: MessagesState) -> Literal["tools", "__end__"]:
    last = state["messages"][-1]
    if hasattr(last, "tool_calls") and last.tool_calls:
        return "tools"
    return "__end__"

# 5. Grafo
builder = StateGraph(MessagesState)
builder.add_node("agent", agent_node)
builder.add_node("tools", tool_node)
builder.add_edge(START, "agent")
builder.add_conditional_edges("agent", router)
builder.add_edge("tools", "agent")
graph = builder.compile(recursion_limit=10)
```

---

## Patrones de ejecución

```python
from langchain_core.messages import HumanMessage

# Invoke — retorna estado final
final_state = graph.invoke({"messages": [HumanMessage(content=query)]})
print(final_state["messages"][-1].content)

# Stream values — estado completo tras cada nodo
for step in graph.stream({"messages": [HumanMessage(content=query)]}, stream_mode="values"):
    last = step["messages"][-1]
    if isinstance(last, AIMessage) and last.tool_calls:
        print(f"🔧 {last.tool_calls[0]['name']}(...)")
    elif isinstance(last, ToolMessage):
        print(f"📥 {last.content[:120]}")
    elif isinstance(last, AIMessage):
        print(f"✅ {last.content}")

# Stream updates — solo delta por nodo (ideal para debugging en clase)
for node_name, update in graph.stream({"messages": [HumanMessage(content=query)]}, stream_mode="updates"):
    print(f"[{node_name}] {update}")

# Visualizar el grafo
from IPython.display import Image, display
display(Image(graph.get_graph().draw_mermaid_png()))
```

---

## Lo que LangGraph añade (que el loop manual no tiene)

- **Visualización** del grafo como diagrama Mermaid o PNG
- **Streaming nativo** sin instrumentar el loop
- **Validación estructural** en compilación (detecta grafos rotos antes de ejecutar)
- **Extensiones declarativas** sin tocar la lógica de los nodos:
  - Memoria persistente → `graph.compile(checkpointer=MemorySaver())`
  - Human-in-the-loop → `graph.compile(interrupt_before=["tools"])`
  - Subgrafos → nodo que es a su vez un `CompiledGraph`

---

*← `02_langgraph_internals.md`*
