# Apuntes de Microservicios con Java y Spring Boot

## Estructura CSR (Controller-Service-Repository)

Este documento contiene apuntes sobre la arquitectura de microservicios en Spring Boot, organizados por capas según el orden de programación.

---

## Índice
1. [Capa Model / Entity](#capa-model--entity)
2. [Capa Service](#capa-service)
3. [Capa Controller](#capa-controller)
4. [Capa Configuration](#capa-configuration)

---

# Capa Model / Entity

## Descripción General
La capa Model (o Entity) representa los objetos de dominio que se persisten en la base de datos. Contiene la estructura de datos, las anotaciones de persistencia y las reglas de validación a nivel de base de datos. Las entidades son los objetos sobre los que gira toda la lógica de negocio.

---

## Anotaciones de Persistencia

### @Entity
- **Propósito**: Marca la clase como una entidad JPA que se mapea a una tabla en la base de datos
- **Características**:
  - Indica que la clase es persistible
  - Spring Data JPA la reconoce y puede generar queries automáticamente
  - La clase debe tener un constructor sin argumentos
  - Se requiere un campo identificador único (@Id)

### @Table
- **Propósito**: Especifica el nombre y características de la tabla en la base de datos
- **Características**:
  - Define el nombre exacto de la tabla
  - Permite configurar esquemas y catálogos
  - Útil cuando el nombre de la clase no coincide con el nombre de la tabla

---

## Anotaciones de Lombak

### @Data
- **Propósito**: Genera automáticamente getters, setters, equals, hashCode y toString
- **Beneficios**:
  - Reduce código boilerplate
  - Mantiene el código limpio y legible
  - Los métodos se generan en tiempo de compilación
  - Facilita cambios futuros en los atributos

### @NoArgsConstructor
- **Propósito**: Genera automáticamente un constructor sin argumentos
- **Características**:
  - Requerido por JPA para la instanciación de entidades
  - Spring y los frameworks de persistencia lo necesitan
  - Permite crear objetos vacíos para posterior asignación de valores

### @AllArgsConstructor
- **Propósito**: Genera automáticamente un constructor con todos los atributos como parámetros
- **Beneficios**:
  - Facilita la creación rápida de instancias con todos los valores
  - Útil en operaciones de testing
  - Simplifica la inicialización de objetos

### @Builder
- **Propósito**: Implementa el patrón Builder para construir objetos de forma elegante
- **Características**:
  - Permite construir entidades de forma fluida y legible
  - Cada atributo tiene un método setter en la cadena de construcción
  - Termina con .build() para crear el objeto final
  - Especialmente útil cuando hay muchos atributos

**Ejemplo de uso**:
```java
Recurso recurso = Recurso.builder()
    .campo1(valor1)
    .campo2(valor2)
    .campo3(valor3)
    .build();
```

---

## Anotaciones de Campos

### @Id
- **Propósito**: Marca el campo como clave primaria de la tabla
- **Características**:
  - Es obligatorio en toda entidad
  - Identifica de forma única cada registro
  - Típicamente es un número Long o String
  - Solo puede haber un @Id por entidad

### @GeneratedValue
- **Propósito**: Especifica cómo se genera automáticamente el valor del identificador
- **Estrategias comunes**:
  - **IDENTITY**: La base de datos genera el ID (autoincrement)
  - **SEQUENCE**: Usa una secuencia de la base de datos
  - **TABLE**: Usa una tabla para generar IDs
  - **AUTO**: Spring elige la estrategia según la base de datos

**Ventajas**:
- El ID se asigna automáticamente sin intervención del aplicativo
- Garantiza unicidad
- Simplifica la creación de nuevos registros

### @Column
- **Propósito**: Configura los detalles del mapeo de un atributo a una columna
- **Parámetros principales**:
  - **name**: Nombre exacto de la columna en la base de datos
  - **nullable**: Si la columna puede contener valores nulos (true/false)
  - **unique**: Si la columna debe tener valores únicos
  - **length**: Longitud máxima permitida para datos de texto

**Ejemplos de configuraciones**:
- `nullable = false`: Campo obligatorio, no puede ser nulo
- `unique = true`: Valor debe ser único en toda la tabla
- `length = 100`: Máximo 100 caracteres
- Combinaciones: `nullable = false, unique = true, length = 150`

---

## Estructura de una Entidad

### Componentes clave:
1. **Identificador (ID)**: Campo único que identifica cada registro
2. **Atributos**: Campos que representan datos de negocio
3. **Restricciones**: Reglas de validación a nivel de base de datos

---

## Buenas Prácticas

### En el diseño:
✅ Usar nombres significativos para clase y columnas  
✅ Definir restricciones en la base de datos  
✅ Usar tipos de datos apropiados  
✅ Mantener la lógica de negocio fuera de las entidades simples  
✅ Documentar atributos complejos  

### En la validación:
✅ Marcar campos obligatorios con nullable = false  
✅ Definir restricciones de unicidad cuando sea necesario  
✅ Establecer límites de longitud apropiados  
✅ Usar tipos específicos (Integer, LocalDate, etc.)  

---

## Rol de la Entity en la Arquitectura

**Responsabilidades**:
- Representar la estructura de datos en la base de datos
- Definir restricciones y validaciones a nivel de persistencia
- Mapear automáticamente datos de BD a objetos Java
- Servir como modelo de dominio para la lógica de negocio

**Lo que NO debe hacer**:
- Contener lógica HTTP o de API
- Exponerse directamente a los clientes (usar DTO)
- Implementar lógica de negocio compleja
- Manejar serialización/deserialización a JSON

---

# Capa Service

## Descripción General
La capa Service contiene toda la lógica de negocio de la aplicación. Es la intermediaria entre el Controller y el Repository, encargándose de procesar los datos, aplicar reglas de negocio, realizar transformaciones y coordinar operaciones con la persistencia.

---

## Anotaciones Principales

### @Service
- **Propósito**: Marca la clase como un servicio que contiene lógica de negocio
- **Características principales**:
  - Indica que la clase es un componente de Spring que puede ser inyectado
  - Organiza el código de lógica de negocio en una capa separada
  - Facilita la testabilidad del código
  - Permite una clara separación de responsabilidades entre capas

---

## Inyección de Dependencias

### Inyección por Constructor
El Service recibe el Repository a través del constructor para acceder a la persistencia.

**Características**:
- El Repository se declara como final, garantizando su inmutabilidad
- Spring infiere automáticamente la inyección sin necesidad de anotaciones adicionales
- Permite un acceso controlado y seguro a la base de datos

---

## Métodos de Operaciones CRUD

### Método CREATE - Guardar un nuevo recurso
**Responsabilidades**:
1. Recibe un DTO con los datos del nuevo recurso
2. Transforma el DTO a una entidad de dominio usando el patrón Builder
3. Persiste la entidad en la base de datos mediante el Repository
4. Transforma la entidad guardada a un DTO
5. Devuelve el DTO con el recurso creado

**Procesos internos**:
- Construcción de la entidad a partir de los datos recibidos
- Llamada al Repository para guardar en base de datos
- Transformación de entidad a DTO para la respuesta
- Inclusión del ID generado por la base de datos

**Validaciones comunes**:
- Verificar que los datos requeridos estén presentes
- Validar el formato de datos (emails, teléfonos, etc.)
- Comprobar duplicados o restricciones de negocio

---

### Método READ ALL - Obtener todos los recursos
**Responsabilidades**:
1. Consulta al Repository para obtener todos los registros
2. Transforma cada entidad en un DTO
3. Retorna una lista de DTOs

**Procesos internos**:
- Recuperación de todas las entidades desde la base de datos
- Iteración sobre la colección de entidades
- Transformación individual de cada entidad a DTO
- Compilación de la lista de DTOs

**Optimizaciones consideradas**:
- Paginación para grandes volúmenes de datos
- Filtrado de campos sensibles
- Ordenamiento de resultados

---

### Método READ BY ID - Obtener un recurso específico
**Responsabilidades**:
1. Consulta al Repository buscando el recurso por identificador
2. Maneja el caso cuando el recurso no existe
3. Transforma la entidad encontrada a DTO
4. Devuelve el DTO del recurso

**Procesos internos**:
- Búsqueda en la base de datos por ID
- Validación de existencia del recurso
- Lanzamiento de excepción si no existe
- Transformación de entidad a DTO

**Manejo de errores**:
- Lanzar excepción cuando el recurso no es encontrado
- Logging de operaciones fallidas
- Devolución de códigos de error apropiados

---

### Método DELETE - Eliminar un recurso
**Responsabilidades**:
1. Verifica si el recurso existe en la base de datos
2. Si existe, lo elimina
3. Devuelve un booleano indicando éxito o fracaso

**Procesos internos**:
- Verificación de existencia mediante el ID
- Eliminación del registro si existe
- Retorno de confirmación de operación

**Validaciones comunes**:
- Comprobar que el recurso existe antes de eliminar
- Validar permisos de eliminación
- Registrar auditoría de eliminaciones
- Considerar eliminación lógica (soft delete) si es necesario

---

### Método UPDATE - Actualizar un recurso existente
**Responsabilidades**:
1. Busca el recurso existente por identificador
2. Valida que el recurso existe
3. Actualiza los campos con los nuevos valores del DTO
4. Persiste los cambios en la base de datos
5. Transforma la entidad actualizada a DTO
6. Devuelve el DTO actualizado

**Procesos internos**:
- Búsqueda de la entidad existente
- Asignación de nuevos valores a los atributos
- Guardado de la entidad modificada
- Transformación a DTO para la respuesta

**Validaciones comunes**:
- Verificar que el recurso existe
- Validar los nuevos datos antes de actualizar
- Comprobar que no se violen restricciones de negocio
- Registrar cambios para auditoría

---

## Transformación entre Entidades y DTOs

### Patrón Builder
La transformación entre DTO (Data Transfer Object) y entidades de dominio se realiza usando el patrón Builder.

**DTO a Entidad**:
- Recibe datos del cliente en formato DTO
- Construye una entidad de dominio con esos datos
- La entidad contiene la lógica de negocio
- Se persiste en la base de datos

**Entidad a DTO**:
- Recupera la entidad de la base de datos
- Transforma la entidad a un DTO
- El DTO es serializado a JSON para la respuesta
- Solo expone los campos necesarios al cliente

**Beneficios**:
- Aislamiento: La entidad de dominio está separada de la API
- Seguridad: Se controla qué datos se exponen
- Flexibilidad: Se puede cambiar la estructura interna sin afectar la API
- Mantenibilidad: Cambios en la base de datos no rompen el contrato con los clientes

---

## Responsabilidades de la Capa Service

**Debe hacer**:
- Implementar toda la lógica de negocio
- Validar datos según reglas de dominio
- Transformar entre DTOs y entidades
- Coordinar operaciones entre múltiples repositorios
- Manejar transacciones
- Aplicar cálculos y procesos complejos
- Registrar auditoría y logging
- Lanzar excepciones de negocio apropiadas

**NO debe hacer**:
- Recibir directamente solicitudes HTTP (eso es del Controller)
- Acceder directamente a la base de datos sin usar Repository
- Retornar entidades directamente (siempre DTO)
- Manejar serialización a JSON

---

## Manejo de Excepciones

### Excepciones Comunes
- **ResourceNotFoundException**: Cuando un recurso buscado no existe
- **InvalidInputException**: Cuando los datos recibidos no cumplen validaciones
- **DuplicateResourceException**: Cuando se intenta crear un recurso que ya existe
- **BusinessLogicException**: Para violaciones de reglas de negocio

**Práctica recomendada**:
- Lanzar excepciones específicas de negocio
- Registrar errores con logging
- Permitir que el Controller maneje la conversión a códigos HTTP

---

## Patrón Builder en Transformaciones

El patrón Builder facilita la construcción de objetos complejos de forma legible y segura.

**Ventajas**:
- Código más legible que constructores con múltiples parámetros
- Permite construir objetos parciales
- Cada campo se asigna explícitamente
- Fácil mantenimiento cuando se agregan campos

**Uso**:
- Construir entidades a partir de DTOs
- Construir DTOs a partir de entidades
- Crear objetos en operaciones de negocio complejas

---

# Capa Controller

## Descripción General
La capa Controller es responsable de manejar las solicitudes HTTP de los clientes, dirigirlas al servicio correspondiente y devolver las respuestas. Es la puerta de entrada de la aplicación y actúa como intermediaria entre el cliente y la lógica de negocio.

---

## Anotaciones Principales

### @RestController
- **Propósito**: Marca la clase como un controlador REST que maneja solicitudes HTTP
- **Características principales**:
  - Combina las funcionalidades de @Controller y @ResponseBody
  - Serializa automáticamente los objetos de retorno a formato JSON
  - Indica que todos los métodos devuelven datos en lugar de vistas o templates HTML
  - Prepara la clase para recibir y responder solicitudes REST

---

## Inyección de Dependencias

### Inyección por Constructor
El Controller recibe las dependencias del Service a través del constructor, siendo esta la mejor práctica en Spring Boot.

---

## Métodos HTTP - Anotaciones y Funcionalidad

### @PostMapping - CREATE (C del CRUD)
**Propósito**: Crear un nuevo recurso
**Anotaciones utilizadas**:
- **@PostMapping**: Define que este método maneja solicitudes HTTP POST en una ruta específica
- **@RequestBody**: Especifica que el parámetro recibido viene en el cuerpo de la solicitud en formato JSON

**Proceso**:
1. El cliente envía una solicitud POST con datos en formato JSON
2. Spring deserializa automáticamente el JSON al DTO o modelo especificado
3. Se pasa el objeto al servicio para procesar la lógica de creación
4. El servicio realiza la operación y persiste los datos
5. Se devuelve el objeto creado serializado a JSON

**Respuesta esperada**: Código 201 (Created) con el recurso creado

---

### @GetMapping (lista completa) - READ ALL (R del CRUD)
**Propósito**: Obtener todos los recursos
**Anotaciones utilizadas**:
- **@GetMapping**: Define que este método maneja solicitudes HTTP GET en una ruta específica

**Proceso**:
1. El cliente envía una solicitud GET
2. Se consulta el servicio para recuperar todos los registros
3. El servicio obtiene la colección de recursos desde la persistencia
4. Se devuelve la lista completa serializada a JSON

**Respuesta esperada**: Código 200 (OK) con un array de recursos

---

### @GetMapping (por identificador) - READ BY ID (R del CRUD)
**Propósito**: Obtener un recurso específico por su identificador
**Anotaciones utilizadas**:
- **@GetMapping**: Define que este método maneja solicitudes HTTP GET
- **@PathVariable**: Extrae el valor del identificador de la ruta y lo asigna al parámetro del método

**Proceso**:
1. El cliente envía una solicitud GET con un identificador en la ruta
2. Spring extrae el identificador de la URL automáticamente
3. Se consulta el servicio con ese identificador específico
4. El servicio busca y recupera el recurso correspondiente
5. Se devuelve el recurso encontrado serializado a JSON

**Respuesta esperada**: Código 200 (OK) si existe el recurso, o 404 (Not Found) si no existe

---

### @PutMapping - UPDATE (U del CRUD)
**Propósito**: Actualizar un recurso existente
**Anotaciones utilizadas**:
- **@PutMapping**: Define que este método maneja solicitudes HTTP PUT
- **@PathVariable**: Extrae el identificador de la ruta para identificar qué recurso actualizar
- **@RequestBody**: Recibe los nuevos datos del recurso en el cuerpo de la solicitud

**Proceso**:
1. El cliente envía una solicitud PUT con un identificador en la ruta y datos actualizados en JSON
2. Spring extrae el identificador de la URL y deserializa los datos
3. Se valida que el recurso con ese identificador existe
4. Se pasa al servicio el identificador y los datos nuevos
5. El servicio actualiza el registro en la persistencia
6. Se devuelve el objeto actualizado serializado a JSON

**Respuesta esperada**: Código 200 (OK) con el recurso actualizado

---

### @DeleteMapping - DELETE (D del CRUD)
**Propósito**: Eliminar un recurso
**Anotaciones utilizadas**:
- **@DeleteMapping**: Define que este método maneja solicitudes HTTP DELETE
- **@PathVariable**: Extrae el identificador de la ruta para identificar qué recurso eliminar

**Proceso**:
1. El cliente envía una solicitud DELETE con un identificador en la ruta
2. Spring extrae el identificador de la URL
3. Se consulta el servicio para eliminar ese recurso
4. El servicio busca y elimina el registro de la persistencia
5. Se devuelve una confirmación de la operación

**Respuesta esperada**: Código 200 (OK) con confirmación, o 404 (Not Found) si no existe el recurso

---

## Flujo General de una Solicitud HTTP en el Controller

1. **Cliente realiza una solicitud HTTP** (POST, GET, PUT, DELETE) a una ruta específica
2. **@RestController recibe la solicitud** y la dirige al método correspondiente según la anotación
3. **Las anotaciones de parámetros** (@RequestBody, @PathVariable) extraen y deserializan los datos
4. **Inyección de dependencias** proporciona la instancia del Service
5. **Se llama al método del Service** que contiene la lógica de negocio
6. **El Service realiza la operación** (consulta, persistencia, cálculos, etc.)
7. **Se retorna la respuesta** serializada a JSON
8. **El cliente recibe** el resultado con el código de estado HTTP correspondiente

---

## Anotaciones de Parámetros

### @RequestBody
- Especifica que el parámetro viene en el cuerpo de la solicitud HTTP
- Deserializa automáticamente el contenido JSON en el DTO o modelo especificado
- Se utiliza típicamente en operaciones POST (crear) y PUT (actualizar)
- El formato debe ser JSON válido

### @PathVariable
- Extrae valores de la ruta URL (parámetros entre llaves)
- Asigna el valor extraído al parámetro del método automáticamente
- Se utiliza para pasar identificadores o filtros en operaciones GET, PUT y DELETE
- El nombre de la variable en la ruta debe coincidir con el nombre del parámetro

---

## Rol del Controller en la Arquitectura

**Responsabilidades**:
- Recibir solicitudes HTTP de los clientes
- Validar la estructura de las solicitudes
- Extraer parámetros y datos de la solicitud
- Delegar la lógica de negocio al Service
- Serializar la respuesta a JSON
- Devolver códigos de estado HTTP apropiados

**Lo que NO debe hacer**:
- Implementar lógica de negocio compleja
- Acceder directamente a la base de datos
- Manejar excepciones específicas del dominio
- Realizar transformaciones de datos complejas

---

# Capa Configuration

## Descripción General
La capa Configuration contiene la configuración centralizada de la aplicación. Define beans reutilizables que serán inyectados en toda la aplicación, como clientes HTTP y componentes globales.

---

## Anotaciones Principales

### @Configuration
- Marca la clase como fuente de definiciones de beans de Spring
- Los métodos anotados con @Bean crean beans automáticamente

### @Bean
- Marca un método como productor de un bean
- El objeto retornado es gestionado por Spring y puede ser inyectado en otras clases

---

## WebClient - Cliente HTTP Reactivo

WebClient es un cliente HTTP asincrónico y no bloqueante para hacer llamadas a otros servicios.

**Estructura básica**:
```java
@Configuration
public class WebClientConfig {
    
    @Bean
    public WebClient miWebClient() {
        return WebClient.builder()
                .baseUrl("https://api.ejemplo.com")
                .defaultHeader("Accept", "application/json")
                .build();
    }
}
```

**Componentes**:
- **baseUrl**: URL base para las solicitudes
- **defaultHeader**: Headers que se envían automáticamente en cada solicitud

**Inyección**:
```java
@Service
public class MiServicio {
    private final WebClient miWebClient;
    
    public MiServicio(WebClient miWebClient) {
        this.miWebClient = miWebClient;
    }
}
```

---

## Ventajas

✅ Centraliza la configuración global  
✅ Beans reutilizables en toda la aplicación  
✅ Fácil de mantener y actualizar  
✅ Permite diferentes configuraciones por ambiente  

---

# Configuración de Propiedades (application.properties)

## Descripción
El archivo `application.properties` contiene la configuración de la aplicación como nombre, conexión a base de datos, y comportamiento de JPA/Hibernate.

## Propiedades Principales

```properties
# Nombre de la aplicación
spring.application.name=alumnos

# Configuración de Base de Datos
spring.datasource.url=jdbc:mysql://localhost:3306/app_db
spring.datasource.username=app_user
spring.datasource.password=secret
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver

# Configuración de JPA/Hibernate
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
```

## Explicación de Propiedades

### Nombre de la Aplicación
- `spring.application.name`: Nombre identificador del microservicio

### Conexión a Base de Datos
- `spring.datasource.url`: URL de conexión (servidor, puerto, base de datos)
- `spring.datasource.username`: Usuario para autenticarse en la BD
- `spring.datasource.password`: Contraseña del usuario
- `spring.datasource.driver-class-name`: Driver JDBC para MySQL

### JPA/Hibernate
- `spring.jpa.hibernate.ddl-auto`: Estrategia de creación/actualización de tablas
  - `create`: Crea las tablas (borra si existen)
  - `create-drop`: Crea y elimina al cerrar
  - `update`: Actualiza las tablas existentes
  - `validate`: Solo valida sin cambios
  - `none`: No realiza cambios
- `spring.jpa.show-sql`: Muestra las consultas SQL en la consola (útil para debugging)

---

## Resumen del Flujo de Programación

```
1. Entity (@Entity, @Table, @Column)
   ↓
2. Service (@Service - contiene lógica de negocio)
   ↓
3. Controller (@RestController - recibe solicitudes HTTP)
   ↓
4. Configuration (@Configuration - configura beans globales)
```

**Flujo de una solicitud**:
```
Cliente HTTP
    ↓
Controller (@PostMapping, @GetMapping, etc.)
    ↓
Service (lógica de negocio)
    ↓
Repository (acceso a datos)
    ↓
Entity (mapeo a base de datos)
    ↓
Respuesta JSON al cliente
```

---

**Última actualización**: 2026  
