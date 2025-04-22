# CaaS (Cashback as a Service)

## Introdução

O CaaS (Cashback as a Service) é uma plataforma SaaS que visa facilitar a implementação de programas de cashback para estabelecimentos comerciais, criando um ecossistema onde os comerciantes podem fidelizar seus clientes através de campanhas de cashback personalizadas, enquanto os usuários podem acumular e gerenciar benefícios em suas carteiras digitais (wallets).

## Domínio Core

Baseado nas informações fornecidas, o domínio principal do CaaS é composto pelos seguintes elementos:

1. **Comerciante/Estabelecimento Comercial**
   - Responsável por criar campanhas de cashback
   - Gera transações para os usuários durante compras
   - Define as regras e parâmetros das campanhas

2. **Usuário/Membro do Cashback**
   - Consumidor final que participa do programa
   - Proprietário das wallets
   - Acumula cashback através de transações

3. **Wallet**
   - Cada usuário pode ter múltiplas wallets
   - Relação 1:1 entre usuário e estabelecimento (uma wallet por estabelecimento para cada usuário)
   - Armazena pontos ou valores acumulados

4. **Transações**
   - Geradas conforme regras das campanhas ativas
   - Representam operações de cashback vinculadas a compras

5. **Campanhas**
   - Podem ser baseadas em pontos ou valor monetário
   - Criadas e gerenciadas pelos estabelecimentos

## Proposta de Arquitetura

# Documento de Discovery: CaaS - Cashback as a Service

## 1. Visão Geral do Produto

O CaaS (Cashback as a Service) é uma solução SaaS que permite estabelecimentos comerciais criar e gerenciar seus próprios programas de cashback para clientes. A plataforma possibilita a criação de campanhas personalizadas de recompensas, gerenciamento de transações e administração de contas de usuários.

## 2. Core Domains do Sistema

### 2.1 Comerciante/Estabelecimento Comercial
- **Responsabilidades**: Criação e gerenciamento de campanhas de cashback, geração de transações para usuários
- **Atributos Principais**:
  - Dados cadastrais (CNPJ, razão social, nome fantasia)
  - Informações de contato
  - Configurações de negócio (segmento, endereços)
  - Métodos de pagamento aceitos
  - Dashboard de desempenho e análises

### 2.2 Usuário/Membro do Cashback
- **Responsabilidades**: Dono das wallets, realiza transações nos estabelecimentos
- **Atributos Principais**:
  - Dados cadastrais (CPF, nome completo)
  - Informações de contato
  - Lista de wallets vinculadas
  - Histórico de transações
  - Preferências de notificação

### 2.3 Wallet
- **Características**: Cada usuário pode ter múltiplas wallets, porém apenas uma wallet por estabelecimento
- **Atributos Principais**:
  - Saldo (em pontos ou valores monetários)
  - Identificador do usuário proprietário
  - Identificador do estabelecimento vinculado
  - Histórico de movimentações
  - Status (ativa, bloqueada, inativa)
  - Data de criação e última atualização

### 2.4 Transações
- **Características**: Geradas conforme as regras das campanhas dos estabelecimentos
- **Atributos Principais**:
  - Identificador único
  - Valor da compra
  - Valor/quantidade de cashback/pontos
  - Referência da campanha vinculada
  - Timestamp da transação
  - Status (pendente, processada, cancelada)
  - Métodos de pagamento utilizados

### 2.5 Campanhas
- **Características**: Criadas pelos estabelecimentos, podem ser baseadas em pontos ou valores monetários
- **Atributos Principais**:
  - Tipo (pontos ou valor)
  - Regras de acumulação
  - Período de vigência
  - Público-alvo
  - Condições específicas (valor mínimo, produtos específicos)
  - Regras de resgate/utilização
  - Métricas de desempenho

## 3. Arquitetura Proposta

### 3.1 Visão de Alto Nível

```
+-----------------------------------------------------------+
|                     Frontend Layer                        |
+---------------+--------------------+----------------------+
| Portal Admin  | Portal Comerciante | App Mobile Usuários  |
+---------------+--------------------+----------------------+
                           |
+-----------------------------------------------------------+
|                        API Layer                          |
+-----------------------------------------------------------+
                           |
+-----------------------------------------------------------+
|                   Application Layer                       |
+-----------------------------------------------------------+
| Comerciante | Usuário | Wallet | Transações | Campanhas  |
+-----------------------------------------------------------+
                           |
+-----------------------------------------------------------+
|                    Domain Layer                           |
+-----------------------------------------------------------+
|  Core Business Logic | Domain Services | Domain Events   |
+-----------------------------------------------------------+
                           |
+-----------------------------------------------------------+
|                 Infrastructure Layer                      |
+-----------------------------------------------------------+
| Database | Message Broker | Cache | External Services    |
+-----------------------------------------------------------+
```

### 3.2 Tecnologias Sugeridas (seguindo convenções .NET)

