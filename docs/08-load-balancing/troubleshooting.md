# Troubleshooting — Lab 08

## Objetivo

Registrar os principais problemas encontrados durante a implementação do **Azure Virtual Machine Scale Set (VMSS) Flexible**, do **Azure Load Balancer Standard** e do **Azure Autoscale**, bem como as respectivas causas e soluções aplicadas.

---

# Problema 1 — Instância do VM Scale Set com Provisioning Failed

## Sintoma

O VM Scale Set apresentou estado **Unhealthy**, com uma das instâncias em estado **Provisioning Failed**.

O Azure Resource Health indicava:

* 1 de 2 instâncias indisponível.
* Estado do conjunto degradado.

### Evidência

[Inserir screenshot do Resource Health]

---

## Diagnóstico

A validação da instância revelou o seguinte erro:

```
ProvisioningState/failed/InvalidResourceReference
```

Mensagem principal:

```
Resource basicNsgvnet-mhcp-core-nic01 was not found
```

### Evidência

[Inserir screenshot do erro InvalidResourceReference]

---

## Causa

Após a criação do VM Scale Set, foi realizada uma alteração manual na configuração de rede.

O conjunto havia sido criado inicialmente utilizando outra subnet, o que levou à criação automática de um **NSG por NIC**.

Posteriormente:

* a subnet foi alterada;
* o NSG automático foi removido manualmente.

O **template do VM Scale Set** continuou referenciando o NSG excluído.

Como consequência, novas instâncias não conseguiam ser provisionadas corretamente.

---

## Solução

O VM Scale Set foi recriado utilizando:

* nova VNet;
* nova subnet;
* NSG aplicado na subnet;
* sem NSGs individuais nas NICs.

Essa abordagem eliminou as referências inválidas do template.

---

# Problema 2 — Inconsistência entre instâncias após Scale Out

## Sintoma

As instâncias criadas durante o Scale Out não possuíam o Nginx instalado.

### Evidência

[Inserir screenshot das instâncias sem Nginx]

---

## Diagnóstico

As instâncias iniciais foram configuradas manualmente após o provisionamento.

O VM Scale Set criou novas instâncias utilizando o **template original do conjunto**, que não continha a instalação do Nginx.

---

## Causa

O VM Scale Set **não replica o estado atual das instâncias existentes**.

Ele replica apenas o **template definido no conjunto**.

---

## Solução

A configuração das instâncias deve ser realizada através de mecanismos declarativos, como:

* Cloud-init;
* Custom Script Extension;
* Azure Image Builder;
* Terraform;
* Azure DevOps.

---

# Problema 3 — Limitação de quota regional

## Sintoma

A criação do VM Scale Set falhou com o erro:

```
Operation could not be completed as it results in exceeding approved Total Regional Cores quota
```

### Evidência

[Inserir screenshot do erro de quota]

---

## Diagnóstico

A assinatura possuía limite de **4 vCPUs na região Brazil South**.

O Scale Set tentava provisionar novas instâncias, excedendo a capacidade disponível.

---

## Causa

Limitação da quota regional da assinatura.

---

## Solução

O laboratório foi recriado em **East US 2**, utilizando:

* instâncias de 1 vCPU;
* nova VNet;
* novo Resource Group;
* ambiente totalmente isolado.

Essa abordagem eliminou as restrições de quota e aumentou a disponibilidade de tamanhos de VM.

---

# Problema 4 — Configuração de zona de disponibilidade

## Sintoma

Diversos tamanhos de VM não estavam disponíveis ao criar o VM Scale Set com **Availability Zones**.

### Evidência

[Inserir screenshot da indisponibilidade de tamanhos]

---

## Diagnóstico

A região apresentava disponibilidade limitada para determinados SKUs em configuração zonal.

---

## Causa

Restrições de capacidade da região para determinados tamanhos de VM.

---

## Solução

O VM Scale Set foi criado **sem redundância de zona**, mantendo o foco do laboratório em:

* elasticidade;
* escalabilidade;
* integração com Load Balancer;
* autoscaling.

---

# Problema 5 — Redimensionamento de instâncias individuais

## Sintoma

Tentativa de alterar o tamanho de uma instância individual do VM Scale Set.

O portal apresentou a mensagem:

```
Changes can be made only when the virtual machine is deallocated
```

### Evidência

[Inserir screenshot da mensagem do portal]

---

## Diagnóstico

O Azure exige **deallocation da VM** para alteração de tamanho individual.

---

## Causa

O redimensionamento vertical altera a alocação de hardware da máquina virtual.

---

## Solução

Foi identificado que o VM Scale Set Flexible permite alterar o **template do conjunto**, porém alterações em instâncias individuais continuam exigindo deallocation.

Como alternativa operacional, recomenda-se utilizar **escalabilidade horizontal**.

---

# Problema 6 — Instâncias heterogêneas

## Observação

Inicialmente foi considerado adicionar uma VM existente ao VM Scale Set.

Durante os testes foi identificado que isso **não é suportado**.

---

## Comportamento observado

No VM Scale Set Flexible:

* as instâncias são VMs independentes;
* porém devem **nascer como membros do conjunto**.

Não é possível anexar posteriormente uma VM criada fora do Scale Set.

---

## Conclusão

O Flexible permite:

* múltiplos tamanhos de VM;
* gerenciamento individual;
* orquestração centralizada.

Entretanto, o ciclo de vida das instâncias pertence ao próprio VM Scale Set.

---

# Problema 7 — Autoscaling baseado em métricas

## Objetivo

Validar o comportamento do Azure Autoscale.

---

## Comportamento observado

Durante o teste:

* geração de carga;
* Scale Out automático após aproximadamente 5 minutos;
* criação automática da instância;
* integração ao Load Balancer;
* Scale In automático após redução da carga.

---

## Resultado

O Azure executou corretamente as políticas configuradas, demonstrando funcionamento completo do ciclo de elasticidade automática.

### Evidência

[Inserir screenshot do histórico do Autoscale]

---

# Lições aprendidas

## O template é a fonte da verdade

Toda nova instância do VM Scale Set é criada a partir do **template do conjunto**, e não do estado atual das VMs existentes.

## NSGs devem ser aplicados na subnet

Evitar NSGs individuais por NIC reduz riscos de referências órfãs durante operações de escala.

## Escalabilidade horizontal é preferível

Para aplicações distribuídas, o VM Scale Set Flexible oferece expansão de capacidade com menor impacto operacional do que o redimensionamento vertical.

## Autoscale requer tempo de convergência

O Azure não reage instantaneamente às métricas.

O ciclo envolve:

* coleta de métricas;
* avaliação da política;
* provisionamento da instância;
* validação pelo Health Probe;
* integração ao Load Balancer.

Esse comportamento deve ser considerado durante o dimensionamento de aplicações críticas.

## Estado final

Após a recriação do ambiente em **East US 2**, todos os problemas foram resolvidos e o laboratório validou com sucesso:

* VM Scale Set Flexible;
* Load Balancer Standard;
* Scale Out manual;
* Scale In manual;
* Autoscaling baseado em métricas;
* Elasticidade horizontal automática.
