# Encerramento da AWS após a avaliação

Procedimento preparado em 14/11/2026. **Nenhuma exclusão foi executada.**
Yuri decidiu manter as máquinas ligadas até a próxima bateria. Confirmar a
janela exigida pelo professor e a autorização de encerramento antes de usar
os comandos de exclusão abaixo. A aplicação ficará indisponível e a exclusão
não é reversível como uma pausa.

## Confirmar o alvo e a retenção

Inventário conhecido de 13/11, a revalidar com sessão renovada:

| Recurso | Alvo deste projeto |
|---|---|
| Conta / região | 975049967635 / us-east-1 |
| EKS / grupo atual | microserv-store / application-economy |
| VPC criada pelo eksctl | vpc-0fed1ba2f11b008ac |
| Stack RDS | microserv-store-rds |
| Banco | microserv-store-rds-database-xaw4nsp3iyky |
| Jenkins EC2 | i-03245ec6369003f79 |
| Stack de identidade Jenkins | microserv-store-jenkins-iam |
| Security group Jenkins | sg-0131b171b13c02060 |
| Volume RabbitMQ | PVC data-rabbitmq-0; PV pvc-61f5495a-030c-49f1-ae20-ba9c58715a73 |

O PV é um identificador Kubernetes, **não** um ID EBS `vol-...`. Obter o
`spec.csi.volumeHandle` do PV antes de excluir o cluster e registrar a decisão
de retenção desse volume. Não apagar volumes por semelhança de nomes.

Guardar relatórios, vídeos, commits e digests nos sites/repositórios antes
de encerrar. Não copiar Secrets, senhas, JWTs ou credenciais Jenkins para o
Git. Backups do Jenkins contêm segredos e precisam de destino privado.
Combinar se os dados são descartáveis ou precisam ser recuperáveis e por
quanto tempo. Criar snapshots extras também exige decisão de custo.

O template RDS usa `DeletionPolicy: Snapshot`: excluir sua stack cria um
snapshot final que permanece e pode cobrar armazenamento. Decidir se vai
mantê-lo e quando excluí-lo. Isso é comportamento previsto, não comprovação
de custo zerado. [Política CloudFormation](https://docs.aws.amazon.com/AWSCloudFormation/latest/TemplateReference/aws-attribute-deletionpolicy.html).

## Sequência após autorização

1. Conferir conta, região e recursos atuais com a identidade administrativa.
   Parar a carga e aguardar pipelines terminarem. Não usar o papel restrito do
   Jenkins para administrar a infraestrutura.
2. Listar Ingress/Services. Se houver ALB/NLB, excluir seus recursos Kubernetes
   e aguardar o balanceador desaparecer **antes** de remover os controllers
   e o cluster. O último inventário usava apenas ClusterIP/port-forward.
   [Ordem recomendada EKS](https://docs.aws.amazon.com/eks/latest/userguide/delete-cluster.html).
3. Preservar as evidências e os dados acordados. Se a retenção depender de
   snapshot, verificar sua conclusão antes de excluir a origem. Não presumir
   que a existência de PVC ou política Retain equivale a backup.
4. Excluir a stack RDS e aguardar seu término. Isso remove banco, subnet group
   e security group dependentes da VPC, mantendo o snapshot previsto.
5. Encerrar a EC2 Jenkins após a decisão sobre seu disco. Conferir
   `DeleteOnTermination` do volume raiz: ele pode ser apagado junto com a EC2.
   Apenas parar a EC2 mantém o EBS e não remove a interface da VPC.
6. Depois de a EC2 terminar e sua interface ser removida, revisar a regra
   adicionada ao security group do cluster que referencia o grupo Jenkins.
   Remover essa referência e o grupo Jenkins se forem exclusivos do projeto.
   Não revogar regras ou apagar grupos de outros ambientes.
7. Excluir o cluster pelo eksctl com espera. Usar o nome e região explícitos;
   o grupo atual é `application-economy`. Não tentar remover novamente o
   antigo grupo `application`, já excluído em 13/11.
8. Conferir as stacks eksctl e a VPC. Se houver `DELETE_FAILED`, consultar os
   eventos e resolver a dependência indicada; não excluir interfaces
   gerenciadas ou recursos alheios por tentativa. RDS e EC2 externos às
   stacks eksctl podem impedir a exclusão da rede.
9. Remover a stack de identidade Jenkins após desvincular suas dependências.
   Revisar roles/addons/OIDC restantes contra as stacks antes de removê-los.
10. Auditar os itens retidos da tabela abaixo e registrar o estado final,
    incluindo os recursos deliberadamente mantidos e seu custo residual.

Referência de comandos destrutivos, **somente após confirmar alvo e autorizar**:

```powershell
aws cloudformation delete-stack --profile store-admin --region us-east-1 --stack-name microserv-store-rds
aws cloudformation wait stack-delete-complete --profile store-admin --region us-east-1 --stack-name microserv-store-rds
```

A exclusão da EC2 depende da decisão sobre seu disco. Depois de cumprir os
passos de dados/Jenkins/rede acima, a exclusão do cluster é:

```powershell
.\.tools\eksctl\eksctl.exe delete cluster --name microserv-store --region us-east-1 --profile store-admin --wait
```

O `--wait` aguarda a exclusão; não tratar uma solicitação aceita como término
confirmado. Se a observação interromper, consultar o estado da mesma operação
antes de repetir. [Guia eksctl](https://docs.aws.amazon.com/eks/latest/eksctl/creating-and-managing-clusters.html).

## Auditoria dos custos remanescentes

| Item | Conferência necessária |
|---|---|
| EKS / nodegroups / Auto Scaling | Cluster inexistente, grupos eliminados, workers terminados |
| EC2 Jenkins | Terminada se encerramento definitivo autorizado |
| RDS | Instância excluída; snapshot final e backups com retenção explícita |
| EBS | Identificar RabbitMQ retido, disco Jenkins e volumes disponíveis; decidir cada um |
| Snapshots EBS/RDS | Registrar dono, retenção e eventual exclusão individual |
| Rede | NAT eliminado, Elastic IP liberado, ALB/NLB ausentes, VPC sem dependências |
| CloudWatch | Identificar logs retidos e sua política; excluir apenas se autorizado |
| IAM / OIDC | Conferir o que foi gerenciado pelas stacks e as identidades exclusivas restantes |
| Faturamento | Verificar após atualização dos dados; não interpretar atraso como gasto zero |

Snapshots e backups retidos continuam sujeitos à cobrança até serem
excluídos. [Documentação RDS](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_DeleteSnapshot.html).

O resultado final precisa informar horário, conta, região, estados observados
e retenções acordadas. Não declarar encerramento financeiro completo apenas
porque `kubectl` deixou de conectar ou as três EC2 não aparecem como running.
Se a decisão for pausa, seguir o [plano de janela](budget-window.md); ele não
encerra a cobrança de EKS, NAT e armazenamento.
