# Implementação — Hybrid network

## Objetivo

Este documento registra a implementação da arquitetura **Hub-and-Spoke com conectividade híbrida Azure ↔ AWS**, detalhando os recursos criados, a sequência de configuração e as decisões técnicas adotadas durante o laboratório.

---

# Escopo da implementação

Foram implementados os seguintes componentes:

* VNet Core
* VNet Hub
* Subnets
* Network Security Groups
* Global VNet Peering
* Azure VPN Gateway
* Local Network Gateway
* VPN Connection
* Network Virtual Appliance (NVA)
* Azure Bastion
* NAT Gateway
* User Defined Routing (UDR)

---

# Estrutura de Resource Groups

A arquitetura foi organizada utilizando Resource Groups dedicados por função.

| Resource Group      | Finalidade          |
| ------------------- | ------------------- |
| rg-mhcp-core        | Recursos principais |
| rg-mhcp-network     | Rede da VNet Core   |
| rg-mhcp-network-hub | Rede da VNet Hub    |
| rg-mhcp-compute     | Máquinas virtuais   |
| rg-mhcp-monitoring  | Monitoramento       |

Essa separação facilita governança, controle de acesso e gerenciamento do ciclo de vida dos recursos.

---

# Implementação da VNet Core

## Configuração

**Região:** Brazil South

**VNet:**

vnet-mhcp-core

**Espaço de endereçamento:**

10.10.0.0/16

Subnets:

* snet-app
* snet-management

A subnet de aplicação foi utilizada para hospedar a VM Core.

---

# Implementação da VNet Hub

## Configuração

**Região:** East US 2

**VNet:**

vnet-mhcp-hub

**Espaço de endereçamento:**

10.100.0.0/16

Subnets criadas:

| Subnet              | Endereçamento   |
| ------------------- | --------------- |
| snet-shared         | 10.100.10.0/24  |
| AzureFirewallSubnet | 10.100.20.0/26  |
| AzureBastionSubnet  | 10.100.30.0/26  |
| GatewaySubnet       | 10.100.254.0/27 |

A escolha da região East US 2 foi motivada pela limitação de cotas da assinatura na região Brazil South.

---

# Network Security Groups

Foi criado um NSG dedicado para a subnet compartilhada.

**NSG:**

nsg-mhcp-shared

Principais regras configuradas:

* SSH
* ICMP
* HTTPS
* tráfego interno entre VNets
* comunicação administrativa via Bastion

O objetivo foi permitir apenas o tráfego necessário para operação da NVA e testes do laboratório.

---

# Global VNet Peering

Foi estabelecido **Global VNet Peering** entre:

* vnet-mhcp-core
* vnet-mhcp-hub

Configurações habilitadas:

* Allow virtual network access
* Allow forwarded traffic

O peering permitiu conectividade privada entre as VNets utilizando a infraestrutura interna do Azure.

---

# Azure VPN Gateway

Foi implantado um **VPN Gateway** na GatewaySubnet da VNet Hub.

Configuração:

* Route-based
* IPsec/IKEv2
* conexão Site-to-Site com AWS

A AWS utilizou **strongSwan** como endpoint remoto.

O túnel passou a fornecer conectividade entre:

Azure:

10.10.0.0/16

10.100.0.0/16

AWS:

192.168.100.0/24

---

# Network Virtual Appliance (NVA)

Foi criada uma VM Linux dedicada ao roteamento de trânsito.

## Configuração

**Subnet:**

snet-shared

**Endereço privado:**

10.100.10.4

**IP Forwarding:**

habilitado na interface de rede.

No sistema operacional:

sudo sysctl -w net.ipv4.ip_forward=1

Persistência:

echo 'net.ipv4.ip_forward=1' | sudo tee -a /etc/sysctl.conf

A NVA passou a atuar como **Virtual Appliance** para as UDRs.

---

# Azure Bastion

Foi implantado um Azure Bastion na subnet:

AzureBastionSubnet

Objetivo:

* acesso administrativo às VMs sem IP público;
* eliminação da exposição direta de SSH.

Durante o laboratório o Bastion foi utilizado para administração da NVA.

---

# NAT Gateway

Foi criado um NAT Gateway associado à subnet:

snet-shared

Objetivos:

* centralizar o egress;
* controlar o endereço IP público de saída;
* suportar o Forced Tunneling.

É importante destacar que o NAT Gateway foi aplicado apenas à subnet da NVA.

---

# User Defined Routing

## Rota para o Hub

Foi criada uma Route Table para a subnet da aplicação.

**Route Table:**

rt-shared-app

Rota configurada:

Prefixo:

10.100.0.0/16

Next Hop:

Virtual Appliance

10.100.10.4

Essa configuração forçou o tráfego destinado ao Hub a passar pela NVA.

---

## Forced Tunneling

Posteriormente foi adicionada a rota:

Prefixo:

0.0.0.0/0

Next Hop:

Virtual Appliance

10.100.10.4

Resultado:

todo o tráfego externo da VNet Core passou a ser encaminhado para a NVA.

---

# Associação das Route Tables

A Route Table foi associada à subnet de aplicação da VNet Core.

Fluxo resultante:

VM Core

↓

UDR

↓

NVA

↓

Hub

↓

VPN Gateway ou NAT Gateway

Essa associação foi confirmada por meio de **Effective Routes**.

---

# Sequência de implementação

A implementação foi executada na seguinte ordem:

1. criação da VNet Core;
2. criação da VNet Hub;
3. criação das subnets;
4. criação dos NSGs;
5. configuração do Global VNet Peering;
6. implantação do Azure VPN Gateway;
7. criação do Local Network Gateway;
8. criação da VPN Connection;
9. implantação da NVA;
10. habilitação do IP Forwarding;
11. implantação do Bastion;
12. implantação do NAT Gateway;
13. criação das UDRs;
14. associação das Route Tables;
15. validação por Effective Routes;
16. validação por tcpdump;
17. testes de conectividade.

---

# Recursos principais

| Recurso     | Nome                |
| ----------- | ------------------- |
| VNet Hub    | vnet-mhcp-hub       |
| VNet Core   | vnet-mhcp-core      |
| NVA         | vm-mhcp-nva-01      |
| VM Core     | vm-mhcp-platform-01 |
| NSG Hub     | nsg-mhcp-shared     |
| Route Table | rt-shared-app       |
| NAT Gateway | nat-mhcp-hub        |
| Bastion     | bastion-mhcp-hub    |

---

# Considerações finais

A implementação consolidou uma arquitetura **Hub-and-Spoke com roteamento centralizado**, permitindo separar claramente:

* conectividade híbrida;
* trânsito entre VNets;
* controle do caminho do tráfego;
* egress para Internet.

A validação da implementação foi realizada por meio de **Effective Routes**, testes de conectividade e análise de tráfego utilizando **tcpdump**, registrados nos documentos de validação e troubleshooting do módulo.
