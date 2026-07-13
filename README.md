<!-- OBSERVABILITY_CHALLENGE_INTRO -->

# Coffee Shop Observability Challenge

Este repositório utiliza uma aplicação-base já existente como workload para um desafio técnico de Observabilidade.

O foco do trabalho desenvolvido foi a criação e integração da infraestrutura, automação, monitoramento, centralização de logs, dashboards e CI/CD.

## Arquitetura implementada

- cluster k3s com três servidores Linux;
- `node-01` como control plane;
- `node-02` e `node-03` como workers;
- aplicação Coffee Shop com duas réplicas;
- métricas coletadas com Prometheus;
- monitoramento dos servidores com Zabbix;
- logs da aplicação enviados pelo Fluent Bit ao OpenSearch;
- visualização consolidada no Grafana;
- build, publicação no GHCR e deploy automático com GitHub Actions.

## Entregáveis desenvolvidos

- [`ansible/`](ansible/) — preparação dos servidores, instalação do k3s e configuração do Zabbix Agent;
- [`kubernetes/coffee-shop/`](kubernetes/coffee-shop/) — manifestos Kubernetes e Kustomize;
- [`helm/`](helm/) — configurações de Prometheus, Zabbix, OpenSearch, OpenSearch Dashboards e Fluent Bit;
- [`grafana/`](grafana/) — dashboard consolidado exportado;
- [`.github/workflows/build-image.yml`](.github/workflows/build-image.yml) — build, publicação da imagem e deploy automático no k3s;
- [`docs/images/`](docs/images/) — evidências visuais;
- [`REPORT.md`](REPORT.md) — relatório técnico completo da implementação.

## Validações realizadas

- três nós do cluster em estado `Ready`;
- aplicação respondendo com HTTP 200;
- endpoint `/metrics` disponível;
- duas réplicas distribuídas entre os workers;
- targets da aplicação ativos no Prometheus;
- três servidores monitorados pelo Zabbix;
- OpenSearch com status `green`;
- dashboard integrando Prometheus, Zabbix e OpenSearch;
- pipeline de CI/CD validado com deploy baseado na SHA do commit.

## Aplicação-base e histórico Git

Parte do código-fonte e do histórico de commits pertence ao projeto original utilizado como base para a aplicação Coffee Shop.

Os arquivos e commits antigos foram preservados para manter a rastreabilidade e o funcionamento da aplicação. O trabalho realizado para o desafio está concentrado nos diretórios e arquivos indicados na seção **Entregáveis desenvolvidos**.

Para detalhes técnicos, decisões, dificuldades e recomendações, consulte o [`REPORT.md`](REPORT.md).

---

## Documentação original da aplicação

![Coffee Shop Logo](https://raw.githubusercontent.com/thaycafe/coffee-shop/master/frontend/src/assets/public/images/CoffeeShop_Logo.png) 

# OWASP Coffee Shop

Coffee shop is a forked OWASP Juice shop application, that is is probably the most modern and sophisticated insecure web application! It can be used in security
trainings, awareness demos, CTFs and as a guinea pig for security tools! Coffee Shop encompasses vulnerabilities from the entire


## Licensing

[![license](https://img.shields.io/github/license/bkimminich/Coffee-shop.svg)](LICENSE)

The MIT License (MIT)

- Copyright © 2014-2023 Bjoern Kimminich & the OWASP Coffee Shop contributors
- Copyright © 2023 Thaynara Mendes and Samuel Gonçalves

