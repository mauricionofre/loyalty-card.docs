```mermaid
erDiagram
    COMERCIANTE ||--o{ CAMPANHA : "cria"
    COMERCIANTE ||--o{ TRANSACAO : "gera"
    USUARIO ||--o{ WALLET : "possui"
    WALLET ||--o{ TRANSACAO : "registra"
    CAMPANHA ||--o{ TRANSACAO : "origina"
    
    COMERCIANTE {
        int id PK
        string razaoSocial
        string nomeFantasia
        string cnpj
        string email
        string telefone
        string segmento
        datetime dataCadastro
        bool ativo
    }
    
    USUARIO {
        int id PK
        string nome
        string email
        string cpf
        string telefone
        datetime dataCadastro
        bool ativo
    }
    
    WALLET {
        int id PK
        int usuarioId FK
        int comercianteId FK
        decimal saldo
        int pontos
        string status
        datetime dataCriacao
        datetime dataAtualizacao
    }
    
    TRANSACAO {
        int id PK
        int walletId FK
        int comercianteId FK
        int campanhaId FK
        decimal valorCompra
        decimal valorCashback
        int pontosGanhos
        string status
        string metodoPagamento
        datetime dataTransacao
    }
    
    CAMPANHA {
        int id PK
        int comercianteId FK
        string nome
        string descricao
        string tipo "PONTOS ou VALOR"
        decimal percentualCashback
        int pontosPorValor
        decimal valorMinimo
        datetime dataInicio
        datetime dataTermino
        bool ativa
    }
    
    RESGATE {
        int id PK
        int walletId FK
        int transacaoId FK "null"
        decimal valorResgatado
        int pontosResgatados
        string status
        datetime dataResgate
    }

```

Este diagrama Mermaid representa as principais entidades do sistema CaaS com seus relacionamentos. As entidades incluídas são:

1. **COMERCIANTE**: Armazena dados dos estabelecimentos comerciais.
2. **USUARIO**: Contém informações dos usuários finais que utilizam o sistema de cashback.
3. **WALLET**: Representa a carteira virtual de cada usuário por estabelecimento.
4. **TRANSACAO**: Registra todas as operações de cashback geradas nas compras.
5. **CAMPANHA**: Contém as configurações das campanhas de cashback criadas pelos comerciantes.
6. **RESGATE**: Registra as operações de resgate de cashback/pontos pelos usuários.

Os relacionamentos estabelecidos seguem a lógica de negócio descrita anteriormente, onde:
- Um comerciante pode criar várias campanhas
- Um usuário possui múltiplas wallets (uma por estabelecimento)
- As transações estão associadas a uma wallet, um comerciante e uma campanha
- Os resgates estão associados a uma wallet
