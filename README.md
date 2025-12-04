# ⚠️ Action Movida e Renomeada

> [!CAUTION]
> **Este repositório (`Malnati/flow-check`) foi descontinuado.**
>
> A lógica de validação foi refatorada para aderir ao Princípio de Responsabilidade Única e agora vive em um novo repositório com melhor performance e flexibilidade.

## 🚀 Novo Endereço

Por favor, atualize seus workflows para utilizar a nova Action:

### 👉 [**Malnati/branch-flow-guard**](https://github.com/Malnati/branch-flow-guard)

---

## 🛠️ Guia de Migração Rápida

A nova arquitetura separa a **Lógica** da **Notificação**.

### ❌ Como era (Antigo)
```yaml
- uses: Malnati/flow-check@v2
  with:
    token: ${{ secrets.GITHUB_TOKEN }}
````

### ✅ Como é agora (Novo)

Você deve usar a nova action de análise combinada com a action de comentário:

```yaml
# 1. Analisa o fluxo
- uses: Malnati/branch-flow-guard@v1
  id: flow
  with:
    token: ${{ secrets.GITHUB_TOKEN }}

# 2. Comenta o resultado (Sticky Mode)
- uses: Malnati/pr-comment@v6
  with:
    token: ${{ secrets.GITHUB_TOKEN }}
    pr_number: ${{ github.event.pull_request.number }}
    # ... inputs de configuração do template
```

Para documentação completa e exemplos, visite o [novo repositório](https://github.com/Malnati/branch-flow-guard).

-----
> [!CAUTION]
> Esta versão antiga não receberá mais atualizações.

