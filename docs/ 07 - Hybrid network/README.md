# Módulo 07 — Hybrid network

## Visão geral

Este módulo documenta a construção de uma arquitetura **Azure Hub-and-Spoke** integrada a uma **VPC na AWS** por meio de **VPN Site-to-Site (IPsec/IKEv2)**. O objetivo principal foi compreender, na prática, como o Azure separa **conectividade**, **roteamento** e **egress para Internet**, utilizando **Global VNet Peering**, **User Defined Routing (UDR)**, **Network Virtual Appliance (NVA)** e **NAT Gateway**.

Diferentemente de um laboratório focado apenas em configuração, este módulo foi conduzido com uma abordagem de engenharia, validando cada alteração por meio de **Effective Routes**, **tcpdump**, testes de conectividade e análise do comportamento do roteamento.

---

# Objetivos do módulo

* Implementar uma arquitetura Hub-and-Spoke.
* Estabelecer conectividade híbrida Azure ↔ AWS.
* Configurar **Global VNet Peering**.
* Implementar **roteamento controlado por UDR**.
* Utilizar uma **NVA como roteador de trânsito**.
* Implementar **Forced Tunneling**.
* Centralizar o egress utilizando **NAT Gateway**.
* Validar o caminho do tráfego por meio de **tcpdump** e **Effective Routes**.

---

# Arquitetura implementada

A arquitetura foi distribuída entre duas regiões do Azure para contornar limitações de cota da assinatura e aproximar o desenho de um ambiente enterprise.

## VNet Core

**Região:** Brazil South

**Espaço de endereçamento:**

10.10.0.0/16

Responsável por hospedar as cargas de trabalho da aplicação.

## VNet Hub

**Região:** East US 2

**Espaço de endereçamento:**

10.100.0.0/16

Subnets:

* snet-shared
* AzureBastionSubnet
* AzureFirewallSubnet
* GatewaySubnet

A VNet Hub concentra os serviços de conectividade e trânsito da arquitetura.

> **Screenshot sugerido:** topologia das VNets e subnets.

---

# Topologia lógica

```
            Internet
                │
          NAT Gateway
                │
        VNet Hub (10.100.0.0/16)
                │
        NVA (10.100.10.4)
                │
        Global VNet Peering
                │
      VNet Core (10.10.0.0/16)
                │
     VM Core (10.10.1.4)
                │
         Azure VPN Gateway
                │
  AWS VPC (192.168.100.0/24)
```

---

# Recursos implementados

## Conectividade

* Azure VPN Gateway
* Local Network Gateway
* VPN Connection
* AWS VPC
* strongSwan (AWS)

## Rede

* Global VNet Peering
* User Defined Routing
* Network Security Groups
* Azure Bastion
* NAT Gateway

## Validação

* Effective Routes
* tcpdump
* ping
* curl
* Azure CLI

---

# VPN Site-to-Site

A conectividade entre Azure e AWS foi implementada utilizando **IPsec/IKEv2**.

O endpoint Azure permaneceu sendo o **Azure VPN Gateway**, enquanto o endpoint AWS utilizou **strongSwan**.

## Fluxo de negociação

1. IKE_SA_INIT
2. IKE_AUTH
3. CHILD_SA
4. Encapsulamento ESP

A criptografia do túnel é responsabilidade dos dois endpoints IPsec.

> **Screenshot sugerido:** Azure VPN Connection.

---

# Global VNet Peering

O peering foi configurado entre a VNet Core e a VNet Hub.

## Objetivo

Permitir conectividade privada entre as VNets utilizando a rede interna do Azure.

O peering adiciona automaticamente rotas do tipo **Virtual Network Peering**.

## Validação

Foi confirmada conectividade entre:

* Core → Hub
* Hub → Core

> **Screenshot sugerido:** configuração do Global VNet Peering.

---

# User Defined Routing (UDR)

Este foi o principal componente do módulo.

Inicialmente, o tráfego entre VNets utilizava exclusivamente o peering.

Posteriormente, foi criada uma **Route Table** para controlar explicitamente o caminho do tráfego.

## Rota para a VNet Hub

Prefixo:

10.100.0.0/16

Next Hop:

Virtual Appliance

10.100.10.4

Resultado:

Todo o tráfego destinado ao Hub passou a ser encaminhado pela NVA.

> **Screenshot sugerido:** Route Table da subnet da aplicação.

---

# Effective Routes

A validação das rotas efetivas foi realizada diretamente na NIC da VM Core.

Essa etapa permitiu confirmar:

* rotas aprendidas pelo peering;
* rotas provenientes da UDR;
* prioridade entre rotas do sistema e rotas definidas pelo usuário.

A análise de **Effective Routes** foi utilizada como principal ferramenta de troubleshooting do Azure.

> **Screenshot sugerido:** Effective Routes da VM Core.

---

# Network Virtual Appliance (NVA)

Foi criada uma VM Linux atuando como **roteador de trânsito**.

## Configuração

### Azure

Enable IP Forwarding na interface de rede.

### Linux

Ativação do encaminhamento IP:

