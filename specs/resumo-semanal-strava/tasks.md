# Tasks — Resumo Semanal Strava

## Checkpoint 1 — Estrutura base e domínio

### TASK-001 — Criar estrutura de pastas e arquivos `__init__.py`
- **US:** US-001, US-002
- **Arquivos:** `src/__init__.py`, `src/application/__init__.py`, `src/domain/__init__.py`, `src/infrastructure/__init__.py`, `tests/__init__.py`, `tests/unit/__init__.py`
- **Critério:** Módulos importáveis sem erro

### TASK-002 — Implementar `WeeklyMetrics` (domínio)
- **US:** US-002
- **Arquivo:** `src/domain/weekly_metrics.py`
- **Critério:** Cálculo correto de todas as métricas; `calcular()` recebe lista de dicts do Strava; `para_dict()` retorna dados serializáveis

### TASK-003 — Testes de `WeeklyMetrics`
- **US:** US-002
- **Arquivo:** `tests/unit/test_weekly_metrics.py`
- **Critério:** Cobertura ≥ 90% do módulo; casos: atividades variadas, sem corrida, corrida virtual, destaques, campos ausentes

### TASK-004 — Implementar `SummaryPrompt` (domínio)
- **US:** US-002
- **Arquivo:** `src/domain/summary_prompt.py`
- **Critério:** `construir()` retorna string com system prompt + dados JSON formatados

### TASK-005 — Testes de `SummaryPrompt`
- **US:** US-002
- **Arquivo:** `tests/unit/test_summary_prompt.py`
- **Critério:** Verifica presença do system prompt, dos dados e das instruções de estrutura

---

## Checkpoint 2 — Infraestrutura (clientes externos)

### TASK-006 — Implementar `SecretsManager` (infraestrutura)
- **US:** US-004
- **Arquivo:** `src/infrastructure/secrets.py`
- **Critério:** `obter_secrets_strava/openai/telegram()` com cache; sem logar valores; erro propagado

### TASK-007 — Testes de `SecretsManager`
- **US:** US-004
- **Arquivo:** `tests/unit/test_secrets.py`
- **Critério:** Mock do boto3; cache testado; erro `ClientError` propagado; modelo default

### TASK-008 — Implementar `StravaClient` (infraestrutura)
- **US:** US-001, US-002
- **Arquivo:** `src/infrastructure/strava_client.py`
- **Critério:** `buscar_atividades_semana()` busca últimos 7 dias; renova token se expirado

### TASK-009 — Testes de `StravaClient`
- **US:** US-001
- **Arquivo:** `tests/unit/test_strava_client.py`
- **Critério:** Token válido não renova; token expirado renova; erro HTTP propagado

### TASK-010 — Implementar `OpenAIClient` (infraestrutura)
- **US:** US-002
- **Arquivo:** `src/infrastructure/openai_client.py`
- **Critério:** `gerar_resumo(prompt)` chama chat completions; modelo configurável; erro propagado

### TASK-011 — Testes de `OpenAIClient`
- **US:** US-002
- **Arquivo:** `tests/unit/test_openai_client.py`
- **Critério:** Mock do SDK; modelo default; erro propagado

### TASK-012 — Implementar `TelegramClient` (infraestrutura)
- **US:** US-001, US-003
- **Arquivo:** `src/infrastructure/telegram_client.py`
- **Critério:** `enviar_mensagem(texto)` envia POST correto; erro HTTP propagado

### TASK-013 — Testes de `TelegramClient`
- **US:** US-001
- **Arquivo:** `tests/unit/test_telegram_client.py`
- **Critério:** Mock do requests; payload correto (chat_id, text); erro propagado

---

## Checkpoint 3 — Aplicação e Handler

### TASK-014 — Implementar `WeeklySummaryService` (application)
- **US:** US-001, US-002, US-003, US-004
- **Arquivo:** `src/application/weekly_summary.py`
- **Critério:** Fluxo completo com atividades; fluxo sem atividades; erros propagados; logs de início/fim/qtd

