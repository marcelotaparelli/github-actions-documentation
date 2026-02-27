# github-actions-documentation

## Conceitos Fundamentais

* **Workflow:** O processo completo definido em um arquivo `.yml` em `.github/workflows/`.
* **Events (on):** Gatilhos que iniciam o workflow (`push`, `pull_request`, `schedule`).
* **Jobs:** Unidades de execução que rodam em paralelo (a menos que use `needs`).
* **Steps:** Sequência de tarefas dentro de um Job.
* **Runners:** Máquinas virtuais (Ubuntu, Windows, macOS) que executam os jobs.

---

## Dicionário de Comandos (YAML)

| Comando | Descrição |
| :--- | :--- |
| `name` | Nome do Workflow ou do Step (exibido na interface). |
| `on` | Define o evento disparador. |
| `runs-on` | Define o SO da máquina virtual (ex: `ubuntu-latest`). |
| `uses` | Utiliza uma Action pronta da comunidade (reuso de código). |
| `with` | Passa parâmetros/argumentos para uma Action (`uses`). |
| `run` | Executa comandos de terminal (shell/bash). |
| `env` | Define variáveis de ambiente. |
| `needs` | Cria dependência entre Jobs (ex: Deploy precisa do Teste). |

---

## Exemplo Real: CI para NestJS/Node.js

Este workflow valida o código em cada Push ou Pull Request.

```yaml
name: Node CI/CD Pipeline

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

jobs:
  lint-and-test:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout do Código
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'

      - name: Instalar Dependências
        run: npm ci

      - name: Rodar Lint
        run: npm run lint

      - name: Rodar Testes de Integração
        run: npm test

  deploy:
    needs: lint-and-test
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    steps:
      - name: Deploy via SSH
        uses: appleboy/ssh-action@master
        with:
          host: ${{ secrets.VPS_HOST }}
          username: ${{ secrets.VPS_USER }}
          key: ${{ secrets.SSH_PRIVATE_KEY }}
          script: |
            cd /app/meu-portfolio
            git pull origin main
            docker-compose up -d --build

```


## Gerenciamento de Segredos (Secrets)

Nunca coloque senhas ou chaves de API direto no YAML.

1. Vá em seu repositório no GitHub: `Settings > Secrets and variables > Actions`.
2. Adicione seus segredos (ex: `DB_PASSWORD`, `SSH_KEY`).
3. No YAML, acesse usando: `${{ secrets.NOME_DO_SEGREDO }}`.

---

## Dicas de Performance

* **Cache de Dependências:** Use `cache: 'npm'` no `setup-node` para acelerar o build.
* **Matrix Testing:** Teste em múltiplas versões simultaneamente:
```yaml
strategy:
  matrix:
    node: [18, 20, 22]

```


* **Conditional Steps:** Use `if: failure()` para rodar um passo apenas se o anterior der erro (ex: enviar alerta no Slack).

---

## Comandos Úteis de Debug

* `echo "${{ toJSON(github) }}"`: Printa todo o contexto do GitHub disponível no workflow.
* `run: ls -R`: Lista os arquivos na máquina virtual para verificar se o checkout funcionou.

```




máquina da Hostinger pela primeira vez?

```
