# Modelo de Domínio - Processo de Onboarding

## Agregados e Entidades

### Agregado Tenant
- **Entidade Raiz:** Tenant
  - TenantId (identificador único)
  - Nome
  - Status (Ativo, Inativo)
  - ConfiguraçãoBancoDados (estratégia de armazenamento)
  - DataCriação
  - DataAtualização

### Agregado Company
- **Entidade Raiz:** Company
  - CompanyId (identificador único)
  - TenantId (referência para o Tenant)
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

- **Entidade:** ConfiguraçãoCompany
  - TimeZone
  - Locale
  - Preferências
  - ConfiguraçõesTécnicas
  - ConnectionString (quando aplicável)

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

## Estratégia de ConnectionString com Tenant Separado

### Abordagem para MVP
Mesmo com a separação entre Tenant e Company, podemos manter uma estratégia simplificada para o MVP:

1. **Schema por Tenant**: Um único banco de dados PostgreSQL com schemas separados por tenant
   ```
   // Exemplo de como gerar schemas dinâmicos
   CREATE SCHEMA IF NOT EXISTS tenant_{tenantId};
   ```

2. **ConnectionString única com contexto de tenant**:
   ```
   // ConnectionString base
   "Host=database;Database=loyalty_card;Username=app;Password=****;"
   
   // Código para trocar de schema dinamicamente baseado no tenant
   using (var command = connection.CreateCommand())
   {
       command.CommandText = $"SET search_path TO tenant_{tenantId}";
       await command.ExecuteNonQueryAsync();
   }
   ```

3. **Mapeamento Tenant-Company**:
   - Cada Tenant pode ter múltiplas Companies
   - A tabela de Companies inclui TenantId como chave estrangeira
   - Todos os outros dados relacionados incluem CompanyId para filtro adicional

### Benefícios desta Abordagem
- **Flexibilidade desde o início**: Prepara o sistema para cenários onde um tenant gerencia múltiplas empresas
- **Simplicidade de implementação**: Mantém uma única base de dados no MVP
- **Isolamento por tenant**: Garante separação de dados mesmo entre empresas do mesmo grupo
- **Evolução facilitada**: Prepara a base para adotar estratégias mais avançadas no futuro

### Considerações de Implementação

1. **Para o MVP**:
   - Implementar TenantContext para resolver o tenant atual
   - Filtrar dados sempre primeiro por Tenant e depois por Company
   - Usar DbContext configurado para definir automaticamente o schema correto

2. **Preparação para Evolução**:
   - Abstrair o acesso a dados através de repositórios específicos por tenant
   - Implementar factory de DbContext que configura o schema correto
   - Adicionar suporte a operações cross-tenant para casos específicos (ex: relatórios consolidados)