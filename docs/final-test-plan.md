# Bateria final nas máquinas menores

Plano preparado em 14/11. **Não executado nem autorizado automaticamente.**
Yuri decidiu manter as máquinas ligadas e retomar os testes com a dupla.
Este roteiro não altera o grupo de nós, não cria recursos e não faz teardown.

## Escopo e custo a confirmar antes da execução

Perfil: dois workers t3a.medium, Jenkins t3a.small, RDS db.t4g.small,
região us-east-1, contexto `store-eks`. Custos fixos estimados de US$ 0,287/h
continuam mesmo sem testes. Uma janela de 30 minutos equivale a cerca de
US$ 0,144 de manutenção; não é estimativa completa do custo incremental.
Tráfego, NAT, logs e créditos CPU podem acrescentar cobrança. O gasto já
acumulado precisa ser descontado do orçamento adicional de US$ 20.

Proposta de carga a autorizar: negócio com 1, 4 e 8 usuários, 60 segundos
por estágio, pausa de 500 ms por iteração e até duas tentativas de criação de
pedido por usuário. São até 26 POSTs de pedidos nos três estágios. Contas e
pedidos ficam como dados de teste; só os produtos de fixture são removidos.
Isso não é um modelo de taxa de chegada constante: é concorrência fechada.

Depois, propor login com quatro usuários por 120 segundos e até 300 segundos
de observação para HPA. Não aumentar nós, réplicas máximas, duração ou usuários
para forçar sucesso. Se a capacidade não for suficiente, registrar a falha e
avaliar com o operador antes de alterar a infraestrutura.

## Antes da carga

1. Confirmar autorização dessa janela e do impacto de custo; manter cada
   estágio separado e interromper a progressão em caso de erro.
2. Consultar o inventário com `scripts/aws-cost-guard.py`. Exigir
   `BASELINE_MATCHES`, três EC2 dos tipos esperados e nenhum autoscaler de nós.
   O script não consulta a fatura nem cria um bloqueio de gastos.
3. Conferir nós Ready, seis Deployments com duas réplicas, RabbitMQ/PVC
   saudáveis, métricas disponíveis e ausência de pods Pending.
4. Guardar commit da raiz, commits dos seis submódulos e digests implantados.
   Comparar com o checkout antes de atribuir resultados ao código local.
5. Abrir port-forward em um terminal separado, usando a identidade de operador:

```powershell
$env:AWS_PROFILE = 'store-admin'
kubectl --context store-eks -n store port-forward service/gateway 18080:8000 --address 127.0.0.1
```

O endereço local aponta para AWS. O túnel fixa um Gateway; não comprova
balanceamento entre Gateways. Ao testar distribuição, usar métricas por pod
dos serviços internos e declarar essa limitação.

## Execução após autorização

Criar uma pasta nova por rodada, sem sobrescrever evidências anteriores.
Executar smoke e aceite antes da carga, com `BASE_URL` apontando para o túnel:

```powershell
$env:BASE_URL = 'http://127.0.0.1:18080'
.\.venv-ops\Scripts\python.exe tests/smoke.py
.\.venv-ops\Scripts\python.exe tests/business_integration.py
```

Para cada estágio, substituir `N` por 1, depois 4, depois 8. O exemplo é
referência: **não executar os três se um estágio falhar**.

```text
python scripts/load-business.py --context store-eks --base-url http://127.0.0.1:18080 --scenario business --users N --duration 60 --pause-ms 500 --max-orders-per-user 2 --output artifacts/eks/RODADA/business-N.json
```

Guardar por rota p50/p95/p99, vazão, status, erros e os snapshots de
CPU/memória/HPA. Não combinar percentis dos estágios por média. Os limites de
escrita contam tentativas, inclusive timeout: resposta perdida pode ter
ocorrido depois da persistência. Não repetir automaticamente uma rodada que
falhou, pois isso gera novas fixtures e carga.

Para o HPA, após autorização própria e com ferramentas de vídeo locais:

```text
python scripts/record-hpa.py --context store-eks --base-url http://127.0.0.1:18080 --users 4 --duration 120 --cooldown 300 --output-directory artifacts/eks/RODADA/hpa --allow-eks-load
```

O recorder verifica o inventário periodicamente. Isso não mede créditos CPU,
tráfego nem fatura. Verificar que o processo terminou antes de repetir qualquer
comando; uma observação demorada não significa que a execução parou.

## Critérios de fechamento

- Aceite funcional aprovado; erros ou falhas de payload na carga analisados,
  com ambiente, duração e versões registrados.
- HPA com métricas válidas, aumento de réplicas realmente prontas e retorno
  à baseline. Se não escalar, registrar o resultado e investigar sem alegar
  sucesso apenas pela existência do objeto HPA.
- Nenhum nó adicional, Pending persistente ou surpresa de inventário.
- Comparação com a rodada antiga identifica diferença de hardware; não é
  experimento controlado para atribuir melhora a uma mudança isolada.
- Relatório e vídeo técnico vinculados; vídeo humano de 1–2 minutos, revisão
  cruzada, Calculator, identificação acadêmica e submissões continuam sendo
  entregáveis separados. Ver [conferência final](delivery.md).
- Combinar retenção para avaliação e desligamento com Yuri. Excluir recursos
  exige autorização específica; escalar workers para zero não encerra EKS/NAT.