sudo sysctl -w net.ipv4.ip_forward=1

Persistência:

echo 'net.ipv4.ip_forward=1' | sudo tee -a /etc/sysctl.conf

A NVA passou a encaminhar tráfego entre a VNet Core, a VNet Hub e os destinos externos definidos pelas UDRs.

---

# NAT Gateway

O NAT Gateway foi associado à **subnet da NVA (snet-shared)**.

## Objetivo

Centralizar o **egress para Internet** utilizando um endereço IP público controlado.

## Conceito importante

O NAT Gateway **não realiza roteamento**.

Ele apenas aplica **Source NAT (SNAT)** ao tráfego de saída da subnet.

Essa distinção foi essencial para compreender o comportamento observado durante os testes de Forced Tunneling.

> **Screenshot sugerido:** NAT Gateway associado à subnet da NVA.

---

# Forced Tunneling

Após a validação do trânsito entre VNets, foi implementado **Forced Tunneling**.

## Rota adicionada

Prefixo:

0.0.0.0/0

Next Hop:

Virtual Appliance

10.100.10.4

A partir desse momento, todo o tráfego externo da VNet Core passou a ser enviado para a NVA.

Fluxo:

VM Core

↓

UDR

↓

NVA

↓

NAT Gateway

↓

Internet

---

# Validação do trânsito via NVA

A principal validação foi realizada utilizando **tcpdump**.

## Captura ICMP

Comando:

sudo tcpdump -n -i any host 10.100.11.4

Resultado:

In  IP 10.10.1.4 > 10.100.11.4

Out IP 10.10.1.4 > 10.100.11.4

Essa captura comprovou que a NVA recebeu o pacote e o encaminhou para outra subnet.

Os endereços IP permaneceram preservados porque a NVA estava **roteando**, e não realizando NAT.

---

# Validação do Forced Tunneling

Comando:

sudo tcpdump -n -i any tcp port 443

Resultado:

In  IP 10.10.1.4:34704 > 34.x.x.x:443

Out IP 10.10.1.4:34704 > 34.x.x.x:443

Essa evidência confirmou que o tráfego HTTPS originado na VNet Core foi desviado pela UDR e encaminhado pela NVA.

O NAT Gateway atua posteriormente, na borda da subnet, aplicando SNAT ao tráfego de saída.

> **Screenshot sugerido:** captura do tcpdump HTTPS.

---

# Hairpin Routing

Durante os testes foi identificado um comportamento anômalo quando a VM de teste estava na **mesma subnet da NVA**.

## Topologia inicial

Core

↓

NVA (10.100.10.4)

↓

VM Teste (10.100.10.5)

O tcpdump mostrou entrada e saída pela mesma interface.

## Causa

NVA e VM pertenciam ao mesmo **domínio L2**.

O roteador precisava receber e retransmitir o tráfego pela mesma interface.

Esse comportamento é conhecido como **Hairpin Routing (U-Turn Routing)**.

## Solução

A VM foi movida para uma subnet dedicada:

10.100.11.0/24

Após a separação em domínios L3 distintos, o comportamento tornou-se previsível e consistente.

---

# L2 vs L3

Este módulo evidenciou uma diferença fundamental.

## L2 (mesma subnet)

* comunicação via ARP;
* endereços MAC;
* mesmo domínio de broadcast.

## L3 (subnets diferentes)

* comunicação via endereços IP;
* roteamento explícito;
* domínios separados.

A principal conclusão foi:

> **Uma NVA deve permanecer em uma subnet dedicada para evitar Hairpin Routing e garantir roteamento de trânsito consistente.**

---

# Troubleshooting realizado

## VPN

* NO_PROPOSAL_CHOSEN
* incompatibilidade de propostas IKE

## NVA

* IP Forwarding desabilitado
* regras de NSG
* conectividade da subnet

## Roteamento

* UDR sobrescrevendo o comportamento do peering
* validação por Effective Routes

## Hairpin

* NVA e destino na mesma subnet

## Egress

* distinção entre roteamento e NAT Gateway

---

# Principais aprendizados

Este módulo consolidou vários conceitos fundamentais de Azure Networking.

## Conectividade

O **Global VNet Peering** fornece conectividade privada entre VNets.

## Roteamento

A **User Defined Route** controla explicitamente o caminho do tráfego.

## Trânsito

A **NVA** atua como roteador de trânsito entre redes.

## Egress

O **NAT Gateway** fornece saída controlada para Internet.

## Arquitetura

A separação entre conectividade, roteamento e NAT é essencial para compreender arquiteturas Hub-and-Spoke.

---

# Conclusão

O laboratório evoluiu de uma simples conexão VPN para uma arquitetura **Hub-and-Spoke com controle explícito de roteamento**, permitindo validar na prática como o Azure trata conectividade, trânsito entre VNets e egress para Internet.

A principal conclusão do módulo é:

> **O peering cria conectividade; a UDR controla o caminho do tráfego.**

Esse comportamento foi comprovado por meio de **Effective Routes**, **tcpdump** e testes de conectividade, reproduzindo uma arquitetura de rede típica de ambientes Azure Enterprise.
