# Execução e deploy

## Docker Compose

Na raiz, com Docker e Python 3.10+:

```powershell
python scripts/setup-local.py
docker compose up --build -d
python tests/smoke.py
docker compose --profile monitoring up -d
python scripts/load.py --users 4 --duration 60 --scenario identity
```

Gateway em `localhost:8080`, Prometheus em `localhost:9090` e Grafana em `localhost:3000`. Só Gateway publica API. Senha Grafana no `.env`, usuário `admin`. Parar com `docker compose --profile monitoring down`; o banco é preservado no volume. Não apagar volumes durante avaliação.

## Kubernetes local

Construir as imagens com Compose. Pode usar Minikube ou Kind. Kind é uma alternativa local para validar os manifests; se a avaliação exigir Minikube nominalmente, repetir nesse ambiente.

```powershell
kind create cluster --config infra/k8s/kind.yaml
kind load docker-image microserv-store/account:local microserv-store/auth:local microserv-store/gateway:local --name microserv-store
./scripts/deploy-local.ps1 -Context kind-microserv-store
kubectl --context kind-microserv-store -n store port-forward service/gateway 8081:8000
```

Em outro terminal: `$env:BASE_URL='http://localhost:8081'; python tests/smoke.py`. Port-forward de um Service fixa um pod: para demonstrar balanceamento do Gateway, gerar carga **dentro do cluster** contra o Service, ou usar um LoadBalancer/Ingress. Chamadas do Gateway ao Service Account ainda podem alcançar várias réplicas.

HPA requer API de métricas (Metrics Server). Em Kind, o componente pode exigir `--kubelet-insecure-tls` apenas para o ambiente local. Não transportar essa exceção para EKS. Validar `kubectl top pods -n store` antes da carga. Requests de CPU: 100m; alvo: 60%; mínimo: 2, máximo: 5. Limites de memória Java são maiores que os de Go/Python.

Após a validação local, Auth usa request de CPU de 250m para refletir o custo da JVM; Account e Gateway continuam em 100m. A exceção TLS local está em `infra/k8s/metrics-server-kind-patch.json`. O vídeo pode ser reproduzido com `python scripts/record-hpa.py`; requer ffmpeg instalado e port-forward em 8082. `scripts/render-evidence.py` gera um replay claramente identificado a partir dos snapshots coletados.

A renderização também requer Pillow (`python -m pip install Pillow`) e fonte Consolas no Windows, ou DejaVu Sans Mono no Linux. Os scripts de operação não dependem dessas ferramentas de vídeo.

## Jenkins por serviço

Agente Linux com label `docker-kubectl`, Docker CLI/daemon e kubectl. Credencial `dockerhub`: username/password ou token. Credencial `store-kubeconfig`: arquivo com acesso limitado ao namespace `store`; no EKS o agente também precisa AWS CLI e identidade IAM autorizada.

O job de integração da raiz usa credencial `github-read` (username/token, somente leitura) para clonar submódulos privados pelo binding `gitUsernamePassword` do plugin Git. Configurar o Git tool `Default` no Jenkins. Os jobs dos serviços também precisam de credencial de checkout nos respectivos SCMs.

Cada Jenkinsfile executa teste, build, publicação com tag igual ao SHA completo e, se `DEPLOY=true`, renderiza manifesto, valida no servidor, aplica e aguarda rollout. Falha de teste interrompe os estágios seguintes. Informar `DOCKERHUB_NAMESPACE`. Segredos e namespace devem existir antes do primeiro pipeline.

Recuperação: identificar tag SHA anterior e executar `kubectl -n store set image deployment/<serviço> <serviço>=<namespace>/store-<serviço>:<sha-anterior>` e `kubectl -n store rollout status deployment/<serviço>`. Registrar motivo, imagem atual/anterior e resultado do smoke test. `rollout undo` também pode ajudar, mas a imagem explícita facilita auditoria.

## Preparação EKS

Pré-requisitos externos: AWS CLI, eksctl, kubectl, credenciais IAM, quotas, orçamento, acesso ao registry e revisão da topologia. `infra/aws/eksctl.yaml` propõe `us-east-1`, duas zonas, nós privados, NAT único e dois nós t3.medium. Revisar versão Kubernetes suportada e fixá-la antes de criar o cluster. HPA de pods não cria nós; o limite do nodegroup, sozinho, não instala Cluster Autoscaler.

1. Revisar custos e identidade: `aws sts get-caller-identity`.
2. Revisar configuração: `eksctl create cluster --config-file infra/aws/eksctl.yaml --dry-run`.
3. Criar cluster somente na janela aprovada: `eksctl create cluster --config-file infra/aws/eksctl.yaml`.
4. Obter VPC, duas sub-redes privadas e security group dos workers a partir dos outputs do cluster. Aplicar `infra/aws/rds.yaml` com esses parâmetros e senha hex aleatória de 32+ caracteres; fornecer senha por arquivo local protegido/gerenciador de segredos, nunca por argumento visível no histórico.
5. Aguardar RDS. Configurar `DATABASE_URL=postgresql+psycopg://store:<senha>@<endpoint>:5432/accounts?sslmode=require` em arquivo de ambiente ignorado pelo Git. RDS não é público; regra de entrada aceita apenas workers.
6. Criar namespace. Executar `python scripts/k8s-secrets.py --context <contexto-eks> --env-file <arquivo-local>`.
7. Publicar imagens pelo Jenkins, aplicar manifests com as imagens SHA. Não usar imagens `:local` no EKS.
8. Instalar Metrics Server compatível com EKS; validar métricas. Aplicar monitoramento. Integrar os três serviços de B.
9. Expor apenas Gateway com `infra/k8s/gateway-public.yaml`. O arquivo é uma base HTTP: **configurar certificado/TLS no balanceador antes de enviar credenciais por rede pública**. A configuração final depende de domínio, ACM e controlador de balanceador escolhidos.
10. Rodar aceite funcional, carga, HPA e teste de recuperação. Guardar evidências reais.

Não foi executado provisionamento AWS durante a preparação local. A validação AWS dos templates, o Jenkins instalado e o pipeline remoto precisam ser comprovados no ambiente de destino.

## Encerramento

Após a janela de avaliação, registrar evidências, remover Service LoadBalancer e recursos Kubernetes, excluir stack RDS conforme política de snapshot e excluir cluster com eksctl. Conferir EC2, NAT, EBS, balanceadores, snapshots, RDS e CloudWatch restantes. Snapshot de banco preservado também gera custo.

Referências: [HPA Kubernetes](https://kubernetes.io/docs/concepts/workloads/autoscaling/horizontal-pod-autoscale/), [Metrics Server](https://kubernetes-sigs.github.io/metrics-server/), [Jenkins Docker](https://www.jenkins.io/doc/book/pipeline/docker/).
