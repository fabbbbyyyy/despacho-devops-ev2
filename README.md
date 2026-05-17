# Flujo CI/CD

Este repositorio usa el workflow `.github/workflows/main.yml` para compilar, publicar y desplegar la aplicación automáticamente.

## Disparador

El pipeline se ejecuta en cada `push` a la rama `main`.

## Variables y secretos usados

### Variable global del workflow

- `REGISTRY_URL`: `${{ secrets.AWS_ACCOUNT_ID }}.dkr.ecr.${{ secrets.AWS_REGION }}.amazonaws.com`

### Secrets requeridos

- `AWS_ACCOUNT_ID`
- `AWS_REGION`
- `AWS_ACCESS_KEY_ID`
- `AWS_SECRET_ACCESS_KEY`
- `AWS_SESSION_TOKEN`
- `AWS_ECR_REPOSITORY`
- `EC2_INSTANCE_ID`
- `DB_NAME`
- `DB_USER`
- `DB_PASSWORD`

## Job 1: Build and Push Image

Nombre del job: `build-and-push`

Pasos:

1. Checkout del repositorio.
2. Configuración de credenciales AWS.
3. Login a Amazon ECR.
4. Build de la imagen Docker con dos tags:
   - `${{ github.sha }}`
   - `latest`
5. Push de ambas tags al repositorio ECR.

Resultado: la imagen de la API queda publicada en ECR.

> Nota: el workflow publica tag inmutable (`${{ github.sha }}`) y `latest`. Para despliegues más predecibles y rollback controlado, se recomienda desplegar por SHA.

## Job 2: Deploy to EC2 via SSM

Nombre del job: `deploy-to-ec2`  
Dependencia: se ejecuta después de `build-and-push`.

Pasos:

1. Configuración de credenciales AWS.
2. Ejecución de un `aws ssm send-command` contra la instancia EC2.
3. En la instancia, el script:
   - Crea el directorio `/home/ec2-user/backend-despacho`.
   - Hace login a ECR.
   - Descarga la imagen `latest`.
   - Crea la red Docker `api-network` (si no existe).
   - Detiene y elimina contenedores previos `springboot-api` y `mysql-db`.
   - Levanta contenedor MySQL (`mysql:8.0`) con volumen `mysql_data`.
   - Espera 15 segundos.
   - Levanta contenedor `springboot-api` en puerto `8081` con variables de entorno para DB.
   - Limpia imágenes no utilizadas con `docker image prune -a -f`.

Resultado: despliegue actualizado de base de datos + API en EC2.

> Notas operativas:
> - El `sleep 15` es una espera fija y puede no garantizar que MySQL esté realmente listo; se recomienda usar healthchecks o una verificación activa antes de iniciar la API.
> - `docker image prune -a -f` elimina todas las imágenes no usadas, lo que puede reducir capacidad de rollback rápido; para una limpieza menos agresiva se puede usar `docker image prune -f`.

## Resumen del flujo

1. Push a `main`.
2. Build de imagen.
3. Push a ECR.
4. Despliegue remoto vía SSM en EC2.
5. Recreación de contenedores MySQL y API con la nueva versión.
