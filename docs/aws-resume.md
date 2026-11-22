# Retomada e aceite — 22/11/2026

Yuri autorizou colocar o ambiente completo em operação. Esta página registra
uma verificação pontual, não disponibilidade contínua ou nova rodada de carga.

| Componente | Estado verificado |
|---|---|
| EKS microserv-store, us-east-1 | Dois workers t3a.medium Ready; min=2, desired=2, max=2 |
| Jenkins EC2 | t3a.small, container healthy, login HTTP 200, sem modo quieto e fila vazia |
| RDS PostgreSQL | Available; já estava ligado antes da retomada dos workers |
| RabbitMQ | Uma réplica saudável com o PVC preservado |
| Seis APIs, Redis, Prometheus e Grafana | Pods saudáveis no mesmo cluster |
| Gateway | ClusterIP, sem endpoint público |
| Inventário de custo | BASELINE_MATCHES: três EC2 previstas e nenhum autoscaler de nós |

Conferência adicional após a retomada: os seis Deployments correspondem
exatamente às imagens fixadas por commit e digest em `infra/aws/images.lock.json`,
com duas réplicas prontas por serviço no instante da consulta. Os imageIDs
observados dos pods foram registrados; um digest de índice multiplataforma
pode diferir do digest da imagem da plataforma. Prometheus retornou os
**12 alvos ativos saudáveis**, sem erro de coleta.
[Inventário vivo de imagens, réplicas e coleta](evidence/2026-11-22-resume/runtime-images.json).
Essa conferência não executou chamadas de negócio nem alterou recursos.

O IP do operador mudou e o endpoint administrativo do EKS foi restrito ao
novo /32. A configuração local ignorada pelo Git foi alinhada; nenhum token,
senha ou endereço administrativo do operador foi publicado nesta evidência.

## Validação funcional

O smoke de Account/Auth/Gateway passou. O aceite dos seis serviços passou em
**40 verificações**, cobrindo cadastro/login, autorização, isolamento entre
contas, produtos, pedidos, cache Redis HIT/BYPASS/invalidação, câmbio externo,
conversão sem alterar USD e processamento/replay idempotente RabbitMQ.
USD 26,44 resultou em BRL 157,78 na cotação desta rodada, via open-er-api.

- [Estado e commits de cada serviço](evidence/2026-11-22-resume/state.json).
- [Aceite funcional](evidence/2026-11-22-resume/business-integration.json).
- [Inventário de configuração e limites do guard](evidence/2026-11-22-resume/cost-guard.json).
- [Auditoria do Jenkins preservado](evidence/2026-11-22-resume/jenkins-release-audit.json):
  build nº 4 `SUCCESS`, com deploy habilitado, commit de release e seis rollouts
  confirmados no console. O nº 5 foi validação sem deploy. Consulta somente
  de leitura realizada após a retomada; não houve execução de build novo.

O teste usou port-forward de loopback, encerrado ao final. Nenhum build,
pipeline, teste de carga ou novo balanceador foi iniciado. O HPA permanece
habilitado e pode ajustar pods; o teto do grupo de workers continua em dois.
As medições de carga de [14/11](final-validation.md) não foram repetidas aqui.

## Acesso do professor e custos

Falta escolher entre demonstração ao vivo, identidade AWS individual restrita
ou publicação HTTPS do Gateway. A retomada não criou URL pública nem concedeu
acesso ao professor. Consulte o [roteiro de avaliação](teacher-evaluation.md).

A referência arquivada é US$ 0,287/h ou US$ 6,89/dia, antes de variáveis.
Não foi consultada a fatura nem calculado saldo do teto original de US$ 20.
Não existe encerramento automático: combinar a janela de avaliação e autorizar
o desligamento ou descarte posterior. A exportação oficial do Calculator e as
[confirmações individuais da entrega](delivery.md) permanecem pendentes.
