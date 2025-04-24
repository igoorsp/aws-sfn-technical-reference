## 🛡️ Diretrizes de Segurança e Compliance

> Esta página documenta as diretrizes obrigatórias para garantir que as implementações utilizando **AWS Step Functions** estejam alinhadas às políticas internas de **Segurança da Informação**, conformidade regulatória e boas práticas da AWS.

As orientações aqui descritas são **mandatórias** para todos os ambientes (desenvolvimento, homologação e produção), visando prevenir riscos técnicos, operacionais e legais.

Esta documentação é **evolutiva**, sendo revisada continuamente para refletir:
- Novas exigências regulatórias;
- Atualizações de segurança da AWS;
- Lições aprendidas em operação;
- Avaliações técnicas da equipe de Segurança e Arquitetura.

---

### 📚 Índice de Referência por Caso de Uso

| Cenário Técnico | Página de Segurança Relacionada |
|-----------------|-----------------------------|
| API Gateway com Lambda Authorizer | [Segurança: API Gateway + Lambda Authorizer](#seguranca-api-gateway--lambda-authorizer) |
| Backend (Java) com AWS SDK em EKS Privado (IRSA) | [Segurança: Backend com IRSA (EKS)](#seguranca-backend-com-irsa-eks) |
| Callback Pattern com SQS e Task Token | [Segurança: Callback com Task Token](#seguranca-callback-com-task-token) |
| Lambda Privada iniciando Step Function | [Segurança: Lambda Privada](#seguranca-lambda-privada) |
| Padrões restritos e não autorizados | [Casos Restritos e Não Autorizados](#casos-restritos-e-nao-autorizados) |

---

## 🔐 Segurança: API Gateway + Lambda Authorizer

Este modelo envolve um fluxo onde o **usuário se autentica via API Gateway**, utilizando **Lambda Authorizer personalizado**, e a execução da **Step Function é iniciada diretamente** pela integração do API Gateway com o serviço `states.amazonaws.com`.

**Recomendações técnicas:**
- Uso obrigatório de Lambda Authorizer com validação de JWT/scopes.
- Role IAM do API Gateway com permissão apenas para `states:StartExecution` em ARN específica.
- Uso de mapping templates para repassar contexto do usuário (userId, tenant, etc.).
- Habilitação de logs no API Gateway e observabilidade via CloudWatch.

🔗 [Exemplo de política IAM e template de integração →](#)

---

## 🔐 Segurança: Backend com IRSA (EKS)

Cenário onde a execução do `StartExecution` ocorre via **AWS SDK Java** em uma aplicação hospedada em **EKS Privado**, utilizando **IRSA (IAM Roles for Service Accounts)**.

**Práticas obrigatórias:**
- Uma IAM Role exclusiva vinculada ao ServiceAccount com `states:StartExecution`.
- Trust policy com OIDC configurada para o cluster.
- Uso de VPC Endpoint Interface para Step Functions (sem NAT Gateway).
- Rastreabilidade via execução nomeada + CloudTrail.

🔗 [Exemplo completo de role + YAML de ServiceAccount →](#)

---

## 🔐 Segurança: Callback com Task Token

Utilizado quando a Step Function pausa usando `waitForTaskToken` e aguarda uma resposta assíncrona via `SendTaskSuccess` ou `SendTaskFailure`.

**Itens de segurança essenciais:**
- Tratamento do token como dado sensível.
- Autorização controlada na aplicação consumidora (Lambda ou EKS).
- VPC Endpoint Interface configurado se consumidor estiver em rede privada.
- TTL e visibilidade controlada para tokens em filas SQS.
- IAM Role com permissões mínimas e uso de tags/contexto no callback.

🔗 [Melhores práticas com SQS, segurança do token e IAM →](#)

---

## 🔐 Segurança: Lambda Privada

Cenário onde uma **Lambda em VPC privada** (sem internet) realiza a chamada `StartExecution`.

**Requisitos técnicos:**
- IAM Role com `states:StartExecution` restrita.
- Acesso ao Step Functions via **VPC Endpoint Interface**.
- Monitoramento e fallback configurado para falhas na execução.
- Observabilidade via X-Ray e CloudWatch.

🔗 [Exemplo Terraform de VPC Endpoint + Lambda Role →](#)

---

## 🚫 Casos Restritos e Não Autorizados

Esta seção detalha os cenários que estão **explicitamente proibidos** ou **ainda não homologados** pela área de Arquitetura ou Segurança.

**Exemplos restritos:**

| Cenário | Motivo |
|--------|--------|
| Uso direto de HTTP Task | Risco de exposição a endpoints não autenticados |
| Lambda Proxy com chamadas síncronas externas | Aumento de latência e risco de timeout |
| `StartExecution` com `"Resource": "*"` | Permissões excessivas não rastreáveis |
| Step Function iniciada diretamente por usuários finais (sem API intermediária) | Falta de validação, rastreabilidade e controle de identidade |

---

## ✅ Checklist de Segurança Geral

| Item | Obrigatório |
|------|-------------|
| IAM com menor privilégio (por cenário) | ✅ |
| Criptografia em repouso (KMS) e em trânsito (TLS 1.2+) | ✅ |
| CloudTrail habilitado para auditoria | ✅ |
| X-Ray e CloudWatch Logs para rastreabilidade | ✅ |
| Tokens de Task protegidos e descartáveis | ✅ |
| Roles segregadas por serviço e fluxo | ✅ |

---

## 👥 Contatos

| Área | Responsável | Contato |
|------|-------------|---------|
| Arquitetura Técnica | João Silva | joao.silva@empresa.com |
| Segurança da Informação | Maria Souza | maria.souza@empresa.com |

---

### 📎 Observações Finais

- Toda exceção às diretrizes aqui descritas deve ser **formalmente registrada**, avaliada pela equipe de Segurança e, se aprovada, documentada como exceção temporária.
- O conteúdo desta página é revisto periodicamente e deve ser tratado como **referência oficial** para todos os projetos que utilizem AWS Step Functions.

---

### 🛠️ Deseja importar?

Se quiser, posso gerar esse conteúdo em:
- `.docx` para edição offline
- Exportação JSON/HTML para Confluence API
- Estrutura em `.drawio` para o índice visual

Só me dizer o formato desejado. Deseja que converta para algum deles agora?
