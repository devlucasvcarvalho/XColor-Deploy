# XColor-Deploy

Repositório central de deploy do projeto **XColor**.  
Ele agrega o código do frontend ([XColor-front](https://github.com/devlucasvcarvalho/XColor-front)) e do backend ([XColor-back](https://github.com/devlucasvcarvalho/XColor-back)) e mantém as três branches de ambiente sincronizadas via GitHub Actions.

---

## Estrutura de branches

| Branch | Descrição |
|--------|-----------|
| `dev` | Ambiente de desenvolvimento |
| `test` | Ambiente de homologação/testes |
| `production` | Ambiente de produção |

Dentro de cada branch o código está organizado assim:

```
frontend/   ← conteúdo de XColor-front (mesma branch)
backend/    ← conteúdo de XColor-back  (mesma branch)
```

---

## Como funciona a pipeline

### Fluxo automático

```
Push em XColor-front/dev
        │
        ▼
notify-deploy.yml (em XColor-front)
  envia repository_dispatch "frontend-push" → XColor-Deploy
        │
        ▼
sync-frontend.yml (neste repo)
  clona XColor-front/dev → copia para frontend/ → commit/push em XColor-Deploy/dev
```

O mesmo fluxo ocorre para as branches `test` e `production`, e de forma análoga para o backend.

### Pipelines deste repositório

| Arquivo | Trigger | O que faz |
|---------|---------|-----------|
| `.github/workflows/sync-frontend.yml` | `repository_dispatch: frontend-push` ou `workflow_dispatch` | Sincroniza `frontend/` a partir de XColor-front |
| `.github/workflows/sync-backend.yml` | `repository_dispatch: backend-push` ou `workflow_dispatch` | Sincroniza `backend/` a partir de XColor-back |

---

## Configuração inicial

### 1. Criar as branches de ambiente neste repositório

```bash
git checkout --orphan dev
git commit --allow-empty -m "chore: init dev branch"
git push origin dev

git checkout --orphan test
git commit --allow-empty -m "chore: init test branch"
git push origin test

git checkout --orphan production
git commit --allow-empty -m "chore: init production branch"
git push origin production
```

### 2. Criar o secret `DEPLOY_TOKEN`

Acesse **Settings → Secrets and variables → Actions** neste repositório e crie:

| Nome | Valor |
|------|-------|
| `DEPLOY_TOKEN` | Personal Access Token com escopo `contents: write` (fine-grained) ou escopo `repo` (classic) para este repositório |

Esse token é usado pelos workflows de sincronização para fazer push nas branches de ambiente.

### 3. Adicionar os workflows de notificação nos repositórios fonte

Copie os arquivos de `docs/` para os repositórios de origem:

- `docs/trigger-frontend.yml` → `.github/workflows/notify-deploy.yml` em **XColor-front**
- `docs/trigger-backend.yml`  → `.github/workflows/notify-deploy.yml` em **XColor-back**

Em cada um desses repositórios, crie o secret:

| Nome | Valor |
|------|-------|
| `DEPLOY_REPO_TOKEN` | Personal Access Token com escopo `actions: write` (fine-grained) ou escopo `repo` (classic) para **XColor-Deploy** — apenas para disparar `repository_dispatch` |

### 4. Execução manual (opcional)

Você pode disparar a sincronização manualmente a qualquer momento em  
**Actions → Sync Frontend** (ou **Sync Backend**) → **Run workflow** → escolha a branch.

---

## Dependências dos secrets

| Repositório | Secret | Para que serve |
|-------------|--------|----------------|
| XColor-Deploy | `DEPLOY_TOKEN` | Permite que os workflows façam push nas branches deste repo |
| XColor-front | `DEPLOY_REPO_TOKEN` | Permite disparar `repository_dispatch` em XColor-Deploy |
| XColor-back | `DEPLOY_REPO_TOKEN` | Permite disparar `repository_dispatch` em XColor-Deploy |

> **Permissões mínimas recomendadas:**
> - `DEPLOY_TOKEN` → fine-grained PAT com `Contents: Read and write` para XColor-Deploy (ou classic PAT com escopo `repo`)
> - `DEPLOY_REPO_TOKEN` → fine-grained PAT com `Actions: Write` para XColor-Deploy (ou classic PAT com escopo `repo`)
