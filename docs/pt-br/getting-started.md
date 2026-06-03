# Introdução ao axon

**axon** é um servidor MCP (Model Context Protocol) local que fornece contexto cirúrgico para agentes de IA que trabalham com código — montando cápsulas com orçamento de tokens a partir do seu projeto em vez de despejar arquivos brutos na janela de contexto.

Este guia leva você do zero até a primeira query `get_context_capsule` no Claude Code.

---

## Pré-requisitos

| Requisito | Observações |
|-----------|-------------|
| **jq** | Necessário pelo `install.sh` para manipulação de JSON. Instale com `brew install jq` (macOS) ou `apt install jq` (Debian/Ubuntu). |
| **git** | Necessário para `detect_changes` e registro de repositórios. Geralmente já instalado. |
| **Sem ferramentas de build** | Não necessário | O axon é distribuído como binário pré-compilado — nenhum compilador necessário. |

---

## Instalação

```mermaid
flowchart TD
    A[Choose Installation Method] --> B{Operating System?}
    B -->|macOS or Linux| C{Homebrew\ninstalled?}
    B -->|Windows| D[Option 3: Windows\ninstall.ps1]
    C -->|Yes| E[Option 1: Homebrew\nRecommended ✓]
    C -->|No| F[Option 2: Direct Download\ncurl + install.sh]
```

![Installation Paths](../assets/installation-paths.png)

### Opção 1 — Homebrew (recomendado para macOS e Linux)

```bash
brew tap HideakiSolutions/axon
brew install axon
```

Após a instalação, execute o script de instalação para indexar seu projeto, baixar o modelo de embeddings (~80 MB, baixado por padrão) e registrar o axon no Claude Code automaticamente:

```bash
./install.sh /caminho/para/seu-projeto
```

O `install.sh` instala hooks, escreve `<projeto>/.claude/settings.json`, indexa o projeto, baixa o modelo de embeddings e registra o servidor MCP via `claude mcp add-json axon ... --scope user`. Se o CLI `claude` não estiver no PATH, ele imprime o bloco JSON para colar manualmente em `~/.claude.json`.

> Para pular o download do modelo: `AXON_DOWNLOAD_MODEL=0 ./install.sh /caminho/para/seu-projeto`

