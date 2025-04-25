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
**Escolha: Callback URL + OTP como fallback**

Implementaremos primariamente a validação via URL de callback (link de confirmação), mas também ofereceremos um código OTP na mesma comunicação como mecanismo alternativo.

#### Justificativa
Esta abordagem oferece:
- Melhor experiência de usuário através do link direto (um clique)
- Opção alternativa caso o cliente tenha problemas com o link (ex: clientes corporativos com firewalls restritivos)
- Maior taxa de conclusão do processo de onboarding
- Segurança aprimorada pela dupla opção de validação

### 2. Armazenamento de Tokens de Validação

#### Problema
Onde armazenar tokens de validação temporários de forma eficiente e segura?

#### Alternativas Consideradas
1. **Redis**: Armazenamento em memória com expiração automática
2. **Database**: Armazenamento persistente em banco de dados relacional
3. **Híbrido**: Armazenamento primário em Redis com backup em banco de dados

#### Decisão
**Escolha: Redis com backup em Database**

Utilizaremos o Redis como armazenamento primário para tokens de validação, com uma rotina de persistência no banco de dados relacional para fins de auditoria e recuperação.

#### Justificativa
Esta abordagem proporciona:
- Alta performance no acesso aos tokens (operações de leitura em microsegundos)
- TTL (Time-To-Live) nativo para expiração automática de tokens
- Persistência para fins de auditoria e compliance
- Resiliência em caso de falhas no Redis

### 3. Validação de CNPJ

#### Problema
Como implementar a validação de CNPJ sem bloquear o fluxo principal de cadastro?

#### Alternativas Consideradas
1. **Síncrona**: Validar o CNPJ no momento do cadastro, bloqueando o processo até a conclusão
2. **Assíncrona**: Permitir o cadastro com validação preliminar e realizar a validação completa em background
3. **Mista**: Validação superficial síncrona e validação completa assíncrona

#### Decisão
**Escolha: Assíncrona com validação preliminar**

Implementaremos uma validação preliminar do formato e dígitos verificadores do CNPJ durante o cadastro, permitindo que o processo continue. A validação completa junto aos órgãos oficiais será realizada de forma assíncrona.

#### Justificativa
Esta abordagem:
- Não bloqueia o fluxo principal de cadastro
- Minimiza impactos de serviços externos lentos ou instáveis
- Permite feedback imediato sobre erros básicos de formato
- Melhora a experiência do usuário com tempos de resposta mais rápidos

### 4. Estratégia de Rollback

#### Problema
Como gerenciar falhas durante o processo de onboarding que envolvem múltiplos serviços?

#### Alternativas Consideradas
1. **Transações Compensatórias**: Implementar operações que desfazem as ações já realizadas
2. **Máquina de Estados**: Modelar o processo como uma máquina de estados para controle granular
3. **Saga Pattern**: Implementar sequência de transações locais com compensações

#### Decisão
**Escolha: Máquina de Estados**

Modelaremos o processo de onboarding como uma máquina de estados explícita, onde cada etapa é registrada com seu estado atual e possíveis transições.

#### Justificativa
Esta abordagem:
- Proporciona visibilidade clara do estado atual do processo
- Facilita a retomada de processos interrompidos
- Simplifica a implementação de mecanismos de recuperação
- Oferece auditoria natural do processo e suas transições

### 5. Timeout para Validação

#### Problema
Por quanto tempo o link/token de validação deve permanecer válido?

#### Alternativas Consideradas
1. **24 horas**: Um dia para validação
2. **72 horas**: Três dias para validação
3. **7 dias**: Uma semana para validação

#### Decisão
**Escolha: 24 horas com renovação sob demanda**

Os tokens de validação expirarão após 24 horas, mas oferecemos um mecanismo simples para o usuário solicitar um novo token caso necessário.

#### Justificativa
Esta abordagem:
- Equilibra segurança (token de curta duração) e conveniência (possibilidade de renovação)
- Incentiva a conclusão rápida do processo de onboarding
- Reduz o risco de tokens comprometidos permanecerem válidos por muito tempo
- Melhora as métricas de conversão do funil de cadastro

## Consequências

### Positivas
- Processo de onboarding resiliente a falhas de componentes individuais
- Experiência de usuário otimizada com feedback imediato
- Maior taxa de conversão do processo de cadastro
- Melhor auditabilidade através da máquina de estados
- Capacidade de recuperação em casos de falha

### Negativas
- Aumento da complexidade da implementação, especialmente da máquina de estados
- Necessidade de mecanismos robustos de comunicação entre serviços
- Potencial atraso na validação completa de empresas (CNPJ)
- Dependência do Redis para funcionamento ideal dos tokens

## Alternativas Não Escolhidas

### Validação de Email Apenas por OTP
Rejeitada por aumentar o atrito no processo de onboarding, exigindo entrada manual de códigos.

### Validação Síncrona de CNPJ
Rejeitada devido ao alto risco de timeout em serviços externos e consequente abandono do processo.

### Transações Distribuídas (2PC)
Rejeitada pela complexidade excessiva e potencial de bloqueio de recursos.

## Conformidade e Verificação
- Métricas de tempo de validação de email serão monitoradas (expectativa: 90% < 4 horas)
- Taxa de abandono no processo de onboarding será comparada com baseline atual
- Tempo total de onboarding (do início à validação) será medido e reportado mensalmente
- Auditoria trimestral dos logs da máquina de estados para verificar integridade do processo

## Referências
- [RFC 6749 - The OAuth 2.0 Authorization Framework](https://tools.ietf.org/html/rfc6749)
- [Pattern: Saga](https://microservices.io/patterns/data/saga.html)
- [Finite State Machines](https://en.wikipedia.org/wiki/Finite-state_machine)