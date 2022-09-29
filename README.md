
### Escuela Colombiana de Ingeniería
### Arquiecturas de Software
## Juan Camilo Molina Leon 

## Construción de un cliente 'grueso' con un API REST, HTML5, Javascript y CSS3. Parte I.

### Trabajo individual o en parejas. A quienes tuvieron malos resultados en el parcial anterior se les recomienda hacerlo individualmente.

![](img/mock.png)

* Al oprimir 'Get blueprints', consulta los planos del usuario dado en el formulario. Por ahora, si la consulta genera un error, sencillamente no se mostrará nada.
* Al hacer una consulta exitosa, se debe mostrar un mensaje que incluya el nombre del autor, y una tabla con: el nombre de cada plano de autor, el número de puntos del mismo, y un botón para abrirlo. Al final, se debe mostrar el total de puntos de todos los planos (suponga, por ejemplo, que la aplicación tienen un modelo de pago que requiere dicha información).
* Al seleccionar uno de los planos, se debe mostrar el dibujo del mismo. Por ahora, el dibujo será simplemente una secuencia de segmentos de recta realizada en el mismo orden en el que vengan los puntos.


## Ajustes Backend

1. Incluya dentro de las dependencias de Gradle (build.gradle) los 'webjars' de jQuery y Bootstrap (esto permite tener localmente dichas librerías de JavaScript al momento de construír el proyecto):

    ```
    dependencies { 
		...
		compile group: 'org.webjars', name: 'webjars-locator', version: '0.14'
        compile group: 'org.webjars', name: 'bootstrap', version: '4.1.2'
        compile group: 'org.webjars', name: 'jquery', version: '3.1.0'
    }               
    ```

	**A continuación, se incluyeron las siguientes dependencias de Gradle en el build.gradle, en el cual se agregaron los 'webjars' de jQuery y Bootstrap de la siguiente forma.**

	```gradle
	group 'edu.eci.arsw'

	buildscript {
	    ext {
		springBootVersion = '2.0.0.BUILD-SNAPSHOT'
	    }
	    repositories {
		mavenCentral()
		maven { url "https://repo.spring.io/snapshot" }
		maven { url "https://repo.spring.io/milestone" }
	    }
	    dependencies {
		classpath("org.springframework.boot:spring-boot-gradle-plugin:${springBootVersion}")
	    }
	}

	apply plugin: 'java'
	apply plugin: 'eclipse'
	apply plugin: 'org.springframework.boot'
	apply plugin: 'io.spring.dependency-management'

	version = '0.0.1-SNAPSHOT'
	sourceCompatibility = 1.8

	repositories {
	    mavenCentral()
	    maven { url "https://repo.spring.io/snapshot" }
	    maven { url "https://repo.spring.io/milestone" }
	}

	dependencies {
		compile group: 'org.webjars', name: 'webjars-locator', version: '0.14'
		compile group: 'org.webjars', name: 'bootstrap', version: '4.1.2'
		compile group: 'org.webjars', name: 'jquery', version: '3.1.0'
		compile('org.springframework.boot:spring-boot-starter-web')
		testCompile group: 'junit', name: 'junit', version: '4.12'
	}
	```

## Front-End - Vistas

