# Overview — Lab 09

## Visão geral

Este laboratório demonstra a implementação de uma arquitetura de **alta disponibilidade de aplicações** utilizando o **Azure Load Balancer Standard**, validando distribuição de tráfego, detecção de falhas, failover automático e recuperação de instâncias.

A solução foi construída utilizando **duas máquinas virtuais Ubuntu executando Nginx**, posicionadas atrás de um **Azure Load Balancer Standard**, responsável por distribuir as conexões dos clientes e monitorar continuamente a saúde das instâncias através de **Health Probes**.

O laboratório complementa o **Lab 08**, que abordou elasticidade e escalabilidade automática, concentrando-se agora em **continuidade de serviço e tolerância a falhas**.

## Objetivos técnicos

Ao final deste laboratório será possível compreender:

* funcionamento do Azure Load Balancer Standard;
* criação de Backend Pools;
* configuração de Health Probes;
* regras de balanceamento de carga;
* detecção automática de indisponibilidade;
* remoção automática de instâncias do backend;
* recuperação automática após restabelecimento do serviço;
* comportamento do Load Balancer durante falhas de aplicação e infraestrutura.

## Arquitetura da solução

A arquitetura implementada foi composta pelos seguintes recursos:

* Resource Group dedicado;
* VNet isolada;
* Subnet App;
* Network Security Group;
* NAT Gateway;
* Azure Load Balancer Standard;
* Public IP Standard;
* duas máquinas virtuais Ubuntu;
* Azure Bastion através de peering com a VNet Hub.

### Diagrama

[Inserir diagrama da arquitetura]

Fluxo simplificado:

Cliente → Public IP → Azure Load Balancer → Backend Pool → VM APP 01 / VM APP 02

## Componentes principais

### Azure Load Balancer Standard

O Azure Load Balancer Standard atua como ponto único de entrada para a aplicação.

Suas responsabilidades incluem:

* distribuição automática de conexões;
* monitoramento da saúde das instâncias;
* remoção automática de instâncias indisponíveis;
* manutenção da continuidade do serviço.

### Backend Pool

O Backend Pool contém as máquinas virtuais responsáveis pelo processamento das requisições.

Neste laboratório o Backend Pool foi composto por:

* vm-app-09-01;
* vm-app-09-02.

Cada VM executa uma instância do **Nginx** com página personalizada para identificação durante os testes.

### Health Probe

O Health Probe realiza verificações periódicas na porta HTTP das máquinas virtuais.

Quando uma instância deixa de responder:

* o Azure a marca como **Down**;
* remove automaticamente a instância do Backend Pool;
* interrompe o envio de novas conexões para essa VM.

Esse comportamento é fundamental para implementação de **failover automático**.

## Fluxo operacional

### Cenário normal

Em condições normais:

* ambas as VMs permanecem **Healthy**;
* o Load Balancer distribui conexões entre elas;
* o serviço permanece disponível.

### Falha da aplicação

Quando o serviço Nginx é interrompido em uma das instâncias:

1. o Health Probe detecta a falha;
2. a VM é marcada como **Down**;
3. o Azure remove automaticamente a instância do Backend Pool;
4. todas as novas conexões passam a ser direcionadas para a instância saudável.

O cliente continua acessando a aplicação sem necessidade de alteração no endpoint público.

### Falha da infraestrutura

Quando uma máquina virtual é desligada:

1. o Backend Health marca a instância como **Down**;
2. o Backend Pool identifica a VM como **Stopped**;
3. o Load Balancer remove automaticamente a instância do balanceamento;
4. o tráfego continua sendo entregue pela instância saudável.

### Recuperação

Após reinicialização do Nginx ou da máquina virtual:

1. o Health Probe volta a receber respostas válidas;
2. a instância é marcada como **Healthy**;
3. o Azure a reintegra automaticamente ao Backend Pool;
4. o balanceamento entre as instâncias é restabelecido.

## Diferença em relação ao Lab 08

O Lab 08 concentrou-se em **elasticidade**.

O Lab 09 concentra-se em **disponibilidade**.

| Lab 08                | Lab 09                       |
| --------------------- | ---------------------------- |
| VM Scale Set Flexible | Azure Load Balancer Standard |
| Escalabilidade        | Alta disponibilidade         |
| Scale Out / Scale In  | Failover                     |
| Autoscaling           | Health Probe                 |
| Elasticidade          | Continuidade de serviço      |

Os dois laboratórios representam pilares complementares da arquitetura Azure.

## Resultados obtidos

Durante o laboratório foram validados os seguintes cenários:

* balanceamento de carga;
* falha da aplicação (Nginx);
* falha da infraestrutura (VM desligada);
* indisponibilidade total do Backend Pool;
* recuperação automática das instâncias;
* reintegração automática ao balanceamento.

Foi observado que o **Azure Load Balancer Standard utiliza Health Probes para monitorar continuamente as cargas da aplicação**, removendo automaticamente instâncias indisponíveis e mantendo a continuidade do serviço sempre que existir pelo menos um backend saudável.

## Principais aprendizados

O laboratório evidenciou aspectos fundamentais da alta disponibilidade em Azure:

* o Health Probe monitora a aplicação, e não apenas a máquina virtual;
* Backend Health e Backend Pool fornecem informações complementares para diagnóstico;
* o Load Balancer distribui tráfego apenas entre instâncias saudáveis;
* existe um tempo de convergência para detecção e recuperação de falhas;
* o Azure executa automaticamente o processo de failover e recuperação do backend.

## Conclusão

Este laboratório validou o funcionamento completo do **Azure Load Balancer Standard** em cenários de alta disponibilidade, demonstrando distribuição de tráfego, detecção automática de falhas, failover e recuperação de instâncias.

A principal conclusão foi que o **Load Balancer Standard fornece alta disponibilidade local da aplicação**, garantindo continuidade do serviço durante falhas individuais, desde que exista pelo menos um backend saudável disponível para atender às requisições dos clientes.
