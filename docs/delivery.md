# Conferência de entrega e defesa

## Links e materiais

| Material | Local | Estado verificado |
|---|---|---|
| Site individual de A e documentação do projeto | https://microserv-1.github.io/store-docs-a/ | Público, HTTP 200 em 13/11 |
| Site individual de B | https://microserv-1.github.io/store-docs-b/ | Público, HTTP 200 em 13/11; atualização final é de B |
| Repositório integrador | https://github.com/Microserv-1/store-root | Público, com ponteiros dos submódulos publicados |
| Serviços individuais | Links na [página inicial](index.md) | Privados; verificar acesso do avaliador |
| Aceite AWS | [Bateria de 14/11](final-validation.md) | 40 verificações, negócio 1/4/8 e login/HPA nas máquinas menores |
| Pipeline AWS | [Registro de release](evidence/2026-11-13-eks/jenkins-release.json) | SUCCESS informado no console; anexar número/console final |
| Vídeo curto | [MP4 de 1min57s e quadros](project-video.md) | Publicado com narração sintética identificada; confirmar aceitação e participação exigida |
| Apresentação | [Conteúdo dos slides e roteiro de 120s](presentation.md) | Texto e quadros prontos; arquivo nativo/editável pendente |
| Vídeo técnico EKS/HPA | [Rodada final](final-validation.md) e [histórico de 13/11](eks-load.md) | Ciclo 2 → 5 → 2; replays identificados, separados do vídeo humano |
| Custos | [Tarifas e estimativa](operations.md) | Estimativa rastreável pronta; exportação do Calculator pendente |

Não entregar o vídeo antigo de Kind como comprovação do EKS, nem uma página
de slides vazia como apresentação. A tentativa de edição da apresentação
visual em 13/11 foi rejeitada pelo serviço antes de salvar qualquer conteúdo.

Nova conferência em 14/11: o serviço de apresentações voltou a rejeitar a
inspeção (`artifact_preparation_failed`, sem possibilidade de repetição
indicada), sem retornar o documento. Os slides visuais seguem pendentes.
Após combinar a retenção para avaliação, usar o [roteiro de encerramento
AWS](aws-shutdown.md). Sua preparação não autoriza nem comprova exclusões.

## Conferência independente por cada integrante

1. Abrir os dois sites e materiais exigidos em uma janela sem autenticação.
2. Conferir nomes acadêmicos completos, grupo, edição 2026.2 e data.
3. Conferir que todos os repositórios listados existem e que o avaliador tem
   o acesso necessário aos privados, sem compartilhar tokens ou senhas.
4. Reproduzir o clone com `git clone --recurse-submodules` usando uma identidade
   autorizada. O checkout recursivo GitHub do Jenkins já foi executado com
   sucesso, mas não prova acesso para outra identidade.
5. Conferir que os dois gargalos próprios têm problema, código, autoria,
   explicação, resultado e limite, ligados às versões efetivamente testadas.
6. Conferir apresentação e vídeo humano de 1–2 minutos; ambos participam.
7. Vincular os vídeos técnicos separados, mostrando cluster real e HPA.
8. Conferir referências ao uso de IA e registrar revisão humana efetiva.
9. Confirmar horário limite e rubrica aplicável no Blackboard.
10. Enviar o próprio link no Blackboard e guardar o recibo. Uma submissão não
    substitui a do colega. Não registrar o envio antes de executá-lo.

## Registro de revisão cruzada

Preencher após a execução humana, sem antecipar aprovação:

| Campo | Conteúdo a registrar |
|---|---|
| Revisor e responsável | Nome de quem executou e de quem implementou |
| Versões | Commit da raiz e dos serviços relevantes |
| Ambiente e data | EKS/Kind/Compose, data e horário |
| Cenário | Ação reproduzida, dados de teste e resultado esperado |
| Resultado | Resultado observado e link para evidência |
| Pendências | Falhas, limites e correções necessárias |

Para revisão de A sobre B, usar o cenário de produto, pedido, USD/BRL,
isolamento entre contas, cache e replay de notificação do aceite integrado.
Para B sobre A, usar cadastro, login, token inválido, identidade forjada e
bloqueio da rota interna. O sucesso de um script executado pelo agente não
substitui a revisão e a compreensão do integrante.

## Perguntas para ensaio individual

| Pergunta | Pontos que a resposta deve explicar |
|---|---|
| Como a senha é armazenada e verificada? | Hash Argon2id em Account; custo CPU/memória; Auth consulta rota interna protegida |
| Por que um header X-Account-ID forjado não troca a conta? | Gateway valida JWT e deriva identidade; serviços validam token; consulta usa proprietário |
| O que o JWT valida? | Assinatura HS256, algoritmo, emissor store-auth, audiência store-api e expiração |
| Qual é o limite de HS256 compartilhado? | Quem possui a chave pode emitir tokens; assinatura assimétrica seria uma evolução |
| Como o pedido preserva seu valor? | USD armazenado; conversão na resposta; taxa sell e arredondamento documentados |
| Como detectar produto obsoleto no cache? | TTL, invalidação na exclusão, comparação HIT/BYPASS e consulta posterior |
| O que acontece quando uma mensagem chega novamente? | Consumo após persistência, chave de idempotência e notificação única; limites das janelas de falha |
| O que exatamente o Jenkins executou? | Checkout de commits, verificações, renderização, preflight, dry-run, aplicação e rollouts |
| Por que o deploy EKS é feito pela raiz? | Overlays mantêm CA do RDS e configuração de nuvem; deploy bruto pode removê-los |
| O que determina a escala do HPA? | CPU sobre requests, Metrics API, janela de carga, limites de réplicas e capacidade dos nós |
| HPA cria máquinas EC2? | HPA escala pods; crescimento de nós depende de outros mecanismos; nodegroup possui teto próprio |
| Qual a diferença entre Service e port-forward? | Service encaminha conexões entre réplicas; o túnel fixa um pod do serviço encaminhado |
| A aplicação tolera qualquer falha? | Banco Single-AZ e NAT único são limites; PDB e réplicas não garantem disponibilidade total |
| Por que o custo continua mesmo sem requisições? | EKS, EC2, RDS, NAT e armazenamento possuem cobrança de manutenção |
| Qual a diferença entre SLA e o teste? | Compromisso do fornecedor, objetivo da aplicação e medição observada têm escopos distintos |

Conferir as respostas no código e nos enunciados, não apenas decorar a tabela.

## Pendências que exigem material externo

As imagens originais da rubrica e os contratos acadêmicos de Account/Auth/
Gateway não foram recuperados pelos links autenticados nesta revisão.
Confirmar pesos, versão do template, exigência nominal de Minikube, horário
de entrega e eventual exigência de URL pública da API. O roadmap transcrito
continua sendo a referência disponível; não alegar validação integral dos
anexos sem confrontá-los.
