# Estado e evidências

## Redução de custos — 13/11/2026

Workers migrados para 2 × t3a.medium; Jenkins reduzido para t3a.small.
O smoke e as 40 verificações de negócio passaram novamente no EKS.
Jenkins build #5 passou na validação sem deploy. Ver [migração, custos e
limites](right-sizing.md). O vídeo de HPA anterior mostra as máquinas grandes;
não representa benchmark nas máquinas menores.

## Deploy real e aceite EKS — 13/11/2026

**Atualização mais recente:** deploy dos seis serviços pelo Jenkins EC2 passou,
com duas réplicas disponíveis por serviço. Smoke e 40 verificações de negócio
passaram no EKS, usando RDS, Redis, RabbitMQ e câmbio externo reais. Prometheus
tem 12 targets saudáveis. Acesso por port-forward, sem ALB nem URL pública.

O [fechamento](closure.md) reúne evidências sanitizadas, versões, pendências e
o limite adicional de US$ 20. A rodada HPA autorizada foi concluída: 550 logins
HTTP 200, zero erros, p95 de 1.187 ms e escala automática Account 2 → 4 → 5 → 2,
com cinco réplicas prontas no pico. Dois workers e uma EC2 Jenkins permaneceram
ligados, sem criação de novas máquinas. [Relatório, gráfico e vídeo EKS](eks-load.md).

A recuperação de um pod Account também foi revalidada no Kind, com o mesmo
digest publicado no EKS: substituição concluída, 30 consultas HTTP 200 e
identidade persistida. [Evidência local e limites](bottlenecks.md).

Os registros de preparação abaixo são históricos. Menções a AWS/Jenkins
remoto “não executados” descrevem o estado na data indicada, anterior ao deploy.

## Acesso sem domínio — 13/11/2026

Por não haver domínio próprio, a configuração de exemplo passou a usar `access_mode=port-forward`. Nesse modo a aplicação roda no EKS com Services ClusterIP, sem Ingress, ALB ou certificado ACM. Domínio/certificado/sub-redes públicas deixam de ser obrigatórios; as verificações correspondentes e as permissões IAM de leitura ACM/ALB ficam opcionais. Conta, região, CIDR, RDS, Secrets e os demais requisitos do cluster continuam necessários.

Os dez testes de preparação passaram, incluindo a ausência de publicação pública no modo privado e a preservação do modo HTTPS opcional. Os 45 recursos privados passaram no dry-run da API Kind; o template IAM passou no cfn-lint sem erros. [Preflight](evidence/2026-11-13/preflight.json) e [registro](evidence/2026-11-13/validation.json).

O Jenkins local estava desligado quando se tentou executar novamente o job; esta alteração foi validada diretamente pelos testes e pelo dry-run, sem alegar uma nova execução de pipeline. Os resultados de Jenkins de 12/11 abaixo permanecem históricos. Não houve provisionamento AWS nesta alteração.

O túnel fica limitado ao computador autorizado e fixa uma réplica do Gateway. Para acesso por outras pessoas através de URL pública ou para medir balanceamento entre Gateways, usar outro caminho de acesso/carga. O [procedimento atualizado](eks-preparation.md) explica o comando de port-forward e os passos opcionais de HTTPS.

## Preparação completa para EKS — 12/11/2026

A preparação pendente foi implementada e validada localmente. O [procedimento completo](eks-preparation.md) descreve geração de parâmetros, isolamento do RDS, dependências, HTTPS, IAM/RBAC, Jenkins com SCM real, preflight, aplicação, rollback, custos e encerramento. Nenhum recurso AWS pago foi criado.

