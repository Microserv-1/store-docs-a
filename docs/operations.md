# Custos e SLA

## Infraestrutura e custos

Região: `us-east-1`. A configuração validada utiliza EKS, dois workers `t3a.medium`, Jenkins `t3a.small`, RDS `db.t4g.small` Single-AZ, Redis e RabbitMQ no cluster. O acesso privado não utiliza ALB.

Estimativa On-Demand com tarifas arquivadas em **13/11/2026**:

| Recurso | Quantidade | Custo aproximado (US$/h) |
|---|---:|---:|
| EKS Standard | 1 cluster | 0,1000 |
| Workers EC2 `t3a.medium` | 2 | 0,0752 |
| Jenkins EC2 `t3a.small` | 1 | 0,0188 |
| RDS `db.t4g.small` | 1 | 0,0320 |
| NAT Gateway | 1 | 0,0450 |
| IPv4 público do NAT | 1 | 0,0050 |
| EBS gp3 | 72 GiB | 0,0079 |
| Armazenamento RDS | 20 GiB | 0,0032 |
| **Total fixo estimado** | | **0,2870** |

O total corresponde a aproximadamente **US$ 6,89/dia**, **US$ 20,67 por 72 horas** ou **US$ 209,54 por 730 horas**. O armazenamento foi rateado pela convenção de 730 horas/mês.

Tráfego, processamento NAT por GB, logs, créditos excedentes de CPU, backups excedentes, snapshots, impostos e câmbio não estão incluídos. Os valores não representam a fatura nem um limite automático de gasto.

[Tarifas e unidades](evidence/2026-11-13-eks/right-sizing/public-price-rates.json) · [Memória de cálculo](evidence/2026-11-13-eks/right-sizing/cost-estimate.json).

A exportação oficial do AWS Pricing Calculator permanece pendente. Os [parâmetros de preenchimento](calculator-inputs.md) incluem a simulação mensal e a janela acadêmica.

## Serviços gerenciados e PaaS

| Serviço | Responsabilidade AWS | Responsabilidade da equipe |
|---|---|---|
| EKS | Operação do plano de controle Kubernetes | Nós, workloads, imagens, IAM, rede e capacidade |
| RDS | Infraestrutura e tarefas gerenciadas do banco | Schema, dados, credenciais, acesso e política de backups |
| EC2 Jenkins | Infraestrutura de computação | Sistema, Jenkins, agentes, credenciais e atualizações |

EKS e RDS fornecem capacidades gerenciadas de plataforma. A aplicação continua sob responsabilidade da equipe; containers locais não constituem, por si, um serviço PaaS.

## Metas e medições

| Indicador | Objetivo inicial | Situação |
|---|---|---|
| Disponibilidade na demonstração | 99% | Exige amostragem durante a janela; não comprovada como SLA mensal |
| Leitura de identidade | p95 abaixo de 500 ms, até 8 usuários | Exige cenário específico; não é comprovada pelo p95 agregado de negócio |
| Erros 5xx em carga estável | Abaixo de 1% | Rodada limitada apresentou zero erros na amostra |

Na carga de 14/11, o p95 de negócio variou entre 414 e 434 ms. Login apresentou p95 de 1.156 ms em 540 requisições. Login possui custo de Argon2id e é analisado separadamente da leitura de identidade.

Disponibilidade deve ser medida pela proporção de verificações bem-sucedidas, com intervalo e frequência registrados. Os [SLAs EKS](https://aws.amazon.com/eks/sla/) e [RDS](https://aws.amazon.com/rds/sla/) têm escopos e condições próprios; não equivalem ao SLA da aplicação. Banco Single-AZ, NAT único e dois workers limitam a tolerância a falhas.

## Encerramento

A infraestrutura continua gerando custos enquanto provisionada, inclusive durante pausas parciais. O inventário de configuração não mede a fatura. Ao concluir a avaliação, definir retenção de dados e seguir o [procedimento de encerramento](aws-shutdown.md).
