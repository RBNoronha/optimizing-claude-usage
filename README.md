# Optimizing Claude Code Usage & Prompting Skill ⚡

Um plugin e skill especializada para o **Claude Code**, desenhada para gerenciar seu orçamento de tokens, blindar o **Prompt Cache**, calibrar modelos/esforço e manter a janela de contexto limpa e cirúrgica.

Construída a partir de benchmarks e das diretrizes oficiais de engenharia da Anthropic:
- [Maximizing the value of your Claude Code sessions](https://claude.com/blog/maximizing-the-value-of-your-claude-code-sessions)
- [Advanced Tool Use: Search, Programmatic Calling & Examples](https://www.anthropic.com/engineering/advanced-tool-use)
- [Prompting Best Practices & Model Guidance (Sonnet 5, Opus 5, Fable 5.1)](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices)

---

## 📦 Instalação Rápida (Recomendado via Claude Code)

Você pode instalar diretamente usando o sistema nativo de plugins/marketplaces do Claude Code:

### 1. Adicionar o Marketplace
No chat do Claude Code (ou no terminal):
```bash
/plugin marketplace add RBNoronha/optimizing-claude-usage-skill
```
*Ou via terminal convencional:*
```bash
claude plugin marketplace add RBNoronha/optimizing-claude-usage-skill
```

### 2. Instalar a Skill / Plugin
```bash
/plugin install optimizing-claude-usage@optimizing-claude-usage
```
*Ou via terminal convencional:*
```bash
claude plugin install optimizing-claude-usage@optimizing-claude-usage
```

---

## 🚀 O que a Skill faz (Detalhamento Completo)

Com a skill instalada e ativa, o Claude Code passa a seguir rigorosamente **5 alavancas estratégicas de economia e eficiência**:

### 1. Blindagem do Prompt Cache (Até 90% de Desconto em Tokens)
- **Bloqueio de Parâmetros na Largada:** Alteraçōes de `/model`, `/effort` ou ativação de *Fast mode* no meio da conversa invalidam o cache de prefixo, fazendo com que todo o histórico seja reprocessado com custo cheio. A skill instrui a travar esses parâmetros no início da sessão.
- **Timing Estratégico de `/compact`:** O prompt cache expira após 1 hora (ou 5 minutos via API key). Compilar e resumir o histórico exige ler toda a conversa; a skill orienta a rodar `/compact` **antes de pausas prolongadas**, aproveitando o cache ainda "quente" com 90% de desconto.
- **Uso de `/rewind` sobre `/compact`:** Para desvios ou erros recentes, orienta a podar os turnos ruins com `/rewind`, mantendo o histórico anterior 100% em cache com **custo zero de tokens adicionais**.

### 2. Higiene de Contexto & Gestão de Sessão
- **Isolamento de Tarefas com `/clear`:** Impede que o contexto de uma tarefa já resolvida seja arrastado para uma nova feature ou bugfix.
- **Auditoria de Instruções com `/context`:** Detecta regras infladas em `CLAUDE.md` e orienta migrar instruções repetitivas para skills sob demanda.
- **Anexação Direta via `@-mentions`:** Obriga referenciar arquivos como `@src/service.ts`. Sem o `@`, o Claude gasta tokens caros de output chamando ferramentas como `Read` ou fazendo buscas greps adicionais.
- **Edições Cirúrgicas (Diffs Pontuais):** Evita reescrever arquivos inteiros de 500+ linhas quando apenas 5 linhas precisam mudar (tokens de saída/decode custam ~5x mais que tokens de entrada).

### 3. Gestão Avançada de Ferramentas e Eliminação do "Imposto MCP"
- **Combate ao MCP Bloat:** Cinco servidores MCP (GitHub, Slack, Sentry, Jira, etc.) podem injetar de **55.000 a 100.000+ tokens de schemas** antes de você digitar uma única palavra. A skill ensina a desativar MCPs desnecessários via `/mcp` ou flags estritas (`--strict-mcp-config`).
- **Chamada Programática & Filtragem na Origem:** Proíbe injetar logs brutos de 10 MB ou dumps gigantes de banco no contexto. Orienta a usar pipelines bash (`grep`, `awk`, `head`) ou scripts auxiliares para que apenas os dados sintetizados entrem no chat.
- **Loteamento de Chamadas (*Tool-Call Batching*):** Agrupa leituras e inspeções de arquivos independentes no mesmo turno, reduzindo round-trips de inferência.
- **Flags Silenciosas para Comandos:** Impõe o uso de flags compactas em testes e linters (`--reporter=dot`, `-q`), evitando que 400 linhas de testes bem-sucedidos fiquem presas no histórico para sempre.
- **Uso Seletivo de Subagentes:** Cria subagentes apenas para isolar ruído pesado (varredura profunda de logs, investigações amplas), evitando criá-los para tarefas simples de leitura única onde a inicialização é mais cara que a execução inline.

### 4. Calibração de Modelo e Esforço por Cenário
Classifica a tarefa e impede a armadilha do *"começar barato e escalar se falhar"* (pois o custo de diagnosticar um código sutilmente incorreto é muito maior que o uso do modelo certo de início):
- **Claude Fable 5.1 / Mythos 5.1:** Tarefas multi-dia, agentes autônomos e geração end-to-end com updates de progresso e regras explícitas de compactação.
- **Claude Opus 5 / Opus 4.8:** Refatorações arquiteturais complexas e revisões sensíveis com especificações completas up-front e contenção de loops de sobreverificação.
- **Claude Sonnet 5:** Desenvolvimento diário, criação de testes e features com controle de verbosidade.
- **Claude Haiku:** Extração mecânica, boilerplates e triagem barata em subagentes.

### 5. Precisão de Prompting & Eliminação de Preâmbulos
- Elimina introduções conversacionais vazias (*"Com certeza, vou analisar..."*), saudações e repetições da pergunta, gerando apenas código e explicações acionáveis.
- Estrutura contextos complexos usando tags XML semânticas (`<context>`, `<specification>`, `<examples>`), maximizando a assertividade em contextos longos.

---

## 🛠️ Como a Skill é Acionada no Claude Code

1. **Gatilho Automático (Background):**  
   O Claude Code detecta a skill e a consulta automaticamente quando:
   - O contexto passa de ~150k tokens.
   - A sessão roda por horas sem fechamento de ciclo.
   - Subagentes começam a ser disparados repetitivamente.
   - Múltiplos servidores MCP estão conectados gerando alto overhead.

2. **Acionamento Manual (Prompt):**  
   Você pode chamar a skill ativamente em qualquer momento da conversa:
   > *"Claude, revise o consumo de tokens e a higiene da nossa sessão com base na skill optimizing-claude-usage."*
   > *"Vamos refatorar o módulo X. Siga as diretrizes da optimizing-claude-usage para não estourar o cache."*

---

## 📂 Instalação Manual (Alternativa sem Marketplace)

Se preferir copiar o arquivo diretamente para sua máquina:

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
