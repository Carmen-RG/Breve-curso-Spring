# 3- Introducción a la configuración de metadatos basada en XML (Spring): nombrar, instanciar y configurar las dependencias de los *beans*

## Metadatos en varios archivos XML
Pueden cargarse varios archivos XML en el constructor del `ApplicationContext` (ver cuaderno 2) o pueden utilizarse `<import resource="..."></import>` para cargar definiciones de `beans` en un archivo XML de otros archivos. Ejemplo: 

    <beans>
        <import resource="services.xml"/>
        <import resource="resources/messageSource.xml"/>
        <import resource="/resources/themeSource.xml"/>

        <bean id="bean1" class="..."/>
        <bean id="bean2" class="..."/>
    </beans>


## Nombrar los `beans`
Cada `bean` tiene uno o varios identificadores que deben ser únicos en el contenedor IoC que los contiene. Normalmente los `beans` solo tienen un identificador; si necesita más de uno, el resto se consideran alias.

Para especificar los identificadores de los `beans` se utiliza el atributo `id`, el atributo `name` o ambos.

- `id` permite especificar un único identificador. Suelen ser identificadores alfanuméricos, pero también pueden contener caracteres especiales.

- `name` permite especificar otros alias separados por coma (`,`), punto y coma (`;`) o un espacio en blanco.

## Instanciar los `beans`
Una definición de un `bean` es un modelo para crear uno o más objetos. El contenedor IoC toma de referencia el modelo para un `bean` específico y utiliza los metadatos encapsulados en la definición de ese `bean` para crear o adquirir un objeto. 

La clase de un `bean` se especifica con el atributo `class`.

Las clases anidadas dentro de otras clases se referencian con el símbolo del dólar (`$`) o con un punto (`.`). Si tuviéramos una clase `Something` en el paquete `com.example`, y esta clase tuviera una clase estática andidada llamada `OtherThing`, la referenciaríamos como `com.example.SomeThing$OtherThing` o `com.example.SomeThing.OtherThing`.

## Configuración de dependencias
### Inyección de dependencias basada en el constructor
Los parámetros del constructor se resuelven utilizando el tipo de dato de los argumentos. Si no existe ambiguidad en los parámetros del constructor en una definición de un `bean`, la siguiente configuración funciona:

- Para la siguiente clase (Java): 

        package x.y;

        public class ThingOne {

            public ThingOne(ThingTwo thingTwo, ThingThree thingThree) {
                // ...
            }
        }

- La siguiente configuración, especificando los parámetros con `constructor-arg` y referenciando el `id` de los otros `beans` con el atributo `ref`:

        <beans>
            <bean id="beanOne" class="x.y.ThingOne">
                <constructor-arg ref="beanTwo"/>
                <constructor-arg ref="beanThree"/>
            </bean>

            <bean id="beanTwo" class="x.y.ThingTwo"/>

            <bean id="beanThree" class="x.y.ThingThree"/>
        </beans>


Sin embargo, cuando los argumentos del constructor son tipos de datos primitivos, hay que ser más específico. Para la  siguiente clase (Java): 

    package examples;

    public class ExampleBean {

        // Number of years to calculate the Ultimate Answer
        private final int years;

        // The Answer to Life, the Universe, and Everything
        private final String ultimateAnswer;

        public ExampleBean(int years, String ultimateAnswer) {
            this.years = years;
            this.ultimateAnswer = ultimateAnswer;
        }
    }

Podemos especificar, siempre proporcionando el valor del parámetro entre comillas `" "` como valor del atributo `value`:

- El tipo de dato con el atributo `type`:

        <bean id="exampleBean" class="examples.ExampleBean">
            <constructor-arg type="int" value="7500000"/>
            <constructor-arg type="java.lang.String" value="42"/>
        </bean>

- El índice de los parámetros con el atributo `index`:
  
        <bean id="exampleBean" class="examples.ExampleBean">
            <constructor-arg index="0" value="7500000"/>
            <constructor-arg index="1" value="42"/>
        </bean>

- El nombre del parámetro con el atributo `name`:

        <bean id="exampleBean" class="examples.ExampleBean">
            <constructor-arg name="years" value="7500000"/>
            <constructor-arg name="ultimateAnswer" value="42"/>
        </bean>

Para que este último funcione debemos compilar el código con la bandera de depuración activada o anotar la clase con `@ConstructorProperties` para nombrar explícitamente los parámetros del constructor (Java):

    package examples;

    public class ExampleBean {

        // Fields omitted

        @ConstructorProperties({"years", "ultimateAnswer"})
        public ExampleBean(int years, String ultimateAnswer) {
            this.years = years;
            this.ultimateAnswer = ultimateAnswer;
        }
    }

### Inyección de dependencias basada en *setters*
Debemos indicar cada propiedad con la etiqueta `<property>`, a la que le daremos un `name`, y utilizaremos `<ref>` para referenciar el id de ese otro `bean`. Para la siguiente clase (Java):

    public class ExampleBean {

        private AnotherBean beanOne;

        private YetAnotherBean beanTwo;

        private int i;

        public void setBeanOne(AnotherBean beanOne) {
            this.beanOne = beanOne;
        }

        public void setBeanTwo(YetAnotherBean beanTwo) {
            this.beanTwo = beanTwo;
        }

        public void setIntegerProperty(int i) {
            this.i = i;
        }
    }

Funcionaría la siguiente configuración: 

    <bean id="exampleBean" class="examples.ExampleBean">
        <!-- setter injection using the nested ref element -->
        <property name="beanOne">
            <ref bean="anotherExampleBean"/>
        </property>

        <!-- setter injection using the neater ref attribute -->
        <property name="beanTwo" ref="yetAnotherBean"/>
        <property name="integerProperty" value="1"/>
    </bean>

    <bean id="anotherExampleBean" class="examples.AnotherBean"/>
    <bean id="yetAnotherBean" class="examples.YetAnotherBean"/>

**A tener en cuenta:** `ref`, como atributo, solo acepta como valor el `id` del `bean` al que referencia; `<ref>`, como etiqueta, acepta `bean` como atributo y el `id` del `bean` como valor de ese atributo. 

***

Según: https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-definition