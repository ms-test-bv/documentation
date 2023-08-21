# Config Server

El servidor de configuración nos permite centralizar todos los archivos de configuración de todos los microservicios (MS) con los que estemos trabajando en el proyecto. De esta forma, podemos organizar nuestro código y la configuración siempre estará accesible; además de poder hacer cambios de forma dinámica para posteriormente refrescar de forma los MS.

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
    - Config Server

Presionamos el botón generate y esperamos que se descargue nuestro comprimido. Una vez terminada la descarga, lo copiamos y descomprimimos en donde más nos guste de la cmputadora. Una vez descomprimido, iniciamos una consola y escribimos el siguiente comando `mvn clean install -DskipTests`; esto nos permitirá instalar las dependencias.

## Configuración inicial

Una vez instaladas las dependencias tenemos que dirigirnos al `Main` de nuestra aplicación. Dentro, agregaremos el siguiente decorador a nuestra clase principa:

- `@EnableConfigServer`

## Creando el repo de config

Para comenzar tenemos que crear una carpeta en nuestro sistema. Dentro iniciamos un repositorio de git con el comando `git init`. Una inicializado, vamos a crear nuestros archivos de propiedades. Spring Config Server admite las siguientes formas para crear un archivo de configuración:

```txt
/{application}/{profile}[/{label}]
/{application}-{profile}.yml
/{label}/{application}-{profile}.yml
/{application}-{profile}.properties
/{label}/{application}-{profile}.properties
```

En nuestro caso vamos a seguir la segunda opción. Por lo que nuestro archivo quedaría nombrado `item-dev.yml`. Podemos dejar dentro de la propia PC los archivos; pero en nuestro caso lo estaremos leyendo directamente desde github.

## Config Server leyendo las configuraciones

Vamos a nuestro archivo `application.yml` y colocamos los siguientes datos:

```yml
server:
  port: 9001
spring:
  cloud:
    config:
      server:
        git:
          uri: https://github.com/ms-test-bv/config-server
          clone-on-start: true
          timeout: 100
```

- **uri**: nos permite definir la dirección del repositorio git. En este caso estamos apuntando a nuestro repositorio y por defecto, a la rama master.
- **clone-on-start**: sirve que para que cada vez que se levante el servidor, clone el repositorio de internet.
- **timeout**: es el tiempo de interrupción de la conexión hacia el config server.

## Conectando el cliente

Con la configuración básica creada para el servidor, podemos conectar nuestra aplicación cliente. Para ello, debemos instalar las siguietnes dependencias:

- **spring-cloud-starter-config**: Nos permite acceder al servidor de configuración y a las propiedades para la configuración.
- **spring-cloud-starter-bootstrap**: Esta dependencia nos permitirá ejecutar el archivo `bootstrap.yml`. El objetivo de este archivo es ejecutar la configuración al inicio del servidor. Normalmente se utiliza para almacenar la configuración de spring cloud. Ejemplo la del config server.
    - Me dio error varias veces la aplicaciones por no poner esta dependencia. Así que ojo.
- **spring-boot-starter-actuator**: Este nos permite acceder a una serie de endpoints que nos brindan información util del estado de nuestra aplicación. El que más nos interesa en este momento es `/refresh`. Si cambiamos la configuración en el repositorio de git y no queremos apagar la aplicación y volverla a prender, podemos realizar una llamada `POST` a este endpoint `actuator/refresh` y lo obligaremos a recargar las configuraciones.
    - Adicionalmente debemos añadir el decorador `@RefreshScope` a nuestra aplicación.
    - No intente ponerlo en el config server. No funciona.

Habiendo visto las dependencias a instalar, podemos pasar a las configuraciones. Vamos a comenzar con la conexión al config server (**ojo esto es el `bootstrap.yml`**):

```yml
spring:
  cloud:
    config:
      uri: http://localhost:9001/ 
      fail-fast: true 
      retry:
        initial-interval: 2000 
        max-attempts: 10
      profile: dev 
  application:
    name: items 
  config:
    import: configserver:http://localhost:9001/ 
```

