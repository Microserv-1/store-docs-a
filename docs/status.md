# Estado e evidências

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
