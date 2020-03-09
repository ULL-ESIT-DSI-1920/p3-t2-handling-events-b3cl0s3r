## Handling events - Chapter 15

### Handling events

```html
<button>Act-once button</button>
<script>
  let button = document.querySelector("button");
  function once() {
    console.log("Done.");
    button.removeEventListener("click", once);
  }
  button.addEventListener("click", once);
</script>
```

La función addEventListener permite añadir un handler a un determinado elemento. Lo que hace esto es que el sistema va a estar comprobando
continuamente que una determinada acción ocurre con dicho elemento. En caso de que se de, se dispara una acción.

```html
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

A las funciones de handler de eventos se les pasa siempre un argumento: el objeto "event".
Este objeto tiene información sobre el evento, como por ejemplo qué tecla o botón fue la que activó el evento.

La propagación de eventos ocurre de hijo a padres por defecto. Si se quiere evitar que se propague un evento, se puede llamar a
la función stopPropagation() del objeto.

Por ejemplo, en este ejemplo, el click izquierdo activa el evento en el párrafo y en el botón cuando se hace click en el botón.
Sin embargo, el click derecho solo se activa en el botón al hacer click sobre el botón debido a la función stopPropagation().


```html
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

Asimismo, en event también existe la propiedad 'target'. Con ella, podemos aplicar acciones en el origen del evento
y así evitar comportamientos no deseados debido a la propagación.

```html
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

Muchos eventos tienen eventos por defecto. Por ejemplo, si pulsas la flecha hacia abajo, el navegador baja,
si haces click derecho, aparece un menu...

Para la mayoría de eventos, los handlers de Javascript son ejecutados antes que las acciones por defecto. Con
preventDefault() podemos detener los comportamientos por defecto.

Hay que tener en cuenta que no se puede hacer con todo, por motivos de seguridad. Por ejemplo, en Chrome, no se puede detener
un CTRL+W (cerrar una pestaña).

```html
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

Cuando se pulsa una tecla, se recibe el evento keydown. Hay que tener cuidado, porque 
mantener la tecla pulsada dispara varias veces el evento keydown.

Así mismo, cuando se deja de pulsar una tecla, se recibe el evento keyup.

### Pointer events

Existen dos maneras de apuntar a cosas en la pantalla: ratón y pantallas táctiles.

Por un lado, tenemos los eventos "mousedown" y "mouseup" que funcionan de la misma manera que lo hace keydown y keyup.

Sin embargo, cuando se realiza un evento "mouseup", también se lanza un evento de tipo "click".

¿La diferencia? Que click abarca desde el inicio del pulso hasta el área en el que se dejó de pulsar y mouseup lanza el evento únicamente y exclusivamente en el punto en el que se soltó el botón del ratón.

Si se realizan dos clicks seguidos, ocurre un evento dblclick.

```html
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

Función básica para dibujar. Se tiene un listener del evento click que introduce un div con un punto en la última posición en la que se dejó de hacer click.

Básicamente, coloca puntos en la página.

![Captura1](screenshots/captura1.png?raw=true "Captura 1")

### Mouse motion

Cada vez que el ratón se mueve, se lanza un evento "mousemove". Este evento puede ser utilizado
para seguir la posición del ratón. Una situación típica de uso es en programas de dibujo.

Como ejemplo, el siguiente programa muestra una barra.

Tiene un handler que escucha si se ha pulsado el botón del ratón principal (el izquierdo). De ser así, va guardando la posición en la que se encuentra el ratón en la variable lastX. Después, añade un listener que está pendiente escuchando si el ratón se esta moviendo o no. 
Si no se mueve, no hace nada, pero en caso de que sí, calcula la nueva posición y se la asigna al div.

```html
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

![Captura2](screenshots/captura2.png?raw=true "Captura 2")

### Eventos touch

En una pantalla táctil, no podemos hacer tracking de los dedos de la persona. Además, cabe la posibilidad de tocar con varios dedos.

Tocar una pantalla táctical activa los eventos mousedown, mouseup y click. En algunos casos funciona bien, pero en el caso del rectángulo, por 
ejemplo, no sería lo ideal.

Existen eventos específicos: touchstart, touchmove y touchend, cuando se toca la pantalla, cuando tocas la pantalla y mueves el dedo y cuando dejas de tocar la pantalla.

Como la pantalla puede ser tocada por varios dedos, se tiene un array de puntos cada uno con sus propiedades clientX, clientY, pageX y pageY.


### Eventos de scroll

Para los eventos de scroll, existe el evento de scroll. Se pueden usar para seguir lo que está leyendo la persona.

### Eventos de focus

Cuando un elemento gana focus, se lanza un evento de focus. Cuando lo pierde, un evento de blur.

A diferencia de los otros eventos, estos dos eventos no avisan al elemento padre de que un elemento hijo ha ganado o perdido
el focus.

