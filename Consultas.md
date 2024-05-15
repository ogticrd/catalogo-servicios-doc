# Parámetros de consulta global

> La mayoría de las operaciones de punto de conexión de la API de Catalogo se pueden manipular con los siguientes parámetros. Es importante entenderlos para sacar el máximo partido a la plataforma.

## Campos

Elija los campos que se devuelven en el conjunto de datos actual. Este parámetro admite la notación de puntos para solicitar campos relacionales anidados. También puede utilizar un comodín (*) para incluir todos los campos a una profundidad específica.

### Ejemplos

- Obtener todos los campos de nivel superior: `*`
- Obtener todos los campos de nivel superior y todos los campos relacionales de segundo nivel: `*.*`

> **Rendimiento y tamaño**
>
> Aunque el comodín `fields` es muy útil para fines de depuración, se recomienda solicitar solo campos específicos para su uso en producción. Al solicitar solo los campos que realmente necesita, puede acelerar la solicitud y reducir el tamaño total de la salida.

- Obtener todos los campos de nivel superior y los campos relacionales de segundo nivel dentro de las imágenes: `*,images.*`
- Obtener solo los campos `name` y `description`: `name,description`
- Obtenga todos los campos relacionales de nivel superior y de segundo nivel, y los campos de tercer nivel dentro de `images.thumbnails`: `*.*,images.thumbnails.*`

#### Many-To-Any (tipos de unión)
Al ver que los campos Many-to-Any (M2A) tienen datos anidados de varias colecciones, no siempre es seguro o se desea obtener el mismo campo de todas las colecciones relacionadas. En los campos M2A, puede usar la siguiente sintaxis para especificar qué campos obtener de qué tipo de colección anidada relacionada: `?fields=<m2a-field>:<collection-scope>.<field>`

Supongamos que tenemos una colección `pages` con un campo de muchos a cualquier llamado `sections` que apunta a `headings`, `paragraphs` y `videos`. Solo queremos fetch `title` y `level` from `headings`, `body` from `paragraphs` y `source` from `videos`. Podemos lograrlo usando:

```
sections.item:headings.title
sections.item:headings.level
sections.item:paragraphs.body
sections.item:videos.source
```

Quedaria de esta manera:

```
GET /items/articles
	?fields[]=title
	&fields[]=sections.item:headings.title
	&fields[]=sections.item:headings.level
	&fields[]=sections.item:paragraphs.body
	&fields[]=sections.item:videos.source
```

## Filtro

Se utiliza para buscar elementos en una colección que coincida con las condiciones del filtro. El parámetro filter sigue la especificación Reglas de filtro, que incluye información adicional sobre operadores lógicos (AND/OR), filtrado relacional anidado y variables dinámicas.

### Ejemplos
Recupera todos los elementos donde es igual a "Rijk"first_name

```JSON
{
	"first_name": {
		"_eq": "Rijk"
	}
}
```

Recupere todos los elementos de una de las siguientes categorías: "verduras", "frutas"

```JSON
{
	"categories": {
		"_in": ["vegetables", "fruit"]
	}
}
```

Recuperar todos los elementos que se publican entre dos fechas

```JSON
{
	"date_published": {
		"_between": ["2021-01-24", "2021-02-23"]
	}
}
```

Recupere todos los elementos en los que la marca "vip" del autor es verdadera

```JSON
{
	"author": {
		"vip": {
			"_eq": true
		}
	}
}
```

> **Filtros anidados**
>
> En el ejemplo anterior se filtrarán los elementos de nivel superior en función de una condición del elemento relacionado. Si quieres filtrar los elementos relacionados, ¡echa un vistazo al parámetro deep!

```
?filter[first_name][_eq]=Rijk

// or

?filter={ "first_name": { "_eq": "Rijk" }}
```

## Buscar

El parámetro de búsqueda permite realizar una búsqueda en todos los campos de tipo cadena y texto de una colección. Es una manera fácil de buscar un artículo sin crear filtros de campo complejos, aunque está mucho menos optimizado. Solo busca en los campos del elemento raíz, no se incluyen los campos de elementos relacionados.

### Ejemplo

Buscar todos los artículos que mencionan `Catalogo`

```
?search=Catalogo
```

## Ordenar

Por qué campo(s) ordenar. La ordenación predeterminada es ascendente, pero se puede usar un signo menos (-) para revertir esto a orden descendente. Los campos se priorizan según el orden del parámetro. La notación de puntos debe usarse cuando se ordena con valores de campos anidados.

### Ejemplos

- Ordenar por fecha de creación descendente: `date_created`
- Ordene por un campo de "ordenación", seguido de la fecha de publicación descendente: `sort,-publish_date`
- Ordenar por un campo de "ordenación", seguido del nombre de un autor anidado: `sort,-author.name`

```
?sort=sort,-date_created,author.name

// or

?sort[]=sort
&sort[]=-date_created
&sort[]=-author.name
```

## Límite

Establezca el número máximo de artículos que se devolverán. El límite predeterminado se establece en `100`.

