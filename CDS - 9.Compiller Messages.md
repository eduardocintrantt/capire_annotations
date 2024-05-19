### Mensagens de Erro Comuns no CDS
Aqui estão algumas mensagens de erro comuns e suas explicações, além de como corrigi-las.

#### `anno-duplicate-unrelated-layer`
**Descrição:** Uma anotação é atribuída várias vezes através de camadas não relacionadas.

**Exemplo:**
```cds
// Base.cds
entity FooBar {  }

// FooAnnotate.cds
using from './Base';
annotate FooBar with @Anno: 'Foo';

// BarAnnotate.cds
using from './Base';
annotate FooBar with @Anno: 'Bar';

// All.cds
using from './FooAnnotate';
using from './BarAnnotate';
```

**Correção:** Remova uma das anotações duplicadas ou adicione uma anotação em `All.cds` que tenha precedência.
```cds
// All.cds
using from './FooAnnotate';
using from './BarAnnotate';
annotate FooBar with @Anno: 'Bar';
```

#### `anno-missing-rewrite`
**Descrição:** Uma anotação propagada contendo expressões não pode ser reescrita e resultaria em caminhos inválidos.

**Exemplo:**
```cds
type T : {
    @anno: (sibling)
    elem: String;
    sibling: String;
};
type TString : T:elem; // ❌ não há `sibling`
```

**Correção:** Substitua a anotação explicitamente.
```cds
type TString : T:elem;
annotate TString with @(anno: null);
```

#### `check-proper-type-of`
**Descrição:** Um elemento em uma expressão de tipo não possui informações de tipo adequadas.

**Exemplo:**
```cds
entity Foo {
  key id : Integer;
};
view ViewFoo as select from Foo {
  1+1 as calculatedField @(anno)
};
entity Bar {
  e : ViewFoo:calculatedField; // ❌ `e` não tem tipo adequado
};
```

**Correção:** Atribua um tipo explícito a `calculatedField`.
```cds
view ViewFoo as select from Foo {
  1+1 as calculatedField @(anno) : Integer
};
```
#### `def-duplicate-autoexposed`
**Descrição:** Duas ou mais entidades com o mesmo nome não podem ser expostas automaticamente no mesmo serviço.

**Exemplo:**
```cds
entity ns.first.Foo {
  key parent : Association to one ns.Base;
};
entity ns.second.Foo {
  key parent : Association to one ns.Base;
};
entity ns.Base {
  key id    : UUID;
  to_first  : Composition of many  ns.first.Foo;
  to_second : Composition of many ns.second.Foo;
}
service ns.MyService {
  entity BaseView as projection on ns.Base; // ❌
};
```

**Correção:** Exponha explicitamente uma ou mais entidades com um nome único.
```cds
service ns.MyService {
  entity first.Foo as projection on ns.first.Foo;
  entity second.Foo as projection on ns.second.Foo;
}
```

#### `def-missing-type`
**Descrição:** Um artefato de tipo não possui informações de tipo adequadas.

**Exemplo:**
```json
{
  "definitions": {
    "MainType": {
      "kind": "type"
    }
  }
}
```

**Correção:** Adicione informações de tipo explícitas a `MainType`.
```json
{
  "definitions": {
    "MainType": {
      "kind": "type",
      "elements": {
        "id": {
          "type": "cds.String"
        }
      }
    }
  }
}
```

#### `extend-repeated-intralayer`
**Descrição:** A ordem dos elementos de um artefato pode não ser estável devido a múltiplas extensões na mesma camada.

**Exemplo:**
```cds
// Definition.cds
using from './Extension.cds';
entity FooBar { };
extend FooBar { foo: Integer; }; // ❌

// Extension.cds
using from './Definition.cds';
extend FooBar { bar: Integer; }; // ❌
```

**Correção:** Mova as extensões para o mesmo bloco de extensão.
```cds
// Extension.cds
using from './Definition.cds';
extend FooBar {
  foo : Integer;
  bar : Integer;
}
```

#### `redirected-to-ambiguous`
**Descrição:** O alvo redirecionado origina-se mais de uma vez do alvo original através de fontes diretas ou indiretas do alvo redirecionado.

**Exemplo:**
```cds
entity Main {
  key id : Integer;
  toTarget : Association to Target;
}
entity Target {
  key id : Integer;
}
view View as select from Main, Target, Target as Duplicate {
  Main.toTarget : redirected to View; // ❌
}
```

**Correção:** Garanta que o alvo original esteja presente apenas uma vez nas fontes diretas e indiretas.
```cds
view View as select from Main, Target {
  Main.toTarget : redirected to View;
}
```

#### `rewrite-not-supported`
**Descrição:** O compilador não consegue reescrever condições ON para algumas associações. Elas precisam ser definidas explicitamente pelo usuário.

**Exemplo:**
```cds
entity Base {
  key id     : Integer;
  primary    : Association to Primary on primary.id = primary_id;
  primary_id : Integer;
}
entity Primary {
  key id       : Integer;
  secondary    : Association to Secondary on secondary.id = secondary_id;
  secondary_id : Integer;
}
entity Secondary {
  key id : Integer;
  text   : LargeString;
}
entity View as select from Base {
  id,
  primary.secondary // ❌
};
```

**Correção:** Forneça uma condição ON explícita.
```cds
entity View as select from Base {
  id,
  primary.secondary_id,
  primary.secondary: redirected to Secondary on
    secondary.id = secondary_id
};
```

#### `syntax-expecting-unsigned-int`
**Descrição:** O compilador espera um número inteiro não negativo seguro.

**Exemplo:**
```cds
type LengthIsUnsafe : String(9007199254740992); // ❌
type NotAnInteger : String(42.1);               // ❌
```

**Correção:** Forneça um número inteiro seguro.
```cds
type LengthIsSafe : String(9007199254740991);
type AnInteger : String(42);
```

#### `type-missing-enum-value`
**Descrição:** Uma definição de enum está faltando valores explícitos para uma ou mais de suas entradas.

**Exemplo:**
```cds
entity Books {
  category: Integer enum {
    Fiction; // ❌
    Action;  // ❌
  } default #Action;
};
```

**Correção:** Atribua valores explícitos ao enum.
```cds
entity Books {
  category: Integer enum {
    Fiction = 1;
    Action = 2;
  } default #Action;
};
```

#### `wildcard-excluding-one`
**Descrição:** Você está substituindo um elemento em sua projeção que já está incluído pelo curinga `*`.

**Exemplo:**
```cds
entity Book {
  key  id : String;
  isbn : String;
  content : String;
};
entity IsbnBook as projection on Book {
  *,
  isbn as id, // ❌
};
```

**Correção:** Adicione o elemento substituído à lista de exclusões do curinga.
```cds
entity IsbnBook as projection on Book {
  *,
  isbn as id
} excluding { id };
```