- `spring.cloud.config.uri`: Aquí definimos el servidor donde estará almacenada nuestra configuración.
- `spring.cloud.config.fail-fast`: Con esto le decimos a nuestro servidor que si no logra conectarse que automaticamente falle y no se quede esperando.
- `spring.cloud.config.retry.initial-interval`: Definimos el intervalo de peticiones hacia el servidor cada vez que falle.
- `spring.cloud.config.retry.max-attempts`: Definimos la cantidad máximas de intentos antes de terminar la conexión.
- `spring.cloud.config.profile`: Le decimos que perfil debe cargar. Recuerden que los archivos en el config server deben declararse con un perfil.
- `spring.application.name`: Esto configura el nombre de la aplicación. Ojo, nombre que definamos aquí será utilizado para buscar el archivo de configuración. Por lo tanto, si aquí mi app se llama **items**, mi archivo debe llamarse **items-{perfil}.yml**
- `spring.config.import`: Esto es sumamente importante. Como no tenemos una declaración de configuración por defecto, le debemos decir a spring que debe importarla desde el servidor de configuración.

Para terminar con el apartado de configuración del cliente, veamos que líneas de código debemos añadir para activar el actuator:

```yml
management:
  endpoints:
    web:
      exposure:
        include: refresh
```

- `management.endpoints.web.exposure.include`: Esta configuración nos permite definir que **endpoint** del actuator estarán disponibles para acceder. El **health check** se activa por defecto. 
    - Si queremos añadir otros endpoint, podemos separarlos por ,

## Probando lo creado

Para probar que se esté reconociendo nuestra configuración, debemos iniciar los dos servidores. El de config server no trae problemas ya que su inicio es sencillo y si da error, automaticamente te lo dirá. Con el que hay que tener ojo es con el cliente; ya que tenemos que monitorizar si da error o no. Aunque, no tengas miedo si se demora un poco al iniciar, debe descargar la configuración del config server. Cuando tengamos hecho esto, pasamos a probar en el endpoint del config server:

- Hacemos una petición `GET` a `127.0.0.1:9001/{applicationName}/{profile}`
- Si todo está bien, debe devolvernos lo siguiente:
```json
{
    "name": "items",
    "profiles": [
        "dev"
    ],
    "label": null,
    "version": "42c2c1910cd1165c16b8ba0dd21767dfc153f266",
    "state": null,
    "propertySources": [
        {
            "name": "https://github.com/ms-test-bv/config-server/items-dev.yml",
            "source": {
                "server.port": 8089,
                "server.address": "0.0.0.0",
                "spring.datasource.url": "jdbc:postgresql://${DB_HOST:localhost}:${DB_PORT:5433}/${DB_NAME:item}",
                "spring.datasource.driver-class-name": "org.postgresql.Driver",
                "spring.datasource.username": "${DB_USER:postgres}",
                "spring.datasource.password": "${DB_PASSWORD:postgres}",
                "spring.jpa.properties.hibernate.dialect": "org.hibernate.dialect.PostgreSQLDialect",
                "spring.jpa.hibernate.ddl-auto": "update"
            }
        }
    ]
}
```

## Tips para una mejor configuración

Existen varias formas de mejorar nuestros códigos de configuración. La primera es utilizando variables de entorno para obtener datos sencibles como constraseñas. Ejemplo de esto lo pedemos encontrar en la configuración anteriormente vista: `${DB_USER:postgres}`; aquí estamos diciendo que se debe utilizar la variable de entorno `DB_USER` y si no se encuetra usar el valor por defecto `postgres`.


# Conclusiones

Espero que este pequeño tutorial haya sido de su agrado. Si encuentran algún error o demora, no duden enviar un mensaje por github para realizar los cambios pertinentes. No dejen de aprender y nos vemos pronto.

# Bibliografía
- https://cloud.spring.io/spring-cloud-static/Greenwich.RELEASE/multi/multi__spring_cloud_config_2.html
- https://www.baeldung.com/spring-cloud-configuration
- https://github.com/npalma9006/shop-ms/tree/develop
- https://www.tutorialspoint.com/spring_boot/spring_boot_cloud_configuration_server.htm
- https://spring.io/guides/gs/centralized-configuration/
- https://www.baeldung.com/spring-boot-actuator-enable-endpoints