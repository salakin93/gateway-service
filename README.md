# Gateway Service

## Descripción

`gateway-service` es el **API Gateway** de la plataforma de microservicios.
Actúa como el **punto único de entrada** para todos los clientes externos y se encarga de enrutar las solicitudes hacia los microservicios internos correspondientes.

Este servicio se integra con:

* **Spring Cloud Gateway**
* **Eureka Service Discovery**
* **Spring Cloud Config Server**

El gateway permite centralizar el acceso, la seguridad, el monitoreo y la gestión del tráfico dentro de la arquitectura de microservicios.

---

# Rol dentro de la arquitectura

En una arquitectura de microservicios, el gateway cumple las siguientes funciones:

* Proveer **un único punto de acceso público**
* **Enrutar solicitudes** hacia los microservicios internos
* Integrarse con **Eureka** para descubrimiento de servicios
* Centralizar **seguridad y autenticación**
* Centralizar configuración de **CORS**
* Permitir **logging y monitoreo centralizado**

Flujo de arquitectura:

```
Cliente
   |
   v
Gateway Service (Puerto 8080)
   |
   v
+---------------------------+
|   Microservicios internos |
|                           |
|  identity-service         |
|  document-service         |
|  ai-service               |
+---------------------------+
```

---

# Tecnologías utilizadas

* Java 21
* Spring Boot 4
* Spring Cloud Gateway
* Spring Cloud Config Client
* Netflix Eureka Client
* Spring Boot Actuator
* Gradle

---

# Responsabilidades del servicio

El gateway tiene las siguientes responsabilidades:

* Servir como punto de entrada para los clientes
* Enrutar solicitudes hacia microservicios internos
* Descubrir servicios mediante **Eureka**
* Cargar configuración desde **Config Server**
* Exponer endpoints de monitoreo con **Actuator**

Funcionalidades que se implementarán posteriormente:

* Validación de JWT
* Rate limiting (limitación de solicitudes)
* Logging de requests
* Distributed tracing
* Configuración centralizada de CORS

---

# Fuente de configuración

Toda la configuración del servicio se encuentra en el repositorio centralizado:

```
microservices-config
```

Archivo de configuración correspondiente:

```
gateway-service.properties
```

---

# Configuración externa

Ubicación del archivo en el repositorio `microservices-config`:

```
gateway-service.properties
```

Ejemplo de configuración:

```
server.port=8080
spring.application.name=gateway-service

eureka.client.service-url.defaultZone=http://localhost:8761/eureka

management.endpoints.web.exposure.include=health,info

spring.cloud.gateway.server.webflux.discovery.locator.enabled=true
spring.cloud.gateway.server.webflux.discovery.locator.lower-case-service-id=true
```

---

# Enrutamiento dinámico con Eureka

El gateway utiliza **service discovery** para enrutar solicitudes automáticamente.

Por ejemplo, si el servicio `identity-service` está registrado en Eureka:

```
http://localhost:8080/identity-service/**
```

El gateway automáticamente enruta la solicitud hacia:

```
lb://identity-service
```

Esto permite:

* No tener URLs de servicios hardcodeadas
* Balanceo de carga automático
* Resolución dinámica de servicios

---

# Configuración de la aplicación

Archivo local:

```
src/main/resources/application.properties
```

Ejemplo:

```
spring.application.name=gateway-service
spring.config.import=configserver:http://localhost:8888
```

Esto indica que la configuración debe ser cargada desde **Config Server**.

---

# Ejecutar el servicio localmente

Antes de iniciar el gateway, deben levantarse los siguientes servicios en este orden:

1. **config-service**
2. **discovery-service**
3. **gateway-service**

Ejecutar con Gradle:

```
./gradlew bootRun
```

O ejecutar desde IntelliJ usando la clase principal:

```
GatewayServiceApplication
```

---

# Verificar registro en Eureka

Abrir el dashboard de Eureka:

```
http://localhost:8761
```

Debería aparecer el servicio registrado como:

```
GATEWAY-SERVICE
```

---

# Probar el Gateway

Una vez iniciado el servicio:

```
http://localhost:8080
```

Ejemplo futuro de enrutamiento:

```
http://localhost:8080/identity-service/api/auth/login
```

El gateway redirigirá la solicitud automáticamente al servicio correspondiente.

---

# Soporte para Docker

Este servicio incluye un **Dockerfile** para facilitar su despliegue mediante contenedores.

## Construir la imagen

```
docker build -t gateway-service .
```

## Ejecutar el contenedor

```
docker run -p 8080:8080 gateway-service
```

---

# Dockerfile

```
FROM gradle:8.7-jdk21 AS builder

WORKDIR /app
COPY . .

RUN gradle bootJar --no-daemon

FROM eclipse-temurin:21-jdk

WORKDIR /app
COPY --from=builder /app/build/libs/*.jar app.jar

EXPOSE 8080

ENTRYPOINT ["java","-jar","/app/app.jar"]
```

---

# Integración con otros servicios

Este servicio interactúa con los siguientes componentes del sistema:

| Servicio          | Propósito                                |
| ----------------- | ---------------------------------------- |
| config-service    | Configuración centralizada               |
| discovery-service | Descubrimiento de servicios (Eureka)     |
| identity-service  | Autenticación y gestión de usuarios      |
| document-service  | Gestión de documentos                    |
| ai-service        | Procesamiento de inteligencia artificial |

---

# Mejoras futuras

Funcionalidades planificadas para el gateway:

* Validación de tokens JWT
* Control de acceso basado en roles
* Rate limiting
* Circuit breaker
* Distributed tracing
* Logging centralizado de APIs
* Filtros de seguridad

---

# Estructura del repositorio

```
gateway-service
│
├── src
│   └── main
│       ├── java
│       │   └── edu.usip.gateway
│       │       └── GatewayServiceApplication.java
│       │
│       └── resources
│           └── application.properties
│
├── build.gradle
├── Dockerfile
└── README.md
```

---

# Parte de la plataforma de microservicios

Este servicio forma parte de una plataforma completa compuesta por:

* identity-service
* config-service
* discovery-service
* gateway-service
* document-service
* ai-service

Todos los servicios serán orquestados posteriormente mediante **Docker Compose**.
