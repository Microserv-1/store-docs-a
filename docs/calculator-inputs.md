# Parâmetros para o AWS Pricing Calculator

Esta tabela contém os recursos efetivamente identificados em 13/11, para
fechar a estimativa exigida pela entrega. Não provisiona nenhum recurso.
O controle do navegador estava indisponível nesta revisão; o link/exportação
do Calculator permanece pendente, sem substituir a etapa por uma planilha
apresentada como se fosse uma exportação oficial.

## Passo a passo para preencher

1. Abra [AWS Pricing Calculator](https://calculator.aws/#/) e clique em
   **Create estimate / Criar estimativa**. Nomeie como `Microserv Store —
   Yuri Henrique e Gustavo Lina — operação mensal`.
2. Em **Add service / Adicionar serviço**, procure cada serviço da tabela
   abaixo, clique em **Configure / Configurar**, preencha e use **Save and
   add service / Salvar e adicionar serviço**. Selecione sempre **US East
   (N. Virginia)**, `us-east-1`.
3. Use **730 horas/mês**, Linux, tenancy compartilhada e pagamento
   **On-Demand**. Não selecionar Savings Plans, Reserved Instances ou
   descontos não contratados. Esta é a simulação da aplicação **em operação**:
   a pausa atual não muda o projeto para zero workers nessa simulação.
4. Cadastre dois itens **Amazon EC2**: `Workers EKS`, com **2 t3a.medium** e
   gp3 de **20 GiB por instância**; `Jenkins`, com **1 t3a.small** e gp3 de
   **30 GiB**. Confira se o formulário de disco é por instância: o total dos
   workers é 40 GiB, não 80. Adicione **2 GiB gp3** para RabbitMQ como volume
   adicional, sem adicionar outra EC2 para o broker.
5. Adicione **Amazon EKS** (um cluster em suporte Standard), **Amazon RDS
   for PostgreSQL** (uma db.t4g.small Single-AZ, 20 GiB gp3), **Amazon VPC**
   (um NAT Gateway e um IPv4 público) e **Amazon CloudWatch** (logs). Os discos
   EC2 + RabbitMQ somam 72 GiB; os 20 GiB RDS são separados. Não adicionar
   Amazon MQ, ElastiCache, ALB, domínio ou outra EC2 para cada microsserviço.
6. Para os campos variáveis, registre uma hipótese explícita, não um consumo
   supostamente medido. Exemplo de simulação mensal acadêmica: **1 GB
   processado pelo NAT, 1 GB de saída internet, 1 GB de logs ingeridos e 1 GB
   médio de logs armazenados**. Declare tráfego entre AZs e créditos CPU como
   componentes adicionais a avaliar. Se não os quantificar, identifique a
   estimativa como parcial; não apresente custo zero como medição. Evite
   duplicar a mesma transferência em mais de uma seção do Calculator.
7. Em **My estimate / Minha estimativa**, confira o resumo. Os fixos da
   topologia de referência somam cerca de **US$ 209,54/mês**; variáveis podem
   elevar o total. Investigue diferenças, sem alterar tamanhos só para fazer
   o total coincidir. Registre data, região e hipóteses.
8. Em **Export / Exportar**, salve **PDF e CSV**. Guarde também o link
   compartilhável: o PDF exportado contém um link para a estimativa. Envie o
   link ou os arquivos para incorporarmos à documentação. A janela acadêmica
   de 72h tem referência fixa de **US$ 20,67**; mantenha a memória de rateio
   `72/730`, sem reduzir artificialmente o tamanho físico dos volumes.

As opções exatas podem variar conforme o idioma da interface. A criação de
estimativa não provisiona os serviços. Referências oficiais:
[criar/configurar](https://docs.aws.amazon.com/pricing-calculator/latest/userguide/create-configure-estimate.html)
e [exportar PDF/CSV](https://docs.aws.amazon.com/pricing-calculator/latest/userguide/export-estimate.html).

## Estimativa mensal de referência

Criar uma estimativa em https://calculator.aws/ com região **US East
(N. Virginia), us-east-1**, preços **On-Demand**, sem assumir créditos/free
tier, Savings Plans ou Reserved Instances. Usar 730 horas como convenção de
operação mensal e registrar a data da cotação.

| Grupo | Configuração |
|---|---|
| EKS | Um cluster Standard; Kubernetes 1.35 em suporte padrão; 730h |
| Workers EC2 | Dois t3a.medium, Linux, tenancy compartilhada, 730h cada; 20 GiB gp3 por worker |
| Jenkins EC2 | Um t3a.small, Linux, tenancy compartilhada, 730h; 30 GiB gp3 |
| RabbitMQ | Dois GiB EBS gp3 adicionais; broker executa nos workers, sem Amazon MQ |
| RDS | PostgreSQL db.t4g.small, Single-AZ, 730h; 20 GiB gp3; armazenamento criptografado |
| NAT | Um NAT Gateway, 730h; declarar separadamente hipótese de GB processados |
| IPv4 público | Um endereço associado ao NAT, 730h; evitar contá-lo duas vezes se já incluído em outra seção |
| CloudWatch | Logs de controle EKS habilitados; declarar hipótese de ingestão/armazenamento |
| Rede | Declarar hipóteses de tráfego entre AZs e saída internet; observar regras de franquia aplicáveis |
| Créditos CPU e backups | Declarar premissas T3/T4g e retenção de backups/snapshots; não presumir custo zero sem justificativa |
| ALB / ACM / domínio | Não utilizados na configuração atual por port-forward |
| ElastiCache / Amazon MQ | Não utilizados; cache e fila são pods no EKS |

Conferir que os discos não foram contados duas vezes: a soma EBS da aplicação
e Jenkins é **72 GiB**, além dos **20 GiB do RDS**. As seis APIs já estão nos
dois workers; não adicionar seis instâncias EC2 independentes.

Os componentes fixos de referência somam cerca de US$ 209,54 por 730h, antes
das variáveis. Esse valor serve para revisar o preenchimento, não para forçar
o Calculator a coincidir. Investigar divergências de região, suporte EKS,
tenancy, horas, armazenamento, impostos e itens incluídos.

## Janela acadêmica

Separar a simulação mensal da duração efetivamente planejada. Para o cenário
contínuo de 72h, a referência de custos fixos é US$ 20,67. Caso os campos do
serviço sejam mensais, preservar tamanhos reais dos recursos e anexar a
memória de rateio por 72/730; não fingir que o volume físico tem outro tamanho.

Para caber no orçamento adicional de US$ 20, usar a
[janela de operação que for aprovada](budget-window.md). Não registrar uma
pausa como executada antes da confirmação. Os volumes e plano de controle
podem continuar cobrando durante a pausa.

## Evidência a guardar

1. Nome da estimativa, região, data, integrantes e hipóteses de consumo.
2. Exportação disponível no Calculator e link compartilhável, quando houver.
3. Identificação de cenário mensal e janela acadêmica, com rateios explícitos.
4. Conciliação com as [tarifas públicas arquivadas](evidence/2026-11-13-eks/public-price-rates.json).
5. Atualizar `operations.md` e o checklist de fechamento com os links reais.

Não publicar credenciais, detalhes de pagamento ou dados privados de billing
na exportação. Referência: [AWS Pricing Calculator](https://docs.aws.amazon.com/pricing-calculator/latest/userguide/).
