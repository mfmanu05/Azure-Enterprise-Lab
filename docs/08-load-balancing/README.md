# Lab 08 — Azure VM Scale Set Flexible and autoscaling

## Visão geral

Este laboratório teve como objetivo implementar uma arquitetura de alta disponibilidade e elasticidade utilizando **Azure Virtual Machine Scale Set (VMSS) com Orchestration Mode Flexible**, integrado a um **Azure Load Balancer Standard** e configurado com **autoscaling baseado em métricas**.

Diferentemente do modelo tradicional de Availability Set, o VMSS Flexible permite gerenciar instâncias como máquinas virtuais individuais, mantendo orquestração centralizada, escalabilidade automática e integração nativa com balanceamento de carga.

O ambiente foi criado de forma **isolada**, utilizando uma nova VNet, um novo Resource Group e uma arquitetura independente da topologia Hub-and-Spoke desenvolvida nos módulos anteriores.

## Objetivos do laboratório

* Implementar um **VM Scale Set Flexible**.
* Integrar o conjunto ao **Azure Load Balancer Standard**.
* Validar **balanceamento de carga** entre múltiplas instâncias.
* Executar **Scale Out e Scale In manualmente**.
* Configurar **autoscaling baseado em métricas**.
* Validar **Scale Out e Scale In automáticos**.
* Compreender o comportamento do **template do Scale Set** durante o provisionamento de novas instâncias.

## Arquitetura

A arquitetura utilizada foi composta pelos seguintes recursos:

* Resource Group dedicado ao laboratório.
* VNet isolada.
* Subnet App.
* NSG aplicado na subnet.
* NAT Gateway para saída à Internet.
* Azure Load Balancer Standard.
* VM Scale Set Flexible.
* Azure Bastion compartilhado através de peering com a VNet Hub.

Fluxo simplificado:

Internet → Public IP → Azure Load Balancer → VM Scale Set Flexible → Instâncias da aplicação

## Componentes implementados

### Rede

* VNet dedicada ao laboratório.
* Subnet App.
* NSG configurado para permitir HTTP.
* NAT Gateway configurado para saída controlada para a Internet.
* Peering com a VNet Hub para utilização do Azure Bastion.

### Compute

* Azure VM Scale Set.
* Orchestration Mode: **Flexible**.
* Instâncias Linux Ubuntu.
* Tamanho inicial: **1 vCPU** por instância.
* Capacidade inicial: **2 instâncias**.

### Balanceamento

* Azure Load Balancer Standard.
* Backend Pool integrado automaticamente ao VM Scale Set.
* Health Probe configurado na porta HTTP.

## Testes realizados

### Balanceamento de carga

Foi instalado **Nginx** nas duas instâncias iniciais, com páginas personalizadas identificando cada VM.

Validação realizada através de múltiplas requisições HTTP ao IP público do Load Balancer.

Resultado observado:

* Alternância entre **VMSS Instance 1** e **VMSS Instance 2**.
* Distribuição automática das conexões.
* Backend Pool atualizado corretamente.

### Scale Out manual

O conjunto foi expandido de **2 para 4 instâncias**.

Resultados:

* Provisionamento das novas instâncias em **menos de 1 minuto**.
* Associação automática ao Backend Pool.
* Health Probe validando automaticamente as novas instâncias.
* Balanceamento distribuído entre as quatro VMs.

### Consistência do template

Durante o Scale Out foi observado que as novas instâncias **não herdaram o Nginx instalado manualmente**.

Conclusão:

O VM Scale Set replica o **template do conjunto**, e não o estado atual das instâncias existentes.

Essa observação demonstra a importância de utilizar:

* Cloud-init.
* Custom Script Extension.
* Azure Image Builder.
* Terraform.
* Azure DevOps.

para garantir consistência entre instâncias.

### Scale In manual

O conjunto foi reduzido de **4 para 2 instâncias**.

Resultados:

* Remoção automática das instâncias.
* Atualização automática do Backend Pool.
* Remoção das instâncias em **menos de 1 minuto**.

Foi observado que o Azure removeu as instâncias mais antigas do conjunto, indicando a aplicação da política padrão de gerenciamento de instâncias.

### Autoscaling baseado em métricas

Foi configurado autoscaling utilizando métricas do Azure Monitor.

Configuração utilizada:

* Capacidade mínima: 2 instâncias.
* Capacidade máxima: 4 instâncias.
* Scale Out após utilização sustentada da métrica por aproximadamente 5 minutos.
* Scale In após normalização da carga.

Durante o teste foi gerada carga utilizando ferramenta de estresse na instância.

Resultados observados:

* **Scale Out automático**.
* Criação automática de nova instância.
* Integração automática ao Load Balancer.
* **Scale In automático** após redução da carga.

O comportamento ocorreu conforme configurado nas políticas do Azure Autoscale.

## Métricas observadas

| Operação                      | Resultado  |
| ----------------------------- | ---------- |
| Scale Out manual              | Sucesso    |
| Scale In manual               | Sucesso    |
| Scale Out automático          | Sucesso    |
| Scale In automático           | Sucesso    |
| Provisionamento de instâncias | < 1 minuto |
| Remoção de instâncias         | < 1 minuto |
| Atualização do Backend Pool   | Automática |
| Integração com Load Balancer  | Automática |

## Lições aprendidas

### VMSS Flexible

As instâncias são **VMs independentes**, porém **orquestradas pelo Scale Set**.

Não é possível adicionar uma VM existente ao conjunto; toda instância deve nascer como membro do VMSS.

### Template do Scale Set

O template é a **fonte da verdade**.

Alterações realizadas manualmente em uma instância não são replicadas para novas instâncias.

### Elasticidade horizontal

O Scale Out e Scale In mostraram-se extremamente rápidos, tornando o VMSS uma solução adequada para cargas variáveis.

### Elasticidade automática

O Azure Autoscale reagiu corretamente às métricas configuradas, demonstrando capacidade de adaptação automática da infraestrutura.

## Conclusão

O laboratório validou o ciclo completo de elasticidade do Azure VM Scale Set Flexible, incluindo:

* alta disponibilidade;
* balanceamento de carga;
* escalabilidade horizontal manual;
* escalabilidade horizontal automática;
* integração com Azure Monitor;
* integração automática com Azure Load Balancer.

A principal conclusão foi que o **VM Scale Set Flexible deve ser tratado como uma plataforma declarativa**, em que o template centralizado define o comportamento de todas as futuras instâncias do conjunto, reforçando a importância de automação e Infrastructure as Code em ambientes escaláveis.
