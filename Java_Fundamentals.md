# ☕ Guia Técnico: Java do Zero ao Avançado

> **Nível:** Zero ao Avançado  
> **Linguagem:** Java  
> **Fontes de referência principais:** [Java Language Specification](https://docs.oracle.com/javase/specs/jls/se25/html/index.html), [Java Virtual Machine Specification](https://docs.oracle.com/javase/specs/jvms/se25/html/index.html) e [Java SE API](https://docs.oracle.com/en/java/javase/25/docs/api/)  
> **Versão de referência:** Java 25 (LTS), com notas sobre Java 26 quando relevante  
> **Atualizado em:** 21/07/2026

---

## Prefácio

Este guia foi escrito para resolver um problema comum: aprender Java como uma coleção de palavras-chave e receitas sem construir um modelo mental da linguagem, da JVM e da biblioteca padrão. Esse aprendizado fragmentado funciona em exemplos pequenos, mas falha quando o engenheiro precisa explicar uma conversão, escolher uma coleção, preservar igualdade, controlar concorrência, interpretar bytecode, diagnosticar memória ou publicar uma aplicação reproduzível.

O objetivo aqui é formar esse modelo em camadas. Java é a linguagem; a JVM executa um formato de classes e bytecode; o JDK fornece compilador, ferramentas e APIs; frameworks e bibliotecas externas acrescentam outros contratos. Saber em qual camada um recurso nasce indica onde procurar sua especificação, como ele evolui e quais garantias realmente oferece.

O texto segue o mesmo padrão didático em todos os capítulos:

- definição clara antes da sintaxe;
- exemplo pequeno, compilável e contextualizado;
- **leitura guiada** dos trechos que podem confundir iniciantes;
- tabelas para comparar contratos e escolhas;
- alertas para armadilhas de produção;
- referências oficiais próximas da afirmação que sustentam.

Java 25 é a base por ser a versão LTS mais recente. Java 26, lançado em março de 2026, é a versão corrente não LTS na data desta revisão. Recursos de preview são identificados explicitamente: exigem `--enable-preview`, podem mudar e não devem ser confundidos com recursos permanentes.

---

## Como usar este guia

Você pode percorrer o material de três formas:

1. **Trilha júnior:** leia na ordem, execute os exemplos e explique o resultado com suas próprias palavras.
2. **Trilha pleno:** use o sumário para revisar tipos, coleções, streams, generics, exceções, concorrência e ferramentas.
3. **Trilha sênior:** procure contratos, custos, compatibilidade binária, Java Memory Model, observabilidade e impacto arquitetural.

Ao estudar qualquer recurso, responda sempre:

1. Isso pertence à linguagem, à JVM, ao JDK ou a uma dependência externa?
2. Qual contrato o compilador ou a API garante?
3. Qual custo, efeito colateral ou risco fica escondido pela sintaxe?
4. Como o comportamento muda com `null`, concorrência, entrada inválida ou volume alto?

Os exemplos assumem arquivos e classes com nomes compatíveis. Quando um bloco representa apenas parte de uma aplicação, o texto indica o contexto omitido. Para testar um exemplo isolado, use `javac`, `java`, `jshell` ou o modo de execução direta de arquivo-fonte.

---

<a id="sumário"></a>

## Sumário Geral

### Como o conteúdo está organizado

As 29 partes avançam do modelo de execução para sintaxe, modelagem, concorrência, engenharia e ecossistema. A tabela mostra a finalidade de cada bloco; dentro de cada parte, o índice detalhado permite chegar ao contrato específico sem perder a sequência pedagógica.

| Bloco | Partes | Assuntos centrais | Resultado esperado | Comece por |
|---|---:|---|---|---|
| 1. Base da linguagem e da JVM | 1–4 | compilação, tokens, terminal, carregamento, packages, módulos, tipos, nulidade e texto | compreender a sintaxe mínima e como o fonte se transforma em classes executáveis | [Parte 1](#parte-1--introdução-e-contextualização) |
| 2. Sintaxe aplicada e modelagem | 5–12 | acesso, campos, modificadores, fluxo, métodos, enums, classes, records e interfaces | construir tipos com contratos e invariantes previsíveis | [Parte 5](#parte-5--modificadores-de-acesso) |
| 3. Composição, consultas e dados | 13–17 | lambdas, interfaces funcionais, Stream API, coleções, tarefas e generics | compor comportamento e processar dados reutilizavelmente | [Parte 13](#parte-13--interfaces-funcionais-lambdas-e-method-references) |
| 4. Robustez, memória e integração avançada | 18–22 | exceções, annotations, referências, memória, concorrência, reflection e código nativo | reconhecer garantias e riscos das APIs avançadas | [Parte 18](#parte-18--tratamento-de-exceções) |
| 5. Domínios de aplicação e arquitetura | 23–24 | backend, desktop, Android, cloud, camadas, DDD, CQRS e eventos | aplicar Java em plataformas e arquiteturas reais | [Parte 23](#parte-23--java-nos-principais-contextos-de-aplicação) |
| 6. Engenharia profissional | 25–27 | JDK tools, projetos, dependências, I/O, HTTP, testes, segurança e publicação | desenvolver, diagnosticar e entregar software Java | [Parte 25](#parte-25--jdk-projetos-dependências-e-qualidade) |
| 7. Catálogo da plataforma e do ecossistema | 28–29 | keywords, Java SE API, estruturas prontas, frameworks e bibliotecas | descobrir o que já existe antes de reimplementar | [Parte 28](#parte-28--catálogo-da-linguagem-e-da-biblioteca-padrão) |
| 8. Consulta e revisão | Anexos | trilhas oficiais, fontes primárias e glossário | revisar e aprofundar conceitos pela documentação original | [Anexo A](#anexo-a--trilhas-oficiais-de-estudo-e-prática) |

### Atalhos por pergunta prática

Use estes atalhos quando a dúvida começa por uma tarefa concreta. Como muitos problemas atravessam linguagem, biblioteca e runtime, algumas respostas apontam para mais de uma parte.

| Se você quer saber... | Consulte primeiro |
|---|---|
| como Java, JDK, JVM, bytecode, class loaders, classpath, módulo e JAR se relacionam | [Partes 1](#parte-1--introdução-e-contextualização), [2](#parte-2--packages-imports-e-módulos) e [25](#parte-25--jdk-projetos-dependências-e-qualidade) |
| qual é a sintaxe mínima e como ler/escrever no terminal | [Seções 1.5](#15-sintaxe-mínima-tokens-comentários-statements-e-blocos) e [1.6](#16-entrada-e-saída-no-terminal) |
| o que todo iniciante precisa dominar e quais contratos distinguem Java | [Seção 1.7](#17-mapa-essencial-do-iniciante-e-características-distintivas-do-java) |
| como escolher tipos, converter valores e entender referências | [Parte 3](#parte-3--variáveis-e-tipos) |
| como modelar classes, records, igualdade, interfaces e extensibilidade | [Partes 5–12](#parte-5--modificadores-de-acesso) |
| como armazenar, consultar e transformar dados em memória | [Partes 14](#parte-14--stream-api-e-processamento-de-sequências), [15](#parte-15--collections-framework) e [28.6](#286-estruturas-de-dados-e-coleções-prontas) |
| como executar I/O concorrente com tarefas ou virtual threads | [Partes 16](#parte-16--tarefas-assincronismo-e-virtual-threads), [21](#parte-21--threads-e-concorrência) e [26](#parte-26--io-serialização-http-e-internacionalização) |
| como tratar erros, recursos e memória corretamente | [Partes 18–20](#parte-18--tratamento-de-exceções) |
| como construir, testar, observar, proteger e distribuir uma aplicação | [Partes 25–27](#parte-25--jdk-projetos-dependências-e-qualidade) |
| qual classe, interface, estrutura ou framework já existe | [Partes 28](#parte-28--catálogo-da-linguagem-e-da-biblioteca-padrão) e [29](#parte-29--ecossistema-externo-frameworks-bibliotecas-e-ferramentas) |
| onde encontrar uma definição curta ou a fonte oficial | [Glossário](#glossário) e [Anexo B](#anexo-b--referências-oficiais-consultadas) |

### Índice detalhado

**Navegação inicial**

- [Como usar este guia](#como-usar-este-guia)

**Bloco 1 — Base da linguagem e da JVM (Partes 1–4)**

- **[Parte 1 — Introdução e Contextualização](#parte-1--introdução-e-contextualização)**
  - [1.0 Convenções oficiais de nomenclatura](#10-convenções-oficiais-de-nomenclatura)
  - [1.1 O que é Java?](#11-o-que-é-java)
  - [1.2 JDK, JVM, bytecode, class files e JARs](#12-jdk-jvm-bytecode-class-files-e-jars)
  - [1.3 Java em 2026: LTS, versão corrente e preview](#13-java-em-2026-lts-versão-corrente-e-preview)
  - [1.4 Estrutura e ponto de entrada de um programa](#14-estrutura-e-ponto-de-entrada-de-um-programa)
  - [1.5 Sintaxe mínima: tokens, comentários, statements e blocos](#15-sintaxe-mínima-tokens-comentários-statements-e-blocos)
  - [1.6 Entrada e saída no terminal](#16-entrada-e-saída-no-terminal)
  - [1.7 Mapa essencial do iniciante e características distintivas do Java](#17-mapa-essencial-do-iniciante-e-características-distintivas-do-java)
  - [1.8 Carregamento, linking, inicialização e identidade de tipos](#18-carregamento-linking-inicialização-e-identidade-de-tipos)
- **[Parte 2 — Packages, Imports e Módulos](#parte-2--packages-imports-e-módulos)**
  - [2.1 Packages](#21-packages)
  - [2.2 Imports, static imports e module imports](#22-imports-static-imports-e-module-imports)
  - [2.3 Java Platform Module System](#23-java-platform-module-system)
- **[Parte 3 — Variáveis e Tipos](#parte-3--variáveis-e-tipos)**
  - [3.1 Variáveis, escopo e inicialização](#31-variáveis-escopo-e-inicialização)
  - [3.2 Tipos primitivos e tipos de referência](#32-tipos-primitivos-e-tipos-de-referência)
    - [3.2.1 Conversões, promoção numérica e casts](#321-conversões-promoção-numérica-e-casts)
  - [3.3 `null` e contratos de ausência](#33-null-e-contratos-de-ausência)
  - [3.4 `var` e inferência local](#34-var-e-inferência-local)
  - [3.5 `final`, constantes e variáveis effectively final](#35-final-constantes-e-variáveis-effectively-final)
  - [3.6 Referências fortes, fracas, soft e phantom](#36-referências-fortes-fracas-soft-e-phantom)
  - [3.7 `Object`, wrappers e autoboxing](#37-object-wrappers-e-autoboxing)
  - [3.8 Literais, arrays e varargs](#38-literais-arrays-e-varargs)
- **[Parte 4 — Strings, Texto e Unicode](#parte-4--strings-texto-e-unicode)**
  - [4.1 `String` é imutável](#41-string-é-imutável)
  - [4.2 `StringBuilder` e `StringBuffer`](#42-stringbuilder-e-stringbuffer)
  - [4.3 Text blocks, formatação e concatenação](#43-text-blocks-formatação-e-concatenação)
  - [4.4 Comparação, busca, Unicode e métodos essenciais](#44-comparação-busca-unicode-e-métodos-essenciais)
  - [4.5 Da linguagem à biblioteca padrão do Java](#45-da-linguagem-à-biblioteca-padrão-do-java)

**Bloco 2 — Sintaxe aplicada e modelagem (Partes 5–12)**

- **[Parte 5 — Modificadores de Acesso](#parte-5--modificadores-de-acesso)**
  - [5.1 `public`, `protected`, package-private e `private`](#51-public-protected-package-private-e-private)
  - [5.2 Encapsulamento e desenho da superfície pública](#52-encapsulamento-e-desenho-da-superfície-pública)
- **[Parte 6 — Campos, Accessors e Imutabilidade](#parte-6--campos-accessors-e-imutabilidade)**
  - [6.1 Campos e métodos de acesso](#61-campos-e-métodos-de-acesso)
  - [6.2 JavaBeans e propriedades convencionais](#62-javabeans-e-propriedades-convencionais)
  - [6.3 Imutabilidade e cópias defensivas](#63-imutabilidade-e-cópias-defensivas)
- **[Parte 7 — Palavras-chave e Modificadores Essenciais](#parte-7--palavras-chave-e-modificadores-essenciais)**
  - [7.1 `static`](#71-static)
  - [7.2 `final`](#72-final)
  - [7.3 `abstract`](#73-abstract)
  - [7.4 Sobrescrita, despacho dinâmico e `@Override`](#74-sobrescrita-despacho-dinâmico-e-override)
  - [7.5 `this` e `super`](#75-this-e-super)
  - [7.6 `instanceof` e pattern matching](#76-instanceof-e-pattern-matching)
  - [7.7 `try`-with-resources e `AutoCloseable`](#77-try-with-resources-e-autocloseable)
  - [7.8 `synchronized`, `volatile`, `transient`, `native` e `strictfp`](#78-synchronized-volatile-transient-native-e-strictfp)
  - [7.9 `sealed`, `non-sealed` e `permits`](#79-sealed-non-sealed-e-permits)
  - [7.10 `yield`, `assert` e controle explícito](#710-yield-assert-e-controle-explícito)
  - [7.11 Recursos permanentes do Java 25](#711-recursos-permanentes-do-java-25)
  - [7.12 Recursos de preview e incubator](#712-recursos-de-preview-e-incubator)
- **[Parte 8 — Controle de Fluxo e Operadores](#parte-8--controle-de-fluxo-e-operadores)**
  - [8.1 `if`, `else` e guard clauses](#81-if-else-e-guard-clauses)
  - [8.2 `switch` statements e expressions](#82-switch-statements-e-expressions)
  - [8.3 Laços e controle de iteração](#83-laços-e-controle-de-iteração)
  - [8.4 Operadores, precedência, curto-circuito e overflow](#84-operadores-precedência-curto-circuito-e-overflow)
  - [8.5 Pattern matching e exaustividade](#85-pattern-matching-e-exaustividade)
- **[Parte 9 — Métodos](#parte-9--métodos)**
  - [9.1 Assinaturas, parâmetros e retorno](#91-assinaturas-parâmetros-e-retorno)
  - [9.2 Java sempre passa argumentos por valor](#92-java-sempre-passa-argumentos-por-valor)
  - [9.3 Sobrecarga, resolução e varargs](#93-sobrecarga-resolução-e-varargs)
  - [9.4 Métodos genéricos, recursão e contratos](#94-métodos-genéricos-recursão-e-contratos)
- **[Parte 10 — Enums](#parte-10--enums)**
  - [10.1 Enum classes e comportamento](#101-enum-classes-e-comportamento)
  - [10.2 `EnumSet`, `EnumMap` e flags](#102-enumset-enummap-e-flags)
- **[Parte 11 — Classes, Objetos e Records](#parte-11--classes-objetos-e-records)**
  - [11.1 Estrutura completa de uma classe](#111-estrutura-completa-de-uma-classe)
  - [11.2 Construtores e ordem de inicialização](#112-construtores-e-ordem-de-inicialização)
  - [11.3 Classes aninhadas, locais e anônimas](#113-classes-aninhadas-locais-e-anônimas)
  - [11.4 Records e modelagem de dados](#114-records-e-modelagem-de-dados)
  - [11.5 Igualdade, hash, representação e cópia](#115-igualdade-hash-representação-e-cópia)
  - [11.6 Builder e APIs fluentes](#116-builder-e-apis-fluentes)
- **[Parte 12 — Herança, Interfaces e Composição](#parte-12--herança-interfaces-e-composição)**
  - [12.1 Herança de classes](#121-herança-de-classes)
  - [12.2 Interfaces modernas](#122-interfaces-modernas)
  - [12.3 Tipos sealed e hierarquias fechadas](#123-tipos-sealed-e-hierarquias-fechadas)
  - [12.4 Composição antes de herança acidental](#124-composição-antes-de-herança-acidental)

**Bloco 3 — Composição, consultas e dados (Partes 13–17)**

- **[Parte 13 — Interfaces Funcionais, Lambdas e Method References](#parte-13--interfaces-funcionais-lambdas-e-method-references)**
  - [13.1 Interfaces funcionais](#131-interfaces-funcionais)
  - [13.2 `Function`, `Consumer`, `Supplier` e `Predicate`](#132-function-consumer-supplier-e-predicate)
  - [13.3 Expressões lambda](#133-expressões-lambda)
  - [13.4 Method references e composição](#134-method-references-e-composição)
  - [13.5 Closures, callbacks e listeners](#135-closures-callbacks-e-listeners)
- **[Parte 14 — Stream API e Processamento de Sequências](#parte-14--stream-api-e-processamento-de-sequências)**
  - [14.1 O que é um stream?](#141-o-que-é-um-stream)
  - [14.2 Operações intermediárias e terminais](#142-operações-intermediárias-e-terminais)
  - [14.3 `Iterable`, `Iterator` e `Spliterator`](#143-iterable-iterator-e-spliterator)
  - [14.4 Collectors, redução e agrupamento](#144-collectors-redução-e-agrupamento)
  - [14.5 Avaliação lazy, streams paralelos e armadilhas](#145-avaliação-lazy-streams-paralelos-e-armadilhas)
- **[Parte 15 — Collections Framework](#parte-15--collections-framework)**
  - [15.1 Hierarquia e contratos principais](#151-hierarquia-e-contratos-principais)
  - [15.2 `ArrayList` e `LinkedList`](#152-arraylist-e-linkedlist)
  - [15.3 `HashMap`, mapas ordenados e chaves](#153-hashmap-mapas-ordenados-e-chaves)
  - [15.4 Sets, filas, deques e filas de prioridade](#154-sets-filas-deques-e-filas-de-prioridade)
  - [15.5 Coleções imutáveis, views e cópias](#155-coleções-imutáveis-views-e-cópias)
  - [15.6 Como escolher a coleção certa](#156-como-escolher-a-coleção-certa)
- **[Parte 16 — Tarefas, Assincronismo e Virtual Threads](#parte-16--tarefas-assincronismo-e-virtual-threads)**
  - [16.1 `ExecutorService`, `Callable` e `Future`](#161-executorservice-callable-e-future)
  - [16.2 `CompletableFuture`](#162-completablefuture)
  - [16.3 Virtual threads](#163-virtual-threads)
  - [16.4 Cancelamento, timeout e interrupção](#164-cancelamento-timeout-e-interrupção)
  - [16.5 Structured Concurrency como preview](#165-structured-concurrency-como-preview)
  - [16.6 Scoped Values](#166-scoped-values)
- **[Parte 17 — Generics](#parte-17--generics)**
  - [17.1 Tipos e métodos parametrizados](#171-tipos-e-métodos-parametrizados)
  - [17.2 Bounds, wildcards e PECS](#172-bounds-wildcards-e-pecs)
  - [17.3 Type erasure e restrições](#173-type-erasure-e-restrições)
  - [17.4 Invariância e captura de wildcard](#174-invariância-e-captura-de-wildcard)

**Bloco 4 — Robustez, memória e integração avançada (Partes 18–22)**

- **[Parte 18 — Tratamento de Exceções](#parte-18--tratamento-de-exceções)**
  - [18.1 Checked e unchecked exceptions](#181-checked-e-unchecked-exceptions)
  - [18.2 `try`, `catch`, `finally` e multi-catch](#182-try-catch-finally-e-multi-catch)
  - [18.3 Exceções customizadas](#183-exceções-customizadas)
  - [18.4 Hierarquia e exceções comuns](#184-hierarquia-e-exceções-comuns)
  - [18.5 Práticas de desenho e propagação](#185-práticas-de-desenho-e-propagação)
- **[Parte 19 — Annotations e Metadados](#parte-19--annotations-e-metadados)**
  - [19.1 Annotations predefinidas](#191-annotations-predefinidas)
  - [19.2 Criando annotations](#192-criando-annotations)
  - [19.3 Retention, Target, Repeatable e Inherited](#193-retention-target-repeatable-e-inherited)
  - [19.4 Annotation processing](#194-annotation-processing)
- **[Parte 20 — Referências, Memória e Recursos](#parte-20--referências-memória-e-recursos)**
  - [20.1 `Optional` e ausência explícita](#201-optional-e-ausência-explícita)
  - [20.2 `WeakReference`, `SoftReference` e `PhantomReference`](#202-weakreference-softreference-e-phantomreference)
  - [20.3 `ByteBuffer`, `MemorySegment` e `Arena`](#203-bytebuffer-memorysegment-e-arena)
  - [20.4 Garbage Collector e alcançabilidade](#204-garbage-collector-e-alcançabilidade)
  - [20.5 Ownership e fechamento determinístico](#205-ownership-e-fechamento-determinístico)
- **[Parte 21 — Threads e Concorrência](#parte-21--threads-e-concorrência)**
  - [21.1 Platform threads e virtual threads](#211-platform-threads-e-virtual-threads)
  - [21.2 Java Memory Model e happens-before](#212-java-memory-model-e-happens-before)
  - [21.3 Monitores, locks e conditions](#213-monitores-locks-e-conditions)
  - [21.4 Atomics, immutability e thread confinement](#214-atomics-immutability-e-thread-confinement)
  - [21.5 Coleções concorrentes e filas bloqueantes](#215-coleções-concorrentes-e-filas-bloqueantes)
- **[Parte 22 — Reflection, Serviços e Interoperabilidade](#parte-22--reflection-serviços-e-interoperabilidade)**
  - [22.1 Reflection](#221-reflection)
  - [22.2 Method handles](#222-method-handles)
  - [22.3 `ServiceLoader` e Dependency Injection](#223-serviceloader-e-dependency-injection)
  - [22.4 JNI](#224-jni)
  - [22.5 Foreign Function and Memory API](#225-foreign-function-and-memory-api)
- [Checkpoint — Fundamentos da Linguagem](#checkpoint--fundamentos-da-linguagem-partes-122)

**Bloco 5 — Domínios de aplicação e arquitetura (Partes 23–24)**

- **[Parte 23 — Java nos Principais Contextos de Aplicação](#parte-23--java-nos-principais-contextos-de-aplicação)**
  - [23.1 Backend e serviços](#231-backend-e-serviços)
  - [23.2 Jakarta EE](#232-jakarta-ee)
  - [23.3 Desktop com JavaFX](#233-desktop-com-javafx)
  - [23.4 Android](#234-android)
  - [23.5 Cloud, containers e serverless](#235-cloud-containers-e-serverless)
  - [23.6 Dados, mensageria e outros domínios](#236-dados-mensageria-e-outros-domínios)
- **[Parte 24 — Arquitetura de Aplicações Java](#parte-24--arquitetura-de-aplicações-java)**
  - [24.1 Java não impõe arquitetura](#241-java-não-impõe-arquitetura)
  - [24.2 Arquitetura em camadas](#242-arquitetura-em-camadas)
  - [24.3 Clean Architecture, Hexagonal e Onion](#243-clean-architecture-hexagonal-e-onion)
  - [24.4 Domain-Driven Design](#244-domain-driven-design)
  - [24.5 CQRS](#245-cqrs)
  - [24.6 Event-Driven Architecture](#246-event-driven-architecture)
  - [24.7 Microsserviços](#247-microsserviços)
  - [24.8 Padrões enterprise clássicos](#248-padrões-enterprise-clássicos)

**Bloco 6 — Engenharia profissional (Partes 25–27)**

- **[Parte 25 — JDK, Projetos, Dependências e Qualidade](#parte-25--jdk-projetos-dependências-e-qualidade)**
  - [25.1 Ferramentas do JDK](#251-ferramentas-do-jdk)
  - [25.2 Fonte, class files, classpath e module path](#252-fonte-class-files-classpath-e-module-path)
  - [25.3 JARs, Maven e Gradle](#253-jars-maven-e-gradle)
  - [25.4 Build, testes, empacotamento e execução](#254-build-testes-empacotamento-e-execução)
  - [25.5 Javadoc, lint e análise estática](#255-javadoc-lint-e-análise-estática)
  - [25.6 Compatibilidade de versões e toolchains](#256-compatibilidade-de-versões-e-toolchains)
- **[Parte 26 — I/O, Serialização, HTTP e Internacionalização](#parte-26--io-serialização-http-e-internacionalização)**
  - [26.1 Paths, Files, streams e channels](#261-paths-files-streams-e-channels)
  - [26.2 Readers, writers, charsets e buffers](#262-readers-writers-charsets-e-buffers)
  - [26.3 Serialização e JSON](#263-serialização-e-json)
  - [26.4 HTTP Client](#264-http-client)
  - [26.5 Datas, tempo, locale e fusos](#265-datas-tempo-locale-e-fusos)
  - [26.6 Expressões regulares e limites](#266-expressões-regulares-e-limites)
- **[Parte 27 — Engenharia para Produção](#parte-27--engenharia-para-produção)**
  - [27.1 Estratégia de testes](#271-estratégia-de-testes)
  - [27.2 Logging, configuração e segredos](#272-logging-configuração-e-segredos)
  - [27.3 JFR, JMX, profiling e observabilidade](#273-jfr-jmx-profiling-e-observabilidade)
  - [27.4 Segurança essencial](#274-segurança-essencial)
  - [27.5 `jlink`, `jpackage`, CDS e Native Image](#275-jlink-jpackage-cds-e-native-image)
  - [27.6 APIs públicas, compatibilidade e evolução](#276-apis-públicas-compatibilidade-e-evolução)

**Bloco 7 — Catálogo da plataforma e do ecossistema (Partes 28–29)**

- **[Parte 28 — Catálogo da Linguagem e da Biblioteca Padrão](#parte-28--catálogo-da-linguagem-e-da-biblioteca-padrão)**
  - [28.1 Linguagem, JVM, JDK, módulos e dependências não são sinônimos](#281-linguagem-jvm-jdk-módulos-e-dependências-não-são-sinônimos)
  - [28.2 Todas as palavras-chave reservadas](#282-todas-as-palavras-chave-reservadas)
  - [28.3 Todas as palavras-chave contextuais](#283-todas-as-palavras-chave-contextuais)
  - [28.4 Tipos primitivos, wrappers e literais](#284-tipos-primitivos-wrappers-e-literais)
  - [28.5 Operações prontas por domínio](#285-operações-prontas-por-domínio)
  - [28.6 Estruturas de dados e coleções prontas](#286-estruturas-de-dados-e-coleções-prontas)
  - [28.7 Interfaces funcionais e contratos prontos](#287-interfaces-funcionais-e-contratos-prontos)
  - [28.8 Packages e módulos essenciais](#288-packages-e-módulos-essenciais)
  - [28.9 Algoritmos e utilitários que você não precisa reimplementar](#289-algoritmos-e-utilitários-que-você-não-precisa-reimplementar)
  - [28.10 Como descobrir se a API já existe](#2810-como-descobrir-se-a-api-já-existe)
- **[Parte 29 — Ecossistema Externo: Frameworks, Bibliotecas e Ferramentas](#parte-29--ecossistema-externo-frameworks-bibliotecas-e-ferramentas)**
  - [29.1 O que é externo ao Java SE](#291-o-que-é-externo-ao-java-se)
  - [29.2 Plataformas e frameworks de aplicação](#292-plataformas-e-frameworks-de-aplicação)
  - [29.3 Dados, bancos e persistência](#293-dados-bancos-e-persistência)
  - [29.4 HTTP, resiliência, mensageria e jobs](#294-http-resiliência-mensageria-e-jobs)
  - [29.5 Serialização, mapeamento, validação e produtividade](#295-serialização-mapeamento-validação-e-produtividade)
  - [29.6 Logging e observabilidade](#296-logging-e-observabilidade)
  - [29.7 Testes, automação e medição](#297-testes-automação-e-medição)
  - [29.8 UI, mobile e jogos](#298-ui-mobile-e-jogos)
  - [29.9 Como avaliar e adotar uma dependência](#299-como-avaliar-e-adotar-uma-dependência)

**Anexos e consulta rápida**

- [Anexo A — Trilhas Oficiais de Estudo e Prática](#anexo-a--trilhas-oficiais-de-estudo-e-prática)
- [Anexo B — Referências Oficiais Consultadas](#anexo-b--referências-oficiais-consultadas)
- [Glossário](#glossário)

---

## Parte 1 — Introdução e Contextualização

[⬆️ Voltar ao Sumário](#sumário)

Esta parte separa os componentes que iniciantes costumam chamar apenas de “Java”. A linguagem define sintaxe e semântica; o JDK oferece ferramentas e bibliotecas; a JVM carrega classes e executa bytecode. Essa distinção acompanha o guia inteiro.

---

### 1.0 Convenções oficiais de nomenclatura

[⬆️ Voltar ao Sumário](#sumário)

Nomes não mudam o comportamento do programa, mas fazem parte de sua API humana. A especificação estabelece convenções que evitam confundir package, tipo, método e constante.

| Elemento | Convenção | Exemplo |
|---|---|---|
| package e módulo | minúsculas; normalmente domínio invertido | `com.exemplo.pedidos` |
| classe, record, enum e interface | UpperCamelCase | `PedidoService`, `Comparable` |
| método e variável | lowerCamelCase | `calcularTotal`, `pedidoAtual` |
| constante | maiúsculas com `_` | `MAX_TENTATIVAS` |
| parâmetro de tipo | letra maiúscula curta ou nome expressivo | `T`, `E`, `K`, `V`, `TResultado` |

```java
package com.exemplo.pedidos;

public final class PedidoService {
    private static final int MAX_TENTATIVAS = 3;

    public String buscarDescricao(long pedidoId) {
        return "Pedido " + pedidoId;
    }
}
```

**Leitura guiada:** `PedidoService` identifica um tipo; `MAX_TENTATIVAS` comunica uma constante; `buscarDescricao` descreve uma ação; `pedidoId` nomeia o dado recebido. O package usa minúsculas porque funciona como endereço hierárquico e precisa minimizar colisões globais.

> **Referências oficiais:** [JLS §6.1 — Declarations](https://docs.oracle.com/javase/specs/jls/se25/html/jls-6.html#jls-6.1), [JLS §7.4.2 — Unique Package Names](https://docs.oracle.com/javase/specs/jls/se25/html/jls-7.html#jls-7.4.2)

---

### 1.1 O que é Java?

[⬆️ Voltar ao Sumário](#sumário)

Java é uma linguagem de propósito geral, concorrente, baseada em classes, orientada a objetos, estática e fortemente tipada. Ela também suporta programação funcional por lambdas e interfaces funcionais, modelagem algébrica parcial com records, tipos sealed e pattern matching, além de APIs de baixo nível controladas pelo JDK.

O fluxo mais comum é:

```text
Código-fonte (.java)
        ↓ javac
bytecode + metadados (.class)
        ↓ carregamento, verificação e execução pela JVM
código interpretado e/ou compilado para a máquina pelo runtime
```

```java
public class OlaJava {
    public static void main(String[] args) {
        System.out.println("Olá, Java!");
    }
}
```

```powershell
javac OlaJava.java
java OlaJava
```

**Como interpretar o exemplo:** `javac` verifica o fonte e produz `OlaJava.class`. O comando `java` inicia uma JVM, carrega essa classe e procura um método de entrada válido. A JVM não é obrigada a interpretar tudo: uma implementação pode compilar métodos dinamicamente, otimizar caminhos quentes ou usar outras estratégias compatíveis com a especificação.

Java e JVM não são sinônimos. Outras linguagens podem produzir class files para a JVM, e a linguagem Java pode ser implementada por ferramentas diferentes desde que respeitem as especificações aplicáveis.

> **Referências oficiais:** [JLS §1 — Introduction](https://docs.oracle.com/javase/specs/jls/se25/html/jls-1.html), [JVMS §1](https://docs.oracle.com/javase/specs/jvms/se25/html/jvms-1.html), [Getting Started with Java](https://dev.java/learn/getting-started/)

---

### 1.2 JDK, JVM, bytecode, class files e JARs

[⬆️ Voltar ao Sumário](#sumário)

| Elemento | Responsabilidade |
|---|---|
| **JVM** | carrega, verifica e executa classes; administra áreas de runtime e serviços como GC |
| **JDK** | distribuição para desenvolvimento: runtime, `javac`, `java`, `jar`, `javadoc`, `jlink`, `jpackage` e outras ferramentas |
| **class file** | formato binário com bytecode, constant pool, campos, métodos e atributos |
| **JAR** | arquivo ZIP com classes, recursos e metadados opcionais em `META-INF` |
| **módulo** | unidade nomeada com dependências, packages exportados, services e encapsulamento forte |

Um arquivo `.java` não corresponde obrigatoriamente a um único `.class`. Classes aninhadas, lambdas e outras construções podem originar artefatos adicionais. Da mesma forma, um JAR pode conter muitas classes e packages.

```powershell
javac -d out src/com/exemplo/Main.java
jar --create --file app.jar --main-class com.exemplo.Main -C out .
java -jar app.jar
```

**Leitura guiada:** `-d out` define a raiz de saída dos class files. `jar --create` empacota o conteúdo de `out`; `--main-class` grava o ponto de entrada no manifest. `java -jar` usa esse metadado. Empacotar não resolve automaticamente dependências externas: elas precisam estar no classpath, module path ou dentro de uma distribuição construída para isso.

O termo histórico **JRE** descreve um ambiente de execução. Nas distribuições modernas, trate a composição concreta como decisão do fornecedor: o JDK inclui o necessário para executar, e `jlink` pode criar uma imagem de runtime contendo apenas os módulos escolhidos.

> **Referências oficiais:** [JVMS §4 — The class File Format](https://docs.oracle.com/javase/specs/jvms/se25/html/jvms-4.html), [JDK Tool Specifications](https://docs.oracle.com/en/java/javase/25/docs/specs/man/), [JAR File Specification](https://docs.oracle.com/en/java/javase/25/docs/specs/jar/jar.html)

---

### 1.3 Java em 2026: LTS, versão corrente e preview

[⬆️ Voltar ao Sumário](#sumário)

Em julho de 2026, **Java 25** é a LTS mais recente na política da Oracle e **Java 26** é a versão corrente não LTS. Versões não LTS recebem evolução mais rapidamente; a política de atualização e suporte depende da distribuição e do fornecedor escolhido.

| Conceito | Significado prático |
|---|---|
| LTS | linha para a qual um fornecedor planeja suporte prolongado |
| feature release | versão semestral da plataforma |
| update release | correções de uma linha já lançada |
| preview feature | recurso completo para avaliação, ainda não permanente |
| incubator module | API não final distribuída em módulo incubador |

```powershell
javac --release 25 Aplicacao.java
java Aplicacao

# Somente quando o projeto decide avaliar um preview da versão instalada
javac --enable-preview --release 26 Experimento.java
java --enable-preview Experimento
```

**Leitura guiada:** `--release 25` limita linguagem e APIs documentadas para essa release; ele é mais completo do que alterar apenas a geração de bytecode. Preview exige concordância explícita tanto na compilação quanto na execução. Não publique uma biblioteca estável dependendo acidentalmente de preview.

> **Referências oficiais:** [Oracle Java SE Support Roadmap](https://www.oracle.com/java/technologies/java-se-support-roadmap.html), [Java SE Specifications](https://docs.oracle.com/javase/specs/), [Preview Features](https://docs.oracle.com/en/java/javase/25/language/preview-language-and-vm-features.html)

---

### 1.4 Estrutura e ponto de entrada de um programa

[⬆️ Voltar ao Sumário](#sumário)

Na forma clássica, uma aplicação declara uma classe e um método `main`:

```java
package com.exemplo;

public class Main {
    public static void main(String[] args) {
        System.out.println("Argumentos: " + args.length);
    }
}
```

| Trecho | Papel |
|---|---|
| `package com.exemplo;` | associa o tipo a um package nomeado |
| `public class Main` | declara o tipo de topo público, normalmente em `Main.java` |
| `public static void main(String[] args)` | ponto de entrada clássico |
| `args` | argumentos recebidos sem incluir o nome da classe |

Java 25 tornou permanentes **compact source files** e instance main methods para primeiros programas e scripts simples:

```java
void main() {
    System.out.println("Olá sem classe explícita");
}
```

Esse formato reduz cerimônia, mas não muda os fundamentos: o compilador ainda sintetiza uma classe e a execução continua na JVM. Projetos maiores devem mover regras para tipos nomeados, packages e módulos claros.

> **Referências oficiais:** [JLS §12.1.4 — Invoke `main`](https://docs.oracle.com/javase/specs/jls/se25/html/jls-12.html#jls-12.1.4), [JEP 512 — Compact Source Files and Instance Main Methods](https://openjdk.org/jeps/512)

---

### 1.5 Sintaxe mínima: tokens, comentários, statements e blocos

[⬆️ Voltar ao Sumário](#sumário)

Antes de estudar orientação a objetos, é preciso saber como o compilador enxerga um fonte. O texto Unicode é separado em espaços, comentários e **tokens**; os tokens são as unidades com significado sintático.

| Categoria de token | O que representa | Exemplos |
|---|---|---|
| identificador | nome escolhido pelo programador | `nome`, `PedidoService` |
| palavra-chave | palavra com papel definido pela linguagem | `class`, `if`, `return` |
| literal | valor escrito diretamente no fonte | `42`, `true`, `"Java"` |
| separador | pontuação que organiza a gramática | `(`, `)`, `{`, `}`, `;`, `,` |
| operador | símbolo que calcula, compara ou atribui | `+`, `==`, `&&`, `=` |

Java diferencia maiúsculas de minúsculas: `total`, `Total` e `TOTAL` são identificadores distintos. Espaços e quebras de linha normalmente separam tokens, enquanto chaves delimitam blocos e, com isso, muitos escopos.

```java
/** Representa uma saudação criada a partir de um nome. */
public final class Saudacao {
    // Este comentário termina no fim da linha.
    public static String criar(String nome) {
        /* Comentários tradicionais podem ocupar várias linhas,
           mas não podem ser aninhados. */
        String nomeLimpo = nome.strip();

        if (nomeLimpo.isEmpty()) {
            return "Olá!";
        }

        return "Olá, " + nomeLimpo + "!";
    }
}
```

**Leitura guiada:** a declaração de `nomeLimpo` e cada `return` terminam com `;`. O `if` não recebe ponto e vírgula depois de seu bloco; `{` e `}` agrupam os statements condicionais. `//` comenta até o fim da linha, `/* ... */` cria comentário tradicional e `/** ... */` é a forma tradicional de comentário de documentação processado por `javadoc`.

Desde o JDK 23, `javadoc` também reconhece comentários de documentação orientados a linha, iniciados por `///`, cujo conteúdo pode usar Markdown. Eles não substituem comentários comuns: são documentação de API, assim como `/** ... */`.

```java
public final class Calculos {
    /// Retorna o dobro de `valor`.
    ///
    /// @param valor número que será multiplicado
    /// @return o resultado de `valor * 2`
    public static int dobrar(int valor) {
        return valor * 2;
    }
}
```

> **Armadilha:** um ponto e vírgula isolado é um statement vazio. Em `if (ativo); executar();`, o `if` controla apenas esse statement vazio e `executar()` roda sempre. Use blocos explícitos quando a leitura puder gerar dúvida.

> **Referências oficiais:** [JLS §3 — Lexical Structure](https://docs.oracle.com/javase/specs/jls/se25/html/jls-3.html), [JLS §14 — Blocks, Statements, and Patterns](https://docs.oracle.com/javase/specs/jls/se25/html/jls-14.html), [Documentation Comment Specification](https://docs.oracle.com/en/java/javase/25/docs/specs/javadoc/doc-comment-spec.html)

---

### 1.6 Entrada e saída no terminal

[⬆️ Voltar ao Sumário](#sumário)

Um processo normalmente recebe três fluxos padrão. Eles são APIs do JDK, não palavras-chave da linguagem.

| API | Fluxo | Uso típico |
|---|---|---|
| `System.in` | entrada padrão de bytes | dados digitados ou redirecionados para o processo |
| `System.out` | saída padrão | resultado normal do programa |
| `System.err` | saída de erro | diagnóstico que deve permanecer separado do resultado |
| `java.lang.IO` | fachada orientada a linhas sobre `System.in`/`System.out`, desde Java 25 | programas introdutórios e utilitários pequenos |
| `Console` | console interativo, quando o ambiente oferece um | prompts e leitura de senha sem eco |

```java
public final class CadastroBasico {
    public static void main(String[] args) {
        String nome = IO.readln("Nome: ");

        if (nome == null) {
            System.err.println("A entrada terminou antes de um nome.");
            return;
        }

        IO.println("Olá, " + nome.strip() + "!");
    }
}
```

**Leitura guiada:** `IO` pertence a `java.lang`, package importado implicitamente. `readln` escreve o prompt e devolve uma linha sem o separador; devolve `null` quando encontra o fim do fluxo sem ler caracteres. O teste ocorre antes de `strip()` para evitar `NullPointerException`. A mensagem normal vai para a saída padrão; a situação anormal vai para `System.err`.

`Scanner` é conveniente quando a entrada precisa ser dividida em tokens como inteiros e palavras; `BufferedReader` oferece leitura de linhas com controle mais explícito. `System.console()` pode devolver `null` em IDEs, testes, pipes e outros ambientes sem console associado, portanto nunca pressuponha sua presença.

> **Armadilha:** escolha uma única abstração para consumir a entrada. Depois da primeira chamada a `IO.readln`, misturar leituras diretas de `System.in`, `Scanner` ou outro wrapper tem comportamento não especificado pela API de `IO`. Fechar um wrapper também fecha o fluxo subjacente; uma biblioteca não deve fechar `System.in` se não é dona do ciclo de vida do processo.

> **Referências oficiais:** [`java.lang.System`](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/lang/System.html), [`java.lang.IO`](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/lang/IO.html), [`java.io.Console`](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/io/Console.html), [`java.util.Scanner`](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/util/Scanner.html)

---

### 1.7 Mapa essencial do iniciante e características distintivas do Java

[⬆️ Voltar ao Sumário](#sumário)

Nenhum recurso precisa ser absolutamente exclusivo de uma única linguagem para ser importante. O que distingue Java é a **combinação** entre regras da linguagem, contratos da JVM e uma biblioteca padrão extensa. Chamar tudo isso de “exclusivo” seria impreciso; a tabela abaixo destaca o comportamento específico que um desenvolvedor Java precisa saber, sobretudo quando vem de C#, C++, Kotlin, JavaScript ou Python.

#### O mínimo que um iniciante deve dominar

| Competência | O que você deve conseguir explicar ou fazer | Onde estudar |
|---|---|---|
| executar código | distinguir fonte, `javac`, class file, JVM, JDK e ponto de entrada | Partes 1 e 25 |
| ler a sintaxe | reconhecer tokens, blocos, statements, comentários e erros básicos do compilador | Seção 1.5 e Partes 7–9 |
| representar dados | escolher primitivos, referências, arrays, `String`, `null` e conversões | Partes 3 e 4 |
| controlar o programa | usar condições, laços, operadores, métodos, parâmetros e retorno | Partes 8 e 9 |
| modelar objetos | criar classes, construtores, records, enums e interfaces com encapsulamento | Partes 5–12 |
| armazenar dados | escolher collections, preservar igualdade/hash e usar generics sem casts frágeis | Partes 15 e 17 |
| compor comportamento | reconhecer interfaces funcionais, lambdas e pipelines de stream | Partes 13 e 14 |
| lidar com falhas e recursos | tratar exceções, validar entrada e fechar recursos com ownership claro | Partes 18, 20 e 26 |
| trabalhar profissionalmente | organizar packages, compilar, testar, documentar, depurar e empacotar | Partes 2, 25 e 27 |

#### Contratos especialmente característicos de Java/JVM

| Característica | Semântica que não deve ser presumida a partir de outra linguagem | Aprofundamento |
|---|---|---|
| fonte, class file e JVM | o compilador gera formato binário verificável; carregamento, linking e inicialização são fases distintas | 1.1, 1.2 e 1.8 |
| identidade de tipo por class loader | o mesmo nome binário definido por loaders diferentes representa tipos distintos em runtime | 1.8 |
| primitivos e referências | os oito primitivos não são objetos; wrappers, boxing, unboxing e caches têm contratos próprios | 3.2 e 3.7 |
| passagem por valor | Java sempre copia o valor do argumento; para um objeto, esse valor é uma referência | 9.2 |
| `==` versus `equals` | em referências, `==` testa identidade; igualdade de valor depende de `equals` e deve combinar com `hashCode` | 4.4 e 11.5 |
| arrays versus generics | arrays são reificados e covariantes; generics normalmente sofrem erasure e são invariantes | 3.8 e 17.3–17.4 |
| checked exceptions | parte das exceções precisa ser capturada ou declarada pelo código chamador | 18.1 |
| enum classes | cada constante é uma instância tipada e pode ter estado, métodos e corpo específico | Parte 10 |
| records, sealed types e patterns | esses recursos combinam portadores transparentes de dados, hierarquias fechadas e análise exaustiva | 8.5, 11.4 e 12.3 |
| interfaces modernas | há herança múltipla de tipos, mas uma só superclasse; interfaces podem ter métodos `default`, `static` e `private` | 12.1–12.2 |
| fechamento com supressão | `try`-with-resources fecha em ordem inversa e preserva falhas secundárias como exceções suprimidas | 7.7 e Parte 18 |
| annotations e processors | metadados podem orientar compilação, gerar código e ser retidos até o runtime | Parte 19 |
| monitores e Java Memory Model | todo objeto pode participar de `synchronized`; visibilidade depende de relações happens-before | Parte 21 |
| JPMS | módulos nomeados expressam dependências, exports, opens, serviços e encapsulamento forte | 2.3 |
| virtual threads e scoped values | a JVM oferece concorrência bloqueante leve e contexto imutável delimitado dinamicamente | 16.3, 16.6 e 21.1 |

Java também faz escolhas deliberadas por ausência. Não há herança múltipla de classes, parâmetros com valor padrão, propriedades como construção da linguagem, sobrecarga de operadores definida pelo usuário nem tipos de referência não nulos verificados pelo compilador padrão. Convenções, builders, métodos nomeados e ferramentas de análise cobrem parte desses casos, mas não alteram a semântica da linguagem.

> **Referências oficiais:** [JLS §4 — Types, Values, and Variables](https://docs.oracle.com/javase/specs/jls/se25/html/jls-4.html), [JLS §8 — Classes](https://docs.oracle.com/javase/specs/jls/se25/html/jls-8.html), [JLS §9 — Interfaces](https://docs.oracle.com/javase/specs/jls/se25/html/jls-9.html), [JLS §11 — Exceptions](https://docs.oracle.com/javase/specs/jls/se25/html/jls-11.html), [JVMS §5 — Loading, Linking, and Initializing](https://docs.oracle.com/javase/specs/jvms/se25/html/jvms-5.html)

---

### 1.8 Carregamento, linking, inicialização e identidade de tipos

[⬆️ Voltar ao Sumário](#sumário)

A JVM não precisa carregar todas as classes ao iniciar. Tipos são localizados e ligados conforme a execução exige, e a inicialização tem gatilhos próprios. Essa separação explica muitos erros que parecem “a classe existe, mas a aplicação não a encontra”.

```text
nome binário + class loader solicitante
                 ↓
carregamento → linking (verificação → preparação → resolução) → inicialização
```

| Fase | Responsabilidade principal | Observação importante |
|---|---|---|
| carregamento | localizar uma representação binária e criar o `Class` correspondente | o loader solicitante pode delegar a outro loader |
| verificação | conferir estrutura, bytecode e segurança de tipos do class file | falhas são `VerifyError` e outros `LinkageError` apropriados |
| preparação | criar campos `static` e atribuir valores iniciais padrão | ainda não executa em geral os inicializadores escritos pelo programa; constantes com atributo `ConstantValue` são um caso especial |
| resolução | transformar referências simbólicas em referências concretas | a JVM pode resolvê-las de forma lazy, dentro das regras da JVMS |
| inicialização | executar inicializadores de campos `static` e blocos `static` na ordem definida | a JVM executa o método de inicialização de classe `<clinit>`, quando ele existe |

```java
public final class InspecaoDeTipo {
    public static void main(String[] args) throws ClassNotFoundException {
        ClassLoader solicitante = ClassLoader.getSystemClassLoader();
        Class<?> tipo = Class.forName(
                "java.util.ArrayList", false, solicitante);

        System.out.println(tipo.getName());
        System.out.println(tipo.getClassLoader());
    }
}
```

**Leitura guiada:** `Class.forName` recebe o nome binário, `false` para não solicitar inicialização e o loader que começa a procura. O loader de sistema normalmente delega classes da plataforma ao loader bootstrap; por isso `getClassLoader()` pode imprimir `null`, representação usada por essa API para o bootstrap loader. Obter o objeto `Class` não cria uma instância de `ArrayList`.

Em runtime, identidade de tipo não é apenas o nome: ela inclui o **class loader definidor**. Duas definições chamadas `com.exemplo.Plugin`, carregadas por loaders definidores diferentes, são tipos diferentes; um cast entre elas falha mesmo que tenham vindo de bytes idênticos. Essa regra viabiliza isolamento de plugins e servidores de aplicação, mas também explica `ClassCastException`, conflitos de dependência e vazamentos de loaders.

> **Referências oficiais:** [JVMS §5 — Loading, Linking, and Initializing](https://docs.oracle.com/javase/specs/jvms/se25/html/jvms-5.html), [JLS §12 — Execution](https://docs.oracle.com/javase/specs/jls/se25/html/jls-12.html), [`ClassLoader`](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/lang/ClassLoader.html), [`Class.forName`](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/lang/Class.html#forName(java.lang.String,boolean,java.lang.ClassLoader))

---

## Parte 2 — Packages, Imports e Módulos

[⬆️ Voltar ao Sumário](#sumário)

Packages organizam nomes; imports encurtam referências; módulos declaram dependências e fronteiras. Nenhum desses mecanismos baixa bibliotecas: resolução de dependências pertence ao build e aos repositórios de artefatos.

---

### 2.1 Packages

[⬆️ Voltar ao Sumário](#sumário)

Um package agrupa tipos e evita colisões de nomes. O nome totalmente qualificado de `Pedido` abaixo é `com.exemplo.vendas.Pedido`.

```java
package com.exemplo.vendas;

public record Pedido(long id, java.math.BigDecimal total) { }
```

O caminho de diretórios costuma refletir o package porque compiladores e ferramentas seguem essa convenção:

```text
src/
└── com/
    └── exemplo/
        └── vendas/
            └── Pedido.java
```

Package não é pasta, JAR nem módulo. Um package pode ser repartido entre JARs no classpath, embora **split packages** sejam proibidos entre módulos nomeados que participam da mesma configuração.

> **Referências oficiais:** [JLS §7 — Packages and Modules](https://docs.oracle.com/javase/specs/jls/se25/html/jls-7.html), [Packages](https://dev.java/learn/packages/)

---

### 2.2 Imports, static imports e module imports

[⬆️ Voltar ao Sumário](#sumário)

```java
import java.time.Instant;              // importa um tipo
import java.util.*;                    // tipos públicos do package, não subpackages
import static java.util.Comparator.comparing;
import module java.sql;                // Java 25

class Exemplo {
    Instant agora = Instant.now();
}
```

| Forma | O que disponibiliza por nome simples |
|---|---|
| `import p.Tipo;` | um tipo específico |
| `import p.*;` | tipos públicos diretamente em `p` |
| `import static p.Tipo.membro;` | membro estático específico |
| `import static p.Tipo.*;` | membros estáticos acessíveis do tipo |
| `import module m;` | tipos públicos dos packages exportados pelo módulo e dependências transitivas aplicáveis |

**Armadilha comum:** wildcard não melhora desempenho nem “carrega o package inteiro”. Import é resolução de nomes em compilação. Imports amplos ou de módulo podem introduzir ambiguidades; um import de tipo específico pode resolvê-las.

`java.lang` é importado implicitamente em unidades ordinárias. Compact source files importam implicitamente os packages exportados por `java.base`, como se usassem `import module java.base`.

> **Referências oficiais:** [JLS §7.5 — Import Declarations](https://docs.oracle.com/javase/specs/jls/se25/html/jls-7.html#jls-7.5), [Module Import Declarations](https://docs.oracle.com/en/java/javase/25/language/module-import-declarations.html)

---

### 2.3 Java Platform Module System

[⬆️ Voltar ao Sumário](#sumário)

Um módulo nomeado possui um descritor `module-info.java`:

```java
module com.exemplo.pedidos {
    requires java.net.http;
    requires transitive java.logging;

    exports com.exemplo.pedidos.api;
    opens com.exemplo.pedidos.dto to com.fasterxml.jackson.databind;

    uses com.exemplo.pedidos.spi.Notificador;
    provides com.exemplo.pedidos.spi.Notificador
        with com.exemplo.pedidos.email.NotificadorEmail;
}
```

| Diretiva | Contrato |
|---|---|
| `requires` | o módulo lê outro módulo |
| `requires transitive` | consumidores que leem este módulo também leem a dependência |
| `exports` | torna tipos públicos do package acessíveis a outros módulos |
| `opens` | permite reflexão profunda no package, opcionalmente para módulos específicos |
| `uses` | declara consumo de um service |
| `provides ... with` | registra implementação de um service |

**Leitura guiada:** `public` sozinho não atravessa uma fronteira modular se o package não for exportado. `opens` não equivale a `exports`: ele atende acesso reflexivo, não uso normal por compilação. O classpath mantém o **unnamed module**, útil para compatibilidade; o module path ativa resolução e encapsulamento do JPMS.

> **Referências oficiais:** [JLS §7.7 — Module Declarations](https://docs.oracle.com/javase/specs/jls/se25/html/jls-7.html#jls-7.7), [Java Platform Module System](https://dev.java/learn/modules/)

---

## Parte 3 — Variáveis e Tipos

[⬆️ Voltar ao Sumário](#sumário)

O sistema de tipos define os valores válidos e as operações disponíveis. Para prever o comportamento de Java, domine quatro ideias: primitivo versus referência, passagem por valor, promoção/conversão e alcançabilidade de objetos.

---

### 3.1 Variáveis, escopo e inicialização

[⬆️ Voltar ao Sumário](#sumário)

```java
class Contador {
    private int total;              // campo recebe 0 por padrão
    private static long instancias; // campo estático recebe 0L

    void incrementar(int quantidade) { // parâmetro precisa ser fornecido
        int anterior;                  // local não recebe valor automático
        anterior = total;              // definite assignment satisfeita
        total += quantidade;
        System.out.println(anterior + " -> " + total);
    }
}
```

Campos recebem valores padrão; variáveis locais precisam estar **definitivamente atribuídas** antes da leitura. Escopo diz onde o nome é visível; tempo de vida diz por quanto tempo o armazenamento ou objeto continua relevante. As duas ideias não são iguais.

> **Referências oficiais:** [JLS §4.12 — Variables](https://docs.oracle.com/javase/specs/jls/se25/html/jls-4.html#jls-4.12), [JLS §16 — Definite Assignment](https://docs.oracle.com/javase/specs/jls/se25/html/jls-16.html)

---

### 3.2 Tipos primitivos e tipos de referência

[⬆️ Voltar ao Sumário](#sumário)

Java possui oito tipos primitivos. Todo array, class, interface, enum e record produz valores de referência.

| Primitivo | Tamanho definido | Exemplo | Observação |
|---|---:|---|---|
| `byte` | 8 bits | `byte b = 100;` | inteiro com sinal |
| `short` | 16 bits | `short s = 20_000;` | inteiro com sinal |
| `int` | 32 bits | `int n = 42;` | inteiro mais comum |
| `long` | 64 bits | `long id = 42L;` | sufixo `L` em literais longos |
| `float` | 32 bits | `float f = 1.5F;` | IEEE 754 binário |
| `double` | 64 bits | `double d = 1.5;` | ponto flutuante padrão |
| `char` | 16 bits | `char c = 'A';` | uma unidade UTF-16, não todo code point |
| `boolean` | tamanho não definido pela JLS | `boolean ok = true;` | apenas `true` ou `false` |

```java
int a = 10;
int b = a;
b = 99;
System.out.println(a); // 10: o valor foi copiado

int[] x = { 1, 2, 3 };
int[] y = x;
y[0] = 99;
System.out.println(x[0]); // 99: a referência foi copiada
```

**Leitura guiada:** `b = a` copia o próprio `int`. `y = x` copia o valor da referência; os dois nomes passam a alcançar o mesmo array. Java sempre passa e atribui valores. A diferença é o que o valor representa.

Não deduza automaticamente “primitivo fica na stack” e “objeto fica no heap”. A JLS define semântica; representação e otimizações pertencem à JVM.

> **Referências oficiais:** [JLS §4 — Types, Values, and Variables](https://docs.oracle.com/javase/specs/jls/se25/html/jls-4.html), [JVMS §2.3–2.7](https://docs.oracle.com/javase/specs/jvms/se25/html/jvms-2.html)

#### 3.2.1 Conversões, promoção numérica e casts

```java
int quantidade = 30;
long ampliado = quantidade;       // widening primitive conversion

double preco = 10.75;
int truncado = (int) preco;       // narrowing: 10

Object valor = "Java";
if (valor instanceof String texto) {
    System.out.println(texto.length());
}
```

Conversões widening normalmente dispensam cast, mas `long` para `float` ou `double` ainda pode perder precisão. Narrowing exige cast e pode truncar, envolver bits ou produzir infinito conforme os tipos. Cast de referência verifica compatibilidade em runtime e pode lançar `ClassCastException`; pattern matching combina teste e variável convertida.

Promoção numérica também aparece em expressões: `byte + byte` resulta em `int`. Use `Math.addExact`, `subtractExact` e `multiplyExact` quando overflow inteiro precisa virar erro, pois o overflow comum é modular e silencioso.

> **Referências oficiais:** [JLS §5 — Conversions and Contexts](https://docs.oracle.com/javase/specs/jls/se25/html/jls-5.html), [JLS §5.6 — Numeric Promotion](https://docs.oracle.com/javase/specs/jls/se25/html/jls-5.html#jls-5.6)

---

### 3.3 `null` e contratos de ausência

[⬆️ Voltar ao Sumário](#sumário)

`null` é o literal nulo e pode ser atribuído a qualquer tipo de referência. Primitivos não aceitam `null`.

```java
String nome = null;

if (nome != null) {
    System.out.println(nome.length());
}

String exibicao = java.util.Objects.requireNonNullElse(nome, "Anônimo");
```

A linguagem não possui nulabilidade estática embutida equivalente a um tipo `String?`. Annotations como `@Nullable` e `@NonNull` pertencem a ferramentas ou bibliotecas e seus significados dependem do ecossistema. Para parâmetros obrigatórios, `Objects.requireNonNull` torna a falha e a mensagem explícitas. Para ausência em retorno, `Optional<T>` pode comunicar o contrato, mas não é substituto universal para todo campo e parâmetro.

> **Referências oficiais:** [JLS §3.10.8 — The Null Literal](https://docs.oracle.com/javase/specs/jls/se25/html/jls-3.html#jls-3.10.8), [Objects](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/util/Objects.html), [Optional](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/util/Optional.html)

---

### 3.4 `var` e inferência local

[⬆️ Voltar ao Sumário](#sumário)

```java
var nomes = new java.util.ArrayList<String>(); // ArrayList<String>
var total = 10L;                               // long

for (var nome : java.util.List.of("Ana", "Bia")) {
    System.out.println(nome.toUpperCase());
}
```

`var` é um nome de tipo reservado contextualmente para variáveis locais com inicializador, índices de `for`, recursos e parâmetros de lambda anotados. O compilador infere um tipo estático; não existe tipagem dinâmica. `var valor = null;` é inválido porque não há tipo inferível.

Use `var` quando o inicializador deixa o tipo e o papel evidentes. Um tipo explícito pode ser mais didático quando comunica abstração, unidade ou precisão relevante.

> **Referências oficiais:** [JLS §14.4 — Local Variable Declarations](https://docs.oracle.com/javase/specs/jls/se25/html/jls-14.html#jls-14.4), [Local Variable Type Inference](https://dev.java/learn/language-basics/using-var/)

---

### 3.5 `final`, constantes e variáveis effectively final

[⬆️ Voltar ao Sumário](#sumário)

```java
final int limite = 10;
final java.util.List<String> nomes = new java.util.ArrayList<>();
nomes.add("Ana");          // permitido: o objeto continua mutável
// nomes = new ArrayList<>(); // erro: a variável final não pode apontar para outra lista

class Configuracao {
    static final int MAX_TENTATIVAS = 3;
    final String ambiente;

    Configuracao(String ambiente) {
        this.ambiente = java.util.Objects.requireNonNull(ambiente);
    }
}
```

`final` impede nova atribuição à variável, campo ou parâmetro; não congela o objeto referenciado. Uma constante em compilação é um `final` de tipo primitivo ou `String`, inicializado por constant expression. Campos `static final` que são constantes públicas podem ser incorporados no bytecode consumidor, o que exige cuidado ao alterar bibliotecas.

Variável **effectively final** não é declarada `final`, mas nunca é reatribuída. Lambdas e classes locais podem capturá-la.

> **Referências oficiais:** [JLS §4.12.4 — `final` Variables](https://docs.oracle.com/javase/specs/jls/se25/html/jls-4.html#jls-4.12.4), [JLS §15.29 — Constant Expressions](https://docs.oracle.com/javase/specs/jls/se25/html/jls-15.html#jls-15.29)

---

### 3.6 Referências fortes, fracas, soft e phantom

[⬆️ Voltar ao Sumário](#sumário)

| Categoria | Impede a coleta? | Uso típico |
|---|---|---|
| forte | sim, enquanto alcançável | uso normal de objetos |
| `SoftReference<T>` | depende da pressão de memória | caches muito específicos; política é pouco previsível |
| `WeakReference<T>` | não | mapas fracos, observação sem ownership |
| `PhantomReference<T>` | não; alvo não é recuperado | notificação pós-morte e coordenação com `ReferenceQueue` |

```java
Object forte = new Object();
var fraca = new java.lang.ref.WeakReference<>(forte);

System.out.println(fraca.get() != null); // true enquanto "forte" alcança o objeto
forte = null;

Object talvezVivo = fraca.get();
if (talvezVivo != null) {
    System.out.println(talvezVivo);
}
```

**Leitura guiada:** `get()` pode devolver o alvo ou `null`. Guardar o retorno em `talvezVivo` cria uma referência forte durante seu uso. Atribuir `null` a `forte` não força GC, e `System.gc()` também não é garantia. Não faça regra de negócio depender do instante de coleta.

> **Referências oficiais:** [`java.lang.ref`](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/lang/ref/package-summary.html), [Reachability](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/lang/ref/package-summary.html#reachability)

---

### 3.7 `Object`, wrappers e autoboxing

[⬆️ Voltar ao Sumário](#sumário)

`Object` é a superclasse de todas as classes. Primitivos não herdam de `Object`; wrappers como `Integer`, `Long`, `Double` e `Boolean` representam valores primitivos como objetos.

```java
Integer caixa = 42; // autoboxing: Integer.valueOf(42)
int numero = caixa; // unboxing: caixa.intValue()

Integer a = 100;
Integer b = 100;
System.out.println(a == b);      // não use este resultado como regra
System.out.println(a.equals(b)); // true: compara o valor
```

**Armadilha comum:** unboxing de `null` lança `NullPointerException`. `==` em referências testa identidade, mesmo que caches de wrappers façam alguns valores parecerem iguais. Para conteúdo, use `equals` ou `Objects.equals`.

Autoboxing é conveniente, mas pode alocar, esconder `null` e introduzir conversões. APIs numéricas quentes costumam preferir primitivas e streams especializados (`IntStream`, `LongStream`, `DoubleStream`).

> **Referências oficiais:** [JLS §5.1.7–5.1.8 — Boxing and Unboxing](https://docs.oracle.com/javase/specs/jls/se25/html/jls-5.html#jls-5.1.7), [Object](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/lang/Object.html)

---

### 3.8 Literais, arrays e varargs

[⬆️ Voltar ao Sumário](#sumário)

```java
int binario = 0b1010;
int hexadecimal = 0xFF;
long populacao = 8_000_000_000L;
double cientifico = 1.2e3;
char unidadeUtf16 = '\u0041';

int[] fixos = { 10, 20, 30 };
String[][] matriz = new String[2][3];

static int somar(int... valores) {
    return java.util.Arrays.stream(valores).sum();
}
```

Array é objeto de tamanho fixo, covariante e reificado: `String[]` é também `Object[]`, mas guardar um tipo incompatível causa `ArrayStoreException`. Generics seguem regras diferentes e são invariantes.

Varargs é açúcar para um array no último parâmetro. Cada chamada pode criar um array; combine generics e varargs com cuidado por causa de heap pollution. `@SafeVarargs` é uma promessa do autor, não uma correção automática.

> **Referências oficiais:** [JLS §3.10 — Literals](https://docs.oracle.com/javase/specs/jls/se25/html/jls-3.html#jls-3.10), [JLS §10 — Arrays](https://docs.oracle.com/javase/specs/jls/se25/html/jls-10.html), [JLS §8.4.1 — Formal Parameters](https://docs.oracle.com/javase/specs/jls/se25/html/jls-8.html#jls-8.4.1)

---

## Parte 4 — Strings, Texto e Unicode

[⬆️ Voltar ao Sumário](#sumário)

Texto envolve imutabilidade, UTF-16, comparação, locale e alocação. `String` resolve muitos casos, mas o método correto depende de o texto representar conteúdo humano, identificador, protocolo ou sequência de code points.

---

### 4.1 `String` é imutável

[⬆️ Voltar ao Sumário](#sumário)

```java
String original = "Java";
String maiuscula = original.toUpperCase(java.util.Locale.ROOT);

System.out.println(original);  // Java
System.out.println(maiuscula); // JAVA

String a = "texto";
String b = new String("texto");
System.out.println(a == b);      // false: identidade
System.out.println(a.equals(b)); // true: conteúdo

String canonica = b.intern();
System.out.println(a == canonica); // true: referência canônica do pool
```

Uma operação não altera a instância; produz outra string ou pode devolver a mesma quando nada muda. Literais e constant expressions do tipo `String` são internados. `intern()` procura uma string igual no pool canônico e devolve essa referência; se não houver, registra a própria instância.

**Leitura guiada:** `new String("texto")` força uma instância diferente, por isso `a == b` é falso. `b.intern()` devolve a referência canônica já associada ao literal, mas isso não transforma `==` em comparação de conteúdo. Use `equals`; interning manual só faz sentido quando identidade canônica e retenção de memória foram deliberadamente avaliadas.

Imutabilidade torna strings seguras como chaves quando o contrato de comparação é adequado, mas não elimina custos de concatenação, normalização ou conversão de encoding.

> **Referências oficiais:** [String](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/lang/String.html), [String.intern](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/lang/String.html#intern()), [JLS §3.10.5 — String Literals](https://docs.oracle.com/javase/specs/jls/se25/html/jls-3.html#jls-3.10.5)

---

### 4.2 `StringBuilder` e `StringBuffer`

[⬆️ Voltar ao Sumário](#sumário)

```java
var builder = new StringBuilder();
for (int i = 1; i <= 3; i++) {
    if (!builder.isEmpty()) {
        builder.append(", ");
    }
    builder.append(i);
}

String resultado = builder.toString();
System.out.println(resultado); // 1, 2, 3
```

**Leitura guiada:** `StringBuilder` mantém um buffer mutável; `append` devolve o próprio builder e permite encadeamento. `toString()` materializa o texto final. Para apenas unir uma coleção, `String.join` ou `Collectors.joining` costuma comunicar melhor.

`StringBuffer` oferece operações sincronizadas e existe principalmente por compatibilidade e cenários específicos. Sincronizar cada chamada não torna uma sequência de várias chamadas atomicamente correta; em código local, prefira `StringBuilder`.

> **Referências oficiais:** [StringBuilder](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/lang/StringBuilder.html), [StringBuffer](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/lang/StringBuffer.html)

---

### 4.3 Text blocks, formatação e concatenação

[⬆️ Voltar ao Sumário](#sumário)

```java
String nome = "Ana";
int itens = 3;

String mensagem = "Olá, %s. Você tem %d itens.".formatted(nome, itens);

String json = """
        {
          "nome": "%s",
          "itens": %d
        }
        """.formatted(nome, itens);
```

Text block continua produzindo `String`; sua indentação incidental é removida segundo regras formais. Ele não é interpolado. `formatted`, `String.format` e `Formatter` usam especificadores e locale; para mensagens traduzíveis, use `ResourceBundle` e `MessageFormat`.

Concatenação com `+` em uma única expressão é idiomática e o compilador/runtime pode otimizá-la. Em laço ou construção incremental, use builder. Nunca monte SQL ou comandos com concatenação de entrada externa: use APIs parametrizadas.

> **Referências oficiais:** [JLS §3.10.6 — Text Blocks](https://docs.oracle.com/javase/specs/jls/se25/html/jls-3.html#jls-3.10.6), [Formatter](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/util/Formatter.html)

---

### 4.4 Comparação, busca, Unicode e métodos essenciais

[⬆️ Voltar ao Sumário](#sumário)

| Objetivo | API comum |
|---|---|
| igualdade exata | `equals`, `Objects.equals` |
| ignorar caixa de forma simples | `equalsIgnoreCase` |
| ordenar texto humano | `Collator` |
| testar prefixo/sufixo | `startsWith`, `endsWith` |
| buscar | `indexOf`, `contains` |
| separar/unir | `split`, `String.join` |
| remover whitespace Unicode nas extremidades | `strip`, `stripLeading`, `stripTrailing` |
| percorrer unidades UTF-16 | `chars()` |
| percorrer code points | `codePoints()` |

```java
String texto = "A😀B";

System.out.println(texto.length());                  // 4 unidades UTF-16
System.out.println(texto.codePointCount(0, texto.length())); // 3 code points

texto.codePoints()
     .mapToObj(Character::toString)
     .forEach(System.out::println);
```

`char` e `String.length()` trabalham com unidades UTF-16; um símbolo suplementar usa um par substituto. Mesmo code points não equivalem sempre a caracteres percebidos pelo usuário, que podem combinar múltiplos pontos de código. Para fronteiras linguísticas, use `BreakIterator` e APIs de internacionalização.

**Armadilha comum:** `toLowerCase()` sem locale pode variar. Para chaves e protocolos definidos independentemente de idioma, use `Locale.ROOT`; para exibição humana, use o locale do usuário.

> **Referências oficiais:** [JLS §3.1 — Unicode](https://docs.oracle.com/javase/specs/jls/se25/html/jls-3.html#jls-3.1), [Character](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/lang/Character.html), [Collator](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/text/Collator.html)

---

### 4.5 Da linguagem à biblioteca padrão do Java

[⬆️ Voltar ao Sumário](#sumário)

Java fornece construções como `if`, `class`, `record` e `synchronized`. A API Java SE fornece `String`, `Math`, `Files`, `HttpClient`, `List`, `Instant` e milhares de membros reutilizáveis.

| Forma no código | Camada | Onde aprofundar |
|---|---|---|
| `if`, `new`, `switch` | linguagem | JLS e Partes 7–9 |
| `int`, `boolean` | tipos primitivos da linguagem | Parte 3 |
| `String`, `Object` | classes de `java.base` com suporte especial da linguagem | Partes 3–4 |
| `List`, `Map`, `Stream` | Java SE API | Partes 14–15 e 28 |
| Spring, Jackson, JUnit | dependências externas | Parte 29 |

Imports não instalam APIs. `java.base` está presente em toda implementação Java SE; outros módulos padrão podem precisar ser resolvidos. Bibliotecas externas entram pelo classpath/module path e pelo sistema de build.

> **Referências oficiais:** [JLS §1.4 — Predefined Classes and Interfaces](https://docs.oracle.com/javase/specs/jls/se25/html/jls-1.html#jls-1.4), [Java SE 25 API](https://docs.oracle.com/en/java/javase/25/docs/api/), [`java.base`](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/module-summary.html)

---

## Parte 5 — Modificadores de Acesso

[⬆️ Voltar ao Sumário](#sumário)

Acessibilidade define quais consumidores podem criar dependências diretas. Em Java, a ausência de modificador também tem significado: o elemento fica acessível no mesmo package.

---

### 5.1 `public`, `protected`, package-private e `private`

[⬆️ Voltar ao Sumário](#sumário)

| Modificador | Mesmo tipo | Mesmo package | Subclasse em outro package | Qualquer consumidor |
|---|:---:|:---:|:---:|:---:|
| `private` | ✅ | ❌ | ❌ | ❌ |
| package-private | ✅ | ✅ | ❌ | ❌ |
| `protected` | ✅ | ✅ | ✅, pelas regras de herança | ❌ |
| `public` | ✅ | ✅ | ✅ | ✅ |

```java
package com.exemplo.contas;

public class Conta {
    private long saldoCentavos;
    String codigoInterno;              // package-private
    protected void auditar() { }
    public long saldoCentavos() { return saldoCentavos; }
}

final class ValidadorConta { }         // tipo de topo package-private
```

Tipos de topo aceitam `public` ou package-private. Tipos membros também podem ser `private` ou `protected`. No JPMS, um tipo `public` só é acessível normalmente fora do módulo se seu package estiver exportado.

`protected` possui uma regra sutil fora do package: a subclasse acessa o membro herdado em uma expressão qualificada por ela própria ou por um subtipo apropriado; não recebe acesso geral a qualquer instância da superclasse.

> **Referências oficiais:** [JLS §6.6 — Access Control](https://docs.oracle.com/javase/specs/jls/se25/html/jls-6.html#jls-6.6), [JLS §8.5 — Member Type Declarations](https://docs.oracle.com/javase/specs/jls/se25/html/jls-8.html#jls-8.5)

---

### 5.2 Encapsulamento e desenho da superfície pública

[⬆️ Voltar ao Sumário](#sumário)

```java
public final class ContaBancaria {
    private final String numero;
    private long saldoCentavos;

    public ContaBancaria(String numero, long saldoInicial) {
        this.numero = java.util.Objects.requireNonNull(numero);
        if (saldoInicial < 0) throw new IllegalArgumentException("saldo negativo");
        this.saldoCentavos = saldoInicial;
    }

    public long saldoCentavos() {
        return saldoCentavos;
    }

    public boolean sacar(long valor) {
        if (valor <= 0 || valor > saldoCentavos) return false;
        saldoCentavos -= valor;
        return true;
    }
}
```

**Como interpretar o exemplo:** o estado mutável permanece `private`; o construtor cria uma instância válida; a leitura pública não permite escrever diretamente; `sacar` concentra a regra da transição. Encapsulamento não significa criar getters e setters para tudo, e sim preservar invariantes atrás de uma API intencional.

Comece com a menor visibilidade que atende ao consumidor real. `public` vira contrato de fonte e, para bibliotecas, também de compatibilidade binária. Modificadores não são barreira contra código hostil e não substituem autorização, validação ou proteção de segredos.

> **Referências oficiais:** [JLS §8.2 — Class Members](https://docs.oracle.com/javase/specs/jls/se25/html/jls-8.html#jls-8.2), [Secure Coding Guidelines for Java SE](https://www.oracle.com/java/technologies/javase/seccodeguide.html)

---

## Parte 6 — Campos, Accessors e Imutabilidade

[⬆️ Voltar ao Sumário](#sumário)

Java não possui properties como construção da linguagem. Estado é declarado em campos; métodos formam a API. Frameworks podem interpretar pares `getX`/`setX` como propriedades JavaBeans, mas isso é convenção e introspecção, não sintaxe especial do compilador.

---

### 6.1 Campos e métodos de acesso

[⬆️ Voltar ao Sumário](#sumário)

```java
public final class Pessoa {
    private String nome;
    private int idade;

    public Pessoa(String nome, int idade) {
        setNome(nome);
        setIdade(idade);
    }

    public String getNome() { return nome; }

    public void setNome(String nome) {
        if (nome == null || nome.isBlank()) {
            throw new IllegalArgumentException("nome obrigatório");
        }
        this.nome = nome;
    }

    public int getIdade() { return idade; }

    public void setIdade(int idade) {
        if (idade < 0 || idade > 150) {
            throw new IllegalArgumentException("idade inválida");
        }
        this.idade = idade;
    }
}
```

**Leitura guiada:** `this.nome` identifica o campo; `nome` sozinho identifica o parâmetro. Os setters validam antes de alterar o objeto. Se idade não deveria mudar livremente, remova o setter e exponha uma operação com significado de domínio.

Não esconda I/O ou trabalho caro em um getter: consumidores esperam que `getNome()` seja barato e sem efeitos colaterais. Uma operação como `carregarRelatorio()` comunica custo melhor.

> **Referências oficiais:** [JLS §8.3 — Field Declarations](https://docs.oracle.com/javase/specs/jls/se25/html/jls-8.html#jls-8.3), [JLS §8.4 — Method Declarations](https://docs.oracle.com/javase/specs/jls/se25/html/jls-8.html#jls-8.4)

---

### 6.2 JavaBeans e propriedades convencionais

[⬆️ Voltar ao Sumário](#sumário)

O padrão JavaBeans associa `getNome`/`setNome` à propriedade convencional `nome`; para boolean, `isAtivo` também é reconhecido. APIs como `java.beans.Introspector` expõem esses descritores.

```java
var info = java.beans.Introspector.getBeanInfo(Pessoa.class);
for (var propriedade : info.getPropertyDescriptors()) {
    System.out.println(propriedade.getName());
}
```

Frameworks de UI, serialização e persistência podem usar campos, accessors, construtores, annotations ou bytecode enhancement. Leia o contrato do framework: “ser um JavaBean” não é requisito universal de toda classe Java.

Records usam accessors com o próprio nome do componente (`pessoa.nome()`), não `getNome()`. Não adicione getters duplicados sem necessidade de integração.

> **Referências oficiais:** [Introspector](https://docs.oracle.com/en/java/javase/25/docs/api/java.desktop/java/beans/Introspector.html), [Record Classes](https://dev.java/learn/records/)

---

### 6.3 Imutabilidade e cópias defensivas

[⬆️ Voltar ao Sumário](#sumário)

```java
public final class Turma {
    private final String nome;
    private final java.util.List<String> alunos;

    public Turma(String nome, java.util.Collection<String> alunos) {
        this.nome = java.util.Objects.requireNonNull(nome);
        this.alunos = java.util.List.copyOf(alunos);
    }

    public String nome() { return nome; }
    public java.util.List<String> alunos() { return alunos; }
}
```

`final` na referência não congela a lista. `List.copyOf` cria ou reutiliza uma lista **não modificável** e rejeita elementos nulos. Como `String` também é imutável, o estado exposto permanece estável. Se os elementos fossem mutáveis, a lista impediria trocar posições, mas não congelaria cada elemento.

Imutabilidade pede:

- estado definido na construção;
- nenhuma referência mutável interna vazada;
- operações que devolvem novos valores ou encapsulam mutação;
- igualdade e invariantes coerentes.

> **Referências oficiais:** [Creating Unmodifiable Collections](https://dev.java/learn/api/collections-framework/creating-immutable-lists-sets-and-maps/), [List.copyOf](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/util/List.html#copyOf(java.util.Collection))

---

## Parte 7 — Palavras-chave e Modificadores Essenciais

[⬆️ Voltar ao Sumário](#sumário)

Esta parte explica modificadores e palavras que alteram inicialização, herança, despacho, sincronização e fluxo. A lista formal completa fica na Parte 28.

---

### 7.1 `static`

[⬆️ Voltar ao Sumário](#sumário)

```java
public final class Conversoes {
    private Conversoes() { }

    public static final int CENTAVOS_POR_REAL = 100;

    public static long paraCentavos(long reais) {
        return Math.multiplyExact(reais, CENTAVOS_POR_REAL);
    }
}

long valor = Conversoes.paraCentavos(25);
```

Membro `static` pertence à classe, não a uma instância. Campo estático mutável é estado global por class loader e exige política de concorrência. Inicialização estática acontece quando a classe é inicializada segundo regras da JVM; falha pode tornar a classe inutilizável naquele class loader.

Classes aninhadas `static` não carregam uma referência implícita para a instância externa; inner classes não estáticas carregam.

> **Referências oficiais:** [JLS §8.3.1.1 — `static` Fields](https://docs.oracle.com/javase/specs/jls/se25/html/jls-8.html#jls-8.3.1.1), [JLS §12.4 — Initialization](https://docs.oracle.com/javase/specs/jls/se25/html/jls-12.html#jls-12.4)

---

### 7.2 `final`

[⬆️ Voltar ao Sumário](#sumário)

`final` tem três usos centrais:

| Alvo | Efeito |
|---|---|
| variável/campo | impede nova atribuição após inicialização válida |
| método | impede sobrescrita |
| classe | impede herança |

```java
public final class Dinheiro {
    private final long centavos;

    public Dinheiro(long centavos) {
        this.centavos = centavos;
    }

    public final long centavos() { return centavos; }
}
```

Um campo `final` precisa ser atribuído uma vez pelos caminhos permitidos de inicialização, mas uma referência `final` não congela o objeto apontado. No JDK 26, as **final field restrictions** passaram a emitir aviso, por padrão, para mutação de campos finais por reflexão profunda; esse passo prepara uma release futura em que a mutação será bloqueada por padrão. Não baseie serialização ou frameworks em violar imutabilidade sem declarar e isolar essa necessidade.

> **Referências oficiais:** [JLS §8.1.1.2 — `final` Classes](https://docs.oracle.com/javase/specs/jls/se25/html/jls-8.html#jls-8.1.1.2), [JLS §8.3.1.2 — `final` Fields](https://docs.oracle.com/javase/specs/jls/se25/html/jls-8.html#jls-8.3.1.2), [JLS §8.4.3.3 — `final` Methods](https://docs.oracle.com/javase/specs/jls/se25/html/jls-8.html#jls-8.4.3.3), [JEP 500 — Prepare to Make Final Mean Final](https://openjdk.org/jeps/500)

---

### 7.3 `abstract`

[⬆️ Voltar ao Sumário](#sumário)

```java
abstract class Forma {
    abstract double area();

    String descricao() {
        return "Área=" + area();
    }
}

final class Circulo extends Forma {
    private final double raio;
    Circulo(double raio) { this.raio = raio; }
    @Override double area() { return Math.PI * raio * raio; }
}
```

Classe abstrata não pode ser instanciada, mas pode ter estado, construtores e métodos concretos. Método abstrato declara assinatura sem implementação. Interfaces também são abstratas por natureza, mas modelam contratos com regras próprias e herança múltipla de tipo.

Evite chamar métodos sobrescrevíveis em construtores: o método da subclasse pode executar antes de seus campos terem sido inicializados.

> **Referência oficial:** [JLS §8.1.1.1 — Abstract Classes](https://docs.oracle.com/javase/specs/jls/se25/html/jls-8.html#jls-8.1.1.1)

---

### 7.4 Sobrescrita, despacho dinâmico e `@Override`

[⬆️ Voltar ao Sumário](#sumário)

```java
class Animal {
    String som() { return "?"; }
}

class Cachorro extends Animal {
    @Override
    String som() { return "au"; }
}

Animal animal = new Cachorro();
System.out.println(animal.som()); // au
```

Métodos de instância sobrescrevíveis usam despacho dinâmico: o runtime seleciona a implementação pela classe real do objeto. Métodos `static` são ocultados, não sobrescritos; campos também não participam de polimorfismo.

`@Override` pede ao compilador que confirme a relação. Use sempre: um nome ou parâmetro errado vira erro, não um método novo silencioso.

> **Referências oficiais:** [JLS §8.4.8 — Inheritance, Overriding, and Hiding](https://docs.oracle.com/javase/specs/jls/se25/html/jls-8.html#jls-8.4.8), [Override](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/lang/Override.html)

---

### 7.5 `this` e `super`

[⬆️ Voltar ao Sumário](#sumário)

```java
class Pessoa {
    private final String nome;

    Pessoa(String nome) {
        this.nome = java.util.Objects.requireNonNull(nome);
    }
}

class Funcionario extends Pessoa {
    private final long matricula;

    Funcionario(String nome, long matricula) {
        super(nome);
        this.matricula = matricula;
    }
}
```

`this` representa a instância atual e também encadeia outro construtor da mesma classe com `this(...)`. `super` seleciona membros ou construtor da superclasse. Java 25 permite um prólogo antes de `super(...)`/`this(...)`, mas esse **early construction context** restringe acesso à instância em construção.

> **Referências oficiais:** [JLS §15.8.3 — `this`](https://docs.oracle.com/javase/specs/jls/se25/html/jls-15.html#jls-15.8.3), [Flexible Constructor Bodies](https://docs.oracle.com/en/java/javase/25/language/flexible-constructor-bodies.html)

---

### 7.6 `instanceof` e pattern matching

[⬆️ Voltar ao Sumário](#sumário)

```java
static int tamanho(Object valor) {
    if (valor instanceof String texto) {
        return texto.length();
    }
    if (valor instanceof java.util.Collection<?> itens) {
        return itens.size();
    }
    return 0;
}
```

O type pattern testa e declara uma variável já convertida. A análise de fluxo controla onde `texto` e `itens` estão definitivamente disponíveis. `null instanceof String` é `false`.

Record patterns e pattern `switch` permitem desconstruir dados. Patterns com primitivos continuam preview no Java 25/26; não misture sua sintaxe experimental aos patterns permanentes de referência.

> **Referências oficiais:** [Pattern Matching](https://dev.java/learn/pattern-matching/), [JLS §15.20.2 — Type Comparison](https://docs.oracle.com/javase/specs/jls/se25/html/jls-15.html#jls-15.20.2)

---

### 7.7 `try`-with-resources e `AutoCloseable`

[⬆️ Voltar ao Sumário](#sumário)

```java
var caminho = java.nio.file.Path.of("dados.txt");

try (var leitor = java.nio.file.Files.newBufferedReader(caminho)) {
    System.out.println(leitor.readLine());
}
```

Recursos são fechados em ordem inversa mesmo se o corpo falhar. Se corpo e `close()` lançarem, a exceção do corpo é propagada e as de fechamento ficam em `getSuppressed()`.

`AutoCloseable` comunica fechamento, não idempotência, ownership compartilhado ou thread safety. Quem cria o recurso normalmente deve fechá-lo; não feche silenciosamente algo recebido se o contrato não transferiu ownership.

> **Referências oficiais:** [JLS §14.20.3 — `try`-with-resources](https://docs.oracle.com/javase/specs/jls/se25/html/jls-14.html#jls-14.20.3), [AutoCloseable](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/lang/AutoCloseable.html)

---

### 7.8 `synchronized`, `volatile`, `transient`, `native` e `strictfp`

[⬆️ Voltar ao Sumário](#sumário)

| Palavra | Papel | Cuidado |
|---|---|---|
| `synchronized` | exclusão mútua + relações happens-before em monitor | todos os acessos devem seguir o protocolo |
| `volatile` | visibilidade e ordem para leituras/escritas daquele campo | não torna `contador++` atômico |
| `transient` | omite campo da serialização Java padrão | não é segredo nem proteção de memória |
| `native` | método implementado fora de Java | exige biblioteca e ABI corretas |
| `strictfp` | hoje obsoleto | não use em código novo |

`transient` afeta somente a serialização nativa baseada em `Serializable`; bibliotecas JSON e ORMs seguem seus próprios metadados. Desde o Java 17, expressões e métodos de ponto flutuante já usam sempre a semântica estrita restaurada pela plataforma, por isso `strictfp` não altera mais o cálculo.

```java
final class Sinalizador {
    private volatile boolean encerrar;

    void solicitarEncerramento() { encerrar = true; }
    boolean deveEncerrar() { return encerrar; }
}
```

**Leitura guiada:** a thread que chama `solicitarEncerramento` escreve no campo `volatile`; outra thread que consultar `deveEncerrar` volta à memória segundo as garantias do modelo de memória, em vez de depender indefinidamente de um valor antigo. Isso resolve visibilidade desse sinal simples, mas não transforma uma sequência como “ler, somar e escrever” em operação atômica. Para estado composto, use sincronização, locks, atomics ou uma estrutura concorrente adequada.

> **Referências oficiais:** [JLS §8.3.1.4 — `volatile` Fields](https://docs.oracle.com/javase/specs/jls/se25/html/jls-8.html#jls-8.3.1.4), [JLS §17 — Threads and Locks](https://docs.oracle.com/javase/specs/jls/se25/html/jls-17.html), [JEP 306 — Restore Always-Strict Floating-Point Semantics](https://openjdk.org/jeps/306)

---

### 7.9 `sealed`, `non-sealed` e `permits`

[⬆️ Voltar ao Sumário](#sumário)

```java
sealed interface Pagamento permits Pix, Cartao, Boleto { }
record Pix(String chave) implements Pagamento { }
record Cartao(String finalNumero) implements Pagamento { }
non-sealed class Boleto implements Pagamento { }
```

Uma subclasse direta permitida precisa ser `final`, `sealed` ou `non-sealed`. A hierarquia fechada melhora modelagem e permite `switch` exaustivo. As classes permitidas precisam respeitar as regras de proximidade do mesmo módulo nomeado ou package no unnamed module.

> **Referências oficiais:** [JLS §8.1.6 — Permitted Direct Subclasses](https://docs.oracle.com/javase/specs/jls/se25/html/jls-8.html#jls-8.1.6), [Sealed Classes and Interfaces](https://dev.java/learn/inheritance/sealed-classes-and-interfaces/)

---

### 7.10 `yield`, `assert` e controle explícito

[⬆️ Voltar ao Sumário](#sumário)

```java
static int prioridade(String categoria) {
    return switch (categoria) {
        case "urgente" -> 10;
        case "normal" -> 5;
        default -> {
            int calculada = categoria.length();
            yield Math.min(calculada, 4);
        }
    };
}
```

`yield` fornece o valor de um bloco dentro de switch expression; não produz iteradores como em C#. `assert condicao : mensagem` verifica invariantes internas quando assertions estão habilitadas (`java -ea`). Como podem estar desligadas, não use assertions para validar entrada externa nem para executar efeitos necessários.

`break`, `continue`, `return` e `throw` encerram ou desviam o fluxo em níveis diferentes. Labels existem para laços/blocos, mas excesso costuma indicar método complexo.

> **Referências oficiais:** [JLS §14.21 — `yield`](https://docs.oracle.com/javase/specs/jls/se25/html/jls-14.html#jls-14.21), [JLS §14.10 — `assert`](https://docs.oracle.com/javase/specs/jls/se25/html/jls-14.html#jls-14.10)

---

### 7.11 Recursos permanentes do Java 25

[⬆️ Voltar ao Sumário](#sumário)

| Recurso | Uso principal |
|---|---|
| module import declarations | importar packages exportados por um módulo |
| compact source files | reduzir cerimônia em programas pequenos |
| instance main methods | permitir ponto de entrada sem `static` em contextos definidos |
| flexible constructor bodies | validar/preparar dados antes de `super(...)`/`this(...)` sob restrições |

Esses recursos são permanentes no Java 25. “Permanente” não significa que todo projeto deve adotá-los: toolchain, style guide, frameworks e versão mínima dos consumidores continuam importando.

> **Referências oficiais:** [Java Language Changes Summary](https://docs.oracle.com/en/java/javase/25/language/java-language-changes-summary.html), [JEP 511](https://openjdk.org/jeps/511), [JEP 512](https://openjdk.org/jeps/512), [JEP 513](https://openjdk.org/jeps/513)

---

### 7.12 Recursos de preview e incubator

[⬆️ Voltar ao Sumário](#sumário)

Preview é parte da plataforma para avaliação, mas ainda pode mudar ou desaparecer. API incubator vive em módulos `jdk.incubator.*` e possui compromisso ainda menor.

```powershell
javac --enable-preview --release 25 ExemploPreview.java
java --enable-preview ExemploPreview
```

No Java 25, primitive types in patterns e Structured Concurrency são preview; a API Stable Values também é preview. Vector API permanece incubator. **Scoped Values é permanente desde o Java 25** e aparece na seção 16.6. Sempre consulte a release exata: número de preview, assinatura e comportamento mudam entre versões.

> **Referências oficiais:** [JLS §1.5 — Preview Features](https://docs.oracle.com/javase/specs/jls/se25/html/jls-1.html#jls-1.5), [JDK 25 Project](https://openjdk.org/projects/jdk/25/), [JEP 506 — Scoped Values](https://openjdk.org/jeps/506)

---

## Parte 8 — Controle de Fluxo e Operadores

[⬆️ Voltar ao Sumário](#sumário)

Controle de fluxo define quais expressões e instruções executam. Java exige `boolean` nas condições; números, strings e referências não possuem truthiness implícita.

---

### 8.1 `if`, `else` e guard clauses

[⬆️ Voltar ao Sumário](#sumário)

```java
static long aplicarDesconto(long total, int percentual) {
    if (total < 0) throw new IllegalArgumentException("total negativo");
    if (percentual < 0 || percentual > 100) {
        throw new IllegalArgumentException("percentual inválido");
    }
    return total - (total * percentual / 100);
}
```

Guard clauses removem entradas inválidas antes do caminho principal. Chaves continuam recomendadas mesmo em um único statement para impedir erros durante manutenção.

> **Referência oficial:** [JLS §14.9 — The `if` Statement](https://docs.oracle.com/javase/specs/jls/se25/html/jls-14.html#jls-14.9)

---

### 8.2 `switch` statements e expressions

[⬆️ Voltar ao Sumário](#sumário)

```java
enum Status { NOVO, PAGO, ENVIADO }

static String descrever(Status status) {
    return switch (status) {
        case NOVO -> "aguardando pagamento";
        case PAGO -> "preparando envio";
        case ENVIADO -> "em transporte";
    };
}
```

Switch expression produz valor e precisa ser exaustiva. Regras com `->` não têm fall-through. O switch statement clássico com `case:` permite fall-through, mas exige `break` quando essa queda não é intencional.

Selector `null` normalmente lança `NullPointerException`; pattern switch pode declarar `case null` explicitamente. Não use `default` em todo switch de enum apenas para silenciar evolução: às vezes a ausência de default permite ao compilador detectar novo caso.

> **Referências oficiais:** [JLS §14.11 — `switch` Statements](https://docs.oracle.com/javase/specs/jls/se25/html/jls-14.html#jls-14.11), [JLS §15.28 — `switch` Expressions](https://docs.oracle.com/javase/specs/jls/se25/html/jls-15.html#jls-15.28)

---

### 8.3 Laços e controle de iteração

[⬆️ Voltar ao Sumário](#sumário)

```java
var nomes = java.util.List.of("Ana", "Bia", "Caio");

for (int i = 0; i < nomes.size(); i++) {
    System.out.println(i + ": " + nomes.get(i));
}

for (String nome : nomes) {
    if (nome.startsWith("B")) continue;
    System.out.println(nome);
}
```

`for` indexado é adequado quando o índice participa da regra e a estrutura possui acesso eficiente. Enhanced `for` usa array ou `Iterable`. `while` testa antes; `do-while` executa ao menos uma vez.

Não remova diretamente de uma coleção durante enhanced `for`; use `Iterator.remove`, `removeIf` ou construa outra coleção conforme o contrato.

> **Referência oficial:** [JLS §14.12–14.14 — Loop Statements](https://docs.oracle.com/javase/specs/jls/se25/html/jls-14.html#jls-14.12)

---

### 8.4 Operadores, precedência, curto-circuito e overflow

[⬆️ Voltar ao Sumário](#sumário)

| Família | Operadores principais |
|---|---|
| aritmética | `+ - * / %` |
| comparação | `< <= > >= == !=` |
| lógica curta | `&& \|\| !` |
| bit a bit | `& \| ^ ~ << >> >>>` |
| atribuição | `= += -= *= /= ...` |
| condicional | `condição ? a : b` |

```java
boolean seguro = usuario != null && usuario.ativo();
int soma = Math.addExact(Integer.MAX_VALUE, 1); // ArithmeticException
```

`&&` e `||` curto-circuitam; `&` e `|` com boolean avaliam os dois lados. Inteiros transbordam silenciosamente nas operações comuns. Divisão inteira trunca em direção a zero; divisão por zero inteira lança, enquanto ponto flutuante segue IEEE 754.

Java não oferece sobrecarga de operadores definida pelo usuário. Dê nomes a operações de domínio (`somar`, `mais`, `combinar`) e preserve clareza.

> **Referências oficiais:** [JLS §15 — Expressions](https://docs.oracle.com/javase/specs/jls/se25/html/jls-15.html), [Math](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/lang/Math.html)

---

### 8.5 Pattern matching e exaustividade

[⬆️ Voltar ao Sumário](#sumário)

```java
sealed interface Resultado permits Sucesso, Falha { }
record Sucesso(String valor) implements Resultado { }
record Falha(String mensagem) implements Resultado { }

static String exibir(Resultado resultado) {
    return switch (resultado) {
        case Sucesso(var valor) -> valor;
        case Falha(var mensagem) when !mensagem.isBlank() -> "Erro: " + mensagem;
        case Falha _ -> "Erro desconhecido";
    };
}
```

**Leitura guiada:** record patterns desconstruem os componentes; `when` adiciona guard; `_` é unnamed pattern/variable no contexto permitido. Como `Resultado` é sealed e os dois subtipos foram cobertos, a expressão é exaustiva sem `default`.

Ordem importa: um pattern dominante antes de outro específico pode tornar o segundo inalcançável.

> **Referências oficiais:** [JLS §14.11.1 — `switch` Blocks](https://docs.oracle.com/javase/specs/jls/se25/html/jls-14.html#jls-14.11.1), [Pattern Matching](https://dev.java/learn/pattern-matching/)

---

## Parte 9 — Métodos

[⬆️ Voltar ao Sumário](#sumário)

Métodos declaram comportamento por nome, parâmetros, retorno, modificadores, type parameters e exceções checked. Uma boa assinatura torna estados inválidos difíceis de representar e custos importantes visíveis.

---

### 9.1 Assinaturas, parâmetros e retorno

[⬆️ Voltar ao Sumário](#sumário)

```java
public static java.math.BigDecimal calcularTotal(
        java.util.List<java.math.BigDecimal> valores,
        java.math.RoundingMode arredondamento) {
    java.util.Objects.requireNonNull(valores);
    java.util.Objects.requireNonNull(arredondamento);

    return valores.stream()
            .reduce(java.math.BigDecimal.ZERO, java.math.BigDecimal::add)
            .setScale(2, arredondamento);
}
```

A assinatura usada para overload considera nome e tipos dos parâmetros, não nome dos parâmetros nem retorno isoladamente. `throws` faz parte do contrato de checked exceptions, mas não diferencia overloads.

Prefira poucos parâmetros coesos. Quando vários valores sempre viajam juntos ou exigem invariantes, record ou classe de comando torna o contrato mais claro.

> **Referência oficial:** [JLS §8.4 — Method Declarations](https://docs.oracle.com/javase/specs/jls/se25/html/jls-8.html#jls-8.4)

---

### 9.2 Java sempre passa argumentos por valor

[⬆️ Voltar ao Sumário](#sumário)

```java
static void trocar(StringBuilder texto) {
    texto.append(" alterado");        // muda o objeto alcançado
    texto = new StringBuilder("novo"); // muda apenas a cópia local da referência
}

var original = new StringBuilder("valor");
trocar(original);
System.out.println(original); // valor alterado
```

**Leitura guiada:** o argumento é o valor de uma referência, e esse valor é copiado para o parâmetro. Ambos alcançam o mesmo builder, por isso `append` é observado. Reatribuir `texto` não muda a variável `original` do chamador. Java não passa objetos “por referência”.

> **Referência oficial:** [JLS §8.4.1 — Formal Parameters](https://docs.oracle.com/javase/specs/jls/se25/html/jls-8.html#jls-8.4.1)

---

### 9.3 Sobrecarga, resolução e varargs

[⬆️ Voltar ao Sumário](#sumário)

```java
static String formatar(int valor) { return "int=" + valor; }
static String formatar(long valor) { return "long=" + valor; }
static String formatar(Object valor) { return "obj=" + valor; }
static String juntar(String separador, String... partes) {
    return String.join(separador, partes);
}
```

O compilador seleciona métodos aplicáveis considerando widening, boxing, unboxing, generics e varargs em fases. Overloads próximos demais podem tornar `null`, lambdas ou novos tipos ambíguos. Varargs é avaliado por último na resolução e deve ser o parâmetro final.

Não use overload para operações com semânticas diferentes. Nomes explícitos são melhores do que obrigar o leitor a inferir intenção pelo tipo.

> **Referência oficial:** [JLS §15.12 — Method Invocation Expressions](https://docs.oracle.com/javase/specs/jls/se25/html/jls-15.html#jls-15.12)

---

### 9.4 Métodos genéricos, recursão e contratos

[⬆️ Voltar ao Sumário](#sumário)

```java
static <T> T primeiro(java.util.List<? extends T> itens) {
    if (itens.isEmpty()) throw new java.util.NoSuchElementException();
    return itens.getFirst();
}
```

Type parameters aparecem antes do retorno. O wildcard permite receber lista de um subtipo de `T`; o método só lê. Documente comportamento para coleção vazia: exceção, `Optional`, sentinel ou outro contrato.

Recursão é natural para árvores e divisões, mas Java não garante tail-call optimization. Profundidade controlada é indispensável para evitar `StackOverflowError`; uma pilha explícita pode ser melhor para entrada arbitrária.

> **Referências oficiais:** [JLS §8.4.4 — Generic Methods](https://docs.oracle.com/javase/specs/jls/se25/html/jls-8.html#jls-8.4.4), [Methods](https://dev.java/learn/classes-objects/more-on-classes/)

---

## Parte 10 — Enums

[⬆️ Voltar ao Sumário](#sumário)

Enum class define um conjunto controlado de instâncias e pode possuir campos, métodos, construtores e implementação específica por constante.

---

### 10.1 Enum classes e comportamento

[⬆️ Voltar ao Sumário](#sumário)

```java
enum StatusPedido {
    NOVO(false), PAGO(false), ENVIADO(true), CANCELADO(true);

    private final boolean terminal;

    StatusPedido(boolean terminal) {
        this.terminal = terminal;
    }

    public boolean terminal() { return terminal; }
}
```

Compare enums com `==`: as constantes possuem identidade única por class loader. Não persista `ordinal()` como contrato; reordenar constantes altera o número. Prefira nome ou código explícito e versionado.

`values()` devolve novo array; `valueOf` exige nome exato e lança `IllegalArgumentException` quando não encontra.

Uma enum class também pode implementar interface e permitir que cada constante forneça seu próprio comportamento:

```java
interface OperacaoBinaria {
    double aplicar(double a, double b);
}

enum Operacao implements OperacaoBinaria {
    SOMAR {
        @Override public double aplicar(double a, double b) { return a + b; }
    },
    MULTIPLICAR {
        @Override public double aplicar(double a, double b) { return a * b; }
    }
}

double total = Operacao.SOMAR.aplicar(10, 5); // 15.0
```

**Leitura guiada:** o contrato comum vem de `OperacaoBinaria`; o corpo depois de cada constante declara uma classe anônima implícita especializada. Isso elimina `switch` quando o comportamento pertence naturalmente ao próprio conjunto fechado. Se cada opção precisar de muitas dependências ou evoluir independentemente, objetos de estratégia compostos costumam manter o enum menor.

> **Referências oficiais:** [JLS §8.9 — Enum Classes](https://docs.oracle.com/javase/specs/jls/se25/html/jls-8.html#jls-8.9), [JLS §8.9.1 — Enum Constants](https://docs.oracle.com/javase/specs/jls/se25/html/jls-8.html#jls-8.9.1)

---

### 10.2 `EnumSet`, `EnumMap` e flags

[⬆️ Voltar ao Sumário](#sumário)

```java
enum Permissao { LER, ESCREVER, EXCLUIR }

var permissoes = java.util.EnumSet.of(Permissao.LER, Permissao.ESCREVER);
boolean podeEscrever = permissoes.contains(Permissao.ESCREVER);

var rotulos = new java.util.EnumMap<Permissao, String>(Permissao.class);
rotulos.put(Permissao.LER, "Leitura");
```

Em Java, prefira `EnumSet` a máscaras manuais de bits quando os flags são enums. Ele expressa o domínio, oferece operações de conjunto e pode usar representação compacta internamente. `EnumMap` é mapa especializado por enum e mantém ordem natural das constantes.

> **Referências oficiais:** [EnumSet](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/util/EnumSet.html), [EnumMap](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/util/EnumMap.html)

---

## Parte 11 — Classes, Objetos e Records

[⬆️ Voltar ao Sumário](#sumário)

Classes combinam estado, comportamento e regras de inicialização. Records reduzem cerimônia para agregados de dados; não substituem toda classe nem tornam componentes mutáveis profundamente imutáveis.

---

### 11.1 Estrutura completa de uma classe

[⬆️ Voltar ao Sumário](#sumário)

```java
public final class Produto implements Comparable<Produto> {
    private static long criados;
    private final long id;
    private final String nome;
    private final java.math.BigDecimal preco;

    public Produto(long id, String nome, java.math.BigDecimal preco) {
        if (id <= 0) throw new IllegalArgumentException("id inválido");
        this.id = id;
        this.nome = java.util.Objects.requireNonNull(nome);
        this.preco = java.util.Objects.requireNonNull(preco);
        criados++;
    }

    public long id() { return id; }
    public String nome() { return nome; }
    public java.math.BigDecimal preco() { return preco; }

    @Override public int compareTo(Produto outro) {
        return Long.compare(id, outro.id);
    }
}
```

**Leitura guiada:** campos de instância pertencem a cada objeto; `criados` é compartilhado e não é thread-safe; `final` protege atribuição; o construtor valida; `Comparable` define ordem natural por id. Se igualdade também usar id, documente a coerência entre `compareTo` e `equals`.

> **Referência oficial:** [JLS §8 — Classes](https://docs.oracle.com/javase/specs/jls/se25/html/jls-8.html)

---

### 11.2 Construtores e ordem de inicialização

[⬆️ Voltar ao Sumário](#sumário)

A construção combina alocação, valores padrão, inicializadores, construtor da superclasse e corpo do construtor. A ordem simplificada é:

```text
alocação e valores padrão
→ prólogo permitido antes de super/this (Java 25)
→ construtor da superclasse
→ inicializadores de campo/blocos de instância
→ restante do construtor
```

Blocos `static` e blocos de instância são construções distintas:

```java
final class Registro {
    private static final java.util.Set<String> TIPOS;

    static {
        TIPOS = java.util.Set.of("ENTRADA", "SAIDA");
    }

    private final java.time.Instant criadoEm;
    private final String tipo;

    {
        criadoEm = java.time.Instant.now();
    }

    Registro(String tipo) {
        if (!TIPOS.contains(tipo)) throw new IllegalArgumentException("tipo inválido");
        this.tipo = tipo;
    }
}
```

**Leitura guiada:** o bloco `static` participa da inicialização da classe e executa uma vez por inicialização naquele class loader, quando algum uso ativo dispara as regras da JLS — não simplesmente porque o bytecode foi localizado. O bloco sem modificador é um instance initializer: executa em toda construção, depois do construtor da superclasse e antes do restante do construtor atual. Inicializadores e campos seguem ordem textual. Prefira inicialização direta, construtor ou factory quando forem mais claros; esses blocos são úteis, mas podem esconder trabalho e falhas.

```java
class Pedido {
    private final long id;
    private final java.time.Instant criadoEm;

    Pedido(long id) {
        this(id, java.time.Instant.now());
    }

    Pedido(long id, java.time.Instant criadoEm) {
        if (id <= 0) throw new IllegalArgumentException("id inválido");
        this.id = id;
        this.criadoEm = java.util.Objects.requireNonNull(criadoEm);
    }
}
```

`this(...)` delega para outro construtor. Antes do Java 25, `this(...)`/`super(...)` precisava ser o primeiro statement; flexible constructor bodies permitem preparação limitada antes da invocação explícita, sem acesso livre ao objeto ainda não construído.

> **Referências oficiais:** [JLS §8.6 — Instance Initializers](https://docs.oracle.com/javase/specs/jls/se25/html/jls-8.html#jls-8.6), [JLS §8.7 — Static Initializers](https://docs.oracle.com/javase/specs/jls/se25/html/jls-8.html#jls-8.7), [JLS §8.8 — Constructor Declarations](https://docs.oracle.com/javase/specs/jls/se25/html/jls-8.html#jls-8.8), [JLS §12.4 — Initialization](https://docs.oracle.com/javase/specs/jls/se25/html/jls-12.html#jls-12.4), [JLS §12.5 — Creation of New Class Instances](https://docs.oracle.com/javase/specs/jls/se25/html/jls-12.html#jls-12.5)

---

### 11.3 Classes aninhadas, locais e anônimas

[⬆️ Voltar ao Sumário](#sumário)

| Forma | Relação com o contexto externo |
|---|---|
| nested class `static` | não possui instância externa implícita |
| inner class | vinculada a uma instância externa |
| local class | declarada dentro de bloco e captura effectively final |
| anonymous class | declara e instancia subtipo sem nome em uma expressão |

```java
class Relatorio {
    static final class Linha {
        final String texto;
        Linha(String texto) { this.texto = texto; }
    }

    java.util.Comparator<String> porTamanho() {
        return new java.util.Comparator<>() {
            @Override public int compare(String a, String b) {
                return Integer.compare(a.length(), b.length());
            }
        };
    }
}
```

Para interfaces funcionais, lambda costuma ser menor do que anonymous class, mas as semânticas de `this`, identidade e escopo são diferentes. Nested `static` evita reter acidentalmente a instância externa.

> **Referência oficial:** [JLS §8.1.3 — Inner Classes and Enclosing Instances](https://docs.oracle.com/javase/specs/jls/se25/html/jls-8.html#jls-8.1.3)

---

### 11.4 Records e modelagem de dados

[⬆️ Voltar ao Sumário](#sumário)

```java
public record Dinheiro(java.math.BigDecimal valor, java.util.Currency moeda) {
    public Dinheiro {
        valor = java.util.Objects.requireNonNull(valor);
        moeda = java.util.Objects.requireNonNull(moeda);
        if (valor.signum() < 0) throw new IllegalArgumentException("valor negativo");
    }

    public Dinheiro somar(Dinheiro outro) {
        if (!moeda.equals(outro.moeda)) throw new IllegalArgumentException("moedas diferentes");
        return new Dinheiro(valor.add(outro.valor), moeda);
    }
}
```

O header declara componentes, campos `private final`, accessors, construtor canônico, `equals`, `hashCode` e `toString`. O construtor compacto valida e pode normalizar parâmetros antes das atribuições implícitas.

Record é shallowly immutable: se um componente for array ou lista mutável, o estado alcançável ainda pode mudar. Faça cópias defensivas quando o contrato exigir.

> **Referências oficiais:** [JLS §8.10 — Record Classes](https://docs.oracle.com/javase/specs/jls/se25/html/jls-8.html#jls-8.10), [Record Classes](https://dev.java/learn/records/)

---

### 11.5 Igualdade, hash, representação e cópia

[⬆️ Voltar ao Sumário](#sumário)

```java
public final class Cliente {
    private final long id;
    private final String nome;

    public Cliente(long id, String nome) {
        this.id = id;
        this.nome = java.util.Objects.requireNonNull(nome);
    }

    @Override public boolean equals(Object obj) {
        return this == obj || (obj instanceof Cliente outro && id == outro.id);
    }

    @Override public int hashCode() { return Long.hashCode(id); }

    @Override public String toString() {
        return "Cliente[id=" + id + ", nome=" + nome + "]";
    }
}
```

`equals` precisa ser reflexivo, simétrico, transitivo, consistente e falso para `null`. Objetos iguais devem ter o mesmo `hashCode`. Campos usados como chave não devem mudar enquanto o objeto estiver em `HashMap`/`HashSet`.

`clone()` possui contrato histórico difícil e cópia geralmente superficial. Prefira construtor de cópia, factory, record ou método nomeado que declare profundidade e ownership.

> **Referências oficiais:** [Object.equals](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/lang/Object.html#equals(java.lang.Object)), [Object.hashCode](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/lang/Object.html#hashCode()), [Object.clone](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/lang/Object.html#clone())

---

### 11.6 Builder e APIs fluentes

[⬆️ Voltar ao Sumário](#sumário)

Um builder acumula opções por métodos nomeados e só então cria o objeto final. É útil quando há vários parâmetros opcionais, combinações que precisam de validação ou construção incremental. O próprio JDK usa esse estilo em `HttpRequest.Builder` e `Thread.Builder`.

```java
public final class ConsultaHttp {
    private final java.net.URI uri;
    private final java.time.Duration timeout;
    private final java.util.Map<String, String> cabecalhos;

    private ConsultaHttp(Builder builder) {
        this.uri = builder.uri;
        this.timeout = builder.timeout;
        this.cabecalhos = java.util.Map.copyOf(builder.cabecalhos);
    }

    public static Builder para(java.net.URI uri) {
        return new Builder(uri);
    }

    public static final class Builder {
        private final java.net.URI uri;
        private java.time.Duration timeout = java.time.Duration.ofSeconds(10);
        private final java.util.Map<String, String> cabecalhos =
                new java.util.HashMap<>();

        private Builder(java.net.URI uri) {
            this.uri = java.util.Objects.requireNonNull(uri);
        }

        public Builder timeout(java.time.Duration timeout) {
            timeout = java.util.Objects.requireNonNull(timeout);
            if (timeout.isZero() || timeout.isNegative()) {
                throw new IllegalArgumentException("timeout deve ser positivo");
            }
            this.timeout = timeout;
            return this;
        }

        public Builder cabecalho(String nome, String valor) {
            cabecalhos.put(
                    java.util.Objects.requireNonNull(nome),
                    java.util.Objects.requireNonNull(valor));
            return this;
        }

        public ConsultaHttp build() {
            return new ConsultaHttp(this);
        }
    }
}

var consulta = ConsultaHttp.para(java.net.URI.create("https://example.com"))
        .timeout(java.time.Duration.ofSeconds(2))
        .cabecalho("Accept", "application/json")
        .build();
```

**Leitura guiada:** `para` exige o argumento obrigatório e restringe a criação do builder. Cada método opcional valida, altera somente o builder e devolve `this` para encadeamento. `build()` chama o construtor privado; `Map.copyOf` impede que alterações posteriores no builder modifiquem o objeto pronto. O resultado só será profundamente imutável se todos os valores alcançáveis também respeitarem esse contrato.

Builder não é obrigatório para classes pequenas: construtor, static factory ou record costuma ser mais direto quando há poucos dados obrigatórios. Também não confunda API fluente com segurança — ordem inválida, chamadas repetidas e reuso do builder precisam de contrato documentado.

> **Referências oficiais:** [HttpRequest.Builder](https://docs.oracle.com/en/java/javase/25/docs/api/java.net.http/java/net/http/HttpRequest.Builder.html), [Thread.Builder](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/lang/Thread.Builder.html), [JLS §8.5 — Member Type Declarations](https://docs.oracle.com/javase/specs/jls/se25/html/jls-8.html#jls-8.5)

---

## Parte 12 — Herança, Interfaces e Composição

[⬆️ Voltar ao Sumário](#sumário)

Java permite herança simples de classe e implementação de múltiplas interfaces. Herança deve preservar substituição; interfaces descrevem capacidades; composição conecta colaboradores sem criar uma relação “é um”.

---

### 12.1 Herança de classes

[⬆️ Voltar ao Sumário](#sumário)

```java
abstract class Documento {
    private final String titulo;
    protected Documento(String titulo) { this.titulo = titulo; }
    public String titulo() { return titulo; }
    public abstract byte[] renderizar();
}

final class DocumentoTexto extends Documento {
    private final String conteudo;
    DocumentoTexto(String titulo, String conteudo) {
        super(titulo);
        this.conteudo = conteudo;
    }
    @Override public byte[] renderizar() {
        return conteudo.getBytes(java.nio.charset.StandardCharsets.UTF_8);
    }
}
```

Subclasse herda membros acessíveis, não construtores. Ela deve manter pré-condições, pós-condições e invariantes prometidas pela base. Uma base concreta projetada sem extensão em mente costuma ser frágil; `final` é uma escolha válida.

> **Referência oficial:** [JLS §8.4.8 — Inheritance and Overriding](https://docs.oracle.com/javase/specs/jls/se25/html/jls-8.html#jls-8.4.8)

---

### 12.2 Interfaces modernas

[⬆️ Voltar ao Sumário](#sumário)

```java
interface Repositorio<T, ID> {
    java.util.Optional<T> buscar(ID id);
    void salvar(T entidade);

    default boolean existe(ID id) {
        return buscar(id).isPresent();
    }

    static <T, ID> Repositorio<T, ID> somenteLeitura(Repositorio<T, ID> origem) {
        return new Repositorio<>() {
            public java.util.Optional<T> buscar(ID id) { return origem.buscar(id); }
            public void salvar(T entidade) { throw new UnsupportedOperationException(); }
        };
    }
}
```

Interfaces podem declarar métodos abstratos, `default`, `static` e `private`, além de constantes implicitamente `public static final` e tipos membros. Métodos de `Object` não recebem implementação default equivalente.

Default methods evoluem interfaces, mas conflitos entre múltiplas interfaces precisam ser resolvidos explicitamente. Não use default para acumular estado oculto; interfaces não têm campos de instância.

> **Referências oficiais:** [JLS §9 — Interfaces](https://docs.oracle.com/javase/specs/jls/se25/html/jls-9.html), [Interfaces](https://dev.java/learn/interfaces/)

---

### 12.3 Tipos sealed e hierarquias fechadas

[⬆️ Voltar ao Sumário](#sumário)

```java
sealed interface Expressao permits Numero, Soma { }
record Numero(int valor) implements Expressao { }
record Soma(Expressao esquerda, Expressao direita) implements Expressao { }

static int avaliar(Expressao e) {
    return switch (e) {
        case Numero(var valor) -> valor;
        case Soma(var a, var b) -> avaliar(a) + avaliar(b);
    };
}
```

Hierarquia fechada torna o conjunto de variantes conhecido ao compilador. É útil para ASTs, estados e resultados. Se extensões externas fazem parte do contrato, uma interface aberta pode ser mais adequada.

> **Referência oficial:** [JLS §9.1.4 — Permits Clause](https://docs.oracle.com/javase/specs/jls/se25/html/jls-9.html#jls-9.1.4)

---

### 12.4 Composição antes de herança acidental

[⬆️ Voltar ao Sumário](#sumário)

```java
interface Notificador { void enviar(String mensagem); }

final class PedidoService {
    private final Notificador notificador;

    PedidoService(Notificador notificador) {
        this.notificador = java.util.Objects.requireNonNull(notificador);
    }

    void confirmar(long pedidoId) {
        // regra de confirmação omitida
        notificador.enviar("Pedido confirmado: " + pedidoId);
    }
}
```

`PedidoService` **tem um** notificador; não **é um** notificador. A dependência entra pelo construtor, pode ser substituída em testes e não expõe detalhes de criação. Isso é composição e dependency injection manual, sem exigir framework.

Prefira herança quando existe substituição verdadeira e contrato estável; composição quando você quer delegar capacidade, variar estratégia ou controlar ciclo de vida separadamente.

> **Referências oficiais:** [JLS §8.1.4 — Superclasses and Subclasses](https://docs.oracle.com/javase/specs/jls/se25/html/jls-8.html#jls-8.1.4), [ServiceLoader](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/util/ServiceLoader.html)

---

## Parte 13 — Interfaces Funcionais, Lambdas e Method References

[⬆️ Voltar ao Sumário](#sumário)

Java trata comportamento como valor por meio de interfaces funcionais. Lambdas e method references não criam um “tipo função” separado: cada expressão precisa de um **target type** funcional.

---

### 13.1 Interfaces funcionais

[⬆️ Voltar ao Sumário](#sumário)

Uma interface funcional possui exatamente um método abstrato relevante. Métodos `default`, `static`, `private` e métodos públicos de `Object` não aumentam essa contagem.

```java
@FunctionalInterface
interface RegraDesconto {
    long aplicar(long totalCentavos);

    default RegraDesconto depois(RegraDesconto outra) {
        return total -> outra.aplicar(aplicar(total));
    }
}

RegraDesconto dezPorCento = total -> total - total / 10;
System.out.println(dezPorCento.aplicar(10_000)); // 9000
```

**Leitura guiada:** `@FunctionalInterface` faz o compilador proteger a intenção. A lambda implementa `aplicar`. O método `depois` devolve outra lambda que executa duas regras em sequência. Interfaces funcionais próprias fazem sentido quando o nome carrega semântica de domínio ou quando o método precisa declarar checked exception.

> **Referências oficiais:** [JLS §9.8 — Functional Interfaces](https://docs.oracle.com/javase/specs/jls/se25/html/jls-9.html#jls-9.8), [FunctionalInterface](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/lang/FunctionalInterface.html)

---

### 13.2 `Function`, `Consumer`, `Supplier` e `Predicate`

[⬆️ Voltar ao Sumário](#sumário)

| Interface | Método | Ideia |
|---|---|---|
| `Function<T,R>` | `R apply(T)` | transforma um valor |
| `UnaryOperator<T>` | `T apply(T)` | transforma sem mudar tipo |
| `Consumer<T>` | `void accept(T)` | consome e produz efeito |
| `Supplier<T>` | `T get()` | fornece valor sob demanda |
| `Predicate<T>` | `boolean test(T)` | testa condição |
| `BiFunction<T,U,R>` | `R apply(T,U)` | transforma dois valores |

```java
java.util.function.Predicate<String> preenchida = s -> s != null && !s.isBlank();
java.util.function.Function<String, Integer> tamanho = String::length;
java.util.function.Supplier<java.time.Instant> relogio = java.time.Instant::now;

if (preenchida.test("Java")) {
    System.out.println(tamanho.apply("Java"));
}
```

As especializações `IntFunction`, `ToLongFunction`, `IntPredicate` e outras evitam boxing em caminhos numéricos. Escolha a interface pelo contrato, não apenas pela quantidade de parâmetros.

> **Referência oficial:** [`java.util.function`](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/util/function/package-summary.html)

---

### 13.3 Expressões lambda

[⬆️ Voltar ao Sumário](#sumário)

```java
var nomes = new java.util.ArrayList<>(java.util.List.of("Bia", "Ana", "Carlos"));

nomes.removeIf(nome -> nome.length() < 4);
nomes.sort((a, b) -> a.compareToIgnoreCase(b));
nomes.forEach(nome -> System.out.println(nome));
```

Tipos dos parâmetros normalmente vêm do target type. Uma lambda de expressão devolve o resultado implicitamente quando o método funcional não é `void`; um bloco precisa de `return` nos caminhos aplicáveis.

`this` dentro de lambda continua sendo o `this` do contexto envolvente, diferente de anonymous class. Não dependa da identidade ou classe concreta do objeto gerado para uma lambda; a especificação não promete nova instância por avaliação.

> **Referências oficiais:** [JLS §15.27 — Lambda Expressions](https://docs.oracle.com/javase/specs/jls/se25/html/jls-15.html#jls-15.27), [Lambda Expressions](https://dev.java/learn/lambdas/)

---

### 13.4 Method references e composição

[⬆️ Voltar ao Sumário](#sumário)

| Forma | Exemplo | Referência |
|---|---|---|
| método estático | `Integer::parseInt` | `Integer.parseInt(texto)` |
| método de objeto específico | `saida::println` | `saida.println(valor)` |
| método de objeto arbitrário | `String::trim` | `texto.trim()` |
| construtor | `ArrayList::new` | `new ArrayList<>()` |

```java
java.util.function.Function<String, String> limpar = String::strip;
java.util.function.Function<String, Integer> converter = Integer::parseInt;

var pipeline = limpar.andThen(converter);
System.out.println(pipeline.apply(" 42 ")); // 42
```

Method reference não executa o método naquele momento; cria uma implementação do target type. Se a referência esconder qual overload será escolhido, uma lambda explícita pode ser mais clara.

> **Referência oficial:** [JLS §15.13 — Method Reference Expressions](https://docs.oracle.com/javase/specs/jls/se25/html/jls-15.html#jls-15.13)

---

### 13.5 Closures, callbacks e listeners

[⬆️ Voltar ao Sumário](#sumário)

```java
static java.util.function.Predicate<String> comPrefixo(String prefixo) {
    String normalizado = prefixo.toLowerCase(java.util.Locale.ROOT);
    return texto -> texto.toLowerCase(java.util.Locale.ROOT).startsWith(normalizado);
}
```

A lambda captura `normalizado`, que é effectively final. A captura preserva o valor necessário mesmo após o método retornar. Campos podem ser mutados porque a lambda captura `this`, mas isso introduz estado compartilhado e precisa de sincronização quando há concorrência.

Java não possui events como construção da linguagem. APIs modelam callbacks e listeners com interfaces funcionais ou interfaces comuns. Defina como registrar, remover, ordenar e tratar falhas dos listeners; referências esquecidas podem impedir GC.

> **Referências oficiais:** [JLS §15.27.2 — Lambda Body](https://docs.oracle.com/javase/specs/jls/se25/html/jls-15.html#jls-15.27.2), [JLS §6.5.6.1 — Simple Expression Names](https://docs.oracle.com/javase/specs/jls/se25/html/jls-6.html#jls-6.5.6.1)

---

## Parte 14 — Stream API e Processamento de Sequências

[⬆️ Voltar ao Sumário](#sumário)

Um stream descreve um pipeline de processamento. Ele não armazena elementos, não é um `InputStream` e normalmente só pode ser consumido uma vez.

---

### 14.1 O que é um stream?

[⬆️ Voltar ao Sumário](#sumário)

```java
record Produto(String nome, long precoCentavos, boolean ativo) { }

var produtos = java.util.List.of(
        new Produto("Livro", 5_000, true),
        new Produto("Cabo", 2_000, false),
        new Produto("Teclado", 15_000, true));

var nomesAtivos = produtos.stream()
        .filter(Produto::ativo)
        .sorted(java.util.Comparator.comparingLong(Produto::precoCentavos))
        .map(Produto::nome)
        .toList();
```

**Leitura guiada:** `stream()` cria a visão sequencial; `filter`, `sorted` e `map` descrevem etapas; `toList` dispara o pipeline e materializa lista não modificável. A coleção original não é alterada.

Streams favorecem transformações sem efeitos colaterais. Um loop continua melhor quando o fluxo possui estado complexo, múltiplas saídas ou depuração passo a passo mais importante que composição.

> **Referências oficiais:** [Stream](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/util/stream/Stream.html), [The Stream API](https://dev.java/learn/api/streams/)

---

### 14.2 Operações intermediárias e terminais

[⬆️ Voltar ao Sumário](#sumário)

| Categoria | Exemplos | Efeito |
|---|---|---|
| intermediária stateless | `map`, `filter`, `peek` | devolve outro stream |
| intermediária stateful | `sorted`, `distinct`, `limit` | pode bufferizar ou depender de ordem |
| terminal | `toList`, `collect`, `reduce`, `count`, `forEach` | consome o pipeline |
| short-circuit | `findFirst`, `anyMatch`, `limit` | pode evitar processar tudo |

```java
long quantidade = java.util.stream.IntStream.rangeClosed(1, 1_000)
        .filter(n -> n % 2 == 0)
        .limit(10)
        .count();
```

Operações intermediárias são lazy. Sem terminal, a função pode nunca executar. `peek` existe principalmente para observação; não coloque regra necessária em efeito colateral que pode desaparecer por otimização ou curto-circuito.

> **Referência oficial:** [Stream package summary](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/util/stream/package-summary.html)

---

### 14.3 `Iterable`, `Iterator` e `Spliterator`

[⬆️ Voltar ao Sumário](#sumário)

```java
Iterable<String> nomes = java.util.List.of("Ana", "Bia");
for (String nome : nomes) {
    System.out.println(nome);
}

java.util.Iterator<String> it = nomes.iterator();
while (it.hasNext()) {
    System.out.println(it.next());
}
```

`Iterable` promete produzir `Iterator`; enhanced `for` usa esse contrato. Iterator é cursor mutável e sua política de remoção depende da implementação. `Spliterator` adiciona divisão e características como `ORDERED`, `SIZED`, `SORTED` e `IMMUTABLE`, que ajudam Stream API e paralelismo.

Expor `Iterable<T>` comunica iteração, não tamanho, índice, reuso ou mutabilidade. Escolha um contrato mais forte somente quando o consumidor precisa dele.

> **Referências oficiais:** [Iterable](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/lang/Iterable.html), [Spliterator](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/util/Spliterator.html)

---

### 14.4 Collectors, redução e agrupamento

[⬆️ Voltar ao Sumário](#sumário)

```java
record Venda(String categoria, long centavos) { }

var vendas = java.util.List.of(
        new Venda("livros", 5_000),
        new Venda("livros", 3_000),
        new Venda("tech", 10_000));

var totais = vendas.stream().collect(
        java.util.stream.Collectors.groupingBy(
                Venda::categoria,
                java.util.stream.Collectors.summingLong(Venda::centavos)));
```

Collector combina supplier, accumulator, combiner, finisher e características. `groupingBy` cria grupos; collector downstream soma cada grupo. Em `toMap`, declare merge function quando chaves podem repetir, ou o collector lançará.

Redução deve usar operação associativa e identidade correta, especialmente em paralelo. Subtração, concatenação mutável compartilhada e arredondamento etapa a etapa podem produzir resultados dependentes da divisão.

> **Referência oficial:** [Collectors](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/util/stream/Collectors.html)

---

### 14.5 Avaliação lazy, streams paralelos e armadilhas

[⬆️ Voltar ao Sumário](#sumário)

```java
try (var linhas = java.nio.file.Files.lines(java.nio.file.Path.of("dados.txt"))) {
    long erros = linhas.filter(linha -> linha.startsWith("ERROR")).count();
    System.out.println(erros);
}
```

`Files.lines` mantém recurso aberto até o stream ser fechado; por isso o try-with-resources envolve toda a operação. Stream não pode ser reutilizado após terminal.

`parallel()` não significa “mais rápido”. Há custo de divisão, coordenação e combinação; o common ForkJoinPool é compartilhado; bloqueio pode degradar throughput; ordem e efeitos laterais complicam correção. Meça com dados representativos e use operações stateless, associativas e thread-safe.

> **Referências oficiais:** [Package `java.util.stream`](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/util/stream/package-summary.html), [Files.lines](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/nio/file/Files.html#lines(java.nio.file.Path))

---

## Parte 15 — Collections Framework

[⬆️ Voltar ao Sumário](#sumário)

Coleções expressam contratos diferentes de ordem, duplicidade, chave, acesso e mutabilidade. Implementação deve ser escolhida pelo padrão de uso, não por hábito.

---

### 15.1 Hierarquia e contratos principais

[⬆️ Voltar ao Sumário](#sumário)

```text
Iterable<E>
└── Collection<E>
    ├── List<E>        ordem posicional e duplicatas
    ├── Set<E>         unicidade
    │   └── SortedSet / NavigableSet
    └── Queue<E>
        └── Deque<E>

Map<K,V>               hierarquia separada de Collection
└── SortedMap / NavigableMap
```

`Collection` não promete thread safety nem imutabilidade. Métodos mutadores podem ser opcionais e lançar `UnsupportedOperationException`. As interfaces `SequencedCollection`, `SequencedSet` e `SequencedMap` modelam ordem de encontro e operações nas duas extremidades.

> **Referências oficiais:** [Collections Framework](https://dev.java/learn/api/collections-framework/), [`java.util`](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/util/package-summary.html)

---

### 15.2 `ArrayList` e `LinkedList`

[⬆️ Voltar ao Sumário](#sumário)

```java
var nomes = new java.util.ArrayList<String>();
nomes.add("Ana");
nomes.add("Carlos");
nomes.add(1, "Bia");

System.out.println(nomes.get(1)); // Bia
System.out.println(nomes.size()); // 3
```

`ArrayList` é array redimensionável: acesso por índice é constante; inserção/remoção no meio desloca elementos; crescimento pode realocar. É a escolha inicial comum para lista em memória.

`LinkedList` implementa `List` e `Deque`, mas acesso posicional percorre nós e cada elemento carrega overhead. Use-a principalmente quando o contrato de deque e o perfil medido justificarem; para pilha/fila, `ArrayDeque` costuma ser melhor.

> **Referências oficiais:** [ArrayList](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/util/ArrayList.html), [Choosing Between ArrayList and LinkedList](https://dev.java/learn/api/collections-framework/arraylist-vs-linkedlist/)

---

### 15.3 `HashMap`, mapas ordenados e chaves

[⬆️ Voltar ao Sumário](#sumário)

```java
var estoque = new java.util.HashMap<String, Integer>();
estoque.put("livro", 10);
estoque.merge("livro", 2, Integer::sum);
estoque.computeIfAbsent("cabo", chave -> 0);

int livros = estoque.getOrDefault("livro", 0); // 12
```

| Implementação | Ordem | Busca típica |
|---|---|---|
| `HashMap` | não garante ordem | média constante com hash adequado |
| `LinkedHashMap` | inserção ou acesso configurado | média constante |
| `TreeMap` | ordem natural/comparator | logarítmica |
| `ConcurrentHashMap` | concorrente, sem `null` | operações compostas atômicas específicas |

Chave deve preservar `equals` e `hashCode` enquanto armazenada. `computeIfAbsent` não é apenas açúcar: em mapa concorrente, consulte as garantias atômicas e não faça computação recursiva sobre a mesma chave.

> **Referências oficiais:** [Map](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/util/Map.html), [HashMap](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/util/HashMap.html)

---

### 15.4 Sets, filas, deques e filas de prioridade

[⬆️ Voltar ao Sumário](#sumário)

| Necessidade | Interface/implementação inicial |
|---|---|
| unicidade por hash | `HashSet` |
| unicidade ordenada | `TreeSet` |
| FIFO/deque/pilha | `ArrayDeque` |
| menor/maior prioridade primeiro | `PriorityQueue` |
| comunicação bloqueante | `BlockingQueue` |

```java
var fila = new java.util.ArrayDeque<String>();
fila.addLast("primeiro");
fila.addLast("segundo");
System.out.println(fila.removeFirst());

var prioridades = new java.util.PriorityQueue<Integer>();
prioridades.addAll(java.util.List.of(30, 10, 20));
System.out.println(prioridades.remove()); // 10
```

`PriorityQueue` garante apenas o elemento da cabeça segundo a ordem; iterar não devolve tudo ordenado. `ArrayDeque` não aceita `null`, o que evita confundir ausência com elemento.

Sets mutáveis também oferecem operações clássicas de álgebra de conjuntos:

```java
var a = java.util.Set.of("A", "B", "C");
var b = java.util.Set.of("B", "C", "D");

var uniao = new java.util.HashSet<>(a);
uniao.addAll(b);                  // A, B, C, D

var intersecao = new java.util.HashSet<>(a);
intersecao.retainAll(b);          // B, C

var diferenca = new java.util.HashSet<>(a);
diferenca.removeAll(b);           // A
```

**Leitura guiada:** `addAll`, `retainAll` e `removeAll` alteram o set que recebe a chamada. As cópias com `new HashSet<>(a)` preservam `a`, que neste exemplo veio de `Set.of` e não aceita mutação. Ordem de iteração não faz parte do contrato de `HashSet`; use implementação ordenada somente quando essa ordem for requisito.

> **Referências oficiais:** [Set](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/util/Set.html), [Queue](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/util/Queue.html), [ArrayDeque](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/util/ArrayDeque.html), [PriorityQueue](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/util/PriorityQueue.html)

---

### 15.5 Coleções imutáveis, views e cópias

[⬆️ Voltar ao Sumário](#sumário)

```java
var origem = new java.util.ArrayList<>(java.util.List.of("A", "B"));
var view = java.util.Collections.unmodifiableList(origem);
var copia = java.util.List.copyOf(origem);

origem.add("C");
System.out.println(view.size());  // 3: reflete a origem
System.out.println(copia.size()); // 2: snapshot
```

**Leitura guiada:** `unmodifiableList` cria view que bloqueia mutação por aquela referência, mas observa alterações na origem. `List.copyOf` captura o conteúdo naquele momento e devolve lista não modificável. Nenhuma opção congela elementos mutáveis.

`List.of`, `Set.of` e `Map.of` rejeitam `null`; factories de set/map também rejeitam duplicatas relevantes.

> **Referências oficiais:** [Creating Unmodifiable Collections](https://dev.java/learn/api/collections-framework/creating-immutable-lists-sets-and-maps/), [Collections.unmodifiableList](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/util/Collections.html#unmodifiableList(java.util.List))

---

### 15.6 Como escolher a coleção certa

[⬆️ Voltar ao Sumário](#sumário)

| Pergunta dominante | Escolha inicial |
|---|---|
| preciso de índice e ordem? | `ArrayList` |
| preciso localizar por chave? | `HashMap` |
| preciso impedir duplicatas? | `HashSet` |
| preciso de ordenação navegável? | `TreeSet`/`TreeMap` |
| preciso das duas extremidades? | `ArrayDeque` |
| preciso compartilhar entre threads? | coleção de `java.util.concurrent` adequada |
| preciso apenas devolver leitura estável? | cópia não modificável |

Considere complexidade, memória, padrão de iteração, `null`, mutabilidade, ordenação e concorrência. Exponha a interface mínima que mantém o contrato verdadeiro, sem esconder capacidades essenciais.

> **Referência oficial:** [Outline of the Collections Framework](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/util/doc-files/coll-reference.html)

---

## Parte 16 — Tarefas, Assincronismo e Virtual Threads

[⬆️ Voltar ao Sumário](#sumário)

Java não possui `async`/`await`. Assincronismo e concorrência são expressos por threads, executors, futures, callbacks e APIs estruturadas. Escolha pelo tipo de trabalho e pelo contrato de cancelamento.

---

### 16.1 `ExecutorService`, `Callable` e `Future`

[⬆️ Voltar ao Sumário](#sumário)

```java
try (var executor = java.util.concurrent.Executors.newFixedThreadPool(4)) {
    java.util.concurrent.Future<Integer> futuro =
            executor.submit(() -> 20 + 22);

    int resultado = futuro.get(2, java.util.concurrent.TimeUnit.SECONDS);
    System.out.println(resultado);
}
```

`Callable<T>` devolve valor e pode lançar checked exception; `submit` agenda e devolve `Future`. `get` bloqueia a thread chamadora e encapsula falhas em `ExecutionException`. Desde Java 19, `ExecutorService` é `AutoCloseable`; o fechamento realiza shutdown e aguarda término segundo o contrato.

Pool limitado é útil para CPU ou recursos escassos. Não crie fila ilimitada sem política de overload.

> **Referências oficiais:** [ExecutorService](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/util/concurrent/ExecutorService.html), [Future](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/util/concurrent/Future.html)

---

### 16.2 `CompletableFuture`

[⬆️ Voltar ao Sumário](#sumário)

```java
var futuro = java.util.concurrent.CompletableFuture
        .supplyAsync(() -> "42")
        .thenApply(Integer::parseInt)
        .thenApply(n -> n * 2)
        .orTimeout(2, java.util.concurrent.TimeUnit.SECONDS)
        .exceptionally(falha -> -1);

System.out.println(futuro.join());
```

`thenApply` transforma resultado; `thenCompose` evita future aninhado quando a função já devolve outro estágio; `thenCombine` combina operações independentes. Métodos sem sufixo `Async` podem executar na thread que completa a etapa; métodos `Async` sem executor usam o executor padrão documentado.

`join` propaga `CompletionException` unchecked; `get` usa checked exceptions. Timeout não garante interromper a operação subjacente. Defina executor, cancelamento e política de erro conscientemente.

> **Referência oficial:** [CompletableFuture](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/util/concurrent/CompletableFuture.html)

---

### 16.3 Virtual threads

[⬆️ Voltar ao Sumário](#sumário)

```java
try (var executor = java.util.concurrent.Executors.newVirtualThreadPerTaskExecutor()) {
    var futuros = java.util.stream.IntStream.range(0, 1_000)
            .mapToObj(i -> executor.submit(() -> buscarRemotamente(i)))
            .toList();

    for (var futuro : futuros) {
        System.out.println(futuro.get());
    }
}

// método ilustrativo
static String buscarRemotamente(int id) throws InterruptedException {
    Thread.sleep(10);
    return "item-" + id;
}
```

Virtual threads são threads leves gerenciadas pelo runtime, adequadas a grande quantidade de tarefas que bloqueiam esperando I/O. Elas preservam o estilo thread-per-request e stack traces compreensíveis. Não tornam CPU mais rápida e não devem ser agrupadas em pool arbitrário; limite o recurso escasso real com semaphore, pool de conexão ou protocolo.

Evite caches baseados em `ThreadLocal` por thread virtual sem medir memória. Desde o JDK 24, a JVM consegue desmontar virtual threads que bloqueiam dentro de `synchronized` nos casos cobertos pelo JEP 491; chamadas nativas e foreign functions ainda podem prender a virtual thread à carrier durante a execução. Meça no JDK realmente usado.

> **Referências oficiais:** [Virtual Threads](https://docs.oracle.com/en/java/javase/25/core/virtual-threads.html), [Thread](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/lang/Thread.html), [JEP 444](https://openjdk.org/jeps/444), [JEP 491 — Synchronize Virtual Threads without Pinning](https://openjdk.org/jeps/491)

---

### 16.4 Cancelamento, timeout e interrupção

[⬆️ Voltar ao Sumário](#sumário)

```java
static void processar() throws InterruptedException {
    while (!Thread.currentThread().isInterrupted()) {
        executarUnidade();
    }
}

static void executarUnidade() throws InterruptedException {
    Thread.sleep(50);
}
```

Interrupção é solicitação cooperativa. Métodos bloqueantes podem lançar `InterruptedException` e limpar o flag. Se o método não puder propagá-la, restaure o estado com `Thread.currentThread().interrupt()` antes de retornar ou encapsular.

`Future.cancel(true)` tenta interromper; não garante que código que ignora interrupção pare. Timeout limita a espera de uma operação específica, não é cancelamento transitivo universal.

> **Referências oficiais:** [Thread interruption](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/lang/Thread.html#interrupt()), [Future.cancel](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/util/concurrent/Future.html#cancel(boolean))

---

### 16.5 Structured Concurrency como preview

[⬆️ Voltar ao Sumário](#sumário)

Structured Concurrency trata subtarefas relacionadas como unidade: escopo, falha, cancelamento, join e observabilidade seguem a mesma árvore. No Java 25 ela é preview e exige `--enable-preview`.

```java
// Esboço conceitual; compile com preview da release correspondente.
try (var scope = java.util.concurrent.StructuredTaskScope.open()) {
    var usuario = scope.fork(() -> buscarUsuario());
    var pedidos = scope.fork(() -> buscarPedidos());
    scope.join();
    return new Pagina(usuario.get(), pedidos.get());
}
```

Não copie esse exemplo entre releases sem consultar a assinatura oficial: a API mudou durante previews. A ideia estável é que subtarefas não escapem do lifetime do escopo.

> **Referências oficiais:** [Structured Concurrency](https://docs.oracle.com/en/java/javase/25/core/structured-concurrency.html), [JEP 505](https://openjdk.org/jeps/505)

---

### 16.6 Scoped Values

[⬆️ Voltar ao Sumário](#sumário)

`ScopedValue<T>` compartilha um valor imutável do chamador com métodos chamados direta ou indiretamente dentro de um escopo dinâmico delimitado. A API tornou-se permanente no Java 25. Ela é especialmente útil para contexto de requisição, identidade, tracing e outras informações que precisam fluir pela pilha — inclusive para subtarefas estruturadas — sem virar parâmetro de todos os métodos nem estado mutável em `ThreadLocal`.

```java
final class ContextoRequisicao {
    private static final ScopedValue<String> CORRELACAO =
            ScopedValue.newInstance();

    static void atender(String id) {
        ScopedValue.where(CORRELACAO, id)
                .run(ContextoRequisicao::registrar);
    }

    private static void registrar() {
        System.out.println("correlação=" + CORRELACAO.get());
    }
}
```

**Leitura guiada:** `CORRELACAO` é a chave, não o valor de uma requisição específica. `where(...).run(...)` cria uma ligação válida apenas durante a chamada de `run`; métodos alcançados nesse intervalo podem ler com `get`. Ao sair do escopo, a ligação desaparece automaticamente. Não há `set` para alterar uma ligação distante, e `get` fora de um escopo vinculado falha. O objeto armazenado ainda pode ser mutável, por isso prefira valores imutáveis.

`ScopedValue` não substitui todo `ThreadLocal`: caches mutáveis por thread e integração legada possuem contratos diferentes. Também não transmite contexto automaticamente para tarefas arbitrárias em executors; a herança definida pela plataforma está associada a subtarefas criadas por Structured Concurrency.

> **Referências oficiais:** [ScopedValue API](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/lang/ScopedValue.html), [JEP 506 — Scoped Values](https://openjdk.org/jeps/506)

---

## Parte 17 — Generics

[⬆️ Voltar ao Sumário](#sumário)

Generics adicionam segurança estática e reutilização, mas são implementados principalmente por type erasure. O runtime não cria uma classe diferente para cada argumento de tipo.

---

### 17.1 Tipos e métodos parametrizados

[⬆️ Voltar ao Sumário](#sumário)

```java
final class Caixa<T> {
    private final T valor;
    Caixa(T valor) { this.valor = valor; }
    T valor() { return valor; }
}

static <T> java.util.List<T> duplicar(T valor) {
    return java.util.List.of(valor, valor);
}

var caixa = new Caixa<>("Java");
String texto = caixa.valor();
```

Diamond `<>` permite inferência do argumento. Tipo raw (`Caixa caixa`) desliga parte da segurança para compatibilidade legada e pode introduzir heap pollution; não use em código novo.

> **Referências oficiais:** [JLS §4.5 — Parameterized Types](https://docs.oracle.com/javase/specs/jls/se25/html/jls-4.html#jls-4.5), [Generics](https://dev.java/learn/generics/)

---

### 17.2 Bounds, wildcards e PECS

[⬆️ Voltar ao Sumário](#sumário)

```java
static double somar(java.util.List<? extends Number> numeros) {
    return numeros.stream().mapToDouble(Number::doubleValue).sum();
}

static void adicionarInteiros(java.util.List<? super Integer> destino) {
    destino.add(1);
    destino.add(2);
}
```

`? extends Number` é produtor: você lê como `Number`, mas não adiciona um subtipo arbitrário. `? super Integer` é consumidor: aceita `Integer`, mas a leitura segura é `Object`. A regra mnemônica **PECS** (*Producer Extends, Consumer Super*) ajuda, mas não substitui analisar operações reais.

Bounds múltiplos usam `T extends Classe & Interface1 & Interface2`, com a classe primeiro.

> **Referência oficial:** [Wildcards](https://dev.java/learn/generics/wildcards/)

---

### 17.3 Type erasure e restrições

[⬆️ Voltar ao Sumário](#sumário)

Após erasure, muitos argumentos de tipo não estão disponíveis integralmente. Consequências:

- não existe `new T()`;
- não existe `new T[10]`;
- `obj instanceof List<String>` é ilegal; use `List<?>`;
- não há overload apenas por `List<String>` versus `List<Integer>`;
- campos `static` não pertencem a cada instanciação genérica;
- primitivos não são argumentos de tipo (`List<int>` é inválido).

```java
static <T> T criar(java.util.function.Supplier<? extends T> fabrica) {
    return fabrica.get();
}

var lista = criar(java.util.ArrayList<String>::new);
```

Passe factory, `Class<T>` ou type token quando criação/reflection precisa de informação explícita. Bridges sintetizados pelo compilador preservam polimorfismo após erasure.

> **Referências oficiais:** [Type Erasure](https://dev.java/learn/generics/type-erasure/), [JLS §4.6 — Type Erasure](https://docs.oracle.com/javase/specs/jls/se25/html/jls-4.html#jls-4.6)

---

### 17.4 Invariância e captura de wildcard

[⬆️ Voltar ao Sumário](#sumário)

`List<Integer>` **não** é subtipo de `List<Number>`. Se fosse, alguém poderia adicionar `Double` a uma lista de inteiros. Wildcards representam variância de uso:

```java
java.util.List<Integer> inteiros = java.util.List.of(1, 2);
java.util.List<? extends Number> numeros = inteiros;
Number primeiro = numeros.getFirst();
```

Às vezes um método público aceita `List<?>` e delega a helper genérico para capturar o tipo desconhecido. Não espalhe casts unchecked para “vencer” o compilador; o erro pode reaparecer longe como `ClassCastException`.

> **Referências oficiais:** [Wildcard Capture](https://dev.java/learn/generics/wildcard-capture/), [JLS §4.10 — Subtyping](https://docs.oracle.com/javase/specs/jls/se25/html/jls-4.html#jls-4.10)

---

## Parte 18 — Tratamento de Exceções

[⬆️ Voltar ao Sumário](#sumário)

Exceções representam conclusão abrupta. O contrato profissional não é apenas “capturar”: é escolher categoria, preservar causa, fechar recursos, adicionar contexto e decidir em qual fronteira a recuperação é possível.

---

### 18.1 Checked e unchecked exceptions

[⬆️ Voltar ao Sumário](#sumário)

| Categoria | Base | Regra do compilador | Exemplos |
|---|---|---|---|
| checked | `Exception`, exceto `RuntimeException` | capturar ou declarar | `IOException`, `SQLException` |
| unchecked | `RuntimeException` | declaração não obrigatória | `IllegalArgumentException`, `NullPointerException` |
| erro da JVM/ambiente | `Error` | normalmente não recuperar localmente | `OutOfMemoryError`, `StackOverflowError` |

```java
static String ler(java.nio.file.Path caminho) throws java.io.IOException {
    return java.nio.file.Files.readString(caminho);
}
```

Checked exception torna a possibilidade visível no tipo do método. Unchecked costuma indicar violação de pré-condição, estado inválido ou falha que não pode ser tratada significativamente em toda chamada. A categoria não mede severidade.

Não capture `Throwable`: isso inclui erros e mecanismos de controle que a camada raramente sabe recuperar.

> **Referências oficiais:** [JLS §11.1 — Kinds and Causes of Exceptions](https://docs.oracle.com/javase/specs/jls/se25/html/jls-11.html#jls-11.1), [Exceptions](https://dev.java/learn/exceptions/)

---

### 18.2 `try`, `catch`, `finally` e multi-catch

[⬆️ Voltar ao Sumário](#sumário)

```java
static java.util.Properties carregar(java.nio.file.Path caminho) {
    var propriedades = new java.util.Properties();

    try (var entrada = java.nio.file.Files.newInputStream(caminho)) {
        propriedades.load(entrada);
        return propriedades;
    } catch (java.nio.file.NoSuchFileException e) {
        throw new IllegalStateException("Configuração ausente: " + caminho, e);
    } catch (java.io.IOException | SecurityException e) {
        throw new IllegalStateException("Não foi possível ler " + caminho, e);
    }
}
```

**Leitura guiada:** o catch específico vem antes do amplo; multi-catch compartilha tratamento entre tipos que não possuem relação de subtipo na alternativa; a causa original é preservada. O recurso fecha antes de o catch executar.

`finally` executa na conclusão normal ou abrupta do try/catch, salvo encerramentos extraordinários da JVM. Nunca retorne de `finally`: isso pode suprimir retorno ou exceção anterior.

> **Referência oficial:** [JLS §14.20 — The `try` Statement](https://docs.oracle.com/javase/specs/jls/se25/html/jls-14.html#jls-14.20)

---

### 18.3 Exceções customizadas

[⬆️ Voltar ao Sumário](#sumário)

```java
public final class LimiteCreditoExcedidoException extends RuntimeException {
    private final long limite;
    private final long solicitado;

    public LimiteCreditoExcedidoException(long limite, long solicitado) {
        super("Solicitado " + solicitado + ", limite " + limite);
        this.limite = limite;
        this.solicitado = solicitado;
    }

    public long limite() { return limite; }
    public long solicitado() { return solicitado; }
}
```

Crie tipo próprio quando consumidores precisam distinguir a falha, capturar dados estruturados ou mapear para protocolo. Nome, mensagem, causa e campos devem preservar contexto sem incluir segredo.

Escolher checked versus unchecked é decisão de API: o chamador consegue e deve recuperar ali? Não crie hierarquia profunda sem necessidade.

> **Referência oficial:** [JLS §11 — Exceptions](https://docs.oracle.com/javase/specs/jls/se25/html/jls-11.html)

---

### 18.4 Hierarquia e exceções comuns

[⬆️ Voltar ao Sumário](#sumário)

| Exceção | Significado típico |
|---|---|
| `IllegalArgumentException` | argumento não atende ao contrato |
| `IllegalStateException` | objeto/serviço não pode executar no estado atual |
| `NullPointerException` | referência nula usada onde não é aceita |
| `IndexOutOfBoundsException` | índice fora do intervalo |
| `NoSuchElementException` | elemento esperado não existe |
| `UnsupportedOperationException` | operação opcional não suportada |
| `ArithmeticException` | falha aritmética inteira/decimal específica |
| `CompletionException` / `ExecutionException` | falha encapsulada em tarefa |

Use a exceção padrão quando ela já comunica o contrato. `Objects.requireNonNull`, `Objects.checkIndex` e métodos `Math.*Exact` produzem diagnósticos padronizados.

> **Referência oficial:** [Package `java.lang`](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/lang/package-summary.html)

---

### 18.5 Práticas de desenho e propagação

[⬆️ Voltar ao Sumário](#sumário)

- capture apenas o que pode tratar, traduzir ou enriquecer;
- preserve a causa ao traduzir (`new MinhaException(..., causa)`);
- não use exceção para fluxo esperado de alta frequência quando uma consulta resolve;
- não registre e relance em toda camada, evitando logs duplicados;
- restaure interrupção quando não propagar `InterruptedException`;
- documente checked e unchecked relevantes em APIs públicas;
- nunca inclua tokens, senhas ou dados sensíveis em mensagens.

```java
try {
    cliente.enviar(requisicao);
} catch (java.io.IOException e) {
    throw new IntegracaoException("Falha ao enviar pedido " + pedidoId, e);
}
```

Traduza na fronteira entre abstrações: a camada de domínio não precisa expor detalhes de socket, mas o diagnóstico deve conservar a causa.

> **Referências oficiais:** [Throwable](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/lang/Throwable.html), [Javadoc `@throws`](https://docs.oracle.com/en/java/javase/25/javadoc/javadoc.html)

---

## Parte 19 — Annotations e Metadados

[⬆️ Voltar ao Sumário](#sumário)

Annotations anexam metadados a declarações ou usos de tipo. Elas não executam lógica por si: compilador, annotation processor, runtime, framework ou ferramenta precisa interpretá-las.

---

### 19.1 Annotations predefinidas

[⬆️ Voltar ao Sumário](#sumário)

| Annotation | Propósito |
|---|---|
| `@Override` | verificar sobrescrita |
| `@Deprecated` | marcar API desaconselhada |
| `@SuppressWarnings` | suprimir categorias específicas no menor escopo |
| `@SafeVarargs` | afirmar segurança de varargs genérico em declarações permitidas |
| `@FunctionalInterface` | verificar contrato SAM |

```java
/** @deprecated Use {@link #buscar(long)}. */
@Deprecated(since = "3.0", forRemoval = true)
public Cliente localizar(long id) {
    return buscar(id);
}
```

`@Deprecated` deve vir com documentação de migração. `forRemoval=true` informa intenção, não remove automaticamente. Suprimir warning sem entender a causa pode esconder heap pollution ou API insegura.

> **Referências oficiais:** [Package `java.lang`](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/lang/package-summary.html), [JLS §9.6.4 — Predefined Annotation Types](https://docs.oracle.com/javase/specs/jls/se25/html/jls-9.html#jls-9.6.4)

---

### 19.2 Criando annotations

[⬆️ Voltar ao Sumário](#sumário)

```java
@java.lang.annotation.Retention(java.lang.annotation.RetentionPolicy.RUNTIME)
@java.lang.annotation.Target(java.lang.annotation.ElementType.METHOD)
public @interface Auditavel {
    String acao();
    Nivel nivel() default Nivel.NORMAL;

    enum Nivel { NORMAL, SENSIVEL }
}

class PedidoService {
    @Auditavel(acao = "cancelar", nivel = Auditavel.Nivel.SENSIVEL)
    void cancelar(long pedidoId) { }
}
```

Elementos de annotation possuem tipos restritos: primitivos, `String`, `Class`, enum, annotation e arrays desses tipos. Valores padrão precisam ser constantes compatíveis. Annotation não deve carregar objeto arbitrário.

> **Referência oficial:** [JLS §9.6 — Annotation Interfaces](https://docs.oracle.com/javase/specs/jls/se25/html/jls-9.html#jls-9.6)

---

### 19.3 Retention, Target, Repeatable e Inherited

[⬆️ Voltar ao Sumário](#sumário)

| Meta-annotation | Pergunta respondida |
|---|---|
| `@Retention` | fica apenas no fonte, class file ou runtime? |
| `@Target` | pode anotar tipo, método, campo, parâmetro ou uso de tipo? |
| `@Repeatable` | pode aparecer várias vezes no mesmo alvo? |
| `@Inherited` | subclasses consultadas herdam annotation de classe? |
| `@Documented` | aparece no Javadoc gerado? |

`@Inherited` só afeta consulta de annotations em classes; não se propaga da mesma forma para interfaces, membros ou parâmetros. Type-use annotations podem marcar qualquer uso de tipo, mas a semântica depende da ferramenta que as interpreta.

> **Referência oficial:** [Package `java.lang.annotation`](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/lang/annotation/package-summary.html)

---

### 19.4 Annotation processing

[⬆️ Voltar ao Sumário](#sumário)

Annotation processors rodam durante compilação, analisam elementos/modelos de tipos e podem gerar novos fontes, classes ou recursos. Eles não devem alterar arbitrariamente o AST por contrato padrão.

```java
@javax.annotation.processing.SupportedAnnotationTypes("com.exemplo.Auditavel")
@javax.annotation.processing.SupportedSourceVersion(javax.lang.model.SourceVersion.RELEASE_25)
public final class AuditavelProcessor
        extends javax.annotation.processing.AbstractProcessor {

    @Override
    public boolean process(
            java.util.Set<? extends javax.lang.model.element.TypeElement> annotations,
            javax.annotation.processing.RoundEnvironment roundEnv) {
        // validar elementos e gerar arquivos por Filer
        return false;
    }
}
```

Processors participam do build e podem afetar tempo, incrementalidade e segurança da cadeia. Fixe versões e trate código gerado como artefato reproduzível, não como mágica da IDE.

> **Referências oficiais:** [Package `javax.annotation.processing`](https://docs.oracle.com/en/java/javase/25/docs/api/java.compiler/javax/annotation/processing/package-summary.html), [`javac` annotation processing](https://docs.oracle.com/en/java/javase/25/docs/specs/man/javac.html)

---

## Parte 20 — Referências, Memória e Recursos

[⬆️ Voltar ao Sumário](#sumário)

GC administra memória gerenciada, não fecha deterministicamente arquivo, socket, transação ou memória externa. Esta parte separa ausência, alcançabilidade e ownership.

---

### 20.1 `Optional` e ausência explícita

[⬆️ Voltar ao Sumário](#sumário)

```java
static java.util.Optional<Cliente> buscar(long id) {
    Cliente encontrado = consultarBanco(id);
    return java.util.Optional.ofNullable(encontrado);
}

String nome = buscar(42)
        .map(Cliente::nome)
        .filter(n -> !n.isBlank())
        .orElse("Desconhecido");
```

`Optional<T>` é contêiner de zero ou um valor, projetado principalmente para retorno. `orElse` avalia o fallback imediatamente; `orElseGet` recebe supplier lazy. `get` sem teste costuma ser pior que `orElseThrow` com contrato explícito.

Optional é value-based: não compare identidade, não sincronize nele e não conte com serialização. Evite `Optional` como campo, parâmetro ou elemento de coleção sem razão clara; `OptionalInt`/`Long`/`Double` evitam boxing.

> **Referência oficial:** [Optional](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/util/Optional.html)

---

### 20.2 `WeakReference`, `SoftReference` e `PhantomReference`

[⬆️ Voltar ao Sumário](#sumário)

```java
var fila = new java.lang.ref.ReferenceQueue<Object>();
Object alvo = new Object();
var fraca = new java.lang.ref.WeakReference<>(alvo, fila);

alvo = null;
Object aindaVivo = fraca.get();
if (aindaVivo != null) {
    System.out.println(aindaVivo);
}
```

Weak e soft references podem ser limpas pelo GC; phantom reference sempre devolve `null` em `get` e funciona com `ReferenceQueue` para coordenação pós-morte. `Cleaner` oferece abstração baseada em phantom references, mas é fallback: prefira `AutoCloseable` e try-with-resources.

Soft references não são cache com SLA previsível. Use cache com limites, expiração e métricas quando comportamento operacional importa.

> **Referências oficiais:** [`java.lang.ref`](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/lang/ref/package-summary.html), [Cleaner](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/lang/ref/Cleaner.html)

---

### 20.3 `ByteBuffer`, `MemorySegment` e `Arena`

[⬆️ Voltar ao Sumário](#sumário)

```java
var buffer = java.nio.ByteBuffer.allocate(16)
        .order(java.nio.ByteOrder.LITTLE_ENDIAN);
buffer.putInt(42).putLong(99L);
buffer.flip();
System.out.println(buffer.getInt());
```

`ByteBuffer` possui posição, limite e capacidade. `flip` muda de escrita para leitura; esquecer essa transição é fonte clássica de bugs. Buffer direto pode usar memória externa e não oferece fechamento determinístico público.

```java
try (var arena = java.lang.foreign.Arena.ofConfined()) {
    var segmento = arena.allocate(java.lang.foreign.ValueLayout.JAVA_INT);
    segmento.set(java.lang.foreign.ValueLayout.JAVA_INT, 0, 42);
    System.out.println(segmento.get(java.lang.foreign.ValueLayout.JAVA_INT, 0));
}
```

`MemorySegment` modela região com tamanho, lifetime e regras de acesso; `Arena` controla alocação e liberação. Acesso fora do limite, após fechamento ou pela thread errada em arena confinada falha de forma definida, em vez de corrupção silenciosa.

> **Referências oficiais:** [ByteBuffer](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/nio/ByteBuffer.html), [Foreign Function and Memory API](https://docs.oracle.com/en/java/javase/25/core/foreign-function-and-memory-api.html)

---

### 20.4 Garbage Collector e alcançabilidade

[⬆️ Voltar ao Sumário](#sumário)

Um objeto torna-se elegível quando deixa de ser alcançável por caminhos considerados pelo runtime. Coleta e finalização de memória não acontecem em instante previsível.

```text
GC roots
  ├── variáveis ativas de threads
  ├── campos estáticos alcançáveis
  ├── referências internas da JVM/JNI
  └── objetos alcançados transitivamente
```

Java possui coletores com objetivos diferentes. Heap máximo, latência, throughput e footprint são decisões operacionais; não escolha flags copiadas sem observar logs de GC, JFR e carga real. `System.gc()` é uma solicitação e pode ser desabilitada/ignorada.

Finalization foi deprecated for removal; não use `finalize`. Para recurso gerenciado, `AutoCloseable`; para salvaguarda, `Cleaner` cuidadosamente.

> **Referências oficiais:** [Java Garbage Collection](https://dev.java/learn/jvm/tool/garbage-collection/), [Object.finalize](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/lang/Object.html#finalize())

---

### 20.5 Ownership e fechamento determinístico

[⬆️ Voltar ao Sumário](#sumário)

| Recurso | Encerramento esperado |
|---|---|
| stream/reader/writer | `close` / try-with-resources |
| JDBC connection/statement/result set | `close` em ordem de escopo |
| `ExecutorService` criado localmente | fechamento/shutdown |
| `Arena` confinada/compartilhada | `close` pelo dono |
| objeto in-memory comum | GC; não existe `delete` |

O dono é quem cria ou recebe transferência explícita. Retornar stream aberto transfere responsabilidade e precisa ser documentado; consumir callback dentro do método pode manter ownership interno.

Fechar cedo demais causa uso após fechamento; não fechar causa vazamento de descritores, conexões ou memória externa. Estruture escopos para que ownership seja visualmente óbvio.

> **Referências oficiais:** [AutoCloseable](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/lang/AutoCloseable.html), [JLS §14.20.3](https://docs.oracle.com/javase/specs/jls/se25/html/jls-14.html#jls-14.20.3)

---

## Parte 21 — Threads e Concorrência

[⬆️ Voltar ao Sumário](#sumário)

Concorrência exige coordenação de visibilidade, atomicidade, ordem e lifetime. “Funcionou no teste” não prova ausência de data race; a Java Memory Model define quais observações são permitidas.

---

### 21.1 Platform threads e virtual threads

[⬆️ Voltar ao Sumário](#sumário)

```java
Thread plataforma = Thread.ofPlatform().name("worker").start(() -> executar());
Thread virtual = Thread.ofVirtual().name("request-1").start(() -> executar());

plataforma.join();
virtual.join();
```

Platform thread normalmente corresponde a thread agendada pelo sistema operacional. Virtual thread é agendada pelo runtime sobre carriers e custa muito menos, mas continua sendo `Thread` com stack, interrupção e ThreadLocal.

`start()` agenda a thread para executar `run()` concorrentemente e cada instância só pode ser iniciada uma vez. Chamar `run()` diretamente é apenas uma chamada de método na thread atual; em virtual thread, a própria API informa que a invocação direta não executa a tarefa. Prefira builders, factories e executors a subclasses de `Thread` na aplicação comum.

Use virtual threads para alta concorrência bloqueante; paralelismo de CPU continua limitado por cores. Nunca use `Thread.stop`, `suspend` ou `resume`, APIs inseguras/deprecated.

> **Referências oficiais:** [Thread](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/lang/Thread.html), [Virtual Threads](https://docs.oracle.com/en/java/javase/25/core/virtual-threads.html)

---

### 21.2 Java Memory Model e happens-before

[⬆️ Voltar ao Sumário](#sumário)

Happens-before garante visibilidade e ordem entre ações. Relações importantes:

- desbloquear um monitor happens-before de bloqueio posterior no mesmo monitor;
- escrever em `volatile` happens-before de leitura posterior desse campo;
- ações antes de `Thread.start` são visíveis à thread iniciada;
- ações da thread terminada são visíveis após `join` bem-sucedido;
- inicialização correta de campos `final` recebe garantias especiais.

```java
final class Publicacao {
    private int dado;
    private volatile boolean pronto;

    void publicar() {
        dado = 42;
        pronto = true;
    }

    int ler() {
        return pronto ? dado : -1;
    }
}
```

A escrita volatile publica a escrita anterior de `dado` para a thread que lê `pronto` como true. Isso não torna sequências arbitrárias atômicas.

> **Referência oficial:** [JLS §17.4.5 — Happens-before Order](https://docs.oracle.com/javase/specs/jls/se25/html/jls-17.html#jls-17.4.5)

---

### 21.3 Monitores, locks e conditions

[⬆️ Voltar ao Sumário](#sumário)

```java
final class ContadorSeguro {
    private long valor;

    synchronized void incrementar() { valor++; }
    synchronized long valor() { return valor; }
}
```

Cada objeto possui monitor. Método `synchronized` de instância usa `this`; estático usa o objeto `Class`. Proteja todo o estado que forma a invariável com o mesmo protocolo e mantenha a região crítica curta.

`ReentrantLock` oferece tentativa, interrupção, fairness opcional e múltiplas `Condition`. Sempre desbloqueie em `finally`:

```java
lock.lock();
try {
    alterarEstado();
} finally {
    lock.unlock();
}
```

Não bloqueie em string internada ou objeto público que código externo possa reutilizar.

> **Referências oficiais:** [JLS §14.19 — The `synchronized` Statement](https://docs.oracle.com/javase/specs/jls/se25/html/jls-14.html#jls-14.19), [Lock](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/util/concurrent/locks/Lock.html)

---

### 21.4 Atomics, immutability e thread confinement

[⬆️ Voltar ao Sumário](#sumário)

```java
final class Metricas {
    private final java.util.concurrent.atomic.LongAdder requisicoes =
            new java.util.concurrent.atomic.LongAdder();

    void registrar() { requisicoes.increment(); }
    long totalAproximado() { return requisicoes.sum(); }
}
```

Atomics oferecem operações indivisíveis sobre variáveis e efeitos de memória documentados. `AtomicLong` é adequado a valor único exato; `LongAdder` reduz contenção para estatística, mas `sum` não é snapshot atômico com updates concorrentes.

Imutabilidade e publicação segura reduzem sincronização. Thread confinement mantém estado mutável acessível a uma thread; transferir ownership por fila é frequentemente mais simples que compartilhar.

> **Referências oficiais:** [`java.util.concurrent.atomic`](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/util/concurrent/atomic/package-summary.html), [LongAdder](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/util/concurrent/atomic/LongAdder.html)

---

### 21.5 Coleções concorrentes e filas bloqueantes

[⬆️ Voltar ao Sumário](#sumário)

| Estrutura | Uso |
|---|---|
| `ConcurrentHashMap` | mapa concorrente com operações compostas específicas |
| `CopyOnWriteArrayList` | muitas leituras e poucas escritas; snapshot por iterator |
| `ConcurrentLinkedQueue` | fila não bloqueante |
| `ArrayBlockingQueue` | fila bloqueante limitada e backpressure |
| `LinkedBlockingQueue` | fila bloqueante opcionalmente limitada |

```java
var fila = new java.util.concurrent.ArrayBlockingQueue<String>(100);
fila.put("evento");          // aguarda espaço, respeita interrupção
String evento = fila.take(); // aguarda item
```

Capacidade limitada torna overload visível. Escolha entre bloquear, rejeitar, descartar, persistir ou desacelerar o produtor; fila ilimitada apenas adia o problema para memória e latência.

> **Referência oficial:** [`java.util.concurrent`](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/util/concurrent/package-summary.html)

---

## Parte 22 — Reflection, Serviços e Interoperabilidade

[⬆️ Voltar ao Sumário](#sumário)

Essas APIs atravessam fronteiras do sistema de tipos ou da JVM. São essenciais para frameworks e integração, mas pedem encapsulamento, validação e atenção a módulos, segurança e AOT.

---

### 22.1 Reflection

[⬆️ Voltar ao Sumário](#sumário)

```java
record Usuario(long id, String nome) { }

Class<?> tipo = Usuario.class;
System.out.println(tipo.isRecord());

for (var componente : tipo.getRecordComponents()) {
    System.out.println(componente.getName() + ": " + componente.getType());
}
```

Reflection inspeciona classes, campos, métodos, construtores, records, annotations e generics. `getDeclared*` vê declarações próprias; `get*` público segue regras diferentes de herança. Acesso profundo pode ser bloqueado por JPMS.

Não use `setAccessible(true)` como solução automática. Prefira API pública, `opens` estreito, method handles ou geração em compilação conforme o caso. Reflection acrescenta falhas de runtime e dificulta análise estática.

> **Referências oficiais:** [Package `java.lang.reflect`](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/lang/reflect/package-summary.html), [Reflection](https://dev.java/learn/reflection/)

---

### 22.2 Method handles

[⬆️ Voltar ao Sumário](#sumário)

```java
var lookup = java.lang.invoke.MethodHandles.lookup();
var tipo = java.lang.invoke.MethodType.methodType(int.class);
var length = lookup.findVirtual(String.class, "length", tipo);

int tamanho = (int) length.invokeExact("Java");
System.out.println(tamanho);
```

Method handle é referência tipada e diretamente executável a método/campo/construtor. `invokeExact` exige assinatura exata; `invoke` adapta conversões permitidas. Lookup carrega regras de acesso. A API sustenta lambdas, dynamic languages e frameworks de alto desempenho, mas não é substituta comum de chamada direta.

O exemplo precisa estar em método que declare ou trate `Throwable`, porque `invokeExact` pode lançar qualquer falha do alvo.

> **Referências oficiais:** [Package `java.lang.invoke`](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/lang/invoke/package-summary.html), [Method Handles](https://dev.java/learn/introduction_to_method_handles/)

---

### 22.3 `ServiceLoader` e Dependency Injection

[⬆️ Voltar ao Sumário](#sumário)

```java
public interface Exportador {
    String formato();
    byte[] exportar(Object valor);
}

var exportadores = java.util.ServiceLoader.load(Exportador.class);
for (Exportador exportador : exportadores) {
    System.out.println(exportador.formato());
}
```

No classpath, providers podem ser listados em `META-INF/services`; no JPMS, consumidor usa `uses` e provider usa `provides ... with`. `ServiceLoader` descobre implementações lazy e pode falhar por configuração inválida.

Dependency Injection é princípio de design, não recurso da linguagem. Construtor explícito costuma ser a base mais clara; containers externos acrescentam lifecycle, scopes, interceptors e descoberta.

> **Referências oficiais:** [ServiceLoader](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/util/ServiceLoader.html), [JLS §7.7.3–7.7.4](https://docs.oracle.com/javase/specs/jls/se25/html/jls-7.html#jls-7.7.3)

---

### 22.4 JNI

[⬆️ Voltar ao Sumário](#sumário)

```java
public final class NativeHash {
    static { System.loadLibrary("nativehash"); }
    public native long calcular(byte[] dados);
}
```

JNI conecta Java a C/C++. A declaração `native` não possui corpo; a biblioteca precisa exportar função com assinatura, calling convention e tipos compatíveis. Erro nativo pode corromper memória ou derrubar a JVM, ultrapassando as garantias usuais do Java.

Use `javac -h` para gerar headers. Encapsule JNI em módulo pequeno, valide tamanhos e lifetime, teste em cada OS/arquitetura e prefira API Java quando disponível.

> **Referências oficiais:** [JNI Specification](https://docs.oracle.com/en/java/javase/25/docs/specs/jni/), [`javac -h`](https://docs.oracle.com/en/java/javase/25/docs/specs/man/javac.html)

---

### 22.5 Foreign Function and Memory API

[⬆️ Voltar ao Sumário](#sumário)

FFM oferece acesso nativo com `Linker`, `SymbolLookup`, function descriptors, method handles, memory layouts e segments. É API permanente desde Java 22 e busca substituir grande parte do uso direto de JNI com modelo mais seguro e declarativo.

```java
var linker = java.lang.foreign.Linker.nativeLinker();
var strlen = linker.defaultLookup().find("strlen").orElseThrow();
var descriptor = java.lang.foreign.FunctionDescriptor.of(
        java.lang.foreign.ValueLayout.JAVA_LONG,
        java.lang.foreign.ValueLayout.ADDRESS);
var handle = linker.downcallHandle(strlen, descriptor);
```

O descritor precisa corresponder à ABI da plataforma; `long` de C não possui tamanho universal. Operações restritas podem emitir warnings e exigir opções de native access. Arena deve permanecer viva enquanto a chamada usa sua memória.

> **Referências oficiais:** [Foreign Function and Memory API](https://docs.oracle.com/en/java/javase/25/core/foreign-function-and-memory-api.html), [Package `java.lang.foreign`](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/lang/foreign/package-summary.html)

---

## Checkpoint — Fundamentos da Linguagem (Partes 1–22)

[⬆️ Voltar ao Sumário](#sumário)

| Você consegue explicar ou fazer? | Retome se necessário |
|---|---|
| separar Java, JVM, JDK, class file, JAR, package e módulo | Partes 1, 2 e 25 |
| prever cópia de primitivo, cópia de referência, boxing e `null` | Parte 3 |
| modelar tipo com acesso, invariantes, igualdade e imutabilidade | Partes 5, 6 e 11 |
| escolher interface, composição, herança ou hierarquia sealed | Parte 12 |
| escolher coleção e pipeline stream sem esconder custo | Partes 14 e 15 |
| compor tarefas com timeout, interrupção e executor apropriado | Partes 16 e 21 |
| distinguir checked, unchecked e fechamento de recursos | Partes 18 e 20 |
| reconhecer riscos de reflection, JNI e FFM | Parte 22 |

Se uma resposta for apenas sintaxe memorizada, volte ao exemplo e explique também o contrato, o custo e o caso de falha.

---

## Parte 23 — Java nos Principais Contextos de Aplicação

[⬆️ Voltar ao Sumário](#sumário)

A linguagem permanece Java, mas cada plataforma define lifecycle, APIs, empacotamento, reflexão, threads e versões suportadas. Conhecer Java SE não elimina a necessidade de ler a documentação do ambiente real.

---

### 23.1 Backend e serviços

[⬆️ Voltar ao Sumário](#sumário)

Backends Java costumam combinar HTTP, serialização, validação, persistência, mensageria e observabilidade. Java SE já fornece `HttpServer` simples, `HttpClient`, JDBC, concorrência, segurança e I/O; frameworks acrescentam servidor, roteamento, DI, configuração e convenções.

```java
var servidor = com.sun.net.httpserver.HttpServer.create(
        new java.net.InetSocketAddress(8080), 0);
servidor.createContext("/health", exchange -> {
    byte[] corpo = "ok".getBytes(java.nio.charset.StandardCharsets.UTF_8);
    exchange.sendResponseHeaders(200, corpo.length);
    try (var saida = exchange.getResponseBody()) {
        saida.write(corpo);
    }
});
servidor.start();
```

Esse servidor do JDK é útil para protótipos e ferramentas; não equivale automaticamente a um framework de produção. Defina limites de payload, timeouts, executor, TLS, shutdown e observabilidade.

> **Referências oficiais:** [Simple Web Server](https://docs.oracle.com/en/java/javase/25/docs/specs/man/jwebserver.html), [HttpServer](https://docs.oracle.com/en/java/javase/25/docs/api/jdk.httpserver/com/sun/net/httpserver/HttpServer.html)

---

### 23.2 Jakarta EE

[⬆️ Voltar ao Sumário](#sumário)

Jakarta EE é um conjunto de especificações acima do Java SE para aplicações empresariais. Inclui Jakarta REST, CDI, Persistence, Transactions, Validation, Servlet, Messaging e outras APIs implementadas por runtimes compatíveis.

```java
@jakarta.ws.rs.Path("/pedidos")
@jakarta.enterprise.context.ApplicationScoped
public class PedidoResource {
    @jakarta.ws.rs.GET
    @jakarta.ws.rs.Produces(jakarta.ws.rs.core.MediaType.APPLICATION_JSON)
    public java.util.List<PedidoDto> listar() {
        return java.util.List.of();
    }
}
```

As annotations não pertencem ao Java SE. O container interpreta lifecycle, escopo, injeção e transporte. Compile contra a versão da plataforma escolhida e não confunda namespace histórico `javax.*` com o atual `jakarta.*`.

> **Referências oficiais:** [Jakarta EE Platform](https://jakarta.ee/specifications/platform/), [Jakarta EE Tutorial](https://jakarta.ee/learn/docs/jakartaee-tutorial/current/)

---

### 23.3 Desktop com JavaFX

[⬆️ Voltar ao Sumário](#sumário)

JavaFX é projeto OpenJFX separado do JDK. Ele fornece scene graph, controles, CSS, FXML, mídia e propriedades observáveis. A UI possui JavaFX Application Thread; trabalho longo deve sair dela e atualizações visuais devem retornar de forma segura.

```java
public final class App extends javafx.application.Application {
    @Override
    public void start(javafx.stage.Stage palco) {
        var botao = new javafx.scene.control.Button("Olá");
        botao.setOnAction(evento -> botao.setText("Clicado"));
        palco.setScene(new javafx.scene.Scene(new javafx.scene.layout.StackPane(botao), 300, 120));
        palco.show();
    }
}
```

**Leitura guiada:** `Application` fornece o ciclo de vida básico; `start` recebe o palco principal; `Scene` contém a árvore visual e `setOnAction` registra uma lambda executada quando o botão é acionado. O exemplo altera somente a UI e termina rápido. Consulta a banco, rede ou processamento demorado deve ocorrer fora da JavaFX Application Thread, publicando de volta apenas a atualização visual.

> **Referência oficial do projeto:** [OpenJFX Documentation](https://openjfx.io/openjfx-docs/)

---

### 23.4 Android

[⬆️ Voltar ao Sumário](#sumário)

Android aceita Java como linguagem, mas executa sobre Android Runtime e expõe Android SDK, não toda a API Java SE de um JDK desktop/servidor. Versão da linguagem, desugaring, API level, lifecycle de componentes e main thread pertencem à plataforma Android.

```java
public final class MainActivity extends androidx.appcompat.app.AppCompatActivity {
    @Override
    protected void onCreate(android.os.Bundle estado) {
        super.onCreate(estado);
        setContentView(R.layout.activity_main);
    }
}
```

Não escolha API apenas porque aparece no JDK 25. Consulte compatibilidade do Android Gradle Plugin, desugaring e nível mínimo. Evite I/O na main thread e respeite recriação de activities/processo.

> **Referências oficiais:** [Build local unit tests](https://developer.android.com/training/testing/local-tests), [Java 8+ API desugaring](https://developer.android.com/studio/write/java8-support)

---

### 23.5 Cloud, containers e serverless

[⬆️ Voltar ao Sumário](#sumário)

HotSpot moderno reconhece limites de container nas configurações suportadas. Ainda assim, heap não é toda a memória: metaspace, code cache, stacks, buffers diretos, bibliotecas nativas e sidecars consomem orçamento.

Para cloud/serverless, meça:

- tempo de startup e warmup;
- memória residente, não apenas `-Xmx`;
- comportamento sob CPU throttling;
- shutdown gracioso e sinais;
- DNS, TLS, pools e timeouts;
- criação de imagem, SBOM e atualizações do JDK.

CDS/AppCDS, `jlink`, virtual threads e AOT podem ajudar cenários diferentes; não são uma única opção “cloud”.

> **Referências oficiais:** [Documentação do JDK 25](https://docs.oracle.com/en/java/javase/25/), [Class Data Sharing](https://docs.oracle.com/en/java/javase/25/vm/class-data-sharing.html)

---

### 23.6 Dados, mensageria e outros domínios

[⬆️ Voltar ao Sumário](#sumário)

Java aparece em processamento de dados, brokers, ferramentas de build, IDEs, automação, jogos e sistemas embarcados. Frequentemente a API central vem de projeto externo: Kafka, Hadoop, Spark, Eclipse Vert.x, libGDX e outros.

Antes de adotar, confirme:

1. JDKs suportados e política de release;
2. modelo de threads e backpressure;
3. garantias de entrega/consistência;
4. serialização e compatibilidade de schema;
5. licença, manutenção e resposta a vulnerabilidades.

Uma biblioteca Java não vira parte da linguagem por ser popular. O catálogo externo da Parte 29 aponta para a documentação primária de cada projeto.

---

## Parte 24 — Arquitetura de Aplicações Java

[⬆️ Voltar ao Sumário](#sumário)

Arquitetura organiza dependências, responsabilidades, dados e implantação. Java oferece packages, módulos, interfaces e tipos; nenhum deles escolhe automaticamente uma arquitetura correta.

---

### 24.1 Java não impõe arquitetura

[⬆️ Voltar ao Sumário](#sumário)

Os mesmos recursos suportam monólito modular, serviço, aplicação desktop ou biblioteca. Uma arquitetura precisa responder:

- onde ficam regras de negócio;
- quem depende de infraestrutura;
- como transações e falhas atravessam fronteiras;
- como módulos são testados e implantados;
- quais contratos podem evoluir independentemente.

JPMS oferece encapsulamento técnico, mas não identifica bounded contexts nem separa domínio automaticamente. Framework de DI monta objetos, mas não corrige dependências apontando na direção errada.

---

### 24.2 Arquitetura em camadas

[⬆️ Voltar ao Sumário](#sumário)

```text
presentation / API
        ↓
application services
        ↓
domain
        ↓
infrastructure / persistence
```

Camadas funcionam quando dependências e responsabilidades são claras. O risco é transformar cada operação em passagem mecânica por controller, service, manager, repository e DAO sem regra própria.

```java
public final class CriarPedidoUseCase {
    private final PedidoRepository repositorio;
    public CriarPedidoUseCase(PedidoRepository repositorio) {
        this.repositorio = repositorio;
    }
    public Pedido executar(CriarPedido comando) {
        var pedido = Pedido.criar(comando.clienteId(), comando.itens());
        repositorio.salvar(pedido);
        return pedido;
    }
}
```

**Leitura guiada:** o caso de uso recebe a abstração `PedidoRepository` pelo construtor, coordena a criação do agregado e pede sua persistência. A regra de criação permanece em `Pedido.criar`; o caso de uso não conhece SQL, HTTP nem framework. Essa divisão só é útil se as fronteiras forem reais: uma classe intermediária que apenas repassa chamadas sem decisão ou política acrescenta cerimônia, não arquitetura.

> **Referência oficial de plataforma:** [Jakarta EE Tutorial — Overview](https://jakarta.ee/learn/docs/jakartaee-tutorial/current/intro/overview/overview.html)

---

### 24.3 Clean Architecture, Hexagonal e Onion

[⬆️ Voltar ao Sumário](#sumário)

Essas famílias colocam regras centrais independentes de detalhes e representam integrações por portas/interfaces.

```text
driving adapter (HTTP/CLI)
          ↓
application port → domain ← repository port
                              ↑
                    driven adapter (JDBC)
```

```java
interface CarregarClientePort {
    java.util.Optional<Cliente> porId(long id);
}

final class ConsultarCliente {
    private final CarregarClientePort clientes;
    ConsultarCliente(CarregarClientePort clientes) { this.clientes = clientes; }
}
```

O benefício é substituir detalhes; o custo é mais conceitos e arquivos. Não crie interface para toda classe por ritual. Crie fronteira onde existem consumidores, variação, teste ou independência relevante.

> **Fonte primária:** [Alistair Cockburn — Hexagonal Architecture](https://alistair.cockburn.us/hexagonal-architecture/)

---

### 24.4 Domain-Driven Design

[⬆️ Voltar ao Sumário](#sumário)

DDD prioriza modelo e linguagem do domínio. Conceitos importantes:

| Conceito | Papel |
|---|---|
| entity | identidade ao longo do tempo |
| value object | significado por valor e imutabilidade desejável |
| aggregate | fronteira de consistência e invariantes |
| repository | coleção conceitual de aggregates |
| domain service | regra que não pertence naturalmente a uma entidade |
| bounded context | fronteira em que um modelo e linguagem são coerentes |

Records ajudam value objects simples; classes encapsulam aggregates; packages/módulos podem reforçar contextos. DDD não significa criar sufixos para cada classe nem evitar completamente ORM.

> **Fonte primária:** Eric Evans, *Domain-Driven Design: Tackling Complexity in the Heart of Software* (Addison-Wesley)

---

### 24.5 CQRS

[⬆️ Voltar ao Sumário](#sumário)

CQRS separa modelos de comando e consulta quando suas necessidades divergem.

```java
record CriarPedido(long clienteId, java.util.List<ItemPedido> itens) { }
record PedidoResumo(long id, String cliente, long totalCentavos) { }

interface CriarPedidoHandler { long handle(CriarPedido comando); }
interface PedidoQuery { java.util.Optional<PedidoResumo> porId(long id); }
```

Separação de classes não obriga bancos diferentes, mensageria ou consistência eventual. Comece simples; introduza modelos/armazenamentos separados apenas quando escala, segurança ou forma de consulta justificarem.

> **Fonte primária:** [Martin Fowler — CQRS](https://martinfowler.com/bliki/CQRS.html)

---

### 24.6 Event-Driven Architecture

[⬆️ Voltar ao Sumário](#sumário)

Evento descreve fato passado e imutável. Consumidores podem receber novamente, fora de ordem ou depois de atraso, conforme broker e contrato.

```java
public record PedidoCriado(
        java.util.UUID eventId,
        long pedidoId,
        java.time.Instant ocorridoEm,
        int versao) { }
```

Projete idempotência, schema evolution, particionamento, retry, dead letter, observabilidade e transação entre banco e publicação. O padrão Outbox registra estado e evento na mesma transação local e publica posteriormente.

Evento interno em memória não possui as mesmas garantias de mensagem distribuída.

---

### 24.7 Microsserviços

[⬆️ Voltar ao Sumário](#sumário)

Microsserviço é fronteira organizacional e operacional, não classe pequena. Cada serviço adiciona rede, deploy, observabilidade, autenticação, consistência distribuída e resposta a falhas parciais.

Prefira monólito modular quando:

- domínio e equipe ainda mudam rapidamente;
- deploy conjunto é aceitável;
- transações locais simplificam regras;
- custo operacional de distribuição não se paga.

Distribua quando autonomia, isolamento, escala independente ou fronteiras maduras justificarem. Virtual threads facilitam concorrência de I/O, mas não eliminam latência e falhas da rede.

> **Referência oficial de especificações cloud-native:** [Eclipse MicroProfile](https://microprofile.io/)

---

### 24.8 Padrões enterprise clássicos

[⬆️ Voltar ao Sumário](#sumário)

| Padrão | Problema |
|---|---|
| Repository | isolar acesso e coleção de aggregates |
| Unit of Work | coordenar mudanças e transação |
| Data Mapper | separar objeto de persistência |
| Transaction Script | organizar caso de uso procedural simples |
| Service Layer | definir fronteira de operações da aplicação |
| Identity Map | uma instância por identidade no contexto |
| Optimistic Lock | detectar atualização concorrente por versão |

ORMs implementam alguns internamente. Duplicá-los por cima sem conhecer lifecycle pode criar caches e unidades de trabalho conflitantes.

> **Fonte primária:** Martin Fowler, *Patterns of Enterprise Application Architecture* (Addison-Wesley)

---

## Parte 25 — JDK, Projetos, Dependências e Qualidade

[⬆️ Voltar ao Sumário](#sumário)

Engenharia Java inclui transformar fonte em artefato reproduzível. O JDK fornece ferramentas; Maven e Gradle acrescentam modelo de projeto, resolução de dependências e plugins.

---

### 25.1 Ferramentas do JDK

[⬆️ Voltar ao Sumário](#sumário)

| Ferramenta | Função |
|---|---|
| `java` | iniciar classe, JAR, módulo ou arquivo-fonte |
| `javac` | compilar fontes |
| `jar` | criar/listar/extrair JAR |
| `javadoc` | gerar documentação de API |
| `jshell` | REPL da linguagem |
| `jdeps` | analisar dependências |
| `javap` | inspecionar class file/bytecode |
| `jlink` | criar runtime image modular |
| `jpackage` | criar pacote/instalador nativo |
| `jcmd`, `jfr`, `jstack`, `jmap` | diagnóstico e observação |

```powershell
java --version
javac --version
javac -d out src/com/exemplo/Main.java
java -cp out com.exemplo.Main
javap -c -p out/com/exemplo/Main.class
```

> **Referência oficial:** [JDK Tool Specifications](https://docs.oracle.com/en/java/javase/25/docs/specs/man/)

---

### 25.2 Fonte, class files, classpath e module path

[⬆️ Voltar ao Sumário](#sumário)

```text
src/main/java      fonte de produção (convenção de build)
src/test/java      fonte de teste
target/classes     saída Maven típica
build/classes      saída Gradle típica
```

Classpath é sequência de diretórios/JARs em que classes são procuradas e pertencem ao unnamed module. Module path contém módulos explícitos ou automáticos e participa da resolução JPMS.

```powershell
javac -cp "lib/*" -d out src/com/exemplo/Main.java
java -cp "out;lib/*" com.exemplo.Main       # separador Windows

javac --module-path mods -d out --module-source-path src -m com.exemplo.app
java --module-path out;mods -m com.exemplo.app/com.exemplo.Main
```

Não dependa da ordem acidental de classes duplicadas no classpath. Conflitos de versão podem compilar e falhar em runtime com `NoSuchMethodError` ou `ClassNotFoundException`.

> **Referências oficiais:** [`javac`](https://docs.oracle.com/en/java/javase/25/docs/specs/man/javac.html), [`java`](https://docs.oracle.com/en/java/javase/25/docs/specs/man/java.html)

---

### 25.3 JARs, Maven e Gradle

[⬆️ Voltar ao Sumário](#sumário)

JAR é formato do JDK; Maven e Gradle são ferramentas externas. Um projeto Maven básico:

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0">
  <modelVersion>4.0.0</modelVersion>
  <groupId>com.exemplo</groupId>
  <artifactId>pedidos</artifactId>
  <version>1.0.0</version>
  <properties>
    <maven.compiler.release>25</maven.compiler.release>
  </properties>
</project>
```

Um build Gradle pode configurar Java toolchain:

```groovy
java {
    toolchain {
        languageVersion = JavaLanguageVersion.of(25)
    }
}
```

Fixe wrapper e plugins, revise grafo transitivo, use repositórios confiáveis e trate lock/verificação de dependências conforme a ferramenta. `groupId:artifactId:version` identifica artefato, não package Java.

> **Fontes oficiais dos projetos:** [Apache Maven](https://maven.apache.org/guides/), [Gradle Java Toolchains](https://docs.gradle.org/current/userguide/toolchains.html), [JAR Specification](https://docs.oracle.com/en/java/javase/25/docs/specs/jar/jar.html)

---

### 25.4 Build, testes, empacotamento e execução

[⬆️ Voltar ao Sumário](#sumário)

| Etapa | Resultado |
|---|---|
| compile | class files de produção |
| test compile | classes de teste |
| test | resultados pelo test engine |
| package | JAR/WAR/distribuição |
| verify | verificações adicionais do lifecycle |
| deploy/publish | disponibilização em repositório/ambiente |

```powershell
./mvnw verify
./gradlew build
jar --describe-module --file target/pedidos.jar
```

Teste o mesmo artefato que será entregue. Fat/uber JAR, thin JAR, WAR, runtime image e container image possuem resolução e startup diferentes. Builds reproduzíveis precisam declarar JDK, encoding, locale relevante, plugins e dependências.

> **Fontes oficiais:** [Maven Lifecycle](https://maven.apache.org/guides/introduction/introduction-to-the-lifecycle.html), [Gradle Java Plugin](https://docs.gradle.org/current/userguide/java_plugin.html)

---

### 25.5 Javadoc, lint e análise estática

[⬆️ Voltar ao Sumário](#sumário)

```java
/**
 * Calcula o total após desconto.
 *
 * @param subtotal valor não negativo em centavos
 * @param percentual número entre 0 e 100
 * @return total com desconto
 * @throws IllegalArgumentException se um argumento estiver fora do contrato
 */
public static long aplicarDesconto(long subtotal, int percentual) { ... }
```

```powershell
javac -Xlint:all -Werror -d out src/com/exemplo/*.java
javadoc -d docs -sourcepath src -subpackages com.exemplo
```

`-Xlint` revela raw types, unchecked, serialização, deprecation e outros riscos. `-Werror` exige política de baseline e upgrades; não suprima globalmente. Ferramentas externas como Error Prone, SpotBugs e Checkstyle têm contratos próprios.

> **Referências oficiais:** [`javac -Xlint`](https://docs.oracle.com/en/java/javase/25/docs/specs/man/javac.html), [Javadoc Guide](https://docs.oracle.com/en/java/javase/25/javadoc/)

---

### 25.6 Compatibilidade de versões e toolchains

[⬆️ Voltar ao Sumário](#sumário)

`--source` controla sintaxe; `--target` controla versão do class file; `--release` combina linguagem, class file e APIs documentadas da plataforma-alvo. Prefira `--release`.

```powershell
javac --release 21 -d out src/com/exemplo/Main.java
```

Compilar com JDK 25 para `--release 21` não torna dependência externa compatível nem testa comportamento de outra JVM. CI deve executar testes na matriz de runtimes suportados quando necessário.

Class file mais novo causa `UnsupportedClassVersionError` em runtime antigo. Multi-release JAR permite variantes em `META-INF/versions`, mas aumenta custo de teste e manutenção.

> **Referências oficiais:** [`javac --release`](https://docs.oracle.com/en/java/javase/25/docs/specs/man/javac.html), [JAR Multi-Release](https://docs.oracle.com/en/java/javase/25/docs/specs/jar/jar.html#multi-release-jar-files)

---

## Parte 26 — I/O, Serialização, HTTP e Internacionalização

[⬆️ Voltar ao Sumário](#sumário)

Dados externos exigem limites, charset, formato, timeout e ownership explícitos. Não presuma que arquivo cabe em memória, texto é UTF-8 ou rede responderá.

---

### 26.1 Paths, Files, streams e channels

[⬆️ Voltar ao Sumário](#sumário)

```java
var base = java.nio.file.Path.of("dados").toAbsolutePath().normalize();
var arquivo = base.resolve("entrada.txt").normalize();

if (!arquivo.startsWith(base)) throw new SecurityException("path traversal");

try (var entrada = java.nio.file.Files.newInputStream(arquivo);
     var saida = java.nio.file.Files.newOutputStream(
             base.resolve("copia.txt"),
             java.nio.file.StandardOpenOption.CREATE,
             java.nio.file.StandardOpenOption.TRUNCATE_EXISTING)) {
    entrada.transferTo(saida);
}
```

`Path` é representação; `Files` executa operações; streams processam bytes; channels suportam buffers, posicionamento e I/O avançado. Normalizar caminho é apenas parte da defesa: symlinks e TOCTOU exigem modelo de ameaça e APIs apropriadas.

> **Referências oficiais:** [Package `java.nio.file`](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/nio/file/package-summary.html), [InputStream](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/io/InputStream.html)

---

### 26.2 Readers, writers, charsets e buffers

[⬆️ Voltar ao Sumário](#sumário)

```java
var caminho = java.nio.file.Path.of("nomes.txt");

try (var writer = java.nio.file.Files.newBufferedWriter(
        caminho, java.nio.charset.StandardCharsets.UTF_8)) {
    writer.write("Ana");
    writer.newLine();
}
```

`InputStream`/`OutputStream` trabalham com bytes; `Reader`/`Writer`, com chars. A conversão exige charset. Declare `UTF_8` para formatos definidos; charset default depende do ambiente, embora seja UTF-8 por padrão em JDKs modernos segundo JEP 400, com exceções/configurações documentadas.

Buffer reduz chamadas ao sistema. `flush` envia dados acumulados ao próximo nível, mas não equivale necessariamente a persistir fisicamente em disco.

> **Referências oficiais:** [Charset](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/nio/charset/Charset.html), [JEP 400 — UTF-8 by Default](https://openjdk.org/jeps/400)

---

### 26.3 Serialização e JSON

[⬆️ Voltar ao Sumário](#sumário)

Java Object Serialization é mecanismo histórico e perigoso para dados não confiáveis. Preferir formatos com schema/contrato explícito reduz acoplamento à implementação.

```java
var filtro = java.io.ObjectInputFilter.Config.createFilter(
        "maxdepth=10;maxrefs=1000;java.base/*;com.exemplo.dto.*;!*" );

try (var entrada = new java.io.ObjectInputStream(
        java.nio.file.Files.newInputStream(java.nio.file.Path.of("dados.bin")))) {
    entrada.setObjectInputFilter(filtro);
    Object valor = entrada.readObject();
}
```

Java SE não inclui uma API JSON geral equivalente a Jackson. Jakarta JSON Processing/Binding e bibliotecas externas entram como dependências. Defina nomes, números, datas, `null`, membros desconhecidos, polimorfismo e evolução antes de publicar o contrato.

> **Referências oficiais:** [Serialization Filtering](https://docs.oracle.com/en/java/javase/25/core/serialization-filtering1.html), [Object Serialization Specification](https://docs.oracle.com/en/java/javase/25/docs/specs/serialization/), [Jakarta JSON Processing](https://jakarta.ee/specifications/jsonp/)

---

### 26.4 HTTP Client

[⬆️ Voltar ao Sumário](#sumário)

```java
var cliente = java.net.http.HttpClient.newBuilder()
        .connectTimeout(java.time.Duration.ofSeconds(5))
        .followRedirects(java.net.http.HttpClient.Redirect.NORMAL)
        .build();

var requisicao = java.net.http.HttpRequest.newBuilder(
        java.net.URI.create("https://example.com/api/status"))
        .timeout(java.time.Duration.ofSeconds(10))
        .header("Accept", "application/json")
        .GET()
        .build();

var resposta = cliente.send(requisicao,
        java.net.http.HttpResponse.BodyHandlers.ofString(
                java.nio.charset.StandardCharsets.UTF_8));

if (resposta.statusCode() / 100 != 2) {
    throw new IllegalStateException("HTTP " + resposta.statusCode());
}
```

Cliente é reutilizável e gerencia conexões. Timeout de conexão e de requisição são diferentes. `sendAsync` devolve `CompletableFuture`; o body handler decide streaming versus materialização. Limite resposta e valide URI/redirecionamento quando entrada é externa.

> **Referência oficial:** [Java HTTP Client](https://docs.oracle.com/en/java/javase/25/docs/api/java.net.http/java/net/http/HttpClient.html)

---

### 26.5 Datas, tempo, locale e fusos

[⬆️ Voltar ao Sumário](#sumário)

| Tipo | Representa |
|---|---|
| `Instant` | instante UTC na timeline |
| `LocalDate` | data civil sem hora/fuso |
| `LocalTime` | hora civil sem data/fuso |
| `LocalDateTime` | data e hora sem fuso |
| `ZonedDateTime` | data/hora com zone rules |
| `OffsetDateTime` | data/hora com offset fixo observado |
| `Duration` | quantidade baseada em segundos/nanos |
| `Period` | quantidade baseada em anos/meses/dias |

```java
var zona = java.time.ZoneId.of("America/Sao_Paulo");
var instante = java.time.Instant.now();
var local = instante.atZone(zona);

var formato = java.time.format.DateTimeFormatter
        .ofPattern("dd/MM/uuuu HH:mm", new java.util.Locale("pt", "BR"));
System.out.println(formato.format(local));
```

Regra “todo dia às 9h em São Paulo” precisa preservar horário civil e `ZoneId`, não apenas offset atual. Transições podem criar horário inexistente ou ambíguo. Use `Clock` injetável em testes.

> **Referências oficiais:** [Package `java.time`](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/time/package-summary.html), [Date Time API](https://dev.java/learn/date-time/)

---

### 26.6 Expressões regulares e limites

[⬆️ Voltar ao Sumário](#sumário)

```java
var padrao = java.util.regex.Pattern.compile("^[A-Z]{2}-\\d{4}$");
boolean valido = padrao.matcher("SP-1234").matches();
```

Compile padrões reutilizados. `matches()` exige a entrada inteira; `find()` procura ocorrência. Escape Java e escape regex são camadas diferentes (`"\\d"` no fonte representa `\d` no padrão).

`java.util.regex` usa engine de backtracking e não oferece timeout por matcher. Para entrada hostil, limite tamanho, evite padrões catastróficos, use possessive quantifiers/atomic groups quando corretos e não aceite regex arbitrária. Executar em outra task com timeout não garante interromper imediatamente a engine.

> **Referências oficiais:** [Pattern](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/util/regex/Pattern.html), [Regular Expressions](https://dev.java/learn/regex/)

---

## Parte 27 — Engenharia para Produção

[⬆️ Voltar ao Sumário](#sumário)

Software de produção precisa ser testável, observável, configurável, seguro, atualizável e distribuível. A sintaxe é apenas uma parcela do trabalho.

---

### 27.1 Estratégia de testes

[⬆️ Voltar ao Sumário](#sumário)

Combine níveis:

| Nível | Verifica |
|---|---|
| unitário | regra isolada e rápida |
| integração | banco, broker, filesystem, container ou framework real |
| contrato | compatibilidade entre produtor e consumidor |
| end-to-end | fluxo crítico pelo sistema implantado |
| propriedade | invariantes sobre muitos dados gerados |

```java
// JUnit é dependência externa, não parte do Java SE.
@org.junit.jupiter.api.Test
void somaDoisValores() {
    org.junit.jupiter.api.Assertions.assertEquals(5, Calculadora.somar(2, 3));
}
```

Teste comportamento observável, não detalhes privados. Use relógio, random e I/O controláveis. Testes concorrentes precisam de coordenação determinística; `Thread.sleep` costuma produzir flakiness.

> **Fonte oficial do projeto:** [JUnit 5 User Guide](https://docs.junit.org/5.13.4/user-guide/)

---

### 27.2 Logging, configuração e segredos

[⬆️ Voltar ao Sumário](#sumário)

Java SE oferece `System.Logger` e `java.util.logging`; ecossistema frequentemente usa SLF4J com implementação. Não concatene trabalho caro nem segredos em mensagens.

```java
private static final System.Logger LOG =
        System.getLogger(PedidoService.class.getName());

LOG.log(System.Logger.Level.INFO, "Pedido {0} confirmado", pedidoId);
```

Configuração pode vir de argumento, arquivo, environment variable, system property, secret manager ou plataforma. Defina precedência e validação na inicialização. `System.getenv` e `System.getProperty` não são equivalentes.

Nunca registre senha, token, cookie de sessão, chave privada ou payload sensível. Mascaramento deve ocorrer antes de sinks externos.

> **Referências oficiais:** [System.Logger](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/lang/System.Logger.html), [Package `java.util.logging`](https://docs.oracle.com/en/java/javase/25/docs/api/java.logging/java/util/logging/package-summary.html)

---

### 27.3 JFR, JMX, profiling e observabilidade

[⬆️ Voltar ao Sumário](#sumário)

JDK Flight Recorder coleta eventos de JVM e aplicação com overhead controlado. Java Mission Control analisa gravações. JMX expõe gerenciamento; `jcmd` solicita diagnósticos; logs de GC mostram memória.

```powershell
jcmd <pid> JFR.start name=producao settings=profile duration=60s filename=app.jfr
jcmd <pid> Thread.dump_to_file -format=json threads.json
jcmd <pid> GC.heap_info
```

```java
@jdk.jfr.Name("com.exemplo.PedidoConfirmado")
final class PedidoConfirmadoEvent extends jdk.jfr.Event {
    long pedidoId;
}
```

Observabilidade combina logs, métricas e traces. OpenTelemetry é externo; JFR é diagnóstico do JDK. Meça antes de otimizar e reproduza com carga, heap e JDK próximos da produção.

> **Referências oficiais:** [JDK Flight Recorder](https://docs.oracle.com/en/java/javase/25/jfapi/), [Monitoring and Management Guide](https://docs.oracle.com/en/java/javase/25/management/)

---

### 27.4 Segurança essencial

[⬆️ Voltar ao Sumário](#sumário)

- valide autorização no recurso, não apenas na rota;
- use prepared statements; nunca concatene SQL;
- limite tamanho, tempo, profundidade e cardinalidade de entrada;
- use `SecureRandom`, TLS e algoritmos modernos por APIs JCA/JSSE;
- atualize JDK e dependências continuamente;
- trate desserialização, XXE, path traversal, SSRF e command injection;
- não dependa do Security Manager: ele foi permanentemente desabilitado.

```java
var random = new java.security.SecureRandom();
byte[] token = new byte[32];
random.nextBytes(token);
String valor = java.util.Base64.getUrlEncoder().withoutPadding().encodeToString(token);
```

Não implemente criptografia própria. Escolha algoritmo, modo, padding e gestão de chaves com especialista e documentação atual.

> **Referências oficiais:** [Secure Coding Guidelines](https://www.oracle.com/java/technologies/javase/seccodeguide.html), [Security Developer’s Guide](https://docs.oracle.com/en/java/javase/25/security/), [JEP 486 — Permanently Disable the Security Manager](https://openjdk.org/jeps/486)

---

### 27.5 `jlink`, `jpackage`, CDS e Native Image

[⬆️ Voltar ao Sumário](#sumário)

| Mecanismo | Resultado |
|---|---|
| JAR | classes/recursos; requer runtime compatível |
| `jlink` | runtime image modular customizada |
| `jpackage` | pacote/instalador nativo com aplicação e runtime |
| CDS/AppCDS | arquivo compartilhado de metadados/classes para startup/memória |
| GraalVM Native Image | executável AOT externo ao Java SE |

```powershell
jlink --add-modules com.exemplo.app --module-path mods --output image
jpackage --name MinhaApp --input dist --main-jar app.jar --type app-image
```

`jlink` exige módulos e não produz instalador. `jpackage` cria artefato específico de plataforma. AOT impõe análise fechada e configuração para reflection, resources, JNI e proxies; teste o artefato final.

> **Referências oficiais:** [`jlink`](https://docs.oracle.com/en/java/javase/25/docs/specs/man/jlink.html), [`jpackage`](https://docs.oracle.com/en/java/javase/25/docs/specs/man/jpackage.html), [GraalVM Native Image](https://www.graalvm.org/latest/reference-manual/native-image/)

---

### 27.6 APIs públicas, compatibilidade e evolução

[⬆️ Voltar ao Sumário](#sumário)

Compatibilidade possui dimensões:

- **fonte:** consumidor recompila?
- **binária:** class file existente liga e executa?
- **comportamental:** resultado e exceções permanecem?
- **serializada:** JSON, banco ou Java serialization permanecem legíveis?
- **modular:** exports, services e reflection continuam acessíveis?

Adicionar método abstrato a interface quebra implementações; default method pode preservar binário, mas criar conflito. Alterar constante `public static final` pode não afetar consumidores sem recompilação por inlining. Mudar `equals` de chave altera coleções e persistência.

Use `@Deprecated` com alternativa, semantic versioning do projeto, release notes e ferramentas como `jdeps`/`japicmp` externas quando aplicável.

> **Referências oficiais:** [JLS §13 — Binary Compatibility](https://docs.oracle.com/javase/specs/jls/se25/html/jls-13.html), [`jdeps`](https://docs.oracle.com/en/java/javase/25/docs/specs/man/jdeps.html)

---

## Parte 28 — Catálogo da Linguagem e da Biblioteca Padrão

[⬆️ Voltar ao Sumário](#sumário)

Este capítulo é catálogo de consulta. A API Java SE contém milhares de tipos; a lista completa pertence ao Javadoc oficial. Aqui estão as palavras e famílias que um engenheiro encontra com maior frequência.

---

### 28.1 Linguagem, JVM, JDK, módulos e dependências não são sinônimos

[⬆️ Voltar ao Sumário](#sumário)

| Camada | Exemplos | Fonte correta |
|---|---|---|
| linguagem | `class`, generics, lambdas, conversões | JLS |
| formato/runtime | bytecode, class loading, frames, heap | JVMS e guias da JVM |
| Java SE API | `String`, `List`, `Files`, `HttpClient` | Javadoc/guia do JDK |
| ferramenta JDK | `javac`, `jar`, `jlink`, `jcmd` | tool specifications |
| módulo | `java.base`, `java.sql`, `jdk.jfr` | module summary/JPMS |
| dependência externa | Spring, Jackson, JUnit | documentação do projeto |
| ferramenta de build | Maven, Gradle | documentação do build |

Package organiza nomes; módulo declara fronteiras; JAR empacota; Maven coordinate identifica artefato; classpath/module path resolvem classes. Usar o termo certo evita procurar “função do Java” no lugar errado.

> **Referências oficiais:** [Java SE 25 Specifications](https://docs.oracle.com/en/java/javase/25/docs/specs/), [Java SE 25 API](https://docs.oracle.com/en/java/javase/25/docs/api/)

---

### 28.2 Todas as palavras-chave reservadas

[⬆️ Voltar ao Sumário](#sumário)

A JLS 25 define **51 sequências reservadas**. Elas não podem ser identificadores comuns.

| | | | | |
|---|---|---|---|---|
| `abstract` | `continue` | `for` | `new` | `switch` |
| `assert` | `default` | `if` | `package` | `synchronized` |
| `boolean` | `do` | `goto` | `private` | `this` |
| `break` | `double` | `implements` | `protected` | `throw` |
| `byte` | `else` | `import` | `public` | `throws` |
| `case` | `enum` | `instanceof` | `return` | `transient` |
| `catch` | `extends` | `int` | `short` | `try` |
| `char` | `final` | `interface` | `static` | `void` |
| `class` | `finally` | `long` | `strictfp` | `volatile` |
| `const` | `float` | `native` | `super` | `while` |
| `_` |  |  |  |  |

`const` e `goto` são reservadas mas não usadas. `strictfp` é obsoleta. `_` não pode ser identificador comum isolado, mas aparece como unnamed variable/pattern nas posições permitidas. `true`, `false` e `null` são literais, não keywords.

| Grupo | Palavras mais relevantes |
|---|---|
| tipos/declarações | `class`, `interface`, `enum`, `extends`, `implements` |
| acesso/modificação | `public`, `protected`, `private`, `static`, `final`, `abstract` |
| fluxo | `if`, `else`, `switch`, `case`, `for`, `while`, `break`, `continue`, `return` |
| exceções/recursos | `try`, `catch`, `finally`, `throw`, `throws` |
| concorrência/interop | `synchronized`, `volatile`, `native`, `transient` |
| tipos primitivos | `boolean`, `byte`, `short`, `int`, `long`, `char`, `float`, `double`, `void` |

> **Referência oficial:** [JLS §3.9 — Keywords](https://docs.oracle.com/javase/specs/jls/se25/html/jls-3.html#jls-3.9)

---

### 28.3 Todas as palavras-chave contextuais

[⬆️ Voltar ao Sumário](#sumário)

A JLS 25 define **17 sequências contextuais**: o significado depende da posição gramatical.

| | | | | |
|---|---|---|---|---|
| `exports` | `opens` | `requires` | `uses` | `yield` |
| `module` | `permits` | `sealed` | `var` |  |
| `non-sealed` | `provides` | `to` | `when` |  |
| `open` | `record` | `transitive` | `with` |  |

| Contexto | Palavras |
|---|---|
| módulos | `module`, `open`, `requires`, `transitive`, `exports`, `opens`, `to`, `uses`, `provides`, `with` |
| inferência | `var` |
| switch/patterns | `yield`, `when` |
| hierarquias | `record`, `sealed`, `non-sealed`, `permits` |

Algumas não podem ser usadas em certos nomes de tipo mesmo fora do ponto gramatical principal. A tokenização é definida formalmente pela JLS; não trate “contextual” como “sempre pode ser variável”.

> **Referência oficial:** [JLS §3.9 — Contextual Keywords](https://docs.oracle.com/javase/specs/jls/se25/html/jls-3.html#jls-3.9)

---

### 28.4 Tipos primitivos, wrappers e literais

[⬆️ Voltar ao Sumário](#sumário)

| Primitivo | Wrapper | Parse/conversão comum | Stream especializado |
|---|---|---|---|
| `boolean` | `Boolean` | `Boolean.parseBoolean` | — |
| `byte` | `Byte` | `Byte.parseByte` | `IntStream` após promoção |
| `short` | `Short` | `Short.parseShort` | `IntStream` |
| `int` | `Integer` | `Integer.parseInt` | `IntStream` |
| `long` | `Long` | `Long.parseLong` | `LongStream` |
| `char` | `Character` | APIs de code point | `IntStream` por `codePoints` |
| `float` | `Float` | `Float.parseFloat` | `DoubleStream` após conversão |
| `double` | `Double` | `Double.parseDouble` | `DoubleStream` |

`void` é keyword usada em retorno sem valor; `Void` é classe placeholder e não wrapper instanciável equivalente. `BigInteger` e `BigDecimal` são classes para precisão/faixa além dos primitivos.

Literais importantes:

```java
int decimal = 1_000;
int hex = 0xCAFE;
int bits = 0b1010;
long grande = 9_000_000_000L;
float simples = 1.5F;
double especial = Double.NaN;
String bloco = """
        texto
        em várias linhas
        """;
```

**Leitura guiada:** `_` separa visualmente dígitos e não altera o valor; `0x` e `0b` escolhem bases hexadecimal e binária; `L` força `long`; `F` força `float`, pois um literal decimal de ponto flutuante é `double` por padrão. `Double.NaN` é um valor especial, não um literal da gramática. O text block continua sendo `String` e aplica regras próprias de delimitador, indentação incidental e escapes.

> **Referências oficiais:** [JLS §4.2 — Primitive Types](https://docs.oracle.com/javase/specs/jls/se25/html/jls-4.html#jls-4.2), [JLS §3.10 — Literals](https://docs.oracle.com/javase/specs/jls/se25/html/jls-3.html#jls-3.10)

---

### 28.5 Operações prontas por domínio

[⬆️ Voltar ao Sumário](#sumário)

| Domínio | Tipos/APIs que devem ser consultados primeiro |
|---|---|
| texto | `String`, `StringBuilder`, `Character`, `Normalizer`, `BreakIterator` |
| matemática | `Math`, `StrictMath`, `BigInteger`, `BigDecimal` |
| datas/tempo | `java.time.*`, `Clock`, `DateTimeFormatter` |
| arquivos | `Path`, `Files`, `FileSystem`, `WatchService` |
| bytes/encoding | `ByteBuffer`, `Charset`, `Base64`, `HexFormat` |
| identificadores | `UUID` |
| aleatoriedade | `RandomGenerator`, `ThreadLocalRandom`, `SecureRandom` |
| regex | `Pattern`, `Matcher` |
| HTTP/URI | `URI`, `URL`, `HttpClient`, `HttpRequest`, `HttpResponse` |
| processos | `ProcessBuilder`, `ProcessHandle` |
| criptografia | `MessageDigest`, `Mac`, `Cipher`, `Signature`, `KeyStore` |
| compactação | `ZipInputStream`, `ZipFile`, `GZIPInputStream` |
| XML | DOM, SAX, StAX, XPath e Transformer dos módulos XML |

#### Exemplo: inverter texto sem reimplementar o básico

```java
String invertidoPorUnidadeUtf16 = new StringBuilder("Java").reverse().toString();
```

**Leitura guiada:** `StringBuilder` cria um buffer mutável com o conteúdo, `reverse()` inverte sua sequência e `toString()` produz o resultado imutável. O método já preserva pares substitutos válidos melhor que um laço ingênuo por `char`, mas “inverter texto humano” pode exigir grapheme clusters e regras de apresentação. API pronta resolve o mecanismo documentado; o domínio ainda define o significado.

#### Exemplo: utilitários modernos

```java
String hex = java.util.HexFormat.of().formatHex(new byte[] { 1, 2, 15 });
byte[] bytes = java.util.Base64.getUrlDecoder().decode("SGVsbG8=");
int limitado = Math.clamp(150, 0, 100);
var id = java.util.UUID.randomUUID();
```

**Leitura guiada:** `HexFormat` codifica cada byte como dígitos hexadecimais; o decoder Base64 reconstrói bytes a partir de texto; `Math.clamp` limita o valor ao intervalo inclusivo; `UUID.randomUUID` cria um identificador aleatório. Cada API resolve um mecanismo, mas o domínio ainda precisa decidir formato, segurança e política de erro.

> **Referências oficiais:** [All Classes and Interfaces](https://docs.oracle.com/en/java/javase/25/docs/api/allclasses-index.html), [`java.base`](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/module-summary.html)

---

### 28.6 Estruturas de dados e coleções prontas

[⬆️ Voltar ao Sumário](#sumário)

| Estrutura | Implementações principais | Observação |
|---|---|---|
| array fixo | `T[]`, primitivas `int[]` etc. | reificado, covariante, tamanho fixo |
| lista | `ArrayList`, `LinkedList` | posição e duplicatas |
| set hash | `HashSet`, `LinkedHashSet` | unicidade por `equals`/`hashCode` |
| set ordenado | `TreeSet` | árvore navegável por comparator |
| mapa hash | `HashMap`, `LinkedHashMap` | chave/valor |
| mapa ordenado | `TreeMap` | navegação por ordem |
| deque | `ArrayDeque`, `LinkedList` | fila/pilha pelas extremidades |
| prioridade | `PriorityQueue` | cabeça pela prioridade |
| enum | `EnumSet`, `EnumMap` | especialização compacta |
| bits | `BitSet` | conjunto indexado de bits |
| referência fraca | `WeakHashMap` | entradas não mantêm chaves fortemente |
| concorrente | `ConcurrentHashMap`, `CopyOnWriteArrayList` | contratos específicos de concorrência |
| bloqueante | `ArrayBlockingQueue`, `LinkedBlockingQueue`, `DelayQueue` | coordenação produtor/consumidor |
| skip list | `ConcurrentSkipListMap/Set` | concorrente e ordenada |
| não modificável | `List.of`, `Set.of`, `Map.of`, `copyOf` | factories/snapshots |

Java SE não fornece árvore binária de propósito geral como interface pública porque “árvore” sozinha não define contrato; `TreeMap`/`TreeSet` resolvem mapa/set ordenado. Para grafo, trie ou árvore de domínio, escolha biblioteca externa ou modelo próprio fundamentado no problema.

> **Referências oficiais:** [Collections Framework](https://dev.java/learn/api/collections-framework/), [`java.util.concurrent`](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/util/concurrent/package-summary.html)

---

### 28.7 Interfaces funcionais e contratos prontos

[⬆️ Voltar ao Sumário](#sumário)

| Contrato | Significado |
|---|---|
| `Iterable<T>` | pode fornecer iterator |
| `Collection<T>` | grupo mutável/operações opcionais |
| `List<T>` | sequência posicional |
| `Set<T>` | unicidade |
| `Map<K,V>` | chave para valor |
| `Comparable<T>` | ordem natural |
| `Comparator<T>` | estratégia externa de comparação |
| `AutoCloseable` | requer fechamento |
| `Runnable` | tarefa sem retorno checked |
| `Callable<V>` | tarefa com retorno e checked exception |
| `Future<V>` | resultado eventual e cancelável |
| `Flow.Publisher/Subscriber` | Reactive Streams com backpressure |
| `CharSequence` | sequência legível de chars |
| `Appendable` | destino que recebe chars |
| `Readable` | fonte de chars |
| `RandomGenerator` | contrato moderno de gerador aleatório |

Interfaces funcionais centrais ficam em `java.util.function`. Antes de criar `TriFunction`, `ThrowingFunction` ou contrato assíncrono próprio, veja se o nome de domínio ou uma biblioteca já expressa melhor a semântica.

> **Referências oficiais:** [Java SE API — Interfaces](https://docs.oracle.com/en/java/javase/25/docs/api/allclasses-index.html), [`java.util.function`](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/util/function/package-summary.html)

---

### 28.8 Packages e módulos essenciais

[⬆️ Voltar ao Sumário](#sumário)

| Módulo/package | Conteúdo |
|---|---|
| `java.base/java.lang` | tipos fundamentais, threads, processos, FFM |
| `java.base/java.util` | coleções, Optional, UUID, Scanner, properties |
| `java.base/java.util.stream` | pipelines de dados |
| `java.base/java.util.concurrent` | executors, locks, atomics, coleções concorrentes |
| `java.base/java.io` | streams clássicos e serialização |
| `java.base/java.nio*` | buffers, channels, paths, charsets |
| `java.base/java.time` | datas, instantes, duração e fusos |
| `java.net.http` | cliente HTTP/WebSocket |
| `java.sql` | JDBC |
| `java.logging` | logging padrão |
| `java.xml` | XML, XPath e transformação |
| `java.management` / `jdk.management` | JMX e gestão da JVM |
| `jdk.jfr` | Flight Recorder API |
| `java.desktop` | AWT, Swing, JavaBeans, imagem e áudio |

Nem todo módulo entra automaticamente na configuração da aplicação. Use `java --list-modules`, `java --describe-module`, Javadoc e `jdeps` para descobrir dependências.

> **Referência oficial:** [Java SE Module Summary](https://docs.oracle.com/en/java/javase/25/docs/api/)

---

### 28.9 Algoritmos e utilitários que você não precisa reimplementar

[⬆️ Voltar ao Sumário](#sumário)

| Necessidade | API pronta |
|---|---|
| ordenar/buscar lista | `List.sort`, `Collections.sort`, `binarySearch` |
| comparar objetos | `Comparator.comparing`, `thenComparing`, `nullsFirst` |
| copiar/preencher array | `Arrays.copyOf`, `fill`, `setAll` |
| diferença/comparação arrays | `Arrays.equals`, `mismatch`, `compare` |
| agregar sequências | Stream API e collectors |
| mínimo/máximo/clamp | `Math.min`, `max`, `clamp` |
| aritmética exata | `Math.addExact`, `multiplyExact`, `toIntExact` |
| hash/equality null-safe | `Objects.equals`, `hash`, `deepEquals` |
| validar índices | `Objects.checkIndex`, `checkFromToIndex` |
| parsing numérico | wrappers, `BigInteger`, `BigDecimal`, `Scanner` |
| encoding | `Charset`, `Base64`, `HexFormat` |
| checksum/digest | `CRC32`, `Adler32`, `MessageDigest` |
| rate de tempo monotônico | `System.nanoTime` para duração |

```java
var porNomeDepoisId = java.util.Comparator
        .comparing(Cliente::nome, String.CASE_INSENSITIVE_ORDER)
        .thenComparingLong(Cliente::id);
```

Conheça semântica e complexidade. `binarySearch` exige coleção previamente ordenada pelo mesmo comparator; `nanoTime` mede intervalo, não instante civil.

> **Referências oficiais:** [Collections](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/util/Collections.html), [Arrays](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/util/Arrays.html), [Objects](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/util/Objects.html)

---

### 28.10 Como descobrir se a API já existe

[⬆️ Voltar ao Sumário](#sumário)

Use este fluxo antes de criar `Utils`, estrutura ou dependência:

1. identifique o contrato: texto, coleção, tempo, bytes, rede, concorrência;
2. pesquise no [Java SE API](https://docs.oracle.com/en/java/javase/25/docs/api/) pelo substantivo e verbo;
3. abra o package/module summary, não apenas um snippet de busca;
4. leia `Since`, `Deprecated`, nulidade documentada, exceções e thread safety;
5. verifique se a API existe no `--release` suportado;
6. só então avalie biblioteca externa ou implementação própria.

O Javadoc possui busca por classe, método, campo, system property e termo. `jshell` acelera experimentos; `javap` confirma bytecode; `jdeps` mostra dependências reais.

> **Referências oficiais:** [Javadoc Help](https://docs.oracle.com/en/java/javase/25/docs/api/help-doc.html), [Learn Java](https://dev.java/learn/)

---

## Parte 29 — Ecossistema Externo: Frameworks, Bibliotecas e Ferramentas

[⬆️ Voltar ao Sumário](#sumário)

Este é um mapa curado, não ranking. Cada item aponta para a documentação mantida pelo projeto. Versão, licença, suporte e compatibilidade mudam: valide no momento da adoção.

---

### 29.1 O que é externo ao Java SE

[⬆️ Voltar ao Sumário](#sumário)

| Categoria | Exemplo | Compromisso adicional |
|---|---|---|
| especificação de plataforma | Jakarta EE, MicroProfile | runtime/implementação compatível |
| framework | Spring Boot, Quarkus, Micronaut | lifecycle, DI, configuração e build próprios |
| biblioteca | Jackson, Guava, Resilience4j | API, versão, transitivos e licença |
| driver | PostgreSQL JDBC, MongoDB Java Driver | protocolo e compatibilidade do produto |
| ferramenta | Maven, Gradle, JUnit, JMH | plugins, CI e atualização |
| runtime/distribuição | Temurin, Oracle JDK, GraalVM | suporte, licença e características do fornecedor |

Java SE não inclui servidor Jakarta, ORM, JSON mapper geral, framework de testes ou sistema de build completo. Não instale pacote para substituir sem necessidade o que `java.base` já resolve; não force API padrão quando o requisito pede uma biblioteca especializada.

> **Referência oficial:** [Java SE API](https://docs.oracle.com/en/java/javase/25/docs/api/)

---

### 29.2 Plataformas e frameworks de aplicação

[⬆️ Voltar ao Sumário](#sumário)

| Tecnologia | Foco | Fonte oficial |
|---|---|---|
| Jakarta EE | especificações empresariais padronizadas | [Jakarta EE](https://jakarta.ee/specifications/) |
| Eclipse MicroProfile | APIs cloud-native sobre Jakarta | [MicroProfile](https://microprofile.io/) |
| Spring Framework | DI, web, dados, transações e infraestrutura | [Spring Framework](https://docs.spring.io/spring-framework/reference/) |
| Spring Boot | convenções, auto-configuração e operação Spring | [Spring Boot](https://docs.spring.io/spring-boot/index.html) |
| Quarkus | cloud-native, build-time e integração GraalVM | [Quarkus](https://quarkus.io/guides/) |
| Micronaut | DI/AOP em compilação e cloud | [Micronaut](https://docs.micronaut.io/latest/guide/) |
| Helidon | microservices, SE e MP | [Helidon](https://helidon.io/docs/latest/) |
| Eclipse Vert.x | toolkit reativo/event-loop | [Vert.x](https://vertx.io/docs/) |
| Dropwizard | serviços REST com stack opinativa | [Dropwizard](https://www.dropwizard.io/en/latest/) |

Não compare apenas startup. Avalie modelo de threads, DI, maturidade, ecossistema, native image, observabilidade e conhecimento da equipe.

---

### 29.3 Dados, bancos e persistência

[⬆️ Voltar ao Sumário](#sumário)

| Tecnologia | Papel | Fonte oficial |
|---|---|---|
| JDBC | API padrão de acesso relacional | [JDBC](https://docs.oracle.com/en/java/javase/25/docs/api/java.sql/java/sql/package-summary.html) |
| Jakarta Persistence | especificação ORM | [Jakarta Persistence](https://jakarta.ee/specifications/persistence/) |
| Hibernate ORM | ORM/implementação Jakarta Persistence | [Hibernate ORM](https://hibernate.org/orm/documentation/) |
| EclipseLink | implementação de persistência Jakarta | [EclipseLink](https://eclipse.dev/eclipselink/documentation/) |
| jOOQ | SQL tipado e geração de código | [jOOQ](https://www.jooq.org/doc/latest/manual/) |
| MyBatis | mapeamento SQL explícito | [MyBatis](https://mybatis.org/mybatis-3/) |
| HikariCP | pool JDBC | [HikariCP](https://github.com/brettwooldridge/HikariCP) |
| Flyway | migrations versionadas | [Flyway](https://documentation.red-gate.com/flyway) |
| Liquibase | migrations e changelogs | [Liquibase](https://docs.liquibase.com/) |
| MongoDB Java Driver | driver oficial MongoDB | [MongoDB Java Driver](https://www.mongodb.com/docs/drivers/java/sync/current/) |

ORM não elimina SQL, índices, transações, N+1, locking ou lifecycle do persistence context. Pool não aumenta capacidade do banco; limite e monitore.

---

### 29.4 HTTP, resiliência, mensageria e jobs

[⬆️ Voltar ao Sumário](#sumário)

| Tecnologia | Capacidade | Fonte oficial |
|---|---|---|
| Netty | rede assíncrona e codecs | [Netty](https://netty.io/wiki/) |
| OkHttp | cliente HTTP | [Repositório oficial](https://github.com/square/okhttp) |
| Apache HttpComponents | clientes/servidores HTTP de baixo nível | [HttpComponents](https://hc.apache.org/) |
| Resilience4j | retry, circuit breaker, rate limiter, bulkhead | [Resilience4j](https://resilience4j.readme.io/) |
| Apache Kafka | event streaming | [Kafka](https://kafka.apache.org/documentation/) |
| RabbitMQ Java Client | AMQP/RabbitMQ | [RabbitMQ Java Client](https://www.rabbitmq.com/client-libraries/java-api-guide) |
| Apache Pulsar | streaming/mensageria | [Pulsar](https://pulsar.apache.org/docs/) |
| Quartz | scheduling persistente | [Quartz](https://www.quartz-scheduler.org/documentation/) |
| Spring Batch | processamento em lotes | [Spring Batch](https://docs.spring.io/spring-batch/reference/) |

Retry só é seguro com semântica idempotente ou deduplicação. Mensagens podem repetir e sair de ordem. Circuit breaker não corrige timeout ausente; fila não elimina necessidade de backpressure.

---

### 29.5 Serialização, mapeamento, validação e produtividade

[⬆️ Voltar ao Sumário](#sumário)

| Biblioteca | Uso | Fonte oficial |
|---|---|---|
| Jackson | JSON e outros formatos com data binding | [Jackson](https://github.com/FasterXML/jackson-docs) |
| Gson | JSON simples | [Gson](https://github.com/google/gson) |
| JSON-B / JSON-P | especificações Jakarta para JSON | [JSON-B](https://jakarta.ee/specifications/jsonb/), [JSON-P](https://jakarta.ee/specifications/jsonp/) |
| MapStruct | mapeamento gerado em compilação | [MapStruct](https://mapstruct.org/documentation/stable/reference/html/) |
| Jakarta Validation | contratos declarativos de validação | [Jakarta Validation](https://jakarta.ee/specifications/bean-validation/) |
| Apache Commons | utilitários por módulos | [Apache Commons](https://commons.apache.org/) |
| Guava | coleções/utilitários Google | [Guava](https://github.com/google/guava/wiki) |
| Lombok | geração via integração com compilador | [Project Lombok](https://projectlombok.org/features/) |

Serialização é contrato externo. Polimorfismo aberto em entrada não confiável pode ser vulnerável. Lombok reduz fonte visível, mas afeta compilador, IDE e entendimento do bytecode; records ou geração padrão podem ser alternativas.

---

### 29.6 Logging e observabilidade

[⬆️ Voltar ao Sumário](#sumário)

| Tecnologia | Papel | Fonte oficial |
|---|---|---|
| `java.util.logging` / `System.Logger` | APIs padrão | [JUL](https://docs.oracle.com/en/java/javase/25/docs/api/java.logging/java/util/logging/package-summary.html) |
| SLF4J | fachada de logging | [SLF4J](https://www.slf4j.org/manual.html) |
| Logback | implementação de logging | [Logback](https://logback.qos.ch/manual/) |
| Log4j 2 | API/implementação com appenders | [Log4j 2](https://logging.apache.org/log4j/2.x/manual/index.html) |
| OpenTelemetry Java | traces, métricas e logs | [OpenTelemetry Java](https://opentelemetry.io/docs/languages/java/) |
| Micrometer | instrumentação dimensional | [Micrometer](https://docs.micrometer.io/micrometer/reference/) |

Use uma fachada e uma implementação coerentes; múltiplos bindings/bridges podem criar loops. Templates estruturados mantêm campos separados; limite cardinalidade de métricas e não registre segredos.

---

### 29.7 Testes, automação e medição

[⬆️ Voltar ao Sumário](#sumário)

| Ferramenta | Uso | Fonte oficial |
|---|---|---|
| JUnit | testes e extensões | [JUnit](https://docs.junit.org/current/user-guide/) |
| TestNG | testes com suites/configuração | [TestNG](https://testng.org/) |
| Mockito | test doubles | [Mockito](https://site.mockito.org/) |
| AssertJ | assertions fluentes | [AssertJ](https://assertj.github.io/doc/) |
| Testcontainers | dependências reais em containers | [Testcontainers](https://java.testcontainers.org/) |
| WireMock | simulação HTTP | [WireMock](https://wiremock.org/docs/) |
| Awaitility | espera declarativa em testes assíncronos | [Awaitility](https://github.com/awaitility/awaitility/wiki/Usage) |
| JMH | microbenchmark do OpenJDK | [JMH](https://openjdk.org/projects/code-tools/jmh/) |
| JaCoCo | cobertura de bytecode | [JaCoCo](https://www.jacoco.org/jacoco/trunk/doc/) |

Microbenchmark manual com `System.nanoTime` costuma medir warmup, dead-code elimination ou ambiente. Use JMH. Cobertura indica linhas executadas, não qualidade de assertions.

---

### 29.8 UI, mobile e jogos

[⬆️ Voltar ao Sumário](#sumário)

| Tecnologia | Contexto | Fonte oficial |
|---|---|---|
| Swing/AWT | desktop incluído em `java.desktop` | [Java Desktop](https://docs.oracle.com/en/java/javase/25/docs/api/java.desktop/module-summary.html) |
| JavaFX/OpenJFX | desktop moderno separado do JDK | [OpenJFX](https://openjfx.io/) |
| Android SDK | mobile Android com Java/Kotlin | [Android Developers](https://developer.android.com/) |
| libGDX | jogos multiplataforma | [libGDX](https://libgdx.com/wiki/) |
| LWJGL | bindings nativos para gráficos/áudio/computação | [LWJGL](https://www.lwjgl.org/guide) |
| jMonkeyEngine | engine Java 3D | [jMonkeyEngine](https://wiki.jmonkeyengine.org/) |

Cada framework controla main thread, lifecycle, assets, empacotamento e APIs disponíveis. Valide versão do JDK/Android e plataformas de destino antes de usar recurso recente da linguagem.

---

### 29.9 Como avaliar e adotar uma dependência

[⬆️ Voltar ao Sumário](#sumário)

Antes de instalar, responda:

1. A Java SE API já resolve?
2. Quem mantém, qual licença e frequência de release?
3. Quais JDKs, módulos e frameworks são suportados?
4. Quais transitivos, CVEs e conflitos entram?
5. A API domina tipos públicos do projeto e cria lock-in?
6. Como configura timeout, memória, threads e shutdown?
7. Existe documentação de migração e compatibilidade?
8. Como remover ou substituir depois?

```powershell
./mvnw dependency:tree
./mvnw versions:display-dependency-updates
./gradlew dependencies
./gradlew dependencyInsight --dependency jackson-databind
jdeps --multi-release 25 --recursive app.jar
```

Fixe versões de plugins e wrapper. Use lock/verificação de checksums e repositórios permitidos conforme a ferramenta. Ausência de alerta conhecido não prova segurança; atualização é processo contínuo.

> **Fontes oficiais:** [Maven Dependency Mechanism](https://maven.apache.org/guides/introduction/introduction-to-dependency-mechanism.html), [Gradle Dependency Management](https://docs.gradle.org/current/userguide/dependency_management.html), [OpenJDK Vulnerability Group](https://openjdk.org/groups/vulnerability/)

---

## Anexo A — Trilhas Oficiais de Estudo e Prática

[⬆️ Voltar ao Sumário](#sumário)

| Recurso | Uso recomendado |
|---|---|
| [Learn Java](https://dev.java/learn/) | tutoriais progressivos da linguagem, APIs e JVM |
| [Java Language Specification](https://docs.oracle.com/javase/specs/jls/se25/html/index.html) | semântica normativa da linguagem |
| [Java Virtual Machine Specification](https://docs.oracle.com/javase/specs/jvms/se25/html/index.html) | class files, bytecode e runtime abstrato |
| [Java SE API](https://docs.oracle.com/en/java/javase/25/docs/api/) | consulta de módulos, packages, tipos e membros |
| [JDK 25 Documentation](https://docs.oracle.com/en/java/javase/25/) | ferramentas, guias, segurança, GC e troubleshooting |
| [OpenJDK JEP Index](https://openjdk.org/jeps/0) | motivação e histórico de recursos do JDK |
| [Inside Java](https://inside.java/) | comunicação técnica do time Java |

Prática sugerida:

1. reimplemente exemplos sem copiar;
2. altere entradas e preveja o resultado antes de executar;
3. compile com `-Xlint:all`;
4. inspecione um class file com `javap -c -v`;
5. escreva testes para invariantes e erros;
6. use JFR em uma aplicação pequena;
7. leia a seção da JLS quando o comportamento surpreender.

---

## Anexo B — Referências Oficiais Consultadas

[⬆️ Voltar ao Sumário](#sumário)

### Linguagem, JVM e releases

Estas são as fontes primárias para regras da linguagem, formato e execução da JVM e conjunto de recursos de cada release. Em caso de conflito com tutorial, livro ou postagem, a especificação aplicável é a referência normativa.

- [Java Language Specification, Java SE 25](https://docs.oracle.com/javase/specs/jls/se25/html/index.html)
- [Java Virtual Machine Specification, Java SE 25](https://docs.oracle.com/javase/specs/jvms/se25/html/index.html)
- [Java SE Specifications](https://docs.oracle.com/javase/specs/)
- [Java Language Changes Summary](https://docs.oracle.com/en/java/javase/25/language/java-language-changes-summary.html)
- [Oracle Java SE Support Roadmap](https://www.oracle.com/java/technologies/java-se-support-roadmap.html)
- [OpenJDK JDK 25](https://openjdk.org/projects/jdk/25/)
- [OpenJDK JDK 26](https://openjdk.org/projects/jdk/26/)

### API, ferramentas e runtime

Use a API Specification para contratos de classes e métodos; use as tool specifications e os guias do JDK para comportamento operacional. Flags internas ou observações de uma implementação não devem ser promovidas a garantia universal da linguagem.

- [Java SE 25 API](https://docs.oracle.com/en/java/javase/25/docs/api/)
- [JDK 25 Documentation](https://docs.oracle.com/en/java/javase/25/)
- [JDK Tool Specifications](https://docs.oracle.com/en/java/javase/25/docs/specs/man/)
- [Documentation Comment Specification for the Standard Doclet](https://docs.oracle.com/en/java/javase/25/docs/specs/javadoc/doc-comment-spec.html)
- [Core Libraries Developer Guide](https://docs.oracle.com/en/java/javase/25/core/)
- [`java.lang.IO` API](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/lang/IO.html)
- [`java.lang.ClassLoader` API](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/lang/ClassLoader.html)
- [Security Developer’s Guide](https://docs.oracle.com/en/java/javase/25/security/)
- [Garbage Collection Tuning Guide](https://docs.oracle.com/en/java/javase/25/gctuning/)
- [JDK Flight Recorder API Guide](https://docs.oracle.com/en/java/javase/25/jfapi/)
- [Java Platform Module System](https://dev.java/learn/modules/)
- [JAR File Specification](https://docs.oracle.com/en/java/javase/25/docs/specs/jar/jar.html)
- [JNI Specification](https://docs.oracle.com/en/java/javase/25/docs/specs/jni/)

### Aprendizagem oficial

- [Learn Java](https://dev.java/learn/)
- [Language Basics](https://dev.java/learn/language-basics/)
- [Classes and Objects](https://dev.java/learn/classes-objects/)
- [Generics](https://dev.java/learn/generics/)
- [Collections Framework](https://dev.java/learn/api/collections-framework/)
- [Stream API](https://dev.java/learn/api/streams/)
- [Virtual Threads](https://dev.java/learn/new-features/virtual-threads/)
- [Foreign Function and Memory API](https://dev.java/learn/ffm/)

Para projetos externos da Parte 29, cada tabela aponta para o site, manual ou repositório mantido pelo próprio projeto. Essas fontes são necessárias porque versão, licença e compatibilidade não são definidas pela especificação Java.

---

## Glossário

[⬆️ Voltar ao Sumário](#sumário)

| Termo | Definição resumida | Aprofundamento |
|---|---|---|
| **annotation** | metadado associado a declaração ou uso de tipo e interpretado por ferramenta/runtime | [Parte 19](#parte-19--annotations-e-metadados) |
| **API** | contrato público de tipos, métodos, campos e comportamento | [28.1](#281-linguagem-jvm-jdk-módulos-e-dependências-não-são-sinônimos) |
| **array** | objeto reificado de tamanho fixo com elementos de um tipo | [3.8](#38-literais-arrays-e-varargs) |
| **autoboxing** | conversão automática de primitivo para wrapper | [3.7](#37-object-wrappers-e-autoboxing) |
| **builder** | objeto mutável de construção que acumula opções antes de produzir o resultado | [11.6](#116-builder-e-apis-fluentes) |
| **bytecode** | instruções independentes de máquina armazenadas em class files | [1.2](#12-jdk-jvm-bytecode-class-files-e-jars) |
| **checked exception** | exceção que o compilador exige capturar ou declarar | [18.1](#181-checked-e-unchecked-exceptions) |
| **class loader** | componente que carrega classes; o loader definidor participa da identidade de um tipo em runtime | [1.8](#18-carregamento-linking-inicialização-e-identidade-de-tipos) |
| **classpath** | sequência de diretórios/JARs usada para localizar classes do unnamed module | [25.2](#252-fonte-class-files-classpath-e-module-path) |
| **closure** | função que captura valores do escopo envolvente | [13.5](#135-closures-callbacks-e-listeners) |
| **collector** | estratégia mutável de acumulação usada por streams | [14.4](#144-collectors-redução-e-agrupamento) |
| **compact source file** | arquivo-fonte simplificado sem classe explícita, permanente no Java 25 | [1.4](#14-estrutura-e-ponto-de-entrada-de-um-programa) |
| **effectively final** | variável não declarada final, mas nunca reatribuída | [3.5](#35-final-constantes-e-variáveis-effectively-final) |
| **erasure** | tradução que remove grande parte dos argumentos genéricos do runtime | [17.3](#173-type-erasure-e-restrições) |
| **executor** | componente que recebe tarefas e controla sua execução | [16.1](#161-executorservice-callable-e-future) |
| **FFM** | API para memória externa e chamadas nativas | [22.5](#225-foreign-function-and-memory-api) |
| **garbage collector** | mecanismo que recupera memória de objetos não alcançáveis | [20.4](#204-garbage-collector-e-alcançabilidade) |
| **happens-before** | relação da JMM que garante visibilidade/ordem entre ações | [21.2](#212-java-memory-model-e-happens-before) |
| **heap pollution** | variável parametrizada referencia valor incompatível com seu tipo | [17.3](#173-type-erasure-e-restrições) |
| **interface funcional** | interface com um único método abstrato relevante, target de lambda | [13.1](#131-interfaces-funcionais) |
| **JAR** | arquivo ZIP padronizado com classes, recursos e metadados | [1.2](#12-jdk-jvm-bytecode-class-files-e-jars) |
| **Javadoc** | ferramenta e formato de documentação de API baseados em comentários de documentação | [1.5](#15-sintaxe-mínima-tokens-comentários-statements-e-blocos) e [25.5](#255-javadoc-lint-e-análise-estática) |
| **JDK** | conjunto de runtime, ferramentas e APIs para desenvolver Java | [1.2](#12-jdk-jvm-bytecode-class-files-e-jars) |
| **JFR** | gravador de eventos de diagnóstico integrado ao JDK | [27.3](#273-jfr-jmx-profiling-e-observabilidade) |
| **JNI** | interface nativa histórica entre JVM e C/C++ | [22.4](#224-jni) |
| **JPMS** | sistema de módulos da plataforma Java | [2.3](#23-java-platform-module-system) |
| **JVM** | máquina abstrata que executa class files compatíveis | [1.2](#12-jdk-jvm-bytecode-class-files-e-jars) |
| **lambda** | expressão convertida para um target type funcional | [13.3](#133-expressões-lambda) |
| **module path** | caminho usado para resolver módulos nomeados/automáticos | [25.2](#252-fonte-class-files-classpath-e-module-path) |
| **monitor** | mecanismo associado a objeto usado por `synchronized` | [21.3](#213-monitores-locks-e-conditions) |
| **package** | namespace hierárquico de tipos | [2.1](#21-packages) |
| **pattern matching** | teste e extração estruturada de valores | [8.5](#85-pattern-matching-e-exaustividade) |
| **PECS** | orientação Producer Extends, Consumer Super para wildcards | [17.2](#172-bounds-wildcards-e-pecs) |
| **preview feature** | recurso completo para avaliação, ainda não permanente | [7.12](#712-recursos-de-preview-e-incubator) |
| **primitivo** | valor dos oito tipos internos não referenciais | [3.2](#32-tipos-primitivos-e-tipos-de-referência) |
| **record** | classe especial orientada a agregados de dados | [11.4](#114-records-e-modelagem-de-dados) |
| **reflection** | inspeção e invocação dinâmica de elementos em runtime | [22.1](#221-reflection) |
| **scoped value** | ligação imutável, dinamicamente delimitada, compartilhada do chamador com seus callees | [16.6](#166-scoped-values) |
| **sealed type** | tipo que restringe subtipos diretos permitidos | [12.3](#123-tipos-sealed-e-hierarquias-fechadas) |
| **statement** | unidade sintática executável, como declaração local, atribuição, `return` ou estrutura de controle | [1.5](#15-sintaxe-mínima-tokens-comentários-statements-e-blocos) |
| **String Pool** | conjunto canônico mantido por `String` para literais, constantes e valores internados | [4.1](#41-string-é-imutável) |
| **stream** | pipeline consumível de operações sobre elementos | [14.1](#141-o-que-é-um-stream) |
| **target type** | tipo esperado pelo contexto para lambda, method reference ou expressão | [13.1](#131-interfaces-funcionais) |
| **token** | unidade lexical classificada como identificador, palavra-chave, literal, separador ou operador | [1.5](#15-sintaxe-mínima-tokens-comentários-statements-e-blocos) |
| **try-with-resources** | construção que fecha `AutoCloseable` em ordem inversa | [7.7](#77-try-with-resources-e-autocloseable) |
| **unchecked exception** | `RuntimeException` que não exige declaração/captura | [18.1](#181-checked-e-unchecked-exceptions) |
| **unnamed module** | módulo que reúne código carregado pelo classpath | [2.3](#23-java-platform-module-system) |
| **varargs** | parâmetro final representado por array e chamado com número variável de argumentos | [3.8](#38-literais-arrays-e-varargs) |
| **virtual thread** | thread leve agendada pelo runtime, adequada a concorrência bloqueante | [16.3](#163-virtual-threads) |
| **wildcard** | argumento genérico desconhecido representado por `?` | [17.2](#172-bounds-wildcards-e-pecs) |

**Como usar a tabela:** trate o glossário como índice de revisão. As definições são curtas; contratos, versões, custos e exemplos permanecem nas seções vinculadas.
