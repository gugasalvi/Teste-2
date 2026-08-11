
---

# Roteiro 2 – Branches

## 1. Voltando a um commit (Revert / Reset) - Roteiro 1 (ainda)

| Método          | Quando usar                             | Comando principal                     |
| --------------- | --------------------------------------- | ------------------------------------- |
| **Nova branch** | Precisa testar mudança sem tocar `main` | `git checkout -b volta-commit <hash>` |
| **git revert**  | Quer preservar histórico (opção segura) | `git revert <hash>`                   |
| **git reset**   | Precisa reescrever histórico local      | `git reset --hard <hash>`             |

1. **Descobrir hash** do commit alvo:
   ```bash
   git log --oneline --graph --decorate -n 10
   ```
2. **Criar branch** (opcional, mas recomendado):
   ```bash
   git checkout -b hotfix/volta-commit <hash>
   ```
3. **Reverter (seguro):**
   ```bash
   git revert <hash>
   git push origin main
   ```
4. **Reset (perigoso):**
   ```bash
   # Desfaz commits locais e move ponteiro; use --soft para manter staging, --mixed (padrão) ou --hard.
   git reset --hard <hash>
   git push --force-with-lease
   ```

> **Dica:** nunca use `git reset --hard` depois que commits já foram enviados e utilizados por outros.

---

## 2) Convenções de **Git Semântico** (Conventional Commits)

Use mensagens padronizadas e curtas, com **tipo**, **escopo** (opcional) e **resumo no imperativo**.

**Formato:**

```
<tipo>(<escopo>): <resumo>

[corpo opcional explicando o porquê]
[Refs: #<issue>]
```

**Tipos comuns:**

- `feat` (nova funcionalidade)
- `fix` (correção de bug)
- `docs` (documentação)
- `style` (formatação, sem mudança de lógica)
- `refactor` (refatoração, sem alterar comportamento externo)
- `test` (testes)
- `build` (build, dependências)
- `ci` (pipelines de CI)
- `chore` (tarefas diversas que não afetam src/test)

**Exemplos práticos:**

```bash

feat(readme): inclusão do campo about me
fix(api): tratar null pointer ao criar pedido
docs(contrib): adicionar guia de pull request
refactor(core): extrair serviço de cálculo de frete
chore(deps): atualizar axios para ^1.7.0
```

> **Por que usar?** Facilita *changelog*, leitura do histórico e automações (versionamento semântico, release notes, validações de CI).


## 3) Trabalhando com `git stash`

O `git stash` é usado para **guardar mudanças temporariamente** sem precisar fazer commit, permitindo trocar de branch ou atualizar código sem perder alterações não finalizadas.

**Comandos principais:**

- Guardar alterações:
  ```bash
  git stash
  ```
- Guardar com mensagem:
  ```bash
  git stash save "ajustes parciais no componente header"
  ```
- Listar stashes salvos:
  ```bash
  git stash list
  ```
- Recuperar o último stash (mantendo no histórico):
  ```bash
  git stash apply
  ```
- Recuperar e remover do histórico:
  ```bash
  git stash pop
  ```
- Aplicar stash específico:
  ```bash
  git stash apply stash@{2}
  ```
- Limpar todos os stashes:
  ```bash
  git stash clear
  ```

> Útil quando precisa trocar de branch no meio do desenvolvimento ou atualizar a `main` sem perder o que já começou.

## 4) Convenções de **nomenclatura** de branches

Use nomes curtos, descritivos e com *slug* (kebab-case). Inclua tipo e, se houver, ID do ticket.

- `feature/<id>-<descricao>` → `feature/1234-criar-endpoint-pedidos`
- `fix/<id>-<descricao>` → `fix/235-bug-null-pointer-checkout`
- `chore/<descricao>` → `chore/atualizar-dependencias`
- `hotfix/<descricao>` → `hotfix/corrigir-regra-frete`
- `docs/<descricao>` → `docs/roteiro-git-pr`

---

## 5) Criar branch a partir da `main`

Sempre atualize sua `main` local antes de ramificar.

```bash
git checkout main
git pull

git checkout -b feature/1234-criar-endpoint-pedidos

```

---

## 6) Ciclo de commits (aplicando Git Semântico)

Mantenha commits pequenos, coesos e com mensagens claras.

**Exemplo completo:**

```bash
git add .
git commit -m "feat(api): criar endpoint POST /pedidos Adiciona validações e integra com serviço de estoque. Refs: #1234"
```

---

## 7) Manter sua branch atualizada (rebase recomendado)

Enquanto desenvolve, replique alterações da `main` para evitar *big bang merges*.

```bash
git fetch origin
git rebase origin/main
# Resolva conflitos (se houver), depois:
git rebase --continue

```

---

## Extra: O que é `--force-with-lease` e por que usar?

Ao reescrever histórico (ex.: `git rebase`) e enviar para o remoto, o comando `git push` normal será rejeitado se houver divergências. O `--force` sobrescreve tudo **sem verificar** se alguém mais atualizou o remoto, podendo apagar trabalho alheio.

O `--force-with-lease` é mais seguro: ele **só força o push se a referência remota ainda estiver como você a viu no último **``**/**``. Se outra pessoa tiver feito um push no intervalo, o comando falha, evitando sobrescrever mudanças sem querer.

**Fluxo recomendado:**

```bash
git fetch origin
git rebase origin/main
git push --force-with-lease
```

> Sempre combine com a equipe antes de usar, especialmente em branches compartilhadas.

---

## 8) Material de estudo extra:

- [Video da explicação do que é branche (legendado)](https://www.youtube.com/watch?v=e9lnsKot_SQ)

---