| Verificação | Resultado e evidência |
|---|---|
| Configuração EKS 1.35 | Passou no schema oficial eksctl; [validação e versões das ferramentas](evidence/2026-11-12-preparation/validation.json) |
| Templates RDS/IAM | Sem erros no cfn-lint 1.60.1; W1011 mantido e documentado para o parâmetro NoEcho |
| Recursos renderizados | 46 recursos passaram no dry-run da API Kind; [preflight](evidence/2026-11-12-preparation/preflight.json) |
| Bancos e usuários | Bootstrap executado duas vezes no PostgreSQL 16 isolado; DDL/DML e seis bloqueios cruzados por execução; [evidência](evidence/2026-11-12-preparation/database-bootstrap.json) |
| Segredos | Credencial mestre separada das três URLs de aplicação com verify-full; [verificação](evidence/2026-11-12-preparation/secret-separation.json) |
| Permissões do CI | Atualização de aplicação permitida; Secrets, outro namespace, administração e exclusão de PVC negados; [RBAC](evidence/2026-11-12-preparation/rbac.json) |
| Configuração adicional da nuvem | Seis rollouts saudáveis e 40 testes passaram no Kind; [aceite](evidence/2026-11-12-preparation/overlay-acceptance.json) |
| Rollback | Estado anterior dos seis serviços restaurado; rollouts e smoke passaram; [registro](evidence/2026-11-12-preparation/rollback.json) |
| Fila e cache | Mensagem durável sobreviveu à substituição do pod RabbitMQ com o mesmo PVC; Redis voltou vazio e disponível; [teste isolado](evidence/2026-11-12-preparation/dependencies.json) |
| Jenkins GitHub real | `store-eks-release` nº 2 passou com raiz e submódulos acessíveis, sete testes de proteção e deploy AWS desabilitado; [resultado](evidence/2026-11-12-preparation/jenkins.json) |

Os testes de bancos usaram PostgreSQL local sem TLS; o teste de volume usou Kind/standard. A montagem das CAs e os manifests foram exercitados no Kind, mas o RDS/TLS real, EBS CSI, ALB/ACM, identidade IAM do agente, DNS e preços finais permanecem verificações do destino AWS. O Jenkins local autenticou no GitHub; um agente EC2 ainda não foi provisionado.

### Carga de negócio adicionada

Compose, duas contas concorrentes, 15 segundos por cenário e pausa de 100 ms por iteração. O relógio usado para latência é perf_counter, com resolução adequada no Windows. Há validação dos payloads, criação limitada de pedidos e rastreio dos seis commits e do hash do script.

| Cenário | Requisições | req/s | p95 | Erros |
|---|---:|---:|---:|---:|
| Produtos, cache aquecido | 480 | 31,71 | 19,92 ms | 0 |
| Produtos, bypass | 468 | 31,14 | 27,66 ms | 0 |
| Produtos, pedidos USD/BRL e câmbio | 870 | 57,87 | 32,94 ms | 0 |

As rodadas são curtas e sequenciais, sujeitas ao ruído do ambiente local; não estabelecem um ganho geral nem capacidade/SLA do EKS. Evidências: [cache](evidence/2026-11-12-preparation/load-cache.json), [bypass](evidence/2026-11-12-preparation/load-bypass.json) e [negócio completo](evidence/2026-11-12-preparation/load-business.json). Rodar níveis progressivos e registrar réplicas/métricas no destino antes de concluir sobre capacidade.

## Revalidação de 12/11/2026

A Pessoa B publicou as correções com sua própria autoria. O aceite Compose passou novamente nos 40 testes de negócio com Redis, RabbitMQ e AwesomeAPI reais, além do smoke de identidade. Os registros abaixo de 11/11 são históricos.

| Serviço | Commit revalidado |
|---|---|
| Account | `7b61a3d265e0f8beaf11f9b0770ca9a6edc27cad` |
| Auth | `d803717bcf09943028f2727cfa97a9515eb17e8d` |
| Gateway | `a3944171d6d008cd0c88027ab2a963558ee7d1da` |
| Product | `c40bec1c96f786d11c89c021f7fb891af2bd048c` |
| Order | `a97fe5583e8e0e3615aab062a570bd2263b57089` |
| Exchange | `ea580c5b6e8c6eb6f552ac56390764a2f215c76f` |

O Jenkins local executou o job raiz nº 1 com sucesso: clone recursivo de mirrors dos commits, build dos seis serviços, smoke e 40 verificações integradas. A cotação real dessa rodada resultou em BRL 149,08 para USD 26,44; valores de câmbio variam por horário.

Os sete Jenkinsfiles passaram no linter do Jenkins. O job negativo nº 1 falhou na asserção proposital; Build, Publish e Deploy foram ignorados por falha anterior. Os seis jobs de serviço nº 2 passaram em Test, Build, Publish e Deploy, incluindo dry-run no servidor, aplicação e confirmação de rollout no Kind. Product nº 1 foi abortado durante uma interrupção e retomado no nº 2. Os primeiros builds dos outros cinco serviços também passaram com `DEPLOY=false`.

