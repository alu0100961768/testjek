---
layout: page
title: Práctica 1- p0-t0-esprima-logging. Fases de un compilador
permalink: /documentation-p1/
---


## Índice

- [Práctica: p0-t0-esprima-logging. Fases de un compilador.](#práctica-p0-t0-esprima-logging-fases-de-un-compilador)
	- [Índice](#índice)
	- [Objetivos](#objetivos)
	- [Árboles  sintácticos](#árboles--sintácticos)
		- [Esprima](#esprima)
		- [ATS Explorer](#ats-explorer)
		- [Aplicación práctica](#aplicación-práctica)
			- [Objetivos](#objetivos-1)
			- [Procedimiento](#procedimiento)
				- [Resolviendo con Regexp](#resolviendo-con-regexp)
				- [Resolviendo con esprima](#resolviendo-con-esprima)
					- [Localizar los nodos función](#localizar-los-nodos-función)
					- [Acceder a los nodos función](#acceder-a-los-nodos-función)
					- [Modificar los hijos de nuestro modo función](#modificar-los-hijos-de-nuestro-modo-función)
					- [Transformar nuestro arbol de nuevo a código](#transformar-nuestro-arbol-de-nuevo-a-código)
	- [Commander.js](#commanderjs)
		- [Instalación](#instalación)
		- [Opciones](#opciones)
			- [Declarar opciones](#declarar-opciones)
			- [Parsear opciones](#parsear-opciones)
			- [Mostrar ayuda](#mostrar-ayuda)
		- [Ejecutar opciones](#ejecutar-opciones)
	- [Herramientas Adicionales](#herramientas-adicionales)
	- [Autor](#autor)



## Objetivos

**Descripcion de la práctica:** https://ull-esit-pl-1819.github.io/introduccion/tema0-introduccion-a-pl/practicas/p0-t0-esprima-logging/

- [X] Aceptar la asignación en Classroom de esta tarea.

- [X] Envie su github y su alu en el formulario facilitado por el profesor.

- [X] Entregar el enlace al repositorio en el campus virtual.

- [X] Ejecutar paso a paso el código de `logging.js` usando el debugger de chromium, intentando comprender el funcionamiento de la transformación realizada. Hacer un resumen de lo aprendido en el fichero Markdown.

- [X] Modificar el programa para que los `console.log` insertados informen de los valores de los parámetros pasados a la función.

- [X] Reto.




## Árboles  sintácticos

### Esprima

[Esprima](https://esprima.org) es una herramienta para realizar análisis léxico y sintáctico de programas JavaScript. Durante esta práctica, trabajaremos con esta herramienta como módulo de Node.js.

Una vez iniciado node, instalamos Esprima via npm `npm install esprima`. También necesitaremos instalar los módulos  `estraverse` y `escodegen`. Finalmente, importaremos dichos nodos usando la función `require('module')`.

**La funcionalidad de cada módulo es la siguiente**:

* **esprima:** Es nuestro parser. Transforma una cadena que contiene código JavaScript en un arbol sintáctico (que se comporta como un objeto JS).

* **estraverse:** Nos permitirá recorrer el arbol sintáctico generado por `esprima`, recorriendo sus nodos y deteniéndonos en los que nos interesan. Tambien nos permitirá modificar dicho arbol, añadiendo, eliminando o modificando nodos de la forma que indiquemos.

* **escodegen:**  Permite reconstruir el codigo JavaScript a partir del arbol sintactico de dicho código.

  

### ATS Explorer

[ATS Explorer](https://astexplorer.net/#/gist/b5826862c47dfb7dbb54cec15079b430/161f9396189f8e0bae46de9d5e1fb94a2ce87f24) es una herramienta que nos permite parsear un código JavaScript en su respectivo arbol sintáctico de forma cómoda y ágil. Gracias a ella, podemos explorar el arbol sintáctico que generará nuestro código JavaScript via web, sin necesidad de ejecutar un programa o usar el debugger.  

ATS Explorer nos da una gran multitud de lenguajes y analizadores sintácticos para construir los arboles sintácticos del código. En el caso de JavaScript, si bien no incluye directamente esprima, si que incluye **espree**. Como esprima y espree han sido diseñados para ser compatibles el uno con el otro, utilizaremos espree. 



### Aplicación práctica

#### Objetivos

Uno de los objetivos de esta práctica es modificar un codigo JavaScript dado por referencia para insertar en el mismo un `console.log` avisándonos de cuando hemos entrado en  una función, así como cuales son las variables con las que hemos accedido a la misma.

Ejemplo:

Dado el siguiente input:

```javascript
function foo(a, b) {
  var x = 'blah';
  var y = (function (z) {
    return z+3;
  })(2);
}
foo(1, 'wut', 3);
```

Nuestro programa debe devolver:

```javascript
function foo(a, b) {
    console.log(`Entering foo(${ a },${ b })`);
    var x = 'blah';
    var y = function (z) {
        console.log(`Entering <anonymous function>(${ z })`);
        return z + 3;
    }(2);
}
foo(1, 'wut', 3);
```

#### Procedimiento

##### Resolviendo con Regexp

Una primera idea podría ser el no utilizar esprima y resolver este problema mediante el uso de expresiones regulares.  

Pero, como reza el dicho popular:

> Some people, when confronted with a problem, think "I know, I'll use regular expressions." Now they have two problems.
>
> ***"*Jaime Zawinski**

Al enfrentarnos al problema con expresiones regulares, nos topamos con que no debemos tan solo limitarnos a buscar una función. También debemos comprobar que dicha funcion no exista en una cadena, esté comentada o que no sea una verdadera función por algun motivo, pese a tener la estructura de función; y aún tenemos que lidiar con el requisito de mencionar que atributos estamos incluyendo en susodicha función...

##### Resolviendo con esprima

Así pues, una vez hemos descartado las funciones regulares, trataremos de resolver el probema usando esprima.

###### Localizar los nodos función

En primer lugar, exploraremos como funcionan las funciones en JavaScript usando ATS Explorer.

Tras probar diferentes códigos y explorar sus árboles, podemos llegar a la siguiente conclusión: Existen dos tipos diferentes de declaraciones de funciones: **FunctionDeclaration** y **FunctionExpression**.

**FunctionDeclaration** es una funcion que se ha declarado en el código, y que se espera utilizar más adelante durante la ejecución del mismo. Por ejemplo, `function bar(x, y){return x*y;}`

**FunctionExpression** es una funcion que puede ser almacenada en una variable, por ejemplo, `var x= function(x, y){return x*y};` .

###### Acceder a los nodos función

Nuestro objetivo es acceder a los nodos de FunctionDeclaration y FunctionExpression. Pero para ello, primero debemos transformar nuestro código JS a modificar en un arbol sintáctico. Para ello, usaremos la función `espree.parse(code, {ecmaVersion:6})`. Es importante que usemos ecmaVersion6 para que no tengamos errores a la hora de interpretar las cadenas entre los carácteres ""\`"".

Ahora que tenemos nuestro arbol sintáctico, podemos proceder a explorar sus nodos y detenernos en los nodos de funciones.

Para ello, usaremos la función `estraverse.traverse(arbol, {enter: funcion()})`, que nos permitirá recorrer nuestro arbol sintáctico nodo a nodo, y ejecutar una función al entrar a dicho nodo.

Una vez dentro del nodo, es tan facil como explorar si el tipo de nuestro nodo `node.type` es una función, y ejecutar una nueva función para incorporar un nuevo nodo  hijo a nuestro nodo función.

###### Modificar los hijos de nuestro modo función

Ahora que estamos en nuestro nodo función, es hora de modificarlo.

En primer lugar, podemos acceder al nombre de la función via `node.id.name`, y al vector de nombre de los atributos que le pasamos por referencia via `node.params`.

Primero, creamos una cadena con el mensaje que queremos mostrar utilizando la información de los parámetros del nodo función. Una vez tenemos nuestra cadena, debemos transformarla a nodo de un arbol usando `espree.parse`, ya que no podemos introducir código directamente en un arbol. Después, copiamos el nodo correspondiente a nuestra cadena y lo introducimos en `node.body.body` (donde se alamcena el código de nuestra función) haciendo uso de `node.body= stringNode.concat(node.body.body)`.

###### Transformar nuestro arbol de nuevo a código

Finalmente, una vez hemos modificado todos los nodos que neceistabamos, para transformar nuestro nuevo arbol sintáctico en código JS tan solo tendremos que usar `escodegen.generate(arbol)`. ¡Tachán! Nuestro programa ya puede modificar código para introducir un `console.log` al entrar en cada función, que nos anuncie con que variables estamos accediendo.  





## Commander.js

[Commander.js](https://github.com/tj/commander.js/) es un módulo de Node.js que nos permite manejar con comodidad los argumentos que recibimos como parámetros al ejecutar nuestro programa. Generalmente utilizartemos los parámetros recividos en `proccess.argv`



### Instalación

Para poder utilizar comander, debemos instalar su módulo via `npm install commander`. Posteriormente, lo incluiremos en nuestro código a través de un require `const program= require('commander')`.



### Opciones

Commander cuenta con una serie de opciones que nos permite especificar como debe reacionar el programa al recibir ciertos parámetros como argumentos.

#### Declarar opciones

| program.    | Explicación                                                  | Uso                                  |
| :---------- | :----------------------------------------------------------- | ------------------------------------ |
| version     | Devuelve la version actual del programa al ejecutar `node ejemplo -V`. | version("0.0.0")                     |
| description | Devuelve una descripción del programa.                       | description("programa de prueba")    |
| usage       | Extension de help donde podemos especificar como usar el programa correctamente. Se mostrará al usar `program.help`. | usage('[options] <filename\> [...]') |
| option      | Permite declarar una opción, como puede ser llamada y que argumentos es necesario que reciba para poder ser ejecutada. | option('-o, --output <filename\>')   |

Notese que en el ejemplo de`program.option`, podremos comprobar si la opcion ha sido flageada mediante un `if(program.output)`, ya que output es el nombre de dicha opción.

#### Parsear opciones

Para parsear un array de opciones, deberemos ejecutar la función `program.parse(process.argv)`. En este caso utilizaremos `process.argv` porque nos interesa utilizar los comandos que han sido pasado por línea de comandos al ejecutar el programa, pero podría ser posible utilizar otro array si así lo necesitasemos.

#### Mostrar ayuda

Podemos mostrar la auyda mediante `program.help()`. Esta funcion nos mostrará los posibles comandos reconocidos por el programa (aquellos que hayamos declarado en el objeto commander); asi como el uso del programa (definido en `program.usage`) 



 ### Ejecutar opciones

Una vez ejecutado el programa y parseada la linea de comandos, podemos especificar el comportamiento del programa teniendo en cuenta que instrucciones se han recibido o no por línea de comandos.

Si una opción no ha sido espeficicada, `program.nombreOpcion` valdrá `false`. Si la opción ha sido declarada junto a parámetros adicionales, estos se guardarán en `program.nombreOpcion`. En otro caso, `program.nombreOpcion` valdrá `true`.





## Herramientas Adicionales

Se han utilizado las siguientes herramientas externas para la resolución de la práctica.

* **[Visual Studio Code](https://code.visualstudio.com):**  Editor de texto enriquecido utilizado para la programación del código. 
* **[Typora](https://typora.io/#windows):** Editor de texto enriquecido orientado a Markdown utilizado para el desarrollo del README.md.
* **[Markdown-toc-generate](https://magnetikonline.github.io/markdown-toc-generate/):** Herramienta que permite generar facilmente una TOC de nuestro código Markdown.




## Autor

* **Alumno:** Germán Alfonso Teixidó.
* **Asignatura:** Procesadores de Lenguajes.
* **Fecha:** 12/2/2020.

* **alu:** alu0100961768.
