## Generar Cliente Angular

- ### Copiar proyecto del contenedor al PC anfitrión en Windows

  Si el primer comando no funciona, ejecutar el segundo:

  **1. Primer Comando**

  ```bash
  docker cp hygen:/go/src/github.com/udistrital/_templates/plantillas_cliente $GOPATH/src/github.com/udistrital
  ```

  **2. Segundo Comando**

  ```bash
  docker cp hygen:/go/src/github.com/udistrital/_templates/plantillas_cliente %GOPATH%/src/github.com/udistrital
  ```
