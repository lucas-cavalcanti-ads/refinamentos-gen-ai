# Spec — Resumo Semanal Strava via Telegram

## Visão Geral

Lambda AWS executada semanalmente que busca atividades do Strava, consolida métricas dos últimos 7 dias, gera um resumo textual via OpenAI API e envia ao usuário via Telegram.

**Quem é impactado:** Lucas — atleta que usa Strava e quer receber um resumo semanal personalizado das suas atividades diretamente no Telegram.

---

## User Stories

### US-001 — Receber resumo semanal automático

**Como** Lucas,  
**quero** receber automaticamente toda semana um resumo das minhas atividades do Strava no Telegram,  
**para** acompanhar minha evolução sem precisar abrir o aplicativo.

**Critérios de aceite:**
- [ ] A Lambda é acionada pelo EventBridge Scheduler toda segunda-feira às 9h (horário de Brasília)
- [ ] O resumo é enviado no chat configurado via `TELEGRAM_CHAT_ID`
- [ ] O resumo cobre os últimos 7 dias a partir da data de execução
- [ ] O resumo é escrito em português do Brasil com tom motivador

### US-002 — Resumo com métricas consolidadas

**Como** Lucas,  
**quero** que o resumo inclua os principais números da semana (distância, tempo, tipos de atividade),  
**para** ter uma visão quantitativa do meu desempenho.

**Critérios de aceite:**
- [ ] O resumo inclui total de atividades
- [ ] O resumo inclui distância total em km
- [ ] O resumo inclui tempo total formatado (hh:mm)
- [ ] O resumo inclui tipos de atividade (corrida, ciclismo etc.)
- [ ] O resumo inclui a atividade mais longa
- [ ] O resumo inclui pace médio quando houver corridas
- [ ] O resumo inclui destaques da semana

### US-003 — Tratamento de semana sem atividades

**Como** Lucas,  
**quero** ser notificada mesmo quando não treinar nenhum dia na semana,  
**para** saber que o sistema está funcionando e ter um lembrete leve.

**Critérios de aceite:**
- [ ] Quando não há atividades nos últimos 7 dias, uma mensagem informativa é enviada via Telegram
- [ ] A mensagem de "sem atividades" não chama a OpenAI API

### US-004 — Segurança e confiabilidade

**Como** Lucas,  
**quero** que meus secrets e dados estejam protegidos e que erros sejam registrados,  
**para** ter segurança e conseguir diagnosticar problemas facilmente.

**Critérios de aceite:**
- [ ] Nenhum secret aparece em logs, código ou GitHub Actions
- [ ] Erros de Strava, OpenAI e Telegram são capturados e logados com contexto
- [ ] A Lambda não silencia falhas — exceções são propagadas para CloudWatch Alarms
- [ ] Todos os secrets estão no AWS Secrets Manager

### US-005 — Infraestrutura gerenciada como código

**Como** Lucas,  
**quero** que toda a infraestrutura AWS seja criada e atualizada via Terraform,  
**para** garantir reprodutibilidade e facilitar manutenção futura.

**Critérios de aceite:**
- [ ] Lambda, IAM Role, EventBridge Scheduler, CloudWatch Log Group e Secrets Manager gerenciados via Terraform
- [ ] Terraform apply executado automaticamente no pipeline de deploy após merge na `main`
- [ ] Nenhum recurso AWS criado manualmente

### US-006 — Pipeline CI/CD automatizado

**Como** Lucas,  
**quero** que testes rodem automaticamente em toda branch e o deploy ocorra ao mergear na `main`,  
**para** ter confiança no código e deploy sem intervenção manual.

**Critérios de aceite:**
- [ ] CI roda em qualquer push em branches diferentes de `main`
- [ ] CI valida cobertura mínima de 90%
- [ ] Deploy ocorre automaticamente após merge na `main`
- [ ] Autenticação AWS via OIDC (sem chaves estáticas)

---

## Requisitos Funcionais

| ID     | Descrição                                                                 |
|--------|---------------------------------------------------------------------------|
| RF-001 | Lambda executada semanalmente via EventBridge Scheduler                   |
| RF-002 | Buscar atividades do Strava dos últimos 7 dias                            |
| RF-003 | Calcular métricas: total, distância, tempo, tipos, mais longa, pace, destaques |
| RF-004 | Gerar resumo via OpenAI API com prompt estruturado em PT-BR               |
| RF-005 | Enviar resumo via Telegram Bot API                                        |
| RF-006 | Enviar mensagem informativa quando não há atividades                      |
| RF-007 | Tratar e logar erros sem expor secrets                                    |

## Requisitos Não Funcionais

| ID      | Descrição                                                              |
|---------|------------------------------------------------------------------------|
| RNF-001 | Cobertura mínima de testes unitários: 90%                              |
| RNF-002 | Nenhum secret em código, logs, README ou CI/CD                         |
| RNF-003 | Toda infraestrutura AWS via Terraform                                  |
| RNF-004 | Deploy automático após merge na `main`                                 |
| RNF-005 | Otimizado para execução semanal e baixo custo                          |
| RNF-006 | CloudWatch Logs com início, fim, qtd. atividades e status de envio     |
| RNF-007 | Lambda idempotente                                                     |

---

## Fora de Escopo

- API Gateway
- Interface web
- Multiusuário
- Banco de dados
- Dashboard
- E-mail, WhatsApp
- Recomendações médicas

---

## Secrets Necessários

Gerenciados via AWS Secrets Manager. Nunca versionados.

```
OPENAI_API_KEY
OPENAI_MODEL
STRAVA_CLIENT_ID
STRAVA_CLIENT_SECRET
STRAVA_ACCESS_TOKEN
STRAVA_REFRESH_TOKEN
STRAVA_TOKEN_EXPIRES_AT
TELEGRAM_BOT_TOKEN
TELEGRAM_CHAT_ID
```
