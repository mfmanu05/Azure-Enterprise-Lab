# Azure ↔ AWS site-to-site VPN with StrongSwan (IKEv2)

Este módulo documenta a implementação de uma VPN site-to-site entre Microsoft Azure e Amazon AWS utilizando Azure VPN Gateway e StrongSwan (Linux), com autenticação por chave pré-compartilhada (PSK), negociação IKEv2 e validação de conectividade entre redes privadas.

## Objetivo

Implementar uma arquitetura Azure Hub-and-Spoke utilizando Azure VPN Gateway como ponto de conectividade Site-to-Site com uma VPC AWS, Global VNet Peering para integração entre redes, User Defined Routing para controle do caminho do tráfego, e uma Network Virtual Appliance (NVA) como roteador de trânsito, implementando Forced Tunneling e egress centralizado por meio de NAT Gateway.

## Arquitetura

```text
AWS VPC (192.168.100.0/24)
        │
   VPN Site-to-Site
        │
Azure VPN Gateway
        │
VNet Hub (10.100.0.0/16)
        │
NVA (10.100.10.4)
        │
UDR / Transit
        │
VNet Core (10.10.0.0/16)
        │
VM Core (10.10.1.4)
10.10.0.0/16
```

## Componentes

### Azure

* Virtual Network
* GatewaySubnet
* VPN Gateway
* Local Network Gateway
* VPN Connection

### AWS

* VPC
* Public Subnet
* Internet Gateway
* Route Table
* EC2 Ubuntu
* Elastic IP

### Linux

* StrongSwan
* IKEv2
* ESP
* XFRM

## Fluxo IKEv2

1. IKE_SA_INIT
2. IKE_AUTH
3. CHILD_SA
4. ESP

## Principais aprendizados

* funcionamento do Azure VPN Gateway;
* negociação IKEv2;
* troubleshooting de `NO_PROPOSAL_CHOSEN`;
* NAT Traversal (NAT-T);
* políticas XFRM do kernel Linux;
* conectividade híbrida Azure ↔ AWS.

## Resultado

Validação realizada com sucesso entre:

* `In  IP 10.10.1.4.34704 > 34.160.111.145.443`
* `Out IP 10.10.1.4.34704 > 34.160.111.145.443`

com 0% de perda de pacotes através do túnel IPsec.
