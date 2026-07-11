## Logs com OpenSearch e Fluent Bit

Foi configurado o OpenSearch em modo single-node, juntamente com o OpenSearch Dashboards para consulta e análise dos logs.

O Fluent Bit foi implantado como DaemonSet nos nós workers `node-02` e `node-03`, utilizando buffer em memória e coleta dos logs da aplicação Coffee Shop.

Os logs são enviados para o índice:

`kubernetes-logs-*`

A integração foi validada com sucesso por meio da mensagem:

`FLUENT_BIT_TEST_1783793287 coffee-shop observability`

O registro foi localizado no OpenSearch Dashboards, apresentando os campos:

- `collector: fluent-bit`
- `environment: observability-lab`
- `stream: stdout`
- `message`
- `@timestamp`

Também foi configurada uma política de retenção de 1 dia para os índices de logs, evitando crescimento excessivo do armazenamento.
