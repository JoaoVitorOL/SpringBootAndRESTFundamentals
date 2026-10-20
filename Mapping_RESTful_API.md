# Camada MAPPING — RESTful API (PT-BR)

## 🎯 Função principal

O **Mapping** (mapeamento) é o mecanismo que **associa uma requisição HTTP (verbo + URL) a um método Java** do Controller.
Ele define **qual código será executado** quando o cliente acessa determinada rota, **quais dados entram** (parâmetros, corpo, headers) e **qual resposta HTTP sai** (status + corpo).

* O mapping **não é uma camada separada**: ele é declarado **dentro da camada Controller**, por meio de anotações.
* Uma API **RESTful** é uma API que segue as restrições do estilo arquitetural **REST** (*Representational State Transfer*), usando o protocolo HTTP de forma padronizada.

Fluxo interno do Spring ao receber uma requisição: <br>
**Cliente → DispatcherServlet → HandlerMapping → Controller (método mapeado) → Service → Repository → resposta HTTP → Cliente**

| Componente | Papel |
|---|---|
| `DispatcherServlet` | Ponto de entrada único; recebe todas as requisições HTTP. |
| `HandlerMapping` | Consulta as anotações de mapping e descobre qual método Java atende à requisição (verbo + URL). |
| Controller | Executa o método encontrado, delegando as regras de negócio ao Service. |

---

# 📌 Princípios REST

| Princípio | Descrição | Consequência prática |
|---|---|---|
| Cliente-Servidor | Cliente e servidor são independentes. | O mesmo back-end atende web, mobile, etc. |
| Stateless (sem estado) | Cada requisição contém tudo que o servidor precisa; o servidor **não guarda sessão** entre requisições. | Autenticação enviada em todas as requisições (ex.: header `Authorization`). |
| Recursos identificados por URI | Tudo é um **recurso** (aluno, pedido, chamado) com um endereço único. | `/alunos/10` identifica o aluno de id 10. |
| Interface uniforme | Operações padronizadas pelos **verbos HTTP**. | GET lê, POST cria, PUT substitui, DELETE remove. |
| Representações | O recurso trafega em um formato (normalmente **JSON**). | `@RequestBody` e `@ResponseBody` convertem JSON ↔ objeto Java. |
| Respostas padronizadas | O resultado é informado por **códigos de status HTTP**. | `201 Created`, `404 Not Found`, etc. |

---

# 📖 Mapeamento de RESTful API

## 🛣️ 1. Verbo HTTP × Anotação × Operação

| Verbo | Anotação | Operação CRUD | Seguro¹ | Idempotente² | Uso típico |
|---|---|---|---|---|---|
| GET | `@GetMapping` | Read (leitura) | Sim | Sim | Listar ou buscar recursos. |
| POST | `@PostMapping` | Create (criação) | Não | Não | Criar um novo recurso. |
| PUT | `@PutMapping` | Update (substituição total) | Não | Sim | Substituir o recurso inteiro. |
| PATCH | `@PatchMapping` | Update (parcial) | Não | Não necessariamente | Alterar apenas alguns campos. |
| DELETE | `@DeleteMapping` | Delete (remoção) | Não | Sim | Remover um recurso. |

¹ **Seguro:** a operação não altera o estado do servidor. <br>
² **Idempotente:** executar a operação 1 vez ou N vezes produz o **mesmo estado final** no servidor. (Ex.: `DELETE /alunos/10` repetido deixa o servidor no mesmo estado: aluno 10 inexistente.)

> **PUT × PATCH:** PUT envia o recurso **completo** (campos omitidos são sobrescritos); PATCH envia **somente os campos alterados**.

---

## 🚦 2. Status HTTP × Situação