### Ejemplos

- Consigue los primeros 200 artículos: `200`
- Obtener el número máximo permitido de artículos: `-1`

> **Máximo de artículos**
> 
> Dependiendo del tamaño de la colección, obtener la cantidad máxima de elementos puede dar lugar a un rendimiento degradado o a tiempos de espera mas largos, úselo con precaución.

```
?limit=200
```

## Omitir

Omita los primeros `n` elementos de la respuesta. Se puede utilizar para la paginación.

### Ejemplos

Consigue los objetos 101—200

```
?offset=100
```

## Página

Una alternativa a `offset`. La página es una forma de establecer `offset` bajo el capó mediante el cálculo de `limit * page`. La página está indexada 1.

### Ejemplos

Obtener artículos 101-200

```
?page=2
```

## Agregación y agrupación

Las funciones de agregado le permiten realizar cálculos en un conjunto de valores, devolviendo un único resultado.

Las siguientes funciones de agregación están disponibles en Catalogo:

| Nombre         | Descripción                                          |
|---|---|
| count          | Cuenta cuántos elementos hay                            |
| countDistinct  | Cuenta cuántos objetos únicos hay                       |
| sum            | Suma los valores del campo dado                          |
| sumDistinct    | Suma los valores únicos en el campo dado                 |
| avg            | Obtener el valor promedio del campo dado                  |
| avgDistinct    | Obtener el valor promedio de los valores únicos en el campo dado |
| min            | Devuelve el valor más bajo del campo                    |
| max            | Devuelve el valor más alto del campo                     |

```
?aggregate[count]=*
```

### Agrupación

De forma predeterminada, las funciones de agregación anteriores se ejecutan en todo el conjunto de datos. Para permitir informes más flexibles, puede combinar la agregación anterior con la agrupación. La agrupación permite ejecutar las funciones de agregación en función de un valor compartido. Esto permite cosas como "Calificación promedio por mes" o "Ventas totales de artículos en la categoría de jeans".

La consulta permite agrupar varios campos simultáneamente. En combinación con las funciones, esto permite la generación de informes agregados por año-mes-fecha.groupBy

```
?aggregate[count]=views,comments
&groupBy[]=author
&groupBy[]=year(publish_date)
```

### Profundo

Deep le permite establecer cualquiera de los otros parámetros de consulta en un conjunto de datos relacional anidado.

#### Ejemplos

Limite los artículos relacionados anidados a 3

```JSON
{
	"related_articles": {
		"_limit": 3
	}
}
```

Solo obtenga 3 artículos relacionados, con solo el comentario mejor calificado anidado

```JSON
{
	"related_articles": {
		"_limit": 3,
		"comments": {
			"_sort": "rating",
			"_limit": 1
		}
	}
}
```

```
?deep[translations][_filter][languages_code][_eq]=en-US


// or

?deep={ "translations": { "_filter": { "languages_code": { "_eq": "en-US" }}}}
```

## Alias

Los alias le permiten cambiar el nombre de los campos sobre la marcha y solicitar el mismo conjunto de datos anidados varias veces utilizando diferentes filtros.

> **Campos anidados**
> 
> Solo es posible asignar alias a los campos del mismo nivel.
> El alias de los campos anidados, por ejemplo `field.nested`, no funcionará.

```
?alias[all_translations]=translations
&alias[dutch_translations]=translations
&deep[dutch_translations][_filter][code][_eq]=nl-NL
```

## Exportar

Guarde la respuesta de API actual en un archivo. Acepta uno de `csv`, `json`, `xml`, `yaml`.

```
?export=csv
?export=json
?export=xml
?export=yaml
```

## Funciones

Las funciones permiten la modificación "en vivo" de los valores almacenados en un campo. Las funciones se pueden usar en cualquier parámetro de consulta que normalmente proporcionaría una clave de campo, incluidos los campos, la agregación y el filtro.

Las funciones se pueden usar envolviendo la clave de campo en una sintaxis similar a JavaScript, por ejemplo:

`timestamp` -> `year(timestamp)`

### Funciones de fecha y hora

| Filtro | Descripción |
|---|---|
| `year` | Extraer el año de un campo de fecha y hora/fecha/marca de tiempo |
| `month` | Extraer el mes de un campo datetime/date/timestamp |
| `week` | Extraer la semana de un campo datetime/date/timestamp |
| `day` | Extraer el día de un campo de fecha y hora/fecha/marca de tiempo |
| `weekday` | Extraer el día de la semana de un campo datetime/date/timestamp |
| `hour` | Extraer la hora de un campo datetime/date/timestamp |
| `minute` | Extraer el minuto de un campo de fecha y hora/fecha/marca de tiempo |
| `second` | Extraiga el segundo de un campo de fecha y hora/fecha/marca de tiempo |


### Funciones de matriz

| Filtro | Descripción |
|---|---|
| `count` | Extraer el número de elementos de una matriz JSON o un campo relacional |

```
?fields=id,title,weekday(date_published)
&filter[year(date_published)][_eq]=2021
```

