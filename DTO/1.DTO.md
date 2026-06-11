#  Camada DTO (PT-BR)
Data Transfer Object

## 🎯 Função principal

A camada **DTO** é um objeto simples, geralmente composto apenas por atributos e métodos de acesso, sendo  utilizados para **transportar dados entre camadas**. O transporte é feito **sem utilizar a lógica de negócio**
e serve como **uma interface de comunicação segura e enxuta**.

São separados em: <br>
⚫ RequestDTO <br>
⚫ ResponseDTO  <br>

Em resumo, DTOs de response e request basicamente filtram as Entidades antes de transportá-las às outras camadas da aplicação.

**Por que usar DTOs?** <br>
📥 Encapsulamento de dados: Evita expôr diretamnte entidades do banco de dados. <br>
🔑 Segurança: permite controlar quais dados serão enviados e recebidos. <br>
🎭 Performance: reduz o volume de dados trafegados (Apenas o necessário).  <br>


## Características dos DTOs
❌ Não contém regras de negócio, apenas os dados de entidades. <br>
✔️ Realiza validações dos dados. <br>


# ⚙️ Etapas de validação de uma aplicação

## 1º Validação Estrutural ( Realizada na camada de DTO)
🎯 Objetivos: <br>
⚫ Garantir que dados preenchidos têm o formato esperado. <br>
⚫ Realizar a validação dos dados (Email existe? CPF existe? ...) <br>
⚫ Conferir campos obrigatórios. <br>
⚫ Tamanho máximo e mínimo dos dados. <br>
ferramentas: Bean Validation,

```java

public class UsuarioDTO {
 @NotBlank (message = "Nome é obrigatório")
 @Size (min = 3, max = 50)
 private String nome;

 @Email(message = "E-mail inválido")
 private string email;
}
```
Essas validações são automáticas com **@Valid** nos Controllers.

## 2º Validação Semântica ( Realizada na camada de Serviço)
🎯 Objetivos: <br>
⚫ Validar regras de negócio que dependem do contexto da aplicação. <br>
("Email já cadastrado", "Data de nascimento não pode ser 2 anos anteriores ao atual", "Nome de usuário já está em uso" ....)

```java
if (usuarioRepository.existsByEmail(dto.getEmail()) {
 throw new BusinessException("E-mail já cadastrado") ;
}
```

## 3º Validação de Persistência ( Realizada no Banco de Dados)
🎯 Objetivos: <br>
⚫ Garantir integridade referencial e unicidade no banco real da aplicação (Postgre, MySQL ...) <br>
⚫ Uso  de constraints do banco real: UNIQUE, FOREIGN KEY, PRIMARY KEY ...

```SQL
CREATE TABLE USUARIO (
id_usuario SERIAL NOT NULL,
nome VARCHAR(100) NOT NULL,
email_hash VARCHAR(320) UNIQUE NOT NULL,
senha_hash VARCHAR(100) NOT NULL,
data_criacao TIMESTAMP NOT NULL DEFAULT NOW(),
PRIMARY KEY (id_usuario)
);
```


# :open_book: Mapeamento de DTO 

### 👻 Validações de PRESENÇA E NULIDADE
| Anotação | Descrição |
|----------|-----------|
| `@NotNull` | Garante que o campo **não seja null** (Valida apenas que o valor não pode ser null). |
| `@NotEmpty` | Garante que o campo **não seja vazio "" e nem null**|
| `@NotBlank` | Garante que o campo **não seja vazio "", não seja nulo e nem apenas espaços em branco**  (Aplicável somente em Strings)|

### 🐋 validações de TAMANHO E VALOR
| Anotação | Descrição |
|----------|-----------|
| `@Size(min = x, max = y)` | Valida a length (comprimento) mínimo e máximo de strings,listas,arrays ...|
| `@Min(value = x)` | Define o valor mínimo para **números inteiros** |
| `@Max(value = y)` | Define o valor máximo para **números inteiros** |
| `@DecimalMin(value = x)` | Define o valor mínimo para **números decimais** |
| `@DecimalMax(value = y)` | Define o valor máximo para **números decimais** |
| `@Positive` | Exige que o valor seja **maior que zero**|
| `@Negative` | Exige que o valor seja **menor que zero**|
| `@PositiveOrZero` | Exige que o valor seja **maior igual a zero**|
|`@NegativeOrZero` | Exige que o valor seja **menor igual a zero**|


### 📫 Validações de FORMATO
| Anotação | Descrição |
|----------|-----------|
| `@Email` | Garante que o campo seja um email válido |
| `@Pattern(regexp="regex")` | Valida uma String usando uma expressão regular (regex)|
```java
@Pattern(regexp = "\\d{11}", message = "CPF deve ter 11 dígitos")
private String cpf;  //Valida a formatação, não serve para conferir se o CPF é verdadeiro!
```
| `@Digits(integer = x, fraction = y)` | Valida números que possuem um limite exato de casas inteiras e casas decimais|
```java
@Digits(integer = 5, fraction = 2)
private BigDecimal salario;
```

### ⏲️Validações de DATA
| Anotação | Descrição |
|----------|-----------|
| `@Past` | Garante que a data esteja no **passado** |
| `@PastOrPresent` | Garante que a data esteja no **passado ou presente** |
| `@Future` | Garante que a data esteja no **futuro**|
| `@FutureOrPresent` | Garante que a data esteja no **futuro ou presente**|


### 0️⃣|1️⃣  Validações de BOOLEANS E OBJETOS ANINHADOS
| Anotação | Descrição |
|----------|-----------|
| `@AssertTrue` | Exige que o campo seja **True** |
| `@AssertFalse` | Exige que o campo seja **False** |
| `@Valid`  | Ativa validação em objetos aninhados **(DTOs dentro de DTOs)**|



