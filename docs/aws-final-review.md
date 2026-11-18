# Revisão de encerramento — 18/11/2026

Consulta somente de leitura, na conta 975049967635, us-east-1. **Nenhuma
exclusão ou novo snapshot foi executado.** O ambiente continua pausado;
o encerramento depende da liberação da avaliação e da decisão de retenção.

## Inventário conferido

| Recurso | Estado / identificador |
|---|---|
| EKS | microserv-store, ACTIVE, Kubernetes 1.35 |
| Grupo de workers | application-economy, min=0 / desired=0 / max=2 |
| Jenkins | i-03245ec6369003f79, t3a.small, stopped |
| RDS PostgreSQL | microserv-store-rds-database-xaw4nsp3iyky, stopped, 20 GiB gp3 |
| Reinício automático RDS | 21/11/2026 às 12h08 BRT, informado pela API |
| NAT Gateway | nat-02ddeffa30ddf5be8, available |
| IPv4 NAT | eipalloc-0b9462a8849430f2f |
| VPC do projeto | vpc-0fed1ba2f11b008ac |
| Disco Jenkins | vol-0287491fbe9e135f4, gp3 30 GiB, associado à EC2 parada |
| Disco RabbitMQ | vol-0f7ae6d5ad56ef8c0, gp3 2 GiB, available |
| Backups RDS | Dois snapshots automatizados existentes; nenhum manual listado |
| Snapshots dos dois EBS | Nenhum listado |
| Logs EKS | /aws/eks/microserv-store/cluster, 718.332.248 bytes, sem retenção definida |
| Balanceadores | Nenhum ALB/NLB ou Classic Load Balancer encontrado nessa VPC |

[Inventário estruturado](evidence/2026-11-18-review/inventory.json).
As regras de grupos de segurança foram reduzidas a referências e contagens,
sem publicar os endereços IP do operador. O snapshot não inclui Secrets,
conteúdo dos bancos ou logs, senhas ou credenciais Jenkins.

O EKS, o NAT e o armazenamento continuam cobrando durante a pausa. O valor
efetivamente faturado não foi consultado; não confundir projeções com saldo
disponível do teto adicional de US$ 20. Não houve chamada à API Cost Explorer.

## Plano concreto a aprovar

1. Confirmar que o professor não precisa mais do ambiente disponível e
   escolher uma das retenções abaixo. Código, sites e vídeos no GitHub ficam
   preservados em ambas.
2. Excluir a stack `microserv-store-rds` conforme a escolha de snapshot e
   backups. O template atual usa `DeletionPolicy: Snapshot`; o plano com
   retenção mantém esse snapshot final. Descarte total exige tratamento
   explícito dessa política e dos snapshots, evitando retenção por engano.
3. Encerrar a EC2 Jenkins. **O disco raiz está com DeleteOnTermination=true**:
   na opção de preservar dados, mudar esse atributo antes de encerrar a EC2.
   Na opção de descarte, conferir sua exclusão, sem tocar em outros discos.
4. Remover a regra de referência e o security group Jenkins exclusivos do
   projeto, após a saída da interface da EC2. RDS e Jenkins precisam deixar
   de depender da VPC antes de excluir sua stack.
5. Excluir `microserv-store` pelo eksctl, com espera, incluindo grupo de nós
   e stacks associadas do projeto. Conferir a remoção do NAT e liberação do
   endereço `eipalloc-0b9462a8849430f2f`, resolvendo dependências reais se houver.
6. Remover a stack de identidade Jenkins e conferir identidades exclusivas
   restantes. Revisar o volume RabbitMQ, os logs e backups conforme a escolha,
   sem assumir que a exclusão do cluster remove armazenamento Retain.
7. Verificar os estados finais dos recursos e registrar o que permaneceu.
   Uma operação aceita ou um timeout de observação não comprova exclusão.

As seis stacks encontradas estão listadas no inventário. O procedimento
detalhado e as referências oficiais estão no [guia de encerramento](aws-shutdown.md).

## Opções de retenção

| Escolha | O que permanece na AWS | Consequência |
|---|---|---|
| Encerrar preservando os dados | Discos Jenkins 30 GiB e RabbitMQ 2 GiB; snapshot final RDS privado | Remove custos EKS/NAT/instâncias; ainda cobra armazenamento e exige recursos novos para restaurar |
| Encerrar e descartar os dados | Nenhum banco/disco/backup/log exclusivo do projeto deve ficar retido | Perda dos dados e da configuração Jenkins na AWS; auditoria final necessária para confirmar ausência de itens cobrados |
| Manter a pausa | Recursos atuais, sem novas exclusões | Custos residuais continuam e o RDS pode iniciar automaticamente em 21/11 |

Para a primeira opção, o exemplo de retenção com **32 GiB gp3 e 20 GB de
backup RDS** custa aproximadamente **US$ 4,46 por mês**, antes de logs,
impostos, tráfego, outros snapshots e restauração. São tarifas públicas
consultadas em 18/11: EBS gp3 US$ 0,08/GB-mês; backup PostgreSQL
US$ 0,095/GB-mês. Backup RDS é cobrado por armazenamento medido: 20 GB é uma
hipótese de planejamento, não o tamanho faturável comprovado do snapshot.
[Tarifas com SKU e cálculo](evidence/2026-11-18-review/retention-estimate.json).

Esse cálculo é uma estimativa de armazenamento futuro, não custo incremental
total nem garantia de teto. Criar o snapshot final só ocorrerá se a opção
com retenção for aprovada. Logs podem ser eliminados após a evidência ter sido
preservada e a exclusão ter sido autorizada.