| Código | Nome | Quando retornar | Método Spring |
|---|---|---|---|
| 200 | OK | Leitura ou atualização bem-sucedida, com corpo na resposta. | `ResponseEntity.ok(corpo)` |
| 201 | Created | Recurso criado com sucesso (POST). Deve informar o header `Location`. | `ResponseEntity.created(uri).body(corpo)` |
| 204 | No Content | Sucesso sem corpo (ex.: DELETE). | `ResponseEntity.noContent().build()` |
| 400 | Bad Request | Dados inválidos (falha de validação estrutural, JSON malformado). | Tratado em `@RestControllerAdvice` |
| 401 | Unauthorized | Cliente não autenticado. | Spring Security |
| 403 | Forbidden | Cliente autenticado, mas sem permissão. | Spring Security |
| 404 | Not Found | Recurso não existe. | `ResponseEntity.notFound().build()` ou exceção tratada |
| 409 | Conflict | Violação de regra de negócio (ex.: "e-mail já cadastrado"). | Exceção tratada em `@RestControllerAdvice` |
| 500 | Internal Server Error | Falha inesperada no servidor. | Tratado em `@RestControllerAdvice` |

Faixas de código: **2xx** sucesso · **4xx** erro do cliente · **5xx** erro do servidor.

---

## 🧭 3. Desenho de URIs

### ✔️ Boas práticas
- Usar **substantivos no plural**, nunca verbos: `/alunos`, `/pedidos`.
- Usar **minúsculas** e hífen para separar palavras: `/ordens-de-servico`.
- Expressar hierarquia pela URL: `/clientes/5/chamados` (chamados do cliente 5).
- Versionar a API: `/api/v1/alunos`.
- Deixar o **verbo HTTP** indicar a ação.

### ❌ Evitar
| Incorreto | Correto | Motivo |
|---|---|---|
| `GET /buscarAlunos` | `GET /alunos` | Verbo na URL; o verbo HTTP já indica a ação. |
| `POST /criarAluno` | `POST /alunos` | Idem. |
| `DELETE /alunos/deletar/10` | `DELETE /alunos/10` | Redundância. |
| `GET /aluno` | `GET /alunos` | Coleção deve estar no plural. |

### Mapa completo de rotas do recurso `aluno`

| Verbo | URL | Ação | Status de sucesso |
|---|---|---|---|
| POST | `/api/v1/alunos` | Criar aluno | 201 |
| GET | `/api/v1/alunos` | Listar alunos | 200 |
| GET | `/api/v1/alunos/{id}` | Buscar aluno por id | 200 |
| PUT | `/api/v1/alunos/{id}` | Atualizar aluno | 200 |
| DELETE | `/api/v1/alunos/{id}` | Remover aluno | 204 |

---

## 🧩 4. Anotações de MAPEAMENTO (atributos)

| Anotação | Descrição |
|---|---|
| `@RestController` | Combina `@Controller` + `@ResponseBody`: o retorno dos métodos é serializado (JSON) diretamente no corpo da resposta. |
| `@RequestMapping` | Define a rota base (nível de classe) ou mapeamentos genéricos (nível de método). |
| `@GetMapping`, `@PostMapping`, `@PutMapping`, `@PatchMapping`, `@DeleteMapping` | Atalhos de `@RequestMapping(method = ...)` para cada verbo HTTP. |

Atributos aceitos pelas anotações de mapping:

| Atributo | Descrição | Exemplo |
|---|---|---|
| `value` / `path` | URL (ou fragmento de URL) mapeada. | `@GetMapping("/{id}")` |
| `method` | Verbo HTTP (apenas em `@RequestMapping`). | `@RequestMapping(method = RequestMethod.GET)` |
| `consumes` | Formato **aceito** no corpo da requisição (header `Content-Type`). | `consumes = "application/json"` |
| `produces` | Formato **devolvido** na resposta (header `Accept`). | `produces = "application/json"` |
| `params` | Exige a presença de um parâmetro de query. | `params = "ativo=true"` |
| `headers` | Exige um header específico. | `headers = "X-Versao=1"` |

**Combinação de rotas:** a rota final é a **soma** da rota da classe com a rota do método.

```java
@RestController
@RequestMapping("/api/v1/alunos")   // rota base
public class AlunoController {

    @GetMapping("/{id}")             // rota final: GET /api/v1/alunos/{id}
    public ...
}
```

---

## 📥 5. Anotações de PARÂMETROS

