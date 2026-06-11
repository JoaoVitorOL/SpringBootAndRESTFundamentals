#  Camada SERVICE (PT-BR)

## 🎯 Função principal

* A camada de serviço é responsável por conter a **Lógica de negócio** (composta por rergras de negócio), as quais definem **como** as operações devem ocorrer na aplicação.
  
* Service serve como um intermediário entre **Controller** e **Repository** <br>
  Controller --> Recebe e responde requisições <br>
  Service    --> Aplica regras de negócio <br>
  Repository --> Interage com o banco de dados <br>

## O que são as regras de negócio?

Regras de negócio são as condições exigidas pela sua aplicação (Dependendo do seu modelo de negócio),
para que sejam aceitas as requisições do usuário antes de interagir com o banco de dados.

| Exemplo de Regras e negócio |
|-----------|
|"Não é permitido criar o cadastro de dois clientes diferentes usando o mesmo email" |
|"CPF devem ser único, não pode repetir" |
|"Nome de usuário (username) não pode estar em uso." |
|"Senha deve ter no mínimo 8 caracteres, incluir letra maiúscula e número." |
|"Data de nascimento deve indicar idade ≥ 18 anos." |
|"Chamado só pode ser criado se o cliente associado existir."|
|"Cliente só pode ser deletado se não possuir chamados abertos."|
| "Usuário só pode ser ativado após verificação de e-mail."|
| "Chamado só pode ser alterado para “Concluído” se o status atual for “Em andamento”|
|"Não permitir agendamento para datas passadas."|
|"Tokens de recuperação de senha expiram após X minutos."|
|"Desconto máximo permitido é de 20% por pedido."|
|"Somente administradores podem alterar o perfil de um usuário."|
|"Toda operação deve registrar log de auditoria."|

## Quando  usar SERVICE? <br>

⚫ Sempre que houver regras de negócio <br>
⚫ Quando é necessário manipular dados antes de salvar/retornar <br>
⚫ Para centralizar a lógica e evitar duplicação <br>


## :beans: O que são Beans? <br>

:beans: São objetos Java gerenciados pelo Spring.
Quando você marca uma classe de serviço com @Service (ou outra anotação de stereotype), essa classe se torna um bean, ou seja: o Spring vai gerenciar sua instância, permitir injeção de dependências (ex.: repositórios), gerenciar seu escopo, transações, etc. Isso permite que sua camada de negócio seja desacoplada, testável e configurável.

# :open_book: Mapeamento de SERVIÇOS

### 🛠️ Anotação de INICIALIZAÇÃO
| Anotação | Descrição |
|----------|-----------|
| `@Service` | Indica ao Spring Boot que a classe faz parte da camada de serviço. |
| `@Autowired` | Faz automaticamente a injeção de dependencias, permitindo que o service use os métodos do repositório sem instanciá-los novamente. |
| `@Qualifier` | Permite indicar explicitamente qual bean será injetado (Usada quando há múltimas implementações de uma interface)|
<br>

### 💸  Anotação de TRANSAÇÃO
| Anotação | Descrição |
|----------|-----------|
| `@Transactional` | Indica que os métodos da classe (ou o método anotado) devem ser executados dentro de uma transação do banco de dados (Isso garante as propriedades ACID). |
| `@Transactional(readOnly = true)` | Variante de @Transactional usada para métodos de apenas leitura. |
| `@TransactionalEventListener` | Utilizada para executar um método somente após um evento ser publicado e após uma transação chegar em determinado estado. (reagir a algo que só deve ocorrer se a transação realmente tiver sido confirmada.)|
<br>
Estados possíveis: 
* TransactionPhase.BEFORE_COMMIT <br>
* TransactionPhase.AFTER_COMMIT (mais comum) <br>
* TransactionPhase.AFTER_ROLLBACK <br>
* TransactionPhase.AFTER_COMPLETION <br>
  
<br>
⚫ ACID: Atomicidade (tudo ou nada), Consistência, Isolamento e Durabilidade. Ou seja, se alguma operação falhar durante a transação, todas as mudanças serão revertidas (“rollback”). <br>
⚫ Use-a quando você for fazer operações de escrita/alteração no banco (save, update, delete), ou quando o método envolve várias operações que precisam ser atômicas (ex: salvar várias entidades, atualizar relacionamento,      etc.). <br>

<br>


### ➕  Anotações EXTRA
| Anotação | Descrição |
|----------|-----------|
| `@Lazy` | Adia a criação/inicialização do bean até ser efetivamente necessário, (bom para beans pesados ou dependências opcionais). |
| `@Validated` | Ativa validação de parâmetros no nível de método do service (Bean Validation). |
| `@Async` | Indica que o método será executado em outra thread, sem bloquear o fluxo principal.(Requer Classe de configuração contendo @EnableAsync)|
| `@Retryable` |Tenta executar o método novamente automaticamente caso ocorra erro específico. |
| `@Cacheable, @CacheEvict, @CachePut` | Quando o service utiliza caching|
<br>
O que é Caching?  <br>
Caching é uma técnica de otimização na qual resultados de operações custosas (como consultas ao banco, chamadas externas, cálculos pesados) são armazenados temporariamente em memória ou outro mecanismo rápido, para evitar recomputação. <br>

1.Um método é executado. <br>
2.O resultado é salvo no cache com uma chave. <br>
3.Na próxima execução com os mesmos parâmetros, <br>
Spring verifica o cache: <br>
Se existir: retorna imediatamente do cache <br>
Se não existir: executa o método e salva o novo valor no cache <br>

### ✔️ Usar quando
- Consultas ao banco são repetitivas  
- Dados mudam pouco  
- Operação é custosa (CPU, IO)  
- Chamada externa é lenta (HTTP, API)  
- Há necessidade de melhorar o tempo de resposta da aplicação  

### ❌ Não usar quando
- Os dados mudam constantemente  
- As operações exigem sempre o estado mais atualizado possível  
- Os parâmetros variam muito (reduzindo a chance de “cache hit”)  




# Exemplo de Service
```java

@Service
@Transactional
 public class AlunoService {
   @Autowired
   private AlunoRepository alunoRepository;

   public Aluno salvar(Aluno aluno) {
       //regra de negócio: não permitir nome vazio
     if (aluno.getNome() == null || aluno.getNome().isEmpty {
        throw new IllegalArgumetnException("Nome é obrigatório");
      }
      return alunoRepository.save(aluno);
    }


   public Aluno buscaPorId(Long id) {
     return alunoRepository.findById(id).OrElseThrow(()-> new RunTimeException("Não encontrado"));
   }


   public Aluno deletar(Long id) {
     Aluno aluno = buscarPorId(id);
     alunoRepository.delete(aluno);
   }

```






