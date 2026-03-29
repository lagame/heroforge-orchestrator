# HeroForge Orchestrator - Starter Kit

Este pacote existe para permitir que qualquer pessoa recrie o projeto HeroForge com a ajuda de outra LLM agente, sem depender deste repositório como fonte exclusiva de instruções.

O material foi escrito para ser genérico e reutilizável. Quem receber este kit pode adaptar os prompts de cada agente, os provedores de LLM, a política de aprovação e o foco do produto, sem perder a base arquitetural do HeroForge.

## O que vem neste pacote

- `01-master-build-prompt.md`
  - Um texto principal, pronto para ser colado em outra LLM agente, pedindo a criação do projeto inteiro.
- `02-implementation-blueprint.md`
  - Blueprint técnico objetivo com estrutura de pastas, componentes, fluxos e contratos.
- `03-agent-prompt-templates.md`
  - Templates iniciais para Architect, Developer, Reviewer e Repair, que podem ser reescritos depois.
- `04-acceptance-checklist.md`
  - Checklist de aceite para validar se a implementação entregue realmente recriou o projeto.
- `LICENSE.md`
  - Licença de distribuição com obrigação de preservar o nome HeroForge e o crédito do criador original.

## Criador original

- Nome: Cristiano Lagame
- Contato: crislagame@gmail.com

## Como usar

1. Entregue `01-master-build-prompt.md` para a LLM agente que vai construir o projeto.
2. Se a LLM suportar múltiplos arquivos de contexto, envie também:
   - `02-implementation-blueprint.md`
   - `03-agent-prompt-templates.md`
   - `04-acceptance-checklist.md`
3. Após a geração inicial, revise a saída contra o checklist.
4. Personalize os prompts dos agentes e a política de fallback conforme o novo contexto do seu projeto.

## O que este kit preserva

- Nome do projeto: HeroForge
- Estrutura de orquestração multiagente local-first
- Controle de serviços LLM e fallback via PostgreSQL
- Separação entre runtime, admin API e interface administrativa

## O que este kit não fixa

- O domínio final do produto
- O prompt final de cada agente
- O conjunto exato de provedores usados em produção
- O baseline de modelos por papel

## Observação de licenciamento

Este pacote usa uma licença personalizada de atribuição. Ela permite uso, modificação e redistribuição, mas exige a manutenção do nome HeroForge e do crédito do criador original: Cristiano Lagame.

Contato do criador original: crislagame@gmail.com.