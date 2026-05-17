# API REST de Despachos

Proyecto Spring Boot para gestionar despachos mediante una API REST conectada a MySQL. Permite crear, consultar, actualizar y eliminar registros de despacho, además de exponer documentación interactiva con Swagger/OpenAPI.

## Tecnologías

- Java 17
- Spring Boot 3.4.4
- Spring Web
- Spring Data JPA
- MySQL
- Lombok
- springdoc OpenAPI / Swagger UI
- Docker
- GitHub Actions + AWS (ECR, EC2, SSM)

## Qué hace el proyecto

La aplicación administra entidades `Despacho` con los siguientes datos:

- `idDespacho`: identificador del despacho
- `fechaDespacho`: fecha del despacho
- `patenteCamion`: patente del camión
- `intento`: número de intento de entrega
- `idCompra`: identificador de la compra asociada
- `direccionCompra`: dirección de entrega
- `valorCompra`: valor de la compra
- `despachado`: indica si el despacho fue realizado

## Cómo está organizado

```text
src/main/java/com/citt
├── config                # CORS y OpenAPI
├── controller            # Endpoints REST
├── exceptions            # Manejo de errores
└── persistence
    ├── entity            # Entidad Despacho
    ├── repository        # JpaRepository
    └── services          # Lógica de negocio
```

## Flujo de funcionamiento

1. El cliente consume los endpoints bajo `/api/v1/despachos`.
2. `DespachoController` recibe la solicitud HTTP.
3. `DespachoServiceImpl` aplica la lógica de negocio.
4. `DespachoRepository` persiste y consulta los datos en MySQL.
5. Si ocurre un error de validación o no existe un despacho, `RestResponseEntityExceptionHandler` responde con un error estructurado.

## Configuración

El proyecto usa estas variables de entorno para la conexión a base de datos:

- `DB_ENDPOINT`
- `DB_PORT`
- `DB_NAME`
- `DB_USERNAME`
- `DB_PASSWORD`

Propiedades principales:

- Puerto de la API: `8081`
- Swagger UI: `/swagger-ui.html`
- Hibernate: `spring.jpa.hibernate.ddl-auto=update`
- CORS abierto para todos los orígenes

## Cómo ejecutar el proyecto localmente

### 1. Requisitos

- Java 17
- Maven Wrapper (`./mvnw`)
- MySQL disponible

### 2. Definir variables de entorno

Ejemplo:

```bash
export DB_ENDPOINT=localhost
export DB_PORT=3306
export DB_NAME=despacho
export DB_USERNAME=root
export DB_PASSWORD=tu_clave
```

### 3. Ejecutar la aplicación

```bash
./mvnw spring-boot:run
```

La API quedará disponible en:

```text
http://localhost:8081
```

## Cómo ejecutar con Docker

Construir imagen:

```bash
docker build -t despacho-api .
```

Ejecutar contenedor:

```bash
docker run --rm -p 8081:8081 \
  -e DB_ENDPOINT=host.docker.internal \
  -e DB_PORT=3306 \
  -e DB_NAME=despacho \
  -e DB_USERNAME=root \
  -e DB_PASSWORD=tu_clave \
  despacho-api
```

## Documentación Swagger

Con la aplicación levantada, la documentación interactiva está en:

```text
http://localhost:8081/swagger-ui.html
```

## Endpoints disponibles

Base URL:

```text
/api/v1/despachos
```

### Crear despacho

- Método: `POST`
- Ruta: `/api/v1/despachos`

Body de ejemplo:

```json
{
  "fechaDespacho": "2026-05-17",
  "patenteCamion": "ABCD12",
  "intento": 1,
  "idCompra": 1001,
  "direccionCompra": "Av. Siempre Viva 123",
  "valorCompra": 25990,
  "despachado": false
}
```

### Obtener todos los despachos

- Método: `GET`
- Ruta: `/api/v1/despachos`

### Obtener despacho por ID

- Método: `GET`
- Ruta: `/api/v1/despachos/{idDespacho}`

### Actualizar despacho

- Método: `PUT`
- Ruta: `/api/v1/despachos/{idDespacho}`

### Eliminar despacho

- Método: `DELETE`
- Ruta: `/api/v1/despachos/{idDespacho}`

## Respuestas de error

Cuando un despacho no existe, la API responde con `404 Not Found`.

Cuando hay errores de validación, la API responde con `400 Bad Request` y devuelve un objeto con:

- `status`
- `message`
- `errors`

## Pruebas

Para ejecutar pruebas:

```bash
./mvnw test
```

Importante: la prueba de carga de contexto necesita que las variables de base de datos estén definidas y que MySQL sea accesible, porque la aplicación inicializa JPA al arrancar.

## Despliegue automático

El flujo definido en `.github/workflows/main.yml` hace lo siguiente al hacer push a `main`:

1. Compila la imagen Docker.
2. La publica en Amazon ECR.
3. Se conecta a AWS SSM.
4. En EC2 crea una red Docker.
5. Levanta un contenedor MySQL.
6. Levanta el contenedor de la API con las variables de entorno necesarias.

## Archivo Dockerfile

El `Dockerfile` usa construcción multi-stage:

1. Compila el proyecto con Maven y Java 17.
2. Copia el `.jar` generado a una imagen liviana con JRE 17.
3. Expone el puerto `8081`.

## Uso recomendado

- Usar Swagger para probar endpoints rápidamente.
- Definir variables de entorno antes de ejecutar pruebas o levantar la app.
- Tener MySQL disponible antes de iniciar la API.
