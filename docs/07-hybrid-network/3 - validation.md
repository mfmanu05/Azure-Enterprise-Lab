# Validação — Hybrid network

## Objetivo

Este documento registra os testes realizados para validar a arquitetura **Hub-and-Spoke com conectividade híbrida Azure ↔ AWS**, comprovando o funcionamento do **Global VNet Peering**, **User Defined Routing (UDR)**, **Network Virtual Appliance (NVA)**, **Forced Tunneling** e **NAT Gateway**.

Todos os testes foram executados após a conclusão da implementação do módulo.

---

# Metodologia de validação

A validação foi baseada em três níveis de análise:

* conectividade;
* roteamento;
* inspeção de tráfego.

Ferramentas utilizadas:

* ping
* curl
* tcpdump
* Azure CLI
* Effective Routes
* ip route

---

# Validação do Global VNet Peering

## Objetivo

Confirmar conectividade privada entre a VNet Core e a VNet Hub.

## Teste

Da VM Core:

ping 10.100.11.4

## Resultado esperado

Resposta ICMP entre as VNets.

## Resultado obtido

64 bytes from 10.100.11.4

## Conclusão

O **Global VNet Peering** forneceu conectividade privada entre as VNets.

> **Screenshot:** peering conectado.

---

# Validação do estado do Peering

## Azure CLI

az network vnet peering show 
--resource-group rg-mhcp-network 
--vnet-name vnet-mhcp-core 
--name peer-core-to-hub

## Resultado esperado

"peeringState": "Connected"

## Resultado obtido

Connected

## Conclusão

O peering encontrava-se operacional.

---

# Validação das UDRs

## Objetivo

Verificar se o tráfego estava sendo desviado para a NVA.

## Route Table

Rota configurada:

Prefixo:

10.100.0.0/16

Next Hop:

Virtual Appliance

10.100.10.4

## Resultado esperado

Todo o tráfego para a VNet Hub deve utilizar a NVA como próximo salto.

## Conclusão

A UDR substituiu o comportamento padrão do peering.

> **Screenshot:** Route Table.

---

# Validação das Effective Routes

## Objetivo

Confirmar que a UDR foi efetivamente aplicada na VM Core.

## Verificação

Portal Azure

VM

Networking

Effective Routes

## Resultado esperado

Rota:

10.100.0.0/16

Virtual Appliance

10.100.10.4

## Resultado obtido

A rota apareceu como **User Route**.

## Conclusão

A UDR passou a ter prioridade sobre a rota do peering.

Essa validação confirmou o princípio:

> **A UDR controla o caminho do tráfego.**

> **Screenshot:** Effective Routes.

---

# Validação do trânsito via NVA

## Objetivo

Comprovar que o tráfego entre VNets passou a atravessar a NVA.

## Captura

sudo tcpdump -n -i any host 10.100.11.4

## Resultado observado

In  IP 10.10.1.4 > 10.100.11.4

Out IP 10.10.1.4 > 10.100.11.4

## Interpretação

O pacote:

* entrou na NVA;
* foi encaminhado pela NVA;
* manteve os endereços IP de origem e destino.

## Conclusão

A NVA passou a atuar como **roteador de trânsito**.

> **Screenshot:** tcpdump ICMP.

---

# Validação do Forced Tunneling

## Objetivo

Comprovar que o tráfego destinado à Internet foi desviado para a NVA.

## UDR aplicada

Prefixo:

0.0.0.0/0

Next Hop:

Virtual Appliance

10.100.10.4

## Captura

sudo tcpdump -n -i any tcp port 443

## Resultado observado

In  IP 10.10.1.4:34704 > 34.x.x.x:443

Out IP 10.10.1.4:34704 > 34.x.x.x:443

## Interpretação

A VM Core iniciou uma conexão HTTPS.

A NVA:

* recebeu a conexão;
* encaminhou a conexão;
* preservou o endereço IP de origem.

## Conclusão

O **Forced Tunneling** foi validado com sucesso.

> **Screenshot:** tcpdump HTTPS.

---

# Validação da VPN Site-to-Site

## Objetivo

Confirmar conectividade entre Azure e AWS.

## Teste

Da VM na AWS:

ping 10.10.1.4

## Resultado esperado

Resposta ICMP.

## Resultado obtido

Conectividade estabelecida.

## Conclusão

O túnel IPsec/IKEv2 encontrava-se operacional.

> **Screenshot:** VPN Connection.

---

# Validação da tabela de roteamento da NVA

## Comando

ip route

## Resultado observado

default via 10.100.10.1

## Interpretação

A NVA utilizava o gateway padrão da subnet da VNet Hub.

## Conclusão

O encaminhamento para destinos externos estava corretamente configurado.

---

# Validação do NAT Gateway

## Objetivo

Verificar a saída controlada para Internet.

## Teste

curl ifconfig.me

## Resultado obtido

Endereço IP público controlado pelo NAT Gateway.

## Interpretação

O egress da subnet da NVA estava sendo realizado pelo NAT Gateway.

## Conclusão

A saída para Internet foi centralizada.

> **Screenshot:** NAT Gateway.

---

# Validação do IP Forwarding

## Objetivo

Confirmar que a NVA estava apta a encaminhar tráfego.

## Configuração

sysctl net.ipv4.ip_forward

## Resultado esperado

net.ipv4.ip_forward = 1

## Resultado obtido

1

## Conclusão

O kernel Linux estava encaminhando pacotes entre interfaces lógicas.

---

# Validação do Hairpin Routing

## Cenário

A VM de teste foi inicialmente criada na mesma subnet da NVA.

## Sintoma

O tcpdump mostrou entrada e saída pela mesma interface.

## Diagnóstico

A NVA e a VM pertenciam ao mesmo domínio L2.

## Ação corretiva

A VM foi movida para:

10.100.11.0/24

## Resultado

O tráfego passou a ser roteado entre subnets distintas.

## Conclusão

A separação em domínios L3 eliminou o comportamento de Hairpin Routing.

---

# Resumo das validações

| Validação              | Status   |
| ---------------------- | -------- |
| Global VNet Peering    | Validado |
| Estado do Peering      | Validado |
| UDR para VNet Hub      | Validado |
| Effective Routes       | Validado |
| Trânsito via NVA       | Validado |
| Forced Tunneling       | Validado |
| VPN Site-to-Site       | Validado |
| Tabela de rotas da NVA | Validado |
| NAT Gateway            | Validado |
| IP Forwarding          | Validado |
| Hairpin Routing        | Validado |

---

# Conclusão

A arquitetura foi validada com sucesso em todos os seus componentes principais.

Os testes demonstraram que:

* o **Global VNet Peering fornece conectividade**;
* a **UDR controla explicitamente o caminho do tráfego**;
* a **NVA realiza o trânsito entre redes**;
* o **Forced Tunneling desvia o tráfego externo**;
* o **NAT Gateway centraliza o egress**;
* o **Azure VPN Gateway mantém a conectividade híbrida com a AWS**.

As validações por **Effective Routes** e **tcpdump** forneceram evidências concretas do comportamento do roteamento, permitindo compreender o funcionamento interno da arquitetura Hub-and-Spoke implementada neste laboratório.
