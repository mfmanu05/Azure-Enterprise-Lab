# Lab 08 — Azure VM Scale Set Flexible and autoscaling

## Objetivo

Este laboratório demonstra a implementação de uma arquitetura escalável utilizando **Azure Virtual Machine Scale Set (VMSS) com Orchestration Mode Flexible**, integrado a um **Azure Load Balancer Standard** e configurado com **autoscaling baseado em métricas**.

O foco do laboratório é validar o comportamento de **elasticidade horizontal**, **balanceamento de carga** e **escalabilidade automática**, utilizando um ambiente totalmente isolado e independente da arquitetura Hub-and-Spoke desenvolvida nos módulos anteriores.

## Arquitetura

A implementação inclui:

* Azure VM Scale Set (Flexible Orchestration)
* Azure Load Balancer Standard
* Azure Monitor Autoscale
* NAT Gateway
* Network Security Group
* VNet dedicada ao laboratório
* Integração com Azure Bastion através de peering

### Diagrama

[Inserir diagrama da arquitetura]

## Principais cenários testados

* Balanceamento de carga entre múltiplas instâncias
* Scale Out manual
* Scale In manual
* Scale Out automático por métricas
* Scale In automático por métricas
* Integração automática com Azure Load Balancer
* Comportamento do template do VM Scale Set

## Estrutura da documentação

| Documento              | Descrição                                  |
| ---------------------- | ------------------------------------------ |
| **overview.md**        | Visão geral do laboratório                 |
| **implementation.md**  | Processo completo de implementação         |
| **validation.md**      | Testes executados e resultados obtidos     |
| **troubleshooting.md** | Problemas encontrados e soluções aplicadas |

## Principais aprendizados

O laboratório evidenciou que o **VM Scale Set Flexible** utiliza o **template do conjunto como fonte da verdade**, permitindo escalabilidade horizontal automática, integração nativa com o Load Balancer e elasticidade baseada em políticas do Azure Monitor.

## Próximo laboratório

O próximo módulo abordará **alta disponibilidade e continuidade de serviços**, expandindo a arquitetura para cenários enterprise com componentes redundantes e estratégias de resiliência.
