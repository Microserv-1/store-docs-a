# APIs

Todas as chamadas de negócio passam pelo Gateway. As rotas protegidas utilizam o cabeçalho `Authorization: Bearer <token>`.

## Contas e autenticação

| Método e rota | Acesso | Resultado |
|---|---|---|
| `POST /accounts` | Público | Conta criada: 201; e-mail duplicado: 409 |
| `POST /auth/login` | Público | Token JWT: 200; credencial inválida: 401 |
| `GET /accounts/me` | JWT | Dados da própria conta: 200 |
| `GET /auth/me` | JWT | Identificador da conta: 200 |
| `GET /health-check` | Público | Verificação do Gateway |
| `GET /ready` | Público | Prontidão do processo |

Cadastro:

```json
{"name": "Usuário de teste", "email": "usuario@example.com", "password": "SenhaTeste123!"}
```

O login recebe `email` e `password` e retorna `access_token`, `token_type` e `expires_in`. Account normaliza o e-mail e armazena apenas o hash Argon2id da senha. A senha aceita entre 8 e 128 caracteres.

## Produtos, pedidos e câmbio

| Método e rota | Resultado |
|---|---|
| `POST /products` | Produto criado: 201 |
| `GET /products` e `GET /products/{id}` | Consulta: 200 |
| `DELETE /products/{id}` | Exclusão: 204 |
| `POST /orders` | Pedido criado: 201; produto inexistente: 400 |
| `GET /orders` | Apenas os pedidos da conta autenticada |
| `GET /orders/{id}` | Detalhe: 200; pedido de outra conta: 404 |
| `GET /orders/{id}?currency=BRL` | Resposta convertida; moeda não suportada: 422 |
| `GET /exchanges/{from}/{to}` | Cotação com `sell`, `buy`, `date` e `id-account` |

Os pedidos preservam os valores em USD. A conversão usa a taxa `sell`, arredonda os totais dos itens para duas casas decimais e soma os valores convertidos. Exchange é implementado em Python e utiliza um provedor externo.

## Segurança e comunicação

O JWT usa HS256, emissor `store-auth`, audiência `store-api` e validade padrão de 1.800 segundos. O campo `sub` identifica a conta. Gateway e serviços validam o token; um `X-Account-ID` enviado pelo cliente não altera a identidade autenticada.

A verificação de senha ocorre em `POST /internal/credentials/verify`, protegida por chave interna e sem rota pública no Gateway. Destinos indisponíveis geram 503; timeout gera 504. O Gateway não repete automaticamente operações de escrita.

O uso de HS256 pressupõe confiança nos serviços que compartilham a chave. Revogação de tokens, refresh tokens e rate limiting não fazem parte desta versão.

## Verificação

O [aceite integrado](evidence/2026-11-22-resume/business-integration.json) cobre autenticação, isolamento entre contas, produtos, pedidos, conversão, cache e fila. A conferência dos contratos de Account, Auth e Gateway com os enunciados acadêmicos originais permanece pendente.
