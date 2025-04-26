# API Specification - Onboarding

## Tenant Management API

### POST /api/v1/tenants
Registra um novo tenant no sistema.

#### Request Body
```json
{
  "nome": "Grupo XYZ",
  "status": "ativo"
}
```

#### Response (201 Created)
```json
{
  "tenantId": "t-abc123",
  "nome": "Grupo XYZ",
  "status": "ativo",
  "createdAt": "2025-04-25T14:30:00Z"
}
```

### POST /api/v1/register
Realiza o registro completo em uma única operação, criando o tenant e a primeira company.

#### Request Body
```json
{
  "empresa": {
    "razaoSocial": "Empresa XYZ Ltda",
    "nomeFantasia": "XYZ Solutions",
    "cnpj": "12.345.678/0001-99",
    "endereco": {
      "logradouro": "Av. Paulista",
      "numero": "1000",
      "complemento": "Sala 501",
      "bairro": "Bela Vista",
      "cidade": "São Paulo",
      "estado": "SP",
      "cep": "01310-100",
      "pais": "Brasil"
    },
    "contato": {
      "email": "contato@xyz.com.br",
      "telefone": "+55 11 98765-4321"
    }
  },
  "administrador": {
    "nome": "João Silva",
    "email": "joao.silva@xyz.com.br",
    "cargo": "Diretor de Tecnologia"
  }
}
```

#### Response (201 Created)
```json
{
  "tenantId": "t-abc123",
  "tenantName": "XYZ", 
  "companyId": "c-def456",
  "status": "pendingValidation",
  "admin": {
    "userId": "u-admin123",
    "email": "joao.silva@xyz.com.br",
    "emailValidationSent": true
  },
  "createdAt": "2025-04-25T14:35:00Z"
}
```

#### Error Responses
- **400 Bad Request**: Dados inválidos ou incompletos
- **409 Conflict**: CNPJ/Email já cadastrado
- **422 Unprocessable Entity**: CNPJ com formato inválido

### GET /api/v1/tenants/{tenantId}/config
Obtém as configurações do tenant.

#### Response (200 OK)
```json
{
  "tenantId": "t-abc123",
  "schemaName": "tenant_abc123",
  "storageStrategy": "sharedDatabase",
  "timeZone": "America/Sao_Paulo",
  "locale": "pt-BR"
}
```

### PUT /api/v1/tenants/{tenantId}/config
Atualiza as configurações do tenant.

#### Request Body
```json
{
  "timeZone": "America/Sao_Paulo",
  "locale": "pt-BR",
  "preferences": {
    "theme": "light",
    "notificationChannels": ["email"]
  }
}
```

### POST /api/v1/tenants/{tenantId}/admin
Cria o usuário administrador inicial para o tenant.

#### Request Body
```json
{
  "nome": "João Silva",
  "email": "joao.silva@xyz.com.br",
  "cargo": "Diretor de Tecnologia"
}
```

#### Response (201 Created)
```json
{
  "userId": "u-admin123",
  "tenantId": "t-abc123",
  "email": "joao.silva@xyz.com.br",
  "status": "pendingValidation",
  "emailValidationSent": true,
  "isAdmin": true
}
```

## Company Registration API

### POST /api/v1/tenants/{tenantId}/companies
Registra uma nova empresa associada ao tenant.

#### Request Body
```json
{
  "razaoSocial": "Empresa XYZ Ltda",
  "nomeFantasia": "XYZ Solutions",
  "cnpj": "12.345.678/0001-99",
  "endereco": {
    "logradouro": "Av. Paulista",
    "numero": "1000",
    "complemento": "Sala 501",
    "bairro": "Bela Vista",
    "cidade": "São Paulo",
    "estado": "SP",
    "cep": "01310-100",
    "pais": "Brasil"
  },
  "contato": {
    "email": "contato@xyz.com.br",
    "telefone": "+55 11 98765-4321"
  }
}
```

#### Response (201 Created)
```json
{
  "companyId": "c-def456",
  "tenantId": "t-abc123",
  "status": "pendingValidation",
  "createdAt": "2025-04-25T14:35:00Z"
}
```

#### Error Responses
- **400 Bad Request**: Dados inválidos ou incompletos
- **404 Not Found**: Tenant não encontrado
- **409 Conflict**: CNPJ já cadastrado
- **422 Unprocessable Entity**: CNPJ com formato inválido

### GET /api/v1/tenants/{tenantId}/companies/{companyId}/status
Retorna o status atual do processo de onboarding de uma empresa.

#### Response (200 OK)
```json
{
  "companyId": "c-def456",
  "tenantId": "t-abc123",
  "status": "emailValidationPending",
  "completedSteps": ["registration", "initialValidation"],
  "pendingSteps": ["emailValidation", "accountSetup"],
  "lastActivity": "2025-04-25T14:35:00Z"
}
```

## User Management API

### POST /api/v1/tenants/{tenantId}/companies/{companyId}/users
Cria um novo usuário associado a uma empresa específica.

#### Request Body
```json
{
  "nome": "Maria Souza",
  "email": "maria@xyz.com.br",
  "cargo": "Gerente Financeiro",
  "permissoes": ["finance.read", "finance.write"]
}
```

#### Response (201 Created)
```json
{
  "userId": "u-xyz789",
  "companyId": "c-def456",
  "tenantId": "t-abc123",
  "email": "maria@xyz.com.br",
  "status": "pendingValidation",
  "emailValidationSent": true
}
```

### PUT /api/v1/users/validate-email
Valida o email de um usuário através do token.

#### Request Body
```json
{
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```

#### Response (200 OK)
```json
{
  "validated": true,
  "userId": "u-xyz789",
  "tenantId": "t-abc123",
  "companyId": "c-def456",
  "nextSteps": {
    "passwordSetup": "/account/set-password",
    "loginUrl": "/auth/login"
  }
}
```

## Email Validation API

### POST /api/v1/email-validation/resend
Solicita reenvio de email de validação.

#### Request Body
```json
{
  "email": "joao.silva@xyz.com.br"
}
```

#### Response (200 OK)
```json
{
  "sent": true,
  "expiresAt": "2025-05-02T14:30:00Z",
  "remainingAttempts": 2
}
```