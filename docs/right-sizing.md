# Máquinas menores e validação — 13/11/2026

A redução foi solicitada e autorizada por Yuri após medir a aplicação real.
O problema era o tamanho das três máquinas, não a família T3 por si só.

| Uso | Antes | Depois |
|---|---|---|
| EKS | 2 × t3.large, 8 GiB por nó | 2 × t3a.medium, 4 GiB por nó |
| Jenkins EC2 | t3.large, 8 GiB | t3a.small, 2 GiB |
| Grupo de nós | application | application-economy; min/desired/max = 2 |

Os serviços continuam em imagens amd64, sem mudança nos microsserviços de B.
Não foram criados ALB, banco ou NAT adicionais. O grupo antigo foi drenado
respeitando os PodDisruptionBudgets e removido; a migração usou dois nós
temporários menores. O Jenkins manteve o disco, os jobs e as credenciais.

## Por que este tamanho

Antes da migração, 28 pods ativos pediam aproximadamente 2,83 vCPU e
4,00 GiB de memória. Dois nós small não oferecem memória alocável suficiente
nem capacidade de pods com o VPC CNI atual. Um único nó de 2 vCPU também não
acomoda os requests de CPU. Cada novo nó oferece 1.930 millicores,
3.383.820 KiB de memória alocável e limite de 17 pods.

Assim, dois medium são o menor perfil avaliado para esta topologia, com
folga moderada. Não é prova de que sejam o mínimo absoluto após reengenharia.
Na conferência final, o uso de memória dos nós foi 38% e 44%.
O Jenkins iniciou com aproximadamente 521 MiB usados de 1.915 MiB totais.

## Testes e persistência

- [Snapshot final e versões testadas](evidence/2026-11-13-eks/right-sizing/migration.json):
  todos os Deployments prontos e as duas EC2 antigas terminadas.
- Smoke de Account/Auth/Gateway aprovado após a migração.
- [40 verificações reais dos seis serviços](evidence/2026-11-13-eks/right-sizing/business-integration.json)
  aprovadas: USD 26,44, conversão BRL 149,08 via open-er-api, Redis,
  RabbitMQ, replay idempotente e isolamento entre contas.
- [Jenkins build #5](evidence/2026-11-13-eks/right-sizing/jenkins-validation.json)
  SUCCESS no host pequeno, com `DEPLOY_EKS=false`.
- RDS e o PVC original RabbitMQ foram preservados. No remount do RabbitMQ,
  foi corrigida na infraestrutura da raiz a permissão 0600 do cookie Erlang,
  sem ler seu conteúdo, recriar credenciais ou apagar dados.
- [Inventário final](evidence/2026-11-13-eks/right-sizing/cost-guard.json)
  verifica duas EC2 medium e uma small ligadas, grupo limitado a dois nós e
  ausência de Cluster Autoscaler/Karpenter.

O teste de carga e vídeo de HPA anteriores foram executados nas máquinas
grandes. Seus valores de latência/vazão não são benchmark do novo perfil.
O aceite funcional passou nas menores; não foi executada nova carga de
estresse. A capacidade atual não garante todos os seis HPAs no máximo ao
mesmo tempo. O Jenkins pequeno foi validado para o pipeline de release;
builds Java pesados podem exigir ajuste de heap ou mais memória.

## Estimativa revisada

Tarifas Linux On-Demand us-east-1: t3a.medium US$ 0,0376/h; t3a.small
US$ 0,0188/h. [Tarifas com SKU](evidence/2026-11-13-eks/right-sizing/public-price-rates.json)
consultadas na AWS Price List Query API em 13/11/2026.

| Estimativa fixa | Antes | Depois |
|---|---:|---:|
| EC2 por hora | US$ 0,2496 | US$ 0,0940 |
| Toda a topologia por dia | US$ 10,62 | US$ 6,89 |
| Toda a topologia por 72h futuras | US$ 31,87 | US$ 20,67 |

[Cálculo completo](evidence/2026-11-13-eks/right-sizing/cost-estimate.json):
redução aproximada de 62% em EC2 e 35% no total fixo. EKS, RDS, NAT, IP e
armazenamento continuam cobrando. Exclui tráfego, créditos excedentes CPU,
logs, backups excedentes e impostos. Não mede a fatura nem desconta o gasto
já acumulado do teto adicional de US$ 20. Não é garantia de teto.

Referências: [tipos T3](https://aws.amazon.com/ec2/instance-types/t3/),
[escolha de instâncias EKS](https://docs.aws.amazon.com/eks/latest/userguide/choosing-instance-type.html),
[atualização de grupos gerenciados](https://docs.aws.amazon.com/eks/latest/userguide/update-managed-node-group.html).
