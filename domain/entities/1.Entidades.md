#  Camada Entity (PT-BR)

## 🎯 Função principal

A camada **Entity** é responsável por representar as **entidades do domínio**, ou seja, os **objetos Java que refletem as tabelas do banco de dados**.  
Cada classe Entity mapeia uma tabela real e define seus **campos (colunas)**, **chaves primárias e estrangeiras**, **relacionamentos** e **restrições**.

Essa camada permite que a aplicação trabalhe com **objetos Java em vez de comandos SQL diretos**, facilitando a **persistência de dados** e tornando o código mais **limpo, seguro e orientado a objetos**.

---

## ⚙️ Atributos principais (resumido)

| Anotação | Descrição |
|-----------|------------|
| `@Entity` | Indica que a classe é uma entidade gerenciada pelo **JPA (Java Persistence API)**. |
| `@Table(name = "nome_tabela")` | Define o nome da tabela associada. Se não especificado, o nome da classe será usado. |
| `@Id` | Identifica o campo que representa a **chave primária**. |
| `@GeneratedValue(strategy = GenerationType.IDENTITY)` | Define a estratégia de geração automática do ID (`IDENTITY`, `AUTO`, `SEQUENCE`, `TABLE`). |
| `@Column(nullable = false)` | Personaliza o nome e as propriedades de uma coluna (ex.: obrigatoriedade, tamanho, unicidade). |

---

###  Exemplo

```java
@Entity
@Table(name = "clientes")
public class Cliente {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false, length = 100)
    private String nome;

    @Column(unique = true, nullable = false)
    private String email;
}
```

🔗 Relacionamentos
|Anotação|	Tipo|	Descrição|
|---------|-------|-----------|
| @OneToOne|	1 : 1	| Um registro em uma tabela está ligado a exatamente um registro em outra tabela. |
| @OneToMany|	1 : N	| Um registro está relacionado a vários registros de outra tabela.|
| @ManyToOne|	N : 1	| Vários registros fazem referência a um único registro principal.|
| @ManyToMany|	N : N	| Múltiplos registros de uma tabela estão associados a múltiplos registros de outra.|

Essas anotações podem usar atributos como mappedBy, cascade e fetch para controlar comportamento de carregamento, persistência e exclusão em cascata.

🏗️ Construtores

Um construtor padrão (sem parâmetros) é obrigatório para o funcionamento do JPA.
```java
public MinhaEntidade() {
    // Construtor padrão exigido pelo JPA
}
```

Construtores adicionais podem ser usados para inicializar objetos com valores específicos.
```java
public MinhaEntidade(String nome) {
    this.nome = nome;
}
```
🧰 Getters e Setters

Os métodos Get e Set são essenciais para que o Hibernate consiga ler e gravar valores nas propriedades da entidade.
Eles também mantêm o encapsulamento e permitem manipular os dados de forma controlada.
```java
// Getter - obtém o valor
public Long getId() {
    return id;
}

// Setter - define o valor
public void setId(Long id) {
    this.id = id;
}
```


Cada campo da entidade deve possuir seu Getter e Setter correspondente.

##  Equals e HashCode

### 🔍 Por que sobrescrever?

Os métodos `equals()` e `hashCode()` devem ser sobrescritos em **entidades JPA** porque o comportamento padrão herdado da classe `Object` compara **endereços de memória**, não o **conteúdo lógico** do objeto.

Em um sistema com persistência, duas instâncias diferentes de uma mesma entidade (por exemplo, dois objetos `Cliente` com o mesmo `id`) representam o **mesmo registro no banco de dados**.  
Sem sobrescrever `equals()` e `hashCode()`, o Java consideraria essas instâncias **diferentes**, mesmo que apontem para o mesmo registro.

