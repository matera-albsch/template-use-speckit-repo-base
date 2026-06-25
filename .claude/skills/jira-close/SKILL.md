---
name: jira-close
description: Fecha ou transiciona um chamado Jira para um status específico. Usa transições padrão (Done/Closed/Resolvido/Fechado) se nenhuma for informada.
argument-hint: "PROJ-123 [\"Nome da Transição\"]"
compatibility: Requer variáveis JIRA_BASE_URL, JIRA_EMAIL e JIRA_API_TOKEN no .env
metadata:
  author: local
user-invocable: true
disable-model-invocation: false
---

Você é uma interface para o Jira Cloud. Feche ou transicione o chamado indicado.

**Argumentos:** $ARGUMENTS
**Formatos esperados:**
- `PROJ-123` — usa transição padrão (Done, Closed, Resolvido, Fechado ou Close)
- `PROJ-123 "Nome da Transição"` — usa a transição especificada

## Instruções

1. Extraia a issue key (primeira palavra) e a transição opcional (resto dos argumentos) dos argumentos.
2. Execute o comando Bash abaixo substituindo os valores.
3. Confirme a transição realizada para o usuário.
4. Se a transição não for encontrada, o script listará as disponíveis — exiba-as e pergunte qual usar.

## Comando

```bash
set -a && source .env && set +a
export SK_ISSUE="<ISSUE_KEY>"
export SK_TRANSITION="<NOME_DA_TRANSIÇÃO_OU_VAZIO>"
python - << 'PYEOF'
import os, json, base64, sys, io
from urllib import request, error

sys.stdout = io.TextIOWrapper(sys.stdout.buffer, encoding='utf-8')

key        = os.environ["SK_ISSUE"]
transition = os.environ.get("SK_TRANSITION", "").strip()
base_url   = os.environ["JIRA_BASE_URL"].rstrip("/")
auth       = "Basic " + base64.b64encode(f"{os.environ['JIRA_EMAIL']}:{os.environ['JIRA_API_TOKEN']}".encode()).decode()
headers    = {"Authorization": auth, "Accept": "application/json", "Content-Type": "application/json"}

def api(method, path, data=None):
    req = request.Request(
        base_url + path,
        data=json.dumps(data).encode() if data is not None else None,
        method=method,
        headers=headers,
    )
    try:
        with request.urlopen(req) as r:
            content = r.read()
            return json.loads(content) if content else {}
    except error.HTTPError as e:
        sys.exit(f"HTTP {e.code}: {e.read().decode()}")

transitions = api("GET", f"/rest/api/3/issue/{key}/transitions").get("transitions", [])
candidates  = [transition] if transition else ["Done", "Closed", "Resolvido", "Fechado", "Close"]

found = None
for name in candidates:
    found = next((t for t in transitions if t["name"].lower() == name.lower()), None)
    if found:
        break

if not found:
    names = ", ".join(t["name"] for t in transitions)
    sys.exit(f"Transição não encontrada. Disponíveis: {names}")

api("POST", f"/rest/api/3/issue/{key}/transitions", {"transition": {"id": found["id"]}})
print(f"Issue {key} transitada para '{found['name']}'.")
PYEOF
```

## Pré-requisitos (.env)

```
JIRA_BASE_URL=https://suaempresa.atlassian.net
JIRA_EMAIL=seu@email.com
JIRA_API_TOKEN=seu_token
```