| Anotação | Origem do dado | Exemplo de URL | Atributos úteis |
|---|---|---|---|
| `@PathVariable` | Trecho da **URL** | `/alunos/10` | `name`, `required` |
| `@RequestParam` | **Query string** (`?chave=valor`) | `/alunos?nome=ana` | `name`, `required`, `defaultValue` |
| `@RequestBody` | **Corpo** da requisição (JSON → objeto Java) | `POST /alunos` com JSON | `required` |
| `@RequestHeader` | **Header** HTTP | `Authorization: Bearer ...` | `name`, `required`, `defaultValue` |

**Quando usar cada um (critério de decisão):**

| Situação | Anotação adequada |
|---|---|
| Identificar **um recurso específico** | `@PathVariable` |
| **Filtrar, ordenar ou paginar** uma coleção | `@RequestParam` |
| Enviar **dados para criar/atualizar** um recurso | `@RequestBody` |
| Enviar **metadados** (token, idioma, versão) | `@RequestHeader` |

```java
// Exemplo combinando as três origens em uma única rota
// URL: GET /api/v1/alunos/10/notas?bimestre=2   |  Header: Accept-Language: pt-BR
@GetMapping("/{id}/notas")
public ResponseEntity<List<NotaResponseDTO>> listarNotas(
        @PathVariable("id") Long id,                                   // vem da URL: 10
        @RequestParam(name = "bimestre", defaultValue = "1") int bimestre, // vem da query: 2 (padrão 1)
        @RequestHeader("Accept-Language") String idioma) {             // vem do header
    ...
}
```

---

## 📤 6. Respostas: `ResponseEntity` × `@ResponseStatus`

Existem **duas formas** de definir o status HTTP da resposta.

| Alternativa | Como funciona | Prós | Contras |
|---|---|---|---|
| `ResponseEntity<T>` | Objeto que representa a resposta completa (status + headers + corpo), construído no método. | Controle total; permite status dinâmico e headers (ex.: `Location`). | Código mais verboso; o tipo de retorno fica mais complexo. |
| `@ResponseStatus` | Anotação que fixa o status do método (ex.: `@ResponseStatus(HttpStatus.CREATED)`). | Código mais enxuto; retorno direto do objeto. | Status **fixo**; não permite headers nem status condicional. |

```java
// Alternativa A: ResponseEntity (recomendada quando é necessário o header Location)
@PostMapping
public ResponseEntity<AlunoResponseDTO> criar(@Valid @RequestBody AlunoRequestDTO dto) { ... }

// Alternativa B: @ResponseStatus (status fixo, retorno direto)
@PostMapping
@ResponseStatus(HttpStatus.CREATED)
public AlunoResponseDTO criar(@Valid @RequestBody AlunoRequestDTO dto) { ... }
```

Métodos estáticos mais usados de `ResponseEntity`:

| Método | Resultado |
|---|---|
| `ResponseEntity.ok(corpo)` | 200 com corpo. |
| `ResponseEntity.created(uri).body(corpo)` | 201 com header `Location` e corpo. |
| `ResponseEntity.noContent().build()` | 204 sem corpo. |
| `ResponseEntity.notFound().build()` | 404 sem corpo. |
| `ResponseEntity.status(HttpStatus.X).body(corpo)` | Status customizado. |

> Atenção: os nomes são **case-sensitive**. É `ok(...)`, e não `OK(...)`. O tipo sem corpo é `ResponseEntity<Void>` (com `V` maiúsculo).

---

## 🛡️ 7. Tratamento de erros: `@RestControllerAdvice`

Sem tratamento central, uma exceção lançada no Service resulta em **500** genérico. Para devolver o status correto (404, 409, 400), usa-se um **handler global**.

| Anotação | Descrição |
|---|---|
| `@RestControllerAdvice` | Classe global que intercepta exceções de **todos** os Controllers e devolve o resultado como JSON. |
| `@ExceptionHandler(Tipo.class)` | Método que trata um tipo específico de exceção. |
| `@ExceptionHandler` **dentro do Controller** | Trata exceções **apenas daquele** Controller (alcance local). |

**Mapeamento exceção → status HTTP**

| Exceção | Origem | Status |
|---|---|---|
| `RecursoNaoEncontradoException` | Service (`findById` sem resultado) | 404 |
| `BusinessException` | Service (violação de regra de negócio) | 409 |
| `MethodArgumentNotValidException` | Falha do `@Valid` no DTO (lançada pelo Spring) | 400 |

---

