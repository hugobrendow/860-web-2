## Consumindo API do Cartola

1. O primeiro passo será criar uma classe que irá representar o objeto que será retornado no consumo da `API` de Clubes, será criado no pacote `cartola/dto` com o nome `EscudoCartolaDTO.java`:

```java
@Data
@AllArgsConstructor
@NoArgsConstructor
public class EscudoCartolaDTO {
    private String tamanho60x60;
    private String tamanho45x45;
    private String tamanho30x30;
}

```

2. O segundo passo será criar uma classe que irá representar o objeto que será retornado no consumo da `API` de Clubes, será criado no pacote `cartola/dto` com o nome `ClubeCartolaDTO.java`:

```java
@Data
@AllArgsConstructor
@NoArgsConstructor
public class ClubeCartolaDTO {
    private Long id;
    private String nome;
    private String abreviacao;
    private Escudo escudo;
    private String nomeFantasia;
}
```

3. O terceiro passo será criar uma classe que irá representar o objeto que será retornado no consumo da `API` de Jogadores, será criado no pacote `cartola/dto` com o nome `JogadorCartolaDTO.java`:

```java
@Data
@AllArgsConstructor
@NoArgsConstructor
public class JogadorCartolaDTO {
    private Long id;
    private Long clubeId;
    private Long posicaoId;
    private Long statusId;
    private BigDecimal preco;
    private Long media;
    private String slug;
    private String apelido;
    private String apelidoAbreviado;
    private String nome;
    private String foto;

}
```

4. Criar uma classe que irá realizar as chamadas para a API do Cartola, criar a classe `ClubeCartolaClient.java` no pacote `cartola/client`:

```java
@Service
public class ClubeCartolaClient {
    private static final String URL = "https://api.cartola.globo.com";

    public List<ClubeCartolaDTO> listarClubes() throws IOException, InterruptedException {
        HttpClient httpClient = HttpClient.newHttpClient();
        HttpRequest request = HttpRequest.newBuilder()
                .uri(URI.create(URL + "/clubes"))
                .header("User-Agent", "PostmanRuntime/7.26.2")
                .build();

        HttpResponse<String> response = httpClient.send(request, HttpResponse.BodyHandlers.ofString());

        ObjectMapper mapper = new ObjectMapper();
        JsonNode node = mapper.readTree(response.body());
        return extrairClubes(node);
    }

    private static List<ClubeCartolaDTO> extrairClubes(JsonNode root) {
        List<ClubeCartolaDTO> clubes = new ArrayList<>();
        if (root.isObject()) {
            extrairClube(root, clubes);
        }
        return clubes;
    }

    private static void extrairClube(JsonNode root, List<ClubeCartolaDTO> clubes) {
        Iterator<String> fieldNames = root.fieldNames();
        while (fieldNames.hasNext()) {
            String fieldName = fieldNames.next();
            JsonNode fieldValue = root.get(fieldName);
            if (fieldName.equals("id")) {
                EscudoCartolaDTO escudo = new EscudoCartolaDTO(root.get("escudos").get("60x60").textValue(), root.get("escudos").get("45x45").textValue(), root.get("escudos").get("30x30").textValue());
                clubes.add(new ClubeCartolaDTO(root.get("id").longValue(), root.get("nome").textValue(),
                        root.get("abreviacao").textValue(), escudo,
                        root.get("nome_fantasia").textValue()));
            } else {
                extrairClube(fieldValue, clubes);
            }
        }
    }
}
```

5. Criar uma classe que irá realizar as chamadas para a API do Cartola para buscar jogadores, criar a classe `JogadorCartolaClient.java` no pacote `cartola/client`:

```java
@Service
public class JogadorCartolaClient {
    private static final String URL = "https://api.cartola.globo.com";

    public List<JogadorCartolaDTO> listarJogadores() throws IOException, InterruptedException {
        HttpClient httpClient = HttpClient.newHttpClient();
        HttpRequest request = HttpRequest.newBuilder()
                .uri(URI.create(URL + "/atletas/mercado"))
                .header("User-Agent", "PostmanRuntime/7.26.2")
                .build();

        HttpResponse<String> response = httpClient.send(request, HttpResponse.BodyHandlers.ofString());

        ObjectMapper mapper = new ObjectMapper();
        JsonNode node = mapper.readTree(response.body());
        return extrairJogador(node.get("atletas"));
    }
    private static List<JogadorCartolaDTO> extrairJogador(JsonNode root) {
        List<JogadorCartolaDTO> jogadores = new ArrayList<>();
        root.forEach(obj -> {
            jogadores.add(new JogadorCartolaDTO(
                    obj.get("atleta_id").longValue(), obj.get("clube_id").longValue(),
                    obj.get("posicao_id").longValue(), obj.get("status_id").longValue(),
                    BigDecimal.valueOf(obj.get("preco_num").longValue()),
                    obj.get("media_num").longValue(),
                    obj.get("slug").textValue(),
                    obj.get("apelido").textValue(),
                    obj.get("apelido_abreviado").textValue(),
                    obj.get("nome").textValue(),
                    obj.get("foto").textValue()
            ));
        });
        return jogadores;
    }
}
```

5. Criar um arquivo `CarregarInformacoes.java` no pacote `service` com o conteúdo abaixo:

```java
@Component
@AllArgsConstructor
public class CarregarInformacoes implements ApplicationRunner {

    ClubeService clubeService;
    ClubeCartolaClient clubeCartolaClient;

    @Override
    public void run(ApplicationArguments args) throws Exception {
        if (clubeService.listar().isEmpty()) {
            List<ClubeCartolaDTO> clubeCartolaDTO = clubeCartolaClient.listarClubes();
            List<Clube> clubes = clubeCartolaDTO
                    .stream()
                    .map(objeto -> {
                        EscudoCartolaDTO escudoDTO = objeto.getEscudo();
                        Escudo escudo = new Escudo();
                        BeanUtils.copyProperties(escudoDTO, escudo);
                        return new Clube(
                                null,
                                objeto.getNome(),
                                objeto.getAbreviacao(),
                                objeto.getNomeFantasia(),
                                escudo);
                    }).collect(Collectors.toList());
            clubeService.salvarTodos(clubes);
        }

        // TODO implementar a regra para salvar jogadores
    }
}
```