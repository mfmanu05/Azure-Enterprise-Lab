# Validation — Lab 09

## Objetivo da validação

Validar o funcionamento do **Azure Load Balancer Standard** em cenários de alta disponibilidade, confirmando distribuição de tráfego, detecção automática de falhas, remoção de instâncias do Backend Pool e recuperação automática após restabelecimento do serviço.

---

# Teste 1 — Balanceamento de carga

## Objetivo

Validar que o Azure Load Balancer distribui automaticamente as conexões entre as duas máquinas virtuais.

## Procedimento

Foram realizadas múltiplas requisições HTTP para o IP público do Load Balancer.

```bash
for i in {1..20}; do
  curl -s http://<IP_PUBLICO_DO_LB>
done
```

## Resultado esperado

As respostas devem alternar entre:

* VM APP 01
* VM APP 02

## Resultado obtido

O Load Balancer distribuiu corretamente as conexões entre as duas instâncias do Backend Pool.

**Status:** Validado com sucesso

### Evidência

[Inserir screenshot do teste de balanceamento]

---

# Teste 2 — Falha da aplicação (Nginx parado)

## Objetivo

Validar a detecção automática de indisponibilidade da aplicação.

## Procedimento

Foi interrompido o serviço Nginx na VM APP 01.

```bash
sudo systemctl stop nginx
```

## Resultado esperado

O Health Probe deve marcar a instância como **Down** e removê-la automaticamente do Backend Pool.

## Resultado obtido

O Azure identificou corretamente a falha da aplicação.

A VM APP 01 foi marcada como **Down**.

O Backend Pool passou a encaminhar todas as conexões para a VM APP 02.

**Status:** Validado com sucesso

### Evidência

[Inserir screenshot do Backend Health]

---

# Teste 3 — Continuidade do serviço

## Objetivo

Verificar que a aplicação permanece disponível após falha de uma instância.

## Procedimento

Após interrupção do Nginx na VM APP 01, foram executadas múltiplas requisições HTTP.

```bash
for i in {1..10}; do
  curl -s http://<IP_PUBLICO_DO_LB>
done
```

## Resultado esperado

Todas as respostas devem ser atendidas pela VM APP 02.

## Resultado obtido

O Load Balancer direcionou **100% do tráfego para a VM APP 02**.

Não houve indisponibilidade da aplicação.

**Status:** Validado com sucesso

### Evidência

[Inserir screenshot do curl respondendo apenas pela VM APP 02]

---

# Teste 4 — Recuperação da aplicação

## Objetivo

Validar a reintegração automática da instância ao Backend Pool.

## Procedimento

Foi reiniciado o serviço Nginx na VM APP 01.

```bash
sudo systemctl start nginx
```

## Resultado esperado

O Health Probe deve marcar a instância como **Healthy** e reintegrá-la automaticamente ao balanceamento.

## Resultado obtido

A VM APP 01 voltou a responder ao Health Probe.

O Backend Pool foi atualizado automaticamente.

O balanceamento entre as duas instâncias foi restabelecido.

**Status:** Validado com sucesso

### Evidência

[Inserir screenshot do Backend Health recuperado]

---

# Teste 5 — Falha da infraestrutura (VM desligada)

## Objetivo

Comparar o comportamento do Load Balancer durante falha da infraestrutura.

## Procedimento

Foi desligada completamente a VM APP 01.

## Resultado esperado

O Health Probe deve marcar a instância como **Down** e o Backend Pool deve indicar a VM como parada.

## Resultado obtido

O comportamento foi semelhante ao da falha da aplicação.

Diferença observada:

* Backend Health mostrou a instância como **Down**.
* Backend Pool informou explicitamente que a VM estava **Stopped**.

O tráfego permaneceu disponível através da VM APP 02.

**Status:** Validado com sucesso

### Evidência

[Inserir screenshot da VM desligada]

---

# Teste 6 — Indisponibilidade total

## Objetivo

Validar o comportamento do Azure Load Balancer quando não existem backends saudáveis.

## Procedimento

Foram desligadas simultaneamente:

* vm-app-09-01
* vm-app-09-02

Foi realizado acesso HTTP ao IP público do Load Balancer.

## Resultado esperado

O Frontend IP permanece ativo, porém sem capacidade de encaminhar conexões.

## Resultado obtido

O Backend Health não apresentou instâncias saudáveis.

O Backend Pool indicou ambas as VMs como **Stopped**.

As requisições HTTP retornaram **timeout**.

**Status:** Validado com sucesso

### Evidência

[Inserir screenshot do timeout]

---

# Teste 7 — Tempo de convergência do Health Probe

## Objetivo

Medir o tempo de detecção e recuperação do Azure Load Balancer.

## Resultados observados

| Evento                            | Tempo observado |
| --------------------------------- | --------------- |
| Detecção da falha                 | ~1 minuto       |
| Remoção do backend                | ~1 minuto       |
| Recuperação após restart do Nginx | < 1 minuto      |
| Reintegração ao Backend Pool      | < 1 minuto      |

Esses tempos demonstram o ciclo completo de monitoramento do Health Probe.

### Evidência

[Inserir screenshot do histórico do Health Probe]

---

# Matriz de comportamento

| Cenário             | Backend Health         | Resultado para o cliente |
| ------------------- | ---------------------- | ------------------------ |
| Nginx parado        | Down                   | Serviço disponível       |
| VM desligada        | Down                   | Serviço disponível       |
| Duas VMs desligadas | Sem backends saudáveis | Timeout                  |

---

# Resultado geral

Todos os testes previstos para o laboratório foram executados com sucesso.

O ambiente validou:

* distribuição de tráfego;
* alta disponibilidade local;
* detecção automática de falhas;
* failover automático;
* recuperação automática de instâncias;
* comportamento do Azure Load Balancer durante falha de aplicação;
* comportamento durante falha de infraestrutura;
* comportamento durante indisponibilidade total do Backend Pool.

A principal conclusão foi que o **Azure Load Balancer Standard utiliza Health Probes para monitorar continuamente as cargas da aplicação**, removendo automaticamente instâncias indisponíveis e mantendo a continuidade do serviço sempre que existir pelo menos um backend saudável.

**Status geral do laboratório:** Aprovado
