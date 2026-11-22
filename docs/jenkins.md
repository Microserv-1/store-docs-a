# CI/CD

Jenkins automatiza a validação dos serviços e a implantação no Kubernetes. As imagens são publicadas no Docker Hub em `uyris/store-<serviço>`, com tag correspondente ao SHA completo do código. O deploy EKS utiliza também digests fixados.

## Pipeline por serviço

Cada serviço possui um `Jenkinsfile` com as etapas:

1. Checkout do código.
2. Execução dos testes.
3. Build da imagem Docker.
4. Publicação no registry.
5. Deploy opcional e verificação do rollout.

Falhas de teste interrompem as etapas seguintes. O parâmetro `DEPLOY` controla a aplicação dos manifests; as credenciais do registry são fornecidas pelo Jenkins.

## Release EKS

O job `store-eks-release` executa o `Jenkinsfile.eks` do repositório principal:

1. Checkout recursivo dos submódulos.
2. Testes da preparação de infraestrutura.
3. Renderização e validação dos recursos.
4. Preflight AWS e dry-run no servidor.
5. Aplicação dos seis serviços e acompanhamento dos rollouts.

O deploy exige `DEPLOY_EKS=true` e contexto `store-eks`. O agente utiliza uma role EC2 com credenciais temporárias e permissões Kubernetes específicas. GitHub e Docker Hub utilizam credenciais armazenadas no Jenkins. A interface administrativa é acessada por túnel Session Manager.

## Resultados registrados

| Execução | Resultado |
|---|---|
| Pipelines dos seis serviços — 12/11 | Testes, build, publicação e deploy no Kind aprovados |
| Release EKS nº 4 — 13/11 | SUCCESS; deploy habilitado e seis rollouts concluídos |
| Release nº 5 — 13/11 | SUCCESS; somente validação, sem deploy |

O histórico e os marcadores do console foram conferidos diretamente no Jenkins em 22/11. [Evidência da release](evidence/2026-11-22-resume/jenkins-release-audit.json) e [pipelines por serviço](evidence/2026-11-12/jenkins.json).

## Reprodução local

```powershell
python scripts/setup-jenkins-local.py
docker compose --env-file .env.jenkins -f compose.jenkins.yaml up --build -d
python scripts/jenkins-local.py wait
python scripts/jenkins-local.py lint
```

A interface local utiliza `http://127.0.0.1:8085`. A senha fica no arquivo `.env.jenkins`, ignorado pelo Git. O volume preserva os jobs e as credenciais.

## Recuperação

Para restaurar uma versão, selecione a imagem anterior registrada por SHA/digest, atualize o Deployment e acompanhe o rollout. Em seguida, execute o smoke e o aceite integrado. No EKS, a recuperação deve preservar o overlay de configuração do RDS. O procedimento detalhado está em [preparação EKS](eks-preparation.md).