# 🧪 Exemplo completo (CRUD de Aluno)

Pré-requisito: dependência `spring-boot-starter-validation` no projeto (necessária para o `@Valid` funcionar).

## 7.1 DTOs

```java
// AlunoRequestDTO.java — dados que ENTRAM na API
public record AlunoRequestDTO(

        @NotBlank(message = "Nome é obrigatório")          // não aceita null, "" ou apenas espaços
        @Size(min = 3, max = 50, message = "Nome deve ter entre 3 e 50 caracteres")
        String nome,

        @NotBlank(message = "E-mail é obrigatório")
        @Email(message = "E-mail inválido")                // valida apenas o FORMATO do e-mail
        String email
) {}
```

```java
// AlunoResponseDTO.java — dados que SAEM da API (nunca expõe a entidade diretamente)
public record AlunoResponseDTO(Long id, String nome, String email) {}
```

## 7.2 Contrato esperado do Service

O Controller depende destes métodos. **Os nomes devem ser idênticos** nas duas camadas.

| Método do Service | Retorno | Observação |
|---|---|---|
| `salvar(AlunoRequestDTO dto)` | `AlunoResponseDTO` | Valida regras de negócio (ex.: e-mail duplicado → `BusinessException`). |
| `listarTodos()` | `List<AlunoResponseDTO>` | Leitura (`@Transactional(readOnly = true)`). |
| `buscarPorId(Long id)` | `AlunoResponseDTO` | Lança `RecursoNaoEncontradoException` se não existir. |
| `atualizar(Long id, AlunoRequestDTO dto)` | `AlunoResponseDTO` | Lança `RecursoNaoEncontradoException` se não existir. |
| `deletar(Long id)` | `void` | Lança `RecursoNaoEncontradoException` se não existir. |

> **Onde converter DTO ↔ Entidade?** Existem três alternativas viáveis:
>
> | Alternativa | Prós | Contras |
> |---|---|---|
> | No **Controller** | Service trabalha só com entidades. | Controller ganha responsabilidade de conversão; risco de duplicação. |
> | No **Service** (adotada neste exemplo) | Controller permanece enxuto; a conversão fica junto das regras de negócio. | Service passa a conhecer os DTOs. |
> | Classe **Mapper** dedicada (manual ou MapStruct) | Conversão centralizada e reutilizável; menos código repetitivo. | Adiciona uma classe (ou dependência) ao projeto. |

## 7.3 Controller

```java
@RestController                                   // 1
@RequestMapping("/api/v1/alunos")                 // 2
public class AlunoController {

    private final AlunoService alunoService;      // 3

    public AlunoController(AlunoService alunoService) {   // 4
        this.alunoService = alunoService;
    }

    @PostMapping                                  // 5
    public ResponseEntity<AlunoResponseDTO> criar(@Valid @RequestBody AlunoRequestDTO dto) {  // 6
        AlunoResponseDTO criado = alunoService.salvar(dto);                                   // 7
        URI location = ServletUriComponentsBuilder.fromCurrentRequest()                       // 8
                .path("/{id}")
                .buildAndExpand(criado.id())
                .toUri();
        return ResponseEntity.created(location).body(criado);                                 // 9
    }

    @GetMapping                                   // 10
    public ResponseEntity<List<AlunoResponseDTO>> listar() {
        return ResponseEntity.ok(alunoService.listarTodos());                                 // 11
    }

    @GetMapping("/{id}")                          // 12
    public ResponseEntity<AlunoResponseDTO> buscar(@PathVariable("id") Long id) {
        return ResponseEntity.ok(alunoService.buscarPorId(id));
    }

    @PutMapping("/{id}")                          // 13
    public ResponseEntity<AlunoResponseDTO> atualizar(
            @PathVariable("id") Long id,
            @Valid @RequestBody AlunoRequestDTO dto) {
        return ResponseEntity.ok(alunoService.atualizar(id, dto));
    }

    @DeleteMapping("/{id}")                       // 14
    public ResponseEntity<Void> deletar(@PathVariable("id") Long id) {
        alunoService.deletar(id);
        return ResponseEntity.noContent().build();                                            // 15
    }
}
```

**Documentação linha a linha**

