# Gargalos, observabilidade e balanceamento

## Tratamento A1: observabilidade

Problema: sem métricas e logs, aumento de latência pode ser confundido com falta de réplicas. Solução: métricas de requisições e processos em Python/Go; Actuator/Micrometer em Java; Prometheus e dashboard Grafana versionado. Account registra JSON sem payload, senha, token ou email. Gateway registra rota agregada e duração, evitando alta cardinalidade por ID.

Roteiro: executar cenário `identity`, observar p95, vazão, CPU e memória; executar cenário `login` sob condições equivalentes. Login inclui verificação Argon2id em Account, consumindo mais CPU/memória. Correlacionar comportamento observado com processos e pods; não interpretar a simples instalação de Grafana como resolução de gargalo. Limites: sem tracing distribuído e sem agregador de logs central; logs podem ser consultados com `kubectl logs`.

## Tratamento A2: balanceamento

Problema: uma réplica concentra carga e interrompe serviço ao cair. Solução: Deployment com duas réplicas, Service ClusterIP, probes, rolling update e PDB. HPA é mecanismo complementar de ajuste de capacidade e não uma segunda evidência independente para contar o mesmo gargalo duas vezes.

`X-Served-By` em Account identifica o pod que respondeu. Compare distribuição por instância durante carga, exclua **um pod local de teste**, observe recriação e repita chamadas. Não excluir banco nem múltiplos pods. PDB protege certas interrupções voluntárias, não garante disponibilidade em qualquer falha.

O balanceamento ocorre por conexões. Keep-alive pode produzir distribuição desigual e não representa erro do Service. Port-forward de um Service fixa um pod; gerar carga interna para provar distribuição de Gateway.

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
