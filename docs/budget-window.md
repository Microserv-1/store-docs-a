# Janela de uso e orçamento adicional

## Decisão necessária antes de alterar a AWS

O teto informado por Yuri em 13/11 é **US$ 20 adicionais**. Não há autorização
para executar carga, criar recursos ou alterar a infraestrutura sem explicar
o impacto antes. Nenhuma pausa descrita aqui foi executada.

Com a topologia atual, o custo fixo estimado é US$ 0,4426/h, mais variáveis.
Manter todos os recursos ligados durante 72h custa cerca de US$ 31,87 antes
de tráfego, logs e créditos de CPU; portanto, não cabe no teto de US$ 20.

## Cenários para decidir a janela de avaliação

| Cenário de planejamento | Estimativa fixa | Efeito |
|---|---:|---|
| Tudo ligado por 72h | US$ 31,87 | Aplicação e Jenkins disponíveis continuamente |
| Só Jenkins parado durante 72h | US$ 25,88 | Aplicação permanece acessível, mas ainda excede US$ 20 |
| 8h completas e 64h com workers zerados, Jenkins e RDS parados | US$ 13,85 | Aplicação indisponível durante a pausa; exige retomada e validação |

O terceiro cenário é uma **simulação**, não uma agenda aprovada. O cálculo
mantém EKS, NAT, IPv4 e todo o armazenamento no período pausado de forma
conservadora. O custo fixo estimado dessa pausa é US$ 0,1610/h. Não presume
desconto, crédito, exclusão do cluster ou ausência de custos variáveis.
Alterações, transições de estado, downloads na retomada e cobranças de hora
parcial podem mudar o total. Não é garantia de teto na fatura.

## Recursos identificados

| Recurso | Identificador / configuração |
|---|---|
| Região | us-east-1 |
| Cluster | microserv-store |
| Nodegroup | application; configuração observada min=2, desired=2, max=4 |
| EC2 Jenkins | i-03245ec6369003f79 |
| RDS | microserv-store-rds-database-xaw4nsp3iyky |
| Persistência RabbitMQ | PVC data-rabbitmq-0, StorageClass store-gp3 com Retain |
| Dados de Account/Product/Order | Bancos separados no RDS |

## Procedimento proposto, sujeito à autorização

1. Combinar a janela exigida pelo professor. Guardar evidências, commits,
   digests e configurações locais; não exportar Secrets para a documentação.
2. Terminar os testes autorizados, confirmar que nenhum pipeline está em
   execução e registrar o estado saudável antes de pausar.
3. Se a pausa for autorizada, zerar o nodegroup por sua configuração de escala,
   sem excluir cluster, namespaces, PVCs ou bancos. As máquinas worker podem
   ser terminadas pelo grupo gerenciado; não tratar essa ação como simples
   suspensão dos processos. A aplicação ficará indisponível.
4. Parar temporariamente o RDS e depois a EC2 Jenkins, mantendo os volumes.
   Não criar snapshot extra ou excluir recursos sem decisão explícita.
5. Confirmar os estados finais e a ausência de reposição inesperada de nós.
   Os custos de plano de controle EKS, NAT, IP e armazenamento continuam.
6. Antes da avaliação, autorizar a retomada: iniciar RDS e Jenkins, restaurar
   duas máquinas worker e verificar nós, PVC, pods, métricas e acesso. Guardar
   margem para downloads e inicialização, sem prometer retomada instantânea.
7. Repetir o smoke e verificar o fluxo antes da demonstração, conforme a
   autorização de testes. Redis inicia vazio; não usar cache frio como se
   fosse a mesma condição de um benchmark aquecido.

O RDS mantém dados e endpoint quando parado, continua cobrando armazenamento
e backups aplicáveis e só pode ficar parado por até sete dias consecutivos;
a AWS o inicia novamente após esse limite. A retomada pode demorar.
[Documentação RDS](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_StopInstance.html).

O EKS não deixa de cobrar o plano de controle apenas porque não há workers.
Para encerramento definitivo, usar o procedimento de teardown após a
avaliação e conferir recursos retidos. Isso é uma decisão diferente da pausa.
[Documentação EKS](https://docs.aws.amazon.com/eks/latest/userguide/delete-cluster.html).

## Estimativa do Calculator ainda pendente

Os parâmetros completos para preenchimento estão no
[guia do Calculator](calculator-inputs.md). Não confundir a estimativa
reproduzível de tarifas públicas com uma exportação emitida pelo Calculator.

O controle `scripts/aws-cost-guard.py` é uma checagem de inventário pontual,
sem Cost Explorer, alterações ou carga. Ele não mede a fatura nem instala
monitoramento contínuo. A verificação de uma topologia pausada precisa ser
interpretada separadamente, pois o baseline normal exige três EC2 ligadas.
