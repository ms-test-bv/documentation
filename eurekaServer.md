# Eureka server

En un ecosistema de microservicios, normalmente deberíamos definir las conexiones entre aplicaciones de la forma dura, donde debemos fijar el ip y el puerto de cada uno. Para resolver este problema, surguen los Discovery Service, los cuales nos permiten registrar nuestros microservicios y conectarlos utilizando solo los nombres de los mismos; de esta forma, nos quitamos el paso intermedio de memorizar las ip de cada app. En este caso estamos viendo Eureka Server; este Service Discovery pertenece a la suite de Spring Cloud de Java y nos permite de forma sencilla e intuitiva la configuración del servidor. 

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
    - eureka server

## Configurando Eureka

Antes de comenzar, no se nos puede olvidar que debemos instalar las dependencias mediante el comando `mvn clean install`. Una vez que termine la instalación accedemos a nuestra aplicación con el editor de texto que más queramos y vamos directamente a la clase principal. Dentro, añadimos el decorador `@EnableEurekaServer` sobre el nombre de nuestra clase. En nuestro código, quedaría de la siguiente forma:

```java
@SpringBootApplication
@EnableEurekaServer
public class EurekaApplication {
	public static void main(String[] args) {
		SpringApplication.run(EurekaApplication.class, args);
	}
}
```

Ahora, vamos a realilar una serie de configuraciones en el archivo applications.properties o applications.yml:

```yml
eureka:
  client:
    register-with-eureka: false # De esta forma evitamos que el mismo eureka se registre a el mismo
    fetch-registry: false # Indica que no no haga peticiones al mismo servidor
    service-url:
      defaultZone: http://${EUREKA_INSTANCA:localhost}:${EUREKA_PORT}/eureka/ # Por donde estará escuchando el servidor de eureka
  instance:
    hostname: localhost # Nombre de la instancia
```

Estas son configuraciones que deben ir por defecto cuando creemos nuestro discovery service:
- `eureka.client.register-with-eureka`: Esta opción nos permite definir si queremos que el mismo eureka server se registre a si mismo. Si no queremos que esto suceda, debemos asignarle el valor `false`
- `eureka.client.fetch-registry`: Indica si el cliente propio de este servidor debe buscar información en Eureka Server. Como anteriormente decidimos que no se registre a si mismo, esta función debe estar en `false` igualmente.
- `eureka.client.service-url.defaultZone`: Nos permite definir el enpoint que utilizará eureka para registrar los microservicios. En este caso estamos apuntando al mismo servidor.
- `instance.hostname`: Esta opción no es obligatoria. Ya que indica el nombre que se utilizará para crear las instancias de eureka server (si, podemos tener más de una instancia). Si no usamos esta propiedad, el servidor se encargará de hacer esto.

Ya con esto tenemos configurado nuestro servidor de eurka. El siguiente paso es configurar todos los clientes para que sean registrados de forma automática. Ahora, cuando iniciamos nuestro servidor y entramos a la aplicación, veremos el siguiente warning al inicio de la misma:

`EMERGENCY! EUREKA MAY BE INCORRECTLY CLAIMING INSTANCES ARE UP WHEN THEY'RE NOT. RENEWALS ARE LESSER THAN THRESHOLD AND HENCE THE INSTANCES ARE NOT BEING EXPIRED JUST TO BE SAFE.`

Para resolver esta advertencia, solo debemos agregar la siguiente propiedad a nuestro yml:

```yml
eureka:
  server:
    renewal-percent-threshold: 0.95
```

## Configurando clientes

Ahora toca hacer que nuestros clientes se conecten al servidor de eurka. Para ellos, debemos comenzar instalando la siguiente dependencia:

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```

Terminada la instalación de la nueva dependencia, podemos pasar a la configuración para encnotrar nuestro eureka server. A continuación se presentan los datos de configuración usados:

```yml
eureka:
  client:
    service-url:
      defaultZone: http://${EUREKA_SERVER:localhost}:${EUREKA_PORT:9000}/eureka
```

Ya con esta mínima configuración, nuestra app ya puede ser encontrada por el servidor eureka. Solo basta con correr los dos servidores y comenzar a trabajar.

# Conclusiones

Espero que este pequeño tutorial haya sido de su agrado. Si encuentran algún error o demora, no duden enviar un mensaje por github para realizar los cambios pertinentes. No dejen de aprender y nos vemos pronto.

# Bibliografía
- https://www.tutorialspoint.com/spring_boot/spring_boot_eureka_server.htm
- https://spring.io/guides/gs/service-registration-and-discovery/
- https://migueldoctor.medium.com/spring-cloud-series-crea-un-servicio-de-registro-y-descubrimiento-con-spring-cloud-netflix-eureka-4758615ad4cb
- https://www.baeldung.com/spring-cloud-netflix-eureka
- https://cloud.spring.io/spring-cloud-netflix/multi/multi_spring-cloud-eureka-server.html