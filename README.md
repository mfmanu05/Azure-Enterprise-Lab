# Azure Enterprise Lab

Laboratório prático de arquitetura Azure focado na preparação para a certificação **AZ-104**, construído com uma abordagem orientada à engenharia: **entender antes de decorar**.

Este repositório documenta a construção de uma infraestrutura Azure inspirada em ambientes enterprise, cobrindo identidade, governança, armazenamento, computação, redes, conectividade híbrida, segurança, monitoramento e automação.

## Objetivo

Construir uma arquitetura Azure de forma incremental, validando cada componente por meio de testes práticos, troubleshooting e documentação técnica.

A filosofia do projeto é simples:

> **Sem enrolação. Sem pular etapas. Sempre entendendo o porquê das coisas.**

## Roadmap do laboratório

A documentação está organizada em módulos numerados, seguindo uma evolução cronológica da arquitetura.

| Módulo                  | Descrição                                                   | Status        |
| ----------------------- | ----------------------------------------------------------- | ------------- |
| 01 — Foundation         | Assinatura, Resource Groups e Azure CLI                     | Concluído     |
| 02 — Identity           | Microsoft Entra ID, RBAC e grupos                           | Concluído     |
| 03 — Governance         | Organização, tags e governança                              | Concluído     |
| 04 — Storage            | Storage Accounts, Backup e redundância                      | Concluído     |
| 05 — Compute            | Máquinas virtuais, discos e disponibilidade                 | Concluído     |
| 06 — Networking         | VNets, Subnets, NSG e Global Peering                        | Concluído     |
| **07 — Hybrid network** | **VPN Site-to-Site, Hub-and-Spoke, UDR, NVA e NAT Gateway** | **Concluído** |
| 08 — Load balancing     | Azure Load Balancer e alta disponibilidade                  | Em andamento  |
| 09 — Security           | Azure Firewall, Bastion e segmentação                       | Planejado     |
| 10 — Monitoring         | Azure Monitor, Log Analytics e alertas                      | Planejado     |
| 11 — Backup & DR        | Recovery Services Vault e continuidade                      | Planejado     |
| 12 — Automation         | Terraform, Azure CLI e automação                            | Planejado     |
| 13 — Final architecture | Arquitetura enterprise consolidada                          | Planejado     |

## Arquitetura atual

O laboratório implementa uma arquitetura **Hub-and-Spoke** com integração híbrida entre **Azure e AWS**.

Tecnologias utilizadas:

* Microsoft Azure
* AWS
* Azure VPN Gateway
* Global VNet Peering
* User Defined Routing (UDR)
* Network Virtual Appliance (NVA)
* NAT Gateway
* Azure Bastion
* Azure CLI
* Bash
* Linux (Ubuntu)
* strongSwan
* IPsec / IKEv2

## Estrutura da documentação

A documentação detalhada encontra-se no diretório `docs/`, organizada por módulos.

Cada módulo contém:

* objetivo;
* arquitetura;
* implementação;
* validação;
* troubleshooting;
* lições aprendidas;
* evidências.

## Principais conceitos explorados

* arquitetura Hub-and-Spoke;
* conectividade híbrida Azure ↔ AWS;
* VPN Site-to-Site;
* roteamento entre VNets;
* Global Peering;
* Forced Tunneling;
* trânsito via NVA;
* Effective Routes;
* Hairpin Routing;
* troubleshooting de rede.

## Autor

**Manuel Fernandes Andrade**

Este repositório faz parte da minha jornada de especialização em **Cloud Computing, Azure e Arquitetura de Infraestrutura**.
