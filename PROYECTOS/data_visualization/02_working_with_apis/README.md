# Visualizar Datos - Trabajar con APIs

<div id='index'></div>

* [Usar una API web](#usar-una-api-web)
* [Git y GitHub](#git-y-github)
    * [Solicitar datos realizando una llamada a la API](#solicitar-datos-realizando-una-llamada-a-la-API)
    * [Instalar Requests](#instalar-requests)
    * [Procesar la respuesta de una API](#procesar-la-respuesta-de-una-api)
    * [Trabajar con el diccionario de la respuesta](#trabajar-con-el-diccionario-de-la-respuesta)
    * [Resumir los repositorios principales](#resumir-los-repositorios-principales)
    * [Monitorizar los límites de cuota de la API](#monitorizar-los-límites-de-cuota-de-la-api)
    * [Visualizar repositorios usando Plotly](#visualizar-repositorios-usando-plotly)
    * [Modificar los gráficos](#modificar-los-gráficos)
    * [Añadir tooltips personalizados](#añadir-tooltips-personalizados)
    * [Añadir links clicables](#añadir-links-clicables)
    * [Más sobre Plotly y la API de GitHub](#más-sobre-plotly-y-la-api-de-github)
* [Hacker News API](#hacker-news-api)
* [Fin](#fin)

<br/>

<hr/><hr/><br/>

En esta sección, vamos a escribir un programa capaz de generar una visualización basada en datos que recupere de una API (*Application Programming Interface*) web, para solicitar automáticamente una información concreta a un sitio web y usar dichos datos para generar gráficos.

Como las APIs web son lugares que generalmente permanecen actualizados, utilizando APIs nuestra aplicación no se quedará obsoleta, ya que si se modifica el dato de la API, también lo hará la aplicación que hayamos creado.

<br/>

<br/><hr/>
<hr/><br/>

<div align="right">
    <a href="#index">Volver arriba</a>
</div>

# Usar una API web

Una API web es una parte de un sitio web designado a interactuar con programas. Dichos programas utilizan URLs muy concretas para solicitar la información. A esta acción se le llama *llamar a una API*.

La información vendrá facilitada en un formato tipo JSON, CSV o parecidos, es decir, formatos simples para permitir un fácil acceso a la información.

<br/>

<br/><hr/>
<hr/><br/>

<div align="right">
    <a href="#index">Volver arriba</a>
</div>

# Git y GitHub

Vamos a basar nuestras visualizaciones en información obtenida de [GitHub](https://github.com/), un sitio que, entre otras cosas, permite a los programadores colaborar en proyectos de código. Vamos a usar la API de GitHub para solicitar información acerca de proyectos de Python almacenados en la plataforma y realizar visualizaciones sobre la popularidad de dichos proyectos.

> GitHub recibe su nombre de [Git](https://git-scm.com/), un sistema de control de versiones. Git ayuda muchísimo a gestionar el trabajo sobre un proyecto, especialmente si se trabaja en grupo, puesto que el trabajo realizado por un sujeto, no perjudica ni modifica el trabajo realizado por otro.
>
> Al implementar una nueva *feature* en un proyecto, Git realiza el seguimiento de los cambios realizados en cada archivo del mismo.

<br/>

Cuando a los usuarios de GitHub les gusta un proyecto (*también conocido como **repositorio***), pueden añadirlo a favoritos o darle una *estrella*. En esta sección vamos a crear un programa capaz de descargar datos automáticamente de los proyectos más valorados sobre Python en GitHub, y crearemos una visualización de ello.

<br/>

<hr/><br/>

## Solicitar datos realizando una llamada a la API

GitHub permite solicitar una cantidad enorme de información a través de la API. Para saber cómo se ve una llamada a una API, introduce el siguiente link en tu navegador:

```
https://api.github.com/search/repositories?q=language:python&sort=stars
```

<br/>

Vamos a explicar este enlace por partes:

* `https://api.github.com/`: dirige la solicitud a la parte de GitHub que responde a las llamadas de la API.
* `search/repositories`: le indica a la API que dirija la búsqueda a través de los repositorios de GitHub (*es decir, que estamos buscando repositorios*).
* `?`: indica que vamos a pasar argumentos para realizar la búsqueda.
* `q=`: la `q` significa *query*, y el signo `=` nos permite comenzar a escribir una consulta. Al utilizar `language:python` decimos que sólo queremos información de aquellos repositorios cuyo lenguaje principal sea Python. Con `&sort=stars` ordenamos el resultado de la consulta en función de la cantidad de estrellas que tienen los repositorios.

<br/>

Realizando la llamada a través del navegador, verás que se encuentran varios millones de repositorios (*la cantidad devuelta en `total_count`*). Sabemos que se ha realizado correctamente la llamada si en la sección `incomplete_results` tenemos como resultado un `false` en lugar de un `true`.  Después, tendremos `items`, que son los que contienen la información de cada uno de los repositorios encontrados.

<br/>

<hr/><br/>

## Instalar Requests

El paquete `requests` permite a Python realizar solicitudes a una página web de forma muy sencilla y examinar la respuesta. Vamos a usar `pip` para instalarlo:

```powershell
# desde el directorio del proyecto
pip install requests
```

<br/>

<hr/><br/>

## Procesar la respuesta de una API

Ahora que hemos instalado `requests`, vamos a hacer uso de dicha librería y vamos a ver cómo gestionar la respuesta de una API. Para ello, crearemos un archivo llamado `python_repos.py` con el siguiente código:

```python
# python_repos.py

import requests

# Make an API call and store the response
url = "https://api.github.com/search/repositories?q=language:python&sort=stars"
headers = { "Accept": "application/vnd.github.v3+json" }

r = requests.get(url, headers=headers)
print(f"Status code: {r.status_code}")

# Store API response in a variable
response_dict = r.json()

# Process results
print(responde_dict.keys())
```

<br/>

Estos son los pasos que se han seguido escribiendo las líneas de código mostradas:

1. Se importa la librería `requests` para poder utilizarla en el proyecto.
2. Se guarda la URL de la llamada a la API en una variable para poder utilizarla a lo largo del archivo.
3. Definimos el encabezado para la API, de esta forma indicamos que queremos usar la versión 3 de la API de GitHub.
4. Usamos la librería `requests` para hacer la petición utilizando los parámetros previamente definidos.
5. El objeto respuesta, `r`, tiene un atributo llamado `status_code`, el cual nos indica si una petición ha sido satisfactoria o no. Imprimimos su valor para asegurarnos de que todo haya salido correctamente.
6. La API devuelve la información en el formato JSON, por lo que convertimos dicha información en un diccionario de Python con el método `json()`.
7. Finalmente, imprimimos las claves del diccionario, al cual hemos llamado `response_dict`.

<br/>

Tras ejecutar el programa, veremos que se obtiene lo siguiente:

```powershell
Status code: 200
dict_keys(['total_count', 'incomplete_results', 'items'])
```

<br/>

> Como el *Status code* tiene un valor de `200`, sabemos que todo ha salido correctamente.
>
> El diccionario que contiene la respuesta tiene tres claves:
>
> * `total_counts`
> * `incomplete_results`
> * `tems`

<br/>

<hr/><br/>

## Trabajar con el diccionario de la respuesta

Habiendo almacenado la respuesta de la API en un diccionario, podemos trabajar con los datos almacenados en el mismo. Vamos a generar una salida que resuma el contenido de la información obtenida:

```python
# python_repos.py

# ...

# Make an API call and store the response
# ...

# Store API responde in a variable
response_dict = r.json()

print(f"Total repositories: {response_dict['total_count']}")

# Explore information about the repositories
repo_dicts = response_dict["items"]
print(f"Repositories returned: {len(repo)dicts}")

# Examine the first repository
repo_dict = repo_dicts[0]
print(f"\nKeys: {len(repo_dict)}")

for key in sorted(repo_dict.keys()):
    print(key)
```

<br/>

Explicación:

1. Primero imprimimos el valor asociado al `total_count`, que representa el número total de repositorios de Python en GitHub.
2. El valor asociado a `items` es una lista que contiene una cantidad de diccionarios, donde cada uno contiene información acerca de un único repositorio. Lo que hacemos es almacenar la lista completa en `repo_dicts` e imprimimos la longitud de la lista para saber cuántos repositorios hemos recibido.
3. Obtenemos el primer repositorio de la lista y lo almacenamos en `repo_dict`.
4. Imprimimos la cantidad de claves que tiene dicho diccionario.
5. Imprimimos todas las claves del diccionario para saber qué información tenemos del mismo.

<br/>

Como resultado se obtiene lo siguiente:

```
Status code: 200
Total repositories: 9074612
Repositories returned: 30

Keys: 80
allow_forking
archive_url
archived
assignees_url
	...
url
visibility
watchers
watchers_count
web_commit_signoff_required
```

<br/>

Como se puede observar, la API de GitHub devuelve muchísima información de cada uno de los repositorios: hay `80` claves en `repo_dict`. Mirando estas claves, uno se puede hacer una idea de qué tipo de información puede extraer. Por ejemplo:

```python
# python_repos.py

# ...

# Examine the first repository
repo_dict = repo_dicts[0]


print("\nSelected information about first repository:")
print(f"Name: {repo_dict['name']}")						# name of the project
print(f"Owner: {repo_dict['owner']['login']}")			# owner is a dictionary, we get owner's login name
print(f"Stars: {repo_dict['stargazers_count']}")		# how many stars the project has earned
print(f"Repository: {repo_dict['html_url']}")			# url of the project's repository
print(f"Created: {repo_dict['created_at']}")			# when the repository was created
print(f"Updated: {repo_dict['updated_at']}")			# when was the last time the repository was updated
print(f"Description: {repo_dict['description']}")		# repository's description
```

<br/>

Este es el resultado obtenido (*el resultado varía en función de cuándo se haga la petición a la API, este es el resultado obtenido el día que se escribió esta guía*):

```
Status code: 200
Total repositories: 9463565
Repositories returned: 30

Selected information about first repository:
Name: public-apis
Owner: public-apis
Stars: 252728
Repository: https://github.com/public-apis/public-apis
Created: 2016-03-20T23:49:42Z
Updated: 2023-08-23T22:14:25Z
Description: A collective list of free APIs
```

<br/>

<hr/><br/>

## Resumir los repositorios principales

Ahora que sabemos cómo acceder a la información de un único repositorio, vamos a dar el siguiente paso: hacerlo con los repositorios más *top* de la lista. Para ello, vamos a escribir un bucle que imprima la información seleccionada de cada repositorio de la lista `repo_dicts`:

```python
# python_repos.py

# ...

# Explore information about the repositories
# repo_dicts = ...
# print(...)

print("\nSelected information about each repository:")
for repo_dict in repo_dicts:
    print("\nSelected information about first repository:")
    print(f"Name: {repo_dict['name']}")
    print(f"Owner: {repo_dict['owner']['login']}")
    print(f"Stars: {repo_dict['stargazers_count']}")
    print(f"Repository: {repo_dict['html_url']}")
    print(f"Created: {repo_dict['created_at']}")
    print(f"Updated: {repo_dict['updated_at']}")
    print(f"Description: {repo_dict['description']}")
```

<br/>

En primer lugar, imprimimos un mensaje introductorio, y, después, los mismos elementos informativos vistos anteriormente, esta vez de todos los ítems de la lista `repo_dicts`.

A primera vista, el resultado puede ser muy tedioso de leer, pero en seguida comenzaremos a realizar las visualizaciones necesarias para que esto no sea un problema.

<br/>

<hr/><br/>

## Monitorizar los límites de cuota de la API

La mayoría de las APIs tienen limitada la cantidad de llamadas que se les puede hacer en un período determinado de tiempo. Para saber si estamos alcanzando el límite de llamadas permitidas, debemos insertar este enlace en el navegador:

```
https://api.github.com/rate_limit
```

<br/>

Veremos que se muestra información en formato JSON, donde se indica:

* El límite de llamadas por minuto para cada sección de la API.
* La cantidad de llamadas restantes que tenemos.
* El tiempo que queda hasta que se reseteen los valores.

<br/>

Si hemos alcanzado el límite, solo se debe esperar un rato hasta que tengamos permitido volver a realizar las llamadas.

<br/>

<hr/><br/>

## Visualizar repositorios usando Plotly

Vamos a crear un gráfico interactivo con la información que hemos recopilado hasta ahora. Para ello, crearemos el archivo `python_repos_visual.py` donde copiaremos el código de `python_repos.py` para partir desde ese punto.

A continuación, añadiremos algunas líneas al código, de tal forma que el archivo al completo quede de la siguiente manera:

```python
# python_repos_visual.py

import requests

# (1)
from plotly.graph_objs import Bar
from plotly import offline

# Make an API call and store the response
url = "https://api.github.com/search/repositories?q=language:python&sort=stars"
headers = { "Accept": "application/vnd.github.v3+json" }

r = requests.get(url, headers=headers)
print(f"Status code: {r.status_code}")

# Process results
response_dict = r.json()
repo_dicts = response_dict["items"]

# (2)
repo_names, stars = [], []
for repo_dict in repo_dicts:
    repo_names.append(repo_dict["name"])
    stars.append(repo_dict["stargazers_count"])

# Make visualization
data = [{			# (3)
    "type": "bar",
    "x": repo_names,
    "y": stars
}]

my_layout = {		# (4)
    "title": "Most-Starred Python Projects on GitHub",
    "xaxis": { "title": "Repository" },
    "yaxis": { "title": "Stars" }
}

fig = { "data": data, "layout": my_layout }
offline.plot(fig, filename="./02_working_with_apis/git-github/python_repos.html")
```

<br/>

1. Hemos importado la clase `Bar` y el módulo `offline` de `plotly`.
2. Creamos dos listas vacías para almacenar el nombre de cada repositorio y la cantidad de estrellas que tiene.
3. Generamos la lista `data` a partir de los datos obtenidos. Indicamos qué tipo de gráfico va a ser y definimos los valores para los ejes `x` e `y`.
4. Definimos el *layout* del gráfico, donde se indican las etiquetas de los ejes y el título del mismo.

<br/>

Si ejecutamos el código, veremos que se genera un gráfico de barras donde se muestra la información mencionada.

<br/>

<hr/><br/>

## Modificar los gráficos

Viendo que el gráfico se ha generado correctamente, vamos a realizar un par de mejoras en los estilos del mismo.

Para comenzar, vamos a realizar las siguientes modificaciones en `data`, lo que cambiará el estilo de las barras:

```python
# python_repos_visual.py

# ...

# Make visualization
data = [{
    # ...
    "marker": {
        "color": "rgb(60, 100, 150)",
        "line": { "width": 1.5, "color": "rgb(25, 25, 25)" }
    },
    "opacity": 0.6,
}]

# ...
```

<br/>

Los ajustes añadidos afectan a las barras del gráfico. Hemos añadido un color azul y una línea externa de color gris de 1.5 píxeles de grosor. También les hemos dado una opacidad de `0.6` a las barras para que tengan un estilo más *suave*.

A continuación, vamos a modificar `my_layout`:

```python
# python_repos_visual.py

# ...

my_layout = {
    "title": "Most-Starred Python Projects on GitHub",
    "titlefont": { "size": 28 },
    "xaxis": {
        "title": "Repository",
        "titlefont": { "size": 24 },
        "tickfont": { "size": 14 }
    },
    "yaxis": {
        "title": "Stars",
        "titlefont": { "size": 24 },
        "tickfont": { "size": 14 }
    }
}

# ...
```

<br/>

* `titlefont`: con ello hemos definido el tamaño de la fuente.
* `tickfont`: para definir el tamaño de la fuente de las etiquetas.

<br/>

Solo hemos añadido tamaños de la fuente a los ajustes, sin embargo, podrían modificarse también características como el color del texto.

<br/>

<hr/><br/>

## Añadir tooltips personalizados

Lo que queremos conseguir (y la definición de *tooltip*) es que al hacer *hover* (*pasar el ratón*) por encima de las barras del gráfico, se muestre un mensaje emergente con la información del mismo.

En nuestro caso, el *tooltip* deberá mostrar tanto la descripción de cada repositorio como el creador del mismo.

Vamos a necesitar más datos para poder hacer esto, así que añadamos las siguientes líneas:

```python
# python_repos_visual.py

# ...
repo_names, stars, labels = [], [], []
for repo_dict in repo_dicts:
    # ...
    
    owner = repo_dict["owner"]["login"]
    description = repo_dict["description"]
    
    label = f"{owner}<br />{description}"
    labels.append(label)
    
data = [{
    # ...
    "hovertext": labels,
    # markers ...
}]
# ...
```

<br/>

1. Creamos una nueva lista vacía llamada `labels`.
2. Utilizamos el bucle `for` que ya habíamos definido anteriormente para obtener el `owner` y la `description` del repositorio.
3. Creamos una etiqueta (`label`) que sea un *string* que contenga ambos datos.
4. Añadimos la nueva etiqueta a la lista de etiquetas (`labels`).
5. Añadimos la opción `hovertext` dentro del diccionario de `data`, y le damos el valor de la lista que acabamos de crear.


<hr/><br/>


## Añadir links clicables

Como `Plotly` permite utilizar HTML en los elementos de texto, podemos añadir links de forma muy sencilla a los gráficos.

Vamos a utilizar el eje x para añadir links a los repositorios. Para ello, añadimos las siguientes líneas:

```python
# python_repos_visual.py

# ...
repo_links, stars, labels = [], [], []
for repo_dict in repo_dicts:
    # ...
    
    repo_name = repo_dict["name"]
    repo_url = repo_dict["html_url"]
    repo_link = f"<a href='{repo_url}'>{repo_name}</a>"
    repo_links.append(repo_link)
    
    stars.append(repo_dict["stargazers_count"])
    # ...

# Make visualization
data = [{
    # ...
    "x": repo_links,
    # ...
}]

# ...
```

<br/>

1. Creamos una nueva lista vacía llamada `repo_links`, sustituyendo la `repo_names` que teníamos antes.
2. Creamos una variable `repo_name` que almacene el nombre del repositorio.
3. Creamos una variable `repo_url` que almacene la URL del repositorio.
4. Creamos una variable `repo_link` que almacene un *string* con el link del repositorio.
5. Añadimos el link a la lista `repo_links`.
6. Añadimos la lista `repo_links` al diccionario `data` en el eje `x`.


<hr/><br/>


## Más sobre Plotly y la API de GitHub

Si quieres aprender más acerca de `Plotly`, puedes visitar las siguientes páginas:

* [Plotly User Guide in Python](https://plot.ly/python/user-guide/) - Muestra cómo Plotly usa tus datos para generar gráficos.
* [Python figure reference](https://plot.ly/python/reference/) - Muestra todas las opciones de configuración que se pueden utilizar para personalizar los gráficos.

<br/>

Si lo que quieres es más información acerca de la API de GitHub, puedes visitar la siguiente página:

* [GitHub API documentation](https://developer.github.com/v3/) - Muestra toda la información acerca de la API de GitHub.


<br/><hr/>
<hr/><br/>

<div align="right">
    <a href="#index">Volver arriba</a>
</div>

# Hacker News API

Para explorar cómo funcionan otras APIs, vamos a ver la API de [Hacker News](https://news.ycombinator.com/), un sitio web que muestra noticias sobre tecnología.

Esta API permite el acceso sin necesidad de registrarse para obtener claves. Este enlace muestra información sobre uno de los artículos de la misma:

[https://hacker-news.firebaseio.com/v0/item/19155826.json](https://hacker-news.firebaseio.com/v0/item/19155826.json)

<br/>

Para facilitar la visualización de la información json que se obtiene, vamos a hacer uso de `json.dumps()`:

```python
# hn_article.py

import requests
import json

# Make an API call and store the response
url = "https://hacker-news.firebaseio.com/v0/item/19155826.json"
r = requests.get(url)
print(f"Status code: {r.status_code}")

# Explore the structure of the data
response_dict = r.json()
readable_file = "data/readable_hn_data.json"
with open(readable_file, "w") as f:
    json.dump(response_dict, f, indent=4)
```

<br/>

Con este código hemos obtenido información acerca de el artículo de Hacker News con el ID `19155826` y la hemos almacenado en un archivo llamado `readable_hn_data.json`.

Vamos a analizar las claves más interesantes del archivo obtenido:

* La clave `descendants` indica el número de comentarios que tiene el artículo.
* La clave `kids` contiene una lista con los IDs de los comentarios.
* Tenemos la clave `title`, que contiene el título del artículo.

<br/>

El siguiente enlace nos permite obtener los artículos más populares de Hacker News:

[https://hacker-news.firebaseio.com/v0/topstories.json](https://hacker-news.firebaseio.com/v0/topstories.json)

<br/>

Podemos hacer uso de este enlace para obtener los IDs de los artículos más populares, y, después, utilizar el enlace anterior para obtener la información de cada uno de los artículos y mostrar un resumen de cada uno:

```python
# hn_submissions.py

from operator import itemgetter
import requests

# Make an API call and store the response
url = "https://hacker-news.firebaseio.com/v0/topstories.json"
r = requests.get(url)
print(f"Status code: {r.status_code}")

# Process information about each submission
submission_ids = r.json()
submission_dicts = []
for submission_id in submission_ids[:30]:
    # Make a separate API call for each submission
    url = f"https://hacker-news.firebaseio.com/v0/item/{submission_id}.json"
    r = requests.get(url)
    print(f"id: {submission_id}\tstatus: {r.status_code}")
    response_dict = r.json()

    # Build a dictionary for each article
    submission_dict = {
        "title": response_dict["title"],
        "hn_link": f"http://news.ycombinator.com/item?id={submission_id}",
        # "comments": response_dict["descendants"] -> esto devuelve un error si no hay comentarios
        # Mejoramos el código original añadiendo un valor por defecto:
        "comments": response_dict.get("descendants", 0)
        # Ahora, si no hay comentarios, se asigna un valor de 0 en lugar de devolver un error
    }
    submission_dicts.append(submission_dict)

submission_dicts = sorted(
    submission_dicts,
    key=itemgetter("comments"),
    reverse=True
)

for submission_dict in submission_dicts:
    print(f"\nTitle: {submission_dict['title']}")
    print(f"Discussion link: {submission_dict['hn_link']}")
    print(f"Comments: {submission_dict['comments']}")
```

<br/>

1. Hacemos una llamada a la API para obtener los IDs de los artículos más populares.
2. Creamos una lista vacía para almacenar los diccionarios de cada artículo.
3. Hacemos un bucle `for` para obtener la información de cada artículo.
4. Hacemos una llamada a la API para obtener la información de cada artículo.
5. Creamos un diccionario para cada artículo. - *En este caso, hemos añadido un valor por defecto para el número de comentarios, de tal forma que si no hay comentarios, se asigna un valor de 0 en lugar de devolver un error.*
6. Añadimos el diccionario a la lista de diccionarios.
7. Ordenamos la lista de diccionarios por el número de comentarios.
8. Imprimimos la información de cada artículo.


<br/><hr/><br/>


<div align="right">
    <a href="#index">Volver arriba</a>
</div>


# Fin

**Este es el final del proyecto.** Si has llegado hasta aquí, ¡enhorabuena! Has aprendido a generar gráficos a partir de datos obtenidos de diferentes formas, incluyendo APIs web.

Espero que hayas disfrutado de este proyecto tanto como yo. ¡Hasta la próxima!