```mermaid
C4Component
    title Diagrama de Componentes do Loyalty Service
    
    Container_Boundary(loyaltyService, "Loyalty Service") {
        Component(programController, "Program Controller", "Spring REST Controller", "API para gerenciamento de programas")
        Component(rewardController, "Reward Controller", "Spring REST Controller", "API para gerenciamento de recompensas")
        Component(transactionController, "Transaction Controller", "Spring REST Controller", "API para transações e pontos")
        
        Component(programService, "Program Service", "Spring Service", "Lógica de negócio para programas")
        Component(rewardService, "Reward Service", "Spring Service", "Lógica de negócio para recompensas")
        Component(transactionService, "Transaction Service", "Spring Service", "Lógica de negócio para transações")
        Component(pointsCalculator, "Points Calculator", "Spring Service", "Calcula pontos baseado em regras")
        
        Component(programRepo, "Program Repository", "Spring Data JPA", "Acesso a dados de programas")
        Component(rewardRepo, "Reward Repository", "Spring Data JPA", "Acesso a dados de recompensas")
        Component(transactionRepo, "Transaction Repository", "Spring Data JPA", "Acesso a dados de transações")
        
        Component(tenantContext, "Tenant Context", "Spring Component", "Mantém contexto do tenant atual")
        Component(kafkaProducer, "Kafka Producer", "Spring Kafka", "Publica eventos de fidelidade")
    }
    
    ContainerDb(postgresDB, "PostgreSQL", "Database")
    Container(apiGateway, "API Gateway")
    Container(notificationService, "Notification Service")
    ContainerDb(redisCache, "Redis Cache")
    
    Rel(apiGateway, programController, "Chama", "HTTP/JSON")
    Rel(apiGateway, rewardController, "Chama", "HTTP/JSON")
    Rel(apiGateway, transactionController, "Chama", "HTTP/JSON")
    
    Rel(programController, programService, "Usa")
    Rel(rewardController, rewardService, "Usa")
    Rel(transactionController, transactionService, "Usa")
    
    Rel(programService, programRepo, "Usa")
    Rel(rewardService, rewardRepo, "Usa")
    Rel(transactionService, transactionRepo, "Usa")
    Rel(transactionService, pointsCalculator, "Usa")
    
    Rel(programRepo, postgresDB, "Lê/Escreve", "JDBC")
    Rel(rewardRepo, postgresDB, "Lê/Escreve", "JDBC")
    Rel(transactionRepo, postgresDB, "Lê/Escreve", "JDBC")
    
    Rel_D(programRepo, tenantContext, "Usa para filtrar por tenant")
    Rel_D(rewardRepo, tenantContext, "Usa para filtrar por tenant")
    Rel_D(transactionRepo, tenantContext, "Usa para filtrar por tenant")
    
    Rel(transactionService, kafkaProducer, "Publica eventos")
    Rel(kafkaProducer, notificationService, "Envia eventos para")
    
    Rel(programService, redisCache, "Armazena cache", "Redis Protocol")
    Rel(transactionService, redisCache, "Armazena cache", "Redis Protocol")
```