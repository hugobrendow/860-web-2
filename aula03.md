## Aula 03

#### 01. Spring Boot Logging

Adicionando arquivo para visualização de logs.

1. Adicione o conteúdo abaixo no arquivo `pom.xml`:

```properties
logging.level.root = INFO
logging.file.path = src/main/resources/logs
logging.file.name = ${logging.file.path}/spring-letsgoal.log
logging.pattern.console = %d{yyyy-MM-dd HH:mm:ss} %p %c{1} [%t] %m%n
logging.pattern.file = %d{yyyy-MM-dd HH:mm:ss} [%thread] %-5level %logger{36} - %msg%n
```

2. Adicione o conteúdo abaixo no arquivo `PatrocinadorController.java`:

```java
@RestController
@RequestMapping("/patrocinadores")
public class PatrocinadorController {
    Logger logger = LoggerFactory.getLogger(PatrocinadorController.class);
    @GetMapping
    public List<Patrocinador> findAll() {
        logger.info("Acessando recurso do patrocinador");
        Patrocinador patrocinador = new Patrocinador(1L, "Let's Code", "https://letscode.com.br", "https://letscode.com.br", "Let's Code");
        List<Patrocinador> patrocinadores = List.of(patrocinador);
        return patrocinadores;
    }

    @PostMapping
    public Patrocinador savePatrocinador(@RequestBody Patrocinador patrocinador) {
        return patrocinador;
    }

    @GetMapping("/{id}")
    public Patrocinador findById(@PathVariable Long id) {
        Patrocinador patrocinador = new Patrocinador(id, "Let's Code", "https://letscode.com.br", "https://letscode.com.br", "Let's Code");
        return patrocinador;
    }

    @PutMapping("/{id}")
    public Patrocinador updatePatrocinador(@PathVariable Long id,
                                           @RequestBody Patrocinador patrocinador) {
        return patrocinador;
    }
}

```

03. Acesse o recurso `http://localhost:8080/patrocinadores` e confira se o arquivo foi criado no diretório `resources/logs`


#### Configurando Spring Data 

1. O primeiro passo para esta atividade será a criação de um container responsável por disponibilizar o serviço do banco de dados, no terminal execute o seguinte comando para a criação:

```sh
docker run --name postgres -e POSTGRES_PASSWORD=letsgoal -p 5432:5432 -d postgres
```

Com o comando acima estamos criando um container que cuidará do banco de dados e com a senha `letsgoal` na porta `5432`.


2. No arquivo `application.properties` deverá ser acrescentado o conteúdo:

```properties
spring.jpa.database=POSTGRESQL
spring.jpa.show-sql=true
spring.jpa.hibernate.ddl-auto=update
spring.database.driverClassName=org.postgresql.Driver
spring.datasource.url=jdbc:postgresql://localhost:5432/postgres
spring.datasource.username=postgres
spring.datasource.password=letsgoal
```

3. Adicionar as dependências do Spring Data e do Driver do Postgresql no arquivo `pom.xml` entre as tags `<dependencies> </dependencies>`:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
<dependency>
    <groupId>org.postgresql</groupId>
    <artifactId>postgresql</artifactId>
</dependency>
```

4. Nesta etapa iremos criar nossa primeira tabela. Acesse o arquivo `Posicao.java` e inclua:

```java
@Data
@Entity
public class Posicao {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String nome;
}
```

A anotação `@Entity` irá representar uma tabela no banco de dados com o nome `posicao`, a outra anotação em cima do atributo id, `@Id` representa o identificador da tabela a conhecida `Primary Key` no banco de dados, e o `@GeneratedValue(strategy = GenerationType.IDENTITY)` representa o modelo incremental para adicionar um valor no banco de dados para identificador.
