<div align="center">

# 🛡️ Branch Flow Guard

[![GitHub Marketplace](https://img.shields.io/badge/Marketplace-Branch%20Flow%20Guard-6f42c1?style=for-the-badge&logo=github)](https://github.com/marketplace/actions/branch-flow-guard)
[![Version](https://img.shields.io/github/v/release/Malnati/flow-check?style=for-the-badge&color=purple)](https://github.com/Malnati/flow-check/releases)
[![License](https://img.shields.io/github/license/Malnati/flow-check?style=for-the-badge&color=blue)](LICENSE)

**Imponha regras estritas de Git Flow e valide o conteúdo de PRs automaticamente.**

<p align="center">
  <a href="#-sobre">Sobre</a> •
  <a href="#-regras-do-fluxo">Regras</a> •
  <a href="#-instalação">Instalação</a> •
  <a href="#-customizando-a-mensagem">Customização</a> •
  <a href="#-inputs--outputs">Inputs & Outputs</a>
</p>

</div>

---

## 🚀 Sobre

O **Branch Flow Guard** é um "porteiro" para seus Pull Requests. Ele analisa a origem e o destino de cada PR para garantir que o ciclo de vida do software seja respeitado (Dev → Staging → Prod).

Além disso, ele detecta inteligentemente se há alterações reais de código ou apenas documentação, evitando bloqueios desnecessários em tarefas administrativas.

### ✨ O que ele faz
1.  **Validação de Fluxo:** Bloqueia merges diretos de `dev` para `production` ou features direto para `staging`.
2.  **Smart Diff:** Ignora validações estritas se a mudança for apenas em arquivos de documentação (ex: `README.md`).
3.  **Feedback Visual:** Posta um comentário claro no PR (estilo Dashboard) explicando o status.
4.  **Integração:** Expõe `outputs` para você encadear outras actions (como auto-sync ou deploys).

---

## 🚦 Regras do Fluxo

Esta Action impõe a seguinte esteira de promoção:

```mermaid
graph LR
    DEV[Development] -->|✅ Permitido| STG[Staging]
    STG[Staging] -->|✅ Permitido| MAIN[Production]
    
    FEAT[Feature/*] -.->|🚫 Bloqueado| MAIN
    DEV -.->|🚫 Bloqueado| MAIN
````

  * **Production** (main/master) só aceita merges vindos de **Staging**.
  * **Staging** (homolog/release) só aceita merges vindos de **Development**.
  * Qualquer outra combinação gera um alerta de bloqueio.

-----

## 📦 Instalação

### Permissões Necessárias

Como esta action posta comentários no PR, você precisa conceder permissão de escrita.

```yaml
permissions:
  pull-requests: write
  contents: read
```

### Exemplo de Workflow

```yaml
name: "Governance Check"

on:
  pull_request:
    types: [opened, synchronize, reopened]

permissions:
  contents: read        # Necessário para o checkout
  pull-requests: write  # Necessário para comentar na PR

jobs:
  flow-guard:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout Repository
        uses: actions/checkout@v4

      - name: Validate Branch Flow
        uses: Malnati/flow-check@v2.0.0
        with:
          token: ${{ secrets.GITHUB_TOKEN }}
```

-----

## 🎨 Customizando a Mensagem

Por padrão, esta action usa um template visual de "Dashboard". Se você quiser usar seu próprio layout Markdown, basta criar um arquivo no seu repositório e referenciá-lo.

**1. Crie o arquivo `.github/templates/flow-msg.md`**
Você pode usar as variáveis padrão (`$ACTOR`, `$SUBJECT`, etc.).

**2. Aponte no Workflow:**

```yaml
      - name: Validate Branch Flow
        uses: Malnati/flow-check@v2.0.0
        with:
          token: ${{ secrets.GITHUB_TOKEN }}
          custom_template: ".github/workflows/flow-check.md"
```

### 🧩 Mapeamento de Variáveis

Veja o que o `flow-check` preenche automaticamente em cada variável do template:

| Variável | Conteúdo Preenchido pelo Flow Guard |
| :--- | :--- |
| `$TITLE` | "🔀 Branch Flow Guard" |
| `$SUBJECT` | Visualização do fluxo com seta (ex: `feature/login → develop`) |
| `$BODY_MESSAGE` | A mensagem de status principal (ex: "✅ Autorizado..." ou "⛔ Bloqueado..."). |
| `$BODY_SCOPE_BLOCK` | Lista HTML contendo detalhes das branches e, em caso de erro, o motivo da violação. |
| `$FOOTER_BLOCK` | Resumo do resultado ("Permitido", "Negado" ou "Skipped"). |

-----

## ⛓️ Exemplo Avançado (Job Chaining)

Use os **Outputs** para controlar a execução de jobs subsequentes (ex: só rodar testes pesados se o fluxo for válido).

```yaml
jobs:
  governance:
    runs-on: ubuntu-latest
    outputs:
      allowed: ${{ steps.guard.outputs.allowed }}
      has_code: ${{ steps.guard.outputs.has_code }}
    steps:
      - name: Run Guard
        id: guard
        uses: Malnati/flow-check@v1.0.0
        with:
          token: ${{ secrets.GITHUB_TOKEN }}

  heavy-tests:
    needs: governance
    # Só roda se permitido E se tiver código (ignora docs)
    if: ${{ needs.governance.outputs.allowed == 'true' && needs.governance.outputs.has_code == 'true' }}
    runs-on: ubuntu-latest
    steps:
      - run: echo "Rodando testes..."
```

-----

## ⚙️ Inputs & Outputs

### Inputs

| Input | Obrigatório | Padrão | Descrição |
| :--- | :---: | :---: | :--- |
| `token` | **Sim** | - | Token do GitHub (`secrets.GITHUB_TOKEN`) para ler diffs e postar comentários. |
| `custom_template` | Não | `""` | Caminho relativo para um arquivo Markdown caso queira substituir o layout padrão. |

### Outputs

| Output | Tipo | Descrição |
| :--- | :---: | :--- |
| `allowed` | `true/false` | Define se o fluxo de branches respeita as regras. |
| `has_code` | `true/false` | Define se há alterações em arquivos de código (ignora docs). |
| `head_branch` | String | Nome da branch de origem (ex: `feature/login`). |
| `base_branch` | String | Nome da branch de destino (ex: `develop`). |

-----

<div align="center">

<sub>Security & Governance by <a href="https://github.com/Malnati">Ricardo Malnati</a>.</sub>

</div>
