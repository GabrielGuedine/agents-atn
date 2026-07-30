---
name: catalogar-alteracoes
description: Catalogar alterações do projeto via git para medição de UST (Unidade de Serviço Técnico) — classificação P/M/G/GG por categoria backend/frontend/componente
compatibility: opencode
metadata:
  tipo: metricas
---

## Objetivo

Catalogar alterações do projeto para medição de UST usando git. Considere **apenas** alterações do projeto atual.

## Quando utilizar

Utilize esta skill quando o usuário pedir para catalogar as atividades realizadas no projeto, incluindo comandos como "catalogue as alterações", "gere o catálogo de atividades", "meça UST", "conte os itens de backend/frontend/componente" ou equivalentes.

## Comandos git

```bash
# Preparar ambiente
cd "$(git rev-parse --show-toplevel 2>/dev/null || echo .)"
git add -N .  # Arquivos novos aparecem no diff

# Ver alterações
git diff HEAD --stat
git diff HEAD

# Backend: controllers, services, models, config, errors, helpers, utils, workers, tasks, initializers, e describes de testes spec
git diff HEAD -- app/controllers/ app/services/ app/models/ config/ app/errors/ app/helpers/ app/utils/ app/workers/ spec/

# Frontend: AngularJS e views
git diff HEAD -- app/assets/javascripts/ app/views/
```

## Contagem de itens (1 item = 1 unidade)

**Backend:** 
- Novos métodos (`def`)
- Métodos alterados
- Novos Describes RSpec (ignorar `context`, `it`, e a linha `RSpec.describe NomeDaClasse, type: :tipo_do_teste`)
- SEMPRE VERIFIQUE: **Describe existente com alterações:** Se um bloco `describe` já existia e houve qualquer alteração dentro dele (novos `it`, `context`, mudanças em `let`/`before`/`after`/etc.), contabilizar como "Alteração no describe do método 'método'"
- Relações em models
- Novas rotas
- `config/application.rb` e `config/database.yml` também contam
- Alterações em `spec/**/*.rb` referentes a describes também contam como backend.

**Frontend:** 
- Novos métodos AngularJS
- Métodos alterados
- Alterações .js/.js.erb
- Novos itens HTML.

**Componente:** 
- Alterações de plugins/bibliotecas (ex.: `config/initializers/`, `lib/tasks/`).

## Classificação UST

Consultar o arquivo `./CATALOGO_ATIVIDADES.md` para as classificações disponíveis e os seus USTs equivalentes.

## Subdivisão (quando > 18 itens)

Preencher tarefas GG (18 itens) primeiro. Restante forma nova tarefa.
Algoritmo: `GG = floor(N / 18)`, resto = `N % 18`. Se resto = 0, não criar extra.

## Formato do arquivo de saída

SEMPRE use este padrão:

```
.agents/catalogo_tarefas/<timestamp>_catalogo_tarefas_<branch>.txt
```

Onde `<timestamp>` é o horário atual no formato `YYYYMMDD_HHMM` e `<branch>` é o nome da branch atual (sem caracteres especiais, substituir `/` por `-`).

### Versionamento automático

Se já existir um arquivo com o mesmo nome, o timestamp garante que cada execução gere um arquivo novo. **Nunca substituir** um arquivo existente.

### Template de conteúdo
```
# Catálogo de Alterações

[Backend]
- caminho/arquivo.rb
1. Criação do método "nome";
2. Alteração do método "outro";``

- spec/caminho/arquivo.rb
1. Criação do teste describe "nome";
2. Alteração no describe do método "metodo_x";``

[Componente]``
- Gemfile
1. Adição/remoção de plugin/biblioteca;

[Frontend]
- caminho/arquivo.js
1. Criação do método "nome";

## Classificação UST
Backend: N itens -> 1 GG (10-18 itens, 106 USTs)
Componente: N itens -> 1 componente (2 USTs)
Frontend: N itens -> conforme classificação
Total: X USTs

## Sugestão de commit

Com base nas alterações catalogadas, gere um título conciso e uma lista das principais alterações. Use português, voz ativa e imperativo (ex.: "Cria", "Adiciona", "Refatora").

Formato:
```
## Sugestão de commit

Título: <frase curta explicando o propósito>

Principais alterações:
- <alteração relevante 1>
- <alteração relevante 2>
- <alteração relevante 3>
- ...


## Preenchimento do Merge Request Template

Após gerar o catálogo, preencha o template de MR disponível em `./MERGE_REQUEST_TEMPLATE.md` e **anexe-o ao final** do arquivo de catálogo gerado (`.agents/catalogo_tarefas/<timestamp>_catalogo_tarefas_<branch>.txt`).

Regras de preenchimento:

1. **Issues relacionadas** — Vincular a issue/tarefa principal. Se houver múltiplas, listar abaixo.
2. **Descrição** — Extrair o título da issue ou do commit como título. Preencher os tópicos com base na Sugestão de commit gerada.
3. **O que foi feito?** — Listar cada alteração relevante do catálogo, agrupada por arquivo.
4. **Alterações no banco de dados** — Se houver criação/alteração de tabelas, colunas, índices ou schemas, descrever. Caso contrário, manter como "-".
5. **Como testar?** — Descrever passos funcionais. Se houver worker, incluir comando para executá-lo manualmente. Se houver testes, incluir `bundle exec rspec <caminho>`.
6. **Evidências** — Deixar seção vazia para preenchimento manual.
7. **Notas adicionais** — Incluir observações sobre dependências, migrações, necessidade de rebuild, etc.
8. **Fora do escopo** — Listar o que não foi contemplado nas alterações.

Os itens do catálogo (Backend, Frontend, Componente) e a classificação UST devem ser mantidos **antes** do template preenchido.

