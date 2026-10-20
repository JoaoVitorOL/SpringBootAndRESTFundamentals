# Visão Geral das Camadas — Fluxo de uma API Spring Boot (PT-BR)

> **Nível:** Iniciante ao intermediário  
> **Linguagem:** Java  
> **Framework:** Spring Boot (versão 3.x)

---

## 📑 Sumário

- [1. Função principal](#-1-função-principal)
- [2. Visão geral em uma imagem](#-2-visão-geral-em-uma-imagem)
- [3. Resumo de cada camada](#-3-resumo-de-cada-camada)
- [4. A jornada de uma requisição: `POST /api/v1/alunos`](#-4-a-jornada-de-uma-requisição-post-apiv1alunos)
  - [4.1 Linha do tempo](#41-linha-do-tempo)
  - [4.2 Transformação do dado ao longo do fluxo](#42-transformação-do-dado-ao-longo-do-fluxo)
- [5. O mesmo fluxo em código (camada por camada)](#-5-o-mesmo-fluxo-em-código-camada-por-camada)
  - [5.1 Enum — valores permitidos para o status](#51-enum--valores-permitidos-para-o-status)
  - [5.2 Entity — a tabela em formato Java](#52-entity--a-tabela-em-formato-java)
  - [5.3 Repository — acesso ao banco](#53-repository--acesso-ao-banco)
  - [5.4 DTOs — o que entra e o que sai](#54-dtos--o-que-entra-e-o-que-sai)
  - [5.5 Service — regras de negócio](#55-service--regras-de-negócio)
  - [5.6 Controller — porta de entrada HTTP](#56-controller--porta-de-entrada-http)
- [6. Onde cada validação acontece](#-6-onde-cada-validação-acontece)
- [7. Onde colocar cada tipo de código?](#-7-onde-colocar-cada-tipo-de-código)
- [8. Erros frequentes de iniciantes](#-8-erros-frequentes-de-iniciantes)
- [9. Resumo final em uma tabela](#-9-resumo-final-em-uma-tabela)
- [10. Glossário técnico completo](#-10-glossário-técnico-completo)

---

# 🎯 1. Função principal

Este arquivo **resume e relaciona** as camadas estudadas (Controller, DTO, Service, Repository, Entity e Enum) e mostra **como elas trabalham juntas** em uma única requisição.

Cada arquivo anterior explica **uma camada isolada**. Este arquivo responde às perguntas:
- Qual é a **ordem** em que as camadas são acionadas?
- O que **entra e o que sai** de cada camada?
- **Onde** cada tipo de código deve ficar?

> Pré-requisito de leitura: os arquivos de cada camada. Para o detalhamento de rotas, verbos e status HTTP, consultar `Mapping_RESTful_API.md`.

---

# 🗺️ 2. Visão geral em uma imagem

```text
                  REQUISIÇÃO  (ida)
  ┌─────────┐   JSON    ┌────────────┐  RequestDTO  ┌─────────┐  Entity   ┌────────────┐   SQL    ┌───────┐
  │ Cliente │ ────────► │ Controller │ ───────────► │ Service │ ────────► │ Repository │ ───────► │ Banco │
  └─────────┘           └────────────┘              └─────────┘           └────────────┘          └───────┘
       ▲                      │                          │                     │                       │
       │   JSON               ▼                          ▼                     ▼                       │
       └──────────────  ResponseEntity  ◄──────  ResponseDTO  ◄──────────  Entity  ◄──────────────────┘
                  RESPOSTA  (volta)
```

**Regra de dependência (sentido único):** cada camada **só conhece a camada imediatamente à sua direita**.

| Camada | Pode chamar | NÃO pode chamar |
|---|---|---|
| Controller | Service | Repository |
| Service | Repository | Controller |
| Repository | Banco de dados (via JPA) | Service, Controller |

> **Por que essa regra existe?** Se o Controller acessasse o Repository diretamente, as regras de negócio (que ficam no Service) seriam ignoradas. A separação garante que **cada mudança afete poucos arquivos**.

---

# 📋 3. Resumo de cada camada

| Camada | Pergunta que responde | Responsabilidade | Recebe | Devolve | Anotações principais |
|---|---|---|---|---|---|
| **Controller** | "Quem atende esta URL?" | Receber a requisição HTTP, acionar o Service e montar a resposta HTTP. | JSON (requisição) | `ResponseEntity` (status + corpo) | `@RestController`, `@RequestMapping`, `@GetMapping`, `@PostMapping`, `@RequestBody`, `@PathVariable`, `@Valid` |
| **DTO** | "Quais dados podem entrar e sair?" | Transportar dados e validar o **formato** deles. Não possui regra de negócio. | Dados brutos | Dados validados | `@NotBlank`, `@Size`, `@Email`, `@NotNull`, `@Pattern` |
| **Service** | "Isto é permitido pelas regras?" | Aplicar **regras de negócio**, coordenar operações e controlar transações. | DTO | DTO | `@Service`, `@Transactional` |
| **Repository** | "Como leio/gravo no banco?" | Acessar o banco de dados sem escrever SQL na maioria dos casos. | Entity | Entity | `@Repository`, `@Query`, `@Param`, `@Modifying` |
| **Entity** | "Como é a tabela, em Java?" | Representar uma **tabela** do banco como uma classe. | — | — | `@Entity`, `@Table`, `@Id`, `@Column`, `@ManyToOne` |
| **Enum** | "Quais valores são válidos aqui?" | Restringir uma variável a um **conjunto fixo** de valores. | — | — | (usado junto de `@Enumerated` na Entity) |

### Papel de cada objeto: três "formatos" do mesmo dado

| Objeto | Camada onde vive | Finalidade | Exemplo |
|---|---|---|---|
| `AlunoRequestDTO` | Controller → Service | Dados que **entram** na API | `{ "nome": "Ana", "email": "ana@email.com" }` |
| `Aluno` (Entity) | Service ↔ Repository | Representação da **tabela** | Linha da tabela `alunos` |
| `AlunoResponseDTO` | Service → Controller | Dados que **saem** da API | `{ "id": 1, "nome": "Ana", "email": "ana@email.com" }` |

> **Por que não usar a Entity em todas as camadas?** Expor a Entity diretamente revela campos internos (ex.: senha, flags de controle) e acopla o formato da API ao formato do banco. O DTO resolve isso (ver arquivo DTO).

---

# 🔄 4. A jornada de uma requisição: `POST /api/v1/alunos`

Cenário: o cliente deseja **cadastrar um aluno**.

## 4.1 Linha do tempo

| Passo | Camada | O que acontece | Se falhar |
|---|---|---|---|
| 1 | Cliente | Envia `POST /api/v1/alunos` com JSON no corpo. | — |
| 2 | Spring (`DispatcherServlet`) | Localiza o método do Controller pelo verbo + URL (**mapping**). | `404` (rota inexistente) ou `405` (verbo incorreto) |
| 3 | Controller + DTO | `@RequestBody` converte o JSON em `AlunoRequestDTO`; `@Valid` executa as validações do DTO (**validação estrutural**). | `400 Bad Request` |
| 4 | Controller | Chama `alunoService.salvar(dto)`. | — |
| 5 | Service | Verifica as regras de negócio (ex.: e-mail já cadastrado) (**validação semântica**). | `409 Conflict` (`BusinessException`) |
| 6 | Service | Converte o DTO em `Aluno` (Entity). | — |
| 7 | Repository | `save(aluno)` gera e executa o `INSERT`. | — |
| 8 | Banco de dados | Aplica as constraints (`UNIQUE`, `NOT NULL`) (**validação de persistência**). | Erro de integridade → `500` (se não tratado) |
| 9 | Service | Converte a Entity salva em `AlunoResponseDTO`. | — |
| 10 | Controller | Monta `201 Created` + header `Location` + corpo. | — |
| 11 | Cliente | Recebe o JSON do aluno criado. | — |

## 4.2 Transformação do dado ao longo do fluxo

| Etapa | Formato do dado |
|---|---|
| Cliente → Controller | JSON (texto) |
| Dentro do Controller | `AlunoRequestDTO` (objeto Java) |
| Dentro do Service (regras) | `AlunoRequestDTO` → `Aluno` (Entity) |
| Service → Repository → Banco | `Aluno` → comando SQL (`INSERT`) |
| Banco → Repository → Service | Linha da tabela → `Aluno` |
| Dentro do Service (saída) | `Aluno` → `AlunoResponseDTO` |
| Controller → Cliente | `AlunoResponseDTO` → JSON |

---

# 🧪 5. O mesmo fluxo em código (camada por camada)

> O código abaixo usa as **correções** apontadas na análise dos arquivos originais (ex.: `String` com `S` maiúsculo, `ResponseEntity.ok`, `orElseThrow`). Os imports foram omitidos para manter o foco.

## 5.1 Enum — valores permitidos para o status

```java
public enum StatusEnum {
    ATIVO,      // 1: aluno com matrícula ativa
    INATIVO     // 2: aluno com matrícula inativa
}
```

| Trecho | Explicação |
|---|---|
| `public enum StatusEnum` | Declara um tipo com **valores fixos**: qualquer variável `StatusEnum` só pode ser `ATIVO` ou `INATIVO`. |
| `ATIVO`, `INATIVO` | Constantes do enum. Escrever `"ativo "` (texto livre) deixa de ser possível: o compilador rejeita valores fora da lista. |

> **Variação:** o arquivo `Enums` usa o formato com `codigo` e `descricao`. Esse formato é útil quando o valor precisa de um código numérico ou de um texto de exibição, mas exige mais código para ser salvo no banco.

## 5.2 Entity — a tabela em formato Java

```java
@Entity                                                   // 1
@Table(name = "alunos")                                   // 2
public class Aluno {

    @Id                                                   // 3
    @GeneratedValue(strategy = GenerationType.IDENTITY)   // 4
    private Long id;

    @Column(nullable = false, length = 50)                // 5
    private String nome;

    @Column(nullable = false, unique = true)              // 6
    private String email;

    @Enumerated(EnumType.STRING)                          // 7
    @Column(nullable = false)
    private StatusEnum status;

    protected Aluno() { }                                 // 8

    public Aluno(String nome, String email) {             // 9
        this.nome = nome;
        this.email = email;
        this.status = StatusEnum.ATIVO;
    }

    // getters e setters omitidos
}
```

| Nº | Trecho | Explicação |
|---|---|---|
| 1 | `@Entity` | Informa ao JPA que esta classe corresponde a uma tabela. |
| 2 | `@Table(name = "alunos")` | Define o nome da tabela no banco. |
| 3 | `@Id` | Marca `id` como chave primária. |
| 4 | `@GeneratedValue(IDENTITY)` | O próprio banco gera o valor do `id` (auto incremento). |
| 5 | `@Column(nullable = false, length = 50)` | Coluna obrigatória, com até 50 caracteres. |
| 6 | `@Column(unique = true)` | Cria a restrição `UNIQUE`: o banco impede dois e-mails iguais. |
| 7 | `@Enumerated(EnumType.STRING)` | Grava o **nome** do enum (`"ATIVO"`) na coluna. |
| 8 | Construtor sem parâmetros | Exigido pelo JPA para criar o objeto ao ler do banco. |
| 9 | Construtor com dados | Usado pelo Service para criar um aluno novo, já com status inicial `ATIVO`. |

**`@Enumerated`: alternativas para salvar o Enum**

| Alternativa | Prós | Contras |
|---|---|---|
| `EnumType.STRING` (usada) | Legível no banco; seguro contra reordenação do enum. | Ocupa mais espaço; renomear a constante exige migração dos dados. |
| `EnumType.ORDINAL` | Ocupa menos espaço. | **Frágil:** inserir uma constante no meio do enum corrompe os dados existentes. |
| `AttributeConverter` (usa o `codigo` do enum) | Controle total do valor gravado. | Exige uma classe de conversão adicional. |

## 5.3 Repository — acesso ao banco

```java
@Repository                                                          // 1
public interface AlunoRepository extends JpaRepository<Aluno, Long> { // 2
    boolean existsByEmail(String email);                              // 3
}
```

| Nº | Trecho | Explicação |
|---|---|---|
| 1 | `@Repository` | Marca o componente de persistência. |
| 2 | `extends JpaRepository<Aluno, Long>` | Herda métodos prontos (`save`, `findById`, `findAll`, `delete`...). `Aluno` é a entidade e `Long` é o tipo do `id`. |
| 3 | `existsByEmail(String email)` | **Método derivado:** o Spring lê o nome (`existsBy` + `Email`) e gera a consulta automaticamente. Retorna `true` se existir um aluno com o e-mail. |

> **Nota:** como é uma `interface`, **não há implementação escrita**. O Spring a gera em tempo de execução.

## 5.4 DTOs — o que entra e o que sai

```java
public record AlunoRequestDTO(
        @NotBlank(message = "Nome é obrigatório")
        @Size(min = 3, max = 50, message = "Nome deve ter entre 3 e 50 caracteres")
        String nome,

        @NotBlank(message = "E-mail é obrigatório")
        @Email(message = "E-mail inválido")
        String email
) {}

public record AlunoResponseDTO(Long id, String nome, String email) {}
```

| Trecho | Explicação |
|---|---|
| `record` | Tipo do Java que cria uma classe imutável com construtor e acessores (`nome()`, `email()`) automaticamente. Ideal para DTOs. |
| `@NotBlank` | Rejeita `null`, texto vazio e texto apenas com espaços. |
| `@Size(min, max)` | Limita o comprimento do texto. |
| `@Email` | Valida **somente o formato** do e-mail. Não verifica se o e-mail existe nem se já está cadastrado. |
| `AlunoResponseDTO` | Sem validações: é a saída. Não inclui o `status`, pois o cliente não precisa dele neste exemplo. |

## 5.5 Service — regras de negócio

```java
@Service                                                                  // 1
public class AlunoService {

    private final AlunoRepository alunoRepository;                        // 2

    public AlunoService(AlunoRepository alunoRepository) {                // 3
        this.alunoRepository = alunoRepository;
    }

    @Transactional                                                        // 4
    public AlunoResponseDTO salvar(AlunoRequestDTO dto) {
        if (alunoRepository.existsByEmail(dto.email())) {                 // 5
            throw new BusinessException("E-mail já cadastrado");          // 6
        }
        Aluno aluno = new Aluno(dto.nome(), dto.email());                 // 7
        Aluno salvo = alunoRepository.save(aluno);                        // 8
        return new AlunoResponseDTO(salvo.getId(), salvo.getNome(), salvo.getEmail()); // 9
    }
}
```

| Nº | Trecho | Explicação |
|---|---|---|
| 1 | `@Service` | Registra a classe como componente (bean) da camada de negócio. |
| 2 | `private final AlunoRepository` | Dependência do Service; `final` impede troca após a criação. |
| 3 | Construtor | O Spring injeta o `AlunoRepository` aqui automaticamente (injeção por construtor). |
| 4 | `@Transactional` | A operação roda em uma transação: se ocorrer erro, o banco desfaz tudo (**rollback**). |
| 5 | `existsByEmail(dto.email())` | Consulta o banco: já existe aluno com este e-mail? |
| 6 | `throw new BusinessException(...)` | Interrompe o fluxo; o handler global converte em `409 Conflict`. |
| 7 | `new Aluno(...)` | Converte **DTO → Entity**. |
| 8 | `alunoRepository.save(aluno)` | Executa o `INSERT`; devolve a Entity já com `id` preenchido. |
| 9 | `new AlunoResponseDTO(...)` | Converte **Entity → DTO** de saída. |

## 5.6 Controller — porta de entrada HTTP

```java
@RestController                                           // 1
@RequestMapping("/api/v1/alunos")                         // 2
public class AlunoController {

    private final AlunoService alunoService;

    public AlunoController(AlunoService alunoService) {
        this.alunoService = alunoService;
    }

    @PostMapping                                          // 3
    public ResponseEntity<AlunoResponseDTO> criar(
            @Valid @RequestBody AlunoRequestDTO dto) {    // 4
        AlunoResponseDTO criado = alunoService.salvar(dto);                       // 5
        URI location = ServletUriComponentsBuilder.fromCurrentRequest()
                .path("/{id}").buildAndExpand(criado.id()).toUri();               // 6
        return ResponseEntity.created(location).body(criado);                     // 7
    }
}
```

| Nº | Trecho | Explicação |
|---|---|---|
| 1 | `@RestController` | Classe que atende requisições HTTP e devolve JSON. |
| 2 | `@RequestMapping("/api/v1/alunos")` | Rota base de todos os métodos da classe. |
| 3 | `@PostMapping` | Este método atende `POST /api/v1/alunos`. |
| 4 | `@Valid @RequestBody` | Converte o JSON em DTO e **dispara as validações** do DTO. Se reprovar, o método nem é executado (`400`). |
| 5 | `alunoService.salvar(dto)` | Delega **toda a lógica** ao Service. |
| 6 | `ServletUriComponentsBuilder...` | Monta a URL do recurso criado (ex.: `/api/v1/alunos/1`). |
| 7 | `ResponseEntity.created(...)` | Responde `201 Created` com o header `Location` e o aluno no corpo. |

---

# 🛡️ 6. Onde cada validação acontece

A mesma informação passa por **três filtros**, cada um em uma camada diferente.

| Ordem | Tipo | Camada | Pergunta | Exemplo | Resultado em caso de falha |
|---|---|---|---|---|---|
| 1º | Estrutural | DTO (ativada pelo `@Valid` no Controller) | "O dado tem formato válido?" | Nome em branco; e-mail sem `@` | `400 Bad Request` |
| 2º | Semântica | Service | "O dado respeita as regras de negócio?" | E-mail já cadastrado | `409 Conflict` |
| 3º | Persistência | Banco de dados | "O dado respeita a integridade dos dados?" | `UNIQUE`, `NOT NULL`, `FOREIGN KEY` | Erro de integridade |

> **Por que três e não apenas uma?** Cada filtro protege contra um tipo diferente de erro. O DTO é rápido e barato (não consulta o banco). O Service conhece o contexto (consulta o banco). O banco é a **última barreira**, que funciona mesmo se outro sistema gravar dados sem passar pela API.

---

# ❓ 7. Onde colocar cada tipo de código?

| Tenho este código... | Ele pertence a... | Motivo |
|---|---|---|
| Rota, verbo HTTP, status de resposta | Controller | Tratam do protocolo HTTP. |
| "Nome é obrigatório", "e-mail com formato válido" | DTO | Validação de formato, sem consulta ao banco. |
| "Não pode haver dois e-mails iguais" | Service (e UNIQUE no banco) | Regra de negócio que depende do estado do sistema. |
| "Desconto máximo de 20%" | Service | Regra de negócio. |
| Cálculos e decisões | Service | Controller e Repository não devem decidir. |
| Consulta ao banco (`findBy...`, `@Query`) | Repository | Isolar o acesso aos dados. |
| Nome da tabela, colunas, relacionamentos | Entity | Mapeamento objeto-relacional. |
| Lista fixa de valores (status, perfil, forma de pagamento) | Enum | Evita valores inválidos. |
| Conversão de exceção em status HTTP | `@RestControllerAdvice` | Tratamento centralizado de erros. |
| Controle de transação (`@Transactional`) | Service | É o Service que define o que forma uma operação atômica. |

---

# 🚨 8. Erros frequentes de iniciantes

| Erro | Consequência | Correção |
|---|---|---|
| Colocar `if` de regra de negócio no Controller | Regra duplicada e difícil de testar. | Mover para o Service. |
| Chamar o Repository direto no Controller | Ignora as regras de negócio. | Controller → Service → Repository. |
| Receber/devolver a Entity na API | Expõe campos internos e acopla API ao banco. | Usar `RequestDTO` e `ResponseDTO`. |
| Esquecer `@Valid` no parâmetro | As validações do DTO **não executam**. | Adicionar `@Valid` antes do `@RequestBody`. |
| Validar existência de e-mail no DTO | Bean Validation não consulta o banco. | Fazer no Service (`existsByEmail`). |
| Esquecer o construtor sem parâmetros na Entity | Falha ao carregar dados do banco. | Adicionar o construtor (pode ser `protected`). |
| Usar `@Modifying` sem transação | Erro ao executar `UPDATE`/`DELETE`. | Usar `@Transactional` (preferencialmente no Service). |
| Nomes diferentes entre Controller e Service (`buscarPorId` × `buscaPorId`) | Erro de compilação. | Padronizar os nomes. |
| `ResponseEntity.OK(...)` | Erro de compilação (Java diferencia maiúsculas). | `ResponseEntity.ok(...)`. |

---

# 🧭 9. Resumo final em uma tabela

| # | Camada | Em uma frase |
|---|---|---|
| 1 | **Controller** | Recebe a requisição HTTP e devolve a resposta HTTP. Não decide nada. |
| 2 | **DTO** | Define e valida o formato do que entra e sai da API. |
| 3 | **Service** | Aplica as regras de negócio e controla as transações. |
| 4 | **Repository** | Conversa com o banco de dados. |
| 5 | **Entity** | É a tabela do banco representada como classe Java. |
| 6 | **Enum** | Limita um campo a valores fixos e conhecidos. |
| 7 | **Mapping (REST)** | Liga cada verbo HTTP + URL ao método correto do Controller. |

**Ordem de leitura recomendada para estudar:** Entity → Enum → Repository → DTO → Service → Controller → Mapping RESTful API → este arquivo.

**Ordem de construção recomendada de um novo recurso:** Enum → Entity → Repository → DTOs → Service → Controller → tratamento de exceções.

---

# 📖 10. Glossário técnico completo

| Termo | Definição didática |
|---|---|
| **ACID** | Propriedades essenciais de transações bancárias/SQL: Atomicidade, Consistência, Isolamento e Durabilidade. |
| **`@Autowired`** | Anotação do Spring para injeção automática de dependência. *Nota: a injeção via construtor é preferível em código moderno.* |
| **`@Async`** | Anotação que indica que a execução de um método deve ocorrer de forma assíncrona em uma thread separada. |
| **Atomicidade** | Propriedade "tudo ou nada": se qualquer etapa de uma operação falhar, todas as alterações anteriores são desfeitas. |
| **`AttributeConverter`** | Classe JPA usada para converter o valor de um tipo Java no tipo correspondente da coluna do banco (e vice-versa). |
| **Auditoria** | Rastreamento automático de criação e modificação de registros (ex.: `@CreatedDate`, `@LastModifiedDate`). |
| **Bean** | Qualquer objeto cuja criação, ciclo de vida e dependências são gerenciados diretamente pelo Spring IoC Container. |
| **Bean Validation** | Especifição padrão Java (Jakarta) para validação declarativa através de anotações (`@NotBlank`, `@Size`, etc.). |
| **`BusinessException`** | Nome padrão para exceção personalizada lançada pela camada Service para indicar violações de regras de negócio. |
| **Cache** | Mecanismo para armazenar temporariamente dados acessados frequentemente, evitando consultas custosas ao banco. |
| **Cache Hit** | Evento quando o dado solicitado é encontrado com sucesso na memória cache sem precisar acessar a fonte primária. |
| **Camada** | Divisão lógica e estrutural do código responsável por um papel específico na arquitetura da aplicação. |
| **Cardinalidade** | Proporção de relacionamentos entre tabelas/entidades (ex.: 1:1, 1:N, N:N). |
| **Cascade** | Atributo JPA que permite propagar ações executadas em uma entidade pai para suas entidades filhas agregadas. |
| **Chave Estrangeira (FK)** | Campo em uma tabela que estabelece ligação referente à Chave Primária de outra tabela. |
| **Chave Primária (PK)** | Identificador único e obrigatório de cada registro/linha de uma tabela. |
| **Cliente** | A parte consumidora da API (ex.: aplicação Frontend Web, app mobile, Postman ou outro sistema). |
| **Commit** | Efetivação definitiva de todas as alterações feitas durante uma transação no banco de dados. |
| **Constraint** | Regra de integridade imposta diretamente no esquema da tabela do banco (`UNIQUE`, `NOT NULL`, `CHECK`). |
| **`consumes`** | Atributo de mapeamento REST que especifica o tipo de mídia (MIME) aceito no corpo da requisição HTTP (ex.: JSON). |
| **Contêiner IoC** | Mecanismo do Spring Framework que gerencia a instanciação e Injeção de Dependências dos Beans. |
| **Controller** | Camada responsável pela recepção das requisições HTTP, validação de entrada básica e emissão de respostas. |
| **CORS** | (Cross-Origin Resource Sharing) Mecanismo de segurança dos navegadores que restringe chamadas entre origens distintas. |
| **CRUD** | Acrônimo para as quatro operações fundamentais em persistência: Create, Read, Update e Delete. |
| **`DispatcherServlet`** | Ponto de entrada central do Spring MVC que intercepta as requisições HTTP e as roteia aos Controllers corretos. |
| **Dependência** | Objeto ou serviço do qual outra classe precisa para conseguir executar suas atribuições. |
| **DTO** | (Data Transfer Object) Objeto simples e imutável projetado apenas para trafegar dados entre diferentes camadas. |
| **Encapsulamento** | Princípio da POO de proteger os campos de um objeto, liberando acesso apenas através de métodos controlados. |
| **Entity** | Classe Java cujas instâncias e propriedades representam linhas e colunas de uma tabela do banco de dados. |
| **`@EntityGraph`** | Anotação do Spring Data JPA para resolver o problema N+1 ao forçar o carregamento antecipado de relacionamentos. |
| **Enum** | Tipo especial de classe Java destinado à representação de um grupo fixo e imutável de constantes. |
| **`equals()` / `hashCode()`** | Métodos do Java que definem comparabilidade lógica entre objetos e calculam suas chaves de dispersão. |
| **`@ExceptionHandler`** | Anotação utilizada para definir métodos específicos de tratamento para exceções capturadas no sistema. |
| **Fetch (`LAZY`/`EAGER`)** | Configuração de carregamento de relacionamentos: sob demanda (`LAZY`) ou imediata/ansiosa (`EAGER`). |
| **GET** | Verbo HTTP seguro e idempotente destinado exclusivamente à recuperação de dados sem efeitos colaterais no servidor. |
| **Getter / Setter** | Métodos utilitários de acesso (leitura) e modificação (escrita) para atributos privados. |
| **Hibernate** | Framework ORM padrão que implementa a especificação JPA no Spring Boot. |
| **Hibernate Validator** | A implementação oficial de referência da especificação Bean Validation (Jakarta Validation). |
| **`@Id`** | Anotação JPA necessária para indicar qual campo da Entity atua como chave primária. |
| **Idempotência** | Propriedade na qual múltiplas requisições idênticas produzem exatamente o mesmo efeito colateral no servidor. |
| **Injeção de Dependência** | Padrão no qual a criação de objetos dependentes é delegada ao Spring em vez de ser instanciada pela própria classe. |
| **Injeção por Construtor** | Boas práticas de injeção no Spring onde dependências chegam como parâmetros no construtor da classe. |
| **Header (HTTP)** | Metadado transmitido em requisições e respostas HTTP informando contexto extra (`Authorization`, `Content-Type`). |
| **HTTP** | (Hypertext Transfer Protocol) Protocolo de camada de aplicação base para troca de mensagens e APIs na Web. |
| **JPA** | (Jakarta Persistence API) Especificação padrão Java para Mapeamento Objeto-Relacional e persistência de dados. |
| **`JpaRepository`** | Interface central do Spring Data JPA que disponibiliza diversos métodos de persistência prontos para uso. |
| **JPQL** | (Java Persistence Query Language) Linguagem de consulta do JPA focada em Entidades e atributos em vez de tabelas. |
| **JSON** | (JavaScript Object Notation) Formato leve e legível para estruturação e transferência de dados em aplicações REST. |
| **Jackson** | Biblioteca Java padrão do Spring Boot para converter JSON em objetos Java (desserialização) e vice-versa (serialização). |
| **`JOIN FETCH`** | Cláusula JPQL usada para forçar o carregamento de relacionamentos `LAZY` em uma única consulta SQL JOIN. |
| **`@Lazy`** | Anotação que adia o momento de instanciação de um Bean do Spring ou carregamento de uma Entidade até que seja acessada. |
| **LOB** | (Large Object) Mapeamento para grandes volumes de dados no banco: `BLOB` (binário) e `CLOB` (texto longo). |
| **`@Lock`** | Anotação do Spring Data JPA usada para definir o tipo de bloqueio concorrente (Otimista ou Pessimista) em operações. |
| **Lock Otimista** | Mecanismo de concorrência que checa versões do dado no instante do salvamento para detectar alterações simultâneas. |
| **Lock Pessimista** | Bloqueio de registro a nível de banco de dados (`SELECT FOR UPDATE`), impedindo que outros leiam/alterem durante a transação. |
| **`Location` (Header)** | Cabeçalho HTTP retornado em respostas `201 Created` contendo a URI para o novo recurso cadastrado. |
| **Mapper** | Componente ou utilitário focado puramente em realizar a conversão entre objetos DTO e Entities (ex.: MapStruct). |
| **Mapping** | Associação feita entre caminhos e verbos HTTP a métodos executáveis no Controller. |
| **`@MappedSuperclass`** | Anotação para superclasses cujos campos declarados devem ser herdados por subentidades JPA no banco. |
| **Método Derivado** | Consulta montada automaticamente pelo Spring Data apenas analisando a nomenclatura dada ao método na interface do Repository. |
| **`@Modifying`** | Anotação exigida no Repository junto de `@Query` quando a consulta resulta em alterações (`UPDATE`, `DELETE`, `INSERT`). |
| **Native Query** | Consulta declarada diretamente na sintaxe nativa SQL do banco de dados relacional em uso. |
| **ORM** | (Object-Relational Mapping) Técnica de desenvolvimento para mapeamento entre estruturas Orientadas a Objetos e tabelas relacionais. |
| **Parâmetro Nomeado** | Sintaxe na JPQL (`:nomeParametro`) usada para associar dinamicamente variáveis aos valores injetados via `@Param`. |
| **`@PathVariable`** | Anotação que extrai parâmetros dinâmicos diretamente da estrutura de caminho da URI (ex.: `/alunos/{id}`). |
| **PATCH** | Verbo HTTP utilizado para aplicar modificações parciais a um recurso existente. |
| **Persistência** | O processo de gravar o estado de objetos em um armazenamento não-volátil (banco de dados). |
| **POST** | Verbo HTTP não-idempotente focado no envio de dados para criação de novos recursos no servidor. |
| **`produces`** | Atributo das anotações de Mapeamento indicando o formato da resposta enviada ao cliente (`application/json`). |
| **PUT** | Verbo HTTP idempotente projetado para a substituição completa ou atualização integral do recurso especificado. |
| **Query String** | Conjunto de parâmetros anexados ao final da URL através da estrutura `?chave=valor&chave2=valor2`. |
| **`@Query`** | Anotação utilizada para definir consultas customizadas na interface do Repository via JPQL ou SQL nativo. |
| **Record** | Recurso do Java para criação de estruturas imutáveis contendo construtores, getters e `equals/hashCode` automáticos. |
| **Repository** | Camada encarregada de isolar a lógica e comandos do mecanismo de persistência de dados. |
| **`@RequestBody`** | Anotação que indica que o corpo da requisição HTTP deve ser convertido no objeto parâmetro indicado. |
| **`@RequestHeader`** | Anotação utilizada para extrair dados diretos enviados dentro dos cabeçalhos da requisição HTTP. |
| **`@RequestMapping`** | Anotação base responsável por mapear rotas, parâmetros, verbos e tipos de dados em Controllers. |
| **`@RequestParam`** | Anotação que lê parâmetros passados na Query String da URL. |
| **RequestDTO** | DTO formatado exclusivamente para recepcionar e validar os dados trazidos pela requisição HTTP. |
| **ResponseDTO** | DTO projetado especificamente para selecionar e formatar os campos expostos como resposta da API. |
| **`ResponseEntity`** | Invólucro do Spring para construir a resposta HTTP completa (incluindo Corpo, Status Code e Headers). |
| **`@ResponseStatus`** | Anotação para definir diretamente o código de status HTTP retornado por um método ou exceção. |
| **REST** | Estilo arquitetural orientado a recursos identificados por URIs e manipulados via verbos padrões HTTP. |
| **RESTful** | Classificação concedida aos sistemas e APIs que implementam fielmente as restrições e regras do padrão REST. |
| **Rollback** | Ação de desfazer e reverter todas as alterações operadas em uma transação quando ocorre uma falha ou erro. |
| **Rota** | O endereço/caminho (URI) associado à chamada de determinado método do Controller. |
| **`@RestController`** | Anotação especializada que define a classe como Controller REST cujos retornos são serializados direto em JSON. |
| **`@RestControllerAdvice`** | Interceptador global para tratamento centralizado de exceções lançadas em qualquer lugar da camada Controller. |
| **`@RestResource`** | Anotação usada no Spring Data REST para customizar endpoints gerados automaticamente a partir de interfaces. |
| **Regra de Negócio** | Condição, restrição ou fluxo indispensável que atende às necessidades lógicas da aplicação no mundo real. |
| **SGBD** | Sistema de Gerenciamento de Banco de Dados (ex.: PostgreSQL, MySQL, Oracle). |
| **Serialização** | Processo de conversão do estado de um objeto Java em um formato de transmissão legível (como JSON). |
| **Service** | Camada encarregada de concentrar as regras de negócio, casos de uso e controle das transações da aplicação. |
| **`@Service`** | Anotação do Spring que classifica a classe como um componente Bean da camada de serviços. |
| **SQL** | Linguagem de consulta estruturada usada para gerenciar e consultar dados em SGBDs relacionais. |
| **Spring Boot** | Extensão da plataforma Spring criada para simplificar a configuração, inicialização e deploy de aplicações. |
| **Spring Data JPA** | Módulo que simplifica significativamente a implementação de repositórios JPA com suporte automático a métodos. |
| **Spring Data REST** | Módulo capaz de expor automaticamente repositórios Spring Data como endpoints HTTP RESTful. |
| **Stateless** | Princípio REST onde o servidor não armazena nenhum estado sobre a sessão do cliente entre requisições. |
| **Status HTTP** | Código de resposta numérico de 3 dígitos enviado pelo servidor indicando o resultado do processamento da requisição. |
| **Stereotype** | Anotações do Spring usadas para definir e categorizar os papéis desempenhados pelos Beans (`@Component`, `@Service`, etc.). |
| **Stored Procedure** | Procedimento ou função armazenada e compilada diretamente no banco de dados. |
| **`@Table`** | Anotação JPA para explicitar o nome exato e configurações da tabela vinculada à Entity. |
| **`@Transactional`** | Anotação para delimitar a fronteira de uma transação do banco de dados em um método/classe. |
| **`@Transient`** | Anotação que orienta o JPA a ignorar o campo anotado, não mapeando-o para colunas no banco de dados. |
| **Transação** | Conjunto indivisível de operações executadas no banco com garantia de sucesso total ou reversão completa. |
| **`@TransactionalEventListener`** | Event listener que aciona apenas em etapas específicas da transação (ex.: `AFTER_COMMIT`). |
| **URI** | (Uniform Resource Identifier) Endereço usado para identificar um recurso no servidor (ex.: `/api/v1/alunos/1`). |
| **`@Valid`** | Anotação para disparar a validação encadeada do Bean Validation no parâmetro recebido. |
| **`@Validated`** | Variante da anotação do Spring que habilita a validação de métodos em nível de classe e suporte a grupos. |
| **Validação Estrutural** | Verificação inicial sobre o formato e consistência dos dados de entrada realizada na camada DTO. |
| **Validação Semântica** | Verificação das regras de negócio que dependem do estado atual da aplicação, realizada no Service. |
| **Validação de Persistência** | Última barreira de integridade promovida pelas constraints e tipos declarados no banco de dados. |
| **Verbo HTTP** | Método que indica qual ação deve ser realizada sobre o recurso (`GET`, `POST`, `PUT`, `DELETE`, `PATCH`). |
