# Decisões e uso de IA

| Decisão | Justificativa | Limite |
|---|---|---|
| Account FastAPI; Auth Spring Boot; Gateway Go | Linguagens informadas pelo integrante | Enunciados de rotas ainda não disponíveis |
| PostgreSQL | Persistência independente dos pods, evolução para RDS | Schema inicial automático; evolução requer migrações versionadas |
| Argon2id | Senhas com hash adaptativo | Custo CPU/memória deve ser medido em login |
| JWT HS256 | Interoperabilidade e simplicidade inicial | Compartilhar chave amplia confiança entre serviços |
| Gateway sem retries de escrita | Evita duplicar criação por repetição automática | Cliente precisa tratar indisponibilidade |
| Jenkins por serviço e release EKS na raiz | Requisito mais específico do roadmap; overlays de RDS preservados | Deploy remoto validado em 13/11; vincular evidências do build final |
| Kind para QA local | Docker disponível; sem contexto Kubernetes inicial | Repetir em Minikube se exigido nominalmente |
| Duas réplicas e HPA | Balanceamento e capacidade elástica | Métricas e capacidade de nós são pré-condições |

## Uso de IA

Em 13/11, o operador executou a criação AWS, bootstrap e configuração do
Jenkins com instruções assistidas. O agente validou inventário, smoke e aceite
real de 40 verificações no EKS, preparou evidências e documentação. O operador
fixou teto adicional de US$ 20; carga e mudanças com custo precisam ser
apresentadas e autorizadas antes da execução. A consulta inicial tarifada ao
Cost Explorer está declarada no fechamento, sem atribuir à IA revisão humana.

Codex auxiliou na criação inicial de código, contratos provisórios, testes, Dockerfiles, manifests, pipelines e documentação. A ferramenta executou verificações locais e registrou resultados no estado do projeto. A implementação não deve ser atribuída ao colega nem apresentada como compreensão humana já validada.

O integrante deve revisar código e dependências, confrontar os enunciados, reproduzir os cenários, explicar JWT/Argon2id, autenticação interna, rede, HPA, limites e custos, e obter revisão cruzada. Não registrar revisão humana ou defesa como concluída sem ocorrência real.

Referências técnicas: [FastAPI JWT e hashes](https://fastapi.tiangolo.com/tutorial/security/oauth2-jwt/), [Go JWT](https://pkg.go.dev/github.com/golang-jwt/jwt/v5), [Spring Boot](https://docs.spring.io/spring-boot/), [credenciais Jenkins](https://www.jenkins.io/doc/book/using/using-credentials/).
