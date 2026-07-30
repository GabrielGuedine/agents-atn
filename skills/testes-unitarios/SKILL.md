---
name: testes-unitarios
description: Criar e alterar testes RSpec cobrindo métodos privados via .send, seguindo os padrões do projeto, boas práticas Rails e RuboCop
compatibility: opencode
metadata:
  tipo: testing
---

## Objetivo

Criar e alterar testes RSpec no projeto seguindo:
1. Testar métodos privados utilizando `.send`
2. Padrões de teste já estabelecidos no projeto
3. Boas práticas Rails
4. Regras do RuboCop (`.rubocop.yml`)

## Quando utilizar

- Ao criar um novo arquivo de teste (`spec/**/*_spec.rb`)
- Ao adicionar testes para novos métodos (públicos ou privados)
- Ao alterar testes existentes para seguir os padrões do projeto

---

## 1. Testes de métodos privados com `.send`

Sempre testar métodos privados chamando-os diretamente via `object.send(:method_name, args)`.

```ruby
# RUIM — testar indiretamente via método público
it "extrai funcionario indiretamente" do
  subject.call
  expect(registro).to have_attributes(funcionario_id: 42)
end

# BOM — testar diretamente via .send
it "extrai funcionario_id da pessoa do usuario" do
  result = service.send(:extrair_funcionario_id, usuario)
  expect(result).to eq(42)
end
```

Estrutura do describe para métodos privados:
```ruby
RSpec.describe MeuModulo::MeuService, type: tipo_do_teste do
  subject(:service) { described_class.new(param1: valor1) }

  describe "#metodo_privado_1" do
    it "retorna valor esperado" do
      result = service.send(:metodo_privado_1, arg)
      expect(result).to eq(esperado)
    end

    context "quando ocorre erro" do
      it "loga warning e retorna nil" do
        expect(Rails.logger).to receive(:warn).with(/.../)
        result = service.send(:metodo_privado_1, arg_invalido)
        expect(result).to be_nil
      end
    end
  end
end
```

---

## 2. Padrões de teste do projeto 

### Estrutura básica do arquivo

```ruby
require "rails_helper"

RSpec.describe NomeDoModulo::NomeDaClasse, type: tipo_do_teste do
  # let(:nome) { valor }  — dados compartilhados
  # subject(:nome) { described_class.new(...).metodo }  — sujeito do teste

  describe "#metodo_publico" do
    # subject(:executar) { described_class.new(...).metodo }

    context "quando ..." do
      # before do ... end
      it "..." do
        # expect(...)
      end
    end
  end

  describe "#metodo_privado" do
    it "..." do
      result = objeto.send(:metodo_privado, args)
      expect(result).to eq(...)
    end
  end
end
```

### Nomes de context e describe em português

Seguindo a configuração do RuboCop (`RSpec/ContextWording`), usar `quando`, `com`, `sem`, `se`:

```ruby
context "quando o usuario existe" do
context "quando o login falha" do
context "com cookies validos" do
context "sem serventia selecionada" do
context "se o processo nao existe" do
```

### Subject e let

```ruby
# subject nomeado — padrão preferido
subject(:chamar_auditoria) { described_class.new(id_usuario: 1).call }

# let para dados compartilhados com FactoryBot
let(:usuario) { create(:usuario) }
let(:pessoa) { create(:pessoa, nm_pessoa: "João") }
let(:funcionario) { create(:funcionario, pessoa: pessoa) }

# allow/expect no before para setup comum
before do
  allow(Zeus::Usuario).to receive(:find_by).and_return(usuario)
  allow(TribunalJustica::Projudi::AuditoriaSincronizacaoService).to receive(:new).and_call_original
end
```

### Dados com FactoryBot

```ruby
# Criar registro persistido
let(:usuario) { create(:usuario) }
let(:pessoa) { create(:pessoa, nm_pessoa: "Maria") }

# Criar registro sem persistir
let(:auditoria) { build(:auditoria_projudi) }

# Associações
let(:funcionario) { create(:funcionario, pessoa: pessoa, tramita_documentos: true) }

# Stub de queries com dados reais
before do
  allow(Zeus::Usuario).to receive(:find_by).with(id_usuario: usuario.id_usuario).and_return(usuario)
end
```

### Expectativas de log

```ruby
it "loga warning quando nao encontra" do
  expect(Rails.logger).to receive(:warn).with(/nao encontrado/)
  subject
end

it "loga erro quando falha" do
  expect(Rails.logger).to receive(:error).with(/Falha ao processar/)
  subject
end
```

### Expectativas de erro

```ruby
it "lanca erro de autenticacao" do
  expect { subject }.to raise_error(MeuErro, /mensagem/)
end

it "nao propaga excecao" do
  expect { subject }.not_to raise_error
end
```

### Expectativas encadeadas (ordered)

```ruby
it "executa servico de auditoria antes da sincronizacao" do
  expect(servico_auditoria).to receive(:call).ordered
  expect(servico_builder).to receive(:call).ordered
  subject
end
```

### Matchers comuns do projeto