```html
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

### Evento de load

Cuando una página acaba de cargarse, se lanza un evento de load. Esto se usa frecuentemente para establecer la inicialización de una acción que requiere que la página se encuentre totalmente cargada.

Recordar que el contenido de las etiquetas script se ejecutan tan pronto como se encuentren. Elementos como scripts e imágenes tienen también
un evento de load, para indicar que los ficheros a los que referencian han sido cargados.

Cuando una página se cierra, se lanza un evento "beforeunload". El principal uso es prevenir de que un usuario pierda su progreso.

Si evitas el comportamiento por defecto y cambias la propiedad returnValue del evento del objeto a una cadena, el navegador mostrará que al usuario un dialogo preguntando si quieren salir realmente. Este diálogo podría incluir tu cadena, pero como algunas páginas maliciosas han usado estos diálogos para confundir a la gente para que se queden en su página, la mayoría de los navegadores no lo muestran.


### Eventos y el event loop

Los handlers del navegador se comportan como otras notificaciones asíncronas. 

Se programan para que ocurra un evento, pero debe esperar por otros scripts que están corriendo para acabar antes de que puedan ejecutarse.

El hecho de que un evento pueda ser procesado solo cuando nada más esta corriendo significa que, si el bucle de eventos está con otra tarea, cualquier interacción con la página se retrasará hasta que haya tiempo para procesarla. Por eso, si programas demasiadas tareas, con tareas muy pesadas o monton de tareas cortas, la página se volverá lenta y pesada de usar.

Para casos donde realmente quieres cosas que consuman tiempo en background sin congelar la página, los navegadores ofrecen algo llamado web workers. Un worker es un proceso en javascript que corre en paralelo al script principal.

Para evitar problemas de concurrencia, los workers no comparten ni su ámbito global ni cualquier otro tipo de datos. Solo puedes comunicarte con ellos enviando y recibiendo mensajes.

### Timers

setTimeout es una función que programa que una función se ejecute después de un tiempo.

A veces, puedes necesitar cancelar una función que has programado. Esto se puede hacer guardando en una variable
el valor retornado por setTimeout y llamando clearTimeout sobre él.

```javascript
<script>
let bombTimer = setTimeout(() => {
  console.log("BOOM!");
}, 500);

if (Math.random() < 0.5) { // 50% chance
  console.log("Defused.");
  clearTimeout(bombTimer);
}
</script>
```

Alternativamente, existe setInterval() y clearInterval (que funcionan exactamente de la misma manera (timers que ejecutan una tarea cada X segundos)
y requestAnimationFrame() y cancelAnimationFrame()


### Debouncing (rebote)

Algunos eventos tienen el potencial de dispararse muchas veces de manera repetida, como scroll y mousemove. Cuando los usas, hay que tener
cuidado con realizar tareas pesadas o la interacción empezará a ser lenta.

Si necesitas realizar algo no trivial, se puede utilizar setTimeOut() como función de debouncing. Esta serviría para asegurarte que la tarea no se repite con tanta frecuencia.

En este ejemplo, por ejemplo, mostramos el mensaje de Type mientras la persona teclea. Sin embargo, lo volveremos a mostrar únicamente cuando haya una pausa grande y se vuelva a escribir y así se evita que aparezca por cada tecla el mensaje "type".

```javascript
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

Un patrón ligeramente distinto lo tendríamos cuando movemos el ratón.

```javascript
<script>
  let scheduled = null;
  window.addEventListener("mousemove", event => {
    if (!scheduled) {
     setTimeout(() => {
        document.body.textContent =
          `Mouse at ${scheduled.pageX}, ${scheduled.pageY}`;
        scheduled = null;
      }, 250);*/
    }
    scheduled = event;
  });
</script>
```

En este ejemplo, cada vez que movemos el ratón mostramos su posición. Pero solo actualizamos el DOM cada 250ms para evitar que el evento
se dispare demasiadas veces.

## Jekyll

Después de instalar jekyll, para crear un nuevo proyecto debemos de hacer:

```bash
  jekyll new <nombreproyecto>
```

Ahora, debemos de hacer un build de la web con:

```bash
bundle exec jekyll  build -b "p3-t2-handling-events-b3cl0s3r"
```
Modificamos el package.json y añadimos un nuevo script, para automatizar el despliegue en github pages:

```json
"scripts": {
  "deploy": "gh-pages -d _site"
}
```

Finalmente, hacemos un deploy en github:

```bash
npm run deploy
```

Si accedemos a https://ull-esit-dsi-1920.github.io/p3-t2-handling-events-b3cl0s3r/ podremos ver la web desplegada.

![Captura3](screenshots/captura3.png?raw=true "Captura 3")

Asimismo, para configurar nuestra web, como el título y la descripción, debemos modificar el fichero _config.yml que sea crea con jekyll.

Ahí podemos añadir variables que luego podemos utilizar dentro de la página utilizando el lenguaje Liquid, un lenguaje de plantillas. Esta nos puede ayudar para definir como se construirá nuestra web estática.

### HTMLProofer y travis

HTMLProofer es un set de tests que valida el output de un HTML. Estos checks comprueban que las referencias de las imágenes son legitimas, que tienen las etiquetas alt, que los links internos funcionan... entre otros.

El mayor uso de HTMLProofer es para integración continua, para testear la calidad de un documento html.

![Captura4](screenshots/captura4.png?raw=true "Captura 4")

En esta imagen, se ve como funciona htmlproofer. Saca un montón de errores de links que no tienen referencia. Esto se debe porque corrí el programa dentro de la ruta _site/jekyll/update/2020/03/05 en lugar de _site y, de esta manera, no detecta las direcciones del href.

Sin embargo, si lo ejecutamos sobre _site:

![Captura5](screenshots/captura5.png?raw=true "Captura 5")

Sale todo correcto.

Podemos automatizar esta prueba con un Rakefile de la siguiente manera:

```rakefile
require 'html-proofer'

task :test do
  sh "bundle exec jekyll build"
  options = { :assume_extension => true }
  HTMLProofer.check_directory("./_site", options).run
end
```

Así, solo tenemos que ejecutar rake test.


