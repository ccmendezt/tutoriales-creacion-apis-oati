## Generar API MID

- ### Ejecutar el comando para creacion de API CRUD:
  ```golang
  bee api api_mid_beego_request
  ```
- ### Configurar CORS

  Importar el siguiente paquete en `main.go`:

  ```bash
  "github.com/astaxie/beego/plugins/cors"
  ```

  Colocar antes del `beego.run()`:

  ```golang
  beego.InsertFilter("*", beego.BeforeRouter, cors.Allow(&cors.Options{
  	AllowOrigins: []string{"*"},
  	AllowMethods: []string{"PUT", "PATCH", "GET", "POST", "OPTIONS", "DELETE"},
  	AllowHeaders: []string{"Origin", "x-requested-with",
  		"content-type",
  		"accept",
  		"origin",
  		"authorization",
  		"x-csrftoken"},
  	ExposeHeaders:    []string{"Content-Length"},
  	AllowCredentials: true,
  }))
  ```

- ### Configurar Rutas:

  Agregar a los imports donde haga falta:

  ```bash
  "github.com/udistrital/"
  ```

- ### Crear archivo utilidades.go

  Se crea la carpeta `helpers` en la raíz del proyecto y dentro de esta se creará `utilidades.go`, este archivo contendrá las peticiones comunes tales como _GetAll_ y tendrá el siguiente formato:

  ```golang
  package helpers

  import (
    "encoding/json"
    "errors"
    "fmt"
    "net/http"

    "github.com/astaxie/beego"
  )

  func GetRequestNew(endpoint string, route string, target interface{}) error {
    url := beego.AppConfig.String("ProtocolAdmin") + "://" + beego.AppConfig.String(endpoint) + route
    var response map[string]interface{}
    var err error
    err = GetJson(url, &response)
    err = ExtractData(response, &target)
    return err
  }

  func GetJson(url string, target interface{}) error {
    r, err := http.Get(url)
    if err != nil {
      return err
    }

    defer func() {
      if err := r.Body.Close(); err != nil {
        beego.Error(err)
      }
    }()
    return json.NewDecoder(r.Body).Decode(target)
  }

  // Esta función extrae la información cuando se recibe encapsulada en una estructura
  // y da manejo a las respuestas que contienen arreglos con objetos vacíos
  func ExtractData(respuesta map[string]interface{}, v interface{}) error {
    var err error
    if respuesta["Success"] == false {
      err := errors.New(fmt.Sprint(respuesta["Data"], respuesta["Message"]))
      panic(err)
    }
    datatype := fmt.Sprintf("%v", respuesta["Data"])
    switch datatype {
    case "map[]", "[map[]]": //response vacío
      break
    default:
      err = FillStruct(respuesta["Data"], &v)
      respuesta = nil
    }
    return err
  }

  func FillStruct(in interface{}, out interface{}) (err error) {
    var str []byte
    if str, err = json.Marshal(in); err != nil {
      return
    }
    err = json.Unmarshal(str, &out)
    return
  }
  ```

- ### Crear variables de entorno en app.conf:
  ```
  ProtocolAdmin = ${API_MID_PRUEBA_PROTOCOL_ADMIN}
  ```
- ### Crear variables de entorno en dev.env

  ```
  export API_MID_PRUEBA_PROTOCOL_ADMIN=http
  ```
