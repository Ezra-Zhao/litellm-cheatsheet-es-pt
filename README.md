# 🚪 Guía rápida: LiteLLM — un solo endpoint para 100+ modelos (ES · PT)

Una sola página para equipos que pagan facturas de API: **LiteLLM** (BerriAI) expone 100+ APIs de modelos con el formato de OpenAI, con enrutado, *fallbacks*, límites de presupuesto y registro de gasto.

**Autor:** Ezra-Zhao · Octubre 2026

Proyecto original: https://github.com/BerriAI/litellm · Documentación: https://docs.litellm.ai (más de 61.000 estrellas en octubre de 2026)

> **Nota de autoría:** guía original de síntesis y ejemplos propios del autor; no reproduce la documentación del proyecto. El código de ejemplo es mínimo y está pensado para adaptarse.

---

## 🇪🇸 Español

### Instalación

```bash
pip install litellm              # SDK de Python
pip install "litellm[proxy]"     # + servidor proxy para equipos
```

Llamada directa desde Python (sin proxy):

```python
from litellm import completion

r = completion(
    model="openai/gpt-4o-mini",   # prefijo de proveedor + modelo
    messages=[{"role": "user", "content": "Resume este texto en 3 líneas: ..."}],
)
print(r.choices[0].message.content)
print(r._hidden_params["response_cost"])  # lo que costó ESTA llamada
```

### `config.yaml` mínimo para el proxy

```yaml
model_list:
  - model_name: barato              # alias TUYO, estable
    litellm_params:
      model: openai/gpt-4o-mini
      api_key: os.environ/OPENAI_API_KEY
  - model_name: fuerte              # otro alias tuyo
    litellm_params:
      model: anthropic/claude-sonnet-4-5
      api_key: os.environ/ANTHROPIC_API_KEY

router_settings:
  routing_strategy: simple-shuffle  # reparte entre despliegues del mismo alias
  fallbacks:
    - barato: ["fuerte"]            # si «barato» cae, sigue con «fuerte»

general_settings:
  master_key: os.environ/LITELLM_MASTER_KEY   # protege el proxy. SIEMPRE.
```

```bash
litellm --config config.yaml --port 4000
# Tu equipo ahora llama a http://localhost:4000 con model: "barato" o "fuerte"
```

### La tabla

| Tarea | Cómo, en una línea | ⚠️ Error típico |
|---|---|---|
| Llamar a cualquier proveedor | SDK: `completion(model="proveedor/modelo", ...)` — mismo código para OpenAI, Anthropic, Bedrock, Vertex, etc. | Olvidar el prefijo de proveedor en el nombre del modelo |
| Un endpoint para el equipo | Proxy: `litellm --config config.yaml` y todos llaman a `localhost:4000` en formato OpenAI | Levantar el proxy sin `master_key`: queda abierto a quien vea tu red |
| Alias estables | En `model_list`, `model_name` es tu alias; el modelo real vive en `litellm_params` | Usar el nombre real del proveedor en el código del equipo: cambiar de modelo obliga a tocar todo el código |
| Fallback automático | `router_settings.fallbacks`: si el barato falla, pasa al siguiente de la lista | Fallback a un modelo mucho más caro sin techo de gasto: una caída de un día se convierte en factura sorpresa |
| Balanceo de carga | Varias entradas con el mismo `model_name` + `routing_strategy: simple-shuffle` | Balancear sin mirar cuotas: dos despliegues con la misma clave comparten el mismo límite |
| Presupuesto por persona/proyecto | Con el proxy: genera claves virtuales (`/key/generate`) con `max_budget` por clave | Dar la clave maestra al equipo «por comodidad»: pierdes el control por clave |
| Ver el gasto | El proxy acumula gasto por clave; míralo en su panel (`/ui`) y en `response_cost` por llamada en el SDK | El gasto persistente del proxy necesita base de datos (`DATABASE_URL`); sin ella, se pierde al reiniciar |
| Claves de proveedor | En el YAML usa `os.environ/NOMBRE` para leerlas del entorno | Pegar claves reales en el `config.yaml` y subirlo a git |
| Reintentos y tiempos | `num_retries` y `timeout` por despliegue en `litellm_params` | Reintentos altos + fallback + modelo caro: la tormenta perfecta de gasto |

### Elegir modelo por tarea (la parte que paga la factura)

| Tipo de tarea | Qué usar | Por qué |
|---|---|---|
| Clasificar, extraer, resumir, traducir borradores | El alias **barato** (clase mini/flash/lite) | Son tareas de patrón, no de razonamiento profundo; cuestan una fracción |
| Chat con clientes con guion fijo | El alias **barato** + fallback al fuerte | El 95 % de los casos sale bien con el barato; el fallback cubre el resto |
| Código difícil, agentes con herramientas, análisis largo | El alias **fuerte** (clase frontera) | Aquí un error del modelo cuesta más que el ahorro en tokens |
| Trabajo por lotes de madrugada | El alias **barato**, sin fallback | Si falla, se reintenta mañana; nadie está mirando |

