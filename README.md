# trbank
Projeto Spring Boot simulando carteiras e transações financeiras em uma API.

## 1. Inicialização do projeto

## 1.1. Tecnologias

- Spring Boot (4.0.6)
- Java (21)
- Maven

## 1.2. Dependências

- Spring Web

Dependência que será utilizada para o desenvolvimento de aplicação web e serviços RESTful;

- Validation

Dependência que será utilizada para validações automáticas (@NotNull, @Email, etc)

- Spring Boot Actuator 

Dependência com endpoints integrados que ajudam a monitorar e gerenciar as aplicações - como sáude da aplicação, métricas, sessões, etc

- Spring Modulith

Dependência para construir aplicação monolito

- Spring Boot DevTools

Dependência para ajudar com LiveReload, reinício de aplicação, etc

## 1.3. Primeiro teste & Application health check-up

```
[INFO] Tests run: 1, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 2.950 s -- in dev.raponi.trbank.TrbankApplicationTests
[INFO] 
[INFO] Results:
[INFO] 
[INFO] Tests run: 1, Failures: 0, Errors: 0, Skipped: 0
[INFO] 
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  5.286 s
[INFO] Finished at: 2026-06-06T16:10:05-03:00
[INFO] ------------------------------------------------------------------------

raponi@MacBook-Pro-de-Tales ~ % curl http://localhost:8080/actuator/health
{"groups":["liveness","readiness"],"status":"UP"}
```