### TASK-015 — Testes de `WeeklySummaryService`
- **US:** US-001, US-002, US-003
- **Arquivo:** `tests/unit/test_weekly_summary.py`
- **Critério:** Sem atividades → mensagem informativa sem OpenAI; com atividades → resumo enviado; erros de cada cliente propagados

### TASK-016 — Implementar `lambda_handler`
- **US:** US-001
- **Arquivo:** `src/lambda_handler.py`
- **Critério:** `handler(event, context)` retorna `{"statusCode": 200, "body": ...}`; exceções propagadas para CloudWatch

### TASK-017 — Testes do `lambda_handler`
- **US:** US-001
- **Arquivo:** `tests/unit/test_lambda_handler.py`
- **Critério:** Sucesso retorna 200; exceção propagada; mock do service

---

## Checkpoint 4 — Configuração e Cobertura

### TASK-018 — Criar `pyproject.toml`
- **US:** US-006
- **Arquivo:** `pyproject.toml`
- **Critério:** Dependências corretas; `[tool.pytest.ini_options]` com `--cov-fail-under=90`; `[tool.coverage.run]` configurado

### TASK-019 — Verificar cobertura ≥ 90%
- **US:** US-006
- **Critério:** `pytest` passa com `--cov-fail-under=90` sem falhas

---

## Checkpoint 5 — Infraestrutura Terraform

### TASK-020 — `infra/versions.tf` — Providers e backend
- **US:** US-005
- **Arquivo:** `infra/versions.tf`

### TASK-021 — `infra/variables.tf` — Variáveis parametrizáveis
- **US:** US-005
- **Arquivo:** `infra/variables.tf`

### TASK-022 — `infra/secrets.tf` — AWS Secrets Manager
- **US:** US-004, US-005
- **Arquivo:** `infra/secrets.tf`
- **Critério:** Secret criado; placeholder com `lifecycle { ignore_changes }`

### TASK-023 — `infra/iam.tf` — IAM Roles e Policies (mínimo privilégio)
- **US:** US-004, US-005
- **Arquivo:** `infra/iam.tf`
- **Critério:** Lambda Role com logs + GetSecretValue; Scheduler Role com InvokeFunction

### TASK-024 — `infra/lambda.tf` — Lambda e CloudWatch Log Group
- **US:** US-001, US-005
- **Arquivo:** `infra/lambda.tf`
- **Critério:** Empacotamento via `archive_file`; env var `SECRET_NAME`; retenção de logs 30 dias

### TASK-025 — `infra/eventbridge.tf` — EventBridge Scheduler semanal
- **US:** US-001, US-005
- **Arquivo:** `infra/eventbridge.tf`
- **Critério:** Cron segunda às 9h BR; timezone `America/Sao_Paulo`; retry 2x

### TASK-026 — `infra/outputs.tf` e `infra/main.tf`
- **US:** US-005
- **Arquivos:** `infra/outputs.tf`, `infra/main.tf`

---

## Checkpoint 6 — CI/CD

### TASK-027 — `ci.yml` — Pipeline de CI
- **US:** US-006
- **Arquivo:** `.github/workflows/ci.yml`
- **Critério:** Executa em branches ≠ main e PRs para main; instala deps via uv; executa pytest com cobertura

### TASK-028 — `deploy.yml` — Pipeline de Deploy
- **US:** US-006
- **Arquivo:** `.github/workflows/deploy.yml`
- **Critério:** Executa em push na main; OIDC AWS; terraform init/plan/apply; necessita sucesso nos testes

---

## Checkpoint 7 — Documentação

### TASK-029 — `README.md` completo
- **Arquivo:** `README.md`
- **Critério:** Instruções de setup local, configuração de secrets, deploy manual, estrutura do projeto

### TASK-030 — `.gitignore` adequado
- **Arquivo:** `.gitignore`
- **Critério:** Ignora `__pycache__`, `.env`, `*.zip`, `infra/.terraform`, `infra/lambda_package.zip`
