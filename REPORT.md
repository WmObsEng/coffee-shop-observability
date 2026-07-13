# Relatório Técnico - Coffee Shop Observability

## 1. Visão geral

O objetivo deste projeto foi criar um ambiente de observabilidade para a aplicação Coffee Shop utilizando três servidores Linux em um cluster k3s.

A solução integra as seguintes tecnologias:

- Ansible;
- k3s e Kubernetes;
- GitHub Actions;
- Prometheus;
- Grafana;
- Zabbix;
- OpenSearch;
- OpenSearch Dashboards;
- Fluent Bit.

O ambiente permite acompanhar métricas da aplicação, estado do cluster, disponibilidade dos servidores, utilização de recursos e logs.

## 2. Infraestrutura e automação

O cluster foi criado com três nós:

- `node-01`: control plane do k3s e OpenSearch;
- `node-02`: worker e componentes como Zabbix e OpenSearch Dashboards;
- `node-03`: worker.

A preparação dos servidores e a instalação do k3s foram realizadas com Ansible.

Também foram criadas automações para:

- instalação do Zabbix Agent 2;
- configuração dos agentes;
- ajuste do parâmetro `vm.max_map_count`;
- preparação dos servidores para o OpenSearch.

Os playbooks foram executados novamente para validar a idempotência, sem recriar configurações já existentes.

## 3. Aplicação Coffee Shop e CI/CD

A aplicação Coffee Shop foi implantada no namespace:

`coffee-shop`

O Deployment possui duas réplicas, distribuídas entre os nós workers `node-02` e `node-03`.

A aplicação foi exposta através do NodePort:

`30080`

A disponibilidade foi validada com resposta HTTP 200.

O endpoint abaixo também foi validado:

`/metrics`

Esse endpoint disponibiliza métricas no formato Prometheus.

Foi configurado um pipeline no GitHub Actions para realizar o build da aplicação, publicar a imagem no GitHub Container Registry com uma tag baseada na SHA do commit e executar automaticamente o deploy no cluster k3s.

Os manifestos da aplicação estão versionados no repositório e permitem reaplicar o deploy de forma declarativa.

O deploy automatizado foi validado com sucesso. Após a publicação da imagem, o pipeline conecta-se ao node-01 por SSH, aplica os manifestos com Kustomize, acompanha o rolling update e confirma que a imagem correta está executando. As duas réplicas permanecem distribuídas entre os workers node-02 e node-03.

## 4. Prometheus e Grafana

Foi instalado o `kube-prometheus-stack`, contendo:

- Prometheus;
- Grafana;
- Alertmanager;
- kube-state-metrics;
- node-exporter.

Foi criado um ServiceMonitor para que o Prometheus descubra o endpoint `/metrics` da Coffee Shop.

Os dois targets da aplicação foram identificados corretamente pelo Prometheus.

O Grafana foi configurado com três fontes de dados:

- Prometheus;
- Zabbix;
- OpenSearch.

Também foram instalados os plugins:

- `alexanderzobnin-zabbix-app`;
- `grafana-opensearch-datasource`.

## 5. Monitoramento com Zabbix

O ambiente Zabbix foi implantado no Kubernetes com:

- Zabbix Server;
- Zabbix Web;
- PostgreSQL.

O Zabbix Agent 2 foi instalado nos três servidores através do Ansible.

Foram cadastrados os hosts:

- `node-01`;
- `node-02`;
- `node-03`.

Foram validadas métricas como:

- disponibilidade do agente;
- CPU;
- memória;
- uptime;
- filesystem;
- conectividade ICMP;
- disponibilidade da porta SSH.

Os três servidores ficaram disponíveis no Zabbix e enviando dados normalmente.

As triggers foram disponibilizadas principalmente através do template oficial `Linux by Zabbix agent`, aplicado aos três servidores.

Entre os alertas configurados e habilitados estão:

- indisponibilidade do nó por ICMP;
- indisponibilidade do Zabbix Agent;
- espaço em disco baixo ou criticamente baixo;
- carga média elevada;
- alta utilização de CPU;
- alta utilização de memória;
- indisponibilidade do serviço SSH;
- filesystem em modo somente leitura;
- erros e utilização elevada das interfaces de rede.

Também foi configurada uma verificação específica para a disponibilidade da porta SSH em cada servidor.

