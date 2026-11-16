# equipe-dev

Plugin do Claude Code com uma **equipe de agentes para desenvolver sites**. Você descreve o site que quer, e a equipe faz as perguntas, escreve os requisitos, define a arquitetura e o design, implementa, revisa e testa, sempre parando para pedir sua aprovação entre as etapas.

| Agente | Papel | Escreve |
|---|---|---|
| `po` | Product Owner: entende o que você quer e escreve os requisitos | `docs/01-requisitos.md` |
| `arquiteto` | Arquiteto / Tech Lead: escolhe a stack, planeja as tarefas e faz o code review | `docs/02-arquitetura.md`, `docs/05-code-review.md` |
| `ui-ux` | Designer: direção visual, wireframes, design tokens, acessibilidade | `docs/03-design.md` |
| `desenvolvedor` | Implementa o site tarefa por tarefa e corrige o que vier do review e dos testes | código + `docs/progresso.md` |
| `tester` | QA: valida os critérios de aceite, links, formulários, responsividade, SEO e acessibilidade | `docs/04-relatorio-testes.md` |

O comando `/equipe` é o **orquestrador**: roda na sua sessão principal, chama cada agente, repassa as perguntas para você e controla os checkpoints.

## Instalação

Este repositório é ao mesmo tempo o plugin e um marketplace com um único plugin.

```
/plugin marketplace add victordosanjos1/equipe-dev
/plugin install equipe-dev@equipe-dev
```

Depois, reinicie o Claude Code (ou rode `/reload-plugins`, se disponível) para carregar o comando e os agentes.

> **Repositório privado:** o Claude Code baixa o marketplace com o `git` da sua máquina, então o git precisa estar autenticado no GitHub com uma conta que tenha acesso ao repositório. Teste com `git ls-remote https://github.com/victordosanjos1/equipe-dev.git`; se pedir senha ou der erro, autentique com `gh auth login` (e `gh auth setup-git`) ou com o Git Credential Manager. Para atualizações automáticas em segundo plano, defina a variável de ambiente `GITHUB_TOKEN` com um token que tenha acesso ao repositório.

### Testar localmente sem instalar do GitHub

```
/plugin marketplace add C:/caminho/para/equipe-dev
/plugin install equipe-dev@equipe-dev
```

## Como usar

Abra o Claude Code na pasta (vazia ou não) onde o site vai ficar e rode:

```
/equipe quero um site para minha barbearia
```

Se o trabalho for interrompido, rode `/equipe` de novo na mesma pasta: o orquestrador lê `docs/progresso.md` e **retoma de onde parou**.

Você também pode chamar um agente sozinho, por exemplo: "use o agente tester para revisar a acessibilidade do site".

## Fluxo das etapas

```
0. Preparação ── cria docs/ e docs/progresso.md (ou retoma)
1. Descoberta ── PO faz perguntas (até 3 rodadas) e escreve os requisitos  → ✋ CHECKPOINT
2. Arquitetura ─ Arquiteto escolhe a stack e cria o plano de tarefas        → ✋ CHECKPOINT
3. Design ────── UI/UX define direção visual, wireframes e tokens            → ✋ CHECKPOINT
4. Desenvolvimento ─ Desenvolvedor implementa em blocos → Arquiteto revisa
                     → Desenvolvedor corrige críticos/importantes            → ✋ CHECKPOINT (como rodar local)
5. Testes ────── Tester valida → Desenvolvedor corrige → Tester re-testa
                 (até 2 ciclos)                                              → ✋ CHECKPOINT FINAL
```

Em cada checkpoint você recebe um resumo curto e o caminho do arquivo para ler completo. Nada avança sem a sua aprovação explícita; se você pedir mudanças, o agente responsável ajusta e o checkpoint se repete.

## Arquivos gerados em `docs/` (no projeto do site)

| Arquivo | Conteúdo |
|---|---|
| `docs/progresso.md` | Etapa atual, checklist das etapas, perguntas e respostas da descoberta, decisões, histórico, tarefas de desenvolvimento e dúvidas |
| `docs/01-requisitos.md` | Visão, público, escopo (dentro/fora), páginas, histórias de usuário com critérios de aceite, requisitos não funcionais, dúvidas em aberto |
| `docs/02-arquitetura.md` | Stack com justificativa, estrutura de pastas, convenções, como rodar, deploy e plano de tarefas numerado (T01…) |
| `docs/03-design.md` | Direção visual, mapa do site, fluxos, wireframes em texto, design tokens, componentes, responsivo, checklist WCAG AA |
| `docs/04-relatorio-testes.md` | Status de cada critério de aceite (passou/falhou), verificações gerais e bugs com passos para reproduzir e severidade |
| `docs/05-code-review.md` | Problemas classificados em crítico / importante / sugestão |

Os agentes conversam entre si por esses arquivos. Você pode editá-los à mão a qualquer momento; o próximo agente vai ler a versão atualizada.

## Estrutura do plugin

```
equipe-dev/
├── .claude-plugin/
│   ├── plugin.json        # manifesto do plugin
│   └── marketplace.json   # marketplace que lista este próprio plugin (source "./")
├── agents/
│   ├── po.md
│   ├── arquiteto.md
│   ├── ui-ux.md
│   ├── desenvolvedor.md
│   └── tester.md
├── commands/
│   └── equipe.md          # orquestrador (/equipe)
└── README.md
```

## Como atualizar o plugin depois de editar os agentes

1. Edite os arquivos em `agents/` ou `commands/`.
2. Aumente a versão em `.claude-plugin/plugin.json` **e** em `.claude-plugin/marketplace.json` (ex.: `1.0.0` → `1.1.0`). O Claude Code usa a versão para perceber que há atualização.
3. Faça commit e push:
   ```
   git add -A
   git commit -m "Ajusta prompt do tester"
   git push
   ```
4. No Claude Code:
   ```
   /plugin marketplace update equipe-dev
   ```
   e depois atualize o plugin pelo menu `/plugin` (aba de plugins instalados) ou reinstale com `/plugin install equipe-dev@equipe-dev`. Reinicie o Claude Code para carregar as mudanças.

Se instalou a partir de uma pasta local, basta o passo 4 (não precisa de push).
