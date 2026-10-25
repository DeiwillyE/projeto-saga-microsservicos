# Projeto Saga — Fluxo de compra em microsserviços

> Atividade prática de Arquitetura: desenho e modelagem do padrão Saga em um projeto real.

## 1. Contexto

O fluxo de compra é dividido em três serviços independentes, cada um com seu próprio banco de dados:

- **Order Service**: registra o pedido e acompanha seu estado.
- **Inventory Service**: reserva e libera temporariamente os itens.
- **Payment Service**: autoriza/captura o pagamento por meio de um gateway externo.

Como não existe uma transação ACID distribuída envolvendo os três bancos, a consistência é obtida por uma sequência de transações locais coordenadas por uma Saga.

## 2. Estratégia escolhida: orquestração

A proposta usa **Saga Orchestrator** para coordenar a sequência. A escolha é adequada ao exercício porque o fluxo de compra tem passos ordenados, compensações explícitas e necessidade de observar o estado da operação. A orquestração centraliza a máquina de estados e facilita auditoria, rastreamento e recuperação. O custo é adicionar um componente de coordenação que precisa ser altamente disponível e persistir seu estado.

A decisão não significa que coreografia seja incorreta: ela pode funcionar bem em fluxos menores e menos centralizados. Para este cenário didático, a clareza operacional da orquestração pesa mais.

## 3. Fluxo de sucesso

1. O cliente solicita a criação de um pedido.
2. Order Service persiste o pedido como `PENDING` e publica `OrderCreated` via Transactional Outbox.
3. O orquestrador inicia a Saga e solicita `ReserveStock`.
4. Inventory Service reserva os itens e responde `StockReserved`.
5. O orquestrador solicita `ProcessPayment`.
6. Payment Service processa o pagamento e responde `PaymentSucceeded`.
7. O orquestrador solicita a confirmação do pedido; Order Service muda o estado para `CONFIRMED` e publica `OrderConfirmed`.

## 4. Fluxo de falha e compensação

Se o pagamento falhar após a reserva do estoque, o orquestrador solicita `ReleaseStock` e depois determina que o pedido seja marcado como `REJECTED`. A compensação não é um rollback ACID: é uma nova transação de negócio que tenta neutralizar o efeito anterior. Compensações também podem falhar e, nesse caso, devem ser repetidas com segurança e encaminhadas para intervenção operacional após o limite de tentativas.

## 5. Resiliência e garantias

- **Idempotência**: cada comando/evento carrega `messageId` e `sagaId`; consumidores registram mensagens processadas em uma Inbox com restrição única. Repetições não executam novamente o efeito de negócio.
- **Transactional Outbox**: a alteração local e o evento a publicar são gravados na mesma transação de banco. Um relay publica a mensagem posteriormente.
- **Retry**: tentativas limitadas com backoff exponencial e jitter para falhas transitórias; não repetir indefinidamente erros permanentes.
- **Timeout**: toda chamada espera por prazo definido. Timeout não prova que a operação remota falhou; consultar o estado ou repetir com a mesma chave idempotente.
- **Circuit Breaker**: interrompe temporariamente chamadas a uma dependência que falha repetidamente, evitando saturação em cascata. Não substitui idempotência.
- **DLQ**: mensagens que excedem as tentativas vão para uma dead-letter queue para análise, correção e reprocessamento controlado.
- **Observabilidade**: propagar `correlationId`, `sagaId` e `messageId`; registrar transições de estado, latência, retries, compensações e DLQ.

## 6. Diagramas

- [Visão geral da arquitetura (PNG)](diagrams/saga-architecture.png)
- [Fluxo de sucesso (PNG)](diagrams/sequence-success-flow.png)
- [Fluxo de compensação (PNG)](diagrams/sequence-compensation-flow.png)
- Os arquivos `.puml` e `.dot` são fontes editáveis dos diagramas.

## 7. Estrutura do repositório

```text
projeto-saga/
├── README.md
├── LICENSE
├── .gitignore
├── diagrams/
│   ├── saga-architecture.dot / .png / .pdf
│   ├── sequence-success-flow.puml / .png / .pdf
│   └── sequence-compensation-flow.puml / .png / .pdf
└── docs/
    ├── decisions-arquiteturais.md
    ├── contratos-mensagens.md
    └── checklist-entrega.md
```

## 8. Referência principal

Chris Richardson, *Saga pattern*. Microservices.io: https://microservices.io/patterns/data/saga.html

## 9. Escopo

Este repositório documenta uma proposta arquitetural; não contém implementação executável dos microsserviços. Nomes de eventos e estados são exemplos de contrato a serem validados na implementação.