| Nº | Trecho | Explicação |
|---|---|---|
| 1 | `@RestController` | Registra a classe como bean do Spring e serializa os retornos em JSON. |
| 2 | `@RequestMapping("/api/v1/alunos")` | Rota base: todos os métodos herdam este prefixo. |
| 3 | `private final AlunoService alunoService` | Dependência declarada como `final`: não pode ser reatribuída após a construção. |
| 4 | Construtor | **Injeção por construtor**: o Spring fornece o `AlunoService` automaticamente. (Ver comparação com `@Autowired` abaixo.) |
| 5 | `@PostMapping` | Mapeia `POST /api/v1/alunos`. |
| 6 | `@Valid @RequestBody AlunoRequestDTO dto` | `@RequestBody` converte o JSON em DTO; `@Valid` executa as validações do DTO. Se falharem, o Spring lança `MethodArgumentNotValidException` **antes** de executar o método (resultando em 400). |
| 7 | `alunoService.salvar(dto)` | Delega ao Service; o Controller não contém regra de negócio. |
| 8 | `ServletUriComponentsBuilder...` | Monta a URI do novo recurso (ex.: `/api/v1/alunos/7`) a partir da URL da requisição atual. |
| 9 | `ResponseEntity.created(location).body(criado)` | Responde **201** com o header `Location` e o aluno criado no corpo. |
| 10 | `@GetMapping` | Mapeia `GET /api/v1/alunos` (coleção). |
| 11 | `ResponseEntity.ok(...)` | Responde **200** com a lista no corpo. |
| 12 | `@GetMapping("/{id}")` + `@PathVariable("id")` | Mapeia `GET /api/v1/alunos/{id}`; o valor de `{id}` na URL é injetado no parâmetro `id`. |
| 13 | `@PutMapping("/{id}")` | Mapeia `PUT /api/v1/alunos/{id}`; combina `@PathVariable` (qual recurso) e `@RequestBody` (novos dados). |
| 14 | `@DeleteMapping("/{id}")` | Mapeia `DELETE /api/v1/alunos/{id}`. |
| 15 | `ResponseEntity.noContent().build()` | Responde **204** sem corpo. |

**Injeção de dependência: `@Autowired` em campo × construtor**

| Alternativa | Prós | Contras |
|---|---|---|
| `@Autowired` no campo (usada nos arquivos anteriores) | Menos código. | Dificulta testes unitários; esconde dependências; impede campos `final`. |
| Injeção por construtor (usada neste exemplo) | Dependências explícitas e imutáveis (`final`); facilita testes sem Spring. | Um pouco mais de código. |

## 7.4 Exceções e tratamento global

```java
// RecursoNaoEncontradoException.java
public class RecursoNaoEncontradoException extends RuntimeException {
    public RecursoNaoEncontradoException(String mensagem) {
        super(mensagem);                         // repassa a mensagem para RuntimeException
    }
}
```

```java
// ErroResponse.java — formato padronizado de erro devolvido ao cliente
public record ErroResponse(int status, String mensagem, List<String> detalhes, LocalDateTime timestamp) {}
```

```java
// GlobalExceptionHandler.java
@RestControllerAdvice                                                           // 1
public class GlobalExceptionHandler {

    @ExceptionHandler(RecursoNaoEncontradoException.class)                      // 2
    public ResponseEntity<ErroResponse> tratarNaoEncontrado(RecursoNaoEncontradoException ex) {
        ErroResponse erro = new ErroResponse(
                HttpStatus.NOT_FOUND.value(), ex.getMessage(), List.of(), LocalDateTime.now());
        return ResponseEntity.status(HttpStatus.NOT_FOUND).body(erro);          // 3
    }

    @ExceptionHandler(BusinessException.class)                                  // 4
    public ResponseEntity<ErroResponse> tratarRegraDeNegocio(BusinessException ex) {
        ErroResponse erro = new ErroResponse(
                HttpStatus.CONFLICT.value(), ex.getMessage(), List.of(), LocalDateTime.now());
        return ResponseEntity.status(HttpStatus.CONFLICT).body(erro);
    }

    @ExceptionHandler(MethodArgumentNotValidException.class)                    // 5
    public ResponseEntity<ErroResponse> tratarValidacao(MethodArgumentNotValidException ex) {
        List<String> campos = ex.getBindingResult().getFieldErrors().stream()   // 6
                .map(e -> e.getField() + ": " + e.getDefaultMessage())
                .toList();
        ErroResponse erro = new ErroResponse(
                HttpStatus.BAD_REQUEST.value(), "Dados inválidos", campos, LocalDateTime.now());
        return ResponseEntity.status(HttpStatus.BAD_REQUEST).body(erro);
    }
}
```

