---
name: code-review
description: Revisão de código como líder técnico sênior — analisa alterações via git diff, avalia qualidade, segurança, regras de negócio e autorização
compatibility: opencode
metadata:
  tipo: code-review
---

## Objetivo

Realizar code review das alterações do projeto como um líder técnico sênior. Analisar o `git diff` completo, avaliar a qualidade do código, segurança, performance, aderência aos padrões do projeto, regras de negócio e autorização. Produzir parecer estruturado com pontos positivos, pontos de atenção, blockers e sugestões.

## Quando utilizar

Quando o usuário solicitar code review, revisão de código, análise de alterações, pull request review, ou comandos equivalentes como "revise as alterações", "faça code review", "analise o diff", "review das mudanças".

## Comandos git

```bash
# Preparar ambiente
cd "$(git rev-parse --show-toplevel 2>/dev/null || echo .)"
git add -N .  # Arquivos novos aparecem no diff

# Ver alterações
git diff HEAD --stat
git diff HEAD

# Backend
git diff HEAD -- app/controllers/ app/services/ app/models/ app/interactors/ config/ app/errors/ app/helpers/ app/utils/ app/workers/ lib/ spec/

# Frontend AngularJS
git diff HEAD -- app/assets/javascripts/angularjs/ app/views/
```

## Checklist de revisão técnica

### Boas práticas Rails 4.2 / Ruby 2.4
- Código compatível com Ruby 2.4.6 e Rails 4.2.11 — não usar features do Ruby 3+ ou Rails 5+ (ex.: `compact`, `then`, `yield_self`, `**options` em Ruby 3)
- Namespacing explícito por domínio (`Documento::*`, `Administrativo::*`, `Zeus::*`, `TribunalJustica::*`)
- Controllers enxutos — lógica de negócio em services/interactors, não no controller
- Preservar convenções existentes do domínio legado — não refatorar fora do escopo
- Uso de `before_filter`/`around_filter` (Rails 4) em vez de `before_action` quando seguir padrão existente
- `respond_to` com formatos HTML e JSON conforme padrão do projeto

### Segurança
- Brakeman não introduz ofensas novas
- Sem hardcode de credenciais, senhas, tokens, cookies
- CSRF ativo (`protect_from_forgery`) — verificar se rotas que pulam (`skip_before_action :verify_authenticity_token`) têm justificativa
- Sanitização de parâmetros para evitar SQL injection (`sanitize_sql_like`)
- Sessão: `control_filter`, `checa_sessao` respeitados
- `usuario_logado` verificado antes de ações sensíveis
- Verificação de senha em operações críticas (digitalizar_autos, arquivar, desarquivar)
- API key (`X-CHAVE-API`) validada corretamente com `CoreConfig.chave_api`
- Logs não expõem dados sensíveis (cookies, tokens, senhas)

### Performance
- N+1 queries: `includes`/`eager_load` presentes em consultas com associações
- `upsert_all` usado corretamente em operações em lote (evitar inserts únicos em loop)
- Uso consciente de Redis/Resque — evitar jobs que processam dados em memória sem paginação
- Queries com `LIKE` usam índices ou são limitadas
- Consultas em views e controllers sem carregamento desnecessário de dados

### Manutenibilidade
- Métodos com complexidade ciclomática baixa (RuboCop)
- Nomes descritivos de métodos, variáveis e classes
- Sem código morto ou comentado
- Duplicação mínima — extrair lógica repetida para métodos privados ou services
- Baixo acoplamento entre controller e integração externa
- Não remover comportamento legado sem evidência de cobertura funcional

### Testes
- Cobertura mínima para novos métodos públicos
- Testes de métodos privados com `.send`
- Testes de integração com VCR para chamadas externas
- Testes de workers com Resque inline ou mocked
- Logs de erro/warning testados com `expect(Rails.logger).to receive`
- `bundle exec rspec` passa sem falhas

## Regras de negócio

### 1. Idempotência e deduplicação
- Operações de importação em lote usam `upsert_all` ou `find_or_create_by!` (não `create!` direto)
- Execução repetida do mesmo job/importação não gera duplicatas nem efeitos colaterais
- Workers consideram cenário de reprocessamento (retry Resque)

### 2. Integridade de integrações externas
- Chamadas a sistemas externos (Projudi, Elasticsearch, Atena3, webservices) tratam resposta malformada sem propagar exceção
- Mapeamento de dados de terceiros preserva precisão (datas, números, textos grandes)
- Dados opcionais de fontes externas não quebram a importação se ausentes
- Tratamento de encoding e caracteres especiais em integrações

### 3. Tratamento de falhas de integração
- Toda chamada HTTP externa tem rescue com log e retry previsível
- Timeout de conexão tratado com mensagem clara
- HTTP 500 do provedor registrado e não quebra o fluxo principal
- Integrações com Elasticsearch tratam índice indisponível

### 4. Sessão e autenticação do usuário
- Sessão gerenciada via `session[:e_perfil]` e `usuario_logado`
- Login/logout com reset de sessão adequado
- Perfil XML (`ZeusEntidades::EPerfil`) preservado na sessão sem exposição
- CSRF token sincronizado entre Rails e AngularJS (`set_csrf_cookie_for_ng`)

### 5. Consistência cross-schema
- Referências entre schemas mantêm integridade (joins entre `Documento.*`, `Administrativo.*`, `Zeus.*`)
- Alterações em um schema não quebram queries em outro
- Estrutura do banco respeita `db/postgres_estrutura/db_corporativo.sql` — não criar migrations sem alinhamento

