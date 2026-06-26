# Constituição do Projeto — Resumo Semanal Strava

## 1. Qualidade e Confiabilidade

- Cobertura mínima de testes unitários: **90%**
- Todo comportamento crítico deve ter testes automatizados
- Falhas de serviços externos (Strava, OpenAI, Telegram) devem ser tratadas, logadas e não devem expor informações sensíveis
- Erros devem ser observáveis via CloudWatch Logs sem revelar secrets
- A Lambda deve ser idempotente: reexecuções manuais não causam efeitos colaterais indesejados

## 2. Segurança

- Nenhum secret deve estar versionado no repositório (código, README, CI/CD, comments)
- Todos os segredos são gerenciados via AWS Secrets Manager
- A IAM Role da Lambda deve seguir o princípio do mínimo privilégio
- Credenciais AWS no CI/CD via OIDC (sem chaves estáticas)
- Logs não devem conter valores de API keys, tokens ou senhas

## 3. Manutenibilidade e Arquitetura

- Linguagem: **Python 3.11+**
- Arquitetura em camadas: `domain` (regras de negócio puras), `application` (orquestração), `infrastructure` (clientes externos)
- Cada camada deve ter responsabilidade única e clara
- Dependências externas injetadas via construtores para facilitar testes
- Infraestrutura como código: **Terraform** para todos os recursos AWS
- Sem banco de dados, sem API Gateway, sem estado persistente

## 4. Performance e Custo

- A solução é otimizada para execução **semanal** e baixo volume
- Memória Lambda: 256 MB (suficiente para a carga)
- Timeout Lambda: 300 segundos (margem para chamadas externas)
- CloudWatch Log Group com retenção de 30 dias
- Sem recursos superdimensionados

## 5. Observabilidade

- Logs estruturados no início e fim da execução
- Log da quantidade de atividades processadas
- Log do status de envio ao Telegram
- Log de erros com contexto (sem expor secrets)
- Uso de CloudWatch Logs nativo do Lambda

## 6. Automação e CI/CD

- CI executa em qualquer branch diferente de `main`
- Deploy automático ocorre apenas após merge na `main`
- Terraform apply somente após testes passarem
- Autenticação AWS via OIDC no GitHub Actions

## 7. Idioma e Documentação

- Todos os artefatos, código, comentários e documentação em **português do Brasil**
- Mensagens de log em português
- README completo com instruções de setup e deploy
