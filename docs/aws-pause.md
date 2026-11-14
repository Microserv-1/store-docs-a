# Pausa autorizada da AWS — 14/11/2026

Yuri autorizou a pausa temporária após a bateria de testes. A aplicação e o
Jenkins ficam indisponíveis até retomada autorizada; os sites e vídeos no
GitHub Pages continuam públicos. O [registro de estado](evidence/2026-11-14-pause/state.json)
contém os estados efetivamente verificados e o horário da confirmação.
Confirmação: **14/11 às 12:09 BRT (15:09 UTC)**. A API informou possível
reinício automático do RDS em **21/11 às 12:08 BRT (15:08 UTC)**; não deixar
essa data substituir a decisão de encerrar os recursos após a avaliação.

## Estado e preservação

| Componente | Estado após a pausa |
|---|---|
| EKS / VPC / NAT | Mantidos; plano de controle e rede continuam cobrando |
| application-economy | min=0, desired=0, max=2 |
| Duas EC2 worker | Terminadas pelo grupo gerenciado; serão recriadas na retomada |
| EC2 Jenkins | Stopped; disco de 30 GiB preservado |
| RDS PostgreSQL | Stopped; bancos e endpoint preservados |
| RabbitMQ | StatefulSet com zero réplicas; PVC/PV gp3 de 2 GiB preservado |
| Redis / Prometheus | Cache e dados efêmeros podem reiniciar vazios; evidências já arquivadas |

Antes da parada, foi confirmado Jenkins ocioso e fila vazia. O RabbitMQ foi
encerrado antes dos workers, com retenção `Retain` para PVC. O desligamento
total dos pods foi gracioso, liberando a drenagem sem exigir disponibilidade
mínima de aplicação durante a pausa. Nenhum banco, PVC ou namespace foi excluído
e nenhum snapshot adicional foi solicitado.

Os [volumes conferidos](evidence/2026-11-14-pause/retained-volumes.json)
permanecem: 30 GiB gp3 do Jenkins associado à EC2 parada e 2 GiB gp3 do
RabbitMQ disponível, com PVC ainda Bound.

A estimativa conservadora previamente apresentada é cerca de **US$ 0,161/h
ou US$ 3,86/dia**, mantendo EKS/NAT/IP e armazenamento, antes de variáveis.
Ela conserva o armazenamento dos workers no cálculo; volumes raiz terminados
podem reduzir esse subtotal. Não é medição da fatura nem garantia de teto.
O RDS pode iniciar automaticamente após sete dias de parada; conferir o
horário informado pela API no registro e decidir o encerramento após a avaliação.
[Documentação RDS](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_StopInstance.html).

## Retomada — referência, não executada

A retomada volta a cobrar a capacidade em operação, estimada em US$ 0,287/h
mais variáveis. Confirmar autorização e margem para inicialização antes de
executar. Não prometer retomada instantânea nem iniciar carga automaticamente.

1. Renovar o login `store-login` e validar a identidade `store-admin` na conta
   correta. Atualizar o IP `/32` do endpoint EKS se o IP do operador mudar.
2. Iniciar o RDS e aguardar `available`.
3. Iniciar a EC2 Jenkins e aguardar o serviço ficar saudável; conferir se saiu
   do modo quieto antes de solicitar um pipeline.
4. Restaurar `application-economy` para min=2, desired=2, max=2 e aguardar
   **dois nós Ready**, do tipo t3a.medium.
5. Restaurar RabbitMQ para uma réplica, usando o mesmo PVC. Aguardar o broker
   saudável; a correção de permissão do cookie já está no StatefulSet.
6. Aguardar métricas e os seis Deployments, conferir ausência de Pending e
   repetir o inventário de custos. Validar o fluxo com um smoke autorizado.

Comandos para os passos de retomada, **somente quando autorizados**:

```powershell
aws rds start-db-instance --profile store-admin --region us-east-1 --db-instance-identifier microserv-store-rds-database-xaw4nsp3iyky
aws rds wait db-instance-available --profile store-admin --region us-east-1 --db-instance-identifier microserv-store-rds-database-xaw4nsp3iyky
aws ec2 start-instances --profile store-admin --region us-east-1 --instance-ids i-03245ec6369003f79
aws eks update-nodegroup-config --profile store-admin --region us-east-1 --cluster-name microserv-store --nodegroup-name application-economy --scaling-config minSize=2,maxSize=2,desiredSize=2
kubectl --context store-eks get nodes
```

Depois de confirmar os dois nós Ready:

```powershell
kubectl --context store-eks -n store scale statefulset/rabbitmq --replicas=1
kubectl --context store-eks -n store rollout status statefulset/rabbitmq --timeout=300s
python scripts/aws-cost-guard.py --profile store-admin --context store-eks
```

O guard normal espera três EC2 ligadas e dois workers; não deve ser
interpretado como falha inesperada enquanto o ambiente está pausado. A
configuração versionada continua descrevendo a capacidade normal de dois
workers; a pausa é um estado operacional temporário, não uma nova topologia.

Para encerrar definitivamente, usar o [procedimento separado](aws-shutdown.md)
após autorizar a retenção/descarte de dados. Pausa não significa exclusão dos
recursos nem encerramento de todos os custos.