O aceite pós-deploy no Kind passou no smoke e nos 40 testes, executados de um pod Account contra `http://gateway:8000`. O teste da raiz foi corrigido no commit `368f7ce`: os contadores de cache são por processo, e não se pode exigir HIT e BYPASS no contador da mesma réplica após balanceamento. As respostas continuam sendo verificadas individualmente pelos headers HIT e BYPASS. O mesmo teste corrigido passou novamente no Compose.

O job raiz nº 2 falhou antes do aceite porque o RabbitMQ temporário encerrou com código 1. A reprodução isolada iniciou normalmente; a causa dessa ocorrência não foi confirmada. O pipeline foi ajustado no commit `26c5a84` para preservar logs antes da limpeza e remover relatórios antigos antes de testar. Não usar o relatório antigo arquivado no build nº 2 como evidência daquele build. A nova execução é registrada nas evidências Jenkins desta rodada.

**Resultado final:** job raiz nº 3, commit `26c5a84d64a5aa50cddb37f34c11721da8b578f9`, terminou em SUCCESS com smoke, 40 testes e arquivamento do relatório e dos logs. O ambiente temporário foi removido após o teste. Os seis jobs de serviço nº 2 também terminaram em SUCCESS com rollout confirmado. O teste negativo nº 1 mantém FAILURE intencional.

Consultar [resumo Jenkins com commits e digests](evidence/2026-11-12/jenkins.json), [aceite arquivado pelo job nº 3](evidence/2026-11-12/jenkins-integration.json) e [histórico dos jobs de integração](evidence/2026-11-12/jenkins-root-history.json). O clone limpo local recuperou os sete submódulos e validou as duas configurações Compose; a validação do SCM privado em um Jenkins remoto continua pendente.

Os seis repositórios Docker Hub foram criados públicos em `uyris`. A documentação de [Jenkins e registro](jenkins.md) explica como reproduzir o ambiente e distingue o SCM local, o registry público e o destino de deploy. Nenhum deploy EKS foi executado nesta rodada.

As indisponibilidades controladas também passaram com os commits atuais de B: expiração real de chave Redis, consulta pelo banco sem Redis, criação de pedido com broker indisponível e processamento posterior com outbox vazia. São testes controlados; não comprovam todas as janelas de falha entre banco e confirmação da mensagem.

Evidências da rodada: [Compose](evidence/2026-11-12/compose.json), [Kind após deploy](evidence/2026-11-12/kubernetes.json), [resiliência](evidence/2026-11-12/resilience.json), [linter Jenkins](evidence/2026-11-12/jenkins-lint.json), [imagens públicas](evidence/2026-11-12/public-images.json) e [deployments](evidence/2026-11-12/deployments.json). Os testes de Order passaram nos sete casos publicados por B.

## Atualização de 11/11/2026

