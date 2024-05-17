# Temporal Data
### Timeless Models

**Timeless Model** Para gerenciar funcionários e suas atribuições de trabalho, usamos um modelo sem gerenciamento de dados temporais:
```cds
namespace com.acme.hr;
using { com.acme.common.Persons } from './common';

entity Employees : Persons {
  jobs : Composition of many WorkAssignments on jobs.empl=$self;
  job1 : Association to one /*of*/ WorkAssignments;
}

entity WorkAssignments {
  key ID  : UUID;
  role    : String(111);
  empl    : Association to Employees;
  dept    : Association to Departments;
}

entity Departments {
  key ID  : UUID;
  name    : String(111);
  head    : Association to Employees;
  members : Association to many Employees on members.jobs.dept = $self;
}
```
Neste modelo, um funcionário pode ter várias atribuições de trabalho simultâneas, cada uma ligada a um departamento.

**Dados Atuais** 
Um conjunto de dados para este modelo pode parecer assim:
- Alice tem os cargos de desenvolvedora e consultora.
- Bob é construtor.
- Alice trabalha nos departamentos de desenvolvimento core e desenvolvimento de aplicativos.
- Bob está no departamento de construção.

**Declarando Entidades Temporais** Para rastrear dados temporais, adicionamos elementos de data/hora às entidades com anotações @cds.valid.from/to:
```cds
entity WorkAssignments { 
  start : Date @cds.valid.from;
  end   : Date @cds.valid.to;
}
```
Ou usamos o aspecto `temporal` do pacote `@sap/cds/common`:
```cds
using { temporal } from '@sap/cds/common';
entity WorkAssignments : temporal {/*...*/}
```

**Detalhes Temporais Separados** 
Podemos separar elementos temporais de não-temporais:
```cds
entity WorkAssignments {          
  key ID  : UUID;
  empl    : Association to Employees;
  details : Composition of WorkDetails on details.ID = $self.ID;
}
entity WorkDetails : temporal {   
  key ID  : UUID;                 
  role    : String(111);
  dept    : Association to Departments;
}
```

**Servindo Dados Temporais** 
Exponha as entidades em um serviço:
```cds
using { com.acme.hr } from './temporal-model';
service HRService {
  entity Employees as projection on hr.Employees;
  entity WorkAssignments as projection on hr.WorkAssignments;
  entity Departments as projection on hr.Departments;
}
```

**Consultas Temporais** 
Consultas padrão retornam dados válidos "as of now".

Para ler funcionários com suas atribuições de trabalho atuais:
```http
GET Employees?$expand=jobs($select=role&$expand=dept($select=name))
```

**Consultas de Viagem no Tempo** 
Para ler dados válidos em uma data específica:
```http
GET Employees?sap-valid-at=date'2017-01-01'&$expand=jobs($select=role&$expand=dept($select=name))
```

**Consultas de Período de Tempo** 
Para ler o histórico de dados desde uma data específica:
```http
GET Employees?sap-valid-at=date'2017-01-01'&$expand=jobs($select=role&$expand=dept($select=name))
```

**Dados Temporais Transitivos** 
Ao expandir entidades temporais através de várias entidades, tome cuidado para evitar informações redundantes.

Por exemplo, se `WorkAssignments` e `Departments` são temporais:
```cds
using { temporal } from '@sap/cds/common';
entity WorkAssignments : temporal {/*...*/
  dept : Association to Departments;
}
entity Departments : temporal {/*...*/}
```

Para evitar redundância, adicione uma associação alternativa:
```cds
using { temporal } from '@sap/cds/common';
entity WorkAssignments : temporal {/*...*/
  dept : Association to Departments;
  dept1 : Association to Departments on dept1.id = dept.id and dept1.validFrom <= validFrom and validFrom < dept1.validTo;
}
entity Departments : temporal {/*...*/}
```

**Chaves Primárias de Slices de Tempo** 
Enquanto entidades atemporais são identificadas pela chave primária declarada, slices de tempo são identificadas pela chave conceitual + validFrom.
```sql
CREATE TABLE com_acme_hr_WorkAssignments (
    ID : nvarchar(36),
    validFrom : timestamp,
    validTo : timestamp,
    PRIMARY KEY ( ID, validFrom )
)
```

Para ler um slice de tempo específico:
```sql
SELECT from WorkAssignments WHERE ID='WA1' and validFrom='2017-01-01'
```
