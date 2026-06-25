---
name: jira
description: Interface genérica para o Jira Cloud. Interpreta a intenção do usuário e direciona para o skill correto (jira-get, jira-comment, jira-subtask, jira-close).
argument-hint: "get PROJ-123 | comment PROJ-123 texto | subtask PROJ-123 \"título\" | close PROJ-123"
compatibility: Requer variáveis JIRA_BASE_URL, JIRA_EMAIL e JIRA_API_TOKEN no .env
metadata:
  author: local
user-invocable: true
disable-model-invocation: false
---

Você é uma interface para o Jira Cloud. Interprete os argumentos abaixo e execute a ação correspondente.

**Argumentos recebidos:** $ARGUMENTS

---

## Comandos disponíveis

| Intenção | Skill a usar | Argumentos |
|---|---|---|
| Ler/ver chamado | `/jira-get` | `PROJ-123` |
| Adicionar comentário | `/jira-comment` | `PROJ-123 texto do comentário` |
| Criar subtask | `/jira-subtask` | `PROJ-123 "título" ["descrição"]` |
| Fechar / transicionar | `/jira-close` | `PROJ-123 ["Nome da Transição"]` |

## Regras

1. Se os argumentos indicarem claramente um subcomando, execute-o diretamente usando o skill correspondente.
2. Se `$ARGUMENTS` estiver vazio ou for ambíguo, exiba a tabela acima e peça ao usuário para especificar.
3. Para fechar com transição específica: se o nome não for encontrado, liste as transições disponíveis e pergunte qual usar.

## Pré-requisitos (.env)

```
JIRA_BASE_URL=https://suaempresa.atlassian.net
JIRA_EMAIL=seu@email.com
JIRA_API_TOKEN=seu_token
```

Tokens podem ser gerados em: https://id.atlassian.com/manage-profile/security/api-tokens
