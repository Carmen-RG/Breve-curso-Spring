# 2 - Introducción a los metadatos de configuración en Spring

Los metadatos de la configuración que consume el contenedor IoC de Spring pueden representarse en XML o en anotaciones de Java.

## Configuración de los metadatos

### XML

Los metadatos basados en XML configuran los *beans* como elementos `<bean></bean>` dentro de un elemento `<beans></beans>`. 

Estructura básica de los metadatos de configuración basados en XML: 

    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://www.springframework.org/schema/beans
            https://www.springframework.org/schema/beans/spring-beans.xsd">

        <bean id="..." class="...">  
            <!-- collaborators and configuration for this bean go here -->
        </bean>

        <bean id="..." class="...">
            <!-- collaborators and configuration for this bean go here -->
        </bean>

        <!-- more bean definitions go here -->

    </beans>

- El atributo `id` es un String que identifica la definición individual de cada `bean`.

- El atributo `class` define el tipo de dato del `bean` y utiliza el nombre completo de la clase.


### Anotaciones de Java

Los metadatos basados en anotaciones de Java configuran los *beans* como anotaciones `@Bean` en métodos de clases anotadas como `@Configuration`.


## Principales formas de instanciación del contenedor IoC

El contenedor IoC de Spring está formado por los paquetes básicos:

- `org.springframework.beans`.

- `org.springframework.context`.

- Interfaz `BeanFactory`: funcionalidad básica.

- Interfaz `ApplicationContext` (subinterfaz de `BeanFactory`): funcionalidad adicional.

La interfaz `org.springframework.context.ApplicationContext` representa el contenedor IoC de Spring. 

### Instanciación del contenedor IoC con metadatos en XML

Para instanciar el contenedor IoC se le proporcionan a la clase `ApplicationContext` uno o varios String que permiten que el contenedor cargue metadatos de recursos externos (el sistema de archivos local, el `CLASSPATH` de Java, etc).

Ejemplo de instanciación del contenedor IoC:

    ApplicationContext context = new ClassPathXmlApplicationContext("services.xml", "daos.xml");

Ejemplo del archivo de configuración `services.xml` referenciado en el ejemplo anterior:

    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <!-- services -->

    <bean id="petStore" class="org.springframework.samples.jpetstore.services.PetStoreServiceImpl">
        <property name="accountDao" ref="accountDao"/>
        <property name="itemDao" ref="itemDao"/>
        <!-- additional collaborators and configuration for this bean go here -->
    </bean>

    <!-- more bean definitions for services go here -->

    </beans>


Ejemplo del archivo de configuración `daos.xml` referenciado en el primer ejemplo:

    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="accountDao"
        class="org.springframework.samples.jpetstore.dao.jpa.JpaAccountDao">
        <!-- additional collaborators and configuration for this bean go here -->
    </bean>

    <bean id="itemDao" class="org.springframework.samples.jpetstore.dao.jpa.JpaItemDao">
        <!-- additional collaborators and configuration for this bean go here -->
    </bean>

    <!-- more bean definitions for data access objects go here -->

    </beans>

### Instanciación del contenedor IoC con metadatos basados en anotaciones de Java

Puede hacerse mediante la clase `ConfigurableApplicationContext` o `AnnotationConfigApplicationContext`.

#### *ConfigurableApplicationContext*

Para instanciar el contenedor IoC se le proporcionan una o varias clases anotadas como `@Configuration` (Java):

    ConfigurableApplicationContext appContext = new AnnotationConfigApplicationContext(AppConfig.class, AppConfig2.class);

Ejemplo (sin implementación) de la clase AppConfig mencionada en el ejemplo anterior (Java):

    import org.springframework.context.annotation.Bean;
    import org.springframework.context.annotation.Configuration;

    @Configuration
    public class AppConfig {

        @Bean
        public Mundo mundo(){
            return new Mundo();
        }
}

#### *AnnotationConfigApplicationContext*

El contenedor IoC se instancia mediante un constructor:

- Sin parámetros: las clases anotadas con `@Configuration` se registran más tarde mediante el método `.register( )`. Ejemplo (Java):

        AnnotationConfigApplicationContext ctx = new AnnotationConfigApplicationContext();

        ctx.register(AppConfig.class);

        ctx.refresh();

- Con parámetros: las clases anotadas con `@Configuration` se le proporcionan como parámetros. Ejemplo (Java):

        AnnotationConfigApplicationContext ctx = new AnnotationConfigApplicationContext(AppConfig.class, AppConfig2.class);


***

Según: https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-metadata