Pronto. Pule para [Primeiro Índice](#primeiro-índice) se quiser entender o que o `install.sh` faz por baixo dos panos, ou vá direto para [Configurar o Claude Code](#configurar-o-claude-code-manualmente) se precisar de controle manual.

---

### Opção 2 — Download Direto (Linux / macOS tarball)

1. Acesse a **[página de Releases do GitHub](https://github.com/HideakiSolutions/axon-releases/releases/latest)** e baixe o arquivo da sua plataforma.

2. Baixe e extraia:

```bash
# Linux x86-64
curl -fL -o axon.tar.gz https://github.com/HideakiSolutions/axon-releases/releases/latest/download/axon-linux-x64.tar.gz
tar xzf axon.tar.gz
cd axon-*-linux-x64

# macOS Apple Silicon
# curl -fL -o axon.tar.gz https://github.com/HideakiSolutions/axon-releases/releases/latest/download/axon-macos-arm64.tar.gz
# tar xzf axon.tar.gz && cd axon-*-macos-arm64
```

3. Execute o instalador:

```bash
./install.sh /caminho/para/seu-projeto
```

O `install.sh` copia o binário para o PATH, instala hooks, escreve `<projeto>/.claude/settings.json`, indexa o projeto e baixa o modelo de embeddings (~80 MB) por padrão. Para pular: `AXON_DOWNLOAD_MODEL=0 ./install.sh /caminho/para/seu-projeto`.

---

### Opção 3 — Windows x64

```powershell
Invoke-WebRequest `
  "https://github.com/HideakiSolutions/axon-releases/releases/latest/download/axon-windows-x64.zip" `
  -OutFile "axon.zip"
Expand-Archive axon.zip -DestinationPath axon-windows-x64
cd axon-windows-x64
.\install.ps1 C:\caminho\para\seu-projeto
```

Para adicionar `axon.exe` ao PATH na sessão atual:

```powershell
$env:PATH += ";$(Resolve-Path bin)"
```

---

## Primeiro Índice

Seja usando o `install.sh` / `install.ps1` ou tendo instalado manualmente, indexar um projeto usa o mesmo comando:

```bash
axon index /caminho/para/seu-projeto
```

O axon irá:
1. Percorrer todos os arquivos de código no diretório (respeitando `.axonignore` e `.gitignore`).
2. Parsear cada arquivo com grammars tree-sitter para extrair símbolos e arestas.
3. Construir o grafo de dependências e armazená-lo em `.axon/index.duckdb` na raiz do projeto.
4. Opcionalmente computar embeddings para busca semântica (se `AXON_EMBEDDING_MODEL` estiver configurado).

Indexar um projeto de tamanho médio (10k–50k linhas) tipicamente leva 5–30 segundos. Monorepos grandes com mais de 500k linhas podem levar alguns minutos na primeira indexação; reindexações incrementais são muito mais rápidas.

```mermaid
flowchart LR
    A[axon index] --> B[Walk files\ngit-aware]
    B --> C[Parse with\ntree-sitter]
    C --> D[Build\ndependency graph]
    D --> E[Compute\nembeddings]
    E --> F[Store in\nDuckDB]
    F --> G[✓ Index ready\n5-30s typical]
```

---

## Verificar Status

Após indexar, verifique se o índice está saudável:

```bash
axon status
```

Exemplo de saída:

```
axon index status
  Project : /caminho/para/seu-projeto
  Files   : 312
  Symbols : 4.871
  Edges   : 9.204
  Obs.    : 0 observations saved
  Age     : 2 minutes ago
  Model   : nomic-embed-text-v1.5 (loaded)
  Cache   : 0 hits
```

Se a linha do modelo mostrar `not configured`, a busca semântica utilizará apenas traversal de grafo como fallback — todas as outras ferramentas funcionam normalmente. Veja [Modelo de Embeddings](#opcional-modelo-de-embeddings-para-busca-semântica) abaixo.

---

## Iniciar o Servidor MCP

O Claude Code se comunica com o axon via stdio MCP. Inicie o servidor:

```bash
axon serve
```

O servidor roda em primeiro plano, escutando stdin/stdout para mensagens JSON-RPC 2.0. O Claude Code gerencia o ciclo de vida do processo automaticamente após a configuração — você não precisa iniciar o `axon serve` manualmente a cada sessão.

---

## Configurar o Claude Code Manualmente

Se o `install.sh` não foi executado (ou se quiser verificar a configuração), adicione o seguinte ao `~/.claude.json`:

```json
{
  "mcpServers": {
    "axon": {
      "command": "axon",
      "args": ["serve"],
      "env": {
        "AXON_EMBEDDING_MODEL": "/caminho/para/nomic-embed-text-v1.5.Q4_K_M.gguf"
      }
    }
  }
}
```

```mermaid
sequenceDiagram
    participant U as User
    participant CC as Claude Code
    participant A as axon (MCP)
    participant DB as DuckDB index
    
    U->>CC: Ask about code
    CC->>A: get_context_capsule(query)
    A->>DB: semantic search + graph traversal
    DB-->>A: relevant files + symbols
    A-->>CC: token-efficient capsule
    CC-->>U: Informed answer
```

**Observações:**
- Se instalado via Homebrew, `axon` já está no PATH — o campo `command` funciona como está.
- Para instalações via download direto, use o caminho completo do binário: `"command": "/usr/local/bin/axon"`.
- A variável de ambiente `AXON_EMBEDDING_MODEL` é opcional. Omita-a se não tiver o arquivo do modelo.
- Reinicie o Claude Code após modificar o `~/.claude.json`.

---

## Opcional: Modelo de Embeddings para Busca Semântica

O axon suporta um modelo de embeddings local para o modo de query semântica no `get_context_capsule` e para `search_memory`. Sem ele, todas as 26 ferramentas funcionam — o `get_context_capsule` usa apenas traversal de grafo como fallback.

### Modelo é baixado automaticamente por padrão

O `install.sh` baixa o modelo de embeddings automaticamente. Para pular:

```bash
AXON_DOWNLOAD_MODEL=0 ./install.sh /caminho/para/seu-projeto
```

### Baixar manualmente ou usar seu próprio modelo

O modelo recomendado é `nomic-embed-text-v1.5.Q4_K_M.gguf` (~80 MB). Após o download, configure a variável de ambiente:

```bash
export AXON_EMBEDDING_MODEL=/caminho/para/nomic-embed-text-v1.5.Q4_K_M.gguf
```

Adicione isso ao seu perfil de shell (`~/.bashrc`, `~/.zshrc`) ou configure no bloco `env` do `~/.claude.json` (veja acima).

---

## Sua Primeira Query no Claude Code

Uma vez que o axon esteja indexado e o servidor MCP configurado, o Claude Code chamará as ferramentas do axon automaticamente sempre que precisar de contexto de código. Você não precisa invocá-las manualmente.

Para acionar sua primeira capsule de contexto, abra uma sessão do Claude Code no seu projeto e pergunte algo como:

```
Como funciona a autenticação nesse projeto?
```

O Claude Code chamará `get_context_capsule(query="como funciona a autenticação")` nos bastidores, receberá uma capsule eficiente em tokens dos arquivos relevantes e responderá com contexto preciso — sem ler cada arquivo.

Você também pode pedir ao Claude Code que execute ferramentas específicas explicitamente:

```
Use get_overview para mostrar os arquivos mais importantes deste projeto.
```

```
Execute get_impact_graph em src/auth/middleware.ts para eu saber o que quebraria se eu alterasse esse arquivo.
```

---

## Camada de Diálogo (Memória de Conversas)

O axon v1.1.0 introduziu a **Camada de Diálogo** — histórico de conversas armazenado nativamente junto com o índice do código no mesmo banco DuckDB. Isso permite persistir insights, decisões e contexto entre sessões do Claude Code.

### Como funciona

As conversas são organizadas como **threads → sessões → turns**:

```mermaid
graph TD
    T[Thread\ne.g. "refactor-auth"] --> S1[Sessão 1\n2026-05-10]
    T --> S2[Sessão 2\n2026-05-15]
    S1 --> Turn1[turn: user\n"Como funciona a validação JWT?"]
    S1 --> Turn2[turn: assistant\n"Usa a função validateToken..."]
    S2 --> Turn3[turn: user\n"Continuando da última sessão..."]
```

Cada turn é automaticamente **ancorado** aos arquivos e símbolos de código que referencia — sem vinculação manual.

### Início rápido

```
# No Claude Code, peça ao axon para rastrear esta sessão de trabalho:
thread_create(name="refactor-auth", kind="project")
session_start(thread_id=1, label="Investigação de validação JWT")

# Adicione turns conforme a conversa avança:
turn_add(session_id=1, role="user", content="Como funciona a validação JWT?")
turn_add(session_id=1, role="assistant", content="Usa validateToken em src/auth/token.ts...")

# Encerre e gere um digest da sessão ao terminar:
session_end(session_id=1, compute_digest=true)
```

### Recuperar contexto anterior

Em uma sessão futura, recupere o histórico relevante:

```
# Busca semântica sobre todos os turns anteriores:
turn_search(query="decisões sobre validação JWT")

# Ou injete turns anteriores em uma capsule:
get_context_capsule(
  query="validação de token de auth",
  pivot_files=["src/auth/token.ts"],
  dialogue_budget=1000
)
```

Veja [Ferramentas MCP](mcp-tools.md) para a referência completa das ferramentas da Camada de Diálogo (§16–§26).

---

## Próximos Passos

| Tópico | Documento |
|--------|-----------|
| Todos os comandos CLI com flags e exemplos | [Referência CLI](cli-reference.md) |
| Todas as 26 ferramentas MCP com parâmetros e uso | [Ferramentas MCP](mcp-tools.md) |
| Arquivos de configuração e variáveis de ambiente | [Configuração](configuration.md) |
| Padrões de fluxo agentic com prompts passo a passo | [Workflows](workflows.md) |
