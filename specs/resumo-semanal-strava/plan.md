# Plano de Implementação — Resumo Semanal Strava

## Arquitetura

```
EventBridge Scheduler (cron semanal)
        ↓
AWS Lambda Python 3.11
        ├── StravaClient → Strava API v3
        ├── WeeklyMetrics (domínio puro)
        ├── SummaryPrompt (domínio puro)
        ├── OpenAIClient → OpenAI API
        └── TelegramClient → Telegram Bot API
```

Todos os secrets via AWS Secrets Manager (sem variáveis de ambiente diretas com valores sensíveis).

---

## Decisões Técnicas

| Decisão                  | Escolha             | Justificativa                                               |
|--------------------------|---------------------|-------------------------------------------------------------|
| Linguagem                | Python 3.11         | Requisito explícito; bom ecossistema para AWS Lambda        |
| HTTP Client              | `requests`          | Simples, sem async desnecessário para carga semanal         |
| OpenAI SDK               | `openai>=1.30`      | SDK oficial, chat completions                               |
| AWS SDK                  | `boto3`             | SDK padrão AWS Python                                       |
| Testes                   | `pytest` + `pytest-cov` | Padrão Python; cobertura integrada                      |
| Infra                    | Terraform >= 1.5    | IaC declarativo, módulos reutilizáveis                      |
| Scheduler                | EventBridge Scheduler | Mais moderno que CloudWatch Events; retry nativo          |
| Secrets                  | AWS Secrets Manager | Rotação nativa, integração IAM                              |
| CI/CD                    | GitHub Actions      | Integração nativa com GitHub                                |
| Auth AWS no CI           | OIDC                | Sem chaves estáticas                                        |
| Empacotamento Lambda     | `archive_file` TF   | Zip direto via Terraform, sem layer separado               |

---

## Estrutura de Diretórios

```
lambda-resumo-atividades-stava/
├── src/
│   ├── __init__.py
│   ├── lambda_handler.py              # Entrypoint AWS Lambda
│   ├── application/
│   │   ├── __init__.py
│   │   └── weekly_summary.py          # Orquestração do fluxo
│   ├── domain/
│   │   ├── __init__.py
│   │   ├── weekly_metrics.py          # Cálculo de métricas (puro)
│   │   └── summary_prompt.py          # Construção do prompt (puro)
│   └── infrastructure/
│       ├── __init__.py
│       ├── strava_client.py           # Cliente Strava API
│       ├── openai_client.py           # Cliente OpenAI API
│       ├── telegram_client.py         # Cliente Telegram Bot API
│       └── secrets.py                 # Acesso ao Secrets Manager
│
├── tests/
│   ├── __init__.py
│   └── unit/
│       ├── __init__.py
│       ├── test_weekly_metrics.py
│       ├── test_summary_prompt.py
│       ├── test_weekly_summary.py
│       ├── test_strava_client.py
│       ├── test_openai_client.py
│       ├── test_telegram_client.py
│       ├── test_secrets.py
│       └── test_lambda_handler.py
│
├── infra/
│   ├── main.tf
│   ├── variables.tf
│   ├── outputs.tf
│   ├── lambda.tf
│   ├── eventbridge.tf
│   ├── iam.tf
│   ├── secrets.tf
│   └── versions.tf
│
├── .github/
│   └── workflows/
│       ├── ci.yml
│       └── deploy.yml
│
├── pyproject.toml
├── README.md
└── .gitignore
```

---

## Mapeamento de User Stories → Componentes

| User Story | Componentes Envolvidos                                                    |
|------------|---------------------------------------------------------------------------|
| US-001     | `lambda_handler.py`, `eventbridge.tf`, `weekly_summary.py`               |
| US-002     | `weekly_metrics.py`, `summary_prompt.py`, `openai_client.py`             |
| US-003     | `weekly_summary.py` (branch sem atividades)                               |
| US-004     | `secrets.py`, `strava_client.py`, `openai_client.py`, `telegram_client.py` |
| US-005     | `infra/*.tf`                                                              |
| US-006     | `.github/workflows/ci.yml`, `.github/workflows/deploy.yml`               |

---

## Fluxo de Execução da Lambda

```
handler(event, context)
  └── WeeklySummaryService.executar()
        ├── StravaClient.buscar_atividades_semana()
        │     └── (renovar token se expirado)
        ├── [sem atividades] → TelegramClient.enviar_mensagem(msg_informativa)
        └── [com atividades]
              ├── WeeklyMetrics.calcular(atividades)
              ├── SummaryPrompt.construir(metricas)
              ├── OpenAIClient.gerar_resumo(prompt)
              └── TelegramClient.enviar_mensagem(resumo)
```

---

## Estratégia de Testes

- **Domínio** (`weekly_metrics`, `summary_prompt`): testes puros, sem mocks
- **Infrastructure** (`strava_client`, `openai_client`, `telegram_client`, `secrets`): mock das dependências externas (requests, OpenAI SDK, boto3)
- **Application** (`weekly_summary`): mock dos clientes de infraestrutura
- **Handler** (`lambda_handler`): mock do service
- Meta: ≥ 90% de cobertura validada pelo pytest-cov com `--cov-fail-under=90`

---

## Checkpoints de Entrega

1. Estrutura de pastas e arquivos `__init__.py`
2. Camada de domínio (`weekly_metrics.py`, `summary_prompt.py`) + testes
3. Camada de infraestrutura (4 clientes) + testes
4. Camada de aplicação (`weekly_summary.py`) + testes
5. `lambda_handler.py` + testes
6. `pyproject.toml` e verificação de cobertura ≥ 90%
7. Terraform (`infra/`)
8. GitHub Actions (`ci.yml`, `deploy.yml`)
9. `README.md` e `.gitignore`
