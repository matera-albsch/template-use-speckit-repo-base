---
name: jira-get
description: Busca e exibe o conteúdo completo de um chamado Jira (summary, status, prioridade, sprint, responsável, descrição, subtarefas e comentários).
argument-hint: "PROJ-123"
compatibility: Requer variáveis JIRA_BASE_URL, JIRA_EMAIL e JIRA_API_TOKEN no .env
metadata:
  author: local
user-invocable: true
disable-model-invocation: false
---

Você é uma interface para o Jira Cloud. Busque e exiba o chamado completo indicado nos argumentos.

**Argumentos:** $ARGUMENTS
**Formato esperado:** `PROJ-123`

## Instruções

1. Extraia a issue key dos argumentos (ex: PROJ-123 — sempre maiúsculas).
2. Execute o comando Bash abaixo, substituindo `<ISSUE_KEY>` pela key extraída.
3. Exiba a saída para o usuário.
4. Se ocorrer erro HTTP, mostre o código e a mensagem retornada.

## Comando

```bash
set -a && source .env && set +a
export SK_ISSUE="<ISSUE_KEY>"
python - << 'PYEOF'
import os, json, base64, sys, io
from urllib import request, error

sys.stdout = io.TextIOWrapper(sys.stdout.buffer, encoding='utf-8')

key      = os.environ["SK_ISSUE"]
base_url = os.environ["JIRA_BASE_URL"].rstrip("/")
auth     = "Basic " + base64.b64encode(f"{os.environ['JIRA_EMAIL']}:{os.environ['JIRA_API_TOKEN']}".encode()).decode()
headers  = {"Authorization": auth, "Accept": "application/json"}

def api_get(path):
    req = request.Request(base_url + path, headers=headers)
    try:
        with request.urlopen(req) as r:
            return json.loads(r.read().decode("utf-8"))
    except error.HTTPError as e:
        sys.exit(f"HTTP {e.code}: {e.read().decode()}")

def adf(node):
    if not node: return ""
    if node.get("type") == "text": return node.get("text", "")
    sep = "\n" if node.get("type") == "paragraph" else ""
    return sep.join(adf(c) for c in node.get("content", []))

fields = "summary,status,issuetype,assignee,reporter,priority,description,comment,customfield_10020,subtasks"
data = api_get(f"/rest/api/3/issue/{key}?fields={fields}")
f = data["fields"]

sprint_field = f.get("customfield_10020")
sprint = None
if sprint_field:
    active = next((s for s in sprint_field if s.get("state") == "active"), None)
    sprint = (active or sprint_field[-1]).get("name")

desc     = adf(f["description"]).strip() if f.get("description") else "(sem descrição)"
comments = f.get("comment", {}).get("comments", [])
subtasks = f.get("subtasks", [])

print(f"=== {data['key']}: {f.get('summary','')} ===")
print(f"URL: {base_url}/browse/{data['key']}")
print(f"\nTipo:         {f.get('issuetype',{}).get('name','')}")
print(f"Status:       {f.get('status',{}).get('name','')}")
print(f"Prioridade:   {f.get('priority',{}).get('name','Não definida')}")
print(f"Sprint:       {sprint or 'Nenhuma'}")
print(f"Responsável:  {(f.get('assignee') or {}).get('displayName','Não atribuído')}")
print(f"Reporter:     {(f.get('reporter') or {}).get('displayName','Desconhecido')}")
print(f"\n--- Descrição ---\n{desc}")

if subtasks:
    print(f"\n--- Subtarefas ({len(subtasks)}) ---")
    for s in subtasks:
        print(f"  [{s['key']}] {s['fields'].get('summary','')} — {s['fields'].get('status',{}).get('name','')}")
else:
    print("\n--- Subtarefas ---\n(nenhuma subtarefa)")

if comments:
    print(f"\n--- Comentários ({len(comments)}) ---")
    for c in comments:
        author  = (c.get("author") or {}).get("displayName", "Desconhecido")
        created = c.get("created", "")[:10]
        body    = adf(c["body"]).strip() if c.get("body") else ""
        print(f"\n[{author} — {created}]\n{body}")
else:
    print("\n--- Comentários ---\n(nenhum comentário)")
PYEOF
```

## Pré-requisitos (.env)

```
JIRA_BASE_URL=https://suaempresa.atlassian.net
JIRA_EMAIL=seu@email.com
JIRA_API_TOKEN=seu_token
```
