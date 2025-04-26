```mermaid
C4Container
    title Diagrama de Containers do Loyalty Card System
    
    System_Boundary(loyaltySystem, "Loyalty Card System") {
        Container(apiGateway, "API Gateway", "Kong/Envoy", "Roteamento e gerenciamento de APIs")
        
        Container(adminPortal, "Portal Admin", "React/Next.js", "Interface para administradores do sistema")
        Container(companyPortal, "Portal Empresa", "React/Next.js", "Interface para empresas clientes")
        Container(consumerPortal, "Portal Consumidor", "React PWA", "Interface para consumidores finais")
        Container(mobileApp, "App Mobile", "React Native", "Aplicativo para consumidores")
        
        Container(tenantService, "Tenant Service", "Spring Boot", "Gerenciamento de tenants")
        Container(authService, "Auth Service", "Spring Boot", "Autenticação e autorização")
        Container(userService, "User Service", "Spring Boot", "Gerenciamento de usuários")
        Container(loyaltyService, "Loyalty Service", "Spring Boot", "Lógica do programa de fidelidade")
        Container(notificationService, "Notification Service", "Spring Boot", "Gerenciamento de notificações")
        
        ContainerDb(postgresDB, "PostgreSQL", "Database", "Armazenamento persistente com schemas por tenant")
        ContainerDb(redisCache, "Redis", "Cache", "Cache distribuído")
    }
    
    Person(administrador, "Administrador")
    Person(companyAdmin, "Admin da Empresa")
    Person(consumidor, "Consumidor Final")
    Person(funcionario, "Funcionário")
    
    System_Ext(emailSystem, "Sistema de Email")
    System_Ext(paymentGateway, "Gateway de Pagamento")
    
    Rel(administrador, adminPortal, "Utiliza", "HTTPS")
    Rel(companyAdmin, companyPortal, "Utiliza", "HTTPS")
    Rel(consumidor, consumerPortal, "Utiliza", "HTTPS")
    Rel(consumidor, mobileApp, "Utiliza")
    Rel(funcionario, companyPortal, "Utiliza", "HTTPS")
    
    Rel(adminPortal, apiGateway, "Acessa via", "HTTPS/JSON")
    Rel(companyPortal, apiGateway, "Acessa via", "HTTPS/JSON")
    Rel(consumerPortal, apiGateway, "Acessa via", "HTTPS/JSON")
    Rel(mobileApp, apiGateway, "Acessa via", "HTTPS/JSON")
    
    Rel(apiGateway, tenantService, "Roteia para", "HTTP/JSON")
    Rel(apiGateway, authService, "Roteia para", "HTTP/JSON")
    Rel(apiGateway, userService, "Roteia para", "HTTP/JSON")
    Rel(apiGateway, loyaltyService, "Roteia para", "HTTP/JSON")
    
    Rel(tenantService, postgresDB, "Lê/Escreve", "JDBC")
    Rel(authService, postgresDB, "Lê/Escreve", "JDBC")
    Rel(userService, postgresDB, "Lê/Escreve", "JDBC")
    Rel(loyaltyService, postgresDB, "Lê/Escreve", "JDBC")
    
    Rel(authService, redisCache, "Armazena sessões", "Redis Protocol")
    Rel(loyaltyService, redisCache, "Cache", "Redis Protocol")
    
    Rel(notificationService, emailSystem, "Envia emails", "SMTP")
    Rel(tenantService, paymentGateway, "Processa pagamentos", "HTTPS/API")
```