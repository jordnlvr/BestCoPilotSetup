# Required VS Code Extensions

Install all of these. They integrate with Copilot and the agent framework.

## Installation (one command)

```
code --install-extension GitHub.copilot --install-extension GitHub.copilot-chat --install-extension eamodio.gitlens --install-extension esbenp.prettier-vscode --install-extension dbaeumer.vscode-eslint --install-extension usernamehw.errorlens --install-extension ms-azuretools.vscode-docker --install-extension rangav.vscode-thunder-client --install-extension Gruntfuggly.todo-tree --install-extension wix.vscode-import-cost
```

## Extension Details

| Extension | ID | Purpose |
|-----------|-----|---------|
| **GitHub Copilot** | `GitHub.copilot` | AI code completion — the engine |
| **GitHub Copilot Chat** | `GitHub.copilot-chat` | AI chat, agents, modes, prompts — the brain |
| **GitLens** | `eamodio.gitlens` | Git blame, history, visual diffs |
| **Prettier** | `esbenp.prettier-vscode` | Code formatting (format-on-save) |
| **ESLint** | `dbaeumer.vscode-eslint` | JavaScript/TypeScript linting |
| **Error Lens** | `usernamehw.errorlens` | Inline error display — see problems immediately |
| **Docker** | `ms-azuretools.vscode-docker` | Dockerfile/Compose intelligence |
| **Thunder Client** | `rangav.vscode-thunder-client` | REST API testing (like Postman, in VS Code) |
| **TODO Tree** | `Gruntfuggly.todo-tree` | Find and track TODO/FIXME/HACK comments |
| **Import Cost** | `wix.vscode-import-cost` | Show package import sizes inline |

## Optional but Recommended

| Extension | ID | Purpose |
|-----------|-----|---------|
| **GitHub Actions** | `github.vscode-github-actions` | CI/CD workflow editing |
| **Markdown All in One** | `yzhang.markdown-all-in-one` | Better markdown editing |
| **Path Intellisense** | `christian-kohler.path-intellisense` | File path autocomplete |
| **REST Client** | `humao.rest-client` | Alternative API testing with .http files |

## Verification

After installing, run in terminal:
```powershell
code --list-extensions | Select-String "copilot|gitlens|prettier|eslint|errorlens|docker|thunder|todo-tree|import-cost"
```

All 10 core extensions should appear.