Isso causa problemas em:
- **Coleções baseadas em hashing**, como `HashSet` e `HashMap`, que dependem de `hashCode()`.
- **Operações do Hibernate**, que precisam identificar corretamente se duas entidades representam o mesmo registro.
- **Comparações de entidades**, especialmente ao sincronizar estados entre memória e banco.

deve-se realizar a sobrescrita dos métodos equals() e hashCode() dentro da própria classe da entidade, ou seja, no arquivo .java da entidade em que a camada Entity está definida.

---

### ⚙️ O que o novo código faz

```java
@Override
public boolean equals(Object o) {
    if (this == o) return true; // Mesmo objeto em memória → iguais
    if (!(o instanceof MinhaEntidade)) return false; // Tipos diferentes de instância → não iguais
    MinhaEntidade that = (MinhaEntidade) o; //converte a referência o de tipo Object para tipo MinhaEntidade.
    return Objects.equals(id, that.id); // Compara igualdade lógica pelo ID
}

@Override
public int hashCode() {
    return Objects.hash(id); // Gera código de hash baseado no ID
}
```

# :open_book: Mapeamento das Entidades

### 🔗 Anotações de CLASSE 
| Anotação | Descrição |
|----------|-----------|
| `@Entity` | Marca a classe como entidade gerenciada pelo **JPA (Java Persistence API)**. |
| `@Table(name = "nome_tabela")` | Define explicitamente o nome da tabela associada. Se omitida, o nome da classe será usado. |

### :key: Anotações de IDENTIFICAÇÃO 
| Anotação | Descrição |
|----------|-----------|
| `@Id` | Identifica o campo que representa a **chave primária** da entidade. |
| `@GeneratedValue(strategy = GenerationType.IDENTITY)` | Define a estratégia de geração automática do ID (ex.: `IDENTITY`, `AUTO`, `SEQUENCE`, `TABLE`). |

### 📊 Anotações de COLUNAS
| Anotação | Descrição |
|----------|-----------|
| `@Column(nullable = false)` | Personaliza uma coluna: nome, obrigatoriedade (`nullable`), unicidade (`unique`), tamanho (`length`), etc. |
| `@Lob` | Indica que o campo será persistido como objeto de grande tamanho (LOB); geralmente usado para BLOBs ou CLOBs. |
| `@Transient` | Indica que o campo **não será persistido** no banco de dados — apenas na memória da aplicação. |

### :family_man_woman_boy: Anotações de RELACIONAMENTOS
| Anotação | Tipo | Descrição |
|----------|------|-----------|
| `@OneToOne` | 1 : 1 | Um registro está ligado exatamente a outro. |
| `@OneToMany` | 1 : N | Um registro da entidade está ligado a vários de outra entidade. |
| `@ManyToOne` | N : 1 | Vários registros da entidade fazem referência a um registro de outra entidade. |
| `@ManyToMany` | N : N | Vários registros de ambas as entidades estão associados entre si. |
| `@JoinColumn` | — | Define a coluna da chave estrangeira (ex.: `referencedColumnName`, `nullable`, `name`) para mapear o relacionamento. |

### :pencil: Anotações de DATA E AUDITORIA
| Anotação | Descrição |
|----------|-----------|
| `@Temporal` | Controla o tipo de dado para atributos de data/hora (`TemporalType.DATE`, `TIME`, `TIMESTAMP`). |
| `@CreationTimestamp` | Preenche automaticamente com a data/hora da **criação** do registro. |
| `@UpdateTimestamp` | Preenche automaticamente com a data/hora da **última atualização** do registro. |

### :crown: Anotações de HERANÇA  
| Anotação | Descrição |
|----------|-----------|
| `@MappedSuperclass` | Marca uma classe como superclasse de entidades. Não vira uma tabela, mas suas subclasses herdam seus campos. |
| `@Inheritance(strategy = InheritanceType.*)` | Define a estratégia de mapeamento de herança para entidades (ex.: `JOINED`, `SINGLE_TABLE`, `TABLE_PER_CLASS`). |





