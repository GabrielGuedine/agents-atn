# AGENTS.md - Atena4 (Ruby on Rails)

Guia para agentes (humanos ou IA) atuarem com segurança e consistência no backend Rails do Atena4.

## 1) Escopo deste arquivo
- Este arquivo cobre **backend Ruby on Rails** do repositório.
- Para frontend legado AngularJS, seguir: `agents-docs/frontend/AGENTS.md`.

## 2) Objetivo do projeto
- Sistema de registro, tramitação, acompanhamento e controle de documentos/autos no MPGO.
- Monólito Rails 4 com APIs e páginas server-rendered que alimentam frontend AngularJS.

## 3) Stack e contexto técnico
- Ruby: `2.4.6`
- Rails: `4.2.11`
- Gems gerenciadas por Bundler via `mise` — comandos `bundle exec` requerem `mise` instalado e ativo no ambiente.
- Banco: PostgreSQL corporativo (múltiplos schemas)
- Fila: Resque / Resque Pool / Resque Scheduler
- Testes: RSpec (com suíte separada para integração Projudi)
- Qualidade: RuboCop

## 4) Regra crítica de banco de dados
- A estrutura de banco vem de `db/postgres_estrutura/db_corporativo.sql` (submódulo), não de migrations como fonte principal.
- `db:test:prepare` foi sobrescrito para restaurar estrutura via `bin/restore_schemas.sh`.
- Comandos principais:
  1. `bundle exec rake db:test:prepare`
  2. `FORCE_DB_RESTORE=true bundle exec rake db:test:prepare` (força recriação)
  3. `bundle exec rake db:test:force_prepare` (CI)
- Antes de tarefas de dados/schema, confirmar submódulo atualizado: `git submodule update --init --recursive`.
- NUNCA FAÇA INSERÇÕES NEM DELEÇÕES DIRETO NO BANCO

## 5) Organização técnica relevante
- Controllers: `app/controllers/**`
- Models: `app/models/**` (domínios como `Documento`, `Administrativo`, `Zeus`, `TribunalJustica`)
- Services/Interactors: `app/services/**`, `app/interactors/**`
- Workers: `app/workers/workers/**`
- Tasks operacionais: `lib/tasks/**` (há muitas tasks de manutenção/migração de dados)

## 6) Padrões de implementação esperados (Rails)
- O código backend DEVE seguir as convenções do RuboCop definidas no projeto (`.rubocop.yml`). Antes de concluir qualquer alteração, executar `bundle exec rubocop -A` para verificar e auto-corrigir ofensas.
- Preservar convenções e namespaces existentes do domínio (legado grande).
- Evitar refatorações amplas fora do escopo pedido; priorizar mudanças pontuais e seguras.
- Em integrações externas (Projudi, Elasticsearch, etc.), tratar exceções com mensagens úteis e logs.
- Para processamento assíncrono, seguir padrões de workers existentes (Resque).
- Não introduzir dependências modernas incompatíveis com Ruby 2.4 / Rails 4.2 sem alinhamento explícito.

## 7) Fluxo local recomendado (backend)
1. Ambiente:
   - `cp .env.development.template .env` (ou `.env.docker.template` para Docker)
   - `bundle install`
2. Frontend legado (necessário para app completo):
   - `bundle exec rake npm:install:clean`
   - `bundle exec rake bower:install:clean`
3. Banco de teste:
   - `bundle exec rake db:test:prepare`
4. Testes:
   - `make rspec`
   - `make rspec-file SPEC=spec/models/..._spec.rb`
   - `make rspec-projudi` (integração)

## 8) Validação mínima antes de concluir mudança
- Se alteração em Ruby puro/backend, validar com RuboCop nos arquivos alterados:
  - `bundle exec rubocop <arquivos_rb_alterados>`
- O agente DEVE rodar `bundle exec rspec` e `bundle exec rubocop` após qualquer alteração em arquivos de código ou testes. O ambiente (mise) já está configurado para tal — basta executar os comandos diretamente ou utilizar mise exec.

## 9) Convenções de CI/CD relevantes
- Pipeline GitLab roda RSpec (sem projudi_integration por padrão), RuboCop e Sonar.
- Test DB em CI é preparado via `db:test:force_prepare`.
- Testes de integração Projudi rodam separadamente (manual/schedule).

## 10) Limites e cuidados
- Não versionar segredos (`.env`, credenciais, tokens, cookies).
- Evitar mudanças massivas em `lib/tasks/**` sem necessidade: há tarefas operacionais críticas.
- Alterações em autenticação, sessões, permissões e rotas exigem validação extra.
- Não remover comportamento legado sem evidência de cobertura funcional.
- Sempre pré exibir o que será alterado.

## 11) Quando pedir confirmação humana
- Mudanças que afetam restauração/estrutura de banco corporativo.
- Alterações de contrato com sistemas externos (Projudi, Atena3, Elasticsearch).
- Refatorações amplas em módulos de domínio centrais (`Documento`, `TribunalJustica`, `Zeus`).
