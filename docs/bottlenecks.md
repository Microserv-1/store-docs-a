# Gargalos, observabilidade e balanceamento

## Evidências reais no EKS em 13/11

A rodada autorizada de quatro usuários/120s produziu 550 logins HTTP 200,
p95 de 1.187 ms e escala HPA Account de 2 até 5 réplicas prontas, retornando a
2 após cessar a carga. Métricas de CPU e de verificações de credenciais por
pod correlacionam demanda e consumo e mostram três instâncias atendendo.
Não foi provado ganho antes/depois nem distribuição uniforme entre cinco
pods. [Relatório completo, gráfico e vídeo](eks-load.md).

Observabilidade é o tratamento A1; balanceamento é A2. HPA complementa a
evidência de capacidade, sem se tornar um terceiro gargalo individual.

## Tratamento A1: observabilidade

Problema: sem métricas e logs, aumento de latência pode ser confundido com falta de réplicas. Solução: métricas de requisições e processos em Python/Go; Actuator/Micrometer em Java; Prometheus e dashboard Grafana versionado. Account registra JSON sem payload, senha, token ou email. Gateway registra rota agregada e duração, evitando alta cardinalidade por ID.

Roteiro: executar cenário `identity`, observar p95, vazão, CPU e memória; executar cenário `login` sob condições equivalentes. Login inclui verificação Argon2id em Account, consumindo mais CPU/memória. Correlacionar comportamento observado com processos e pods; não interpretar a simples instalação de Grafana como resolução de gargalo. Limites: sem tracing distribuído e sem agregador de logs central; logs podem ser consultados com `kubectl logs`.

## Tratamento A2: balanceamento

Problema: uma réplica concentra carga e interrompe serviço ao cair. Solução: Deployment com duas réplicas, Service ClusterIP, probes, rolling update e PDB. HPA é mecanismo complementar de ajuste de capacidade e não uma segunda evidência independente para contar o mesmo gargalo duas vezes.

`X-Served-By` em Account identifica o pod que respondeu. Compare distribuição por instância durante carga, exclua **um pod local de teste**, observe recriação e repita chamadas. Não excluir banco nem múltiplos pods. PDB protege certas interrupções voluntárias, não garante disponibilidade em qualquer falha.

O balanceamento ocorre por conexões. Keep-alive pode produzir distribuição desigual e não representa erro do Service. Port-forward de um Service fixa um pod; gerar carga interna para provar distribuição de Gateway.

### Recuperação revalidada no Kind em 13/11

Um único pod Account foi removido no **Kind local** e substituído pelo
Deployment. As 30 consultas de acompanhamento retornaram HTTP 200 e
preservaram a identidade; o acesso após a substituição também passou.
O banco foi verificado como Service local, não RDS, e o Gateway foi acessado
por um túnel exclusivo do Kind. Nenhum recurso AWS foi alterado.

A imagem em execução usa o commit `7b61a3d265e0f8beaf11f9b0770ca9a6edc27cad`
e o digest `sha256:1c37183eb58d92d8c055769e54081a9a87c6c1278d1479e003aaf6bea201b3ac`,
o mesmo registrado na release EKS. [Relatório com imagem e versões](evidence/2026-11-13-local/pod-recovery.json).

Isso comprova o cenário observado de remoção de um pod com outra réplica
disponível. Não é teste de falha de máquina, zona ou de disponibilidade mensal,
nem uma execução de recuperação no EKS.

## Evidências disponíveis dos gargalos de B

As medições de cache/bypass de 12/11 registram diretamente os seis commits e
usam Product `c40bec1c96f786d11c89c021f7fb891af2bd048c`, Order
`a97fe5583e8e0e3615aab062a570bd2263b57089` e Exchange
`ea580c5b6e8c6eb6f552ac56390764a2f215c76f`, iguais aos integrados no EKS.
Não é necessário alterar código de B para aproveitar esses registros.

| Tratamento de B | Evidência disponível | Limite |
|---|---|---|
| Cache Redis | [Cache aquecido](evidence/2026-11-12-preparation/load-cache.json), [bypass](evidence/2026-11-12-preparation/load-bypass.json); aceite EKS confirma HIT/BYPASS e invalidação | Benchmark de duas contas/15s em Compose, com pausa por iteração; não comprova capacidade EKS nem ganho geral |
| Mensageria | [Indisponibilidade e retomada local](evidence/2026-11-12/resilience.json); [aceite EKS atual](evidence/2026-11-13-eks/business-integration.json) confirma processamento e replay idempotente | Teste funcional na AWS e teste de falha local têm escopos diferentes; não garantem exatamente uma vez em toda janela de falha |

B continua responsável por publicar sua documentação individual, explicar
os dois tratamentos e registrar sua revisão humana. Não atribuir os testes
executados pelo agente como revisão oral ou contribuição de implementação de B.

## Carga reproduzível

Na raiz:

```powershell
python scripts/load.py --users 1 --duration 60 --scenario identity --output artifacts/baseline.json
python scripts/load.py --users 8 --duration 120 --scenario identity --output artifacts/identity-8.json
python scripts/load.py --users 4 --duration 120 --scenario login --output artifacts/login-4.json
```

O modelo é fechado, sem think time. O script cria dados, autentica e registra duração real, p50/p95/p99, throughput, erros, respostas por instância e commits. Anotar mudanças locais não commitadas. Comparar com mesmo ambiente, duração e usuários. Não representa fluxo de pedidos; B fornece cenários de negócio completos.

Para HPA, confirmar `kubectl top pods`, registrar `kubectl get hpa -w` e `kubectl get pods -w`, gerar carga sustentada e guardar métricas antes/durante/depois. Acompanhar requests, duração e capacidade de agendamento. A evidência precisa mostrar réplica adicional pronta; ter o objeto HPA não comprova escalonamento.

## Evidências e vídeos

Guardar relatório JSON, ambiente, commits, imagens, réplicas, CPU/memória, screenshots e interpretação no relatório de estado. Separar vídeo técnico de cluster/carga/HPA do vídeo de apresentação (1–2 minutos). A participação oral e os dois tratamentos por integrante precisam de validação humana e revisão cruzada.
