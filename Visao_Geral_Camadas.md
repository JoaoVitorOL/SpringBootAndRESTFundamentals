# Visão Geral das Camadas — Fluxo de uma API Spring Boot (PT-BR)

## 🎯 Função principal

Este arquivo **resume e relaciona** as camadas estudadas (Controller, DTO, Service, Repository, Entity e Enum) e mostra **como elas trabalham juntas** em uma única requisição.

Cada arquivo anterior explica **uma camada isolada**. Este arquivo responde às perguntas:
- Qual é a **ordem** em que as camadas são acionadas?
- O que **entra e o que sai** de cada camada?
- **Onde** cada tipo de código deve ficar?

> Pré-requisito de leitura: os arquivos de cada camada. Para o detalhamento de rotas, verbos e status HTTP, consultar `Mapping_RESTful_API.md`.

---

# 🗺️ 1. Visão geral em uma imagem

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

# 📋 2. Resumo de cada camada

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

# 🔄 3. A jornada de uma requisição: `POST /api/v1/alunos`

Cenário: o cliente deseja **cadastrar um aluno**.

## 3.1 Linha do tempo

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

## 3.2 Transformação do dado ao longo do fluxo

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

# 🧪 4. O mesmo fluxo em código (camada por camada)

> O código abaixo usa as **correções** apontadas na análise dos arquivos originais (ex.: `String` com `S` maiúsculo, `ResponseEntity.ok`, `orElseThrow`). Os imports foram omitidos para manter o foco.

## 4.1 Enum — valores permitidos para o status

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

## 4.2 Entity — a tabela em formato Java

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

## 4.3 Repository — acesso ao banco

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

## 4.4 DTOs — o que entra e o que sai

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

## 4.5 Service — regras de negócio

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

## 4.6 Controller — porta de entrada HTTP

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

# 🛡️ 5. Onde cada validação acontece

A mesma informação passa por **três filtros**, cada um em uma camada diferente.

| Ordem | Tipo | Camada | Pergunta | Exemplo | Resultado em caso de falha |
|---|---|---|---|---|---|
| 1º | Estrutural | DTO (ativada pelo `@Valid` no Controller) | "O dado tem formato válido?" | Nome em branco; e-mail sem `@` | `400 Bad Request` |
| 2º | Semântica | Service | "O dado respeita as regras de negócio?" | E-mail já cadastrado | `409 Conflict` |
| 3º | Persistência | Banco de dados | "O dado respeita a integridade dos dados?" | `UNIQUE`, `NOT NULL`, `FOREIGN KEY` | Erro de integridade |

> **Por que três e não apenas uma?** Cada filtro protege contra um tipo diferente de erro. O DTO é rápido e barato (não consulta o banco). O Service conhece o contexto (consulta o banco). O banco é a **última barreira**, que funciona mesmo se outro sistema gravar dados sem passar pela API.

---

# ❓ 6. Onde colocar cada tipo de código?

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

# 🚨 7. Erros frequentes de iniciantes

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

# 📖 8. Glossário rápido

| Termo | Definição |
|---|---|
| **Bean** | Objeto criado e gerenciado pelo Spring. |
| **Injeção de dependência** | O Spring fornece a um objeto as dependências de que ele precisa, em vez de ele mesmo criá-las. |
| **JPA** | Especificação Java para mapear objetos a tabelas. O **Hibernate** é a implementação mais usada. |
| **JPQL** | Linguagem de consulta que usa nomes de entidades e atributos (não de tabelas e colunas). |
| **Transação** | Conjunto de operações executado como unidade: ou tudo é confirmado (`commit`) ou tudo é desfeito (`rollback`). |
| **ACID** | Atomicidade, Consistência, Isolamento e Durabilidade: propriedades de uma transação. |
| **Idempotente** | Operação que, repetida várias vezes, deixa o sistema no mesmo estado final que uma única execução. |
| **Bean Validation** | Conjunto de anotações de validação (`@NotBlank`, `@Size`, `@Email`...). |
| **Record** | Tipo Java imutável com construtor e acessores gerados automaticamente. |

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
