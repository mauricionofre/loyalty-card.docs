# Plataforma de fidelidade de clientes

**Closing date:** 2023-08-31 <!-- YYYY-MM-DD. Ideally allow at least 7 days -->

**Status:** Proposed <!-- proposed, accepted, rejected, superseded -->

## Resumo
Pensando em comércios de pequeno porte e prestadores de serviços, que desejam realizar campanhas de fidelização de seus clientes, seja por meio de cashback ou pontuação. Propomos a criação de um aplicativo mobile para gestão de suas campanhas. Criando um ambiente organizado, de fácil controle e capaz de proporcionar uma experiência única aos seus clientes.

## Motivação
Hoje é muito comum a utilização de cartões de fidelidade, onde a cada compra ou serviço é preenchido um espaço no cartão e ao completar todo, há um desconto ou bônus para o portador.

Com isso temos uma oportunidade para auxiliar Comerciantes que desejam oferecer uma experiencia diferenciada aos seus clientes com a criação de campanhas de cashback.

Campanhas que apenas grandes comércios tinham, podem ser facilmente implantadas nos pequenos ou até prestadores de serviço, tornando um diferencial e ainda fidelizar seus clientes.

## Proposta

**Loyalty** - Plataforma de fidelidade

**Comerciante/Vendedor/Prestador de Serviço** - Usuário da plataforma que Vende ou Presta serviços ao clientes.

**Cliente** - Usuário da plataforma que toma o serviço ou realiza a compra.

Durante o cadastro do cliente no App é necessário informar uma chave, essa é utilizada para que o comerciante envie a transação de cashback, análogo ao PIX, cada cliente tem uma chave única e intransferível, hoje consideramos apenas o CPF.

O Comerciante após finalizar seu cadastro no App, pode criar suas campanhas, que podem ser do tipo Cashback ou Pontos.
- Cashback: Percentagem do valor da compra é inserido na conta fidelidade do cliente.
- Pontos: 1 ponto equivale a X Reais, Ex. 1 ponto para R$10, ao realizar uma compra de R$100 recebe na conta fidelidade 10 pontos.

#### Como ganhar Cashback/Pontos
1. O Cliente paga a compra normalmente, dinheiro, transferência PIX etc...
2. Solicita ao Comerciante seu Cashback/Pontos
3. O Comerciante realiza uma transação no App informando a Chave do cliente e valor total da compra
4. Cliente visualiza o Cashback/Pontos na sua conta fidelidade

Os valores em crédito são calculados conforme as campanhas cadastradas e ativas pelo Comerciante.

#### Como utilizar os Créditos
1. O Cliente acessa sua conta fidelidade no App
2. Escolhe o valor que deseja utilizar 
3. Gera o Voucher em QRCode com o valor escolhido
4. O Comerciante, com o App em mãos, escaneia o QRCode
5. Comerciante realiza o desconto no total da compra

Obs. As transações não são financeiras e cabe ao Comerciante cumprir com o desconto conforme a campanha cadastrada.

![SD_transactions](https://github.com/mauricionofre/loyalty-docs/assets/5630787/e9358044-30c8-47bc-9e49-6781c5bda788)

#### Conta Fidelidade
A conta fidelidade é a relação entre o Cliente e o Comerciante. Dessa forma um Cliente pode ter várias contas fidelidade mas terá apenas uma por Comerciante. Ex. O Cliente X, faz compras em Comercio A e Comercio B, serão exibidas duas Contas Fidelidade no seu App, Comercio A e Comercio B. 
O Cliente é obrigado a utilizar os créditos recebidos do Comerciante no próprio comerciante, não é permitido gerar um Voucher do Comercio A para utilização no Comercio B.

#### Transação
A transação é a movimentação do cashback/pontos entre Comerciante para o Cliente no momento da compra e também entre o Cliente para com o Comerciante no momento da utilização do Voucher.
Nesse primeiro momento consideramos apenas esses dois sentidos mas entende-se que pode haver uma evolução para transferências de Cashback/Pontos entre clientes.


## Evoluções
 - Permitir transações entre clientes.
 - Permitir cadastro de campanhas separadas por itens. 
   - Ex. cashback apenas para o serviço de Corte de cabelo
 - Permitir a integração com sistemas ERP para receber a transação de forma automatica sem necessidade de ação manual.
   - Plugado ao sistema de emissão de nota ou cupom fiscal.
<!-- Describe your proposal, with a focus on clarity of meaning. If it helps,
     you MAY use [RFC2119-style](https://www.ietf.org/rfc/rfc2119.txt) MUST,
     SHOULD and MAY language to help clarify your intentions. -->

