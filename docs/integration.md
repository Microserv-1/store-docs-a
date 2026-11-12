# Integração dos seis serviços — registro histórico de 11/11/2026

**Estado atual em 12/11:** B publicou as correções e o aceite integrado passou novamente. Consulte [Estado e evidências](status.md) para os novos SHAs e resultados do Jenkins. A descrição do rollback e os resultados a seguir preservam o registro de 11/11.

> **Atualização de autoria:** após a validação descrita abaixo, os commits de Yuri em Product e Order foram retirados do `main` a pedido do integrante. Os submódulos atuais apontam para os commits originais de B. A implementação corrigida deve ser refeita e publicada por B antes de nova validação. O relatório abaixo é histórico e não valida os branches restaurados. O Compose foi parado, sem apagar volumes. O roteiro completo está em [HANDOFF-PESSOA-B.md no repositório principal](https://github.com/Microserv-1/store-root/blob/main/HANDOFF-PESSOA-B.md). A execução real do Jenkins fica para depois dessa etapa.

As entregas de `gustavolafz` foram incorporadas ao `store-root` como submódulos de Product, Order e Exchange. Os commits originais e a autoria de B foram preservados. Os ajustes de integração desta rodada têm autoria Git de Yuri Henrique e assistência de IA registrada.

## Validação realizada

Aceite automatizado pelo Gateway com Account Python, Auth Java, Gateway Go, Product, Order e Exchange Python reais. PostgreSQL separado para Account, Product e Order; Redis real; RabbitMQ real; consulta de USD/BRL à AwesomeAPI, sem mocks no aceite.

O fluxo completo foi executado em Compose e no cluster Kubernetes local Kind 1.32.2, namespace `store`, com os seis serviços e dependências no mesmo cluster. Isso é evidência local; deploy EKS continua sendo uma etapa distinta.

As interrupções controladas de Redis e RabbitMQ ocorreram no Compose. No Kind foi repetido o aceite funcional de 40 verificações, com duas réplicas por serviço. O cluster é parado após a validação para poupar recursos locais; o Compose fica disponível em `127.0.0.1:8080`.

| Cenário | Resultado |
|---|---|
| Duas contas e tokens do Auth Java | Cadastro/login e tokens aceitos nos serviços Python/Go |
| Rotas protegidas | Sem token ou token adulterado: 401 |
| Products | POST 201, coleção e detalhe 200, DELETE 204 |
| Redis | HIT, BYPASS pelo Gateway, expiração de chave real e invalidação após DELETE |
| Order | Produto inexistente 400; pedido válido 201; total USD 26,44 para a fixture |
| Propriedade | Outra conta não lista pedidos e recebe 404 para detalhe e notificação; header forjado não concede acesso |
| Exchange externo | Cotação com sell, buy, date e id-account; moeda não suportada 422 |
| Conversão | Consulta em BRL usa sell e arredonda por item; USD persistido permanece igual |
| Mensageria real | Notificação processada, replay idempotente e uma única notificação por pedido |
| Redis indisponível | Product atende pelo banco, sem estourar o timeout do Gateway após ajuste de DNS |
| RabbitMQ indisponível | Pedido persiste com outbox pendente; evento é processado quando o broker volta |
| Controle global de consumo | Desabilitado por padrão, inclusive para usuário autenticado |

Os testes de unidade de B também passaram: Product (1), Exchange (1) e Order (3, incluindo regressões adicionadas). A matemática exata de conversão é testada isoladamente com provedor controlado; no aceite, a cotação vem do provedor real.

## Correções feitas na integração

- Gateway passou a encaminhar somente o header adicional `X-Cache-Control` para Product, mantendo bloqueio de headers internos e identidade forjada. A alternativa `?cache=bypass` de B continua disponível.
- A resolução DNS padrão de containers parados podia consumir o orçamento inteiro da requisição. Product e Order usam `timeout:1` e `attempts:1` no resolver local/Kubernetes, preservando fallback de cache e outbox dentro do timeout do Gateway. Essa escolha precisa ser observada novamente na rede EKS.
- RabbitMQ recebeu hostname estável e correção de permissão do cookie no volume local, preservando identidade do broker e dados em recriações. Não foram removidos volumes para resolver o problema.
- Publicação de eventos passou a exigir confirmação do broker. ACK do consumidor ocorre depois da transação de notificação; falha de processamento provoca NACK com requeue. Há teste de regressão de ordem do ACK e de requeue. O contrato é pelo menos uma vez, com idempotência.
- A pausa global do consumidor ficou desabilitada por padrão. Só ativar `ENABLE_DEMO_CONTROLS=true` em demonstração controlada; o teste de indisponibilidade real não depende dessa rota.
- Os manifests passaram a usar `PRODUCT_DATABASE_URL` e `ORDER_DATABASE_URL`, e conexões de cache/broker por Secret. Não reutilizar o banco/credencial de Account nos serviços de negócio.
- O job de integração Jenkins passou a construir e testar os seis serviços. A execução real desse job no Jenkins ainda precisa ser registrada.

## Reproduzir em Compose

```powershell
git submodule update --init --recursive
python scripts/setup-local.py
docker compose -f compose.yaml -f compose.business.yaml up --build -d
python tests/smoke.py
python tests/business_integration.py
python tests/business_resilience.py
```

O último comando interrompe e reinicia **somente Redis e RabbitMQ do projeto Compose local**. Execute sem tráfego de outras pessoas. O teste de expiração usa `EXPIRE 1` apenas na chave de sua fixture; não altera o TTL padrão de 120s do serviço. Os dados de contas/pedidos de teste permanecem para auditoria; produtos usados são removidos quando o teste termina normalmente.

## Reproduzir em Kubernetes local

Construir as imagens via Compose; carregar **as seis imagens de aplicação** no Kind. Redis, RabbitMQ e PostgreSQL podem ser obtidos diretamente pelo kubelet. Se uma importação de imagens externas multiarch falhar por digest ausente no Docker Desktop/Kind, não interpretar isso como falha da aplicação: permitir pull do registry no nó.

```powershell
kind load docker-image microserv-store/account:local microserv-store/auth:local microserv-store/gateway:local microserv-store/product:local microserv-store/order:local microserv-store/exchange:local --name microserv-store
./scripts/deploy-local.ps1 -Context kind-microserv-store -Business
kubectl --context kind-microserv-store -n store port-forward service/gateway 8082:8000
```

Em outro terminal, `$env:BASE_URL='http://127.0.0.1:8082'; python tests/business_integration.py`. Ao atualizar uma imagem `:local` já usada, carregar a imagem nova e reiniciar o Deployment correspondente. Aguardar os pods prontos depois de reiniciar o Docker/Kind.

## Evidências e limites

Relatórios em `evidence/2026-11-11/`: aceite Compose e Kubernetes, recuperação de dependências, estado do cluster e commits/imagens. Valores BRL variam com a cotação; os relatórios guardam a taxa efetivamente usada no teste.

- [Aceite Compose](evidence/2026-11-11/compose.json)
- [Aceite Kubernetes](evidence/2026-11-11/kubernetes.json)
- [Indisponibilidades e recuperação](evidence/2026-11-11/resilience.json)
- [Estado do cluster](evidence/2026-11-11/cluster.txt) e [CPU/memória](evidence/2026-11-11/resources.txt)
- [Commits e imagens locais](evidence/2026-11-11/revisions.json)

As interrupções controladas e a regressão de ACK não equivalem a um teste de kill do processo exatamente durante commit. O cache continua no modelo cache-aside: concorrência entre leitura e exclusão e invalidação perdida durante indisponibilidade precisam de endurecimento adicional se for exigida consistência forte. As consultas e exclusões sequenciais foram verificadas. Nenhuma dessas limitações deve ser apresentada como garantia de exatamente uma vez ou consistência forte.

Antes do EKS: provisionar bancos/usuários próprios de Product e Order; preencher URLs de banco, Redis e RabbitMQ em Secrets; publicar imagens SHA pelos Jenkinsfiles; configurar acesso externo/TLS, custos e executar novamente os cenários no ambiente AWS. Templates locais de PostgreSQL/Redis/RabbitMQ não são uma alegação de serviços gerenciados em produção. Revisão cruzada, defesa e vídeo da dupla continuam necessários.
