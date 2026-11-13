# Parâmetros para o AWS Pricing Calculator

Esta tabela contém os recursos efetivamente identificados em 13/11, para
fechar a estimativa exigida pela entrega. Não provisiona nenhum recurso.
O controle do navegador estava indisponível nesta revisão; o link/exportação
do Calculator permanece pendente, sem substituir a etapa por uma planilha
apresentada como se fosse uma exportação oficial.

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
