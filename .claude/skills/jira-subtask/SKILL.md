---
name: jira-subtask
description: Cria uma subtask vinculada a um chamado pai no Jira, com título e descrição opcional.
argument-hint: "PROJ-123 \"título da subtask\" [\"descrição opcional\"]"
compatibility: Requer variáveis JIRA_BASE_URL, JIRA_EMAIL e JIRA_API_TOKEN no .env
metadata:
  author: local
user-invocable: true
disable-model-invocation: false
---

Você é uma interface para o Jira Cloud. Crie uma subtask vinculada ao chamado pai indicado.

**Argumentos:** $ARGUMENTS
**Formatos esperados:**
- `PROJ-123 "título da subtask"`
- `PROJ-123 "título da subtask" "descrição opcional"`

## Instruções

1. Extraia dos argumentos:
   - `parent_key`: primeira palavra (ex: PROJ-123)
   - `título`: segundo argumento (pode estar entre aspas)
   - `descrição`: terceiro argumento opcional (pode estar entre aspas)
2. Execute o comando Bash abaixo substituindo os valores.
3. Confirme a subtask criada com a key retornada.
4. Se ocorrer erro HTTP, mostre o código e a mensagem.

## Comando

```bash
set -a && source .env && set +a
export SK_PARENT="<PARENT_KEY>"
export SK_TITLE="<TÍTULO>"
export SK_DESC="<DESCRIÇÃO_OU_VAZIO>"
python - << 'PYEOF'
import os, json, base64, sys, io
from urllib import request, error

sys.stdout = io.TextIOWrapper(sys.stdout.buffer, encoding='utf-8')

parent   = os.environ["SK_PARENT"]
title    = os.environ["SK_TITLE"]
desc     = os.environ.get("SK_DESC", "").strip()
base_url = os.environ["JIRA_BASE_URL"].rstrip("/")
auth     = "Basic " + base64.b64encode(f"{os.environ['JIRA_EMAIL']}:{os.environ['JIRA_API_TOKEN']}".encode()).decode()
headers  = {"Authorization": auth, "Accept": "application/json", "Content-Type": "application/json"}

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

parent_data = api("GET", f"/rest/api/3/issue/{parent}?fields=project")
project_key = parent_data["fields"]["project"]["key"]

payload = {
    "fields": {
        "project":   {"key": project_key},
        "parent":    {"key": parent},
        "summary":   title,
        "issuetype": {"name": "Subtask"},
    }
}
if desc:
    payload["fields"]["description"] = {
        "type": "doc", "version": 1,
        "content": [{"type": "paragraph", "content": [{"type": "text", "text": desc}]}]
    }

result = api("POST", "/rest/api/3/issue", payload)
print(f"Subtask criada: {result.get('key')} — {title}")
PYEOF
```

## Pré-requisitos (.env)

```
JIRA_BASE_URL=https://suaempresa.atlassian.net
JIRA_EMAIL=seu@email.com
JIRA_API_TOKEN=seu_token
```
