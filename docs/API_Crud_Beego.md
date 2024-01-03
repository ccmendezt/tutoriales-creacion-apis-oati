## Generar API CRUD

- ### Crear el modelo de BD
- ### Correr el script SQL en PostgreSQL
- ### Ejecutar el comando para creacion de API CRUD:
  ```golang
  bee api nombre_api -driver=postgres -conn="postgres://usuario_db:contraseña_db@host/nombre_db?sslmode=disable&search_path=nombre_esquema"
  ```
- ### Crear variables de entorno en archivo `app.conf`

  ```
  PGuser = ${AGENDA_API_PGUSER}
  PGpass = ${AGENDA_API_PGPASS}
  PGhost = ${AGENDA_API_PGHOST}
  PGport = ${AGENDA_API_PGPORT}
  PGdb = ${AGENDA_API_PGDB}
  PGschema = ${AGENDA_API_PGSCHEMA}
  ```

- ### Crear el archivo .env:

  ```
  export AGENDA_API_PGUSER=usuario
  export AGENDA_API_PGPASS=contraseña
  export AGENDA_API_PGHOST=host
  export AGENDA_API_PGPORT=puerto
  export AGENDA_API_PGDB=nombre_bd
  export AGENDA_API_PGSCHEMA=schema
  ```

- ### Configurar string de conexion a base de datos:

  ```golang
  orm.RegisterDataBase("default", "postgres", "postgres://"+
  	beego.AppConfig.String("PGuser")+":"+
  	beego.AppConfig.String("PGpass")+"@"+
  	beego.AppConfig.String("PGhost")+":"+
  	beego.AppConfig.String("PGport")+"/"+
  	beego.AppConfig.String("PGdb")+"?sslmode=disable&search_path="+
  	beego.AppConfig.String("PGschema")+"")
  ```

- ### Configurar Ruta main.go:
  Agregar al import de routers:
  ```bash
  "github.com/udistrital/"
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

- ### Ejecutar cambios en los modelos:
  Especificar el id `auto` y cambiar el tipo `time.Time` a `string` en los atributos _fecha_creacion_ y _fecha_modificacion_ en los modelos
- ### Modificar el manejo de errores en los controladores
- ### Modificar fechas de creación/modificación en los controladores:

  Importar en cada controlador la siguiente biblioteca:
  `"github.com/udistrital/utils_oas/time_bogota"`

  Colocar las siguientes líneas de código:

  - #### Para método HTTP POST:

  ```golang
  v.FechaCreacion = time_bogota.TiempoBogotaFormato()
  v.FechaModificacion = time_bogota.TiempoBogotaFormato()
  ```

  - #### Para método HTTP PUT:

  ```golang
  v.FechaCreacion = time_bogota.TiempoCorreccionFormato(v.FechaCreacion)
  v.FechaModificacion = time_bogota.TiempoBogotaFormato()
  ```

- ### Subir las variables de entorno con:

  `source nombre_archivo.env`

- ### Correr la API Crud con documentación en Swagger:
  `bee run -downdoc=true -gendoc=true`