1. Cree el directorio donde residirá la aplicación JavaScript. Como se está usando SpringBoot, la ruta para poner en el mismo contenido estático (páginas Web estáticas, aplicaciones HTML5/JS, etc) es:  

    ```
    src/main/resources/static
    ```

	**A continuación, se crea el directorio donde residirá la aplicación JavaScript en ```src/main/resources/static```, y asimismo, se crea la carpeta js en ese mismo directorio.**

	![img](https://github.com/Skullzo/ARSW-Lab6/blob/main/img/Vistas1.PNG)

2. Cree, en el directorio anterior, la página index.html, sólo con lo básico: título, campo para la captura del autor, botón de 'Get blueprints', campo donde se mostrará el nombre del autor seleccionado, [la tabla HTML](https://www.w3schools.com/html/html_tables.asp) donde se mostrará el listado de planos (con sólo los encabezados), y un campo en donde se mostrará el total de puntos de los planos del autor. Recuerde asociarle identificadores a dichos componentes para facilitar su búsqueda mediante selectores.

	**En el directorio ```src/main/resources/static```, se ha creado la página index.html, con título, campo para la captura del autor y botón de 'Get blueprints' o 'Obtener Planos' en español. También se habilitó un campo en donde se muestra el nombre del autor seleccionado, donde se muestra el listado de planos y el total de puntos de los planos del autor. También se han habilitado identificadores para facilitar su búsqueda mediante selectores. El código del HTML ha quedado de la siguiente forma.**

	```html
	<!DOCTYPE html>
	<html lang="en">
	<style type="text/css">
	    form{
		margin: 20px 0;
	    }
	    form input, button{
		padding: 5px;
	    }
	    table{
		width: 40%;
		margin-bottom: 20px;
		border-collapse: collapse;
	    }
	    table, th, td{
		border: 1px solid #cdcdcd;
	    }
	    table th, table td{
		padding: 10px;
		text-align: left;
	    }
	</style>
	<head>
	    <title>Blueprints</title>
	</head>
	<body>
	<h1>Planos</h1>
	<div>
	    <div >
		<a>Autor:</a>
		<input type="text" id="autor">
		<button type="button"  onclick="app.plansAuthor()">Obtener Planos</button>
		</br>
		</br>
		<body>
		<label>Planos del autor:</label>
		<table id="tabla">
		    <thead>
		    <tr>
			<th>Nombre del plano</th>
			<th>Puntos</th>
			<th>Abrir</th>
		    </tr>
		    </thead>
		    <tbody>
		    <tr>
			<td id="nombreActor">La Monalisa</td>
			<td id="puntos">25</td>
			<td type="button" value=nombreActor onclick="app.drawPlan()">Abrir</td>
		    </tr>
		    </tbody>
		</table>
		<label>Puntos totales de usuario:</label>
		<label id="puntosLabel">0</label>
		</br>
		</br>
		<canvas id="myCanvas" width="500" height="500" style="border:1px solid #000000;">
		</canvas>
		</body>
	    </div>
	</div>
	</body>
	</html>
	```

3. En el elemento \<head\> de la página, agregue las referencia a las librerías de jQuery, Bootstrap y a la hoja de estilos de Bootstrap. 
    ```html
    <head>
        <title>Blueprints</title>
        <meta charset="UTF-8">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">

        <script src="/webjars/jquery/3.1.0/jquery.min.js"></script>
        <script src="/webjars/bootstrap/4.1.2/js/bootstrap.min.js"></script>
        <link rel="stylesheet"
          href="/webjars/bootstrap/4.1.2/css/bootstrap.min.css" />
    ```
    
	**Para realizar el siguiente procedimiento, se modificó el HTML de la página creada en el numeral anterior, pero esta ves agregando la referencia a las librerías de jQuery, Bootstrap y a la hoja de estilos de Bootstrap, quedando el código del HTML de la siguiente forma.**

	```html
	<!DOCTYPE html>
	<html lang="en">
	<style type="text/css">
	    form{
		margin: 20px 0;
	    }
	    form input, button{
		padding: 5px;
	    }
	    table{
		width: 40%;
		margin-bottom: 20px;
		border-collapse: collapse;
	    }
	    table, th, td{
		border: 1px solid #cdcdcd;
	    }
	    table th, table td{
		padding: 10px;
		text-align: left;
	    }
	</style>
	<head>
	    <meta charset="UTF-8">
	    <meta name="viewport" content="width=device-width, initial-scale=1.0">
	    <script src="/webjars/jquery/3.1.0/jquery.min.js"></script>
	    <script src="/webjars/bootstrap/4.1.2/js/bootstrap.min.js"></script>
	    <link rel="stylesheet"
		  href="/webjars/bootstrap/4.1.2/css/bootstrap.min.css" />
	    <title>Blueprints</title>
	</head>
	<body>
	<h1>Planos</h1>
	<div>
	    <div >
		<a>Autor:</a>
		<input type="text" id="autor">
		<button type="button"  onclick="app.plansAuthor()">Obtener Planos</button>
		</br>
		</br>
		<body>
		<label>Planos del autor:</label>
		<table id="tabla">
		    <thead>
		    <tr>
			<th>Nombre del plano</th>
			<th>Puntos</th>
			<th>Abrir</th>
		    </tr>
		    </thead>
		    <tbody>
		    <tr>
			<td id="nombreActor">La Monalisa</td>
			<td id="puntos">25</td>
			<td type="button" value=nombreActor onclick="app.drawPlan()">Abrir</td>
		    </tr>
		    </tbody>
		</table>
		<label>Puntos totales de usuario:</label>
		<label id="puntosLabel">0</label>
		</br>
		</br>
		<canvas id="myCanvas" width="500" height="500" style="border:1px solid #000000;">
		</canvas>
		</body>
	    </div>
	</div>
	</body>
	</html>
	```

4. Suba la aplicación (mvn spring-boot:run), y rectifique:

   1. Que la página sea accesible desde:
   
		```
		http://localhost:8080/index.html
		```

		**Luego de subir la aplicación utilizando el comando ```mvn spring-boot:run```, en el navegador se ingresa la URL: ```http://localhost:8080/index.html```, y al acceder a esa URL se puede visualizar la página creada en HTML.**

		![img](https://github.com/Skullzo/ARSW-Lab6/blob/main/img/Vistas4.1.PNG)
    
   2. Al abrir la consola de desarrollador del navegador, NO deben aparecer mensajes de error 404 (es decir, que las librerías de JavaScript se cargaron correctamente).

		**Luego de subir la aplicación y haber ingresado a la URL ```http://localhost:8080/index.html```, se procede a abrir la consola de desarrollador del navegador, y como se puede ver a continuación, no aparece ningún mensaje de error 404, demostrando que las librerías de JavaScript se cargaron correctamente.**

		

## Front-End - Lógica

1. Ahora, va a crear un Módulo JavaScript que, a manera de controlador, mantenga los estados y ofrezca las operaciones requeridas por la vista. Para esto tenga en cuenta el [patrón Módulo de JavaScript](https://toddmotto.com/mastering-the-module-pattern/), y cree un módulo en la ruta static/js/app.js .

	**A continuación, en la ruta ```static/js``` se crea un módulo JavaScript que a manera de controlador mantiene los estados y ofrece las operaciones requeridas por la vista, quedando de la siguiente forma.**

	

2. Copie el módulo provisto (apimock.js) en la misma ruta del módulo antes creado. En éste agréguele más planos (con más puntos) a los autores 'quemados' en el código.

	**Ahora copiamos el módulo provisto que es ```apimock.js``` en la misma ruta del módulo ```app.js```. Inicialmente el código del ```apimock.js``` sin realizarle ningún tipo de modificación es el siguiente.**

	```javascript
	var apimock = (function () {

	    var mockdata = [];

	    mockdata["JhonConnor"] = [
		{
		    author: "JhonConnor",
		    name: "house",
		    points: [
			{
			    x: 10,
			    y: 20
			},
			{
			    x: 15,
			    y: 25
			},
			{
			    x: 45,
			    y: 25
			}
		    ]
		},
		{
		    author: "JhonConnor",
		    name: "bike",
		    points: [
			{
			    x: 30,
			    y: 35
			},
			{
			    x: 40,
			    y: 45
			}
		    ]
		}
	    ]

	    mockdata['LexLuthor'] = [
		{
		    author: 'LexLuthor',
		    name: 'kryptonite',
		    points: [
			{
			    x: 60,
			    y: 65
			},
			{
			    x: 70,
			    y: 75
			}
		    ]
		}
	    ]

	    return {
		getBlueprintsByAuthor: function(author, callback) {
		    callback(null, mockdata[author]);
		},

		getBlueprintsByNameAndAuthor: function(name, author, callback) {
		    blueprint = mockdata[author].find(function(blueprint) {
			return blueprint.name == name
		    });
		    callback(null, blueprint)
		}
	    }

	})();
	```

3. Agregue la importación de los dos nuevos módulos a la página HTML (después de las importaciones de las librerías de jQuery y Bootstrap):
    ```html
    <script src="js/apimock.js"></script>
    <script src="js/app.js"></script>
    ```

	**Para realizar la respectiva importación de los dos nuevos módulos a la página HTML ya anteriormente creada, agregamos tanto el script de ```apimock.js``` como el script de ```app.js``` debajo de las importaciones de las librerías de jQuery y Bootstrap, quedando el código de ```index.html``` de la siguiente forma.**

	```html
	<!DOCTYPE html>
	<html lang="en">
	<style type="text/css">
	    form{
		margin: 20px 0;
	    }
	    form input, button{
		padding: 5px;
	    }
	    table{
		width: 40%;
		margin-bottom: 20px;
		border-collapse: collapse;
	    }
	    table, th, td{
		border: 1px solid #cdcdcd;
	    }
	    table th, table td{
		padding: 10px;
		text-align: left;
	    }
	</style>
	<head>
	    <meta charset="UTF-8">
	    <meta name="viewport" content="width=device-width, initial-scale=1.0">
	    <script src="/webjars/jquery/3.1.0/jquery.min.js"></script>
	    <script src="/webjars/bootstrap/4.1.2/js/bootstrap.min.js"></script>
	    <script src="js/apimock.js"></script>
	    <script src="js/app.js"></script>
	    <link rel="stylesheet"
		  href="/webjars/bootstrap/4.1.2/css/bootstrap.min.css" />
	    <title>Blueprints</title>
	</head>
	<body>
	<h1>Planos</h1>
	<div>
	    <div >
		<a>Autor:</a>
		<input type="text" id="autor">
		<button type="button"  onclick="app.plansAuthor()">Obtener Planos</button>
		</br>
		</br>
		<body>
		<label>Planos del autor:</label>
		<table id="tabla">
		    <thead>
		    <tr>
			<th>Nombre del plano</th>
			<th>Puntos</th>
			<th>Abrir</th>
		    </tr>
		    </thead>
		    <tbody>
		    <tr>
			<td id="nombreActor">La Monalisa</td>
			<td id="puntos">25</td>
			<td type="button" value=nombreActor onclick="app.drawPlan()">Abrir</td>
		    </tr>
		    </tbody>
		</table>
		<label>Puntos totales de usuario:</label>
		<label id="puntosLabel">0</label>
		</br>
		</br>
		<canvas id="myCanvas" width="500" height="500" style="border:1px solid #000000;">
		</canvas>
		</body>
	    </div>
	</div>
	</body>
	</html>
	```

4. Haga que el módulo antes creado mantenga de forma privada:
    * El nombre del autor seleccionado.
    * El listado de nombre y tamaño de los planos del autor seleccionado. Es decir, una lista objetos, donde cada objeto tendrá dos propiedades: nombre de plano, y número de puntos del plano.

    Junto con una operación pública que permita cambiar el nombre del autor actualmente seleccionado.

	**Para hacer que el módulo mantenga de forma privada el nombre del autor seleccionado y el listado de nombre y tamaño de los planos del autor seleccionado, primero se realizan modificaciones al archivo ```apimock.js```, en el ```return```, se realiza el siguiente cambio para asimismo junto con una operación pública, permita cambiar el nombre del autor actualmente seleccionado.**

	```javascript
	return {
		getBlueprintsByAuthor:function(name, callback) {
		    callback(
			mockdata[name]
		    )
		},
		getBlueprintsByNameAndAuthor:function(autor,obra,callback){
		    callback(
			mockdata[autor].filter(prueb => {return prueb.name === obra;})[0]
		    );
		}
	}
	```

5. Agregue al módulo 'app.js' una operación pública que permita actualizar el listado de los planos, a partir del nombre de su autor (dado como parámetro). Para hacer esto, dicha operación debe invocar la operación 'getBlueprintsByAuthor' del módulo 'apimock' provisto, enviándole como _callback_ una función que:

    * Tome el listado de los planos, y le aplique una función 'map' que convierta sus elementos a objetos con sólo el nombre y el número de puntos.

	    **A continuación, en la función ```_funcModify``` se toma el listado de los planos, y se almacena en una variable llamada ```arreglo```, en la cual se le aplica una función 'map' a los planos o 'blueprints'. Luego se procede a convertir sus elementos a objetos con sólo el nombre y el número de puntos con una variable ```temporal``` que se encarga de almacenar el nombre  con su respectiva ```blueprint.key``` que representa la llave del nombre, y los puntos con su respectivo ```blueprint.value``` que respresenta los puntos de cada plano, agregándolos con un ```.append``` al ```arreglo```, para así retornar el valor total de los puntos con su respectivo autor. El código de la función queda de la siguiente forma.**

		```javascript
		var _funcModify = function (variable) {
			if(variable != null){
			    var arreglo = variable.map(function(blueprint){
				return {key:blueprint.name, value:blueprint.points.length}
			    })
			    $("#tabla tbody").empty();
			    arreglo.map(function(blueprint){
				var temporal = '<tr><td id="nombreActor">'+blueprint.key+'</td><td id="puntos">'+blueprint.value+'</td><td type="button" onclick="app.drawPlan(\''+blueprint.key+'\')">Open</td></tr>';
				$("#tabla tbody").append(temporal);
			    })

			    var valorTotal = arreglo.reduce(function(total, valor){
				return total.value + valor.value;
			    })
			    document.getElementById("autorLabel").innerHTML = author;
			    document.getElementById("puntosLabel").innerHTML = valorTotal;
			}
		};
		```

    * Sobre el listado resultante, haga otro 'map', que tome cada uno de estos elementos, y a través de jQuery agregue un elemento \<tr\> (con los respectvos \<td\>) a la tabla creada en el punto 4. Tenga en cuenta los [selectores de jQuery](https://www.w3schools.com/JQuery/jquery_ref_selectors.asp) y [los tutoriales disponibles en línea](https://www.tutorialrepublic.com/codelab.php?topic=faq&file=jquery-append-and-remove-table-row-dynamically). Por ahora no agregue botones a las filas generadas.

		**Ahora, se creó la función ```_funcDraw```, que se encarga de sobre el listado resultante, realizar otro 'map' que toma cada uno de los elementos, y a través de jQuery agrega un elemento \<tr\> con sus respectivos \<td\>, para así con la tabla creada en el punto 4, mapear cada uno de estos elementos. La función fue implementada de la siguiente forma.**

		```javascript
		var _funcDraw = function (vari) {
			if (vari) {
			    var lastx = null;
			    var lasty = null;
			    var actx = null;
			    var acty = null;
			    var c = document.getElementById("myCanvas");
			    var ctx = c.getContext("2d");

			    ctx.clearRect(0, 0, 500, 500);
			    ctx.beginPath();

			    vari.points.map(function (prue){
				if (lastx == null) {
				    lastx = prue.x;
				    lasty = prue.y;
				} else {
				    actx = prue.x;
				    acty = prue.y;
				    ctx.moveTo(lastx, lasty);
				    ctx.lineTo(actx, acty);
				    ctx.stroke();
				    lastx = actx;
				    lasty = acty;
				}
			    });
			}
		}
		```

    * Sobre cualquiera de los dos listados (el original, o el transformado mediante 'map'), aplique un 'reduce' que calcule el número de puntos. Con este valor, use jQuery para actualizar el campo correspondiente dentro del DOM.

		**Como tal en el ```app.js``` se implementó la funcion callback que junto con el nombre del mockdata se le pasa como parámetros a la función ```getBlueprintsByAuthor``` para que esta se ejecute de forma asíncrona. Para esto, fue implementado de último el ```return```, quedando de la siguiente forma.**

		```javascript
		return {
			    plansAuthor: function () {
				author = document.getElementById("autor").value;
				apimok.getBlueprintsByAuthor(author,_funcModify);

			    },

			    drawPlan: function(name) {
				author = document.getElementById("autor").value;
				obra = name;
				apimok.getBlueprintsByNameAndAuthor(author,obra,_funcDraw);
			    }
		};
		```

6. Asocie la operación antes creada (la de app.js) al evento 'on-click' del botón de consulta de la página.

7. Verifique el funcionamiento de la aplicación. Inicie el servidor, abra la aplicación HTML5/JavaScript, y rectifique que al ingresar un usuario existente, se cargue el listado del mismo.

	**Para verificar el funcionamiento de la aplicación, primero se inicia el servidor, y al ingresar el primer autor que es ```JhonConnor``` se muestra el nombre de los planos que son ```house``` y ```bike``` con su respectivo puntaje, valores que fueron insertados en el ```apimock.js```. La aplicación los retorna en la tabla de la siguiente forma.**

	

	**Para verificar que la aplicación también carga el nombre del plano y los puntos del autor ```LexLuthor```, este es ingresa en el campo provisto. Luego realizar clic en ```Obtener Planos```, se muestra el nombre del plano que es ```kryptonite``` con sus respectivos puntos, como se muestra a continuación.**

	

## Para la próxima semana

8. A la página, agregue un [elemento de tipo Canvas](https://www.w3schools.com/html/html5_canvas.asp), con su respectivo identificador. Haga que sus dimensiones no sean demasiado grandes para dejar espacio para los otros componentes, pero lo suficiente para poder 'dibujar' los planos.

	**Para agregar un elemento de tipo Canvas con su respectivo identificador, donde sus dimensiones no sean demasiado grandes para dejar espacio para los otros componentes, y suficiente para poder dibujar los planos, se implementó en ```app.js``` la función ```_funcDraw``` para 'dibujar los planos', en los que se implementa un mapeo para poder realizar los respectivos dibujos tomando las coordenadas ```x``` y ```y``` para realizar el respectivo dibujo. La implementación de esta función quedó de la siguiente forma.**

	```javascript
	var _funcDraw = function (vari) {
		if (vari) {
		    var lastx = null;
		    var lasty = null;
		    var actx = null;
		    var acty = null;
		    var c = document.getElementById("myCanvas");
		    var ctx = c.getContext("2d");

		    ctx.clearRect(0, 0, 500, 500);
		    ctx.beginPath();

		    vari.points.map(function (prue){
			if (lastx == null) {
			    lastx = prue.x;
			    lasty = prue.y;
			} else {
			    actx = prue.x;
			    acty = prue.y;
			    ctx.moveTo(lastx, lasty);
			    ctx.lineTo(actx, acty);
			    ctx.stroke();
			    lastx = actx;
			    lasty = acty;
			}
		    });
		}
	}
	```

9. Al módulo app.js agregue una operación que, dado el nombre de un autor, y el nombre de uno de sus planos dados como parámetros, haciendo uso del método getBlueprintsByNameAndAuthor de apimock.js y de una función _callback_:
    * Consulte los puntos del plano correspondiente, y con los mismos dibuje consectivamente segmentos de recta, haciendo uso [de los elementos HTML5 (Canvas, 2DContext, etc) disponibles](https://www.w3schools.com/html/tryit.asp?filename=tryhtml5_canvas_tut_path)* Actualice con jQuery el campo <div> donde se muestra el nombre del plano que se está dibujando (si dicho campo no existe, agruéguelo al DOM).

		**A continuación, se agrega una operación donde dado el nombre del autor y el nombre de uno de sus planos, este usa el método de ```getBlueprintsByNameAndAuthor``` de ```apimock.js```, la cual es implementada en el ```return``` de ```app.js``` para su respectiva actualización con jQuery del campo ```div``` donde se muestra el nombre del plano que se está dibujando.**

		```javascript
		return {
			    plansAuthor: function () {
				author = document.getElementById("autor").value;
				apimok.getBlueprintsByAuthor(author,_funcModify);

			    },

			    drawPlan: function(name) {
				author = document.getElementById("autor").value;
				obra = name;
				apimok.getBlueprintsByNameAndAuthor(author,obra,_funcDraw);
			    }
		};
		```

10. Verifique que la aplicación ahora, además de mostrar el listado de los planos de un autor, permita seleccionar uno de éstos y graficarlo. Para esto, haga que en las filas generadas para el punto 5 incluyan en la última columna un botón con su evento de clic asociado a la operación hecha anteriormente (enviándo como parámetro los nombres correspondientes).

11. Verifique que la aplicación ahora permita: consultar los planos de un auto y graficar aquel que se seleccione.

12. Una vez funcione la aplicación (sólo front-end), haga un módulo (llámelo 'apiclient') que tenga las mismas operaciones del 'apimock', pero que para las mismas use datos reales consultados del API REST. Para lo anterior revise [cómo hacer peticiones GET con jQuery](https://api.jquery.com/jquery.get/), y cómo se maneja el esquema de _callbacks_ en este contexto.

13. Modifique el código de app.js de manera que sea posible cambiar entre el 'apimock' y el 'apiclient' con sólo una línea de código.

14. Revise la [documentación y ejemplos de los estilos de Bootstrap](https://v4-alpha.getbootstrap.com/examples/) (ya incluidos en el ejercicio), agregue los elementos necesarios a la página para que sea más vistosa, y más cercana al mock dado al inicio del enunciado.

	**Luego de haber realizado todas las modificaciones para poder implementar un Canvas capaz de realizar los respectivos dibujos de los planos, el código final de ```app.js```, junto con todas sus funciones implementadas teniendo en cuenta el enunciado para poder cumplir con todos los requerimientos, queda de la siguiente forma.**

	```javascript
	app= (function (){
	    var _funcModify = function (variable) {
		if(variable != null){
		    var arreglo = variable.map(function(blueprint){
			return {key:blueprint.name, value:blueprint.points.length}
		    })
		    $("#tabla tbody").empty();
		    arreglo.map(function(blueprint){
			var temporal = '<tr><td id="nombreActor">'+blueprint.key+'</td><td id="puntos">'+blueprint.value+'</td><td type="button" onclick="app.drawPlan(\''+blueprint.key+'\')">Open</td></tr>';
			$("#tabla tbody").append(temporal);
		    })

		    var valorTotal = arreglo.reduce(function(total, valor){
			return total.value + valor.value;
		    })
		    document.getElementById("autorLabel").innerHTML = author;
		    document.getElementById("puntosLabel").innerHTML = valorTotal;
		}
	    };

	    var _funcDraw = function (vari) {
		if (vari) {
		    var lastx = null;
		    var lasty = null;
		    var actx = null;
		    var acty = null;
		    var c = document.getElementById("myCanvas");
		    var ctx = c.getContext("2d");

		    ctx.clearRect(0, 0, 500, 500);
		    ctx.beginPath();

		    vari.points.map(function (prue){
			if (lastx == null) {
			    lastx = prue.x;
			    lasty = prue.y;
			} else {
			    actx = prue.x;
			    acty = prue.y;
			    ctx.moveTo(lastx, lasty);
			    ctx.lineTo(actx, acty);
			    ctx.stroke();
			    lastx = actx;
			    lasty = acty;
			}
		    });
		}
	    }
	    return {
		    plansAuthor: function () {
			author = document.getElementById("autor").value;
			apimok.getBlueprintsByAuthor(author,_funcModify);

		    },

		    drawPlan: function(name) {
			author = document.getElementById("autor").value;
			obra = name;
			apimok.getBlueprintsByNameAndAuthor(author,obra,_funcDraw);
		    }
		};
	})();
	```


