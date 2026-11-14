# Microserv Store — Pessoa A

Projeto em dupla, edição 2026, com prazo informado de 16/11/2026. Confirmar horário no Blackboard.

Pessoa A: **Yuri Henrique**.
Pessoa B: **Gustavo Lina** (`gustavolafz`). Nomes informados por Yuri em 14/11;
grupo e edição acadêmica ainda precisam ser confirmados. A revisão cruzada
da versão final e a defesa individual continuam necessárias.

**Estado atual:** [AWS pausada em 14/11](aws-pause.md), após testes aprovados.
Documentação e vídeos seguem públicos; a API fica indisponível até retomada.

Em 13/11, os seis serviços foram implantados no **Amazon EKS**, por Jenkins
na EC2, e passaram no smoke e em 40 verificações integradas. Confira o
[fechamento e orçamento](closure.md). O acesso à API é por túnel autorizado;
a documentação é independente desse acesso privado.

## Escopo e arquitetura

Account em Python/FastAPI, Auth em Java/Spring Boot e Gateway em Go. Infraestrutura e dois tratamentos de gargalos: observabilidade e balanceamento. Product, Order e Exchange Python pertencem à Pessoa B.

```mermaid
flowchart LR
    Client[Cliente] --> Gateway[Gateway Go]
    Gateway --> Account[Account FastAPI]
    Gateway --> Auth[Auth Spring Boot]
    Auth -->|verificação interna| Account
    Account --> Accounts[(RDS: accounts)]
    Gateway --> Product[Product]
    Gateway --> Order[Order]
    Gateway --> Exchange[Exchange Python]
    Order --> Product
    Order --> Exchange
    Product --> Products[(RDS: products)]
    Order --> Orders[(RDS: orders)]
    Product <--> Redis[(Redis)]
    Order <--> RabbitMQ[(RabbitMQ + EBS)]
    Exchange --> FX[Provedor externo de câmbio]
    Metrics[Prometheus] --> Account
    Metrics --> Auth
    Metrics --> Gateway
    Metrics --> Product
    Metrics --> Order
    Metrics --> Exchange
    Grafana --> Metrics
```

As integrações de negócio foram testadas com os serviços reais de B, Redis, RabbitMQ e um provedor externo de câmbio. Consulte o [relatório de integração de 11/11](integration.md).

## Repositórios e autoria

| Entrega | Repositório |
|---|---|
| Integração e infraestrutura | [store-root](https://github.com/Microserv-1/store-root) |
| Account | [store-account](https://github.com/Microserv-1/store-account) |
| Auth | [store-auth](https://github.com/Microserv-1/store-auth) |
| Gateway | [store-gateway](https://github.com/Microserv-1/store-gateway) |
| Product | [store-product](https://github.com/Microserv-1/store-product) |
| Order | [store-order](https://github.com/Microserv-1/store-order) |
| Exchange | [store-exchange](https://github.com/Microserv-1/store-exchange) |
| Documentação de B | [store-docs-b](https://github.com/Microserv-1/store-docs-b) |
| Documentação | [store-docs-a](https://github.com/Microserv-1/store-docs-a) |

Os serviços são privados na preparação. Confirmar acesso do avaliador ou visibilidade exigida. O site precisa ser público; a existência do workflow Pages não comprova publicação. Código auxiliado por IA exige revisão do integrante e defesa individual.
