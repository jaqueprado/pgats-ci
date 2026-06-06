[![Code coverage badge](https://img.shields.io/badge/coverage-100%25-brightgreen)](https://stryker-mutator.io/robo-coasters-example/reports/coverage/lcov-report/index.html)
[![Mutation testing badge](https://img.shields.io/endpoint?style=flat&url=https%3A%2F%2Fbadge-api.stryker-mutator.io%2Fgithub.com%2Fstryker-mutator%2Frobo-coasters-example%2Fmaster)](https://dashboard.stryker-mutator.io/reports/github.com/stryker-mutator/robo-coasters-example/master)

# PGATS - CI

## Pré-requisitos

1. Instale o [git](https://git-scm.com)
2. Instale o [nodejs](https://nodejs.org/)
3. Instale o Yarn - `npm install -g yarn`
4. Faça um _Fork_ do projeto
5. Clone o repositório para sua máquina (seu fork)
6. Instale as dependências
   ```shell
   cd pgats-ci
   yarn
   ```
7. Execute os testes de unidade - isso vai gerar um relatório
   ```shell
   yarn run test
   ```
8. Abra o relatório de cobertura de código em `reports/coverage/lcov-report`
9. Execute os testes de mutação com o Stryker
   ```shell
   yarn run test:mutation
   ```
10. Abra o relatório de mutação em `reports/mutation`
11. Instale os navegadores do Playwright
    ```shell
    yarn playwright install
    ```
12. Execute os testes end-to-end com o Playwright
    ```shell
    yarn run e2e
    ```
13. Execute a aplicação com `yarn start`
14. Acesse a aplicação publicada [neste link](https://pgats-ci-example.netlify.app)

---

💜⚡️

# pgats-ci

----------------------------------------------------------------------------------------

# 🔄 CI/CD Pipeline: Fluxo de Qualidade (CircleCI)

Este repositório utiliza o **CircleCI** para garantir a qualidade do código através de um fluxo contínuo de integração. A pipeline (configurada na versão `2.1`) realiza verificações estáticas, testes unitários e testes ponta-a-ponta (E2E) em um ambiente isolado e padronizado.

## 🚀 Visão Geral do Workflow (`quality-workflow`)

A pipeline principal é chamada de `quality-workflow` e é projetada para ser eficiente e segura. Ela garante que testes mais pesados só sejam executados se o código passar nas verificações básicas.

O fluxo de execução segue a seguinte ordem:

1. **Estágio 1 (Paralelo):** Os jobs `lint` e `test-unit` são executados simultaneamente assim que o código é "pushado".
2. **Estágio 2 (Condicional):** O job `test-e2e` **só é iniciado** se os jobs `lint` e `test-unit` forem concluídos com sucesso.

---

## 🛠️ Ambientes de Execução (Executors)

Para otimizar o tempo e os recursos, a pipeline utiliza dois ambientes Docker distintos:

| Executor | Imagem Docker | Propósito | Configurações Específicas |
| :--- | :--- | :--- | :--- |
| **`node-executor`** | `node:24` | Linting e Testes Unitários | `NODE_ENV='test'`, `CI_TIMEOUT='1800'` |
| **`playwright-executor`** | `mcr.microsoft.com/playwright:v1.60.0-jammy` | Testes E2E (Playwright) | Já possui os navegadores embutidos. |

---

## ⚙️ Detalhamento dos Jobs

### 1. `lint` (Verificação Estática)
Garante que o código segue os padrões de formatação e tipagem do projeto.
* **Ambiente:** `node-executor`
* **Passos:**
  * Baixa o código fonte (`checkout`).
  * Restaura o cache do `node_modules` usando o `package-lock.json` para acelerar a instalação.
  * Instala dependências usando `npm install --legacy-peer-deps`.
  * Salva o cache atualizado.
  * Executa o comando de linting do TypeScript: `npm run lint:ts`.

### 2. `test-unit` (Testes Unitários)
Executa a suíte de testes unitários com suporte a execução paralela para maior velocidade.
* **Ambiente:** `node-executor`
* **Performance:** Utiliza `parallelism: 3` para dividir os testes em 3 instâncias simultâneas.
* **Passos:**
  * Baixa o código fonte.
  * Busca por caches antigos (tenta o `yarn.lock` primeiro, com fallback para `package-lock.json`).
  * Detecta automaticamente o gerenciador de pacotes correto (se `yarn.lock` existir, usa `yarn install --frozen-lockfile`; senão, `npm ci`).
  * Executa os testes gerando relatório de cobertura: `yarn run test --coverage`.
  * **Artefatos:** Salva os relatórios gerados na aba de artifacts do CircleCI (`reports/coverage/`).

### 3. `test-e2e` (Testes Ponta-a-Ponta)
Valida o comportamento da aplicação simulando a interação real de um usuário no navegador.
* **Ambiente:** `playwright-executor` *(Não requer instalação manual dos browsers, economizando tempo)*.
* **Passos:**
  * Baixa o código e restaura caches de dependência.
  * Instala as dependências de forma segura (`yarn install --frozen-lockfile` ou `npm ci`).
  * Executa a suíte E2E: `npm run e2e`.
  * **Artefatos:** Salva dois diretórios para depuração caso os testes falhem:
    * `playwright-report/` (Relatório visual em HTML do Playwright).
    * `test-results/` (Screenshots, vídeos e traces de falhas).

### Self-hosted runner

Tambem foi configurado um self-hosted runner do GitHub Actions em uma maquina
Windows local. Esse recurso permite executar workflows em uma maquina propria,
em vez de usar apenas os runners hospedados pelo GitHub.

O runner foi configurado no GitHub em:

```text
Settings > Actions > Runners > New self-hosted runner
```

Depois da configuracao no PowerShell, o runner ficou conectado ao GitHub com a
mensagem:

```text
Connected to GitHub
Listening for Jobs
```

Foi criado o workflow `.github/workflows/03-self-hosted.yaml`, configurado
para executar no agent local com:

Esse workflow executa checkout, instalacao do Node.js, instalacao das
dependencias, lint, testes unitarios, testes E2E, publicacao de relatorio e
upload de artefatos.

### Quando usar self-hosted runners?

A decisão de migrar de runners gerenciados na nuvem (SaaS) para self-hosted geralmente envolve um ou mais dos seguintes cenários:

Acesso à Rede Interna (On-Premise): Sua pipeline precisa fazer deploy em servidores que estão atrás de um firewall corporativo, ou precisa rodar testes de integração contra um banco de dados local que não é exposto à internet.

Hardware Específico ou Poder de Processamento: Você precisa de GPUs para treinar modelos de Machine Learning, muita memória RAM para compilar projetos gigantes, ou sistemas operacionais específicos (como um Mac Mini físico para fazer build de aplicativos iOS).

Controle de Custos: Se o seu time roda milhares de builds por dia, pagar pelos minutos de computação das plataformas de CI/CD pode ficar extremamente caro. Ter servidores próprios (ou instâncias EC2 reservadas na AWS) dedicados a isso pode reduzir drasticamente a conta no final do mês.

Segurança e Compliance: Regulações rígidas podem exigir que o código-fonte, chaves de criptografia e artefatos gerados nunca saiam da infraestrutura controlada pela empresa.

Cache Persistente: Como a máquina não é destruída após cada job (a menos que você configure runners efêmeros), você pode manter dependências massivas cacheadas no disco local, acelerando muito os builds.

### Recursos similares em outras plataformas

Outras plataformas tambem oferecem recursos parecidos:

- GitHub Actions: self-hosted runners.
- Azure DevOps: self-hosted agents.
- GitLab CI: GitLab runners.
- Jenkins: agents/nodes.

