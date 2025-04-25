# Modelo de Domínio - Processo de Onboarding

## Agregados e Entidades

### Agregado Tenant (Empresa)
- **Entidade Raiz:** Tenant
  - TenantId (identificador único)
  - RazaoSocial
  - NomeFantasia
  - CNPJ
  - Endereço
  - Status (Pendente, Ativo, Suspenso, Inativo)
  - DataCriação
  - DataAtualização
  
- **Entidade:** PlanoAssinatura
  - PlanoId
  - TipoPlano
  - DataInício
  - DataVencimento
  - LimitesDeUso

- **Entidade:** ConfiguraçãoTenant
  - TimeZone
  - Locale
  - Preferências
  - ConfiguraçõesTécnicas

### Agregado Usuário
- **Entidade Raiz:** Usuário
  - UserId (identificador único)
  - Email
  - Nome
  - Função
  - Status (PendenteValidação, Ativo, Bloqueado)
  - TenantId (referência)
  - DataCriação
  - ÚltimoAcesso

- **Entidade:** Credenciais
  - HashSenha
  - ConfiguraçãoMFA
  - HistóricoSenhas
  - TentativasLogin

### Agregado Processo de Onboarding
- **Entidade Raiz:** ProcessoOnboarding
  - ProcessoId
  - TenantId (referência)
  - StatusProcesso (Iniciado, ValidaçãoEmail, ValidaçãoDados, Completo, Cancelado)
  - EtapasCompletas
  - EtapasRestantes
  - DataInício
  - DataÚltimaAtividade

- **Entidade:** ValidaçãoEmail
  - TokenValidação
  - DataEnvio
  - DataExpiração
  - TentativasEnvio
  - StatusValidação

## Value Objects
- **Endereço**
  - Logradouro
  - Número
  - Complemento
  - Bairro
  - Cidade
  - Estado
  - CEP
  - País

- **CNPJ**
  - Número
  - StatusValidação
  - DataValidação

## Eventos de Domínio
1. **TenantRegistrado**
   - TenantId
   - DataOcorrência
   - DadosBásicos

2. **EmailValidacaoEnviado**
   - UsuarioId
   - EmailDestino
   - TokenGerado
   - DataExpiração

3. **EmailValidado**
   - UsuarioId
   - DataValidação
   - Dispositivo

4. **OnboardingCompleto**
   - TenantId
   - UsuarioAdminId
   - DataConclusão
   - TempoTotal

## Políticas de Domínio
- Email de validação expira em 24 horas
- Após 3 tentativas falhas de validação, um novo token deve ser solicitado
- Uma empresa não pode ter status Ativo sem pelo menos um usuário com email validado
- Dados de CNPJ devem ser validados antes da conclusão do onboarding