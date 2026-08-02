# Arquitetura — Hybrid network

## Visão geral

A arquitetura foi projetada utilizando o modelo **Hub-and-Spoke**, no qual a **VNet Hub** concentra os serviços de conectividade e trânsito da infraestrutura, enquanto a **VNet Core** hospeda as cargas de trabalho da aplicação.

A integração híbrida com a AWS foi implementada por meio de **VPN Site-to-Site (IPsec/IKEv2)**, utilizando o **Azure VPN Gateway** como endpoint do lado Azure e **strongSwan** como endpoint do lado AWS.

O objetivo principal da arquitetura foi centralizar o controle de roteamento, conectividade entre VNets e saída para Internet.

---

## Princípios da arquitetura

O desenho foi baseado nos seguintes princípios:

* separação entre conectividade e roteamento;
* centralização dos serviços de rede;
* controle explícito do caminho do tráfego;
* isolamento de funções por subnet;
* reutilização da infraestrutura para módulos futuros.

---

## Topologia lógica

```text
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

## Componentes da arquitetura

### VNet Core

**Região:** Brazil South

Responsável pelas cargas de trabalho da aplicação.

Espaço de endereçamento:

```text
10.10.0.0/16
```

Funções:

* hospedagem de VMs;
* origem do tráfego da aplicação;
* consumo dos serviços centralizados do Hub.

---

### VNet Hub

**Região:** East US 2

Responsável pelos serviços de conectividade.

Espaço de endereçamento:

```text
10.100.0.0/16
```

Subnets:

| Subnet              | Função                        |
| ------------------- | ----------------------------- |
| snet-shared         | NVA e serviços compartilhados |
| AzureBastionSubnet  | Acesso administrativo         |
| AzureFirewallSubnet | Reserva para Azure Firewall   |
| GatewaySubnet       | Azure VPN Gateway             |

---

### Azure VPN Gateway

O Azure VPN Gateway é responsável por estabelecer o túnel IPsec/IKEv2 com a AWS.

Responsabilidades:

* negociação IKE;
* gerenciamento das Security Associations;
* encapsulamento ESP;
* conectividade híbrida.

O VPN Gateway fornece conectividade segura entre Azure e AWS, enquanto o roteamento interno permanece sob controle das UDRs e da NVA.

---

### Network Virtual Appliance (NVA)

A NVA foi implementada como uma VM Linux dedicada ao roteamento de trânsito.

Endereço:

```text
10.100.10.4
```

Responsabilidades:

* roteamento entre VNets;
* processamento das UDRs;
* encaminhamento de tráfego;
* Forced Tunneling;
* integração com o NAT Gateway.

A NVA não atua como endpoint IPsec da VPN.

---

### NAT Gateway

O NAT Gateway foi associado à subnet da NVA.

Responsabilidade:

* egress centralizado para Internet;
* tradução de endereços de origem (SNAT).

É importante destacar que o NAT Gateway não realiza roteamento.

---

## Fluxo de conectividade

### Comunicação entre VNets

O tráfego entre a VNet Core e a VNet Hub utiliza **Global VNet Peering**.

Sem UDR:

```text
VM Core
   │
Peering
   │
VNet Hub
```

Com UDR:

```text
VM Core
   │
UDR
   │
NVA
   │
VNet Hub
```

---

### Comunicação com a AWS

O tráfego destinado à AWS segue o fluxo:

```text
VM Core
   │
UDR
   │
NVA
   │
VNet Hub
   │
Azure VPN Gateway
   │
AWS
```

O túnel IPsec é estabelecido exclusivamente entre:

* Azure VPN Gateway;
* strongSwan (AWS).

---

### Saída para Internet

O tráfego de saída segue o fluxo:

```text
VM Core
   │
UDR (0.0.0.0/0)
   │
NVA
   │
NAT Gateway
   │
Internet
```

Esse desenho implementa **Forced Tunneling**, garantindo que a saída da VNet Core seja controlada centralmente.

---

## Separação de responsabilidades

### Global VNet Peering

Responsável por:

* conectividade privada;
* troca de rotas entre VNets.

### User Defined Routing

Responsável por:

* controle do caminho do tráfego;
* desvio para a NVA;
* Forced Tunneling.

### NVA

Responsável por:

* trânsito;
* roteamento;
* encaminhamento.

### NAT Gateway

Responsável por:

* tradução de endereços de saída.

### VPN Gateway

Responsável por:

* conectividade híbrida;
* criptografia IPsec.

---

## Decisões de arquitetura

### Hub em região separada

A VNet Hub foi criada em **East US 2** devido às limitações de cota da assinatura na região Brazil South.

Essa decisão permitiu:

* expansão da infraestrutura;
* criação de recursos adicionais;
* separação entre plano de conectividade e plano de aplicação.

---

### NVA em subnet dedicada

A NVA permaneceu isolada na **snet-shared**.

Essa decisão evitou problemas de **Hairpin Routing**, identificados durante os testes quando a NVA e a VM de destino estavam na mesma subnet.

---

### UDR como mecanismo de controle

O roteamento foi implementado por UDR para permitir:

* previsibilidade;
* controle explícito;
* validação por Effective Routes.

---

## Considerações de alta disponibilidade

Nesta fase do laboratório a NVA foi implementada como instância única.

Em um ambiente de produção recomenda-se:

* múltiplas NVAs;
* Load Balancer interno;
* Health Probes;
* tabelas de rota redundantes.

Essa evolução será explorada nos módulos posteriores do laboratório.

---

## Conclusão

A arquitetura implementada separa claramente os papéis de **conectividade, roteamento e egress**, permitindo compreender como o Azure combina **VPN Gateway, Global VNet Peering, User Defined Routing, NVA e NAT Gateway** em uma arquitetura Hub-and-Spoke de padrão enterprise.

O desenho obtido servirá como base para os próximos módulos, especialmente **Load Balancing, Segurança e Monitoramento**, mantendo uma estrutura escalável e reutilizável para a evolução do laboratório.
