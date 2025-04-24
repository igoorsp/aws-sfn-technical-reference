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
|------------------|-----------------------------|
| API Gateway com Lambda Authorizer | [Segurança: API Gateway + Lambda Authorizer](#seguranca-api-gateway--lambda-authorizer) |
| Backend (Java) com AWS SDK em EKS Privado (IRSA) | [Segurança: Backend com IRSA (EKS)](#seguranca-backend-com-irsa-eks) |
| Callback Pattern com SQS e Task Token | [Segurança: Callback com Task Token](#seguranca-callback-com-task-token) |
| Lambda Privada iniciando Step Function | [Segurança: Lambda Privada](#seguranca-lambda-privada) |
| Padrões restritos e não autorizados | [Casos Restritos e Não Autorizados](#casos-restritos-e-nao-autorizados) |

---

## 🔐 Segurança: API Gateway + Lambda Authorizer

Este cenário considera o uso de **API Gateway com Lambda Authorizer** para controlar o acesso à execução de workflows do AWS Step Functions. A chamada `StartExecution` é feita diretamente via integração do API Gateway com o serviço `states.amazonaws.com`, após validação do autorizer.

### 📊 Diagrama do fluxo

```
Usuário → API Gateway (REST ou HTTP)
           ↓
    Lambda Authorizer (valida token)
           ↓
    API Gateway executa Step Function (StartExecution)
           ↓
         Step Functions
```

### 🔑 Importância da validação de claims

O Lambda Authorizer deve validar cuidadosamente o token JWT recebido (ex: Cognito, IAM, SSO), incluindo:
- `exp` (expiração)
- `aud` (audience correto)
- `scope` ou `roles` autorizados
- `sub`, `email`, ou outro identificador único do usuário

Essa validação garante que somente usuários autenticados e autorizados consigam iniciar execuções sensíveis. O resultado pode incluir escopos e permissões que são repassados para o workflow.

### 🎯 Mapeamento de permissões por escopo

Com base nos `scopes` ou `roles` validados, a política de autorização pode retornar diferentes permissões no `policyDocument`, por exemplo:
- Escopo `admin`: acesso total a execuções.
- Escopo `user`: execução limitada a workflows do próprio usuário.

Além disso, os dados podem ser passados via `context` para uso posterior:

**Exemplo de mapping template (VTL):**

```vtl
{
  "executionContext": {
    "userId": "$context.authorizer.userId",
    "roles": "$context.authorizer.scope"
  },
  "payload": $input.body
}
```

### 🛡️ IAM Policy do API Gateway

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "states:StartExecution",
      "Resource": "arn:aws:states:us-east-1:123456789012:stateMachine:MeuWorkflow"
    }
  ]
}
```

Essa role é assumida pelo API Gateway e precisa ser referenciada no método de integração.

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "states:StartExecution",
      "Resource": "arn:aws:states:us-east-1:123456789012:stateMachine:MeuWorkflow"
    }
  ]
}
```

**Exemplo de mapping template (VTL):**

```vtl
{
  "executionContext": {
    "userId": "$context.authorizer.userId",
    "roles": "$context.authorizer.scope"
  },
  "payload": $input.body
}
```

---

## 🔐 Segurança: Backend com IRSA (EKS)

Este cenário abrange aplicações backend executando em **EKS privado**, que utilizam **IRSA (IAM Roles for Service Accounts)** para iniciar execuções do AWS Step Functions via AWS SDK (ex: Java).

### 📊 Diagrama do fluxo

```
Aplicação no Pod (EKS)
     ↓ (IRSA assume IAM Role)
AWS SDK (StartExecution)
     ↓
Step Functions
```

### ⚙️ Como funciona o IRSA no EKS

IRSA permite que os pods assumam uma IAM Role associada via `ServiceAccount`, usando tokens assinados pelo OIDC provider do cluster. Isso evita o uso de secrets estáticos dentro dos containers e garante segregação de permissões por workload.

### 🔎 Auditoria com CloudTrail

- Todas as chamadas `StartExecution` feitas com IRSA são registradas no CloudTrail com o `userIdentity.sessionContext.sessionIssuer.arn` refletindo o ServiceAccount que assumiu a role.
- Recomendado aplicar tags no `input` da execução para facilitar rastreabilidade (`userId`, `source`, `tenantId`).

### 🛡️ Trust Policy da IAM Role

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Federated": "arn:aws:iam::<ACCOUNT_ID>:oidc-provider/<OIDC_PROVIDER_URL>"
      },
      "Action": "sts:AssumeRoleWithWebIdentity",
      "Condition": {
        "StringEquals": {
          "<OIDC_PROVIDER_URL>:sub": "system:serviceaccount:<NAMESPACE>:<SERVICE_ACCOUNT_NAME>"
        }
      }
    }
  ]
}
```

### 🛡️ Política de permissão mínima

```json
{
  "Effect": "Allow",
  "Action": "states:StartExecution",
  "Resource": "arn:aws:states:us-east-1:123456789012:stateMachine:WorkflowPrincipal"
}
```

### 🧩 Exemplo de ServiceAccount

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: stepfunction-invoker
  namespace: backend
  annotations:
    eks.amazonaws.com/role-arn: arn:aws:iam::<ACCOUNT_ID>:role/StepFunctionExecutionRole
```

