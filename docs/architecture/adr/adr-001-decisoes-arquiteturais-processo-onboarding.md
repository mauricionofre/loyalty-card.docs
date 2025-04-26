# ADR-001: Decisões Arquiteturais do Processo de Onboarding

| Atributo | Valor |
|----------|-------|
| **Status** | Aprovado |
| **Data** | 25 de abril de 2025 |
| **Autores** | Mauricio Onofre (Arquiteto de Solução) |

## Contexto
O processo de onboarding é a porta de entrada dos clientes para nossa plataforma de cartões de fidelidade. Precisamos garantir que este processo seja robusto, seguro e ofereça uma excelente experiência ao usuário, mesmo em cenários de falha. Diversas abordagens arquiteturais foram consideradas para os componentes críticos deste fluxo.

## Decisões

### 1. Padrão de Validação de Email

#### Problema
Como implementar uma validação de email que seja segura e ofereça boa experiência ao usuário?

#### Alternativas Consideradas
1. **Callback URL**: Enviar um link único de validação para o email cadastrado.
2. **One-Time Password (OTP)**: Enviar um código numérico temporário para o email cadastrado.
3. **Abordagem híbrida**: Oferecer ambas as opções simultaneamente.

#### Decisão
**Escolha: Callback URL**

Implementaremos a validação via URL de callback (link de confirmação) para simplificar o processo de desenvolvimento e a experiência do usuário no MVP.

#### Justificativa
Esta abordagem oferece:
- Melhor experiência de usuário através do link direto (um clique)
- Simplicidade de implementação para o MVP
- Processo de validação claro e direto
- Redução de complexidade técnica mantendo a segurança básica necessária

### 2. Armazenamento de Tokens de Validação

#### Problema
Onde armazenar tokens de validação temporários de forma eficiente e segura?

#### Alternativas Consideradas
1. **Redis**: Armazenamento em memória com expiração automática
2. **Database**: Armazenamento persistente em banco de dados relacional
3. **Híbrido**: Armazenamento primário em Redis com backup em banco de dados

#### Decisão
**Escolha: Database (PostgreSQL)**

Utilizaremos o PostgreSQL como armazenamento único para tokens de validação, com implementação de lógica para controle de expiração.

#### Justificativa
Esta abordagem proporciona:
- Simplificação da infraestrutura para o MVP (sem serviços adicionais)
- Redução de custos operacionais
- Persistência nativa para fins de auditoria e compliance
- Utilização da infraestrutura já existente para o projeto

### 3. Validação de CNPJ

#### Problema
Como implementar a validação de CNPJ sem bloquear o fluxo principal de cadastro?

#### Alternativas Consideradas
1. **Síncrona**: Validar o CNPJ no momento do cadastro, bloqueando o processo até a conclusão
2. **Assíncrona**: Permitir o cadastro com validação preliminar e realizar a validação completa em background
3. **Mista**: Validação superficial síncrona e validação completa assíncrona

#### Decisão
**Escolha: Apenas validação básica**

Implementaremos somente a validação do formato e dígitos verificadores do CNPJ durante o cadastro, sem integração com serviços externos de validação.

#### Justificativa
Esta abordagem:
- Acelera o desenvolvimento do MVP
- Elimina dependências externas que poderiam atrasar o projeto
- Mantém o nível básico de validação para evitar erros simples
- Permite a implementação futura de validações mais robustas quando necessário

### 4. Estratégia de Rollback

#### Problema
Como gerenciar falhas durante o processo de onboarding que envolvem múltiplos serviços?

#### Alternativas Consideradas
1. **Transações Compensatórias**: Implementar operações que desfazem as ações já realizadas
2. **Máquina de Estados**: Modelar o processo como uma máquina de estados para controle granular
3. **Saga Pattern**: Implementar sequência de transações locais com compensações

#### Decisão
**Escolha: Abordagem Simplificada com Registro de Erros**

Para o MVP, implementaremos uma abordagem simplificada onde cada etapa do processo é tratada como uma transação isolada com registro detalhado de erros. Não desenvolveremos um mecanismo complexo de rollback automatizado nesse momento.

#### Justificativa
Esta abordagem:
- Acelera significativamente o tempo de desenvolvimento do MVP
- Reduz a complexidade técnica inicial
- Fornece informações suficientes para resolução manual de problemas quando necessário
- Permite que o time de suporte resolva casos excepcionais manualmente enquanto ganhamos entendimento do domínio

### 5. Timeout para Validação

#### Problema
Por quanto tempo o link/token de validação deve permanecer válido?

#### Alternativas Consideradas
1. **24 horas**: Um dia para validação
2. **72 horas**: Três dias para validação
3. **7 dias**: Uma semana para validação

#### Decisão
**Escolha: 7 dias sem renovação automática**

Os tokens de validação expirarão após 7 dias. Para o MVP, não implementaremos um mecanismo automático de renovação, mas orientaremos o usuário a reiniciar o processo caso o prazo expire.

#### Justificativa
Esta abordagem:
- Simplifica o desenvolvimento inicial removendo a necessidade de um sistema de renovação de tokens
- Proporciona uma janela de tempo confortável para o usuário completar o processo
- Reduz a taxa de abandono por expiração prematura do token
- Facilita o suporte ao usuário com um prazo mais amplo e fácil de comunicar
- Permite coletar métricas sobre o comportamento real dos usuários antes de implementar soluções mais sofisticadas

## Consequências

### Positivas
- Processo de onboarding simplificado facilitando a entrega do MVP
- Experiência de usuário direta e intuitiva
- Menor dependência de serviços externos
- Redução de custos de infraestrutura
- Melhor auditabilidade através da máquina de estados

### Negativas
- Validação de CNPJ limitada ao formato estrutural, sem verificação de existência real
- Potencial necessidade de implementar otimizações de performance para tokens no banco de dados
- Eventual necessidade de evoluir as funcionalidades em versões futuras

## Alternativas Não Escolhidas

### Validação de Email com OTP
Rejeitada para simplificar o desenvolvimento do MVP e reduzir a complexidade para o usuário.

### Uso de Redis para Tokens
Rejeitado para reduzir custos e complexidade de infraestrutura no MVP.

### Validação Assíncrona com Serviços Externos de CNPJ
Rejeitada para acelerar o desenvolvimento do MVP e reduzir dependências externas.

## Conformidade e Verificação
- Métricas de tempo de validação de email serão monitoradas (expectativa: 90% < 4 horas)
- Taxa de abandono no processo de onboarding será comparada com baseline atual
- Tempo total de onboarding (do início à validação) será medido e reportado mensalmente
- Auditoria trimestral dos logs da máquina de estados para verificar integridade do processo

## Referências
- [RFC 6749 - The OAuth 2.0 Authorization Framework](https://tools.ietf.org/html/rfc6749)
- [Pattern: Saga](https://microservices.io/patterns/data/saga.html)
- [Finite State Machines](https://en.wikipedia.org/wiki/Finite-state_machine)