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
cd "$(git rev-parse --show-toplevel 2>/dev/null || echo .)"
git add -N .

# Ver alterações
git diff HEAD --stat

# Backend
git diff HEAD -- app/controllers/ app/services/ app/models/ config/ app/errors/ app/helpers/ app/utils/ app/workers/

# Frontend
git diff HEAD -- app/assets/javascripts/ app/views/

# Testes
git diff HEAD -- spec/
```

## Contagem de itens (1 item = 1 unidade)

**Backend:** Novos métodos (`def`), métodos alterados, describes RSpec (ignorar `context`/`it`), relações em models, novas rotas. `config/application.rb` e `config/database.yml` também contam.

**Frontend:** Novos métodos AngularJS, métodos alterados, alterações .js/.js.erb, novos itens HTML.

**Componente:** Alterações de plugins/bibliotecas (ex.: `config/initializers/`, `lib/tasks/`).

## Classificação UST

Consultar o arquivo `./CATALOGO_ATIVIDADES.md` para as classificações disponíveis e os seus USTs equivalentes.

## Subdivisão (quando > 18 itens)

Preencher tarefas GG (18 itens) primeiro. Restante forma nova tarefa.
Algoritmo: `GG = floor(N / 18)`, resto = `N % 18`. Se resto = 0, não criar extra.

## Formato do arquivo de saída

SEMPRE use esse exato exemplo de template:

```
.agents/catalogo_tarefas_<branch>.txt
```

Conteúdo:
```
# Catálogo de Alterações

[Backend]
* caminho/arquivo.rb
# Criação do método "nome";
# Alteração do método "outro";``

[Componente]``
* Gemfile
# Adição/remoção de plugin/biblioteca;

[Frontend]
* caminho/arquivo.js
# Criação do método "nome";

## Classificação UST
Backend: N itens -> 1 GG (10-18 itens, 106 USTs)
Componente: N itens -> 1 componente (2 USTs)
Frontend: N itens -> conforme classificação
Total: X USTs
```
