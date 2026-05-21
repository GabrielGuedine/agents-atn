# Instruções para Geração de Catálogo de Alterações

## Objetivo
Catalogar alterações do projeto atual para medição de UST usando git.

## Escopo
Considere **apenas alterações do projeto onde este arquivo está localizado**.
```bash
git rev-parse --show-toplevel  # Confirma diretório raiz
```
- Use '' ou "" nos txt (não use ``)
- Não misture projetos diferentes

## Comandos Git
```bash
# Preparar ambiente
cd "$(git rev-parse --show-toplevel 2>/dev/null || echo .)"
git add -N .  # Arquivos novos aparecem no diff

# Ver alterações
git diff HEAD --stat
git diff HEAD

# Backend: controllers, services, models, config, errors, helpers, utils, workers, tasks, initializers
git diff HEAD -- app/controllers/ app/services/ app/models/ config/ app/errors/ app/helpers/ app/utils/ app/workers/

# Frontend: AngularJS e views
git diff HEAD -- app/assets/javascripts/ app/views/

# Testes
git diff HEAD -- spec/
```

## Contagem de Itens (1 item = 1 unidade)

**Backend:** Novos métodos (`def`), métodos alterados, describes RSpec (ignorar `context`/`it`), relações em models, novas rotas. Alterações em `config/application.rb` e `config/database.yml` também contam como backend.

**Frontend:** Novos métodos AngularJS (`ctrl.metodo = function()`), métodos alterados, alterações .js/.js.erb, novos itens HTML.

**Componente:** Apenas alterações referentes a plugins/bibliotecas (ex.: `config/initializers/redis.rb`, `lib/tasks/resque.rake`).

**Contagem:** Listar arquivos por categoria → contar itens → somar separadamente (backend vs frontend vs componente).

## Classificação UST

Consultar o arquivo `agents-docs/CATALOGO_ATIVIDADES.md` para as classificações disponíveis e os seus USTs equivalentes.

## Subdivisão (quando > 18 itens)

Preencher tarefas GG (18 itens) primeiro. Restante forma nova tarefa classificada conforme tabela acima.

Ex: 20 itens → 1 GG (18) + 1 P (2) | 40 itens → 2 GG (36) + 1 P (4)

Algoritmo: `GG = floor(N/18)`, resto = `N % 18`. Se resto = 0, não criar tarefa extra.

## Formato do Arquivo .txt
```
# Catálogo de Alterações

[Backend]
* caminho/arquivo.rb
# Criação do método "nome";
# Alteração do método "outro";

* caminho/outro_arquivo.rb
# Criação do método "novo";

[Componente]
* Gemfile
# Adição/remoção de plugin/biblioteca;

* config/initializers/exemplo.rb
# Criação de configuração;

[Frontend]
* caminho/arquivo.js
# Criação do método "nome";
# Alteração do resource "meu_resource";

* caminho/outro_arquivo.html
# Criação de novo item HTML;

## Classificação UST
Backend: N itens -> 1 GG (10-18 itens, 106 USTs) (ou classificação conforme tabela)
Componente: N itens -> 1 componente (2 USTs)
Frontend: N itens -> conforme classificação frontend
Total: X USTs

```

## Execução
1. `git add -N .`
2. `git diff HEAD --stat` e analisar mudanças por categoria
3. Contar itens, aplicar subdivisão se > 18, classificar (P/M/G/GG)
4. Criar `agents-docs/catalogo_tarefas_$(git branch --show-current).txt`
5. `git add . && git restore --staged .` (pós-execução)

## Arquivos Gerados
- `agents-docs/catalogo_tarefas_<branch>.txt` - Catálogo das tarefas
- `CATALOGAR.md` - Este arquivo