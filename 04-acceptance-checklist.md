# HeroForge Acceptance Checklist

Use este checklist para validar se outra LLM agente realmente recriou o projeto.

## Backend

- [ ] Existe CLI funcional com `--task`, `--repo`, `--dry-run`, `--apply`, `--api` e `--help`
- [ ] O backend compila sem erros
- [ ] O runtime gera artefatos em `output/`
- [ ] Há escrita segura em modo apply

## PostgreSQL

- [ ] Existe schema SQL com catálogo de serviços e bindings por papel
- [ ] Há script de seed ou bootstrap para catálogo inicial
- [ ] O runtime resolve a cadeia de providers a partir do banco
- [ ] O `.env` não é a fonte de verdade de role/model/fallback

## Providers

- [ ] Existem implementações separadas para Gemini, DeepSeek, Groq e Ollama
- [ ] Todos implementam a mesma interface
- [ ] O fallback por prioridade funciona por papel

## Admin API

- [ ] Existe rota de health check
- [ ] Existe CRUD do catálogo de serviços LLM
- [ ] Existe leitura e substituição de bindings por papel
- [ ] Há proteção contra remoção de serviço ainda vinculado

## Frontend admin

- [ ] Existe tela de execuções
- [ ] Existe tela de catálogo de serviços LLM
- [ ] Existe tela de bindings por papel
- [ ] A tela de bindings mostra a ordem real do runtime

## Governança e operação

- [ ] Existe `.env.example`
- [ ] Existe documentação suficiente para subir o projeto
- [ ] Os prompts dos agentes estão em arquivos editáveis
- [ ] O nome HeroForge foi preservado

## Evidência final desejável

- [ ] Um dry-run real foi executado com sucesso
- [ ] A API admin respondeu corretamente
- [ ] O frontend admin carregou dados reais