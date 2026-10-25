# Checklist de entrega e apresentação

## Antes de entregar

- [ ] Conferir se `README.md`, `docs/` e `diagrams/` aparecem na raiz do GitHub.
- [ ] Abrir cada PNG e verificar legibilidade das setas, nomes e mensagens.
- [ ] Conferir se o fluxo de sucesso termina com pedido `CONFIRMED`.
- [ ] Conferir se a falha de pagamento dispara `ReleaseStock` e rejeita o pedido.
- [ ] Explicar que compensação é uma transação de negócio, não rollback distribuído.
- [ ] Explicar idempotência com Inbox e chave estável para pagamento.
- [ ] Mencionar Outbox, retry limitado, timeout, Circuit Breaker, DLQ e observabilidade.
- [ ] Justificar orquestração comparando-a com coreografia.

## Roteiro curto para apresentação

1. Apresente os três serviços e seus bancos independentes.
2. Explique por que uma transação ACID única não é apropriada para todo o fluxo.
3. Mostre o orquestrador e o fluxo feliz: reservar estoque → processar pagamento → confirmar pedido.
4. Mostre a falha de pagamento e a compensação: liberar estoque → rejeitar pedido.
5. Explique duplicatas e idempotência; destaque que timeout não significa necessariamente falha.
6. Finalize com Outbox, retries, DLQ, Circuit Breaker e métricas de monitoramento.

## Rubrica da atividade

- Modelagem correta do padrão Saga: 40%.
- Fundamentação arquitetural: 25%.
- Resiliência e tratamento de exceções: 20%.
- Qualidade da documentação e diagramas: 15%.
