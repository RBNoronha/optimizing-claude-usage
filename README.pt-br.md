# Optimizing Claude Code Usage & Prompting Skill ⚡

![Version](https://img.shields.io/badge/version-1.1.0-blue.svg)
![Claude Code](https://img.shields.io/badge/Claude_Code-Plugin-orange.svg)

*( 🇺🇸 For English, see [README.md](README.md) )*

Um plugin e skill especializada para o **Claude Code**, desenhada meticulosamente para gerenciar seu orçamento de tokens, blindar o **Prompt Cache**, calibrar modelos/esforço e manter a janela de contexto limpa e cirúrgica durante suas sessões de desenvolvimento.

O **Claude Code** é uma ferramenta poderosa de codificação agentística, mas seu uso contínuo em projetos complexos pode resultar em estouro rápido de contexto, custos desnecessários de API (devido ao reprocessamento contínuo de tokens) e confusões na execução de tarefas se a janela de contexto não for bem gerenciada. 

Esta skill atua como um "mentor de engenharia de prompt e custos" embutido no seu próprio agente, ensinando o Claude a ser mais econômico, direto e eficiente enquanto trabalha para você.

---

## 📚 Bases e Fundamentos (A Filosofia da Skill)

O desenvolvimento desta skill não foi baseado em "achismos", mas sim na documentação oficial e nas diretrizes técnicas de engenharia da **Anthropic**. Ela materializa as práticas recomendadas nos seguintes guias fundamentais:

1. **Eficiência de Sessão & Custos**
   - [Maximizing the value of your Claude Code sessions](https://claude.com/blog/maximizing-the-value-of-your-claude-code-sessions): Guia central sobre como evitar reprocessamentos caros e quando usar comandos como `/compact` e `/rewind`.
2. **Uso Avançado de Ferramentas (Tool Use)**
   - [Advanced Tool Use: Search, Programmatic Calling & Examples](https://www.anthropic.com/engineering/advanced-tool-use): Diretrizes sobre como o modelo deve interagir com o terminal e o sistema de arquivos de forma otimizada (batching, filtragem na origem).
3. **Engenharia de Prompt e Modelos Específicos**
   - [Prompting Best Practices & Model Guidance](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices): Regras de estruturação de prompts usando XML tags e eliminação de preâmbulos.
   - Recomendações específicas por modelo de raciocínio, contemplando como lidar com a verbosidade e a contenção de loops:
     - [Prompting Claude Fable 5 & 5.1](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-fable-5-1)
     - [Prompting Claude Opus 4.8 & 5](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-opus-5)
     - [Prompting Claude Sonnet 5](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-sonnet-5)
4. **Boas Práticas de Criação de Skills**
   - Construída para seguir todas as diretrizes do guia *The Complete Guide to Building Skills for Claude*, implementando *Progressive Disclosure* (Arquivos de referência independentes), exemplos conversacionais explícitos e formatação de *Troubleshooting*.

---

## 📦 Instalação (Recomendado via Claude Code)

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

## 🚀 O que a Skill faz (As 5 Alavancas de Otimização)

Com a skill instalada e ativa, o Claude Code passa a seguir rigorosamente **5 alavancas estratégicas de economia e eficiência**:

### 1. Blindagem do Prompt Cache (Até 90% de Desconto em Tokens)
O Claude API oferece grandes descontos para contextos que permanecem inalterados (Prompt Caching). A skill ensina o Claude a não quebrar esse cache à toa.
- **Bloqueio de Parâmetros na Largada:** Alteraçōes de `/model`, `/effort` ou ativação de *Fast mode* no meio da conversa invalidam o cache de prefixo, fazendo com que todo o histórico seja reprocessado com custo cheio. A skill instrui a travar esses parâmetros no início da sessão.
- **Timing Estratégico de `/compact`:** O prompt cache expira após 1 hora. Compilar e resumir o histórico exige ler toda a conversa; a skill orienta a rodar `/compact` **antes de pausas prolongadas**, aproveitando o cache ainda "quente" com 90% de desconto.
- **Uso de `/rewind` sobre `/compact`:** Para desvios ou erros recentes, orienta a podar os turnos ruins com `/rewind`, mantendo o histórico anterior 100% em cache com **custo zero de tokens adicionais**.

### 2. Higiene de Contexto & Gestão de Sessão
O contexto inflado é o maior inimigo da precisão de um LLM.
- **Isolamento de Tarefas com `/clear`:** Impede que o contexto de uma tarefa já resolvida seja arrastado para uma nova feature ou bugfix.
- **Auditoria de Instruções com `/context`:** Detecta regras infladas no arquivo de diretrizes do projeto (`CLAUDE.md`) e orienta migrar instruções muito repetitivas para skills avulsas sob demanda.
- **Anexação Direta via `@-mentions`:** Obriga referenciar arquivos como `@src/service.ts`. Sem o `@`, o Claude gasta tokens mais caros (tokens de saída) chamando ferramentas do sistema de arquivos ou fazendo buscas greps que poderiam ser evitadas.
- **Edições Cirúrgicas (Diffs Pontuais):** Evita reescrever arquivos inteiros de 500+ linhas quando apenas 5 linhas precisam mudar, focando sempre em modificações in-place.

### 3. Gestão Avançada de Ferramentas e Eliminação do "Imposto MCP"
Ferramentas MCP (Model Context Protocol) adicionam funcionalidades, mas seus schemas consomem tokens invisíveis no início de todo prompt.
- **Combate ao MCP Bloat:** Servidores MCP (GitHub, Slack, Jira, etc.) podem injetar enormes quantidades de tokens de esquema antes mesmo de você digitar algo. A skill ensina o Claude a sugerir a desativação de MCPs desnecessários via `/mcp` ou flags estritas (`--strict-mcp-config`).
- **Chamada Programática & Filtragem na Origem:** Proíbe injetar logs brutos gigantescos ou dumps de banco de dados diretamente no chat. Orienta a usar pipelines bash (`grep`, `awk`, `head`) ou scripts temporários para que apenas as informações filtradas entrem no contexto.
- **Loteamento de Chamadas (*Tool-Call Batching*):** Agrupa leituras e inspeções de múltiplos arquivos independentes no mesmo turno, reduzindo o número de requisições enviadas à API e acelerando a resposta.
- **Flags Silenciosas para Comandos:** Impõe o uso de flags compactas em testes e linters (`--reporter=dot`, `-q`), evitando que centenas de linhas de sucesso fiquem poluindo o histórico.

### 4. Calibração de Modelo e Esforço por Cenário
Classifica a tarefa e impede a armadilha do *"começar barato e escalar se falhar"* (pois o custo de diagnosticar código corrompido é muito maior do que usar o modelo correto logo de início):
- **Claude Fable 5.1 / Mythos 5.1:** Ideal para tarefas multi-dia, desenvolvimento de agentes autônomos e geração end-to-end com regras explícitas de compactação.
- **Claude Opus 5 / Opus 4.8:** Para refatorações arquiteturais complexas e revisões de código muito sensíveis, onde é preciso ter todas as especificações "up-front" e conter loops de sobreverificação.
- **Claude Sonnet 5:** Recomendado para o desenvolvimento diário ágil, criação de testes unitários, pequenos bugfixes e novas features com forte controle de verbosidade.
- **Claude Haiku:** O modelo "trator", excelente para extração mecânica de dados, criação de boilerplates repetitivos e investigações rápidas (triagem) operando como um subagente.

### 5. Precisão de Prompting & Eliminação de Preâmbulos
Como o Claude pensa e entrega os resultados para o usuário:
- Corta introduções conversacionais vazias (*"Com certeza, vou analisar e fazer o que pediu..."*), saudações desnecessárias e repetições da pergunta original. O foco é gerar apenas o código e as explicações realmente acionáveis.
- Estrutura contextos complexos de maneira rígida, sempre usando tags XML semânticas (`<context>`, `<specification>`, `<examples>`), o que maximiza a assertividade quando a janela de contexto está cheia.

---

## 🛠️ Como a Skill é Acionada no Claude Code

1. **Gatilho Automático (Background):**  
   O Claude Code percebe a skill instalada e a consulta automaticamente em momentos de estresse de contexto, por exemplo:
   - Quando o contexto começa a ultrapassar a margem de segurança (~150k tokens).
   - Quando a sessão está rodando por várias horas consecutivas sem que o histórico tenha sido compactado.
   - Quando percebe que múltiplos servidores MCP não essenciais estão ativos gerando alto overhead passivo.

2. **Acionamento Manual (Você no controle):**  
   Você pode direcionar explicitamente o Claude para adotar a postura de otimização a qualquer momento:
   > *"Claude, revise o consumo de tokens e a higiene da nossa sessão com base na skill optimizing-claude-usage."*
   > *"Vou iniciar uma grande refatoração agora. Siga rigorosamente as diretrizes da optimizing-claude-usage para que a gente não estoure o cache à toa."*

---

## 📂 Instalação Manual Alternativa (Sem usar o Marketplace)

Caso você não queira adicionar o marketplace do repositório, pode apenas baixar e salvar a skill diretamente na sua pasta local de configuração do Claude Code:

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
