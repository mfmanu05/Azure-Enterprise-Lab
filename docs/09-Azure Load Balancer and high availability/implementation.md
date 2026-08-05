# Implementation — Lab 09

## Objetivo da implementação

Implementar uma arquitetura de **alta disponibilidade de aplicações** utilizando **Azure Load Balancer Standard**, validando distribuição de tráfego, Health Probe, failover automático e recuperação de instâncias.

O laboratório foi construído em ambiente totalmente isolado, utilizando uma nova VNet, um novo Resource Group e duas máquinas virtuais independentes executando **Nginx**.

---

## Arquitetura implementada

### Recursos criados

| Recurso                      | Finalidade                |
| ---------------------------- | ------------------------- |
| Resource Group               | Isolamento do laboratório |
| VNet dedicada                | Rede do módulo            |
| Subnet App                   | Hospedagem das VMs        |
| NSG                          | Controle de acesso        |
| NAT Gateway                  | Saída para Internet       |
| Azure Load Balancer Standard | Distribuição de tráfego   |
| Public IP Standard           | Exposição da aplicação    |
| Azure Bastion                | Administração das VMs     |

---

## Topologia

[Espaço para diagrama da arquitetura]

Fluxo simplificado:

Internet → Public IP → Azure Load Balancer → Backend Pool → VM APP 01 / VM APP 02

---

## Configuração da rede

### VNet

* Região: **East US 2**
* Espaço de endereçamento dedicado ao laboratório.

### Subnet

* **snet-app**
* Associada ao NSG da aplicação.
* Associada ao NAT Gateway.

### NSG

Regras configuradas:

* HTTP (80)
* HTTPS (443)
* SSH via Azure Bastion

[Espaço para evidência da configuração da VNet e NSG]

---

## Configuração das máquinas virtuais

Foram criadas duas máquinas virtuais Ubuntu:

* vm-app-09-01
* vm-app-09-02

Cada máquina recebeu instalação do **Nginx** com página personalizada para identificação durante os testes de balanceamento.

### Configuração do Nginx

```bash
sudo apt update
sudo apt install -y nginx

echo "<h1>Azure Enterprise Lab</h1><h2>VM APP 01</h2>" | sudo tee /var/www/html/index.html

sudo systemctl restart nginx
```

O mesmo procedimento foi executado na VM APP 02 com identificação correspondente.

[Espaço para evidência da instalação do Nginx]

---

## Configuração do Azure Load Balancer

Foi criado um **Azure Load Balancer Standard** utilizando:

### Frontend

* Public IP Standard

### Backend Pool

* vm-app-09-01
* vm-app-09-02

### Health Probe

* Protocolo: TCP
* Porta: 80
* Intervalo: 5 segundos

### Regra de balanceamento

* Frontend: Public IP
* Backend: Pool das VMs
* Porta: 80

[Espaço para evidência do Load Balancer]

---

## Validação do balanceamento

Foi realizado teste de múltiplas requisições HTTP para o IP público do Load Balancer.

Comando utilizado:

```bash
for i in {1..20}; do
  curl -s http://<IP_PUBLICO_DO_LB>
done
```

Resultado observado:

* distribuição automática entre VM APP 01 e VM APP 02.

[Espaço para evidência do teste de balanceamento]

---

## Teste de falha da aplicação

Foi interrompido o serviço **Nginx** na VM APP 01.

Comando utilizado:

```bash
sudo systemctl stop nginx
```

O Health Probe identificou a indisponibilidade da aplicação.

A VM foi automaticamente removida do Backend Pool.

Todas as conexões passaram a ser direcionadas para a VM APP 02.

[Espaço para evidência do Health Probe]

---

## Teste de recuperação

Após reinicialização do Nginx:

```bash
sudo systemctl start nginx
```

O Health Probe voltou a marcar a instância como **Healthy**.

A VM foi automaticamente reintegrada ao Backend Pool.

O balanceamento entre as duas VMs foi restabelecido.

[Espaço para evidência da recuperação]

---

## Teste de falha da infraestrutura

Foi realizado desligamento completo da VM APP 01.

Observações:

* Backend Health indicou a VM como **Down**.
* Backend Pool identificou a VM como **Stopped**.
* O tráfego permaneceu disponível através da VM APP 02.

Esse teste demonstrou a diferença entre:

* falha da aplicação;
* falha da infraestrutura.

[Espaço para evidência da VM desligada]

---

## Teste de indisponibilidade total

Foram desligadas simultaneamente:

* vm-app-09-01
* vm-app-09-02

Resultados observados:

* Backend Health sem instâncias saudáveis.
* Backend Pool indicando ambas as VMs como **Stopped**.
* Requisições HTTP retornando **timeout**.

Esse comportamento confirmou que o Azure Load Balancer depende da existência de **backends saudáveis** para entrega do serviço.

[Espaço para evidência do timeout]

---

## Tempos observados

| Evento                            | Tempo aproximado |
| --------------------------------- | ---------------- |
| Detecção de falha da aplicação    | ~1 minuto        |
| Remoção do backend                | ~1 minuto        |
| Recuperação após restart do Nginx | < 1 minuto       |
| Reintegração ao Backend Pool      | < 1 minuto       |

Esses tempos representam o ciclo completo de detecção e recuperação do Health Probe.

---

## Estado final da implementação

Ao final do laboratório o ambiente permaneceu com:

* duas VMs Ubuntu;
* Nginx operacional;
* Azure Load Balancer Standard funcional;
* Backend Pool saudável;
* Health Probe operacional;
* conectividade através do Azure Bastion;
* saída para Internet via NAT Gateway.

[Espaço para evidência do estado final do ambiente]
