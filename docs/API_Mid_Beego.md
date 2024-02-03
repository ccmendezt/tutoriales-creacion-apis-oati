## Generar API MID - Beego

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
  UrlCrudParametros = ${API_MID_PRUEBA_URL_CRUD_PARAMETROS}
  UrlCrudAgenda = ${API_MID_PRUEBA_URL_CRUD_AGENDA}
  ```
- ### Crear variables de entorno en dev.env

  ```
  export API_MID_PRUEBA_PROTOCOL_ADMIN=http
  export API_MID_PRUEBA_URL_CRUD_PARAMETROS=localhost:8081/v1/
  export API_MID_PRUEBA_URL_CRUD_AGENDA=localhost:8082/v1/
  ```

- ### Crear modelos necesarios en la API MID con la siguiente estructura:

  ```golang
  package models

  type Contacto struct {
    Id                int
    Nombre            string
    Documento         int64
    Direccion         string
    FechaCreacion     string
    FechaModificacion string
    Activo            bool
  }
  ```

  ```golang
  package models

  type ParametrosContactos struct {
    Parametros []Parametro
    Contactos []Contacto
  }
  ```

- ### Crear el helper necesario con la siguiente estructura:

  ```golang
  package helpers

  import (
    "fmt"

    "github.com/prometheus/common/log"
    "github.com/udistrital/api_mid_parametros_contactos/models"
  )

  func ListarParametrosContactos() (parametros_Contactos models.ParametrosContactos, outputError map[string]interface{}) {
    defer func() {
      if err := recover(); err != nil {
        outputError = map[string]interface{}{"funcion": "ListarParametros", "err": err, "status": "500"}
      }
    }()
    var parametros_query []models.Parametro
    var contactos_query []models.Contacto
    //Limit 0 para traer todos los registros, si no se pone limit trae solo 10
    url := "parametro?query=TipoParametroId__Nombre:Cargos&limit=0"
    if err := GetRequestNew("UrlCrudParametros", url, &parametros_query); err != nil {
      log.Error(err)
      panic(err.Error())
    }
    url = "contacto"
    if err := GetRequestNew("UrlCrudAgenda", url, &contactos_query); err != nil {
      log.Error(err)
      panic(err.Error())
    }
    fmt.Println("Contactos: ", contactos_query)
    parametros_Contactos.Parametros = parametros_query
    parametros_Contactos.Contactos = contactos_query
    return parametros_Contactos, outputError
  }
  ```

- ### Crear el controlador necesario con la siguiente estructura:

  ```golang
  package controllers

  import (
    "github.com/astaxie/beego"
    "github.com/udistrital/api_mid_parametros_contactos/helpers"
  )

  type ParametrosContactosController struct {
    beego.Controller
  }

  func (c *ParametrosContactosController) URLMapping() {
  }

  // GetAll ...
  // @Title Get All
  // @Description get ParametrosContactos
  // @Success 200 {object} models.ParametrosContactos
  // @Failure 403
  // @router / [get]
  func (c *ParametrosContactosController) GetAll() {

    defer helpers.ErrorController(c.Controller, "ParametrosController")
    if v, err := helpers.ListarParametrosContactos(); err == nil {
      c.Ctx.Output.SetStatus(200)
      c.Data["json"] = map[string]interface{}{"Success": true, "Status": 200, "Message": "Listado consultado con éxito", "Data": v}
    } else {
      panic(err)
    }
    c.ServeJSON()
  }
  ```

- ### Configurar el router de acuerdo a los controladores creados:

  ```golang
  package routers

  import (
    "github.com/astaxie/beego"
    "github.com/udistrital/api_mid_parametros_contactos/controllers"
  )

  func init() {
    ns := beego.NewNamespace("/v1",
      beego.NSNamespace("/parametros_contactos",
        beego.NSInclude(
          &controllers.ParametrosContactosController{},
        ),
      ),
    )
    beego.AddNamespace(ns)
  }
  ```

- ### Subir las variables de entorno con:

  `source nombre_archivo.env`

- ### Correr la API Crud con documentación en Swagger:
  `bee run -downdoc=true -gendoc=true`
