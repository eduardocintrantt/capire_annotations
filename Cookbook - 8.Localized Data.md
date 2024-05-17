# Localized Data
Esta sessão estende a localização de conteúdo estático, como rótulos ou mensagens, para servir versões localizadas de dados reais da aplicação.

**Dados Localizados** Refere-se à manutenção de diferentes traduções de dados textuais e à recuperação automática das traduções que correspondem ao idioma preferido dos usuários, com fallback por linha para idiomas padrão, se as traduções necessárias não estiverem disponíveis. Os códigos de idioma estão no formato ISO 639-1.

**Declarando Dados Localizados**
- Use o modificador `localized` para marcar elementos de entidade que requerem textos traduzidos.
```cds
entity Books {
  key ID       : UUID;
      title    : localized String;
      descr    : localized String;
      price    : Decimal;
      currency : Currency;
}
```

**Restrição**
- As chaves das entidades não devem ser associações.
- `localized` em subelementos de entidade não é suportado atualmente.

**Por Trás das Cenas**
- O compilador CDS automaticamente expande a definição, aplicando os mecanismos básicos de Composições Gerenciadas e Nomes Escopados.
- Uma entidade separada `Books.texts` é adicionada para manter os textos traduzidos:
```cds
entity Books.texts {
  key locale : sap.common.Locale;
  key ID : UUID;
  title : String;
  descr : String;
}
```
   
- A entidade original é estendida com associações para `Books.texts`:
```cds
extend entity Books with {
  texts : Composition of many Books.texts on texts.ID=ID;
  localized : Association to Books.texts on localized.ID=ID and localized.locale = $user.locale;
}
```
**Vistas SQL Geradas**

- Vistas são geradas para facilitar a leitura de textos localizados com fallback equivalente:
```cds
entity localized.Books as select from Books {
  *,
  coalesce (localized.title, title) as title,
  coalesce (localized.descr, descr) as descr
};
```

**Resolvendo Textos Localizados via Vistas**
- O compilador CDS cria vistas que resolvem textos traduzidos internamente. SQLite tem suporte limitado para locales. Vistas adicionais são geradas para diferentes idiomas.

**Pesquisa sobre Textos Localizados em Tempo de Execução**
- Para operações de pesquisa, as CAP runtimes otimizam as consultas para resolver textos localizados, especialmente em bancos de dados como SAP HANA.

**Entidades Base Intactas**
- Todos os textos não são externalizados, mantendo os textos originais na entidade de origem, economizando um join ao ler textos localizados com fallback para os originais.

**Extendendo Entidades `.texts`**
- É possível estender coletivamente todas as entidades `.texts` geradas, estendendo o aspecto `sap.common.TextsAspect`.

**Pseudo variável $user.locale**
- Referida para determinar o idioma preferido do usuário e juntar traduções correspondentes das tabelas `.texts`.

**Leitura de Dados Localizados**
- **Código Agnóstico**: Ler textos originais:
```sql
SELECT ID, title, descr from Books
```
- **Para Usuários Finais**: Ler textos na língua preferida do usuário:
```sql
SELECT ID, localized.title, localized.descr from Books
```
- **Para UIs de Tradução**: Ler e escrever textos em todos os idiomas:
```sql
SELECT ID, texts[locale='fr'].title, texts[locale='fr'].descr from Books
```

**Servindo Dados Localizados**
- Os handlers genéricos dos runtimes de serviço servem automaticamente solicitações de leitura de vistas localizadas. Usuários veem textos na língua preferida ou no idioma de fallback.

**Operações de Leitura**
- Handlers genéricos redirecionam automaticamente todas as solicitações de leitura para as vistas localizadas no banco de dados SQL, exceto no modo draft do SAP Fiori.

**Operações de Escrita**
- Use inserts profundos ou upserts para preencher textos específicos de idioma.

**Operações de Atualização**
- Atualize textos específicos de idioma com uma solicitação PUT ou PATCH através da navegação.

**Operações de Exclusão**
- Para excluir textos específicos de idioma, execute uma solicitação DELETE na tabela de textos da entidade através da navegação.

**Dados Localizados Aninhados**
- Associações para outras entidades com textos localizados são automaticamente redirecionadas, facilitando a leitura de dados localizados com lógica de fallback independente.

**Adicionando Dados Iniciais**
- Adicione dados iniciais com dois arquivos .csv, um contendo dados no idioma padrão e o outro com dados traduzidos.
    
    **Exemplo**:
```csv
    Books.csv
ID;title;descr;author_ID;stock;price;currency_code;genre_ID
201;Wuthering Heights;Wuthering Heights, Emily Brontë's only novel ...;101;12;11.11;GBP;11
```
```csv
Books_texts.csv
ID;locale;title;descr
201;de;Sturmhöhe;Sturmhöhe (Originaltitel: Wuthering Heights) ist der einzige Roman...
207;de;Jane Eyre;Jane Eyre. Eine Autobiographie (Originaltitel: Jane Eyre. An Autobiography)...
```
    
Esta seção abrange como fornecer versões localizadas de dados reais da aplicação, incluindo a manutenção de traduções, leitura e escrita de dados localizados, e configuração de fallback por idioma.