| Nº | Trecho | Explicação |
|---|---|---|
| 1 | `@RestControllerAdvice` | Torna a classe um interceptador global de exceções; os retornos viram JSON. |
| 2 | `@ExceptionHandler(RecursoNaoEncontradoException.class)` | O método é chamado sempre que essa exceção for lançada em qualquer Controller (ou Service chamado por ele). |
| 3 | `ResponseEntity.status(...).body(erro)` | Define o status HTTP e o corpo de erro padronizado. |
| 4 | `BusinessException` | Exceção de regra de negócio (ex.: "E-mail já cadastrado"); mapeada para **409 Conflict**. |
| 5 | `MethodArgumentNotValidException` | Lançada pelo Spring quando o `@Valid` reprova o DTO. |
| 6 | `getBindingResult().getFieldErrors()` | Obtém a lista de campos inválidos; cada item fornece o nome do campo e a mensagem definida na anotação de validação. |

---

# 🔁 Exemplos de requisição e resposta

| Requisição | Resposta |
|---|---|
| `POST /api/v1/alunos` com `{"nome":"Ana Lima","email":"ana@email.com"}` | **201 Created**, header `Location: /api/v1/alunos/1`, corpo com o aluno criado. |
| `POST /api/v1/alunos` com `{"nome":"","email":"invalido"}` | **400 Bad Request**, corpo com `detalhes` listando os campos inválidos. |
| `POST /api/v1/alunos` com e-mail já cadastrado | **409 Conflict**, mensagem da regra de negócio. |
| `GET /api/v1/alunos` | **200 OK**, lista de alunos. |
| `GET /api/v1/alunos/999` (inexistente) | **404 Not Found**, corpo de erro padronizado. |
| `PUT /api/v1/alunos/1` com dados válidos | **200 OK**, aluno atualizado. |
| `DELETE /api/v1/alunos/1` | **204 No Content**, sem corpo. |

---

# ⚠️ Erros comuns de mapping

| Sintoma | Causa provável | Solução |
|---|---|---|
| `404` em rota que "deveria existir" | Rota da classe + rota do método diferente da URL chamada; barra duplicada ou ausente. | Conferir a soma `@RequestMapping` da classe + `@XxxMapping` do método. |
| `405 Method Not Allowed` | Rota existe, mas para outro verbo HTTP. | Conferir verbo usado × anotação do método. |
| `415 Unsupported Media Type` | Corpo enviado sem `Content-Type: application/json`. | Enviar o header correto. |
| `400` ao enviar JSON | JSON malformado ou campo com tipo incompatível. | Validar o JSON e os tipos do DTO. |
| `@Valid` não surte efeito | Dependência `spring-boot-starter-validation` ausente, ou `@Valid` omitido no parâmetro. | Adicionar a dependência e a anotação. |
| `PathVariable` não resolve | Nome em `{id}` diferente do nome do parâmetro, sem `name` explícito. | Usar `@PathVariable("id")` explicitamente. |
| Mesma rota mapeada duas vezes | Dois métodos com o mesmo verbo + URL. | Remover ou diferenciar um deles (`Ambiguous mapping`). |

---

## Resumo

| Pergunta | Resposta |
|---|---|
| O que o mapping define? | Qual método Java atende a qual verbo + URL. |
| Onde fica? | Na classe Controller, por anotações. |
| Como indicar a ação? | Pelo verbo HTTP (`@GetMapping`, `@PostMapping`, ...), nunca pela URL. |
| Como identificar o recurso? | Por URI com substantivo no plural e `@PathVariable`. |
| Como informar o resultado? | Por status HTTP adequado, via `ResponseEntity`. |
| Como tratar erros? | Por `@RestControllerAdvice` + `@ExceptionHandler`. |
