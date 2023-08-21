# Api Gateway

En el desarrollo de aplicaciones web modernas basadas en pmicroservicios existe un problema latente de comunicación entre frontend y backend. Al tener un sistema backend que consta de varias aplicaciones, se hace necesario que el frontend se conecte a cada uno por separado, lo que no solo trae problemas de código sino de seguridad también. La capacidad de que la aplicación del lado del cliente tenga acceso a cada uno de los microservicios es un tema de gran sensibilidad. Como solución a este problema surgen las API Gateway; que tienen como propósito crear un punto de acceso único a todos los microservicios que tengamos.

## Creación

Para crear nuestra aplicación nos apoyamos en el sitio web https://start.spring.io/. Aquí dentro vamos a utilizar los siguientes parámetros:

- Project: Maven
- Language: Java
- Spring Boot: 3.1.2
- Grupo: com.game
- Artifact: config.server
- Packagin: Jar
- Java: 17
- Dependencias
    - Eureka Discovery Client
    - Gateway
    - Actuator
    - Security

Presionamos el botón generate y esperamos que se descargue nuestro comprimido. Una vez terminada la descarga, lo copiamos y descomprimimos en donde más nos guste de la cmputadora. Una vez descomprimido, iniciamos una consola y escribimos el siguiente comando `mvn clean install -DskipTests`; esto nos permitirá instalar las dependencias. En este caso estamos poniendo todas las dependencias necesarias para trabajar.

## Configuración inicial

La configuración inicial de API Gateway no es muy complicada, ya que lo unico que necesitamos realizar es la conexión a nuestro Discovery Server y posteriormente veremos la configuración de las rutas. Comencemos viendo las propiedades añadidas a `application.yml`:

```yml
management:
  endpoints:
    web:
      exposure:
        include: refresh
server:
  port: 9002
  address: 0.0.0.0

spring:
  application:
    name: api-gateway

eureka:
  client:
    service-url:
      defaultZone: http://${EUREKA_SERVER:localhost}:${EUREKA_PORT:9000}/eureka/
  instance:
    preferIpAddress: true
```

- `eureka.client.service-url.defaultZone`: esta propiedad nos permite apuntar hacia nuestro servidor eureka
- `spring.application.name`: esta propiedad define el nombre de la aplicación y será el nombre utilizado en eureka para registrar el servicio.
- `management.endpoints.web.exposure.include`: nos permite incluir endpoints del actuator que por defecto están desactivados.

Con esta configuración ya tenemos para comenzar a trabajar. Ahora toca pasar a configurar las rutas que serán manejadas por la **api gateway**.

## Configurando rutas

El Api Gateway de Spring Boot brinda dos formas de manejar rutas. La primera tiene un acercamiento más funcional ya que debemos registrar una función de manejo; la segunda utiliza el mismo archivo de configuración para administrar las rutas.

Comencemos viendo como sería utilizar la primera opción:

### Rutas funcionales

Para comenzar vamos a crear una clase encargada de contener la función de rutas, pero podemos utilizar la clase principal sin ningún problema. La clase a crear debe tener el decorador `@Configuration` y la función que crearemos dentro con el decorador `@Bean`.

La función a crear debe ser de tipo `RouteLocator` y recibir por parámetros un dato de tipo `RouteLocatorBuilder` (el constructor de la ruta). Las rutas a construir siguien el siguiente comportamiento:

```java
builder.routes().route("TIPO DE RUTA", "PATH DE RUTA.APUNTADOR AL MS")
```

Llevando el ejemplo anterior a código, quedaría de la siguiente forma:
```java
@Configuration
public class CustomRouter {
    @Bean
    public RouteLocator customRouter(RouteLocatorBuilder builder){
        return builder.routes().route("path_route", pr -> pr.path("/item/**").uri("lb://ITEMS")).build();
    }
}
```

### Rutas en properties

Vamos a llevar la configuración anterior hacia nuestro archivo de configuración:

```yml
spring:
  application:
    name: api-gateway
  cloud:
    gateway:
      routes:
        - id: items-service
          uri: lb://items
          predicates:
            - Path=/item/**
```

En cualquiera de los dos casos debe tener en cuenta que el valor de **path** debe ser idéntico a como está expuesto el endpoint en el microservicio.

### Filter Chain

> Pendiente

### Security

> Pendiente

# Conclusiones

Espero que este pequeño tutorial haya sido de su agrado. Si encuentran algún error o demora, no duden enviar un mensaje por github para realizar los cambios pertinentes. No dejen de aprender y nos vemos pronto.

# Bibliografía
- https://www.solo.io/topics/api-gateway/api-gateway-spring-boot/
- https://spring.io/projects/spring-cloud-gateway
- https://www.baeldung.com/spring-cloud-gateway
- https://www.javaguides.net/2022/10/spring-boot-microservices-spring-cloud-api-gateway.html
- https://www.geeksforgeeks.org/java-spring-boot-microservices-develop-api-gateway-using-spring-cloud-gateway/
- https://www.appsdeveloperblog.com/spring-cloud-api-gateway-tutorial/#The_repositories_Element