**Estado em 11/11 (histórico):** rollback de autoria em Product e Order concluído; aguardando a Pessoa B implementar/publicar as correções do [handoff](https://github.com/Microserv-1/store-root/blob/main/HANDOFF-PESSOA-B.md). Os aceites abaixo são históricos, anteriores ao rollback. Revalidar os novos SHAs antes de Jenkins.

Product, Order e Exchange de B foram incorporados como submódulos. O aceite de seis serviços, Redis/RabbitMQ reais e provedor externo passou em Compose. Detalhes, correções, reprodução e evidências no [relatório de integração](integration.md).

## Registro histórico da Pessoa A — 09/11/2026

Registro inicial em 09/11/2026. Atualizar com resultados reais, sem confundir preparação com implantação.

| Entrega | Estado |
|---|---|
| Account Python, Auth Java e Gateway Go | Código inicial e containers criados |
| Teste Account | Passou: cadastro, duplicidade, identidade e credencial interna |
| Testes Gateway | Passaram: headers forjados, assinatura, expiração, audiência e query |
| Teste Auth Java | Build Maven com teste passou |
| Fluxo integrado Docker | Passou: cadastro, login Java, JWT em Go/Python, duas contas, credenciais inválidas e bloqueio da rota interna |
| Persistência | Passou após reinício do container Account |
| Cluster Kubernetes local | Kind 1.32.2; três serviços, PostgreSQL, Prometheus e Grafana saudáveis no mesmo cluster |
| Observabilidade e balanceamento | Métricas coletadas; Account escalou de 2 para 5 pods; carga alcançou quatro instâncias; recuperação de um pod passou |
| Deploy EKS e RDS | Templates preparados; não executado |
| Jenkins remoto | Jenkinsfiles preparados; não executado |
| Product/Order/Exchange | Dependência da Pessoa B |
| GitHub Pages | Fontes/workflow preparados; não publicado |
| Custos e SLA de fornecedor | Topologia, premissas e SLA oficiais documentados; cotação completa no Calculator pendente |
| Revisão cruzada, vídeos e defesa | Pendentes |

## Medições locais

Ambiente: Windows com Docker Desktop (12 CPUs lógicas, cerca de 11,5 GiB de memória disponíveis ao daemon), Kind em um único nó, Kubernetes 1.32.2. PostgreSQL local. Chamadas via port-forward para um Gateway; isso fixa o pod de Gateway, mas seus acessos ao Service Account podem alcançar várias réplicas. Não extrapolar para EKS.

| Cenário | Usuários | Duração | Requisições | req/s | p95 | Erros |
|---|---:|---:|---:|---:|---:|---:|
| Identidade via Compose | 1 | 10s | 751 | 74,86 | 31ms | 0 |
| Identidade via Kubernetes | 2 | 15s | 632 | 42,04 | 109ms | 0 |
| Identidade/balanceamento Kubernetes | 16 | 15s | 1.440 | 95,40 | 297ms | 0 |
| Login Kubernetes | 4 | 90s | 588 | 6,51 | 1.125ms | 0 |

São cenários diferentes, não uma comparação controlada de ganho antes/depois. A rodada de leitura distribuiu 543, 830, 64 e 3 respostas entre quatro pods: a distribuição não é uniforme, conforme esperado com conexões persistentes e escala dinâmica. A rodada de login não informa instância Java (`unknown` no relatório); usar os dados de leitura para comprovar distribuição de Account.

Na recuperação, um único pod Account foi removido no cluster local e substituído. As 30 consultas de acompanhamento retornaram 200, preservando a identidade no banco. Não representa teste de falha de zona ou garantia de disponibilidade.

O diagnóstico inicial encontrou demora de aproximadamente 2s no cliente Windows ao usar `localhost`; os relatórios finais usam `127.0.0.1` e proxy desabilitado. JVM em inicialização consumiu CPU suficiente para provocar escala com request de 100m; Auth foi ajustado para 250m. Isso é ajuste de capacidade observado, sem alegação de ganho controlado de throughput.

## Arquivos verificáveis

- [Baseline Compose](evidence/2026-11-09/compose-baseline-final.json)
- [Baseline Kubernetes](evidence/2026-11-09/k8s-baseline-final.json)
- [Carga de login](evidence/2026-11-09/k8s-login-final.json)
- [Distribuição entre réplicas](evidence/2026-11-09/k8s-balance-final.json)
- [Recriação de pod](evidence/2026-11-09/pod-recovery.json)
- [Snapshots HPA e CPU/memória](evidence/2026-11-09/recorded-hpa.json)
- [Vídeo técnico local](evidence/2026-11-09/hpa-local.mp4): replay identificado de saídas reais de comandos coletadas durante carga; não é captura de desktop nem evidência EKS.
- [Estado do cluster](evidence/2026-11-09/cluster.txt), [recursos](evidence/2026-11-09/resources.txt), [eventos](evidence/2026-11-09/events.txt), [alvos Prometheus](evidence/2026-11-09/prometheus-targets.json) e [IDs das imagens](evidence/2026-11-09/images.json).

Os commits dos três serviços estão nos relatórios, com working trees limpas. Os serviços, a documentação e os ponteiros dos submódulos foram publicados no GitHub após autorização do integrante. O clone recursivo direto do GitHub foi validado com acesso aos repositórios privados. O vídeo final com os dois integrantes e o aceite dos serviços de B permanecem pendentes.

Clone limpo local validado com `python scripts/verify-local-clone.py`: os quatro submódulos recuperaram exatamente os commits registrados, Compose validou com novos segredos locais e MkDocs compilou em modo estrito a partir desse clone. Para esse teste, as URLs dos submódulos foram sobrescritas apenas na configuração do clone de teste para apontar às cópias locais, pois os commits ainda não foram enviados ao GitHub. Isso não é uma validação de clone remoto público.

## Entrega humana

Confirmar rubrica, enunciados e horário Blackboard. Identificar Pessoa B. Produzir apresentação e vídeo final de 1–2 minutos com os dois integrantes; manter vídeo de carga/HPA separado. Cada integrante publica seu próprio site e envia seu link ao Blackboard. Guardar confirmação. Esta documentação não constitui submissão.
