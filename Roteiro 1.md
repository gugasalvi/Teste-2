# Roteiro 1 – Aula Prática: Git & GitHub


---

## 1. Pré‑requisitos

| Item            | Detalhes                                                         |
| --------------- | ---------------------------------------------------------------- |
| Conta GitHub    | [https://github.com](https://github.com) (gratuita)              |
| Git bash instalado   | [Download](https://git-scm.com/) – verifique com `git --version` |
| Editor de texto | VS Code, Atom, Sublime ou equivalente                            |

```bash
# Configurar identidade global (substitua pelos seus dados)
git config --global user.name "Seu Nome"
git config --global user.email "voce@email.com"
```

---

## 2. Criar o repositório *de perfil*

1. Acesse `github.com`.
2. Em **Repository name** (digitar próprio usuário), digite `` (ex.: `maria‑dev`).
3. Selecione **Public**.
4. Marque **Add a README file**.
5. (Opcional) Escolha licença *MIT* e arquivo `.gitignore` para *Node*.
6. Clique **Create repository**.
7. Clone localmente:
   ```bash
   git clone https://github.com/<user>/<user>.git
   cd <user>
   ```
8. Edite `README.md` (bio, skills, badges) e faça:
   ```bash
   git add README.md
   git commit -m "feat: primeira versão do README"
   git push origin main
   ```

> **Dica:** ative **GitHub Pages** em *Settings ▸ Pages* para exibir seu portfólio (branch `main`, pasta `/root`).

---

## 3. Criar o repositório ``

1. Novamente em **New repository**, use o nome ``.
2. Descrição: *Anotações da disciplina e comandos Git*.
3. Marque **Add a README** e crie.
4. Estrutura de pastas sugerida:
   ```text
   my-github/
   ├─ README.md              # visão geral do repositório
   ├─ comandos-git.md        # "cheat sheet" de comandos
  
   ```
5. Preencher **comandos-git.md** com tabela inicial:
   | Comando                     | Descrição                           |
   | --------------------------- | ----------------------------------- |
   | `git init`                  | Inicializa um repositório vazio     |
   | `git clone <url>`           | Clona repositório remoto            |
   | `git status`                | Mostra estado da árvore de trabalho |
   | `git add <arq>`             | Adiciona mudanças ao stage          |
   | `git commit -m "msg"`       | Registra snapshot                   |
   | `git push`                  | Envia commits ao GitHub             |
   | `git pull`                  | Sincroniza e integra mudanças       |
   | `git branch`                | Lista ou cria branches              |
   | `git merge`                 | Mescla branches                     |
   | `git log --oneline --graph` | Histórico compacto                  |

---

## 4. Sincronizar repositórios localmente

```bash
# Diretório de trabalho para todos os projetos
mkdir ~/repos-projeto-iot && cd ~/repos-projeto-iot

git clone https://github.com/<user>/<user>.git
git clone https://github.com/<user>/my-github.git
```

---

## 5. Fluxo de trabalho básico

1. **Create/Change**: editar arquivo.
2. **Review**: `git status`.
3. **Stage**: `git add <arquivo>`.
4. **Commit**: `git commit -m "tipo: mensagem"`.
5. **Push**: `git push origin main`.

> Convenção de mensagens: `feature:`, `hotfix:`, `docs:`…

---

### 5.1 Voltando a um commit (Revert / Reset)



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



## 10. Entregáveis (até 19·ago·2025)

- Repositório do Perfil com README preenchido (about, stacks, bages).
- Repositório contendo pelo menos:
  - `comandos-git.md` com 10+ comandos documentados.
  


---

## 11. Links e Livros

- [GitHub Docs – Hello World](https://docs.github.com/en/get-started/quickstart)
- [Pro Git Book (pt‑BR)](https://git-scm.com/book/pt-br/v2)
- [Git Cheat Sheet PDF](https://education.github.com/git-cheat-sheet-education.pdf)

---


