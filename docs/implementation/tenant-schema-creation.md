# Implementação de Schemas Dinâmicos para Multi-tenancy

## Visão Geral

O PostgreSQL permite a criação de schemas em tempo de execução através de comandos SQL padrão. Em uma aplicação multi-tenant, podemos criar automaticamente um novo schema quando um tenant é registrado no sistema.

## Implementação de Exemplo

### 1. Criação de Schema para Novos Tenants

```csharp
public async Task CreateTenantSchemaAsync(Guid tenantId)
{
    // Formatação segura do nome do schema para evitar SQL Injection
    string schemaName = $"tenant_{tenantId.ToString().Replace("-", "_")}";
    
    using (var connection = new NpgsqlConnection(_connectionString))
    {
        await connection.OpenAsync();
        
        // Criar o schema se não existir
        using (var command = connection.CreateCommand())
        {
            command.CommandText = $"CREATE SCHEMA IF NOT EXISTS {schemaName}";
            await command.ExecuteNonQueryAsync();
        }
        
        // Criar as tabelas necessárias no novo schema
        await CreateTablesForTenantAsync(connection, schemaName);
    }
}
```

### 2. Criação das Tabelas no Novo Schema

```csharp
private async Task CreateTablesForTenantAsync(NpgsqlConnection connection, string schemaName)
{
    // Lista das tabelas necessárias para cada tenant
    string[] tableCreationScripts = new[]
    {
        $@"CREATE TABLE IF NOT EXISTS {schemaName}.companies (
            id UUID PRIMARY KEY,
            name VARCHAR(200) NOT NULL,
            trade_name VARCHAR(200),
            cnpj VARCHAR(14) NOT NULL,
            status VARCHAR(20) NOT NULL DEFAULT 'Pending',
            created_at TIMESTAMP NOT NULL DEFAULT NOW(),
            updated_at TIMESTAMP
        )",
        
        $@"CREATE TABLE IF NOT EXISTS {schemaName}.users (
            id UUID PRIMARY KEY,
            company_id UUID NOT NULL REFERENCES {schemaName}.companies(id),
            email VARCHAR(200) NOT NULL,
            name VARCHAR(200) NOT NULL,
            role VARCHAR(50) NOT NULL,
            status VARCHAR(20) NOT NULL DEFAULT 'PendingValidation',
            created_at TIMESTAMP NOT NULL DEFAULT NOW(),
            last_access TIMESTAMP,
            CONSTRAINT uk_email UNIQUE(email)
        )",
        
        // Adicione mais scripts para outras tabelas aqui
    };
    
    foreach (var script in tableCreationScripts)
    {
        using (var command = connection.CreateCommand())
        {
            command.CommandText = script;
            await command.ExecuteNonQueryAsync();
        }
    }
}
```

### 3. Configuração do Contexto de Tenant no DbContext (Entity Framework)

```csharp
public class ApplicationDbContext : DbContext
{
    private readonly ITenantContext _tenantContext;
    
    public ApplicationDbContext(DbContextOptions<ApplicationDbContext> options, ITenantContext tenantContext)
        : base(options)
    {
        _tenantContext = tenantContext;
    }
    
    public DbSet<Company> Companies { get; set; }
    public DbSet<User> Users { get; set; }
    // Outros DbSets...
    
    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        base.OnModelCreating(modelBuilder);
        
        // Configurar as entidades para usar o schema do tenant
        modelBuilder.HasDefaultSchema($"tenant_{_tenantContext.GetCurrentTenantId()}");
        
        // Configurações adicionais...
    }
}
```

### 4. Alterando Conexões em Runtime

Às vezes você pode precisar alternar entre schemas em uma conexão existente:

```csharp
public async Task SwitchTenantSchemaAsync(NpgsqlConnection connection, Guid tenantId)
{
    string schemaName = $"tenant_{tenantId.ToString().Replace("-", "_")}";
    
    using (var command = connection.CreateCommand())
    {
        command.CommandText = $"SET search_path TO {schemaName}";
        await command.ExecuteNonQueryAsync();
    }
}
```

## Considerações Importantes

### Permissões de Banco
O usuário da aplicação no PostgreSQL precisará das seguintes permissões:
- `CREATE SCHEMA` - para criar novos schemas
- `CREATE TABLE` - para criar tabelas em schemas
- Permissões apropriadas para manipular dados em cada schema

### Performance
A criação de schema é uma operação que adquire um lock no banco, então é recomendável:
- Criar o schema durante o processo de registro de tenant
- Implementar um mecanismo de fallback caso o schema já exista
- Considerar a implementação de migrações assíncronas para não bloquear o processo de registro

### Segurança
- Sempre utilize parâmetros ou construa strings de forma segura para evitar SQL Injection
- Garanta que cada tenant só possa acessar seu próprio schema
- Implemente um mecanismo de auditoria para rastrear mudanças no schema

## Exemplo de Fluxo Completo de Criação de Tenant

1. Usuário se registra e fornece dados da empresa
2. Sistema cria um registro de Tenant no schema público
3. Sistema cria um novo schema para o tenant (ex: tenant_[guid])
4. Sistema cria as tabelas necessárias no novo schema
5. Sistema insere os dados iniciais da empresa no schema do tenant
6. Sistema configura permissões apropriadas para o schema
7. Usuário é notificado da criação bem-sucedida do tenant

Este fluxo pode ser incorporado ao seu processo de onboarding, executando a criação do schema assim que o tenant é registrado e validado.
