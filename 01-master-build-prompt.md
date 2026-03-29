# HeroForge Master Build Prompt

Use este documento como prompt principal para outra LLM agente construir o projeto HeroForge do zero.

---

Você vai criar um projeto chamado **HeroForge**.

## Objetivo do produto

Construa um orquestrador local-first multiagente para tarefas de engenharia de software. O sistema deve receber uma tarefa, inspecionar um repositório-alvo, delegar planejamento, implementação e revisão a agentes especializados, registrar telemetria operacional, e expor uma API administrativa e uma interface administrativa para governar provedores de LLM e cadeias de fallback por papel.

O projeto não deve ser acoplado a um domínio específico do usuário final. Os prompts dos agentes devem ser tratáveis como módulos editáveis, para que qualquer pessoa adapte o comportamento depois.

## Nome e identidade

- Nome do projeto: **HeroForge**
- O nome HeroForge deve permanecer no projeto, no README e nos artefatos principais.
- O crédito do criador original deve ser preservado onde fizer sentido em documentação e licença.
- Criador original: **Cristiano Lagame**
- Contato do criador original: **crislagame@gmail.com**

## Entregável esperado

Você deve gerar um projeto funcional com:

1. Backend em TypeScript/Node.js
2. CLI para executar tarefas do orquestrador
3. PostgreSQL como fonte de verdade para catálogo de serviços LLM, bindings por papel, telemetria de execução e orçamento operacional
4. API local administrativa para leitura de runs, catálogo de LLMs e bindings por papel
5. Frontend administrativo separado, com páginas para:
   - visualizar execuções
   - gerenciar catálogo de serviços LLM
   - gerenciar bindings por papel e ordem de fallback
6. Providers plugáveis para:
   - Gemini
   - DeepSeek
   - Groq
   - Ollama
7. Pipeline multiagente com pelo menos estes papéis:
   - Architect
   - Developer
   - Reviewer
   - Repair
8. Modo dry-run e modo apply
9. Saída estruturada em pasta de output com relatório e artefatos por execução

## Requisitos funcionais obrigatórios

### Pipeline

- O fluxo principal é: Architect -> Developer -> Reviewer
- Se o Reviewer rejeitar e ainda houver iterações disponíveis, o sistema deve voltar ao Developer com feedback estruturado
- Deve existir política de aprovação configurável, com pelo menos:
  - reviewer-veto
  - unanimous
  - majority

### Resolução de LLM por papel

- O sistema não deve depender de mapeamento de provider/model por papel em `.env`
- O `.env` deve guardar apenas segredos e parâmetros operacionais
- A seleção de serviço/modelo/fallback por papel deve vir do PostgreSQL
- Cada papel pode ter uma cadeia de fallback ordenada por prioridade
- O runtime deve resolver os bindings em tempo de execução a partir do banco

### Catálogo de serviços LLM

- Criar tabela para catálogo de serviços LLM contendo, no mínimo:
  - `id`
  - `slug`
  - `display_name`
  - `provider_kind`
  - `description`
  - `base_url`
  - `api_key_env_var`
  - `enabled`
  - capacidades como chat, tools, vision, json mode
  - `default_model`
  - `available_models_json`
  - dados de pricing
  - timeout
  - prioridade de catálogo
  - notes
  - metadata_json
  - timestamps

### Bindings por papel

- Criar tabela `role_provider_bindings`
- Cada binding precisa ter:
  - `id`
  - `agent_role`
  - `llm_service_id`
  - `priority`
  - `enabled`
  - timestamps
- Deve existir restrição de unicidade por `(agent_role, priority)`
- Deve ser possível listar e substituir todos os bindings de um papel dentro de transação

### Telemetria e orçamento

- Registrar runs e chamadas de LLM no PostgreSQL
- Cada chamada de LLM deve poder apontar para o `llm_service_id` que a originou
- Registrar orçamento operacional em BRL por run
- O runtime deve conseguir parar quando o orçamento exceder o limite configurado

### CLI

Crie um entrypoint com estes comportamentos:

- `--task <texto>`
- `--task-file <arquivo>`
- `--repo <caminho>`
- `--dry-run`
- `--apply`
- `--policy <nome>`
- `--max-iterations <n>`
- `--output <caminho>`
- `--api` para subir a API local administrativa
- `--help`

