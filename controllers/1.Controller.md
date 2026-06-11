#  Camada CONTROLLER (PT-BR)

## 🎯 Função principal

A camada **Controller** é responsável por **receber requisições HTTP**, **validar entradas básicas**, **chamar os serviços** e **retornar respostas HTTP adequadas**.  
Ela **não** deve conter regras de negócio, isso é responsabilidade do *Service*.
<br>
Fluxo típico: <br>
**Cliente → Controller → Service → Repository → Service → Controller → Cliente**

---

# 📌 Responsabilidades da Camada Controller

### ✔️ O que o Controller faz
- Recebe requisições HTTP (GET, POST, PUT, DELETE)
- Valida dados de entrada básicos (ex.: `@Valid`)
- Converte objetos de requisição (DTOs) em entidades
- Chama a camada de serviço
- Retorna respostas HTTP padronizadas (`ResponseEntity`)
- Controla rotas, parâmetros, path variables e query params

### ❌ O que o Controller NÃO deve fazer
- Não implementa regras de negócio
- Não faz validações complexas
- Não acessa diretamente o banco (Repository)
- Não implementa cálculos, verificações de permissão, etc.

  
---



# 📘 Anotações principais da camada Controller

## 🛣️ Anotações de MAPEAMENTO
| Anotação | Descrição |
|----------|-----------|
| `@RestController` | Notifica o Spring que a classe trata requisições REST |
| `@RequestMapping("/rota")` | Define a rota base do controlador. (caminho base da API) |
| `@GetMapping` | Mapeia requisições HTTP GET. |
| `@PostMapping` | Mapeia requisições HTTP POST. |
| `@PutMapping` | Mapeia requisições HTTP PUT. |
| `@DeleteMapping` | Mapeia requisições HTTP DELETE. |
| `@PatchMapping` | Mapeia requisições HTTP PATCH. |

---

## 📥 Anotações de PARÂMETROS
| Anotação | Descrição |
|----------|-----------|
| `@RequestBody` | Obtém o corpo da requisição (JSON → objeto Java). |
| `@PathVariable` | Obtém valores que estão na URL (ex.: `/aluno/10`). |
| `@RequestParam` | Obtém parâmetros opcionais ou obrigatórios da URL (ex.: `/buscar?nome=ana`). |
| `@RequestHeader` | Lê informações específicas do header da requisição (ex.: Authorization). |

---

## 🛡️ Anotações de VALIDAÇÃO
| Anotação | Descrição |
|----------|-----------|
| `@Valid` | Ativa as validações dos atributos (Bean Validation). |
| `@Validated` | Variante para validar métodos ou grupos de validação. |

---

## 🧱 Anotações EXTRA
| Anotação | Descrição |
|----------|-----------|
| `@ResponseStatus` | Define o código HTTP retornado pelo método (ex.: 201 CREATED). |
| `@CrossOrigin` | Permite acessar o endpoint de outros domínios (CORS). |
| `@ExceptionHandler` | Captura e trata erros personalizados dentro do controller. |



---

###  Exemplo de controller

```java
@RestController
@RequestMapping("/alunos")
public class AlunoController {

   @Autowired
   private AlunoService alunoService ;

  @PostMapping
  public ResponseEntity<Aluno> criar(@RequestBody Aluno aluno) {
    Aluno salvo = alunoService.salvar(aluno);
    return ResponseEntity.status(HttpStatus.CREATED).body(salvo);
   }  

  @GetMapping
  public ResponseEntity<List<Aluno>> listar() {
    List<Aluno> alunos = alunoService.listarTodos();
    return ResponseEntity.OK(alunos);
  }

  @PutMapping("/{id}")
  public ResponseEntity<Aluno> atualizar(@PathVariable Long id, @RequestBody Aluno alunoAtualizado) {
        Aluno aluno = alunoService.atualizar(id, alunoAtualizado);
        return ResponseEntity.ok(aluno);
    }

  @GetMapping("/{id}")
  public ResponseEntity<Aluno> buscar(@PathVariable("id") Long id) {
    Aluno aluno = alunoService.buscarPorId(id);
    return ResponseEntity.OK.(aluno);
   }

  @DeleteMapping("/{id}")
  public ResponseEntity<void> deletar(@PathVeriable("id")) {
    alunoService.deletar(id);
    return ResponseEntity.noContent().build();
   }

}
```






