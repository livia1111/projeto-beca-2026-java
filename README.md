# MS1 - Gestão de Usuários (Arquitetura Hexagonal)

## Objetivo do projeto

Este serviço implementa operações de gestão de usuários com foco em **separação de responsabilidades**, **isolamento do domínio** e **evolução segura** do código.  
A base do desenho segue **arquitetura hexagonal (Ports & Adapters)**, permitindo trocar detalhes de infraestrutura sem contaminar regras de negócio.

---

## Decisões de arquitetura

## 1) Isolamento do domínio

A camada de domínio foi modelada para concentrar regras e invariantes de negócio, evitando acoplamento com framework.  
Isso reduz dependências acidentais e facilita testes de regra.

- Entidade principal: `User`
- Value Objects:
  - `UserId`
  - `UserName`
  - `UserEmail`
  - `UserPassword`
- Exceções de domínio:
  - `DomainException`
  - `DuplicateEmailException`
  - `UserNotFoundException`

**Benefício:** o núcleo de negócio se mantém estável mesmo quando adaptadores (web, banco, segurança) mudam.

---

## 2) Arquitetura Hexagonal (Ports & Adapters)

A aplicação está organizada em:

- **Ports (interfaces)**
  - Entrada (`application.port.in`): define os casos de uso expostos.
  - Saída (`application.port.out`): define contratos para infraestrutura (repositório, encoder etc.).
- **Application Services**
  - Implementam os casos de uso e orquestram fluxo de negócio.
- **Adapters**
  - `adapter.in.web`: entrada HTTP (controllers, DTOs, handlers).
  - `adapter.out.persistence`: persistência (JPA/Spring Data).
  - `adapter.out.security`: detalhes de segurança/criptografia (ex.: BCrypt).

**Benefício:** contratos claros entre camadas, baixo acoplamento e maior facilidade para testes/mocks.

---

## Construtores de `User` e integridade do estado

A modelagem de `User` prioriza criação/reconstrução controlada para evitar estado inválido.

- Fluxo de **criação de novo usuário**: garante aplicação das regras exigidas para registro.
- Fluxo de **reconstrução de usuário existente** (hidratação/persistência): mantém consistência com dados já persistidos.

Esse padrão evita “montagem livre” da entidade e reforça invariantes do domínio desde o nascimento do objeto.

---

## Value Objects (VOs)

`UserId`, `UserName`, `UserEmail` e `UserPassword` encapsulam significado e validações próprias.

### Por que usar VO aqui?
- Evita passar `String` genérica para tudo.
- Centraliza validações em um único lugar.
- Reduz duplicidade de regra no código.
- Melhora legibilidade semântica (tipo carrega intenção).

### Resultado prático
Em vez de validar email em múltiplos pontos, o projeto tende a validar onde o VO é criado, mantendo a regra consistente.

---

## Tratamento de erros

O tratamento segue duas etapas:

1. **Domínio/aplicação lançam exceções semânticas** (`DuplicateEmailException`, `UserNotFoundException`, etc.).
2. **Camada web traduz para HTTP** por meio de `GlobalExceptionHandler`.

Com isso:
- regra de negócio não depende de HTTP;
- API mantém respostas de erro previsíveis;
- facilita rastreamento e manutenção.

---

## Validações

As validações são distribuídas por responsabilidade:

- **Entrada (DTO/controller):** formato, campos obrigatórios e consistência de request.
- **Domínio/VOs:** invariantes de negócio e integridade semântica.
- **Serviços de aplicação:** validações de fluxo (ex.: existência, unicidade, pré-condições de caso de uso).

Essa separação evita sobrecarga em um único ponto e mantém cada regra no lugar correto.

---

## Status operacional desta fase (MS2 teste)

- **Login/autenticação está desligado nesta fase de testes do MS2** para evitar renovações de token durante desenvolvimento/validação funcional.
- **Docker será entregue por PR de outra máquina**, pois há indisponibilidade no ambiente atual (Rancher com problema).

---

## Como executar localmente

Pré-requisitos:
- Java 17+ (ou versão configurada no `pom.xml`)
- Maven (ou Maven Wrapper do projeto)

### Subir a aplicação
No Windows (cmd):
```bat
mvn spring-boot:run
```

Ou com Maven Wrapper:
```bat
mvnw.cmd spring-boot:run
```

---

## Como rodar testes

```bat
mvn test
```

---

## Estrutura resumida

```text
src/main/java/com/nttdata/ms1
├─ adapter
│  ├─ in/web                 # Controller, DTOs, exception handler HTTP
│  └─ out
│     ├─ persistence         # JPA/Spring Data adapters
│     └─ security            # BCrypt adapter
├─ application
│  ├─ port/in                # Use cases (entrada)
│  ├─ port/out               # Contratos de saída
│  └─ service                # Implementação dos casos de uso
├─ domain
│  ├─ model                  # Entidades e Value Objects
│  └─ exception              # Exceções de domínio
```

---

## Observação sobre segurança

O projeto possui suporte para codificação de senha via adapter (`BcryptPasswordEncoderAdapter`) seguindo port de saída (`PasswordEncoderPort`), mantendo o domínio desacoplado de implementação concreta de segurança.