**Trust Policy da IAM Role para IRSA:**

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Federated": "arn:aws:iam::<ACCOUNT_ID>:oidc-provider/<OIDC_PROVIDER_URL>"
      },
      "Action": "sts:AssumeRoleWithWebIdentity",
      "Condition": {
        "StringEquals": {
          "<OIDC_PROVIDER_URL>:sub": "system:serviceaccount:<NAMESPACE>:<SERVICE_ACCOUNT_NAME>"
        }
      }
    }
  ]
}
```

**Permissão Mínima:**

```json
{
  "Effect": "Allow",
  "Action": "states:StartExecution",
  "Resource": "arn:aws:states:us-east-1:123456789012:stateMachine:WorkflowPrincipal"
}
```

**Service Account YAML:**

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: stepfunction-invoker
  namespace: backend
  annotations:
    eks.amazonaws.com/role-arn: arn:aws:iam::<ACCOUNT_ID>:role/StepFunctionExecutionRole
```

---

## 🔐 Segurança: Callback com Task Token

Este padrão é utilizado quando a Step Function pausa aguardando uma resposta externa via `waitForTaskToken`, exigindo um retorno com `SendTaskSuccess` ou `SendTaskFailure`.

### 📊 Diagrama do fluxo

```
Step Function
   ↓ (envia taskToken via SQS ou outro canal)
Aplicação (EKS ou Lambda)
   ↓ (processa evento)
AWS SDK → SendTaskSuccess/Failure
   ↓
Step Function continua
```

### 🛡️ Considerações de segurança do token

O `taskToken` atua como um identificador exclusivo e temporário da execução — deve ser tratado como um **secreto efêmero**:
- **Nunca logar ou persistir em claro**.
- **Validade limitada**: configure `TimeoutSeconds` adequado.
- **Canal de transporte seguro**: preferencialmente SQS com SSE-KMS habilitado.

### 🛡️ IAM Role da aplicação consumidora

```json
{
  "Effect": "Allow",
  "Action": [
    "states:SendTaskSuccess",
    "states:SendTaskFailure"
  ],
  "Resource": "*"
}
```

> Dica: considere restringir o uso do `taskToken` validando metadados associados no `input`, como `executionId` ou `userId`.

### 🧩 Estratégias recomendadas

- Tokens devem ser lidos e descartados imediatamente após uso.
- Auditar chamadas `SendTask*` via CloudTrail.
- Se possível, ative X-Ray para rastrear a chamada de callback no fluxo da execução.

**IAM Role do consumidor:**

```json
{
  "Effect": "Allow",
  "Action": [
    "states:SendTaskSuccess",
    "states:SendTaskFailure"
  ],
  "Resource": "*"
}
```

**Dicas de segurança:**
- O token deve ser considerado um secreto temporário.
- Evitar logar o token.
- Aplicar TTL curto na fila SQS onde o token é publicado.

---

## 🔐 Segurança: Lambda Privada

Este modelo contempla a execução de uma função Lambda alocada em uma **VPC privada**, responsável por iniciar execuções de workflows no AWS Step Functions.

### 📊 Diagrama do fluxo

```
Lambda (em subnet privada)
     ↓ (chamada via AWS SDK)
Step Functions
```

### 📡 Comunicação com Step Functions

Como a Lambda não possui acesso direto à internet, é necessário criar um **VPC Endpoint Interface** para o serviço `states.amazonaws.com`, garantindo que a chamada seja roteada internamente pela rede da AWS.

### 🛡️ IAM Role da Lambda

```json
{
  "Effect": "Allow",
  "Action": "states:StartExecution",
  "Resource": "arn:aws:states:us-east-1:123456789012:stateMachine:WorkflowPrivado"
}
```

### 🌐 VPC Endpoint sugerido

- Tipo: Interface
- Service name: `com.amazonaws.us-east-1.states`
- Subnets: privadas (sem acesso à internet)
- Security Group: liberar apenas portas 443 para as Lambdas

### 🧩 Boas práticas adicionais

- Configurar timeout e retries no SDK para chamadas internas.
- Habilitar X-Ray e CloudWatch para rastreamento do `StartExecution`.
- Garantir segregação de roles por função e ambiente.
- Nunca usar `Resource: *` na role da Lambda.

**IAM Role da Lambda:**

```json
{
  "Effect": "Allow",
  "Action": "states:StartExecution",
  "Resource": "arn:aws:states:us-east-1:123456789012:stateMachine:WorkflowPrivado"
}
```

**VPC Endpoint sugerido:**

- Tipo: Interface
- Service name: `com.amazonaws.us-east-1.states`
- Subnets privadas + SG com porta 443 liberada para a Lambda

---

## 🚫 Casos Restritos e Não Autorizados

| Cenário | Motivo |
|---------|--------|
| HTTP Task direto para API externa | Exposição indevida e falta de autenticação |
| Lambda Proxy como ponte para API | Aumenta latência e acoplamento |
| Uso de `Resource: *` em policies | Risco de escalonamento de privilégio |
| Execução direta por clientes externos | Sem rastreabilidade ou controle |

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

### 📌 Observações Finais

- Toda exceção às diretrizes aqui descritas deve ser **formalmente registrada**, avaliada pela equipe de Segurança e, se aprovada, documentada como exceção temporária.
- O conteúdo desta página é revisado periodicamente e deve ser tratado como **referência oficial** para todos os projetos que utilizem AWS Step Functions.

