# baozi-store

API REST de uma loja de baozi, feita em **Java 17 + Spring Boot 3** com **Maven** e **MySQL**.
Trabalho de faculdade — arquitetura mínima em três camadas: `model`, `repository`, `controller`.

## Requisitos

- JDK 17
- Maven 3.9+
- MySQL 8 rodando em `localhost:3306` (ou use o perfil H2, abaixo)

## Configuração do banco

A aplicacao conecta como `root` em `localhost:3306`. A senha vem da variavel de ambiente
`MYSQL_PASSWORD`, com `baozi123` como valor padrao:

```properties
spring.datasource.url=jdbc:mysql://localhost:3306/baozi_store?createDatabaseIfNotExist=true&useSSL=false&serverTimezone=UTC&allowPublicKeyRetrieval=true
spring.datasource.username=root
spring.datasource.password=${MYSQL_PASSWORD:baozi123}
```

Se a senha do seu `root` for outra, exporte a variavel antes de subir a aplicacao:

```bash
export MYSQL_PASSWORD='sua_senha'
mvn spring-boot:run
```

Se o `root` do seu MySQL ainda usa `auth_socket` (erro `1698 Access denied`), defina uma senha:

```bash
sudo mysql -e "ALTER USER 'root'@'localhost' IDENTIFIED WITH caching_sha2_password BY 'baozi123'; FLUSH PRIVILEGES;"
```

No WSL2 o servico nao sobe sozinho no boot: `sudo service mysql start`.

O banco `baozi_store` é criado automaticamente (`createDatabaseIfNotExist=true`) e as tabelas
são geradas pelo Hibernate via `spring.jpa.hibernate.ddl-auto=update`.
Se preferir criar o banco na mão:

```sql
CREATE DATABASE baozi_store;
```

### Rodar sem MySQL (H2 em memória)

O `application.properties` já traz o bloco do H2 comentado, e existe um perfil pronto:

```bash
mvn spring-boot:run -Dspring-boot.run.profiles=h2
```

Console do H2: <http://localhost:8080/h2-console> (JDBC URL `jdbc:h2:mem:baozi_store`, usuário `sa`, senha vazia).

## Como executar

```bash
mvn clean compile      # compila
mvn spring-boot:run    # sobe a API em http://localhost:8080
mvn clean package      # gera target/baozi-store-0.0.1-SNAPSHOT.jar
java -jar target/baozi-store-0.0.1-SNAPSHOT.jar
```

## Estrutura

```
src/main/java/com/baozi/store
├── BaoziStoreApplication.java
├── model/         Produto, Cliente, Pedido        (@Entity)
├── repository/    ProdutoRepository, ClienteRepository, PedidoRepository (JpaRepository)
└── controller/    ProdutoController, ClienteController, PedidoController (@RestController)
```

## Entidades

| Produto | Cliente | Pedido |
|---|---|---|
| `id` (Long) | `id` (Long) | `id` (Long) |
| `nome` (String) | `nome` (String) | `clienteId` (Long) |
| `preco` (BigDecimal) | `clienteDesde` (LocalDate) | `produtoId` (Long) |
| `estoque` (Boolean) | | `quantidade` (Integer) |

Os IDs em `Pedido` são colunas `Long` simples, sem `@ManyToOne`.

## Endpoints

Todos aceitam e devolvem JSON. Troque `{recurso}` por `produtos`, `clientes` ou `pedidos`.

| Método | Rota | Descrição | Sucesso | Erro |
|---|---|---|---|---|
| POST | `/{recurso}` | Cria um registro | `201 Created` + corpo | — |
| GET | `/{recurso}` | Lista todos | `200 OK` | — |
| GET | `/{recurso}/{id}` | Busca por id | `200 OK` | `404 Not Found` |
| PUT | `/{recurso}/{id}` | Atualiza | `200 OK` + corpo | `404 Not Found` |
| DELETE | `/{recurso}/{id}` | Apaga | `204 No Content` | `404 Not Found` |

### Exemplos com curl

```bash
# Cliente
curl -i -X POST http://localhost:8080/clientes \
  -H "Content-Type: application/json" \
  -d '{"nome":"Pedro5080840","clienteDesde":"2026-08-24"}'

# Produto
curl -i -X POST http://localhost:8080/produtos \
  -H "Content-Type: application/json" \
  -d '{"nome":"Baozi de Porco","preco":8.50,"estoque":true}'

# Pedido
curl -i -X POST http://localhost:8080/pedidos \
  -H "Content-Type: application/json" \
  -d '{"clienteId":1,"produtoId":1,"quantidade":12}'

# Listar / buscar / atualizar / apagar
curl -i http://localhost:8080/produtos
curl -i http://localhost:8080/produtos/1
curl -i -X PUT http://localhost:8080/produtos/1 \
  -H "Content-Type: application/json" \
  -d '{"nome":"Baozi de Porco","preco":9.90,"estoque":false}'
curl -i -X DELETE http://localhost:8080/produtos/1
```

### Respostas de exemplo

```json
POST /clientes  -> 201
{"id":1,"nome":"Pedro5080840","clienteDesde":"2026-08-24"}

POST /produtos  -> 201
{"id":1,"nome":"Baozi de Porco","preco":8.50,"estoque":true}

POST /pedidos   -> 201
{"id":1,"clienteId":1,"produtoId":1,"quantidade":12}
```

## Dependências

`spring-boot-starter-web`, `spring-boot-starter-data-jpa`, `mysql-connector-j`,
`spring-boot-devtools` e `h2` (apenas em runtime, para o perfil de testes sem MySQL).
Sem Lombok — todos os getters/setters são explícitos.
