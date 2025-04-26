# Fluxo de Eventos e Comandos - Onboarding

```mermaid
flowchart TD
    subgraph "Tenant Service"
        C1[RegistrarEmpresaCommand]
        EH1[TenantRegistradoHandler]
        E1[TenantRegistrado]
        S1[Validador CNPJ]
        EH4[OnboardingCompletoHandler]
        C6[AtivarTenantCommand]
        E6[TenantAtivado]
    end
    
    subgraph "User & Identity Service"
        C2[CriarUsuarioAdminCommand]
        E2[UsuarioAdminCriado]
        C4[ValidarEmailUsuarioCommand]
        E4[EmailValidado]
        EH2[UsuarioCriadoHandler]
        EH3[EmailValidadoHandler]
        C5[CompletarOnboardingCommand]
        E5[OnboardingCompleto]
    end
    
    subgraph "Notification Service"
        C3[EnviarEmailValidacaoCommand]
        E3[EmailValidacaoEnviado]
        S2[Provedor Email]
        S3[Gerador Templates]
    end
    
    C1 --> E1
    E1 --> EH1
    EH1 --> C2
    C2 --> E2
    E2 --> EH2
    EH2 --> C3
    C3 --> S3
    S3 --> S2
    S2 --> E3
    C1 --> S1
    S1 -.-> E1
    C4 --> E4
    E4 --> EH3
    EH3 --> C5
    C5 --> E5
    E5 --> EH4
    EH4 --> C6
    C6 --> E6

    %% Fluxo entre serviços
    E2 --> C3
    E3 -.-> C4
    E5 --> EH4
```