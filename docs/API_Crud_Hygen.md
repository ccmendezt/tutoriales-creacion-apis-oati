## Generar API CRUD

Una vez los contenedores se crearon y están ejecutándose se realiza:

- ### Entrar al contenedor de docker

  ```bash
  docker exec -it hygen bash
  ```

- ### Crear API Crud (edite "nombre_api_crud")

  ```bash
  hygen plantilla_api_crud with-prompt --appname nombre_api_crud
  ```

  - Ingrese la configuración correspondiente a su versión de base de datos, credenciales de usuario y base de datos.

- ### Salir del contenedor

  ```bash
  exit
  ```

- ### Copiar proyecto del contenedor al pc anfitrión (edite "nombre_api_crud")

  ```bash
  docker cp hygen:/go/src/github.com/udistrital/nombre_api_crud $GOPATH/src/github.com/udistrital
  ```

- ### Cambiar tipos de datos en los modelos:

  Especificar el id `auto` y cambiar el tipo de dato `time.Time` a `string` en los atributos _fecha_creacion_ y _fecha_modificacion_.

- ### Modificar fechas de creación/modificación en los controladores y modelos:

  Importar en cada controlador la siguiente biblioteca:
  `"github.com/udistrital/utils_oas/time_bogota"`

  Colocar las siguientes líneas de código:

  - #### Para método HTTP POST:

  ```golang
  v.FechaCreacion = time_bogota.TiempoBogotaFormato()
  v.FechaModificacion = time_bogota.TiempoBogotaFormato()
  ```

  - #### Para método HTTP PUT (Editar un registro):

  **En el Controlador:**

  ```golang
  v.FechaModificacion = time_bogota.TiempoBogotaFormato()
  ```

  **En el Modelo en método _UpdateContactoById_:**

  ```golang
  m.FechaCreacion = time_bogota.TiempoCorreccionFormato(v.FechaCreacion)
  ```
