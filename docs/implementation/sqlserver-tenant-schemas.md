# Implementação de Multi-tenancy com SQL Server

## Visão Geral

O SQL Server também suporta a criação e utilização de schemas como forma de organização e isolamento lógico de dados, similar ao PostgreSQL. No entanto, há algumas diferenças específicas na implementação da estratégia de multi-tenancy.

## Diferenças entre SQL Server e PostgreSQL

| Característica | SQL Server | PostgreSQL |
|----------------|------------|------------|
| Sintaxe de criação | `CREATE SCHEMA [nome]` | `CREATE SCHEMA [nome]` |
| Qualificação de objeto | `[schema].[tabela]` | `[schema].[tabela]` |
| Separação de segurança | Sim, via permissões por schema | Sim, via permissões por schema |
| Troca dinâmica de schema | Via `USE` e qualificação de objetos | Via `SET search_path TO` |
| Schema padrão | `dbo` | `public` |

## Implementação com SQL Server

### 1. Criação de Schema para Novos Tenants

```csharp
public async Task CreateTenantSchemaAsync(Guid tenantId)
{
    // Formatação segura do nome do schema para evitar SQL Injection
    string schemaName = $"tenant_{tenantId.ToString().Replace("-", "_")}";
    
    using (var connection = new SqlConnection(_connectionString))
    {
        await connection.OpenAsync();
        
        // Criar o schema se não existir
        using (var command = connection.CreateCommand())
        {
            // SQL Server não tem a cláusula 'IF NOT EXISTS' diretamente no CREATE SCHEMA
            // Precisamos verificar primeiro
            command.CommandText = @"
                IF NOT EXISTS (SELECT * FROM sys.schemas WHERE name = @schemaName)
                BEGIN
                    EXEC('CREATE SCHEMA [' + @schemaName + ']')
                END";
            
            command.Parameters.AddWithValue("@schemaName", schemaName);
            await command.ExecuteNonQueryAsync();
        }
        
        // Criar as tabelas necessárias no novo schema
        await CreateTablesForTenantAsync(connection, schemaName);
    }
}
```

### 2. Criação das Tabelas no Novo Schema

```csharp
private async Task CreateTablesForTenantAsync(SqlConnection connection, string schemaName)
{
    // Lista das tabelas necessárias para cada tenant
    string[] tableCreationScripts = new[]
    {
        $@"IF NOT EXISTS (SELECT * FROM sys.tables t
            JOIN sys.schemas s ON t.schema_id = s.schema_id
            WHERE s.name = '{schemaName}' AND t.name = 'companies')
        BEGIN
            CREATE TABLE [{schemaName}].[companies] (
                id UNIQUEIDENTIFIER PRIMARY KEY,
                name NVARCHAR(200) NOT NULL,
                trade_name NVARCHAR(200),
                cnpj VARCHAR(14) NOT NULL,
                status VARCHAR(20) NOT NULL DEFAULT 'Pending',
                created_at DATETIME NOT NULL DEFAULT GETDATE(),
                updated_at DATETIME
            )
        END",
        
        $@"IF NOT EXISTS (SELECT * FROM sys.tables t
            JOIN sys.schemas s ON t.schema_id = s.schema_id
            WHERE s.name = '{schemaName}' AND t.name = 'users')
        BEGIN
            CREATE TABLE [{schemaName}].[users] (
                id UNIQUEIDENTIFIER PRIMARY KEY,
                company_id UNIQUEIDENTIFIER NOT NULL,
                email NVARCHAR(200) NOT NULL,
                name NVARCHAR(200) NOT NULL,
                role VARCHAR(50) NOT NULL,
                status VARCHAR(20) NOT NULL DEFAULT 'PendingValidation',
                created_at DATETIME NOT NULL DEFAULT GETDATE(),
                last_access DATETIME,
                CONSTRAINT uk_{schemaName}_email UNIQUE(email),
                CONSTRAINT fk_{schemaName}_company FOREIGN KEY (company_id) 
                    REFERENCES [{schemaName}].[companies](id)
            )
        END",
        
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

### 3. Utilizando Entity Framework Core com SQL Server

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
        
        var schema = $"tenant_{_tenantContext.GetCurrentTenantId().ToString().Replace("-", "_")}";
        
        // Configurar as entidades para usar o schema do tenant
        modelBuilder.HasDefaultSchema(schema);
        
        // No SQL Server, você precisa configurar explicitamente cada entidade
        modelBuilder.Entity<Company>().ToTable("companies", schema);
        modelBuilder.Entity<User>().ToTable("users", schema);
        
        // Configurações adicionais...
    }
}
```

### 4. Trabalhando com Consultas Entre Schemas

No SQL Server, não há um equivalente direto ao `SET search_path` do PostgreSQL. Você precisa qualificar os nomes das tabelas com o schema:

```csharp
public async Task<List<Company>> GetCompaniesByTenantAsync(Guid tenantId)
{
    string schema = $"tenant_{tenantId.ToString().Replace("-", "_")}";
    
    using (var connection = new SqlConnection(_connectionString))
    {
        await connection.OpenAsync();
        
        using (var command = connection.CreateCommand())
        {
            command.CommandText = $"SELECT * FROM [{schema}].[companies]";
            
            var companies = new List<Company>();
            using (var reader = await command.ExecuteReaderAsync())
            {
                while (await reader.ReadAsync())
                {
                    companies.Add(new Company
                    {
                        Id = reader.GetGuid(0),
                        Name = reader.GetString(1),
                        // mapeie outras propriedades...
                    });
                }
            }
            
            return companies;
        }
    }
}
```

## Considerações para SQL Server

### Permissões e Segurança

O SQL Server permite um controle granular de permissões por schema:

```sql
-- Conceder permissões de leitura para um usuário específico
GRANT SELECT ON SCHEMA::[tenant_123] TO [app_user];

-- Conceder permissões completas
GRANT CONTROL ON SCHEMA::[tenant_123] TO [admin_user];
```

### Performance

- **Estatísticas separadas**: O SQL Server mantém estatísticas separadas por schema, o que pode melhorar o desempenho de consultas
- **Planos de execução**: Planos podem ser reutilizados entre schemas similares
- **Particionamento**: Considere usar particionamento de tabelas para tenants grandes

### Limitações

1. **Escalabilidade**: Assim como no PostgreSQL, um grande número de schemas pode afetar a performance
2. **Nomes de restrições**: No SQL Server, é importante garantir que os nomes de restrições sejam únicos entre schemas
3. **Tamanho do banco de dados**: O SQL Server tem limites para o tamanho total do banco de dados, que se aplica a todos os schemas

## Conclusão

O SQL Server oferece capacidades robustas para implementar uma estratégia de multi-tenancy baseada em schemas, similar ao PostgreSQL. A principal diferença está na sintaxe e na forma como você trabalha com os schemas em tempo de execução. Com as técnicas descritas acima, é possível implementar uma solução eficiente de isolamento de dados por tenant, mantendo a simplicidade operacional de um banco de dados único.
