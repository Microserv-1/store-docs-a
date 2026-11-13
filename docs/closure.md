# Fechamento da entrega — 13/11/2026

## Estado comprovado na AWS

A release `6617d8774baabe5943b9cbf4ec2e9a257edf3a97` foi aplicada pelo
`store-eks-release` do Jenkins na EC2, com credencial temporária da instance
role. Os dez testes de preparação, preflight AWS, dry-run e seis rollouts
passaram. O [registro de release](evidence/2026-11-13-eks/jenkins-release.json)
foi transcrito do console fornecido pelo operador; vincular também o número e
o console final do build na entrega.

Cluster `microserv-store`, região us-east-1, Kubernetes 1.35: Account, Auth,
Gateway, Product, Order e Exchange estão com duas réplicas disponíveis cada.
Redis, RabbitMQ, Prometheus e Grafana estão no mesmo namespace/cluster.
RabbitMQ usa PVC EBS gp3. O banco RDS PostgreSQL 16.15 é privado, criptografado,
Single-AZ db.t4g.small. Os três usuários de aplicação passaram em DDL/DML e
negação de acesso cruzado; o Job e Secret administrativos foram removidos.

- [Snapshot de serviços, imagens e HPA](evidence/2026-11-13-eks/cluster.json).
- [Bootstrap real informado pelo operador](evidence/2026-11-13-eks/rds-bootstrap.json).
- [Estado RDS anterior ao bootstrap](evidence/2026-11-13-eks/rds-ready.json).
- [Regressão local das permissões PostgreSQL](evidence/2026-11-13-eks/bootstrap-local-regression.json).
- [Prometheus: 12 targets saudáveis](evidence/2026-11-13-eks/prometheus-targets.json).

O smoke de Account/Auth/Gateway passou no EKS. O aceite de negócio passou em
**40 verificações** reais: autenticação, isolamento entre contas, identidade
forjada rejeitada, produtos/pedidos, Redis HIT/BYPASS/invalidação, fonte externa
de câmbio, conversão sem modificar USD, processamento RabbitMQ e replay
idempotente. Nessa rodada, USD 26,44 resultou em BRL 149,08 via open-er-api.
[Relatório do aceite](evidence/2026-11-13-eks/business-integration.json).

O cliente usou `127.0.0.1:18080` por port-forward de **store-eks**: a URL local
é o túnel para a aplicação AWS, não prova de execução local nem URL pública.
O túnel fixa uma réplica de Gateway; demonstrar o balanceamento desse serviço
exige carga interna contra seu Service. Consultas internas do Gateway aos
outros Services continuam sujeitas ao balanceamento Kubernetes.

## HPA e observabilidade comprovados

A rodada de login autorizada em 13/11 passou com 550 respostas HTTP 200,
zero erros e p95 de 1.187 ms. O HPA do Account escalou 2 → 4 → 5 → 2, com cinco
pods prontos no pico e três pods atendendo verificações de credenciais.
O [relatório de carga EKS](eks-load.md) contém vídeo, gráfico, métricas, eventos,
versões e limites. Não foram aumentados nós nem criados novos serviços AWS.
Isso não substitui carga progressiva dos fluxos de negócio nem vídeo humano.

## Orçamento adicional de US$ 20

O operador definiu em 13/11 um teto de planejamento de US$ 20 **adicionais**,
incluindo a manutenção dos recursos existentes. Isso não cria bloqueio de
cobrança na AWS. Inventário: dois workers t3.large e um Jenkins t3.large;
nodegroup desired=2, min=2, max=4. Não foi encontrado Cluster Autoscaler nem
Karpenter em Deployments. Não há ALB na configuração privada.

O [inventário de controle](evidence/2026-11-13-eks/cost-guard.json) é pontual,
não monitoramento contínuo. Para repetir uma checagem de configuração, sem
carga, sem alteração de recursos e sem API Cost Explorer:

```powershell
python scripts/aws-cost-guard.py --profile store-admin --context store-eks
```

A [estimativa com tarifas públicas](evidence/2026-11-13-eks/cost-estimate.json)
é **US$ 0,4426/h, US$ 10,62/dia, US$ 31,87/72h** em custos fixos aproximados.
Tráfego, NAT por GB, logs, créditos excedentes de CPU, backups excedentes,
impostos e conversão cambial não estão incluídos. Sem variáveis, US$ 20 cobrem
aproximadamente 45 horas; não se deve consumir toda essa margem planejada.
Não presumir créditos/free tier. A cotação não é fatura nem exportação do
Calculator. Veja [custos e SLA](operations.md).

Uma consulta inicial à API Cost Explorer foi feita antes de identificar sua
tarifa por chamada; ela pode acrescentar US$ 0,01. Não repeti-la sem autorização.
O dado retornado ainda não refletia EKS/EC2/RDS e não permite concluir gasto zero.

**Não iniciar carga, builds na AWS, aumento de nós, novos recursos ou mudanças
com potencial de custo adicional sem explicar o impacto e obter autorização.**
Também não parar/excluir recursos por conta própria: combinar a janela de
avaliação, a retenção dos dados e o procedimento de retomada.

## Checklist restante

- [ ] Confirmar rubrica/pesos, contratos originais de Account/Auth/Gateway e
  aceitação de Kind se Minikube for obrigatório nominalmente.
- [ ] Confirmar se o professor exige acesso independente à API por URL pública.
- [x] Executar rodada autorizada HPA EKS com latência/vazão/erros/CPU/memória,
  cinco réplicas prontas e vídeo do ciclo de escala.
- [x] Correlacionar carga de login e CPU e comprovar distribuição Account em
  três pods com métricas por instância, distinguindo os dois gargalos de A.
- [x] Consolidar recuperação local de um pod Account, com o mesmo digest EKS:
  30/30 respostas HTTP 200 e identidade preservada. Isso não é recuperação AWS.
- [ ] Concluir baseline e carga progressiva de negócio no EKS conforme o
  roadmap, com autorização própria; a rodada HPA executada foi de login.
- [x] Consolidar os links de evidências dos dois gargalos de B, incluindo
  medições de cache com commits finais; explicação e publicação individual são de B.
- [ ] Exportar estimativa do AWS Pricing Calculator para janela acadêmica e
  operação mensal; registrar link e hipóteses de consumo variável.
- [ ] Rever SLOs com medições da carga, mantendo SLA do fornecedor separado.
- [x] Publicar e verificar o Pages de A: build e URLs públicas passaram em 13/11.
- [ ] Atualizar o site de B por commits de B.
- [ ] Preencher nome acadêmico completo de A e identificação/grupo exigidos.
- [ ] Finalizar apresentação e vídeo humano de 1–2 minutos; ambos participam.
- [ ] Registrar revisão cruzada, ensaio individual e acesso aos repositórios.
- [ ] Cada integrante enviar seu próprio site no Blackboard e guardar recibo;
  conferir horário de 16/11/2026 e janela de retenção AWS antes do encerramento.

Nenhum checklist humano foi marcado como concluído automaticamente.

O [guia de conferência e defesa](delivery.md) reúne links, registro de revisão
cruzada, perguntas para ensaio e checagens individuais antes do Blackboard.
