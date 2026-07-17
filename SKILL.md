---
name: open-pr-azure
description: >
  Abre uma Pull Request no Azure DevOps com descrição formatada e vinculada ao Jira.
  Use SEMPRE que o usuário pedir para abrir PR no Azure, Azure DevOps, ADO, criar PR,
  "abrir pull request", "open PR azure", "abre PR no ADO", "manda pro azure", ou qualquer
  variação que mencione Azure DevOps ou quando o repositório remoto for do Azure DevOps
  (ex: dev.azure.com ou visualstudio.com). A skill verifica push pendente, gera
  a descrição no formato estabelecido e abre a PR via `az repos pr create`.
---

# Open PR — Azure DevOps + Jira

Workflow completo para abrir uma Pull Request no Azure DevOps com descrição formatada e linked ao Jira.

## Pré-requisitos

Verifique se as ferramentas necessárias estão disponíveis:

```bash
which az && az devops configure --list 2>/dev/null || echo "az CLI not configured"
```

Se `az` não estiver instalado:
- Informe o usuário e mostre: `brew install azure-cli && az extension add --name azure-devops`
- Para autenticar: `az login` e depois `az devops configure --defaults organization=https://dev.azure.com/<org>`

## Processo (execute sempre nesta ordem)

### 1. Verificar estado do repositório

```bash
git status --short
git log origin/$(git rev-parse --abbrev-ref HEAD)..HEAD --oneline 2>/dev/null || echo "branch not pushed"
```

Se houver arquivos não commitados, **avise o usuário** antes de continuar — não commite por conta própria a menos que seja explicitamente pedido.

Se a branch não existir no remoto ou houver commits locais não enviados, **confirme com o usuário** antes de fazer push:

```bash
git push -u origin $(git rev-parse --abbrev-ref HEAD)
```

### 2. Coletar contexto da branch

Rode estes comandos em paralelo:

```bash
# Commits desta branch vs master/main
git log master..HEAD --oneline 2>/dev/null || git log main..HEAD --oneline

# Diff completo (arquivos modificados)
git diff master...HEAD --stat 2>/dev/null || git diff main...HEAD --stat

# Diff detalhado dos arquivos principais
git diff master...HEAD -- src/ 2>/dev/null || git diff main...HEAD -- src/
```

Leia os arquivos mais relevantes das mudanças para entender o que foi implementado. Foque em:

- Novos arquivos criados
- Interfaces e tipos adicionados
- Mudanças em lógica de negócio
- Novos testes

### 3. Extrair o número JIRA

Extraia o ticket do Jira a partir do nome da branch. Padrões comuns:

- `feature/master/BEESKMM-1234-description` → `BEESKMM-1234`
- `feat/PROJ-5678-some-feature` → `PROJ-5678`
- `fix/ABC-999-bug-fix` → `ABC-999`

```bash
git rev-parse --abbrev-ref HEAD
```

Se o número JIRA **não estiver na branch**, pergunte ao usuário antes de continuar.

### 4. Verificar template de PR do repositório

Antes de gerar a descrição, verifique se o repositório tem um template de PR:

```bash
find . -maxdepth 3 \( -name "pull_request_template.md" -o -name "PULL_REQUEST_TEMPLATE.md" \) 2>/dev/null | head -5
ls .azuredevops/ 2>/dev/null
```

- Se encontrar um template, leia seu conteúdo — a descrição gerada deve **preencher as seções existentes do template**, não substituí-lo.
- Se não houver template, use o formato padrão descrito abaixo.

### 5. Gerar a descrição da PR

**Se houver template:** preencha cada seção do template com as informações coletadas. Não remova seções, não altere cabeçalhos — apenas complete o conteúdo.

**Se não houver template:** use o formato padrão abaixo. Preencha apenas a seção `📝 Description` com bullets concisos em inglês sobre as mudanças:

```markdown
## 📝 Description

[<JIRA-ID>](https://ab-inbev.atlassian.net/browse/<JIRA-ID>)

- <concise bullet describing the main change>
- <concise bullet describing another change>
- <add as many bullets as needed to cover the main code changes>

## 📱 Visual Changes

Please add here a screenshot if your PR has visual changes
```

**Regras:**
- Preencha os bullets da seção `📝 Description` com as principais mudanças do código — quantos forem necessários para cobrir o escopo do PR
- Para PRs simples, seja direto e enxuto; para PRs complexos, detalhe o suficiente para que o revisor entenda o contexto e a motivação das mudanças
- Todo conteúdo em inglês
- Não altere nem remova nenhuma das demais seções ou checkboxes
- Use `inline code` para nomes de arquivos ou métodos quando ajudar a clareza

### 6. Derivar o título da PR

O título segue o padrão **Conventional Commits + JIRA**:

```
<tipo>[<JIRA-ID>] <Breve descrição no imperativo com primeira letra maiúscula>
```

**Como extrair os componentes:**
- **Tipo**: derive dos commits da branch — `feat` para nova funcionalidade, `fix` para correção, `chore` para tarefas de manutenção, `refactor`, `test`, `docs`, `perf`, `ci`
- **Número JIRA**: extraído no passo 3, entre colchetes `[]`
- **Descrição**: curta, imperativo, sem ponto final, em inglês, com a primeira letra em maiúscula

**Exemplos:**
```
feat[BEESKMM-1234] Implement MemberHub use case
fix[BEESKMM-999] Prevent null pointer on enrollment mapper
chore[BEESKMM-777] Upgrade Kotlin coroutines version
```

### 7. Determinar a branch base

```bash
# Verifica qual é a branch base padrão do repositório
git remote show origin | grep 'HEAD branch'
```

O padrão é `master`. Use `main` se o repositório usar. Se houver dúvida, confirme com o usuário.

### 8. Abrir a PR no Azure DevOps

Use `az repos pr create`. **Não crie nenhum arquivo local** — passe o conteúdo inline:

```bash
az repos pr create \
  --title "<tipo>[<JIRA-ID>] <Descrição>" \
  --description "<conteúdo gerado na Etapa 4>" \
  --source-branch "$(git rev-parse --abbrev-ref HEAD)" \
  --target-branch "master" \
  --open
```

Se o usuário quiser revisar antes de abrir, mostre o título e a descrição e aguarde confirmação.

Após criar a PR, exiba a URL retornada pelo comando.

## Notas

- **Base branch padrão**: `master` — mude para `main` se o repositório usar
- **Draft PR**: se o usuário mencionar "draft" ou "rascunho", adicione `--draft` ao comando
- **Reviewer**: se o usuário mencionar alguém para revisar, adicione `--reviewers <email-ou-alias>`
- **Organização/Projeto ADO**: se `az devops configure` não tiver defaults, adicione `--org https://dev.azure.com/<org> --project <project>` ao comando
- Se `az` não estiver instalado: `brew install azure-cli && az extension add --name azure-devops`
- Para autenticar: `az login` e `az devops configure --defaults organization=https://dev.azure.com/<org>`
