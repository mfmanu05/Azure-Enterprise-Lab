# Troubleshooting — Lab 09

## Objetivo

Registrar os principais comportamentos observados durante os testes do **Azure Load Balancer Standard**, bem como os diagnósticos realizados e as conclusões obtidas durante os cenários de falha e recuperação.

---

# Problema 1 — Instância removida do balanceamento após parada do Nginx

## Sintoma

Após interromper o serviço **Nginx** na VM APP 01, o Load Balancer passou a encaminhar todas as conexões apenas para a VM APP 02.

O teste com `curl` retornava exclusivamente respostas da segunda instância.

### Evidência

[Inserir screenshot do curl respondendo apenas pela VM APP 02]

---

## Diagnóstico

O **Health Probe** deixou de receber respostas válidas na porta 80.

A instância foi marcada como **Down** e removida automaticamente do Backend Pool.

---

## Causa

Falha da aplicação (Nginx indisponível).

A infraestrutura permaneceu operacional, porém o serviço monitorado pelo Health Probe tornou-se indisponível.

---

## Solução

Reinicialização do Nginx.

```bash id=
```
