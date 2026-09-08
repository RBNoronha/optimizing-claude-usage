# Optimizing Claude Code Usage Skill

A specialized plugin/skill for **Claude Code** designed to help you manage your token budget, preserve the prompt cache, and keep your context window clean.

Based on Anthropic's official best practices for [Maximizing the value of your Claude Code sessions](https://claude.com/blog/maximizing-the-value-of-your-claude-code-sessions), this skill guides Claude (and you) on how to operate efficiently within the CLI environment.

---

## 📦 Instalação via Claude Code (Recomendado)

Você pode instalar diretamente usando o sistema de plugins e marketplaces do Claude Code:

### 1. Adicionar o Marketplace
No chat do Claude Code (ou no terminal):
```bash
/plugin marketplace add RBNoronha/optimizing-claude-usage-skill
```
*Ou via terminal:*
```bash
claude plugin marketplace add RBNoronha/optimizing-claude-usage-skill
```

### 2. Instalar a Skill / Plugin
```bash
/plugin install optimizing-claude-usage@optimizing-claude-usage
```
*Ou via terminal:*
```bash
claude plugin install optimizing-claude-usage@optimizing-claude-usage
```

---

## 📂 Instalação Manual (Alternativa)

Se preferir baixar diretamente para a pasta de skills sem usar o gerenciador de plugins:

### Global (Todos os projetos)

**Mac / Linux:**
```bash
mkdir -p ~/.claude/skills/optimizing-claude-usage
curl -L -o ~/.claude/skills/optimizing-claude-usage/SKILL.md https://raw.githubusercontent.com/RBNoronha/optimizing-claude-usage-skill/main/SKILL.md
```

**Windows (PowerShell):**
```powershell
New-Item -ItemType Directory -Force -Path $HOME\.claude\skills\optimizing-claude-usage
Invoke-WebRequest -Uri https://raw.githubusercontent.com/RBNoronha/optimizing-claude-usage-skill/main/SKILL.md -OutFile $HOME\.claude\skills\optimizing-claude-usage\SKILL.md
```

---

## 🚀 O que ela faz

Quando ativa, esta skill instrui o Claude Code a:
- **Preservar o Prompt Cache** (que deixa leituras de contexto 90% mais baratas) evitando ações que quebram o cache (como alterar modelo ou esforço no meio do turno).
- **Usar Subagentes estrategicamente** para isolar saídas ruidosas (como varreduras e logs longos) sem poluir a sessão principal.
- **Gerenciar tamanho de contexto**, incentivando `/clear` entre tarefas, `/context` para auditar ferramentas e `@-mentions` de arquivos em vez de chamadas `Read`.
- **Escolher o modelo ideal** para a complexidade da tarefa logo de início.

## 🛠️ Como usar

Uma vez instalado o plugin:
- Ele é carregado automaticamente pelo Claude Code.
- Dispara automaticamente em sessões longas ou quando o contexto passa de ~150k tokens.
- Você pode forçar a análise a qualquer momento digitando:
  > *"Revise o uso de tokens da sessão com base na skill optimizing-claude-usage"*
