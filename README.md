# Documentação Técnica - Formas de Iniciar uma AWS Step Function (Standard Workflows)

## Visão Geral
Esta documentação aborda métodos para iniciar máquinas de estados do tipo **Standard** no AWS Step Functions, focando em cenários onde desenvolvedores **não possuem acesso privilegiado** (Console/CLI). Inclui detalhes técnicos, exemplos em Java, diagramas e práticas recomendadas para ambientes de produção.

---

## Índice
1. [Formas de Iniciação](#formas-de-iniciar-uma-step-function)
2. [Execuções Assíncronas](#execuções-assíncronas)
3. [Monitoramento e Troubleshooting](#monitoramento-e-troubleshooting)
4. [Melhores Práticas](#melhores-práticas)
5. [Limites e Cotas](#limites-e-cotas)
6. [Exemplos Avançados](#exemplos-avançados)

---

## Formas de Iniciar uma Step Function

| Método               | Caso de Uso                  | Complexidade | Latência  |
|----------------------|------------------------------|--------------|-----------|
| SDK Java             | Automação programática       | Baixa        | Imediata  |
| SQS + Lambda         | Processamento em lote        | Média        | Variável  |
| EventBridge          | Eventos em tempo real        | Média        | Baixa     |
| EventBridge Scheduler| Agendamentos recorrentes     | Baixa        | Previsível|
| API Gateway + Lambda | Integração com APIs externas | Alta         | Alta      |

---

## Execuções Assíncronas

### 1. SDK Java - `startExecution`

... (mesmo conteúdo anterior já presente)

---

## Monitoramento e Troubleshooting

### 1. Consulta do Status
```java
DescribeExecutionRequest request = DescribeExecutionRequest.builder()
    .executionArn("arn:...").build();
DescribeExecutionResponse response = sfnClient.describeExecution(request);
System.out.println(response.status());
```

### 2. Histórico Detalhado
```java
GetExecutionHistoryRequest request = GetExecutionHistoryRequest.builder()
    .executionArn("arn:...").build();
GetExecutionHistoryResponse history = sfnClient.getExecutionHistory(request);
history.events().forEach(System.out::println);
```

### 3. CloudWatch Metrics
```json
{
  "Metrics": [
    {
      "Id": "errors",
      "MetricStat": {
        "Metric": {
          "Namespace": "AWS/States",
          "MetricName": "ExecutionsFailed",
          "Dimensions": [
            {
              "Name": "StateMachineArn",
              "Value": "arn:aws:states:..."
            }
          ]
        },
        "Period": 300,
        "Stat": "Sum"
      },
      "ReturnData": true
    }
  ],
  "Threshold": 1,
  "ComparisonOperator": "GreaterThanOrEqualToThreshold"
}
```

---

## Melhores Práticas

### Segurança
- **Princípio do Menor Privilégio**: restrinja as permissões a somente o necessário.
- **Criptografia**: use KMS para proteger dados sensíveis.
- **Validação de Inputs**: limite de 256 KB. Valide antes de enviar:
```java
boolean isValidInput(String json) {
    return json.length() <= 256 * 1024 && json.startsWith("{") && json.endsWith("}");
}
```

### Idempotência
```java
String name = gerarChaveIdempotente(input);
StartExecutionRequest request = StartExecutionRequest.builder()
    .name(name)
    .input(input)
    .build();
```

### Tratamento de Falhas
```java
try {
    sfnClient.startExecution(request);
} catch (ExecutionLimitExceededException e) {
    // Reenfileirar, alertar, aplicar backoff exponencial, etc.
}
```

---

## Limites e Cotas

| Recurso              | Limite Standard            | Ação Recomendada             |
|----------------------|----------------------------|------------------------------|
| Execuções Ativas     | 1.000.000 por conta        | Solicitar aumento            |
| Tamanho do Input     | 256 KB                     | Usar S3 para payloads grandes|
| Duração da Execução  | 1 ano                      | Dividir em subworkflows      |
| Taxa de Execução     | 2.000/s por conta          | Usar Express Workflows       |

---

## Exemplos Avançados

### 1. Integração com API Gateway + Lambda
```java
public class ApiHandler implements RequestHandler<APIGatewayProxyRequestEvent, APIGatewayProxyResponseEvent> {
    public APIGatewayProxyResponseEvent handleRequest(APIGatewayProxyRequestEvent event, Context context) {
        String input = event.getBody();
        String executionArn = startStepFunction(input);
        return new APIGatewayProxyResponseEvent()
            .withStatusCode(202)
            .withBody("{\"executionArn\":\"" + executionArn + "\"}");
    }
}
```

### 2. Processamento Paralelo com SQS
```java
ReceiveMessageRequest receiveRequest = ReceiveMessageRequest.builder()
    .queueUrl(queueUrl)
    .maxNumberOfMessages(10)
    .build();

while (true) {
    sqsClient.receiveMessage(receiveRequest).thenAccept(response -> {
        response.messages().forEach(msg -> processMessage(msg.body()));
    });
}
```

---

## Diagramas de Arquitetura (Representações Lógicas)

### Cenário Completo: Processamento de Pedidos
```
[Cliente] 
  → (API Gateway) 
  → [Lambda de Validação] 
  → [SQS] 
  → [Lambda Consumidora] 
  → [Step Function] 
    → [Lambda Processamento] 
    → [DynamoDB]
    → [SNS Notificações]
```

### Monitoramento Centralizado
```
[CloudWatch Metrics]
  → [Alarmes] 
  → [SNS → Email/Slack]

[CloudWatch Logs]
  → [Logs Insights]
  → [Dashboards]
```