### 6. Rastreabilidade (audit trail)
- `Rails.logger` cobre erros e sucessos relevantes (não apenas falhas)
- Operações de tramitação, remessa e movimentação registradas
- `ExceptionLogger::ExceptionLoggable` capturando exceções não tratadas

### 7. Concorrência em workers Resque
- Workers podem executar em paralelo sem condição de corrida
- Uso de `unique` jobs ou locks quando necessário
- `find_or_create_by!` preferido a `create!` em operações concorrentes
- Resque Pool e Resque Scheduler respeitam limites de fila

### 8. Regras de domínio jurídico e documental
- Documentos/autos seguem fluxo de tramitação, remessa e ciência
- Arquivos/anexos têm validade vinculada à movimentação ou ao documento
- Numeração de processos formatada conforme padrão judicial
- Movimentações respeitam ordenação e hierarquia de andamentos
- Sigilo e permissões de acesso respeitados por tipo de documento

### 9. Preservação de dados fonte
- Dados de integrações externas preservados sempre que possível para auditoria
- Transformações não descartam informação original sem necessidade
- XML/JSON de retorno de integrações registrados para debugging futuro

### 10. Autorização e controle de acesso
- `control_filter` e `checa_sessao` aplicados nas rotas protegidas
- API key (`X-CHAVE-API`) validada com `CoreConfig.chave_api` quando utilizada
- Ações sensíveis (arquivar, desarquivar, digitalizar autos) exigem verificação de senha
- `authenticate_or_request_with_http_digest` usado em rotas específicas de consolidados
- Perfil do usuário respeitado para funcionalidades restritas

### 11. AngularJS frontend
- `//= require` adicionado no manifesto correto ao criar novo controller/factory/service/directive
- Ordem de includes mantida em `_javascripts.html.erb` e manifests
- ControllerAs pattern seguido (`$scope` evitado onde não padronizado)
- Serviços/factories existentes reutilizados antes de criar novos pontos HTTP
- `.js.erb` com interpolação Rails sem quebrar JS gerado
- Novos estados `$stateProvider` registrados em `main.js.erb` com `templateUrl` via `asset_path`

### 12. Compatibilidade com legado
- Código não utiliza gems modernas incompatíveis com Ruby 2.4 / Rails 4.2
- Não introduz padrões modernos incompatíveis com AngularJS 1.5
- Asset pipeline (Sprockets) respeitado — sem Webpack/Vite
- Bower mantido como gerenciador de dependências frontend

## Formato do arquivo de saída

SEMPRE use este padrão:

```
.agents/code_review/<timestamp>_code_review_<branch>.txt
```

Onde `<timestamp>` é o horário atual no formato `YYYYMMDD_HHMM` e `<branch>` é o nome da branch atual (sem caracteres especiais, substituir `/` por `-`).

### Versionamento automático

Se já existir um arquivo com o mesmo nome, o timestamp garante que cada execução gere um arquivo novo. **Nunca substituir** um arquivo existente.

### Template de conteúdo

```
# Code Review — <branch>

**Data:** <timestamp>
**Commit:** <hash do commit atual>
**Arquivos alterados:** <stat resumido>

## Resumo das alterações

<descrição concisa do que foi alterado>

## Análise por arquivo

### <caminho/arquivo.rb>

**Pontos positivos** ✅
- <item>

**Pontos de atenção** ⚠️
- <item>

**Sugestões** 💡
- <item>

---

## Regras de negócio

### 1. Idempotência e deduplicação
✅ / ⚠️ / ❌ <justificativa>

### 2. Integridade de integrações externas
✅ / ⚠️ / ❌ <justificativa>

### 3. Tratamento de falhas de integração
✅ / ⚠️ / ❌ <justificativa>

### 4. Sessão e autenticação do usuário
✅ / ⚠️ / ❌ <justificativa>

### 5. Consistência cross-schema
✅ / ⚠️ / ❌ <justificativa>

### 6. Rastreabilidade (audit trail)
✅ / ⚠️ / ❌ <justificativa>

### 7. Concorrência em workers Resque
✅ / ⚠️ / ❌ <justificativa>

### 8. Regras de domínio jurídico e documental
✅ / ⚠️ / ❌ <justificativa>

### 9. Preservação de dados fonte
✅ / ⚠️ / ❌ <justificativa>

### 10. Autorização e controle de acesso
✅ / ⚠️ / ❌ <justificativa>

### 11. AngularJS frontend
✅ / ⚠️ / ❌ <justificativa>

### 12. Compatibilidade com legado
✅ / ⚠️ / ❌ <justificativa>

## Blocker

> Listar aqui qualquer impedimento que impeça o merge. Se não houver, manter como "Nenhum blocker identificado."

## Resultado final

[ ] **Approve** — código aprovado sem ressalvas
[ ] **Approve with suggestions** — aprovado, mas sugestões devem ser consideradas
[ ] **Request changes** — necessidade de alterações antes do merge

## Checklist do revisor

- [ ] RuboCop ok
- [ ] Testes existentes não quebram
- [ ] Cobertura de testes adequada
- [ ] Sem hardcode de credenciais
- [ ] CSRF e sessão corretos
- [ ] Integrações externas tratadas com resiliência
- [ ] Manifestos AngularJS atualizados (se aplicável)
- [ ] Compatível com Ruby 2.4 / Rails 4.2
```
