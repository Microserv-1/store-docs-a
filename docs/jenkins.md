# Jenkins local e imagens públicas

Validação em 12/11/2026, com Jenkins real em Docker. O ambiente usa um executor, label `docker-kubectl`, Docker CLI/Compose, Python 3 e kubectl. Inclui Git, Pipeline, credentials-binding, JUnit e Timestamper.

## Reproduzir o controlador

Na raiz do projeto:

```powershell
python scripts/setup-jenkins-local.py
docker compose --env-file .env.jenkins -f compose.jenkins.yaml up --build -d
python scripts/jenkins-local.py wait
python scripts/jenkins-local.py lint
python scripts/jenkins-local.py mirrors
```

A interface fica em `http://127.0.0.1:8085`, usuário `admin`; a senha aleatória está no arquivo local ignorado `.env.jenkins`. Não publicar esse arquivo. O volume preserva jobs e credenciais. O controlador local usa root e o socket Docker para executar os exercícios: acesso administrativo ao Jenkins equivale a controlar o Docker local. Para um Jenkins compartilhado, usar agentes dedicados e acesso restrito.

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
