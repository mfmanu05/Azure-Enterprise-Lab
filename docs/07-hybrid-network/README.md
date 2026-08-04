# Lab 07 — Azure hybrid networking with Hub-and-Spoke and Site-to-Site VPN

## Objetivo

Este laboratório demonstra a implementação de uma arquitetura de rede híbrida utilizando o modelo **Hub-and-Spoke**, conectando uma infraestrutura Azure a uma **VPC na AWS através de uma VPN Site-to-Site (IPsec/IKEv2)**.

O foco do laboratório é validar **segmentação de rede**, **roteamento centralizado**, **forced tunneling**, **integração entre nuvens** e o uso de uma **Network Virtual Appliance (NVA)** como ponto central de controle de tráfego.

## Arquitetura

A implementação inclui:

* VNet Hub
* VNet Core
* Network Virtual Appliance (NVA)
* Azure Bastion
* User Defined Routes (UDR)
* VPN Gateway
* NAT Gateway
* Conectividade Site-to-Site com AWS
* Roteamento centralizado

### Diagrama

[Inserir diagrama da arquitetura]

## Principais componentes

### Hub-and-Spoke

* Segmentação entre redes.
* Administração centralizada.
* Compartilhamento de serviços comuns.

### VPN Site-to-Site

* Conectividade entre Azure e AWS.
* Criptografia IPsec/IKEv2.
* Validação de comunicação entre ambientes.

### Network Virtual Appliance

* Roteamento centralizado.
* Forced Tunneling.
* Controle do tráfego entre VNets.

## Principais cenários testados

* Criação da arquitetura Hub-and-Spoke.
* Configuração de peering entre VNets.
* Implementação da NVA.
* Configuração de UDR.
* Forced Tunneling.
* VPN Site-to-Site Azure ↔ AWS.
* Validação de conectividade híbrida.
* Troubleshooting de roteamento e NSGs.

## Estrutura da documentação

| Documento              | Descrição                                  |
| ---------------------- | ------------------------------------------ |
| **overview.md**        | Visão geral detalhada da arquitetura       |
| **implementation.md**  | Processo completo de implementação         |
| **validation.md**      | Testes executados e resultados obtidos     |
| **troubleshooting.md** | Problemas encontrados e soluções aplicadas |

## Principais aprendizados

O laboratório demonstrou como o modelo **Hub-and-Spoke** permite centralizar serviços de rede e segurança, enquanto a utilização de uma **NVA** fornece controle granular do roteamento entre VNets e ambientes externos.

Também foram validados conceitos fundamentais de **VPN híbrida**, **forced tunneling** e **integração entre Azure e AWS**, preparando a base de rede utilizada nos módulos seguintes do projeto.

## Próximo laboratório

O próximo módulo aborda **Azure VM Scale Set Flexible e Autoscaling**, introduzindo conceitos de elasticidade horizontal, balanceamento de carga e escalabilidade automática da infraestrutura.
