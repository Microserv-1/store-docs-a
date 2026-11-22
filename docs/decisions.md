# Decisões e uso de IA

## Decisões técnicas

| Decisão | Justificativa | Limite |
|---|---|---|
| Account em FastAPI, Auth em Spring Boot e Gateway em Go | Integração entre linguagens e serviços independentes | Contratos acadêmicos originais ainda requerem conferência |
| PostgreSQL com bancos e usuários isolados | Persistência e separação entre serviços | Evoluções de schema exigem migrações versionadas |
| Argon2id | Hash adaptativo de senhas | Custo de CPU e memória no login |
| JWT HS256 | Interoperabilidade entre serviços | Serviços com a chave podem emitir tokens |
| Gateway sem repetição automática de escrita | Redução do risco de operações duplicadas | Cliente deve tratar indisponibilidade |
| Jenkins por serviço e release EKS na raiz | Automação e preservação da configuração RDS | Credenciais e permissões precisam ser administradas |
| Duas réplicas e HPA | Distribuição de tráfego e ajuste de capacidade | Dependência de métricas e recursos disponíveis |
| Kind na validação local | Reprodução com Docker | Aceitação de eventual exigência nominal de Minikube a confirmar |

## Uso de inteligência artificial

O Codex auxiliou na implementação de Account, Auth, Gateway, testes, Dockerfiles, manifests, pipelines, documentação e materiais de apresentação. Também executou verificações e organizou evidências de resultados locais e AWS.

As implementações de Product, Order e Exchange pertencem a Gustavo Lina. Os repositórios desses serviços foram analisados e testados durante a integração, preservando a autoria dos commits.

O vídeo curto foi produzido com apoio de IA a partir das evidências registradas e não contém narração. Os vídeos técnicos são replays identificados de comandos reais. Esses materiais não registram defesa oral ou revisão humana dos integrantes.

A revisão do código, a conferência com os enunciados e a explicação técnica permanecem sob responsabilidade dos autores. A revisão cruzada e a defesa devem ser registradas após sua realização.
