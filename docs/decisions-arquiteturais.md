# Decisões arquiteturais

## ADR-001 — Coordenação da Saga por orquestração

**Status:** proposta aceita para o cenário acadêmico.

### Contexto
O fluxo de compra precisa reservar estoque, processar pagamento e confirmar ou rejeitar o pedido. Os serviços possuem bancos independentes, então uma única transação ACID não cobre o processo inteiro.

### Decisão
Adotar um Saga Orchestrator persistente. Ele envia comandos aos serviços, consome respostas e mantém o estado da Saga, incluindo etapa atual, tentativas e compensações já executadas.

### Alternativas
- **Coreografia:** serviços publicam eventos e os demais reagem. Evita um coordenador central, mas a sequência completa pode ficar distribuída entre consumidores e ser mais difícil de rastrear.
- **Orquestração:** coordenação explícita, mais fácil de visualizar e auditar, mas cria um componente crítico que precisa de persistência, alta disponibilidade e monitoramento.

### Consequências
- Transições explícitas e rastreáveis.
- Compensações são acionadas em ordem definida.
- O orquestrador deve ser idempotente e recuperar seu estado após reinício.
- Evitar lógica de negócio dos serviços dentro do orquestrador: cada serviço continua dono dos seus dados e regras locais.

## ADR-002 — Entrega de mensagens e idempotência

Assumir entrega **pelo menos uma vez** do broker. Duplicatas são esperadas; consumidores deduplicam por `messageId`/chave de negócio e persistem o resultado. Não prometer “exactly once” ponta a ponta apenas por configuração do broker.

## ADR-003 — Compensação de negócio

Se o pagamento falhar após a reserva de estoque, emitir `ReleaseStock` e, após resultado confirmado ou política de exceção, marcar o pedido como `REJECTED`. A compensação é uma ação de negócio independente, não um rollback físico das transações já confirmadas.