```ruby
expect(result).to be_a(Array)
expect(result).to be_present
expect(result).to be_nil
expect(result).to be_truthy / be_falsey
expect(result).to eq(valor)
expect(result).to match(/regex/)
expect(described_class).to receive(:find_or_create_by!).with(hash_including(campo: valor))
expect { ... }.not_to raise_error
expect { ... }.to change { Model.count }.by(1)
```

### Factories

```ruby
# spec/factories/dominio/meu_modelo.rb
FactoryBot.define do
  factory :meu_modelo, class: "MeuModulo::MeuModelo" do
    nome { "Exemplo" }
    association :relacao, factory: :outra_factory
  end
end

# Uso no spec
let(:registro) { build(:meu_modelo) }  # sem persistir
let(:registro) { create(:meu_modelo) } # persistido
```

---

## 3. Boas práticas Rails / RSpec

- `let` sobre variáveis de instância (`@var`)
- `subject` nomeado (`subject(:executar) { ... }`) para clareza
- `context` para cada estado/condição diferente
- Um `it` por comportamento (seguindo `RSpec/MultipleExpectations: Max: 5`)
- `before` para setup repetido entre `it` blocks
- `allow` para stubs, `expect(...).to receive` para spies
- Preferir `FactoryBot` (`create`/`build`) para criar dados de models reais
- Usar `build` (em vez de `create`) quando não precisar persistir
- **Para classes com respaldo em banco (ActiveRecord models), sempre usar `create`/`build` do FactoryBot — nunca `instance_double`.**
  ```ruby
  # RUIM
  let(:usuario) { instance_double(Zeus::Usuario, id: 1) }

  # BOM
  let(:usuario) { create(:usuario) }
  ```
- Usar `instance_double` / `double` apenas para objetos sem factory disponível (services, builders, POROs).
- Testar `Rails.logger` mensagens em cenários de erro/warning
- Usar `described_class` em vez de classe hardcoded

---

## 4. RuboCop (`.rubocop.yml`)

Regras ativas do projeto que o spec deve respeitar:

| Regra | Limite |
|-------|--------|
| `RSpec/MultipleExpectations` | Máx. 5 `expect` por `it` |
| `RSpec/ExampleLength` | Máx. 10 linhas por `it` |
| `RSpec/MultipleMemoizedHelpers` | Máx. 10 `let`/`subject` |
| `RSpec/NestedGroups` | Máx. 5 níveis de `describe`/`context` |
| `RSpec/ContextWording` | Prefixos: `when`, `with`, `without`, `if`, `unless`, `for`, `com`, `quando` |
| `RSpec/MessageSpies` | `EnforcedStyle: receive` (usar `expect(...).to receive`) |
| `RSpec/AnyInstance` | Não usar — exceto em `spec/support/authentication_helper.rb` |
| `RSpec/DescribeClass` | Desabilitado (pode descrever string se necessário) |
| `RSpec/IncludeExamples` | Habilitado (usar `it_behaves_like` quando aplicável) |

---

## 5. Validação obrigatória

Após criar ou alterar qualquer arquivo de teste, executar os comandos abaixo para garantir conformidade:

### RuboCop nos arquivos alterados

```bash
bundle exec rubocop -A spec/caminho/do/arquivo_spec.rb
```

Corrigir automaticamente ofensas com `-A`. Verificar se não restam ofensas.

### Execução dos testes

Rodar os specs alterados/criados para garantir que não quebram:

```bash
# Spec específico
bundle exec rspec spec/caminho/do/arquivo_spec.rb

# Subconjunto do diretório
bundle exec rspec spec/services/tribunal_justica/projudi/
bundle exec rspec spec/workers/workers/importacao_movimento/
bundle exec rspec spec/models/hermes/
```

Verificar se todos os exemplos passam (sem falhas nem pendências).

### Pipeline completo (recomendado antes de commit)

```bash
bundle exec rubocop -A
bundle exec rspec
```

---

## Template completo

```ruby
require "rails_helper"

RSpec.describe MeuModulo::MeuService, type: tipo_do_teste do
  subject(:service) { described_class.new(param1: valor1) }

  let(:usuario) { create(:usuario) }
  let(:auditoria) { build(:auditoria_projudi) }

  before do
    allow(Zeus::Usuario).to receive(:find_by).and_return(usuario)
  end

  describe "#call" do
    subject(:executar) { service.call }

    context "quando tudo ocorre com sucesso" do
      it "retorna resultado esperado" do
        expect(executar).to eq("ok")
      end

      it "chama dependencia" do
        expect(dependencia).to receive(:call)
        executar
      end
    end

    context "quando ocorre erro" do
      before do
        allow(usuario).to receive(:algo).and_raise(StandardError)
      end

      it "loga erro" do
        expect(Rails.logger).to receive(:error).with(/erro/)
        executar
      end

      it "retorna nil" do
        expect(executar).to be_nil
      end
    end
  end

  describe "#metodo_privado" do
    it "retorna valor processado" do
      resultado = service.send(:metodo_privado, arg1: "teste")
      expect(resultado).to eq("TESTE")
    end

    context "quando argumento é nil" do
      it "retorna nil" do
        resultado = service.send(:metodo_privado, arg1: nil)
        expect(resultado).to be_nil
      end
    end
  end
end
```
