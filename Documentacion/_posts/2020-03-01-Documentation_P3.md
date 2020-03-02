---
layout: page
title: Práctica 3- Handling Events
permalink: /documentation-p3/
---

Apuntes de [eloquentJavascript](https://eloquentjavascript.net/15_event.html#c_TpxWIP8ylU)
## Tabla de contenidos

- [Práctica: Handling Events](#pr%C3%A1ctica-handling-events)
  - [Tabla de contenidos](#tabla-de-contenidos)
  - [Handlers](#handlers)
  - [Eventos y nodos DOM](#eventos-y-nodos-dom)
  - [Objetos evento](#objetos-evento)
  - [Propagación](#propagaci%C3%B3n)
  - [Acciones por defecto](#acciones-por-defecto)
  - [Eventos key](#eventos-key)
  - [Eventos de puntero](#eventos-de-puntero)
    - [Clicks de ratón](#clicks-de-rat%C3%B3n)
    - [Movimientos de ratón](#movimientos-de-rat%C3%B3n)
    - [Eventos táctiles](#eventos-t%C3%A1ctiles)
  - [Eventos de scroll](#eventos-de-scroll)
  - [Eventos Focus](#eventos-focus)
  - [Eventos Load](#eventos-load)
  - [Eventos y el evento LOOP](#eventos-y-el-evento-loop)
  - [Timers](#timers)
  - [Deboucing](#deboucing)
  - [Ejercicios resueltos](#ejercicios-resueltos)

## Handlers

Mecanísmo que permite que el sistema notifique activamente a nuestro código cuando ocurre un evento determinado.

Los navegadores nos permiten hacer esto permitiendo registrar funciones como handlers para eventos específicos:

```
<p>Click this document to activate the handler.</p>
<script>
  window.addEventListener("click", () => {
    console.log("You knocked?");
  });
</script>
```

## Eventos y nodos DOM

Los event handlers de cada navegador estan registrados en un contexto. En el ejemplo anterior, llamoamos a `addEventListener` en el objeto ventana para registrar un handler para toda la ventana. Dicho método puede ser encontrado en elementos DOM y otros tipos de objetos. Solo se llama a a Event listenercuando el evento ocurre en el contexto del objeto donde están registrados.

```
<button>Click me</button>
<p>No handler here.</p>
<script>
  let button = document.querySelector("button");
  button.addEventListener("click", () => {
    console.log("Button clicked.");
  });
</script>
```

Podemos usar `removeEventListener`para remover eventos que ya no queremos trackear.



## Objetos evento

A las funciones event handler se les pasa un argumento: el objeto evento. Este objeto contiene información adicional sobre el evento. Por ejemplo, si queremos saber qué botón del ratón se ha pulsado, podemos mirar la propiedad button del objeto evento.

```
<button>Click me any way you want</button>
<script>
  let button = document.querySelector("button");
  button.addEventListener("mousedown", event => {
    if (event.button == 0) {
      console.log("Left button");
    } else if (event.button == 1) {
      console.log("Middle button");
    } else if (event.button == 2) {
      console.log("Right button");
    }
  });
</script>
```



## Propagación

Para la mayoría de tipos de eventos, los handlers registrados en nodos con hijos también recibirán los eventos que ocurran en el hijo. Si un botón dentro de un párrafo es clickado, les event handlers del párrafo también recibirán el evento de clickado.

Pero si ambos parrafo y botón tienen un handler, el handler más específico irá primero. Decimos que el evento se propaga hacia el exterior, desde el nodo donde ha ocurrido hasta el nodo padre, y así hasta el nodo raiz.

En cualquier momento, un event handler puede llamar a `stopPropagation` en el evento objeto para prevenir futuros handlers recibiendo el evento. Esto puede ser util, por ejemplo, cuando tenemos un botón dentro de otro elemento clickeable y no quieres que los clicks del botón activen el comportamiento del clickable exterior.

```
<p>A paragraph with a <button>button</button>.</p>
<script>
  let para = document.querySelector("p");
  let button = document.querySelector("button");
  para.addEventListener("mousedown", () => {
    console.log("Handler for paragraph.");
  });
  button.addEventListener("mousedown", event => {
    console.log("Handler for button.");
    if (event.button == 2) event.stopPropagation();
  });
</script>
```

La mayoría de objetos evento tienen una propiedad `target` que refiere al nodo donde se originaron. Podemos usar resta propiedad para asegurarte de que no estas manejando acidentalmente algo que se ha propagado de un nodo que no quieres manejar.

Tambien es posible usar la propiedad target para lanzar una red para un evento específico. Por ejemplo, si tienes un nodo contieniendo una larga lista de botones, quizas sea más conveniente registrar un único clicker handler en el nodo exterior y dejar que use la propiedad `target` de forma adecuada para figurar que botón ha sido clickado, en lugar de registrar handlers únicos para cada uno de los botones.

```
<button>A</button>
<button>B</button>
<button>C</button>
<script>
  document.body.addEventListener("click", event => {
    if (event.target.nodeName == "BUTTON") {
      console.log("Clicked", event.target.textContent);
    }
  });
</script>
```



## Acciones por defecto

Muchos eventos tienen una ación por defecto asociada con ellos. Si clickas un link, seras redirigido al target del link. Si pulsas una flecha, el navegador desplazará la página hacia abajo. Si haces click derecho, obtendrás un menú. Etc.

Para la mayoría de eventos, los handlers de eventos de JavaScript se llaman antes de que el comportamiento por defecto tome lugar. Si el handler no quiere que ocurra este comprtamiento, (normalmente porque ya se ha encargado de administrar el evento), puede llamar al metodo `preventDefault`.

```
<a href="https://developer.mozilla.org/">MDN</a>
<script>
  let link = document.querySelector("a");
  link.addEventListener("click", event => {
    console.log("Nope.");
    event.preventDefault();
  });
</script>
```

preventDefault se considera mala praxis. Es mejor evitar usarlo salvo  que tengas una buena razón para ello.



## Eventos key

Cuando una tecla de tu teclado es presionada, tu navegador lanza un evento "keydown". Cuando deja de ser presionada, obtienes un evento "keyup".

```
<p>This page turns violet when you hold the V key.</p>
<script>
  window.addEventListener("keydown", event => {
    if (event.key == "v") {
      document.body.style.background = "violet";
    }
  });
  window.addEventListener("keyup", event => {
    if (event.key == "v") {
      document.body.style.background = "";
    }
  });
</script>
```

A pesar de su nombre, "keydown" no sólo se dispara cuando la llave se presiona físicamente hacia abajo. Cuando una tecla se presiona y se mantiene presionada, el evento se dispara de nuevo cada vez que la tecla se repite. Debemos tener cuidado con esto. Por ejemplo, si añadimos un botón al DOM cuando se pulsa una tecla y lo quitamos al soltar dicha tecla, podemos añadir accidentalmente cientos de botones cuando la tecla se mantiene pulsada más de un instante.

En el ejemplo se observó la propiedad de la tecla del objeto del evento para ver de qué tecla se trata. Esta propiedad contiene una cadena que, para la mayoría de las teclas, corresponde a lo que al pulsar esa tecla se escribiría. Para teclas especiales como la de "Enter", contiene una cadena que nombra la tecla ("Enter", en este caso). Si mantienes pulsada la tecla shift mientras pulsas una tecla, eso también puede influir en el nombre de la tecla: "v" se convierte en "V", y "1" puede convertirse en "!", si eso es lo que produce pulsar shift-1 en tu teclado (Existen diferentes distribuciones de teclados segun el idioma).

Las teclas modificadoras como shift, control, alt y meta (comando en Mac) generan eventos de teclas como las teclas normales. Pero cuando busques combinaciones de teclas, también puedes averiguar si estas teclas se mantienen pulsadas mirando las propiedades de las teclas shiftKey, ctrlKey, altKey y metaKey de los eventos de teclado y ratón.

```
<p>Press Control-Space to continue.</p>
<script>
  window.addEventListener("keydown", event => {
    if (event.key == " " && event.ctrlKey) {
      console.log("Continuing!");
    }
  });
</script>
```

Cuando el usuario está escribiendo un texto, usar eventos key para averiguar que escribe puede ser problemático. Algunas plataformas (como el teclado virtual de los teléfonos Android) no disparan eventos key. E incluso en teclados anticuados, algunos tipos de entrada de texto no coinciden con las pulsacionesm de teclas de forma directa, como el IME (imput method editor - software de editor de métodos de entrada), utilizado por personas cuyos scripts no pueden ser abarcados en un teclado, y donde se combinan múltiples pulsaciones de telcas para crear carácteres.

Para notar cuando algo fue escrito, los elementos en los que se puede escribir, como las etiquetas \<input> y \<textarea>, disparan eventos de "entrada" cada vez que el usuario cambia su contenido. Para obtener el contenido real que fue escrito, es mejor leerlo directamente desde el campo de enfoque.



## Eventos de puntero

Hay dos formas principales de apuntar a cosas en una pantalla: ratón y pantallas táctiles. Ambas producen diferentes tipos de eventos.

### Clicks de ratón

Pulsar una tecla de ratón dispara un cierto número de eventos: `mousedown` y `mouseup` son similares a "keydown" y "keyup" y se disparan cuando el boton es presionado y soltado. 

Justo despues del evento "mouseup", un evento `click` es dusoaradi en el nodo más específico que contenga tanto el evento de presionado y soltado del ratón. Por ejemplo, si presiono el botón del ratón en un párrafo y luego muevo el puntero a otro párrafo y suelto el botón, el evento de "click" ocurrirá en el elemento que contiene ambos párrafos.

Si dos clicks ocurren de forma cercana y rápdiamente, un evento `dblclick` también es disparado, justo después del segundo evento "click".

Para obtener información precisa sobre el lugar en el que se produjo un evento del ratón, puede consultar sus propiedades clientX y clientY, que contienen las coordenadas del evento (en píxeles) relativas a la esquina superior izquierda de la ventana, o pageX y pageY, que son relativas a la esquina superior izquierda de todo el documento (que pueden ser diferentes cuando se ha desplazado la ventana).

```
<style>
  body {
    height: 200px;
    background: beige;
  }
  .dot {
    height: 8px; width: 8px;
    border-radius: 4px; /* rounds corners */
    background: blue;
    position: absolute;
  }
</style>
<script>
  window.addEventListener("click", event => {
    let dot = document.createElement("div");
    dot.className = "dot";
    dot.style.left = (event.pageX - 4) + "px";
    dot.style.top = (event.pageY - 4) + "px";
    document.body.appendChild(dot);
  });
</script>
```



### Movimientos de ratón

Cada vez que el puntero del ratón se mueve, se dispara un evento de `mousemove`. Este evento puede ser usado para rastrear la posición del ratón. Una situación común en la que esto es útil es cuando se implementa algún tipo de funcionalidad de arrastre del ratón.

```
<p>Drag the bar to change its width:</p>
<div style="background: orange; width: 60px; height: 20px">
</div>
<script>
  let lastX; // Tracks the last observed mouse X position
  let bar = document.querySelector("div");
  bar.addEventListener("mousedown", event => {
    if (event.button == 0) {
      lastX = event.clientX;
      window.addEventListener("mousemove", moved);
      event.preventDefault(); // Prevent selection
    }
  });

  function moved(event) {
    if (event.buttons == 0) {
      window.removeEventListener("mousemove", moved);
    } else {
      let dist = event.clientX - lastX;
      let newWidth = Math.max(10, bar.offsetWidth + dist);
      bar.style.width = newWidth + "px";
      lastX = event.clientX;
    }
  }
</script>
```



### Eventos táctiles

Los eventos táctiles funcionan de forma diferente a los de un ratón. En esta práctica, no profundizaremos en eventos táctiles, pero algunos ejemplos son: `tochstart`, `tocuhend`, `touchmove` (similares a los eventos de ratón pero distintos en el sentido de ser únicos y tener solo un valor).

```
<style>
  dot { position: absolute; display: block;
        border: 2px solid red; border-radius: 50px;
        height: 100px; width: 100px; }
</style>
<p>Touch this page</p>
<script>
  function update(event) {
    for (let dot; dot = document.querySelector("dot");) {
      dot.remove();
    }
    for (let i = 0; i < event.touches.length; i++) {
      let {pageX, pageY} = event.touches[i];
      let dot = document.createElement("dot");
      dot.style.left = (pageX - 50) + "px";
      dot.style.top = (pageY - 50) + "px";
      document.body.appendChild(dot);
    }
  }
  window.addEventListener("touchstart", update);
  window.addEventListener("touchmove", update);
  window.addEventListener("touchend", update);
</script>
```



## Eventos de scroll

Cuando se desplaza un elemento, este dispara un evento `scroll`. Este tiene varios usos, como conocer que está mirando el usuario o mostrar alguna indicación de progreso.

```
<style>
  #progress {
    border-bottom: 2px solid blue;
    width: 0;
    position: fixed;
    top: 0; left: 0;
  }
</style>
<div id="progress"></div>
<script>
  // Create some content
  document.body.appendChild(document.createTextNode(
    "supercalifragilisticexpialidocious ".repeat(1000)));

  let bar = document.querySelector("#progress");
  window.addEventListener("scroll", () => {
    let max = document.body.scrollHeight - innerHeight;
    bar.style.width = `${(pageYOffset / max) * 100}%`;
  });
</script>
```



## Eventos Focus

Cuando un elemento gana focus, es resaltado y dispara un evento `focus`. Cuando pierde focus, obtiene un evento `blur`.

```
<p>Name: <input type="text" data-help="Your full name"></p>
<p>Age: <input type="text" data-help="Your age in years"></p>
<p id="help"></p>

<script>
  let help = document.querySelector("#help");
  let fields = document.querySelectorAll("input");
  for (let field of Array.from(fields)) {
    field.addEventListener("focus", event => {
      let text = event.target.getAttribute("data-help");
      help.textContent = text;
    });
    field.addEventListener("blur", event => {
      help.textContent = "";
    });
  }
</script>
```



## Eventos Load

``load``: Cuando una página termina de cargar, el evento "load" se dispara en la ventana y los objetos del cuerpo del documento. Esto se suele usar para programar acciones de inicialización que requieren que todo el documento haya sido construido.

Los elementos como las imágenes y las etiquetas de los guiones que cargan un archivo externo también tienen un evento "load" que indica que los archivos a los que hacen referencia fueron cargados. Al igual que los eventos relacionados con el enfoque, los eventos de carga no se propagan.

Cuando se cierra una página, se dispara un evento `beforeunload`. El uso principal de este evento es evitar que el usuario pierda trabajo accidentalmente al cerrar un documento. Si se evita el comportamiento por defecto de este evento y se establece la propiedad `returnValue` en el objeto del evento en una cadena, el navegador mostrará al usuario un diálogo preguntando si realmente quiere abandonar la página. Ese cuadro de diálogo puede incluir dicha cadena, pero como algunos sitios maliciosos intentan utilizar estos cuadros de diálogo para confundir a la gente para que permanezca en su página, la mayoría de los navegadores ya no los muestran.



## Eventos y el evento LOOP

Los event handlers se comportan como otras notificaciones asíncronas. Estan programadas para que el evento ocurra pero deba esperar por otros scrpits que estan siendo ejecutados antes de que tengan una oportunidad de ejecutarse.

El hecho de que que los eventos puedan ser procesados cuando nada más se está ejecutnado significa que si el evento loop está atado con otro trabajo, cualquier interacción con la página será retrasada hasta que no hay más tiempo para procesarlo.

Para los casos en los que realmente quieres hacer algo que consume mucho tiempo en segundo plano sin congelar la página, los navegadores proporcionan web workers. Un web worker es un proceso JavaScript que se ejecuta junto con el guión principal, en su propia limeline.

Imagina que elevar un número al cuadrado es un cálculo pesado y de larga duración que queremos realizar en un hilo separado. Podríamos escribir un archivo llamado `code/squareworker.js` que responda a los mensajes computando un cuadrado y enviando un mensaje de vuelta.

```
addEventListener("message", event => {
  postMessage(event.data * event.data);
});
```

Para evitar el problema de tener mútiples hilos tocando los mismos datos, los web workers no comparten su scope ni ningún otro tipo de datos con el scrpit ejecutandose en el  entorno principal. En su lugar, debemos comunicarlos enviando mensajes entre ellos.

```
let squareWorker = new Worker("code/squareworker.js");
squareWorker.addEventListener("message", event => {
  console.log("The worker responded:", event.data);
});
squareWorker.postMessage(10);
squareWorker.postMessage(24);
```

La función `postMesage`  envia un mensjae, que causará que se dispare un evento message en el receptor.



## Timers

A veces tienes que cancelar una funcion que habías programado, esto se hace alamacenando el valor retornado por `setTimeout` y llamaondo a `clearTimeout` en el.

```
let bombTimer = setTimeout(() => {
  console.log("BOOM!");
}, 500);

if (Math.random() < 0.5) { // 50% chance
  console.log("Defused.");
  clearTimeout(bombTimer);
}
```

La función `cancelAnimationFrame` funciona de la misma forma que `clearTimeout`: llamandola en un valor retornado por `requestAnimationFrame` que cancelará el frame. (asumiendo que aún no ha sido llamado).

Un conjunto similar de funciones, `setInterval` y `clearInterval` son usados para establecer timers que deberían repetirse cada X milisegundos.

```
let ticks = 0;
let clock = setInterval(() => {
  console.log("tick", ticks++);
  if (ticks == 10) {
    clearInterval(clock);
    console.log("stop.");
  }
}, 200);
```



## Deboucing

Algunos tipos de eventos tienen el potencial de dispararse rápidamente, muchas veces seguidas ("mousemove", "scroll"). Cuando manejamos esos eventos, debemos ser cuidadosos de no realizar ninguna actividad que consuma mucho tiempo, o tu handler tardará tanto tiempo que interactuar con el documento se sentirá lento.

Si necesitas hacer algo no trivial en dicho handler, puedes usar `setTimeout` para asegurarte de que no lo haces con demasiada frecuencia. A esto se le suele llamar rebotar el evento. Hay varios enfoques ligeramente diferentes para esto.

En el primer ejemplo, queremos reaccionar cuando el usuario ha escrito algo, pero no queremos hacerlo inmediatamente para cada evento de entrada. Cuando escribimos rapidamente, queremos esperar hasta que se produzca una pausa. En lugar de realizar inmmediatamente una acción en el event handler, establecemos un tiempo de espera. Tambien borramos el tiempo de espera anterior (si lo hay), de forma que cuando los eventos se produzcan muy próximos entre sí, se cancelará el tiempo de espera del evento anterior.

```
<textarea>Type something here...</textarea>
<script>
  let textarea = document.querySelector("textarea");
  let timeout;
  textarea.addEventListener("input", () => {
    clearTimeout(timeout);
    timeout = setTimeout(() => console.log("Typed!"), 500);
  });
</script>
```

Dar un valor indefinido a `clearTimeout` o llamarlo en un timeout que ya ah sido disparado no tendrá ningun efecto. Así pues, no tenemos que ser cuidadosos sobre cuando llamarlo, y tan solo tenemos que preocuparnos por hacerlo para cada evento.

Podemos usar un patron ligeramente diferente si queremos espaciar respuestas de form que estén separadas por al menos cierta cantidad de tiempo pero queremos que se disparen durante una serie de eventos, no jsuto después. Por ejemplo, quizas queramos responder al evento "mousemove" enseñando las actuales coordenadasdel raton, pero solo cada 250 milisegundos.

```
<script>
  let scheduled = null;
  window.addEventListener("mousemove", event => {
    if (!scheduled) {
      setTimeout(() => {
        document.body.textContent =
          `Mouse at ${scheduled.pageX}, ${scheduled.pageY}`;
        scheduled = null;
      }, 250);
    }
    scheduled = event;
  });
</script>
```


## Ejercicios resueltos:

- [Mouse trail]({{site.baseurl}}/mousetrail)
- [Censored Keyboard]({{site.baseurl}}/censoredkeyboard)
- [Tabs]({{site.baseurl}}/tabs)





