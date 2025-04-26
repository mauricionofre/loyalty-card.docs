# Diagrama de Sequência - Processo de Onboarding

O diagrama abaixo ilustra as interações entre os diferentes componentes do sistema durante o processo de onboarding de um novo cliente, desde o registro inicial até a ativação completa.

```mermaid
sequenceDiagram
    participant Cliente as Cliente
    participant API as API Gateway
    participant TS as Tenant Service
    participant US as User & Identity Service
    participant NS as Notification Service
    participant DB as Database

    Cliente->>API: POST /api/v1/register (dados da empresa e admin)
    API->>TS: Processar registro
    
    Note over TS: Inicia transação
    TS->>DB: Criar Tenant
    TS->>DB: Criar Company
    TS->>US: Solicitar criação do admin
    
    US->>DB: Criar usuário admin
    US->>NS: Solicitar envio de e-mail
    
    NS->>NS: Gerar token de validação
    NS->>DB: Salvar token com expiração (7 dias)
    NS->>Cliente: Enviar e-mail com link de validação
    
    Note over Cliente: Aguardando validação (até 7 dias)
    
    Cliente->>API: Acessar link de validação
    API->>US: Validar token
    US->>DB: Marcar e-mail como verificado
    US->>TS: Notificar validação completa
    
    TS->>DB: Atualizar status do tenant/company
    TS->>Cliente: Redirecionar para configuração de senha
    
    Cliente->>US: Configurar senha
    US->>DB: Salvar senha e finalizar cadastro
    US->>Cliente: Redirecionar para dashboard
```

## Interações Principais

1. **Registro Inicial**
   - Cliente submete formulário com dados da empresa e administrador
   - Tenant Service coordena a criação das entidades básicas

2. **Envio de Validação**
   - Notification Service gera token e envia email
   - Token tem validade de 7 dias conforme ADR-001

3. **Validação de Email**
   - Cliente acessa link enviado por email
   - User Service valida o token e atualiza o status

4. **Finalização do Cadastro**
   - Cliente configura senha definitiva
   - Sistema ativa o tenant/company e libera acesso

## Considerações

- Este fluxo implementa a decisão de validação por Callback URL conforme ADR-001
- A comunicação entre serviços é síncrona para o MVP
- As falhas em qualquer etapa são registradas para análise manual
