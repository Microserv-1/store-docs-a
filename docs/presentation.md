# Apresentação e defesa

Roteiro proposto de 120 segundos, a gravar com os dois integrantes. EKS e o
aceite integrado foram validados em 13/11. O HPA também foi comprovado na AWS:
Account escalou de 2 para 5 pods prontos e retornou a 2, com 550 logins sem
erros. [Vídeo técnico, gráfico e resultados](eks-load.md).

| Tempo | Participação | Conteúdo |
|---|---|---|
| 0–15s | Ambos | Nome, dupla, objetivo e escopo do e-commerce |
| 15–35s | A | Arquitetura: Gateway Go, Auth Java, Account Python, APIs de negócio e persistência |
| 35–70s | B, com A | Demonstração real: cadastro/login, produto, pedido, conversão e isolamento entre contas |
| 70–90s | A | Métricas, diagnóstico de CPU em login, distribuição entre pods e HPA real |
| 90–105s | B | Cache e mensageria com evidências próprias |
| 105–120s | Ambos | Resultados medidos, custos, limitações e links |

Separar o vídeo técnico longo de carga/HPA da apresentação. O script de
gravação produz um replay rotulado de comandos reais, sem capturar o desktop.
O contexto e os timestamps aparecem no vídeo; gravação Kind antiga não
comprova EKS nem substitui participação humana no vídeo final.

## Conteúdo dos slides

| Slide | Conteúdo e visual | Evidência |
|---|---|---|
| 1 — Objetivo e autoria | Backend de e-commerce; nomes; seis serviços; divisão A/B | Links dos repositórios e contribuições individuais |
| 2 — Arquitetura implantada | Gateway Go; Auth Java; Account Python; Product/Order/Exchange; RDS, Redis, RabbitMQ; EKS e Jenkins EC2 | Diagrama da página inicial e snapshot de nuvem |
| 3 — Fluxo e segurança | Login → produto → pedido USD → BRL; outra conta recebe 404 | 40 verificações reais no EKS; não mostrar JWT ou senhas |
| 4 — Gargalos e carga | A: diagnóstico e balanceamento; B: cache e fila; inserir curva/escala após medição | Relatórios e vídeo técnico separados, ambiente e commits visíveis |
| 5 — Resultado, custo e limites | Jenkins e seis rollouts; custo fixo estimado US$ 0,4426/h + variáveis; acesso privado, Single-AZ/NAT único | Site, estimativa, limites e procedimento de encerramento |

## Texto para ensaio (aproximadamente 120 segundos)

**A, 0–15s:** “Somos Yuri e Gustavo. Construímos um backend de e-commerce com
seis microsserviços, integrados por um Gateway e implantados no Amazon EKS.”

**A, 15–35s:** “O Gateway usa Go, a autenticação usa Java e o Account usa
Python. Product e Order persistem com usuários separados no RDS, e Exchange
consulta um provedor externo. Jenkins executa as validações e a release;
Prometheus e Grafana acompanham a aplicação.”

**B, 35–70s:** “Aqui cadastramos e autenticamos uma conta, criamos um produto
e um pedido em USD. A consulta em BRL usa o câmbio externo sem alterar o valor
original. Uma segunda conta não pode acessar o pedido. O teste integrado
executou quarenta verificações reais na AWS.”

**A, 70–90s:** “No login, as métricas mostraram aumento de CPU em Account.
O HPA escalou de duas para cinco réplicas prontas e voltou a duas depois da
carga. Três pods atenderam verificações de credenciais. Foram 550 logins sem
erros; não medimos um ganho controlado antes e depois.” Mostrar o gráfico EKS.

**B, 90–105s:** “O cache Redis reduz consultas repetidas e invalida produtos
removidos. A fila desacopla a notificação de pedidos; o consumidor trata
reentregas sem duplicar a notificação.” Mostrar evidências e limites da versão.

**Ambos, 105–120s:** “Os seis serviços passaram no deploy e na integração.
Documentamos custos, responsabilidades e limites: acesso privado, banco
Single-AZ e NAT único. Código, testes e evidências estão nos nossos sites.”

## Revisão e entrega

- Confirmar nomes completos, grupo e edição; cronometrar com as duas pessoas.
- Gravar a voz/participação real de ambos; não apresentar narração automática
  como defesa ou participação de um integrante.
- Evitar telas com credenciais, Secrets, tokens ou dados pessoais.
- Publicar links da apresentação, vídeo final e vídeo técnico; testar acesso
  sem autenticação quando exigido. Inserir os links reais nos dois sites.
- Registrar data, integrante, commit, cenário e resultado da revisão cruzada.
- Cada integrante envia seu próprio site no Blackboard e guarda a confirmação.

## Ensaio individual

Cada integrante deve explicar cadastro, hash de senha, assinatura/validação de JWT, headers de identidade, propriedade de pedidos, USD/câmbio, cache/invalidação, fila/idempotência, Jenkins, tag por commit, rollback, Services, probes, HPA/requests, capacidade dos nós, RDS, custos e diferença entre meta da aplicação e SLA de fornecedor.

Revisão cruzada: a Pessoa B executa pelo menos um cenário de A e vice-versa. Registrar pessoa, commit, data, cenário e resultado quando acontecer; não antecipar essa evidência.
