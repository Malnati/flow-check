# flow-check

Fluxo reutilizável de validação de Pull Requests para GitHub Actions.

Este workflow verifica:

- Se a PR contém alterações de código
- Se o fluxo de branches está de acordo com a promoção dev → staging → main

Expondo as saídas:

- `has_code`: `"true"` ou `"false"`
- `allowed`: `"true"` ou `"false"`
- `head_branch`: nome da branch de origem
- `base_branch`: nome da branch de destino

## Uso

No repositório que contém as PRs:

```yaml
name: "PR flow"

on:
  pull_request:
    types: [opened, synchronize, reopened]

jobs:
  flow-check:
    uses: SEU_USUARIO/flow-check/.github/workflows/flow-check.yml@v1

  auto-sync:
    needs: flow-check
    if: ${{ needs.flow-check.outputs.has_code == 'true' && needs.flow-check.outputs.allowed == 'true' }}
    uses: SEU_USUARIO/auto-sync/.github/workflows/auto-sync.yml@v1
    with:
      base_branch: ${{ needs.flow-check.outputs.base_branch }}
      head_branch: ${{ needs.flow-check.outputs.head_branch }}
      pr_number: ${{ github.event.pull_request.number }}
      changed_files: ${{ github.event.pull_request.changed_files }}
