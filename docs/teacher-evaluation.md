# Preparação do teste do professor

Yuri decidiu manter os recursos preservados e pausados enquanto aguarda a
avaliação. **Nenhuma retomada ou publicação pública está agendada.** Os testes
de integração, carga e HPA já registrados continuam disponíveis no site.

## O que precisa voltar

Para testar as APIs, iniciar RDS, restaurar dois workers t3a.medium e
RabbitMQ com uma réplica. Redis, Prometheus, Grafana e os seis serviços
voltam pelos Deployments existentes. Não é necessário um novo build para
executar as imagens já implantadas e versionadas.

**Jenkins não é uma dependência de execução das APIs.** Pode continuar
parado se o professor testar apenas a aplicação. Reativá-lo separadamente
caso seja necessário demonstrar o pipeline. Seu disco continua preservado.

| Janela | Estimativa fixa de referência |
|---|---:|
| APIs e observabilidade ligadas, Jenkins EC2 parado | US$ 0,268/h; cerca de US$ 6,44/dia |
| APIs e Jenkins ligados | US$ 0,287/h; cerca de US$ 6,89/dia |
| Pausa atual | Cerca de US$ 0,161/h; US$ 3,86/dia, referência conservadora |

Valores derivados das [tarifas arquivadas](evidence/2026-11-13-eks/right-sizing/cost-estimate.json),
antes de variáveis, sem medir a fatura ou descontar gastos anteriores.
O cenário sem Jenkins ligado subtrai apenas US$ 0,0188/h de EC2; o disco
Jenkins permanece no armazenamento. Restauração de workers envolve novos
processos de inicialização e downloads. Não prometer disponibilidade imediata.

## Como o professor terá acesso

Despausar não cria uma URL pública. O Service Gateway atual é interno, e
`kubectl port-forward` abre acesso na máquina do operador.

| Forma de avaliação | O que está preparado | O que falta decidir |
|---|---|---|
| Ao vivo com a dupla | Operador abre o túnel e executa/acompanha as chamadas no seu computador | Horário da demonstração e janela autorizada de retomada |
| Remota com acesso AWS individual | Pode usar identidade própria com permissões restritas e túnel | Identidade do avaliador, RBAC e acesso de rede; nada disso foi concedido |
| Remota por URL pública | Contratos e autenticação da API existem | Método HTTPS, duração do acesso e custo; endpoint público não foi criado |

Acesso remoto independente precisa ser combinado antes da avaliação.
O acesso administrativo da dupla não deve ser compartilhado como credencial
de teste. Uma eventual publicação pública exige plano e autorização próprios;
não incluir ALB/domínio ou túnel externo sem essa decisão.

## Sequência da janela autorizada

1. Confirmar horário, método de acesso e custo da janela. Autorizar a
   retomada e a validação funcional pequena; isso não autoriza uma nova
   bateria de carga nem aumento do número de máquinas.
2. Validar a conta 975049967635, região us-east-1 e contexto `store-eks`.
   Se o IP do operador mudou, ajustar o `/32` do endpoint EKS.
3. Seguir a [retomada documentada](aws-pause.md), mantendo Jenkins parado
   quando o objetivo for só testar a aplicação.
4. Confirmar RDS available, dois workers Ready, PVC RabbitMQ montado e broker
   saudável; seis APIs com duas réplicas disponíveis, sem Pending persistente.
   Redis e histórico Prometheus efêmero podem iniciar vazios.
5. Validar cadastro/login, produto, pedido USD/BRL e isolamento de contas.
   Usar dados de teste; apresentar a versão e o commit realmente executados.
6. Abrir o acesso combinado para o professor. Confirmar que ele consegue
   executar o cenário pelo método escolhido, não apenas que o túnel do
   operador funciona.
7. Ao fim, confirmar se o professor ainda precisa de acesso. Nova pausa ou
   encerramento segue a decisão expressa de Yuri e a retenção combinada.

O guard `scripts/aws-cost-guard.py` foi preparado para o perfil completo,
com três EC2 ligadas. Se Jenkins permanecer parado, ele indicará diferença
em relação a esse perfil: interpretar especificamente essa ausência esperada,
sem ignorar outras divergências de máquinas, limites ou autoscalers.

## Sem execução antecipada

Este roteiro não comprova uma retomada já realizada nem acesso remoto já
concedido. O RDS pode iniciar automaticamente em 21/11 às 12h08 BRT, conforme
o [estado consultado](aws-final-review.md); isso não inicia os workers nem
torna a API disponível. Não usar esse reinício como agendamento da avaliação.
