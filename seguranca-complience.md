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

