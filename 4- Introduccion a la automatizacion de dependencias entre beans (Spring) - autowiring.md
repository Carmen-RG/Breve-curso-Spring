# 4- Introducción a la automatización de las dependencias de los *beans* (Spring): *autowiring*
El contenedor IoC de Spring puede configurar de forma automática (en adelante *autowire*) las relaciones entre `beans` colaboradores o dependientes.

## Ventajas del *autowiring*

- Puede reducir de forma significativa la necesidad de especificar propiedades o parámetros del constructor.

- Puede actualizar la configuración a medida que los objetos evolucionan. Por ejemplo, si añadimos una dependencia a una clase, esa dependencia se resolverá automáticamente sin necesidad de especificarlo en la configuración. 

## Desventajas y limitaciones del *autowiring*
El *autowiring* funciona mejor si se utiliza de forma consistente en un proyecto.

Las limitaciones y desventajas son:

- Las dependencias explícitas especificadas en la configuración de `property` y `constructor-arg` siempre sobreescriben el *autowiring*. Las propiedades simples (datos primitivos, Strings, clases) no pueden configurarse con *autowire*.

- Es menos preciso que la configuración explícita.

- Puede que varias definiciones de `beans` del contenedor IoC coincidan con el tipo especificado por el *setter* o por el parámetro del constructor para ser *autowired*. Si hay ambigüidad y no hay una única definición de `bean` disponible, se lanza una excepción. Posibles soluciones:
  
  - Abandonar el *autowiring* por la configuración explícita.
  
  - Evitar el *autowiring* para una definición de un `bean` concreta: estableciendo su atributo `autowire-candidate` a `false` (en los metadatos XML).
  
  - Designar una única definición de `bean` como candidato principal: estableciendo su atributo `primary` como `true` (en los metadatos XML).
  
  - Implementar un control más explícito con la configuración basada en anotaciones de Java.

## Formas de *autowiring*

El *autowiring* puede especificarse en los metadatos XML o con anotaciones de Java.

### XML
Puede activarse el modo *autowiring* con el atributo `autowire` del elemento `<bean/>`. Este atributo tiene cuatro modos:

- `no`: el *autowiring* está desactivado. Es el valor por defecto.

- `byName`: *autowiring* por el nombre de las propiedades.

- `byType`: *autowiring* por el tipo de dato de las propiedades (inyección de dependencias basadas en *setters*). (Ver el último punto del apartado de desventajas y limitaciones).

- `constructor`: análogo a `byType` aplicado a los aprámetros del constructor.

### Anotaciones de Java
#### `@AutoWired` e `@Inject`
Puede utilizarse la anotación `@Autowired` o `@Inject`. Pueden aplicarse a:

- Constructores. Ejemplo (Java):

        public class MovieRecommender {

            private final CustomerPreferenceDao customerPreferenceDao;

            @Autowired
            public MovieRecommender(CustomerPreferenceDao customerPreferenceDao) {
                this.customerPreferenceDao = customerPreferenceDao;
            }

            // ...
        }

- *Setters*. Ejemplo (Java):

        public class SimpleMovieLister {

            private MovieFinder movieFinder;

            @Autowired
            public void setMovieFinder(MovieFinder movieFinder) {
                this.movieFinder = movieFinder;
            }

            // ...
        }

- Métodos y variables (se consideran dependencias requeridas). Ejemplos (Java):
- 
        public class MovieRecommender {

            private MovieCatalog movieCatalog;

            private CustomerPreferenceDao customerPreferenceDao;

            @Autowired
            public void prepare(MovieCatalog movieCatalog,
                    CustomerPreferenceDao customerPreferenceDao) {
                this.movieCatalog = movieCatalog;
                this.customerPreferenceDao = customerPreferenceDao;
            }

            // ...
        }

-
        public class MovieRecommender {

            private final CustomerPreferenceDao customerPreferenceDao;

            @Autowired
            private MovieCatalog movieCatalog;

            @Autowired
            public MovieRecommender(CustomerPreferenceDao customerPreferenceDao) {
                this.customerPreferenceDao = customerPreferenceDao;
            }

            // ...
        }

#### `@Primary`
Para una configuración más precisa existe la anotación `@Primary`. Indica que un `bean` particular tiene preferencia sobre los demás cuando múltiples `beans` sean candidatos de *autowiring*, solucionando el problema visto en el último punto del apartado sobre limitaciones y desventajas. Ejemplo (Java):

- Con la siguiente configuración:

        @Configuration
        public class MovieConfiguration {

            @Bean
            @Primary
            public MovieCatalog firstMovieCatalog() { ... }

            @Bean
            public MovieCatalog secondMovieCatalog() { ... }

            // ...
        }

- El siguiente `MovieRecommender` es *autowired* con `firstMovieCatalog`:

        public class MovieRecommender {

            @Autowired
            private MovieCatalog movieCatalog;

            // ...
        }

#### `@Qualifier`
`@Qualifier` permite aún más control que la anotación `@Primary`. Podemos asociar valores del *qualifier* con parámetros específicos (de un constructor o de un método), acotando los tipos que coinciden. Ejemplo (Java):

    public class MovieRecommender {

        @Autowired
        @Qualifier("main")
        private MovieCatalog movieCatalog;

        // ...
    }

Para la siguiente configuración de `beans`:

    <beans>

        <context:annotation-config/>

        <bean class="example.SimpleMovieCatalog">
            <qualifier value="main"/> 

            <!-- inject any dependencies required by this bean -->
        </bean>

        <bean class="example.SimpleMovieCatalog">
            <qualifier value="action"/> 

            <!-- inject any dependencies required by this bean -->
        </bean>

        <bean id="movieRecommender" class="example.MovieRecommender"/>

    </beans>

El `bean` con el valor *qualifier* `main` se asocia con el parámetro del constructor que tiene el mismo valor *qualified*.

***

Según: https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-autowired-annotation