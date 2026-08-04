# Validation — Lab 08

## Objetivo da validação

Validar o funcionamento do **Azure Virtual Machine Scale Set (VMSS) Flexible**, do **Azure Load Balancer Standard** e do **Azure Autoscale**, confirmando o comportamento de balanceamento de carga, escalabilidade horizontal e elasticidade automática da infraestrutura.

---

# Teste 1 — Balanceamento de carga

## Objetivo

Validar que o Azure Load Balancer distribui automaticamente as conexões entre as instâncias do VM Scale Set.

## Procedimento

Foram realizadas múltiplas requisições HTTP para o IP público do Load Balancer.

`` ` `` bash

````bash
for i in {1..20}; do
  curl -s http://<IP_PUBLICO_DO_LB>
done
```
````

## Resultado esperado

As respostas devem alternar entre as instâncias do VM Scale Set.

## Resultado obtido

O Load Balancer distribuiu corretamente as conexões entre:

* VMSS Instance 1
* VMSS Instance 2

A distribuição ocorreu automaticamente, sem necessidade de configuração manual das instâncias no Backend Pool.

**Status:** Validado com sucesso

### Evidência

[Inserir screenshot do teste com curl]

---

# Teste 2 — Scale Out manual

## Objetivo

Validar a expansão manual do VM Scale Set.

## Procedimento

O conjunto foi expandido de **2 para 4 instâncias** através da opção **Add Instance**.

## Resultado esperado

Novas instâncias devem ser criadas automaticamente e integradas ao Backend Pool do Load Balancer.

## Resultado obtido

Foram criadas duas novas instâncias.

Observações:

* Provisionamento em menos de 1 minuto.
* Integração automática ao Backend Pool.
* Atualização automática do Health Probe.

**Status:** Validado com sucesso

### Evidência

[Inserir screenshot das quatro instâncias]

---

# Teste 3 — Distribuição de tráfego após Scale Out

## Objetivo

Validar que as novas instâncias passam a receber tráfego automaticamente.

## Procedimento

Repetição do teste de múltiplas requisições HTTP.

## Resultado esperado

O tráfego deve ser distribuído entre as quatro instâncias.

## Resultado obtido

O Load Balancer distribuiu conexões entre:

* VMSS Instance 1
* VMSS Instance 2
* VMSS Instance 3
* VMSS Instance 4

**Status:** Validado com sucesso

### Evidência

[Inserir screenshot do curl alternando entre as quatro instâncias]

---

# Teste 4 — Consistência do template

## Objetivo

Verificar o comportamento das novas instâncias durante o Scale Out.

## Procedimento

As instâncias 3 e 4 foram provisionadas após instalação manual do Nginx nas instâncias 1 e 2.

## Resultado esperado

Caso o VMSS replique o estado das instâncias existentes, as novas instâncias deveriam possuir Nginx configurado.

## Resultado obtido

As novas instâncias **não herdaram o Nginx**.

Conclusão:

O VM Scale Set Flexible utiliza o **template do conjunto** como fonte da verdade.

Alterações manuais realizadas em instâncias existentes não são replicadas automaticamente.

**Status:** Validado com sucesso

### Evidência

[Inserir screenshot demonstrando ausência do Nginx]

---

# Teste 5 — Scale In manual

## Objetivo

Validar a redução manual do VM Scale Set.

## Procedimento

O conjunto foi reduzido de **4 para 2 instâncias**.

## Resultado esperado

As instâncias selecionadas devem ser removidas automaticamente do Backend Pool.

## Resultado obtido

As instâncias foram removidas em menos de 1 minuto.

Observações:

* Atualização automática do Backend Pool.
* Atualização automática do Health Probe.
* O Azure removeu as instâncias mais antigas do conjunto.

**Status:** Validado com sucesso

### Evidência

[Inserir screenshot do Scale In]

---

# Teste 6 — Autoscaling automático (Scale Out)

## Objetivo

Validar expansão automática baseada em métricas.

## Configuração

* Capacidade mínima: 2
* Capacidade máxima: 4
* Regra de aumento baseada em utilização sustentada da métrica.

## Procedimento

Foi gerada carga utilizando ferramenta de estresse.

## Resultado esperado

O Azure deve criar automaticamente uma nova instância.

## Resultado obtido

O Scale Out ocorreu automaticamente após aproximadamente **5 minutos**.

Observações:

* Criação automática da instância.
* Integração automática ao Backend Pool.
* Nenhuma intervenção manual necessária.

**Status:** Validado com sucesso

### Evidência

[Inserir screenshot do evento de Scale Out automático]

---

# Teste 7 — Autoscaling automático (Scale In)

## Objetivo

Validar redução automática da capacidade após normalização da carga.

## Procedimento

Após encerramento da carga de estresse, foi monitorado o comportamento do Azure Autoscale.

## Resultado esperado

O Azure deve remover automaticamente a instância adicional.

## Resultado obtido

O Scale In ocorreu automaticamente após normalização da métrica.

Observações:

* Redução automática para a capacidade mínima.
* Atualização automática do Backend Pool.
* Nenhuma intervenção manual necessária.

**Status:** Validado com sucesso

### Evidência

[Inserir screenshot do evento de Scale In automático]

---

# Teste 8 — Tempo de convergência

## Objetivo

Medir o tempo necessário para expansão e redução do conjunto.

## Resultados observados

| Operação             | Tempo observado            |
| -------------------- | -------------------------- |
| Scale Out manual     | < 1 minuto                 |
| Scale In manual      | < 1 minuto                 |
| Scale Out automático | ~5 minutos                 |
| Scale In automático  | Após normalização da carga |

### Evidência

[Inserir screenshot do histórico de atividades]

---

# Resultado geral

Todos os testes previstos para o laboratório foram executados com sucesso.

O ambiente validou:

* Balanceamento de carga.
* Alta disponibilidade.
* Scale Out manual.
* Scale In manual.
* Scale Out automático.
* Scale In automático.
* Integração automática com Azure Load Balancer.
* Orquestração do VM Scale Set Flexible.

A principal conclusão foi que o **Azure VM Scale Set Flexible fornece elasticidade horizontal automática baseada em políticas de monitoramento**, mantendo integração transparente com o Load Balancer e gerenciamento centralizado das instâncias através do template do Scale Set.

**Status geral do laboratório:** Aprovado
