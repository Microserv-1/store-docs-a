# Apresentação e defesa

Roteiro proposto de 120 segundos, a gravar com os dois integrantes após integrar B e validar EKS. Não narrar resultados de nuvem antes de executá-los.

| Tempo | Participação | Conteúdo |
|---|---|---|
| 0–15s | Ambos | Nome, dupla, objetivo e escopo do e-commerce |
| 15–35s | A | Arquitetura: Gateway Go, Auth Java, Account Python, APIs de negócio e persistência |
| 35–70s | B, com A | Demonstração real: cadastro/login, produto, pedido, conversão e isolamento entre contas |
| 70–90s | A | Métricas, diagnóstico de CPU em login, distribuição entre pods e HPA real |
| 90–105s | B | Cache e mensageria com evidências próprias |
| 105–120s | Ambos | Resultados medidos, custos, limitações e links |

Separar o vídeo técnico longo de carga/HPA da apresentação. `scripts/record-hpa.py` produz um vídeo de saída real de comandos e carga concorrente no cluster local, sem capturar o desktop. Isso não comprova execução EKS nem substitui a participação humana no vídeo final.

## Ensaio individual

Cada integrante deve explicar cadastro, hash de senha, assinatura/validação de JWT, headers de identidade, propriedade de pedidos, USD/câmbio, cache/invalidação, fila/idempotência, Jenkins, tag por commit, rollback, Services, probes, HPA/requests, capacidade dos nós, RDS, custos e diferença entre meta da aplicação e SLA de fornecedor.

Revisão cruzada: a Pessoa B executa pelo menos um cenário de A e vice-versa. Registrar pessoa, commit, data, cenário e resultado quando acontecer; não antecipar essa evidência.