As triggers utilizam diferentes níveis de severidade, permitindo acompanhar problemas de disponibilidade, desempenho e capacidade.

## 6. OpenSearch e Fluent Bit

O OpenSearch foi instalado em modo single-node devido aos limites de disco e recursos do laboratório.

O cluster foi validado com status:

`green`

O OpenSearch Dashboards foi configurado para consulta e análise dos logs.

O Fluent Bit foi implantado como DaemonSet nos workers `node-02` e `node-03`.

A coleta utiliza buffer em memória e envia os logs para o índice:

`kubernetes-logs-*`

Os registros recebem campos como:

- `collector: fluent-bit`;
- `environment: observability-lab`;
- `stream`;
- `message`;
- `@timestamp`.

A integração foi validada com a mensagem:

`FLUENT_BIT_TEST_1783793287 coffee-shop observability`

A mensagem foi localizada no OpenSearch Dashboards e no Grafana.

Também foi criada uma política de retenção de 15 dias, permitindo preservar o histórico de logs para análise no Grafana, com acompanhamento do crescimento dos índices e do consumo de disco.

## 7. Dashboard consolidado

Foi criado o dashboard:

`Coffee Shop Observability`

O dashboard reúne dados do Prometheus, Zabbix e OpenSearch.

Os painéis apresentam:

- réplicas disponíveis da Coffee Shop;
- pods disponíveis;
- disponibilidade dos três servidores;
- utilização de CPU;
- utilização de memória;
- volume de logs;
- tabela detalhada de logs;
- mensagens recentes da aplicação.

O dashboard foi exportado e versionado em:

`grafana/coffee-shop-observability-dashboard.json`

## 8. Dificuldades, decisões e aprendizados

Durante a implantação do OpenSearch ocorreram problemas de pressão de disco nos nós do cluster.

Para estabilizar o ambiente foram realizadas as seguintes ações:

- limpeza de imagens de containers sem uso;
- movimentação do OpenSearch Dashboards para outro worker;
- execução do Fluent Bit apenas nos workers;
- utilização de buffer em memória;
- monitoramento do crescimento dos índices e do consumo de disco durante o período de retenção de 15 dias;
- configuração dos índices sem réplicas.

O plugin de segurança do OpenSearch foi desabilitado apenas para simplificar o laboratório. Em produção seria necessário utilizar autenticação, TLS, armazenamento maior, backup e alta disponibilidade.

Minha experiência anterior é principalmente com Zabbix e Grafana.

Durante este desafio aprofundei conhecimentos em Kubernetes, k3s, Ansible, Helm, Prometheus, OpenSearch, Fluent Bit e CI/CD.

Foram utilizadas documentação, pesquisa e ferramentas de IA como apoio, mas cada etapa foi executada, analisada e validada diretamente no ambiente.

## 9. Status atual e recomendações futuras

O projeto foi desenvolvido e validado ao longo de aproximadamente três dias, incluindo a preparação da infraestrutura, implantação da aplicação e integração da stack de observabilidade.

Atualmente, os três nós do cluster k3s estão disponíveis. A aplicação Coffee Shop executa com duas réplicas distribuídas entre os workers `node-02` e `node-03`, responde via NodePort e publica métricas no endpoint `/metrics`.

O pipeline de CI/CD está funcional, realizando o build, a publicação da imagem no GHCR e o deploy automático no cluster, com validação do rolling update e da imagem baseada na SHA do commit.

Prometheus, Zabbix, Grafana, OpenSearch, OpenSearch Dashboards e Fluent Bit permanecem operacionais e integrados.

Como melhorias futuras, recomenda-se:

- ampliar o armazenamento disponível para o OpenSearch;
- habilitar autenticação e TLS;
- implementar backup dos índices;
- adicionar alta disponibilidade aos componentes críticos;
- implementar Grafana Alerting e um Health Score consolidado;
- ampliar a coleta para incluir logs dos nós e dos componentes internos do cluster.

## Evidência visual do dashboard

A imagem abaixo apresenta o dashboard consolidado no Grafana.

### Visão geral do ambiente

![Coffee Shop Observability - Visão Geral](docs/images/coffee-shop-observability-overview.png)

### Logs e eventos dos nós Kubernetes

![Coffee Shop Observability - Logs e Eventos](docs/images/coffee-shop-observability-logs.png)
