# Contratos de mensagens (exemplos)

Todas as mensagens devem incluir metadados de rastreamento e versão do esquema. Payloads abaixo são ilustrativos.

## Envelope comum

```json
{
  "messageId": "uuid-unico-da-mensagem",
  "sagaId": "uuid-da-saga",
  "correlationId": "uuid-da-requisicao",
  "occurredAt": "2026-10-25T12:00:00Z",
  "schemaVersion": 1,
  "type": "OrderCreated",
  "payload": {}
}
```

## Mensagens principais

| Tipo | Produtor | Consumidor | Finalidade |
|---|---|---|---|
| `OrderCreated` | Order Service | Orquestrador | Iniciar a Saga para um pedido persistido |
| `ReserveStock` | Orquestrador | Inventory Service | Solicitar reserva de itens |
| `StockReserved` | Inventory Service | Orquestrador | Confirmar reserva |
| `StockReservationFailed` | Inventory Service | Orquestrador | Informar impossibilidade de reservar |
| `ProcessPayment` | Orquestrador | Payment Service | Solicitar processamento de pagamento |
| `PaymentSucceeded` | Payment Service | Orquestrador | Confirmar pagamento |
| `PaymentFailed` | Payment Service | Orquestrador | Informar falha de pagamento |
| `ReleaseStock` | Orquestrador | Inventory Service | Compensar a reserva |
| `StockReleased` | Inventory Service | Orquestrador | Confirmar compensação |
| `ConfirmOrder` | Orquestrador | Order Service | Finalizar pedido com sucesso |
| `RejectOrder` | Orquestrador | Order Service | Encerrar pedido com falha |

## Regras de idempotência

1. O consumidor verifica `messageId` na Inbox antes de aplicar o efeito.
2. A gravação da Inbox e a alteração de negócio devem ocorrer na mesma transação local sempre que possível.
3. Comandos externos de pagamento devem usar uma chave de idempotência estável, derivada de `sagaId` + operação, junto ao gateway quando suportado.
4. Uma mensagem duplicada deve retornar/reproduzir o resultado já registrado, sem reservar estoque ou cobrar novamente.
5. Para `ReleaseStock`, liberar somente a reserva associada à Saga e permitir repetição sem saldo negativo ou efeito duplicado.

## Estados sugeridos

- Pedido: `PENDING`, `CONFIRMED`, `REJECTED`.
- Saga: `STARTED`, `STOCK_RESERVED`, `PAYMENT_PROCESSING`, `COMPENSATING`, `COMPLETED`, `FAILED`, `MANUAL_INTERVENTION`.

## Tratamento de falhas

- Erro transitório: retry limitado com backoff exponencial e jitter.
- Timeout: consultar estado ou repetir com a mesma chave idempotente; não presumir falha definitiva.
- Tentativas esgotadas: enviar para DLQ, alertar e registrar contexto suficiente para diagnóstico.
- Compensação falha: manter Saga em estado recuperável e tentar novamente; não declarar encerramento bem-sucedido antes de resolver ou encaminhar formalmente para intervenção manual.
