# HeroForge Implementation Blueprint

## Stack recomendada

- Backend: TypeScript + Node.js
- Banco: PostgreSQL
- Frontend admin: Angular ou React, desde que entregue páginas equivalentes
- Providers: abstração única de LLM com implementações específicas

## Estrutura de pastas sugerida

```text
heroforge/
  .agents/
    prompts/
      architect.md
      developer.md
      reviewer.md
      repair.md
  frontend/
  specs/
  src/
    api/
    config/
    core/
    interfaces/
    orchestrator/
    providers/
    telemetry/
      sql/
    tools/
    index.ts
  output/
  .env.example
  package.json
  tsconfig.json
```

## Componentes mínimos

### Backend core

- `TaskOrchestrator`
  - coordena scan, architect, developer, reviewer, approval policy e escrita segura
- `AgentLoop`
  - executa o ciclo ReAct de cada agente
- `RepoScanner`
  - inspeciona árvore e conteúdos relevantes do repositório-alvo
- `SafeFileWriter`
  - aplica mudanças aprovadas com escrita atômica

### Provider layer

- `ILlmProvider`
- `ProviderFactory`
- `GeminiProvider`
- `DeepSeekProvider`
- `GroqProvider`
- `OllamaProvider`
- `ResilientProvider`

### Admin/API layer

- `LocalApiServer`
- `LlmServiceRepository`
- `RoleProviderBindingRepository`

### Telemetry layer

- `PostgresConnection`
- `PostgresRuntimeTelemetryRepository`
- `SqlScriptRunner`
- `schema.sql`
- `seed_llm_control_plane.sql`

## Fluxo principal

1. CLI recebe tarefa e caminho do repositório
2. Configuração é validada
3. TaskOrchestrator inicializa telemetria e scanner
4. ProviderFactory resolve a cadeia por papel a partir do banco
5. Architect recebe contexto e devolve plano
6. Developer gera patches
7. Reviewer aprova ou rejeita
8. Em `dry-run`, gera artefatos
9. Em `apply`, escreve no repositório após aprovação

## Modelo de dados mínimo

### `llm_services`

Tabela para catálogo de serviços e modelos.

### `role_provider_bindings`

Tabela para definir qual serviço atende cada papel e em qual ordem.

### `task_runs`

Tabela para resumir execuções completas.

### `llm_call_records`

Tabela para registrar chamadas de modelo e associar custo e serviço.

### `budget_states`

Tabela para acompanhar orçamento operacional por run.

## Regras de qualidade

- O runtime deve ler a verdade operacional do banco
- O `.env` deve conter só segredos e parâmetros gerais
- Toda mutação relevante de bindings deve ser transacional
- O frontend deve refletir a ordem real do runtime
- O projeto deve suportar extensão futura sem reescrever o núcleo

## Riscos que devem ser tratados

- bindings inválidos ou vazios
- serviço sem segredo disponível no ambiente
- serviço indisponível em runtime
- custo não calculável por falta de pricing
- run interrompido antes de finalizar telemetria