# Jenkins local e imagens públicas

## Release AWS executada em 13/11

O Jenkins dedicado na EC2 executou `store-eks-release` com `DEPLOY_EKS=true`
e `KUBE_CONTEXT=store-eks`. Checkout recursivo GitHub privado, dez testes de
preparação, preflight, dry-run e rollouts dos seis serviços passaram. A
identidade AWS é uma instance role com credenciais temporárias; permissões
de aplicação vêm do grupo Kubernetes `store-deployers`. Acesso de rede à API
privada usa TCP 443 do security group do Jenkins para o do control plane.

Ver [evidência da release](evidence/2026-11-13-eks/jenkins-release.json) e
[aceite real no EKS](evidence/2026-11-13-eks/business-integration.json).
Os registros locais abaixo permanecem como histórico e reprodução; não são
uma indicação de que o Jenkins AWS ainda esteja pendente.

Na instalação Amazon Linux, Docker/Compose precisam também do plugin Buildx.
O container de Jenkins precisa alcançar IMDSv2 (hop limit 2) para usar a role.
Um token GitHub que só alcança a raiz pública não basta: configurar owner
Microserv-1, repositórios privados, Contents Read-only e aprovação da
organização quando exigida. Não conceder escrita para resolver checkout.

Para acessar a UI, o operador abre um túnel Session Manager para a porta 8086;
não há publicação pública da interface. Kubeconfig do job usa autenticação
exec/AWS CLI da role EC2, sem o perfil SSO pessoal do Windows embutido.

**Estado atual:** o Jenkins local também foi configurado com SCM GitHub real, por `scripts/jenkins-github.py --from-gh`. Os jobs `github-store-*` usam os repositórios remotos e `store-eks-release` usa `Jenkinsfile.eks`. O procedimento para AWS e os resultados dessa preparação estão em [Preparação EKS](eks-preparation.md) e [Estado e evidências](status.md). Os mirrors descritos abaixo registram a primeira validação e são um modo alternativo de reprodução.

Validação em 12/11/2026, com Jenkins real em Docker. O ambiente usa um executor, label `docker-kubectl`, Docker CLI/Compose, Python 3 e kubectl. Inclui Git, Pipeline, credentials-binding, JUnit e Timestamper.

## Reproduzir o controlador

Na raiz do projeto:

```powershell
python scripts/setup-jenkins-local.py
docker compose --env-file .env.jenkins -f compose.jenkins.yaml up --build -d
python scripts/jenkins-local.py wait
python scripts/jenkins-local.py lint
```

A interface fica em `http://127.0.0.1:8085`, usuário `admin`; a senha aleatória está no arquivo local ignorado `.env.jenkins`. Não publicar esse arquivo. O volume preserva jobs e credenciais. O controlador local usa root e o socket Docker para executar os exercícios: acesso administrativo ao Jenkins equivale a controlar o Docker local. Para um Jenkins compartilhado, usar agentes dedicados e acesso restrito.

Para reproduzir o modo de snapshots, executar `python scripts/jenkins-local.py mirrors`. Para o modo atual com GitHub real, executar `python scripts/jenkins-github.py --from-gh` com a conta correta já autenticada no GitHub CLI; não combinar os dois modos, pois os redirects de mirrors substituem URLs remotas.

Os mirrors locais contêm os commits exatos do checkout, sem exportar token GitHub para o Jenkins. O checkout recursivo local não comprova a autenticação do futuro Jenkins remoto no GitHub. Nesse ambiente, substituir `github-read` por uma credencial com acesso aos submódulos privados e configurar SCM remoto e webhook.

Se reaproveitar este controlador para SCM remoto, remover também os redirects locais `url.*.insteadOf` da configuração Git global. Caso contrário, URLs do GitHub continuarão apontando aos snapshots locais. O comando `mirrors` é específico da validação local e não deve ser executado nesse Jenkins remoto.

O volume Jenkins é montado no caminho físico do daemon para os containers de teste enxergarem o workspace. O job raiz conecta temporariamente o controlador à rede `store-integration_backend`, executa testes por DNS Docker e remove sua rede e volumes temporários ao terminar. O Compose de desenvolvimento é separado.

## Registro e credenciais

Foram criados, com autorização de Yuri, seis repositórios **públicos** na conta Docker Hub `uyris`: [Account](https://hub.docker.com/r/uyris/store-account), [Auth](https://hub.docker.com/r/uyris/store-auth), [Gateway](https://hub.docker.com/r/uyris/store-gateway), [Product](https://hub.docker.com/r/uyris/store-product), [Order](https://hub.docker.com/r/uyris/store-order) e [Exchange](https://hub.docker.com/r/uyris/store-exchange).

`python scripts/dockerhub-project.py jenkins-credential` transfere a credencial autenticada no helper Docker Desktop para o armazenamento criptografado do Jenkins local. Não imprime nem grava o segredo em arquivo do projeto. Cada pipeline usa `dockerhub`, login por stdin e configuração Docker temporária removida no fim. As tags usam o SHA completo do serviço, relacionando código, imagem e deploy.

```powershell
python scripts/jenkins-local.py job --service account
python scripts/jenkins-local.py run --service account
python scripts/jenkins-local.py status --service account
```

Repetir para `auth`, `gateway`, `product`, `order`, `exchange`; usar `root` para integração. `DEPLOY=false` é o padrão: executa Test, Build e Publish sem modificar o cluster.

## Deploy local e preparação para EKS

Com o Kind `microserv-store` preparado, `python scripts/jenkins-kind-credential.py` configura a credencial de arquivo `store-kubeconfig` **exclusivamente para esse cluster local** e conecta o controlador à rede Docker Kind. Nenhuma credencial AWS é usada. `python scripts/jenkins-local.py run --service account --deploy` habilita explicitamente o estágio Deploy.

O estágio renderiza o manifesto com a imagem SHA publicada, executa dry-run no servidor, aplica e aguarda rollout. Para EKS, preparar namespace, dependências e Secrets, trocar a credencial pelo kubeconfig do destino com permissões adequadas e repetir o aceite. A validação local não comprova deploy AWS.

## Evidências

Os comandos `status` e `lint` gravam resultados e consoles em `artifacts/jenkins/`, ignorado pelo Git. O teste `negative` provoca uma asserção falsa em um job separado, sem mudar os serviços; Build, Publish e Deploy devem ser ignorados por falha anterior. Guardar capturas da interface e logs para a defesa.

`python scripts/jenkins-local.py summary` reúne número, resultado, commit, digest publicado e confirmação de rollout de cada job em `artifacts/jenkins/summary.json`, preservando também os consoles por número de build.

As versões e os resultados da rodada constam em [Estado e evidências](status.md). Os repositórios de B foram apenas lidos e testados: alterações próprias devem ser publicadas pelo integrante responsável.
