## Generar API MID

- ### Entrar al contenedor de docker

  ```bash
  docker exec -it hygen bash
  ```

- ### Crear API Mid (edite "nombre_api_mid")

  ```bash
  hygen plantilla_api_mid with-prompt --appname nombre_api_mid
  ```

  - Ingrese la configuración correspondiente a su versión de base de datos, credenciales de usuario y base de datos.

- ### Salir del contenedor

  ```bash
  exit
  ```

- ### Copiar proyecto del contenedor al pc anfitrión (edite "nombre_api_mid")

  ```bash
  docker cp hygen:/go/src/github.com/udistrital/nombre_api_mid $GOPATH/src/github.com/udistrital
  ```
