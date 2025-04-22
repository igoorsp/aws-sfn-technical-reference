### **Cenários Autorizados para Uso do AWS Step Functions**

Esta página documenta, de maneira objetiva e detalhada, os cenários permitidos para o uso das máquinas de estado (*state machines*) AWS Step Functions dentro da arquitetura da empresa. Os cenários descritos aqui foram previamente validados tecnicamente pelo time de Arquitetura e homologados pelo time responsável por Segurança da Informação.

Dado o caráter **dinâmico e evolutivo** das necessidades técnicas e de negócio, é importante ressaltar que esta documentação será continuamente revisada e atualizada à medida que novos padrões sejam homologados pela organização.

**É obrigatório** que todos os serviços utilizados nos fluxos implementados com AWS Step Functions sejam previamente submetidos à homologação técnica interna, garantindo aderência às normas de segurança, desempenho e compliance corporativos antes da sua utilização produtiva.

Além disso, recomenda-se que as equipes técnicas consultem periodicamente esta documentação para garantir que suas implementações estejam alinhadas com os cenários atualmente autorizados. Em caso de dúvidas ou para submissão de novos cenários para homologação, as equipes devem procurar o time de Arquitetura e Segurança para orientação adicional.

**Complementos recomendados nesta página:**

- **Lista dos serviços AWS homologados pela empresa** (ex.: Lambda, DynamoDB, SQS, SNS).
- **Procedimento interno para homologação de novos serviços** (passo a passo simplificado).
- **Pontos de contato internos** para esclarecimento e abertura de novos cenários.
- **Exemplos práticos e curtos** para cada cenário autorizado, facilitando o entendimento.

---

### **Cenários Restritos para Uso do AWS Step Functions**

Este documento apresenta, de forma clara e objetiva, os cenários atualmente restritos ou proibidos para utilização do serviço AWS Step Functions na empresa. As restrições aqui listadas têm como objetivo garantir conformidade técnica, segurança e desempenho, evitando práticas arquiteturais inadequadas ou implementações que possam gerar riscos operacionais ou de segurança.

É importante destacar que esta documentação possui um caráter **dinâmico e evolutivo**, sendo regularmente revisada pela equipe técnica de Arquitetura e Segurança da Informação, conforme novos padrões e requisitos técnicos forem identificados. Dessa forma, a relação de cenários restritos pode sofrer alterações periódicas.

Caso haja necessidade de utilizar algum cenário atualmente classificado como restrito, as equipes deverão, obrigatoriamente, submeter esse cenário à revisão técnica e homologação pelo time de Arquitetura antes de qualquer implementação prática ou produtiva.

**Complementos sugeridos para esta página:**

- **Justificativas técnicas claras** para cada cenário restrito, detalhando riscos e implicações.
- **Alternativas homologadas** recomendadas para substituição dos cenários restritos.
- **Procedimento formal para solicitar revisão e eventual liberação de cenários restritos**.
- **Contato interno da equipe técnica responsável** pela revisão e homologação.
Abaixo estão as introduções completas, explicativas, formais e evolutivas para cada um dos tópicos restantes:

---

## 4. **Guardrails e Políticas de Governança**

Esta documentação descreve formalmente os **guardrails e as políticas de governança** obrigatórias para utilização do AWS Step Functions dentro da empresa. O objetivo desta seção é garantir consistência técnica, segurança operacional e conformidade com padrões corporativos estabelecidos pela área de Arquitetura e Segurança da Informação.

A definição desses guardrails é um processo evolutivo, estando sujeita a revisões periódicas conforme novas práticas recomendadas pela AWS ou padrões internos da organização forem incorporados. As equipes técnicas devem consultar regularmente este documento para garantir que suas implementações estejam rigorosamente alinhadas com os padrões vigentes.

**Complementos recomendados nesta página:**

- **Checklist claro** das práticas obrigatórias de implantação e governança.
- **Recomendações de versionamento** (ex.: uso de versions e aliases).
- **Exemplos práticos** para cada guardrail definido (ex.: configuração IaC).
- **Procedimentos para auditoria interna** sobre o cumprimento dessas políticas.

---

## 5. **Diretrizes de Segurança e Compliance**

Esta página documenta as diretrizes obrigatórias para garantir que as implementações utilizando AWS Step Functions estejam alinhadas às políticas internas de **Segurança da Informação** e compliance corporativo. As orientações aqui descritas são mandatórias, visando prevenir riscos técnicos, operacionais e regulatórios relacionados à segurança e proteção dos dados e recursos da organização.

Esta documentação possui um caráter evolutivo, sendo revisada continuamente para refletir novas exigências regulatórias, atualizações de segurança pela AWS, e melhorias sugeridas pela equipe de Segurança da Informação da empresa.

**Complementos sugeridos nesta página:**

- **Exemplos detalhados de políticas IAM** (menor privilégio).
- **Checklist de segurança obrigatória** (criptografia, logs, auditoria).
- **Procedimentos internos para auditoria e validação de segurança**.
- **Contato da equipe de segurança da informação** para dúvidas ou esclarecimentos adicionais.

---

## 6. **Monitoramento, Logging e Observabilidade**

Este documento descreve formalmente as práticas recomendadas e obrigatórias relacionadas ao **monitoramento, logging e observabilidade** das execuções do AWS Step Functions. Essas diretrizes visam garantir visibilidade operacional completa, identificação precoce de incidentes, e suporte efetivo às equipes técnicas e operacionais.

Devido à natureza dinâmica das necessidades operacionais e técnicas, as práticas descritas aqui estarão sujeitas a revisões contínuas, conforme novas ferramentas, métricas ou padrões sejam definidos internamente ou recomendados pela AWS.

**Complementos sugeridos nesta página:**

- **Exemplos de configurações** detalhadas de CloudWatch Logs e X-Ray.
- **Métricas obrigatórias e recomendadas** (CloudWatch Metrics).
- **Procedimentos recomendados para criação de dashboards operacionais**.
- **Checklist operacional** para garantia de observabilidade completa e eficaz.

---

## 7. **Práticas de Desenvolvimento e Testes Locais**

Esta página define formalmente as práticas técnicas obrigatórias e recomendadas para o **desenvolvimento e testes locais** relacionados ao uso do AWS Step Functions. Essas diretrizes objetivam garantir eficiência, qualidade técnica e consistência entre ambientes locais e ambientes superiores (homologação e produção).

Como as práticas técnicas de desenvolvimento evoluem continuamente, esta documentação será revisada periodicamente para incorporar novas ferramentas, técnicas ou recomendações internas e externas (AWS).

**Complementos sugeridos nesta página:**

- **Ferramentas recomendadas** (AWS SAM, LocalStack, Step Functions Local).
- **Procedimentos detalhados para configuração de ambientes locais**.
- **Exemplos práticos para testes unitários e integrados**.
- **Checklist técnico obrigatório** antes de promover implementações para ambientes superiores.

---

Essas introduções formalizadas e estruturadas facilitam o entendimento claro por parte das equipes técnicas, proporcionam governança eficiente, segurança operacional e evolução contínua do uso do AWS Step Functions dentro da organização.
