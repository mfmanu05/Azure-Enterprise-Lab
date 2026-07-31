# Azure ↔ AWS site-to-site VPN with StrongSwan (IKEv2)

Este módulo documenta a implementação de uma VPN site-to-site entre Microsoft Azure e Amazon AWS utilizando Azure VPN Gateway e StrongSwan (Linux), com autenticação por chave pré-compartilhada (PSK), negociação IKEv2 e validação de conectividade entre redes privadas.

## Objetivo

Estabelecer comunicação segura entre:

* Azure VNet: `10.10.0.0/16`
* AWS VPC: `192.168.100.0/24`

por meio de um túnel IPsec criptografado.

## Arquitetura

```text
AWS (EC2 + StrongSwan)
192.168.100.0/24
        │
        │ IPsec IKEv2
        │
Azure VPN Gateway
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

* `192.168.100.252`
* `10.10.1.4`

com 0% de perda de pacotes através do túnel IPsec.