Regla práctica: **empieza todo por el barato y sube de alias solo donde midas errores**, no al revés.

---

## 🇵🇹 Português

### Instalação

```bash
pip install litellm              # SDK de Python
pip install "litellm[proxy]"     # + servidor proxy para equipes
```

Chamada direta do Python (sem proxy):

```python
from litellm import completion

r = completion(
    model="openai/gpt-4o-mini",   # prefixo do provedor + modelo
    messages=[{"role": "user", "content": "Resuma este texto em 3 linhas: ..."}],
)
print(r.choices[0].message.content)
print(r._hidden_params["response_cost"])  # quanto custou ESTA chamada
```

### `config.yaml` mínimo para o proxy

```yaml
model_list:
  - model_name: barato              # alias SEU, estável
    litellm_params:
      model: openai/gpt-4o-mini
      api_key: os.environ/OPENAI_API_KEY
  - model_name: forte               # outro alias seu
    litellm_params:
      model: anthropic/claude-sonnet-4-5
      api_key: os.environ/ANTHROPIC_API_KEY

router_settings:
  routing_strategy: simple-shuffle  # distribui entre deploys do mesmo alias
  fallbacks:
    - barato: ["forte"]             # se o «barato» cair, continua no «forte»

general_settings:
  master_key: os.environ/LITELLM_MASTER_KEY   # protege o proxy. SEMPRE.
```

```bash
litellm --config config.yaml --port 4000
# Sua equipe agora chama http://localhost:4000 com model: "barato" ou "forte"
```

### A tabela

| Tarefa | Como, em uma linha | ⚠️ Erro típico |
|---|---|---|
| Chamar qualquer provedor | SDK: `completion(model="provedor/modelo", ...)` — o mesmo código para OpenAI, Anthropic, Bedrock, Vertex etc. | Esquecer o prefixo do provedor no nome do modelo |
| Um endpoint para a equipe | Proxy: `litellm --config config.yaml` e todos chamam `localhost:4000` no formato OpenAI | Subir o proxy sem `master_key`: fica aberto para quem enxergar sua rede |
| Aliases estáveis | Em `model_list`, `model_name` é o seu alias; o modelo real mora em `litellm_params` | Usar o nome real do provedor no código da equipe: trocar de modelo obriga a mexer no código inteiro |
| Fallback automático | `router_settings.fallbacks`: se o barato falhar, passa para o próximo da lista | Fallback para um modelo muito mais caro sem teto de gasto: uma queda de um dia vira fatura surpresa |
| Balanceamento de carga | Várias entradas com o mesmo `model_name` + `routing_strategy: simple-shuffle` | Balancear sem olhar cotas: dois deploys com a mesma chave dividem o mesmo limite |
| Orçamento por pessoa/projeto | No proxy: gere chaves virtuais (`/key/generate`) com `max_budget` por chave | Entregar a chave mestra para a equipe «por praticidade»: você perde o controle por chave |
| Ver o gasto | O proxy acumula gasto por chave; veja no painel dele (`/ui`) e no `response_cost` por chamada no SDK | O gasto persistente do proxy precisa de banco de dados (`DATABASE_URL`); sem ele, some ao reiniciar |
| Chaves dos provedores | No YAML, use `os.environ/NOME` para lê-las do ambiente | Colar chaves de verdade no `config.yaml` e subir para o git |
| Retentativas e tempos | `num_retries` e `timeout` por deploy em `litellm_params` | Retentativas altas + fallback + modelo caro: a tempestade perfeita de gasto |

### Escolher modelo por tarefa (a parte que paga a fatura)

| Tipo de tarefa | O que usar | Por quê |
|---|---|---|
| Classificar, extrair, resumir, traduzir rascunhos | O alias **barato** (classe mini/flash/lite) | São tarefas de padrão, não de raciocínio profundo; custam uma fração |
| Chat com clientes com roteiro fixo | O alias **barato** + fallback para o forte | 95 % dos casos saem bem com o barato; o fallback cobre o resto |
| Código difícil, agentes com ferramentas, análise longa | O alias **forte** (classe fronteira) | Aqui um erro do modelo custa mais que a economia em tokens |
| Trabalho em lote de madrugada | O alias **barato**, sem fallback | Se falhar, tenta amanhã de novo; ninguém está olhando |

Regra prática: **comece tudo pelo barato e suba de alias só onde você medir erros**, nunca o contrário.

---

## 🇬🇧 English summary

One-page cheat sheet (Spanish + Portuguese) for **LiteLLM** (BerriAI): one OpenAI-format endpoint for 100+ model APIs, aimed at teams/production. Covers the Python SDK one-liner with per-call cost, a minimal `config.yaml` (stable aliases, fallbacks, master key), the main table (routing, load balancing, per-key budgets via virtual keys, spend tracking, secret handling), and a task→model-class table for keeping the API bill down. Original examples and synthesis by the author.

**Author:** Ezra-Zhao · October 2026 · MIT License
