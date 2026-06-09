# ADR-001: Adotar monolito modular organizado por dominio

## Status

Aceito em 9 de junho de 2026.

## Contexto

O TRBank sera uma API REST para gerenciar clientes, carteiras e, em etapas
posteriores, transferencias financeiras.

O sistema sera inicialmente desenvolvido e implantado como uma unica aplicacao
Spring Boot. Apesar de possuir um unico artefato executavel, o codigo precisa
manter limites claros entre as capacidades de negocio para evitar acoplamento
excessivo conforme novas funcionalidades forem adicionadas.

Uma organizacao global por tipos tecnicos, como `controller`, `service` e
`repository`, colocaria classes de dominios diferentes nos mesmos pacotes. Essa
abordagem dificulta a identificacao das responsabilidades de cada dominio e
facilita dependencias indevidas entre suas implementacoes.

A arquitetura hexagonal completa tambem foi considerada. No entanto, sua
adocao imediata exigiria portas, adaptadores e mapeamentos antes de o dominio
apresentar complexidade suficiente para justificar esse custo.

## Decisao

O TRBank adotara um monolito modular organizado por dominio, com uma API REST e
camadas tecnicas internas em cada modulo.

Monolito significa que o sistema sera executado como uma unica aplicacao, em um
unico processo e com um unico artefato de deploy.

Modular significa que o codigo sera agrupado primeiro por capacidades de
negocio, e nao por tipos tecnicos globais. Os modulos iniciais serao:

1. `customer`: cadastro e gerenciamento dos clientes do TRBank.
2. `wallet`: gerenciamento das carteiras pertencentes aos clientes.

O nome `customer` sera usado em vez de `user` para diferenciar o cliente do
dominio financeiro da identidade autenticada pelo Spring Security.

Cada modulo podera conter os seguintes componentes, conforme a necessidade:

1. `controller`: contrato HTTP, validacao da entrada e traducao da resposta.
2. `service`: coordenacao dos casos de uso, regras de negocio e transacoes.
3. `repository`: acesso e persistencia de dados.
4. `entity`: representacao dos dados persistidos.
5. `dto`: contratos de request e response da API.
6. `mapper`: traducao entre DTOs, entidades e outros modelos quando necessaria.

Nao serao criados pacotes, interfaces ou mapeamentos vazios apenas para seguir
um template. A estrutura interna podera evoluir quando o tamanho e a
complexidade de cada modulo justificarem novas separacoes.

## Estrutura de referencia

```text
src/main/java/dev/raponi/trbank
|-- customer
|   |-- CustomerController.java
|   |-- CustomerService.java
|   |-- CustomerRepository.java
|   |-- Customer.java
|   |-- dto
|   |   |-- request
|   |   `-- response
|   `-- mapper
|       `-- CustomerMapper.java
|-- wallet
|   `-- ...
`-- shared
    `-- ... somente para conceitos realmente compartilhados
```

## Regras arquiteturais

1. Controllers devem chamar services e nunca acessar repositories diretamente.
2. Controllers nao devem conter regras de negocio.
3. Services devem coordenar os casos de uso e definir os limites transacionais.
4. Repositories devem conter apenas responsabilidades de acesso aos dados.
5. DTOs de request e response nao devem ser usados como entidades persistidas.
6. Services nao devem depender de DTOs de response HTTP.
7. Um modulo nao deve acessar controllers ou repositories de outro modulo.
8. A comunicacao entre modulos deve ocorrer por services publicos bem definidos
   ou por eventos de aplicacao.
9. O pacote `shared` deve conter apenas conceitos estaveis e realmente
   compartilhados, nao utilitarios especificos de um modulo.
10. Os endpoints devem seguir `/api/v1/{resource}` e usar recursos no plural.
11. Dependencias ciclicas entre modulos nao sao permitidas.
12. Spring Modulith sera usado para identificar os modulos e validar seus
    limites.

## Seguranca

A seguranca sera implementada com Spring Security.

Autenticacao e autorizacao serao aplicadas por meio de `SecurityFilterChain`,
filtros e configuracoes fornecidas pelo framework. Controllers nao deverao
interpretar tokens manualmente.

A autorizacao tecnica podera utilizar roles ou scopes, como `ADMIN`, `CUSTOMER`
e `SUPPORT`. Regras de autorizacao relacionadas ao negocio permanecerao nos
services.

A estrategia completa de autenticacao, emissao e validacao de tokens sera
definida em uma decisao arquitetural posterior.

## Consequencias positivas

1. A organizacao do codigo acompanha as capacidades do negocio.
2. Cada modulo mantem controllers, regras e persistencia proximos entre si.
3. Os limites entre dominios ficam mais visiveis e podem ser verificados.
4. O projeto permanece simples para aprendizado e desenvolvimento inicial.
5. Deploy, testes e operacao sao mais simples do que em uma arquitetura de
   microservicos.
6. Cada modulo pode evoluir para uma estrutura mais rigorosa quando houver
   necessidade concreta.

## Consequencias negativas e trade-offs

1. As camadas tecnicas ainda podem ficar acopladas ao Spring e a tecnologia de
   persistencia.
2. Os limites entre modulos dependem de disciplina e de testes arquiteturais.
3. Services podem crescer excessivamente se os casos de uso nao forem separados
   conforme a complexidade aumentar.
4. A separacao em um unico processo oferece menos isolamento do que
   microservicos.
5. Uma futura migracao para arquitetura hexagonal podera exigir criacao de
   portas, adaptadores e mapeamentos adicionais.

## Alternativas consideradas

### Arquitetura em camadas globais

Foi rejeitada porque pacotes globais de controllers, services e repositories
misturariam dominios diferentes. Isso aumentaria o risco de dependencias
indevidas e tornaria menos visiveis as capacidades de negocio.

### Arquitetura hexagonal

Foi adiada porque adicionaria abstracoes e mapeamentos antes de existirem
integracoes ou regras complexas que justificassem esse custo. Modulos
especificos poderao adota-la posteriormente.

### Microservicos

Foram rejeitados neste momento porque o dominio, a escala e os requisitos
operacionais ainda nao justificam multiplos deploys, bancos independentes e
comunicacao distribuida.

## Criterios de validacao

1. Spring Modulith deve reconhecer os modulos da aplicacao.
2. A verificacao do Spring Modulith nao deve identificar ciclos ou acessos
   indevidos entre modulos.
3. Testes arquiteturais devem impedir controllers de acessarem repositories.
4. Testes arquiteturais devem impedir um modulo de acessar controllers ou
   repositories de outro modulo.
5. Regras de negocio dos services devem possuir testes sem dependencia do
   controller.
6. DTOs, entidades e responsabilidades de persistencia devem permanecer
   separados.