- **Backend**:
  - ASP.NET Core para APIs RESTful
  - Entity Framework Core para ORM
  - Identity Server para autenticação/autorização
  - Azure Service Bus para mensageria
  - SQL Server ou CosmosDB para banco de dados

- **Frontend**:
  - Portal Admin/Comerciante: Blazor ou React
  - Aplicativo Móvel: Xamarin.Forms ou MAUI

- **DevOps/Infraestrutura**:
  - Azure DevOps para CI/CD
  - Azure App Services para hospedagem
  - Azure Monitor para telemetria

## 4. Modelagem de Dados (Diagrama Conceitual)

```
Comerciante 1 --- * Campanha
Comerciante 1 --- * Transação
Usuário     1 --- * Wallet
Wallet      1 --- * Transação
Campanha    1 --- * Transação
Transação   1 --- 1 Wallet
```
[Detalhamento do Banco de Dados Relacional](./db-simple.md)

## 5. Casos de Uso Principais

### 5.1 Para Comerciantes:
1. Cadastro e configuração da conta
2. Criação e gestão de campanhas de cashback
3. Geração de transações para clientes
4. Visualização de relatórios e métricas
5. Configuração de regras de resgate e utilização

### 5.2 Para Usuários:
1. Cadastro e configuração da conta
2. Visualização de wallets e saldos
3. Histórico de transações e resgates
4. Descoberta de campanhas disponíveis 
5. Resgate de benefícios

## 6. Fluxos de Negócio Principais

### 6.1 Fluxo de Acumulação de Cashback
1. Usuário realiza uma compra no estabelecimento
2. Estabelecimento registra a transação no sistema
3. Sistema verifica as regras da campanha aplicável
4. Sistema calcula o valor/pontos de cashback
5. Sistema credita o valor/pontos na wallet do usuário
6. Usuário e estabelecimento recebem confirmação

### 6.2 Fluxo de Resgate de Cashback
1. Usuário solicita resgate de cashback
2. Sistema verifica saldo disponível e regras aplicáveis
3. Sistema processa o resgate conforme regras da campanha
4. Sistema debita o valor/pontos da wallet do usuário
5. Estabelecimento recebe notificação do resgate
6. Usuário recebe confirmação e detalhes do resgate

## 7. Requisitos Não-Funcionais

1. **Segurança**: Implementação de criptografia de dados, autenticação multifator, proteção contra fraudes
2. **Escalabilidade**: Arquitetura que suporte crescimento horizontal e vertical
3. **Disponibilidade**: SLA de 99.9% de disponibilidade
4. **Performance**: Tempo de resposta médio inferior a 500ms
5. **Integrabilidade**: APIs abertas para integração com sistemas de pagamento e ERPs
6. **Conformidade**: Atendimento à LGPD e regulamentações do setor financeiro

## 8. Roadmap Proposto

### Fase 1 - MVP (3 meses)
- Implementação do core domain
- Portal de administração para comerciantes
- Funções básicas de cadastro e gerenciamento de campanhas
- API para integração com sistemas de PDV

### Fase 2 - Expansão (3 meses)
- Aplicativo móvel para usuários
- Dashboard com métricas e análises
- Funcionalidades de gamificação
- Sistema de notificações

### Fase 3 - Consolidação (6 meses)
- Integração com sistemas de pagamento
- Funcionalidades avançadas de marketing
- Módulo de relatórios personalizados
- API completa para desenvolvedores

## 9. Considerações de Implementação

### 9.1 Domain-Driven Design
Recomendamos seguir os princípios de DDD para manter o sistema coeso e alinhado com o negócio:

- **Agregados**: Comerciante, Usuário, Wallet, Campanha
- **Value Objects**: Dados de transação, regras de campanha, configurações
- **Services**: TransactionService, CampaignRuleService, WalletService

### 9.2 Arquitetura em Camadas

Propõe-se uma arquitetura Clean Architecture com as seguintes camadas:

```csharp
// Exemplo de estrutura de projeto
CaaS.Domain                // Entidades e regras de negócio
CaaS.Application           // Casos de uso e serviços de aplicação
CaaS.Infrastructure        // Implementações concretas (DB, mensageria)
CaaS.API                   // APIs REST para comunicação externa
CaaS.Web                   // Interfaces Web
CaaS.Mobile                // Aplicativo móvel
CaaS.Common                // Componentes compartilhados
```

## 10. Próximos Passos

1. **Workshop de Refinamento**: Sessão com stakeholders para validar o modelo proposto
2. **Prototipagem**: Criação de protótipos de interface para validação com usuários
3. **Arquitetura Técnica Detalhada**: Especificação completa da arquitetura proposta
4. **Estimativa de Recursos**: Definição de equipe, custos e prazos detalhados
5. **Priorização de Backlog**: Definição das funcionalidades prioritárias para o MVP

Este documento serve como ponto de partida para o desenvolvimento do CaaS e deverá ser refinado com a participação de todas as partes interessadas antes do início da implementação.
