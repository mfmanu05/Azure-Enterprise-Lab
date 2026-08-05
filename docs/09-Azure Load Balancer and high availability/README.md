# Lab 09 — Azure Load Balancer and high availability

## Objetivo

Este laboratório demonstra a implementação de uma arquitetura de **alta disponibilidade de aplicações** utilizando o **Azure Load Balancer Standard**, validando distribuição de tráfego, Health Probe, failover automático e recuperação de instâncias.

O foco do laboratório é demonstrar como o Azure Load Balancer mantém a continuidade do serviço quando uma ou mais instâncias da aplicação tornam-se indisponíveis.

## Arquitetura

A implementação inclui:

* Azure Load Balancer Standard
* Backend Pool com múltiplas máquinas virtuais
* Health Probe HTTP
* Load Balancing Rules
* VNet dedicada ao laboratório
* NSG
* NAT Gateway
* Azure Bastion através de peering com a VNet Hub

### Diagrama

[Inserir diagrama da arquitetura]

## Principais componentes

### Azure Load Balancer Standard

* Distribuição automática de conexões
* Backend Pool
* Health Probe
* Regras de balanceamento

### Máquinas virtuais

* Ubuntu Linux
* Nginx
* Páginas personalizadas para identificação das instâncias

### Alta disponibilidade

* Múltiplas instâncias da aplicação
* Failover automático
* Recuperação automática do backend

## Principais cenários testados

* Balanceamento de carga
* Health Probe
* Falha do serviço Nginx
* Falha de uma instância
* Remoção automática do backend
* Recuperação do backend
* Continuidade do atendimento

## Estrutura da documentação

| Documento              | Descrição                                  |
| ---------------------- | ------------------------------------------ |
| **overview.md**        | Visão geral detalhada da arquitetura       |
| **implementation.md**  | Processo completo de implementação         |
| **validation.md**      | Testes executados e resultados obtidos     |
| **troubleshooting.md** | Problemas encontrados e soluções aplicadas |

## Principais aprendizados

O laboratório demonstra como o **Azure Load Balancer Standard** utiliza **Health Probes** para detectar indisponibilidade de aplicações e remover automaticamente instâncias do Backend Pool, mantendo a continuidade do serviço para os clientes.

Também evidencia a diferença entre **balanceamento de carga** e **alta disponibilidade**, mostrando que o Load Balancer atua como componente central de distribuição e failover da aplicação.

## Próximo laboratório

O próximo módulo abordará **Azure Monitor, Log Analytics e Monitoramento**, integrando observabilidade e coleta de métricas à arquitetura implementada.
