---
name: jira-comment
description: Adiciona um comentário em um chamado Jira existente.
argument-hint: "PROJ-123 texto do comentário aqui"
compatibility: Requer variáveis JIRA_BASE_URL, JIRA_EMAIL e JIRA_API_TOKEN no .env
metadata:
  author: local
user-invocable: true
disable-model-invocation: false
---

Você é uma interface para o Jira Cloud. Adicione um comentário no chamado indicado.

**Argumentos:** $ARGUMENTS
**Formato esperado:** `PROJ-123 texto do comentário aqui`

## Instruções

1. Extraia a issue key (primeira palavra) e o texto do comentário (tudo depois da primeira palavra) dos argumentos.
2. Execute o comando Bash abaixo, substituindo `<ISSUE_KEY>` e `<TEXTO>` pelos valores extraídos.
3. Confirme o sucesso para o usuário.
4. Se ocorrer erro HTTP, mostre o código e a mensagem.

## Comando

```bash
set -a && source .env && set +a
export SK_ISSUE="<ISSUE_KEY>"
export SK_TEXT="<TEXTO>"
python - << 'PYEOF'
import os, json, base64, sys, io
from urllib import request, error

sys.stdout = io.TextIOWrapper(sys.stdout.buffer, encoding='utf-8')

key      = os.environ["SK_ISSUE"]
text     = os.environ["SK_TEXT"]
base_url = os.environ["JIRA_BASE_URL"].rstrip("/")
auth     = "Basic " + base64.b64encode(f"{os.environ['JIRA_EMAIL']}:{os.environ['JIRA_API_TOKEN']}".encode()).decode()
headers  = {"Authorization": auth, "Accept": "application/json", "Content-Type": "application/json"}

payload = {
    "body": {
        "type": "doc", "version": 1,
        "content": [{"type": "paragraph", "content": [{"type": "text", "text": text}]}]
    }
}

req = request.Request(
    f"{base_url}/rest/api/3/issue/{key}/comment",
    data=json.dumps(payload).encode(),
    method="POST",
    headers=headers,
)
try:
    with request.urlopen(req) as r:
        r.read()
    print(f"Comentário adicionado em {key}.")
except error.HTTPError as e:
    sys.exit(f"HTTP {e.code}: {e.read().decode()}")
PYEOF
```

## Pré-requisitos (.env)

```
JIRA_BASE_URL=https://suaempresa.atlassian.net
JIRA_EMAIL=seu@email.com
JIRA_API_TOKEN=seu_token
```
