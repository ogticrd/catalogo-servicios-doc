# Referencia de la API de Catalogo de Servicios

> Catalogo ofrece una API REST para consumir los datos en la base de datos. La API tiene direcciones URL predecibles orientadas a los recursos, se basa en códigos de estado HTTP estándar y utiliza JSON para la entrada y la salida de datos.

**Open API Specification**

Catalogo tiene un endpoint para mostrar las especificaciones Open API para describir y/o detallar el API REST. La idea es que los desarrolladores puedan hacer pruebas de manera más intuitivas.

Para obtener las especificaciones pueden usar el siguiente endpoint:

```
GET /server/specs/oas
```

Una vez tengan las especificaciones del API, pueden usar el [Editor de Swagger](https://editor.swagger.io/) u otro, pegar las especificaciones en el editor para generar dinamicamente todos los endpoints existentes a los que su usuario tiene acceso.

## Introducción

### Autenticación

De forma predeterminada, todos los datos del catalogo están fuera del alcance de los usuarios no autenticados. Para obtener acceso a los datos protegidos, **debe solicitar un token de acceso**.

Más sobre Autenticación en catalogo [aqui](Autenticacion.md).

### REST API y los Datos relacionales

De forma predeterminada, Catalogo solo recupera el valor de referencia de un campo relacional en los elementos. Para recuperar también datos anidados de un campo relacional, se puede utilizar el parámetro `fields` en REST. Esto le permite recuperar datos referenciados de un servicio.

Para definir el ámbito de los campos que se devuelven por tipo de colección, puede usar la sintaxis `<field>:<scope>` del parámetro fields de la siguiente manera: 

```http
GET /items/services
	?fields[]=name
	&fields[]=description
	&fields[]=legal_framework_ids.description
	&fields[]=services_electronic_gov_type_ids.electronic_gov_type_id.type
```

Otra alternativa seria:

```http
GET /items/services
	?fields[]=name,description,legal_framework_ids.description,services_electronic_gov_type_ids.electronic_gov_type_id.type
```

Para conocer más sobre las consultas en catalogo vsita este [enlace](Consultas.md).

#### Método HTTP SEARCH

Al usar la API de REST para leer varios elementos mediante filtros (muy) avanzados, es posible que se encuentre con el problema de que la dirección URL simplemente no puede contener suficientes datos para incluir la estructura de consulta completa. En esos casos, puede usar el método HTTP SEARCH como reemplazo directo de GET, donde se le permite colocar la consulta en el cuerpo de la solicitud de la siguiente manera:

**Antes**:

```HTTP
GET /items/services?filter[name][_eq]=Hello World
```

**Después**:

```HTTP
SEARCH /items/services

{
	"query": {
		"filter": {
			"name": {
				"_eq": "Hello World"
			}
		}
	}
}
```

Hay mucha discusión sobre si se debe o no poner un cuerpo en una solicitud GET, usar POST para crear consultas de búsqueda o confiar en un método completamente diferente. A partir de ahora, hemos optado por alinearnos con la [especificación del método HTTP SEARCH de IETF](https://datatracker.ietf.org/doc/draft-ietf-httpbis-safe-method-w-body).

**Lectura útil**:

- [Método de búsqueda HTTP (IETF, 2021)](https://datatracker.ietf.org/doc/draft-ietf-httpbis-safe-method-w-body)
- [Definición de un nuevo método HTTP: HTTP SEARCH (Tim Perry, 2021)](https://httptoolkit.tech/blog/http-search-method)
- [HTTP GET con cuerpo de solicitud (StackOverflow, 2009 y en curso)](https://stackoverflow.com/questions/978061/http-get-with-request-body)
- [Uso del cuerpo de Elastic Search GET (elástico, s.f.)](https://www.elastic.co/guide/en/elasticsearch/guide/current/_empty_search.html)
- [Dropbox comienza a usar POST y por qué se trata de un diseño deficiente de la API. (Evert Pot, 2015)](https://evertpot.com/dropbox-post-api)

#### Códigos de error

A continuación se muestran los códigos de error globales utilizados en Catalogo y lo que significan.

| Código de error | Estado | Descripción |
|---|---|---|
| FAILED VALIDATION | 400 | Error en la validación de este elemento en particular |
| FORBIDDEN | 403 | No se le permite realizar la acción actual |
| INVALID_TOKEN | 403 | El token proporcionado no es válido |
| TOKEN_EXPIRED | 401 | El token proporcionado es válido pero ha caducado |
| INVALID_CREDENTIALS | 401 | El nombre de usuario/contraseña o el token de acceso son incorrectos |
| INVALID IP | 401 | Su dirección IP no está en la lista de permitidos para ser utilizada con este usuario |
| INVALID_OTP | 401 | Se proporcionó una OTP incorrecta |
| INVALID_PAYLOAD | 400 | La carga útil proporcionada no es válida |
| INVALID_QUERY | 400 | No se pueden utilizar los parámetros de consulta solicitados |
| UNSUPPORTED_MEDIA_TYPE | 415 | El formato de carga útil o el encabezado proporcionados no son compatibles `Content-Type` |
| REQUESTS_EXCEEDED | 429 | Alcanza el límite de velocidad |
| ROUTE_NOT_FOUND | 404 | El punto de conexión no existe |
| SERVICE_UNAVAILABLE | 503 | No se pudo usar el servicio externo |
| UNPROCESSABLE_CONTENT | 422 | Intentaste hacer algo ilegal |

> **Seguridad**
>
> Para evitar que se filtren los elementos existentes, todas las acciones de los elementos no existentes devolverán un error FORBIDDEN.