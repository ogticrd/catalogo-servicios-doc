# Referencia de la API de Catalogo de Servicios

> Catalogo ofrece una API REST para consumir los datos en la base de datos. La API tiene direcciones URL predecibles orientadas a los recursos, se basa en códigos de estado HTTP estándar y utiliza JSON para la entrada y la salida de datos.

## Introducción

### Autenticación

De forma predeterminada, todos los datos del catalogo están fuera del alcance de los usuarios no autenticados. Para obtener acceso a los datos protegidos, debe solicitar un token de acceso.

### Datos relacionales

De forma predeterminada, Catalogo solo recupera el valor de referencia de un campo relacional en los elementos. Para recuperar también datos anidados de un campo relacional, se puede utilizar el parámetro fields en REST o consultas anidadas regulares en GraphQL. Esto le permite recuperar el autor de su artículo incluido en los datos de los artículos o obtener puntos de entrada de registro relacionados para los datos de análisis de su aplicación, por ejemplo.

#### Creación / Actualización / Eliminación

De manera similar a la búsqueda, el contenido relacional también se puede modificar profundamente.

**De muchos a uno**

Las relaciones de muchos a uno son bastante sencillas de gestionar relacionalmente. Simplemente puede enviar los cambios que desee como un objeto en la clave relacional de la colección. Por ejemplo, si quisieras crear un nuevo artículo destacado en tu página, podrías enviar:

```JSON
{
	"featured_article": {
		"title": "This is my new article!"
	}
}
```

Esto creará un nuevo registro en la colección relacionada y guardará su clave principal en el campo de este elemento. Para actualizar un elemento existente, simplemente proporcione la clave principal con las actualizaciones y Catalogo lo tratará como una actualización en lugar de una creación:featured_article

```JSON
{
	"featured_article": {
		"id": 15,
		"title": "This is an updated title for my article!"
	}
}
```

Al ver que la relación de varios a uno almacena la clave externa en el propio campo, se puede eliminar el elemento anulando el campo:

```JSON
{
	"featured_article": null
}
```

**Uno a Muchos (/ Muchos-a-Varios)**

Las relaciones de uno a varios y, por lo tanto, de varios a varios y de varios a cualquiera, se pueden actualizar de una de estas dos maneras:

1. Básico

La API devolverá campos de uno a varios como una matriz de claves o elementos anidados (según el parámetro). Puede utilizar esta misma estructura para seleccionar cuáles son los elementos relacionados:fields

```JSON
{
	"children": [2, 7, 149]
}
```

También puede proporcionar un objeto en lugar de una clave principal para crear nuevos elementos anidados sobre la marcha, o un objeto con una clave principal incluida para actualizar un elemento existente:

```JSON
{
	"children": [
		2, // assign existing item 2 to be a child of the current item
		{
			"name": "A new nested item"
		},
		{
			"id": 149,
			"name": "Assign and update existing item 149"
		}
	]
}
```

Para eliminar elementos de esta relación, simplemente omítalos de la matriz:

```JSON
{
	"children": [2, 149]
}
```

Este método de actualización de uno a varios es muy útil para conjuntos de datos relacionales más pequeños.

**"Detallado"**

Como alternativa, puede proporcionar un objeto que detalle los cambios de la siguiente manera:

```JSON
{
	"children": {
		"create": [{ "name": "A new nested item" }],
		"update": [{ "id": 149, "name": "A new nested item" }],
		"delete": [7]
	}
}
```

Esto es útil si necesita tener un control más estricto sobre los cambios preconfigurados o cuando trabaja con un gran conjunto de datos relacionales.

Varios a cualquiera (tipos de unión)
Los campos de varios a cualquier funcionan de forma muy similar a un varios a varios "normal", con la excepción de que el campo relacionado puede extraer los campos de cualquiera de las colecciones relacionadas, por ejemplo:

```JSON
{
	"sections": [
		{
			"collection": "headings",
			"item": {
				/* headings fields */
			}
		},
		{
			"collection": "paragraphs",
			"item": {
				/* paragraphs fields */
			}
		}
	]
}
```

### REST API

Para definir el ámbito de los campos que se devuelven por tipo de colección, puede usar la sintaxis del parámetro fields de la siguiente manera:<field>:<scope>

```
GET /items/pages
	?fields[]=sections.item:headings.id
	&fields[]=sections.item:headings.title
	&fields[]=sections.item:paragraphs.body
	&fields[]=sections.item:paragraphs.background_color
```

> **Actualización**
>
> La actualización de registros en un varios a cualquiera es idéntica a los otros tipos de relación.

#### Método HTTP SEARCH

Al usar la API de REST para leer varios elementos mediante filtros (muy) avanzados, es posible que se encuentre con el problema de que la dirección URL simplemente no puede contener suficientes datos para incluir la estructura de consulta completa. En esos casos, puede usar el método HTTP SEARCH como reemplazo directo de GET, donde se le permite colocar la consulta en el cuerpo de la solicitud de la siguiente manera:

**Antes**:

```HTTP
GET /items/articles?filter[title][_eq]=Hello World
```

**Después**:

```HTTP
SEARCH /items/articles

{
	"query": {
		"filter": {
			"title": {
				"_eq": "Hello World"
			}
		}
	}
}
```

Hay mucha discusión sobre si se debe o no poner un cuerpo en una solicitud GET, usar POST para crear consultas de búsqueda o confiar en un método completamente diferente. A partir de ahora, hemos optado por alinearnos con la especificación del método HTTP SEARCH de IETF. Si bien reconocemos que esto todavía es un borrador de especificaciones, el método SEARCH se ha utilizado ampliamente antes en el mundo WebDAV (especificación) y, en comparación con las otras opciones disponibles, se siente como el "más limpio" y correcto para manejar esto en el futuro. Al igual que con todo lo demás, si tiene alguna idea, opinión o inquietud, nos encantaría conocer sus pensamientos.

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