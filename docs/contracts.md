# Contratos e integração com a Pessoa B

Contratos **provisórios**: as linguagens foram confirmadas; os enunciados detalhados de Account, Auth e Gateway ainda precisam ser conferidos. Não afirmar conformidade acadêmica completa antes dessa revisão.

## APIs da Pessoa A

| Método e rota no Gateway | Acesso | Resposta |
|---|---|---|
| `POST /accounts` | Público | 201: `id`, `name`, `email`; 409 para email duplicado |
| `POST /auth/login` | Público | 200: `access_token`, `token_type=bearer`, `expires_in` |
| `GET /accounts/me` | Bearer JWT | 200: conta do token; 401 para token inválido |
| `GET /auth/me` | Bearer JWT | 200: `id-account` |
| `GET /health-check` | Público | Vida do Gateway |
| `GET /ready` | Público | Prontidão local do processo |

Cadastro recebe `{"name":"Yuri","email":"yuri@example.com","password":"senha-com-8-ou-mais"}`. Login recebe email e password. Email é normalizado para minúsculas. Password tem 8–128 caracteres; banco guarda somente hash Argon2id. Dados inválidos geram 422 em Account e 400 em Auth.

`POST /internal/credentials/verify` existe apenas em Account, exige `X-Internal-Auth-Key` e não possui rota no Gateway. Nunca enviar essa chave ao cliente.

## Identidade

JWT HS256, emissor `store-auth`, audiência `store-api`, `sub` igual ao UUID da conta, `iat`, `exp` e `jti`. Validade padrão: 1800 segundos. Enviar `Authorization: Bearer <token>`.

Gateway valida assinatura, algoritmo, emissor, audiência e expiração, descarta headers arbitrários e encaminha `Authorization` e `X-Account-ID` calculado a partir do token. Serviços B devem **validar o JWT por conta própria**, especialmente dentro do cluster. Não confiar em header fornecido diretamente pelo cliente.

HS256 simplifica a integração acadêmica, mas todo serviço com a chave consegue emitir tokens. Evolução: assinatura assimétrica com chave privada exclusiva de Auth. Ainda não há refresh token, revogação ou rate limiting.

## Rotas para B

| Prefixo canônico | Variável | Destino |
|---|---|---|
| `/products` | `PRODUCT_URL` | `http://product:8000` |
| `/orders` | `ORDER_URL` | `http://order:8000` |
| `/exchanges` | `EXCHANGE_URL` | `http://exchange:8000` |

Método, caminho e query string são preservados. Todas essas rotas exigem token. Destino indisponível: 503; timeout: 504. Sem repetição automática de escritas. Corpo limitado a 1 MiB. Paths com escapes percentuais são rejeitados.

Para Compose, B fornece serviços `product`, `order`, `exchange` na rede `backend`, porta 8000, com suas dependências. Usar `docker compose -f compose.yaml -f compose.business.yaml up -d`. Para Kubernetes, namespace `store` e Services ClusterIP com esses nomes. Manter dados de cada serviço isolados.

## Aceite conjunto (integrado com Pessoa B)

1. Duas contas autenticadas pelo Gateway — **Validado (`tests/smoke.py`)**.
2. Criar produto (201), ler (200), busca por nome (`?name=`), excluir (204 -> 404) e Cache Redis (`X-Cache: MISS/HIT/BYPASS`) — **Validado (`store-product`)**.
3. Criar pedido (201); produto inexistente (400) — **Validado (`store-order`)**.
4. Listar só pedidos da conta; detalhe de outra conta (404) — **Validado (`store-order`)**.
5. Persistir USD; converter resposta para BRL via Exchange Python real sem alterar armazenamento — **Validado (`store-order` + `store-exchange`)**.
6. Moeda não suportada (422); chamadas sem token rejeitadas (401) — **Validado**.
7. Mensageria assíncrona com RabbitMQ e consumidor idempotente por `order_id` — **Validado (`tests/messaging_bottleneck.py`)**.

Documentação detalhada dos contratos e evidências da Pessoa B: [store-docs-b](https://microserv-1.github.io/store-docs-b/).