### Admin API

Implemente rotas locais para pelo menos:

- health check
- listar runs
- detalhar run
- listar serviços LLM
- criar serviço LLM
- editar serviço LLM
- remover serviço LLM com proteção contra remoção de serviço vinculado
- listar bindings por papel
- substituir bindings de um papel

### Frontend administrativo

Crie um frontend administrativo moderno e simples com páginas para:

- Execuções
- CRUDs
- Catálogo de serviços LLM
- Bindings por papel

Na tela de bindings por papel:

- listar os serviços disponíveis
- editar a ordem de fallback
- habilitar e desabilitar bindings
- salvar papel individualmente
- refletir a ordem real usada no runtime

## Arquitetura sugerida

Use uma arquitetura modular monolítica com estes blocos:

- `src/index.ts` para CLI e bootstrap
- `src/orchestrator/TaskOrchestrator.ts` como fachada principal
- `src/core/AgentLoop.ts` para o loop ReAct
- `src/providers/ProviderFactory.ts` para resolver providers a partir do banco
- `src/api/*` para o servidor local e repositórios administrativos
- `src/telemetry/*` para schema, conexão PostgreSQL, repositórios e scripts SQL
- `src/tools/*` para tools do loop de agentes
- `src/interfaces/*` para contratos TypeScript
- `.agents/prompts/*` para prompts modulares dos agentes
- `frontend/` para a interface administrativa

## Restrições importantes

- O projeto deve ser local-first
- Não usar Telegram, Discord ou chat UI como interface principal
- O runtime principal deve ser CLI + API local + frontend administrativo local
- O projeto deve ser escrito com TypeScript estrito
- Use escrita segura em disco no modo apply
- Não armazene segredos no banco
- Faça o catálogo referenciar segredos por nome de variável de ambiente

## Provedores

Implemente classes separadas para Gemini, DeepSeek, Groq e Ollama, todas aderindo à mesma interface de provider.

Requisitos adicionais:

- Ollama deve usar API compatível com OpenAI local
- DeepSeek e Groq devem aceitar `base_url` configurável a partir do catálogo
- O provider deve expor telemetria suficiente para custo e debugging

## Seeds e bootstrap

Crie scripts SQL ou CLI para:

- garantir o schema PostgreSQL
- popular dados de telemetria de exemplo se necessário
- popular um catálogo LLM inicial com ao menos:
  - Gemini premium
  - Gemini rápido
  - Ollama local
  - DeepSeek
  - Groq
- popular bindings padrão por papel

## Prompts dos agentes

Crie prompts iniciais, mas mantenha-os genéricos e editáveis.

- Architect: planejamento, leitura de contexto e decisões estruturais
- Developer: implementação, correções e propostas de patch
- Reviewer: auditoria, veto técnico e aprovação final
- Repair: segunda passagem orientada por feedback estruturado

Os prompts não devem assumir um domínio específico. Eles devem funcionar para qualquer repositório de software.

## Critérios de aceite

Considere a entrega concluída apenas quando:

1. o backend compilar
2. o frontend compilar
3. a API local responder health check
4. o catálogo de serviços LLM puder ser lido pela API
5. os bindings por papel puderem ser lidos e atualizados
6. o banner do runtime mostrar a cadeia resolvida do banco para architect, developer e reviewer
7. um dry-run completo gerar artefatos em `output/`
8. existir documentação para outro desenvolvedor rodar o projeto

## Resultado esperado no repositório

Entregue:

- código-fonte
- schema SQL
- scripts npm úteis
- `.env.example`
- prompts modulares em markdown
- README principal
- pasta de output funcional
- frontend administrativo

## Modo de execução esperado

O projeto final deve poder ser usado com comandos equivalentes a:

```bash
npm install
npm run build
npm run start:api
node dist/index.js --task "Adicionar endpoint de health check" --repo ./repo-alvo --dry-run
```

## Instrução final

Implemente o projeto inteiro com foco em clareza, modularidade e governança operacional. Não personalize o produto para um negócio específico. Preserve o nome HeroForge e deixe explícito que os prompts dos agentes podem ser trocados depois.