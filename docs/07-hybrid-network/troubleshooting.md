# Troubleshooting — Hybrid network

## Visão geral

Durante a implementação da arquitetura Hub-and-Spoke foram encontrados diversos problemas relacionados a **roteamento, conectividade entre VNets, comportamento da NVA e interpretação do tráfego**.

Este documento registra os incidentes observados, o processo de diagnóstico e as soluções aplicadas.

O objetivo não é apenas documentar erros, mas registrar **o raciocínio utilizado para entender o comportamento da arquitetura**.

---

# Problema 1 — Comunicação direta via Peering

## Sintoma

Após a criação do Global VNet Peering, as VMs conseguiam se comunicar diretamente entre a VNet Core e a VNet Hub.

## Hipótese inicial

A comunicação estava ocorrendo sem utilizar a NVA.

## Diagnóstico

A análise das **Effective Routes** mostrou que o Azure havia criado automaticamente rotas do tipo **Virtual Network Peering**.

O tráfego seguia diretamente pelo backbone do Azure.

## Solução

Foi criada uma **User Defined Route (UDR)** apontando para a NVA.

Prefixo:

10.100.0.0/16

Next Hop:

Virtual Appliance

10.100.10.4

## Aprendizado

> **O peering cria conectividade, mas não controla o caminho do tráfego.**

---

# Problema 2 — UDR sobrescrevendo o Peering

## Sintoma

Após a criação da UDR, a comunicação entre as VNets deixou de funcionar.

## Hipótese inicial

Problema de NSG.

## Diagnóstico

As **Effective Routes** mostraram que a rota da UDR passou a ter prioridade sobre a rota do peering.

O tráfego estava sendo enviado para a NVA.

A NVA ainda não estava configurada para encaminhar pacotes.

## Solução

Foi habilitado **IP Forwarding**:

### Azure

Enable IP Forwarding na NIC.

### Linux

sudo sysctl -w net.ipv4.ip_forward=1

## Aprendizado

As UDRs têm prioridade sobre as rotas do sistema.

---

# Problema 3 — Hairpin Routing

## Cenário

A VM de teste foi criada na mesma subnet da NVA.

Topologia:

Core

↓

NVA (10.100.10.4)

↓

VM Teste (10.100.10.5)

## Sintoma

O tcpdump mostrava entrada e saída pela mesma interface.

Exemplo:

In  IP 10.10.1.4 > 10.100.10.5

Out IP 10.10.1.4 > 10.100.10.5

## Diagnóstico

A NVA e a VM pertenciam ao mesmo **domínio L2**.

O roteador precisava receber e retransmitir o pacote pela mesma interface.

Esse comportamento é conhecido como **Hairpin Routing (U-Turn Routing)**.

## Solução

A VM foi movida para uma subnet diferente:

10.100.11.0/24

## Aprendizado

Uma NVA deve permanecer em uma **subnet dedicada**.

---

# Problema 4 — Interpretação do tcpdump

## Sintoma

Mesmo após mover a VM para outra subnet, o tcpdump mostrava:

In  IP 10.10.1.4 > 10.100.11.4

Out IP 10.10.1.4 > 10.100.11.4

## Hipótese inicial

A NVA não estava roteando.

## Diagnóstico

O tcpdump estava capturando **antes da tradução de endereços (NAT)**.

A interface observava:

* pacote entrando;
* pacote sendo encaminhado.

Como a NVA estava apenas roteando, o endereço IP de origem permanecia preservado.

## Conclusão

O comportamento era esperado.

## Aprendizado

O tcpdump na NVA mostra o tráfego **antes do NAT Gateway**.

---

# Problema 5 — NAT Gateway e ICMP

## Sintoma

Após implementar Forced Tunneling, o ping para 8.8.8.8 falhava.

## Hipótese inicial

O NAT Gateway não estava funcionando.

## Diagnóstico

O tcpdump mostrou que a NVA encaminhava os pacotes ICMP.

O NAT Gateway aplica **SNAT para conexões de saída**, principalmente TCP e UDP.

O teste de ICMP não era a melhor forma de validar o egress.

## Solução

A validação foi realizada utilizando:

curl https://ifconfig.me

## Aprendizado

O NAT Gateway deve ser validado preferencialmente com tráfego **TCP/HTTPS**.

---

# Problema 6 — Bastion inacessível

## Sintoma

Após alterar as rotas do Hub, o acesso ao Azure Bastion deixou de funcionar.

## Diagnóstico

Uma UDR havia sido aplicada de forma ampla na VNet Hub.

O tráfego do Bastion passou a ser desviado para a NVA.

## Solução

Foi ajustado o escopo das UDRs, mantendo o Bastion fora do caminho de trânsito.

## Aprendizado

Nem todo tráfego deve ser forçado pela NVA.

---

# Problema 7 — Atualização de pacotes na NVA

## Sintoma

A NVA não conseguia executar:

sudo apt update

Erro:

Could not connect to azure.archive.ubuntu.com

## Hipótese inicial

Problema de DNS.

## Diagnóstico

A subnet não possuía saída para Internet.

## Solução

Foi associado um **NAT Gateway** à subnet da NVA.

## Resultado

A atualização de pacotes passou a funcionar normalmente.

## Aprendizado

A NVA também precisa de um caminho válido para egress.

---

# Problema 8 — Effective Routes como ferramenta principal

## Observação

Diversos problemas inicialmente atribuídos a NSG ou VPN eram, na realidade, problemas de roteamento.

A consulta às **Effective Routes** permitiu identificar:

* precedência das UDRs;
* rotas do peering;
* rotas do sistema;
* próximo salto efetivo.

## Aprendizado

As Effective Routes foram a principal ferramenta de diagnóstico do Azure durante este módulo.

---

# Lições aprendidas

## 1. Peering não substitui roteamento

O peering fornece conectividade.

As UDRs definem o caminho.

---

## 2. A NVA precisa encaminhar

IP Forwarding é obrigatório.

---

## 3. Subnets importam

A posição da NVA na topologia influencia diretamente o comportamento do roteamento.

---

## 4. O tcpdump exige interpretação

Entrada e saída na mesma interface não significam necessariamente erro.

---

## 5. NAT e roteamento são funções diferentes

A NVA roteia.

O NAT Gateway traduz endereços.

---

## 6. Effective Routes deve ser sempre consultado

Ela mostra o comportamento real do Azure.

---

# Resumo dos incidentes

| Problema                        | Causa                         | Solução               |
| ------------------------------- | ----------------------------- | --------------------- |
| Tráfego ignorando a NVA         | rota do peering               | criar UDR             |
| Comunicação interrompida        | IP Forwarding desabilitado    | habilitar forwarding  |
| Hairpin Routing                 | NVA e destino na mesma subnet | subnet dedicada       |
| tcpdump aparentemente duplicado | captura pré-NAT               | interpretação correta |
| Ping para Internet falhando     | teste ICMP inadequado         | validar com HTTPS     |
| Bastion inacessível             | UDR aplicada ao Hub           | ajustar escopo        |
| apt update falhando             | ausência de egress            | NAT Gateway           |
| Diagnóstico complexo            | foco apenas em NSG            | usar Effective Routes |

---

# Conclusão

O principal aprendizado deste módulo foi compreender que **a maioria dos problemas de Azure Networking é resolvida entendendo o caminho do tráfego**.

A combinação entre **Effective Routes**, **tcpdump** e análise da topologia permitiu identificar com precisão o comportamento da arquitetura, reforçando o princípio que guiou todo o laboratório:

> **Entender antes de decorar.**
