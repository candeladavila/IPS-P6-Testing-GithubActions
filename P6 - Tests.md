# Practica 6

## Pruebas

### Crear tests

- Estructura básica de un test:

    ```java
    @Test
    @DisplayName("nombre que se muestra al ejecutar los tests")
    void nombreDelTest() throws Exception {
        // código del test
    }
    ```

- Los métodos con `@Test` normalmente **no llevan parámetros**.

  Incorrecto:

    ```java
    @Test
    void testCrearObjeto(Objeto objeto) throws Exception {
    }
    ```

  Correcto:

    ```java
    @Test
    void testCrearObjeto() throws Exception {
    }
    ```

---

### Tests con MockMvc

- Se usa `MockMvc` para probar controladores sin levantar un servidor real.
- Es útil para peticiones normales tipo `GET`, `POST`, `PUT`, `DELETE` con JSON.
- Normalmente se declara así:

    ```java
    @Autowired
    private MockMvc mockMvc;

    @Autowired
    private ObjectMapper objectMapper;
    ```

- Estructura general con `MockMvc`:

    ```java
    mockMvc.perform(get("/url/{id}", id))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$.atributo").value("valorEsperado"));
    ```

- Si enviamos un objeto en formato JSON, por ejemplo con `POST` o `PUT`:

    ```java
    mockMvc.perform(post("/url")
            .contentType(MediaType.APPLICATION_JSON)
            .content(objectMapper.writeValueAsString(objeto)))
            .andExpect(status().isCreated());
    ```

- Para actualizar con `PUT`:

    ```java
    mockMvc.perform(put("/url")
            .contentType(MediaType.APPLICATION_JSON)
            .content(objectMapper.writeValueAsString(objeto)))
            .andExpect(status().isNoContent());
    ```

- Para eliminar con `DELETE`:

    ```java
    mockMvc.perform(delete("/url/{id}", id))
            .andExpect(status().isOk());
    ```

- Ideas importantes:
  - `contentType(MediaType.APPLICATION_JSON)` indica que mandamos JSON.
  - `objectMapper.writeValueAsString(objeto)` convierte el objeto Java a JSON.
  - `jsonPath("$.atributo")` comprueba campos concretos de la respuesta JSON.
  - En un `GET` normalmente no hace falta poner `contentType` ni `content`.

---

### Tests con WebTestClient

- Se usa `WebTestClient` cuando necesitamos probar la aplicación con un servidor real de prueba.
- Suele usarse cuando:
  - Hay subida de ficheros.
  - Hay peticiones `multipart/form-data`.
  - Se quiere probar un flujo más completo de integración.
  - El controlador necesita que la aplicación esté levantada en un puerto.

- Para usarlo, la clase suele tener:

    ```java
    @SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
    public class NombreDelTest extends AbstractIntegration {
    }
    ```

- También se necesita el puerto aleatorio:

    ```java
    @LocalServerPort
    private Integer port;
    ```

- Y se crea el cliente:

    ```java
    private WebTestClient testClient;

    @PostConstruct
    public void init() {
        testClient = WebTestClient.bindToServer()
                .baseUrl("http://localhost:" + port)
                .build();
    }
    ```

- Estructura general de una petición con `WebTestClient`:

    ```java
    testClient.get().uri("/url/{id}", id)
            .exchange()
            .expectStatus().isOk()
            .expectBody()
            .jsonPath("$.atributo").isEqualTo("valorEsperado");
    ```

- Para enviar un objeto como JSON:

    ```java
    testClient.post().uri("/url")
            .contentType(MediaType.APPLICATION_JSON)
            .body(Mono.just(objeto), Objeto.class)
            .exchange()
            .expectStatus().isCreated();
    ```

- Para actualizar:

    ```java
    testClient.put().uri("/url")
            .contentType(MediaType.APPLICATION_JSON)
            .body(Mono.just(objeto), Objeto.class)
            .exchange()
            .expectStatus().isNoContent();
    ```

- Para eliminar:

    ```java
    testClient.delete().uri("/url/{id}", id)
            .exchange()
            .expectStatus().isOk();
    ```

---

### Subida de ficheros con WebTestClient

- Para enviar ficheros se usa `MultipartBodyBuilder`.
- Esto se usa cuando el controlador recibe `multipart/form-data`.

    ```java
    MultipartBodyBuilder builder = new MultipartBodyBuilder();

    builder.part("nombreDelCampo",
            new FileSystemResource(Paths.get("src/test/resources/fichero.png").toFile()));

    testClient.post().uri("/url")
            .contentType(MediaType.MULTIPART_FORM_DATA)
            .body(BodyInserters.fromMultipartData(builder.build()))
            .exchange()
            .expectStatus().isOk();
    ```

- Si además hay que enviar otro objeto junto al fichero:

    ```java
    builder.part("objeto", objeto);
    ```

---

### Diferencia entre MockMvc y WebTestClient

- `MockMvc`:
  - No levanta servidor real.
  - Simula peticiones HTTP.
  - Más simple.
  - Adecuado para controladores con JSON normal.

- `WebTestClient`:
  - Usa un servidor de prueba con puerto aleatorio.
  - Se configura con `RANDOM_PORT`.
  - Más completo.
  - Adecuado para multipart, ficheros o pruebas de integración más reales.

---

### Comprobaciones habituales

- Comprobar código de estado:

    ```java
    .andExpect(status().isOk());
    .andExpect(status().isCreated());
    .andExpect(status().isNoContent());
    ```

  Con `WebTestClient`:

    ```java
    .expectStatus().isOk();
    .expectStatus().isCreated();
    .expectStatus().isNoContent();
    ```

- Comprobar campos JSON:

    ```java
    .andExpect(jsonPath("$.nombre").value("valor"));
    ```

  Con `WebTestClient`:

    ```java
    .expectBody()
    .jsonPath("$.nombre").isEqualTo("valor");
    ```

- Comprobar listas JSON:

    ```java
    .andExpect(jsonPath("$[0].nombre").value("valor"));
    ```

  Con `WebTestClient`:

    ```java
    .expectBody()
    .jsonPath("$[0].nombre").isEqualTo("valor");
    ```

---

### Comandos útiles

- Ejecutar todos los tests:

    ```bash
    ./mvnw verify --no-transfer-progress
    ```

- Ejecutar solo una clase de tests de integración:

    ```bash
    ./mvnw -Dit.test=NombreDeLaClaseIT verify
    ```