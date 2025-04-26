# Arquitetura de Referência - Onboarding de Clientes (MVP)

## Visão Geral
Este documento estabelece a arquitetura de referência simplificada para o processo de onboarding de clientes no MVP, definindo os componentes essenciais, suas interações e os padrões arquiteturais a serem seguidos.

## Princípios Arquiteturais
- **Simplicidade:** Foco em funcionalidades essenciais para o MVP
- **Desacoplamento:** Serviços com responsabilidades bem definidas
- **Observabilidade:** Monitoramento das etapas principais do fluxo
- **Segurança básica:** Proteção adequada de dados sensíveis

## Componentes Arquiteturais
1. **API Gateway**
   - Roteamento de requisições
   - Rate limiting básico
   - Autenticação inicial

2. **User & Identity Service (Consolidado)**
   - Gestão de identidades e usuários
   - Autenticação
   - Gestão de permissões básicas

3. **Tenant Service**
   - Registro e gestão de empresas
   - Validação básica de dados empresariais (formato CNPJ)
   - Configurações específicas por tenant

4. **Notification Service**
   - Envio de emails transacionais (principalmente validação)
   - Templates personalizados simples

## Padrões de Comunicação
- **Request-response:** Comunicação síncrona via REST para operações críticas
- **Tratamento básico de erros:** Logs detalhados para análise manual

## Armazenamento de Dados
- **Abordagem simplificada:**
  - PostgreSQL único com schemas separados para isolamento lógico
  - Criptografia para dados sensíveis
  - Tokens de validação armazenados no banco de dados com lógica de expiração

## Fluxo de Onboarding para MVP
1. Usuário realiza cadastro com informações básicas
2. Sistema valida formato dos dados (incluindo CNPJ)
3. Email com link de confirmação é enviado (validade: 7 dias)
4. Usuário confirma email através do link
5. Sistema ativa automaticamente o tenant após validação do email
6. Usuário obtém acesso à plataforma

## Considerações de Deployment
- Implantação simplificada com poucos serviços
- CI/CD básico para garantir qualidade
- Infraestrutura mínima viável

## Métricas e Monitoramento
- Tempo médio de onboarding completo
- Taxa de conversão do funil de onboarding
- Taxa de validação de email
- Principais erros do processo

## Evolução Pós-MVP
As seguintes funcionalidades estão planejadas para implementação após o MVP:
- Validação externa de CNPJ com serviços governamentais
- Implementação de cache distribuído (Redis)
- Separação completa dos serviços (User e Identity)
- Padrões avançados como Event-driven, Saga e Circuit breaker
- Event store para auditoria completa

## Referências

- [ADR-001: Decisões Arquiteturais do Processo de Onboarding](adr/adr-001-decisoes-arquiteturais-processo-onboarding.md)