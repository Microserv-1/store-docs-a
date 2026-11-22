# Execução

## Ambiente local

Pré-requisitos: Git, Docker Compose e Python 3.10 ou superior. Execute na raiz do projeto:

```powershell
git clone --recurse-submodules https://github.com/Microserv-1/store-root.git
cd store-root
python scripts/setup-local.py
docker compose -f compose.yaml -f compose.business.yaml up --build -d
python tests/smoke.py
python tests/business_integration.py
```

O Gateway fica disponível em `http://localhost:8080`. Os segredos são gerados em `.env`, ignorado pelo Git. Os bancos utilizam volumes persistentes.

Para habilitar a observabilidade local:

```powershell
docker compose -f compose.yaml -f compose.business.yaml --profile monitoring up -d
```

Prometheus: `http://localhost:9090`. Grafana: `http://localhost:3000`, usuário `admin` e senha definida em `.env`.

## Kubernetes local

Os manifests foram validados em Kind. O procedimento completo está em [integração local](integration.md). Com as imagens carregadas no cluster:

```powershell
./scripts/deploy-local.ps1 -Context kind-microserv-store -Business
kubectl --context kind-microserv-store -n store port-forward service/gateway 8081:8000
```

Em outro terminal, defina `BASE_URL` como `http://localhost:8081` e execute os testes. A equivalência de Kind para eventual exigência nominal de Minikube depende da avaliação acadêmica.

## Implantação AWS

| Componente | Configuração validada |
|---|---|
| Cluster | EKS `microserv-store`, Kubernetes 1.35, `us-east-1` |
| Workers | 2 × `t3a.medium`; limite de dois nós |
| CI/CD | Jenkins em EC2 `t3a.small` |
| Banco | RDS PostgreSQL `db.t4g.small`, Single-AZ, 20 GiB |
| Dependências | Redis e RabbitMQ no cluster; RabbitMQ com EBS persistente |
| Acesso | Gateway ClusterIP; serviços internos sem exposição pública |

Account, Product e Order utilizam bancos e usuários isolados no RDS. Credenciais ficam em Secrets, fora do controle de versão. Deployments usam recursos definidos, verificações de saúde e HPA.

O deploy EKS é realizado pelo `Jenkinsfile.eks` da raiz, preservando certificados do RDS e configurações de nuvem. O [procedimento de infraestrutura](eks-preparation.md) contém provisionamento, configuração e recuperação.

## Acesso para avaliação

Com identidade AWS autorizada e contexto Kubernetes configurado:

```powershell
kubectl --context store-eks -n store port-forward service/gateway 8080:8000
```

A API pode ser testada em `http://localhost:8080` por Postman, curl ou pelos scripts do projeto. O túnel atende o computador que o abriu e deve permanecer em execução durante o teste. Ainda não existe URL pública para acesso independente do avaliador.

## Operação

A [retomada de 22/11](aws-resume.md) foi validada com smoke e 40 verificações. A infraestrutura gera custos enquanto provisionada. Os procedimentos de [pausa](aws-pause.md), [encerramento](aws-shutdown.md) e [avaliação](teacher-evaluation.md) detalham a operação e a preservação dos dados.
