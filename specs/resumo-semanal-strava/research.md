# Research — Resumo Semanal Strava

## Tech Stack

### Runtime
- **Python 3.11** — Versão LTS suportada pelo AWS Lambda (até Nov 2026); tipagem melhorada; bom desempenho

### Dependências de produção
| Biblioteca    | Versão mínima | Uso                              |
|---------------|---------------|----------------------------------|
| `boto3`       | 1.34.0        | AWS SDK: Secrets Manager, Lambda |
| `openai`      | 1.30.0        | OpenAI Chat Completions API      |
| `requests`    | 2.32.0        | HTTP: Strava API, Telegram API   |

### Dependências de desenvolvimento
| Biblioteca    | Versão mínima | Uso                                    |
|---------------|---------------|----------------------------------------|
| `pytest`      | 8.0.0         | Framework de testes                    |
| `pytest-cov`  | 5.0.0         | Cobertura de código                    |
| `pytest-mock` | 3.12.0        | Fixtures de mock para pytest           |

### Infraestrutura AWS
| Serviço                   | Versão/Tier | Uso                                           |
|---------------------------|-------------|-----------------------------------------------|
| AWS Lambda                | Python 3.11 | Execução do código                            |
| EventBridge Scheduler     | Standard    | Agendamento semanal com retry                 |
| CloudWatch Logs           | —           | Observabilidade e diagnóstico                 |
| AWS Secrets Manager       | —           | Armazenamento seguro de secrets               |
| IAM                       | —           | Permissões mínimas para Lambda e Scheduler    |

### Terraform
- Versão mínima: **1.5.0**
- Provider AWS: **~> 5.0**
- Backend: S3 (bucket configurável)

### GitHub Actions
- `actions/checkout@v4`
- `actions/setup-python@v5`
- `aws-actions/configure-aws-credentials@v4` (OIDC)
- `hashicorp/setup-terraform@v3` (Terraform 1.9.0)

## APIs Externas

### Strava API v3
- Base: `https://www.strava.com/api/v3`
- Endpoint: `GET /athlete/activities?after={unix_timestamp}&per_page=100`
- Auth: Bearer token (OAuth2, com refresh automático)
- Token refresh: `POST https://www.strava.com/oauth/token`

### OpenAI API
- SDK: `openai>=1.30` com `chat.completions.create`
- Modelo default: `gpt-4o-mini` (custo-benefício para geração de texto)
- Parâmetros: `temperature=0.7`, `max_tokens=1500`

### Telegram Bot API
- Endpoint: `POST https://api.telegram.org/bot{TOKEN}/sendMessage`
- Payload: `{"chat_id": ..., "text": ..., "parse_mode": "Markdown"}`

## Decisões de Design

### Por que `requests` e não `httpx`?
- Sem necessidade de async na Lambda semanal de baixo volume
- `requests` é mais simples e mais utilizado em Lambda Python

### Por que Secrets Manager e não SSM Parameter Store?
- Secrets Manager tem rotação automática nativa
- Integração IAM mais granular
- Melhor para tokens OAuth (Strava) que precisam ser atualizados

### Por que EventBridge Scheduler e não CloudWatch Events Rule?
- EventBridge Scheduler é mais novo, tem retry policy nativa
- Suporte a timezone (América/São Paulo)
- Melhor integração com o padrão moderno AWS

### Por que empacotamento via `archive_file` Terraform e não Lambda Layer?
- A Lambda tem poucas dependências e é executada raramente
- Sem necessidade de compartilhar dependências entre funções
- Mais simples de manter
