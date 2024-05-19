### Expressões em CDS
Expressões em definições e consultas CDS podem ser uma das seguintes:
```js
expr = // uma das seguintes...
  val    |   // [valores literais]
  ref    |   // referências ou funções
  xpr    |   // expressões de operadores
  func   |   // chamadas de funções
  list   |   // listas/tuplas
  param  |   // parâmetros de vinculação
  sym    |   // símbolo enum
  SELECT     // subconsultas
```
#### Valores Literais
Valores literais são representados como `{val:...}`, com a propriedade `val` contendo o valor literal real, conforme especificado em JSON. Exemplos:
```js
cds.parse.expr(`'uma string'`)  == {val:'uma string'}
cds.parse.expr(`11`)  == {val:11}
cds.parse.expr(`true`)  == {val:true}
cds.parse.expr(`null`)  == {val:null}
cds.parse.expr(`date'2023-04-15'`)  == {val: '2023-04-15', literal: 'date'}
cds.parse.expr(`time'13:05:23Z'`)  == {val: '13:05:23Z', literal: 'time'}
cds.parse.expr(`timestamp'2023-04-15T13:05:23Z'`)  == {val: '2023-04-15T13:05:23Z', literal: 'timestamp'}
```

#### Referências
Uma referência é representada como `{ref: …}` com a propriedade `ref` contendo um array de segmentos de referência como strings identificadoras. Exemplos:
```js
let cqn4 = cds.parse.expr
cqn4(`![keyword]`) == {ref:['keyword']}
cqn4(`foo.bar`) == {ref:['foo','bar']}
cqn4(`foo[9].bar`) == {ref:[{ id:'foo', where:[{val:9}] }, 'bar' ]}
cqn4(`foo(p:x).bar`) == {ref:[{ id:'foo', args:{p:{ref:['x']}} }, 'bar' ]}
cqn4(`foo[where a=1 group by b having b>2 order by c limit 7].bar`)
  == {ref:[{ id:'foo', where:[{ref:['a']}, '=', {val:9}],
                       groupBy: [{ref: ['b']}], having: [{ref: ['b']}, '>',  {val:2}],
                       orderBy: [{ref: ['c']}], limit: {rows: {val: 7}} },
           'bar' ]}
```

#### Chamadas de Funções
Chamadas de funções são representadas da seguinte forma:
```js
func = { func:string, args: _positional | _named, xpr:_xpr }
_positional = [ ...expr ]
_named = { ... <name>:expr }
```

Exemplos:
```js
let cqn4 = cds.parse.expr
cqn4(`foo(p=>x)`) == {func:'foo', args:{p:{ref:['x']}}}
cqn4(`sum(x)`)   == {func:'sum', args:[{ref:['x']}]}
cqn4(`count(*)`) == {func:'count', args:['*']}
cqn4(`rank() over (...)`) == {func:'rank', args:[], xpr:['over', {xpr:[...]}]}
cqn4(`shape.ST_Area()`)
  == {xpr: [{ref: ['shape']}, '.', {func: 'ST_Area', 'args': []}]}
cqn4(`new ST_Point(2, 3)`)
  == {xpr: ['new', {func: 'ST_Point', args: [{val: 2}, {val: 3}]}]}
```

#### Listas
Listas ou tuplas são representadas como `{list:...}`, com a propriedade `list` contendo um array das entradas da lista. Exemplos:
```js
cds.parse.expr(`(1, 2, 3)`) == {list: [{val: 1}, {val: 2}, {val: 3}]}
cds.parse.expr(`(foo, bar)`) == {list: [{ref: ['foo']}, {ref: ['bar']}]}
```

#### Expressões de Operadores
Operadores unem uma ou mais expressões em complexas, representadas como `{xpr:...}`. A propriedade `xpr` mantém uma sequência de operadores e operandos. Exemplos:
```js
cds.parse.expr(`x<9`)  == {xpr:[ {ref:['x']}, '<', {val:9} ]}
cds.parse.expr(`x<9 and (y=1 or z=2)`)  == {xpr:[ {ref:['x']}, '<', {val:9}, 'and', {xpr:[ {ref:['y']}, '=', {val:1}, 'or', {ref:['z']}, '=', {val:2} ]}]}
cds.parse.expr(`exists books[year = 2000]`)  == {xpr:[ 'exists', {ref: [ {id:'books', where:[ {'ref':['year']}, '=', {'val': 2000} ]}]} ]}
```

#### Parâmetros de Vinculação

Parâmetros de vinculação para instruções preparadas são representados como `{ref:..., param:true}`. Exemplos:
```js
cds.parse.expr(`x=:1`) == [{ref:['x']}, '=', {ref:[1], param:true}]
cds.parse.expr(`x=:y`) == [{ref:['x']}, '=', {ref:['y'], param:true}]
cds.parse.expr(`x=?`)  == [{ref:['x']}, '=', {ref:['?'], param:true}]
```