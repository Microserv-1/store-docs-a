# Microserv Store — Pessoa A

Projeto em dupla, edição 2026, com prazo informado de 16/11/2026. Confirmar horário no Blackboard.

Pessoa A: **Yuri Henrique** (identidade Git configurada; confirmar nome acadêmico completo). Pessoa B: **a identificar**. A revisão cruzada e a defesa individual continuam necessárias.

## Escopo e arquitetura

Account em Python/FastAPI, Auth em Java/Spring Boot e Gateway em Go. Infraestrutura e dois tratamentos de gargalos: observabilidade e balanceamento. Product, Order e Exchange Python pertencem à Pessoa B.

```mermaid
flowchart LR
    Client[Cliente] --> Gateway[Gateway Go]
    Gateway --> Account[Account FastAPI]
    Gateway --> Auth[Auth Spring Boot]
    Auth -->|verificação interna| Account
    Account --> DB[(PostgreSQL / RDS)]
    Gateway -. integração B .-> Product[Product]
    Gateway -. integração B .-> Order[Order]
    Gateway -. integração B .-> Exchange[Exchange Python]
    Order -.-> Product
    Order -.-> Exchange
    Metrics[Prometheus] --> Account
    Metrics --> Auth
    Metrics --> Gateway
    Grafana --> Metrics
```

Setas pontilhadas representam integrações previstas, que precisam ser validadas com os serviços reais de B.

## Repositórios e autoria

| Entrega | Repositório |
|---|---|
| Integração e infraestrutura | [store-root](https://github.com/Microserv-1/store-root) |
| Account | [store-account](https://github.com/Microserv-1/store-account) |
| Auth | [store-auth](https://github.com/Microserv-1/store-auth) |
| Gateway | [store-gateway](https://github.com/Microserv-1/store-gateway) |
| Documentação | [store-docs-a](https://github.com/Microserv-1/store-docs-a) |

Os serviços são privados na preparação. Confirmar acesso do avaliador ou visibilidade exigida. O site precisa ser público; a existência do workflow Pages não comprova publicação. Código auxiliado por IA exige revisão do integrante e defesa individual.
