# Acceso a los elementos

> Los elementos son datos individuales de la base de datos. Pueden ser cualquier cosa, desde artículos hasta comprobaciones de estado de IoT. 

## El objeto Item

Los elementos tienen un esquema predefinido. El formato depende completamente de los datos y estructuras definidas. 

> Datos relacionales
>
> De forma predeterminada, el objeto item no contiene datos relacionales anidados. Para recuperar datos anidados, consulte Datos relacionales y parámetros de campo.

## Obtener artículos

Enumera todos los elementos que existen en la colección definida.

### Request

```
GET /items/:collection

SEARCH /items/:collection
```

Si utiliza SEARCH, puede proporcionar un objeto de consulta como cuerpo de la solicitud. Más información sobre [SEARCH](README.md#método-http-search).

**Parámetros de consulta**

Admite todos los parámetros de consulta globales.

> Datos relacionales
>
> El parámetro Field es necesario para devolver datos relacionales anidados.

### Respuesta

Matriz de objetos de elemento hasta límite. Si no hay elementos disponibles, los datos serán una matriz vacía.

**Ejemplo**

```
SEARCH /items/services
```

## Obtener artículo por ID

Obtén un elemento que exista en en la colección definida.

### Request

```
GET /items/:collection/:id
```

**Parámetros de consulta**

Admite todos los parámetros de consulta globales.

### Respuesta

Devuelve un objeto de elemento si se proporcionó una clave principal válida.

**Ejemplo**

```
GET /items/services/15
```
