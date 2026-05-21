# AGENTS.md - Atena4 (AngularJS)

Guia para agentes (humanos ou IA) atuarem no frontend legado AngularJS do Atena4.

## 1) Escopo deste arquivo
- Escopo: `app/assets/javascripts/angularjs/**` e templates associados.

## 2) Stack frontend real deste projeto
- AngularJS `1.5.x` (legado)
- ui-router, angular-resource, angular-translate, ui-select, ng-file-upload, etc.
- Bower + Asset Pipeline (Sprockets), sem bundler moderno (Webpack/Vite)
- Muitos arquivos `.js.erb` com interpolação via Rails (`asset_path`, ENV, etc.)

## 3) Arquitetura e pontos de entrada
- Módulo principal: `angularjs/main.js.erb` (`angular.module('Atena', ...)`)
- Manifests por domínio:
  - `angularjs/controllers/AtenaControllers.js`
  - `angularjs/factories/AtenaFactories.js`
  - `angularjs/services/AtenaServices.js`
- Ordem de includes do app vem de `app/views/layouts/_javascripts.html.erb` e manifests:
  - `deps.js`, `angular-core.js`, `angular-deps.js`, `application.js`, etc.

## 4) Regra crítica para adicionar arquivos AngularJS
- Ao criar controller/factory/service/directive/filter, adicionar `//= require ...` no manifesto correto.
- Se esquecer o `require`, o arquivo não entra no bundle e a funcionalidade quebra em runtime.
- Se incluir nova lib de terceiros:
  1. atualizar `Bowerfile` (quando aplicável)
  2. atualizar manifest de dependências (`angular-core.js`, `angular-deps.js` ou `deps.js`)
  3. garantir ordem correta de carga.

## 5) Convenções de código AngularJS no projeto
- Preferir padrão já existente de `controllerAs` nos states/controls.
- Manter nomes e organização por domínio (pastas `controllers/<dominio>`, `factories/<dominio>`, etc.).
- Reusar serviços/factories existentes antes de criar novos pontos de acesso HTTP.
- Evitar introduzir padrões modernos incompatíveis com AngularJS 1.5.
- Em `.js.erb`, tomar cuidado com interpolação de Rails para não quebrar JS gerado.

## 6) Rotas/estados e templates
- Estados ficam majoritariamente em `main.js.erb` com `$stateProvider`.
- `templateUrl` usa `asset_path`; sempre manter caminho consistente com `angularjs/templates/**`.
- Ao alterar estado, validar navegação, breadcrumb e parâmetros (`params`) impactados.

## 7) CSS/visual
- Estilos em `app/assets/stylesheets/**` (SASS legado).
- Evitar mexer em assets compilados/copiados de vendor sem necessidade.
- Manter mudanças de estilo estritamente no escopo da funcionalidade solicitada.

## 8) Como validar mudanças frontend
- Subir aplicação e validar fluxo manualmente no navegador (AngularJS legado depende muito de runtime).
- Conferir console JS (erros de injeção, módulo não encontrado, template não encontrado).
- Validar telas/rotas impactadas e chamadas HTTP correspondentes.
- Quando mudança depender do backend, rodar pelo menos os testes Rails impactados.

## 9) Limites e cuidados
- Não migrar arquitetura frontend (ex.: Angular moderno/React) dentro de tarefas pontuais.
- Não alterar ordem global de includes em `_javascripts.html.erb` sem necessidade clara.
- Evitar mudanças abrangentes em manifests gigantes; editar apenas blocos necessários.

## 10) Quando pedir confirmação humana
- Inclusão/substituição de bibliotecas JS de base.
- Mudança estrutural em `main.js.erb` que impacte muitas rotas.
- Alterações de comportamento global de autenticação/sessão no frontend.
