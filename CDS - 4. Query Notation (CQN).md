### Estrutura de Queries CQN
#### SELECT
Um query SELECT completo segue este modelo (propriedades opcionais):
```js
SELECT = {SELECT:{
  distinct: true,
  from: source | join,
  mixin: { ...element },
  columns: projection,
  excluding: [ ...string ],
  where: _xpr,
  groupBy: [ ...expr ],
  having: _xpr,
  orderBy: [ ...ordering_term ],
  limit: { rows:expr, offset:expr },
  forUpdate: { wait: number },
  forShareLock: { wait: number },
  search: _xpr,
  count: Boolean
}}
```

**Propriedades:**
- `from`: fonte primária ou fontes combinadas
- `mixin`: dicionário de várias definições de elementos CSN
- `columns`: array de expressões de coluna
- `excluding`: array de nomes a excluir
- `where`, `groupBy`, `having`, `orderBy`, `search`: expressões
- `limit`: dicionário com `rows` e `offset`
- `count`: booleano
#### Exemplo de SELECT
Query CQL:
```sql
SELECT from samples.bookshop.Books {
  title, author.name as author,
  1 as one,
  x+2 as two : Integer,
} excluding {
  dummy
}
WHERE ID=111
GROUP BY x.y
HAVING x.y<9
ORDER BY title asc
LIMIT 11 OFFSET 22
```
Representação CQN:
```js
CQN = {SELECT:{
  from: {ref:["samples.bookshop.Books"]},
  columns: [
    {ref:["title"]},
    {ref:["author","name"], as: "author"},
    {val:1, as: "one"},
    {xpr:[{ref:['x']}, '+', {val:2}], as: "two", cast: {type:"cds.Integer"}}
  ],
  excluding: ["dummy"],
  where: [{ref:["ID"]}, "=", {val: 111}],
  groupBy: [{ref:["x","y"]}],
  having: [{ref:["x","y"]}, "<", {val: 9}],
  orderBy: [{ref:["title"], sort:'asc' }],
  limit: {rows:{val:11}, offset:{val:22}}
}}
```

### Outras Operações CQN
#### UPSERT
```js
UPSERT = {UPSERT:{
   into: (ref + { as:string }) | string,
   entries: [ ...{ ...column:any } ],
   as: SELECT
}}
```
#### INSERT
```js
INSERT = {INSERT:{
   into: (ref + { as:string }) | string,
   columns: [ ...string ],
   values: [ ...any ],
   rows: [ ...[ ...any ] ],
   entries: [ ...{ ...column:any } ],
   as: SELECT
}}
```

**Exemplos de INSERT:**

1. **Valores Únicos**
   ```js
   CQN = {INSERT:{
  into: { ref: ['Books'] },
  columns: [ 'ID', 'title', 'author_id', 'stock' ],
  values: [ 201, 'Wuthering Heights', 101, 12 ]
}}
```
2. **Várias Linhas**
    ```js
    CQN = {INSERT:{
  into: { ref: ['Books'] },
  columns: [ 'ID', 'title', 'author_id', 'stock' ],
  rows: [
    [ 201, 'Wuthering Heights', 101, 12 ],
    [ 251, 'The Raven', 150, 333 ],
    [ 252, 'Eleonora', 150, 234 ]
  ]
}}
```
    
3. **Entradas com Relacionamentos**
    ```js
    CQN = {INSERT:{
  into: { ref: ['Authors'] }, entries: [
    { ID:150, name:'Edgar Allen Poe', books:[
      { ID:251, title:'The Raven' },
      { ID:252, title:'Eleonora' }
    ] }
  ]
}}
```
#### UPDATE
```js
UPDATE = {UPDATE:{
   entity: ref + { as:string },
   data: { ...column:any },
   where: _xpr
}}
```

#### DELETE
```js
DELETE = {DELETE:{
   from: ref + { as:string },
   where: _xpr
}}
```

#### CREATE
```js
CREATE = {CREATE:{
   entity: entity | string,
   as: SELECT
}}
```

#### DROP
```js
DROP = {DROP:{
   table: ref,
   view: ref,
   entity: ref
}}
```

**Exemplos de DROP:**
1. **Tabela**
    ```js
    CQN = {DROP:{ table: { ref: ['Books'] } }}
```
    
2. **Visão**
    ```js
    CQN = {DROP:{ view: { ref: ['Books'] } }}
```
    
3. **Entidade**
  ```js
  CQN = {DROP:{ entity: { ref: ['Books'] } }}
```