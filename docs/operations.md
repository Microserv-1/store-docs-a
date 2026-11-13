# Custos, PaaS e SLA

## Implantação efetiva e estimativa de 13/11/2026

Topologia efetivamente consultada: EKS Standard em us-east-1; dois workers
t3.large; agente Jenkins t3.large; RDS PostgreSQL 16.15 db.t4g.small Single-AZ;
72 GiB EBS gp3 (40 workers + 30 Jenkins + 2 RabbitMQ), 20 GiB RDS gp3, NAT
único e um IPv4 público associado ao NAT. Acesso privado por port-forward;
nenhum ALB foi criado para a aplicação.

Tarifas On-Demand obtidas pela AWS Price List Query API em 13/11; SKU,
descrição e unidade disponíveis nas [tarifas arquivadas](evidence/2026-11-13-eks/public-price-rates.json).
Essa API de preços é [gratuita](https://aws.amazon.com/blogs/aws-cloud-financial-management/aws-price-list-gets-a-natural-language-upgrade-introducing-the-aws-pricing-mcp-server/).

| Recurso | Quantidade | Tarifa USD | Subtotal aproximado USD/h |
|---|---:|---:|---:|
| EKS Standard | 1 | 0,10/h | 0,1000 |
| EC2 t3.large Linux | 3 | 0,0832/h | 0,2496 |
| RDS db.t4g.small | 1 | 0,032/h | 0,0320 |
| NAT Gateway | 1 | 0,045/h | 0,0450 |
| IPv4 público | 1 | 0,005/h | 0,0050 |
| EBS gp3 | 72 GiB | 0,08/GB-mês | 0,0079 |
| RDS gp3 | 20 GiB | 0,115/GB-mês | 0,0032 |
| **Total fixo estimado** | | | **0,4426** |

Com a convenção de 730 horas para ratear armazenamento, isso resulta em
**US$ 10,62 por 24h, US$ 31,87 por 72h e US$ 323,13 por 730h**, sem variáveis.
Não são valores faturados ou uma garantia de custo máximo. O [cálculo JSON](evidence/2026-11-13-eks/cost-estimate.json)
declara exclusões e hipóteses. Acrescentar tráfego entre AZs/internet, NAT por
GB, CloudWatch, créditos excedentes de CPU T3/T4g, backups excedentes,
snapshots, impostos e câmbio. Não foram presumidos descontos ou créditos.

O orçamento adicional acordado é US$ 20: aproximadamente 45 horas dos custos
fixos, antes das variáveis. Portanto, manter tudo ligado por 72 horas excede
o teto. Combinar antecipadamente a janela de teste/avaliação e eventual
desligamento. Não há hard cap AWS nem monitoramento contínuo instalado.
Ver [regras de operação e pendências](closure.md).

O [plano de janela e pausa](budget-window.md) compara alternativas para esse
orçamento e declara o impacto de disponibilidade. Nenhuma pausa foi executada
ou autorizada implicitamente pela existência desse documento.

Ainda falta gerar/exportar a estimativa no **AWS Pricing Calculator**, com
hipóteses de tráfego/logs e link compartilhável. A tabela acima fornece uma
estimativa rastreável de tarifas públicas, mas não substitui essa evidência.
Os [parâmetros do Calculator](calculator-inputs.md) já estão preparados para
preenchimento e conferência, incluindo o Jenkins e a ausência de ALB.

## Estimativa de custos

O planejamento histórico abaixo antecede o provisionamento e não deve ser
interpretado como descrição mais atual que a seção anterior.

Configuração revista em 12/11/2026: us-east-1; EKS 1.35; dois t3.large, discos de 20 GiB; PostgreSQL RDS 16 db.t4g.small single-AZ com 20 GiB gp3 e três bancos/usuários isolados; RabbitMQ com EBS gp3 de 2 GiB; Redis reconstruível; NAT único; ALB HTTPS; logs de controle. É uma hipótese de infraestrutura, ainda não uma cotação. Incluir o host Jenkins quando dedicado. A revisão substitui as classes menores inicialmente propostas em 09/11.

O renderizador da [preparação EKS](eks-preparation.md) gera `artifacts/eks/cost-inputs.json`, com quantidades, região e janelas de 72 e 730 horas. O arquivo não inventa preços nem total: registrar as tarifas/consumo atuais e o link/exportação do Calculator antes do provisionamento. Há custo de IPv4 público e podem existir créditos CPU excedentes em instâncias T3, além de NAT, LCU, tráfego, logs e retenções.

**Atualização de 13/11:** o modo padrão port-forward não cria ALB e não exige domínio ou ACM. A quantidade de ALB na planilha gerada é zero nesse modo. O ALB/LCU da proposta anterior só se aplica a https-alb; EKS, EC2, RDS, NAT, volumes e outros recursos usados continuam sendo cobrados.

No [AWS Pricing Calculator](https://calculator.aws/), registrar EKS, EC2 e créditos CPU quando aplicáveis, EBS, RDS, backups/snapshots, Load Balancer, NAT por hora e processamento, tráfego e CloudWatch. Registrar data, região, volume de dados e link/exportação da estimativa. Separar janela acadêmica (horas efetivas) de operação mensal (730 horas como convenção de simulação). Não inventar preço ou valor total.

| Item | Janela acadêmica | Mensal | Evidência |
|---|---|---|---|
| EKS/EC2/EBS | Preencher horas | Simular 730h | Exportação Calculator pendente |
| RDS/backup | Preencher horas e GB | Simular 730h | Pendente |
| NAT/LB/tráfego/logs | Preencher uso | Simular carga mensal | Pendente |

NAT único e banco single-AZ reduzem custo, mas limitam tolerância a falhas. HPA aumenta pods dentro da capacidade disponível; custos de nós dependem da configuração e eventual autoscaling de nós. Remover snapshots remanescentes apenas após decidir retenção.

## Serviços gerenciados e PaaS

EKS gerencia o plano de controle Kubernetes; a equipe permanece responsável por workloads, imagens, IAM, rede, workers, updates e capacidade. RDS gerencia tarefas do banco, enquanto schema, dados, credenciais, acesso e política de backups continuam sob responsabilidade da equipe. Containers/Compose locais não equivalem a PaaS em execução. Documentar o enquadramento conceitual exigido pelo curso.

## Metas e medições

Objetivos iniciais de aplicação, sujeitos à baseline: 99% de disponibilidade na janela de demonstração, p95 inferior a 500 ms para leitura de identidade com até 8 usuários concorrentes, erros 5xx abaixo de 1% em carga estável. Login com Argon2id tem perfil separado e deve ser medido antes de definir meta de latência.

Definir disponibilidade como proporção de verificações bem-sucedidas sobre total no intervalo medido, com frequência de amostragem documentada; medir latência/vazão/erros pelo script e Prometheus. Esses valores são **objetivos, não resultados comprovados nem SLA contratual**. Teste curto não comprova disponibilidade mensal.

Consulta oficial em 09/11/2026: o [SLA EKS](https://aws.amazon.com/eks/sla/) prevê compromisso mensal de 99,95% para o endpoint Kubernetes no plano de controle Standard, com condições, exclusões e créditos. Isso não garante disponibilidade dos pods ou da aplicação. O [SLA RDS](https://aws.amazon.com/rds/sla/) diferencia Multi-AZ de Single-DB Instance; na topologia Single-DB proposta, a tabela de créditos começa abaixo de 99,5%. Não atribuir a ela o patamar Multi-AZ de 99,95%. Banco single-AZ e NAT único continuam limites conhecidos.

Como referência parcial verificada, o [preço EKS Standard](https://aws.amazon.com/eks/pricing/) é US$ 0,10 por cluster/hora: US$ 7,20 em uma hipótese de 72 horas, ou US$ 73,00 em 730 horas. **Isso é somente o plano de controle**, sem nós, banco, volumes, rede e logs; não constitui estimativa total nem exportação do Calculator.
