# 🎮 PROXY QUEST - Guia Educativo sobre o Padrão Proxy

## 📚 Índice
1. [Conceito Geral](#-nível-1-o-conceito-geral-mundo-real)
2. [Tipos de Proxy](#-nível-2-tipos-de-proxy-categorias)
3. [Proxy no MvcUriComponentsBuilder](#-nível-3-o-proxy-do-mvcuricomponentsbuilder-este-caso)
4. [Código Passo a Passo](#-nível-4-o-código-passo-a-passo)
5. [Challenge](#-nível-5-challenge-seu-turno)
6. [Resumo](#-resumo-final-mapa-mental)

---

## 🏠 NÍVEL 1: O Conceito Geral (Mundo Real)

### Imagina isso:

Você quer pedir comida em um restaurante, mas não quer ir lá pessoalmente. Então você liga para um **amigo** que vai:
1. Ouvir seu pedido
2. **Não fazer nada de diferente**, só ir ao restaurante e fazer o pedido por você
3. Trazer a comida de volta

Seu **amigo é um PROXY** 🤝

### Definição Geral

> **Proxy** é um **intermediário que age em nome de alguém**, interceptando requisições.

Ele fica entre você e a "coisa real", controlando como a interação acontece.

---

## 🎯 NÍVEL 2: Tipos de Proxy (Categorias)

```
┌─────────────────────────────────────────┐
│          PROXY = INTERMEDIÁRIO          │
├─────────────────────────────────────────┤
│                                         │
│  🛡️ PROTETOR (Security Proxy)          │
│     "Só deixo passar se tiver permissão"│
│                                         │
│  📊 CONTADOR (Logging Proxy)            │
│     "Eu vou anotar o que você faz"      │
│                                         │
│  🎭 ESPIÃO (Interceptor Proxy)          │
│     "Deixa eu ver como você trabalha"   │
│                                         │
│  📸 FOTÓGRAFO (Capture Proxy)           │
│     "Deixa eu capturar o que você faz"  │
│                                         │
└─────────────────────────────────────────┘
```

### Exemplos de cada tipo:

- **Security Proxy**: Validar permissões antes de chamar um método sensível
- **Logging Proxy**: Registrar cada chamada de método para auditoria
- **Interceptor Proxy**: Adicionar comportamento antes/depois de uma ação
- **Capture Proxy**: Observar uma ação sem realmente executá-la (nosso caso!)

---

## 🎬 NÍVEL 3: O Proxy do MvcUriComponentsBuilder (Este Caso)

### 📸 Tipo: **FOTÓGRAFO** - Capture Proxy

#### Comparação: Com e Sem Proxy

```java
// ❌ SEM PROXY (você mesmo executa):
customerController.findById(UUID.randomUUID());
// → Realmente executa o método
// → Tenta buscar no banco de dados
// → Pode lançar exceção se não encontrar

// ✅ COM PROXY (proxy observa):
MvcUriComponentsBuilder.on(CustomerController.class).findById(customerId)
// → Cria um "dublê" (fake)
// → Não executa de verdade
// → Só observa: "Ah, você chamou findById com esse ID"
// → Extrai as informações da chamada
```

### 📚 Analogia da Vida Real: Estúdio de Cinema

```
┌──────────────────────────────────────────┐
│  CENA: Você vai pedir um café            │
├──────────────────────────────────────────┤
│                                          │
│  SEM PROXY (Filmagem real):              │
│  🎬 DIRETOR: "Ação!"                     ��
│  👨 VOCÊ: Vai para a cozinha, pede café │
│  → Café é realmente feito ☕            │
│  → Tempo e recursos gastos               │
│                                          │
│  COM PROXY (Ensaio com dublê):           │
│  🎬 DIRETOR: "Ação! Mas use um dublê!"  │
│  👤 DUBLÊ: Faz os mesmos gestos que você│
│  → Café NÃO é feito                      │
│  → Diretor nota: "Ele faz isso ✓"       │
│  🎬 DIRETOR: "Agora eu sei o roteiro!"  │
│  → Nada de recurso desperdiçado          │
│                                          │
└──────────────────────────────────────────┘
```

### No Contexto do CustomerController

No método `create()`:

```java
UUID customerId = customerManagementApplicationService.create(input);

UriComponentsBuilder builder = MvcUriComponentsBuilder.fromMethodCall(
    MvcUriComponentsBuilder.on(CustomerController.class).findById(customerId)
);

httpServletResponse.addHeader("Location", builder.toUriString());
```

**O Proxy aqui é usado para:**
- Observar qual método foi chamado (`findById`)
- Capturar qual parâmetro foi passado (`customerId`)
- Extrair informações de roteamento da anotação `@GetMapping("/{customerId}")`
- Construir a URL corretamente: `/api/v1/customers/{customerId}`

---

## 💻 NÍVEL 4: O Código Passo a Passo

```java
// PASSO 1: Você cria um PROXY da classe
MvcUriComponentsBuilder.on(CustomerController.class)
//                       ↑
//    "Cria um fake/dublê da classe para observação"

// PASSO 2: Você "executa" um método no proxy (mas não é de verdade!)
.findById(customerId)
//  ↑
// "O proxy observa silenciosamente: 
//  'Ah, a pessoa quer chamar findById com ID: 123'"

// PASSO 3: Você extrai as informações capturadas
MvcUriComponentsBuilder.fromMethodCall(...)
//                       ↑
//            "Ei proxy, o que você capturou?"
//            Resposta: "Method: findById, Param: 123"

// PASSO 4: Constrói a URL baseado nas anotações da classe
// @GetMapping("/{customerId}") → /api/v1/customers/123
```

### O que Spring faz internamente:

1. **Reflection**: Analisa a classe `CustomerController`
2. **Dynamic Proxy**: Cria um objeto falso que implementa a mesma interface
3. **Interceptação**: Quando você chama `findById()`, o proxy intercepta
4. **Extração**: Extrai metadados (método, parâmetros, anotações)
5. **Construção**: Monta a URL final

---

## 🎮 NÍVEL 5: Challenge (Seu Turno!)

### ❓ Pergunta de Teste

Se você **remover** o proxy e tentar fazer isso:

```java
customerController.findById(customerId)  // ← Execução REAL!
```

**O que aconteceria se o cliente NÃO existisse no banco de dados?**

<details>
<summary>🔍 Revelar Resposta</summary>

### Resposta: Lançaria uma exceção! 💥

Porque o método `findById` de verdade iria:

1. **Consultar o banco de dados** com o `customerId`
2. **Não encontraria o cliente** (pois não existe)
3. **Retornaria um erro ou null** dependendo da implementação
4. **A aplicação quebraria** ou precisaria tratar a exceção

### Por isso usamos Proxy:

```
Proxy permite "observar sem executar"
    ↓
Você vê qual é a "intenção" sem pagar o "custo" da execução real
    ↓
Neste caso: Vemos qual método foi chamado sem precisar do banco de dados
```

</details>

### 🏆 Desafio Extra

Tente responder:

1. **Se mudássemos a rota para `@GetMapping("/{customerId}/details")`**, 
   o proxy automaticamente construiria a URL correta? Por quê?

2. **Qual é o benefício de usar proxy aqui em vez de construir a string manualmente?**
   ```java
   // Sem proxy (ruim):
   String url = "/api/v1/customers/" + customerId;
   
   // Com proxy (bom):
   MvcUriComponentsBuilder.on(...).findById(customerId)
   ```

---

## 🏆 RESUMO FINAL (Mapa Mental)

```
┌─────────────────────────────────────────────────────┐
│                    PROXY PATTERN                    │
├─────────────────────────────────────────────────────┤
│                                                     │
│  O QUÊ?                                             │
│  └─ Intermediário que "observa" sem executar        │
│                                                     │
│  ONDE?                                              │
│  └─ Entre você e o "trabalho real"                  │
│                                                     │
│  POR QUÊ?                                           │
│  └─ Capturar informações, logs, validações          │
│                                                     │
│  COMO?                                              │
│  └─ Cria um objeto fake que intercepta chamadas     │
│                                                     │
│  NESTE CASO (MvcUriComponentsBuilder)?              │
│  └─ Captura qual método foi chamado                 │
│     e com quais parâmetros → Gera URL              │
│                                                     │
└─────────────────────────────────────────────────────┘
```

### Fluxo Visual

```
Customer criado
    ↓
Proxy captura: "Quer chamar findById(customerId)"
    ↓
Extrai: Método + Parâmetros + Anotações (@GetMapping)
    ↓
Constrói: /api/v1/customers/{customerId}
    ↓
Retorna no header Location
```

---

## 📖 Leitura Complementar

### Padrão Proxy na Literatura

O **Proxy Pattern** é um dos 23 padrões de design do Gang of Four (GoF). Ele é classificado como um padrão estrutural e tem muitas aplicações:

- **Remote Proxy**: Representa um objeto remoto (ex: serviços web)
- **Virtual Proxy**: Adia a criação de objetos pesados
- **Protection Proxy**: Controla acesso a um objeto sensível
- **Smart Reference**: Adiciona lógica quando o objeto é referenciado

### Contexto Spring

Spring usa proxies amplamente:

- **AOP (Aspect-Oriented Programming)**: Intercepta chamadas de métodos
- **Transações**: `@Transactional` usa proxy para gerenciar transações
- **Security**: `@Secured` usa proxy para validar permissões
- **MvcUriComponentsBuilder**: Usa proxy para capturar chamadas de métodos

---

## 🎯 Próximos Passos

Agora que você entende proxy, você pode:

1. ✅ Entender melhor como funciona `@Transactional` no Spring
2. ✅ Compreender o conceito de AOP (Aspect-Oriented Programming)
3. ✅ Aplicar proxy pattern em seus próprios projetos
4. ✅ Debugging: Saber por que `this.method()` não funciona com proxies

---

**Criado em**: 2026-09-29  
**Contexto**: CustomerController - MvcUriComponentsBuilder  
**Nível**: Iniciante ➜ Intermediário
