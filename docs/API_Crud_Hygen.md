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
