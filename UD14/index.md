# UD 14 - Otros elementos a estudiar



- [UD 14 - Otros elementos a estudiar](#ud-14-otros-elementos-a-estudiar)
   * [1. Computed](#1-computed)
   * [2. Watchers](#2-watchers)
   * [3. Acceder al DOM: 'ref'](#3-acceder-al-dom-ref)
      + [3.1 nextTick](#31-nexttick)
   * [4. Clases HTML](#4-clases-html)
      + [4.1 Sintaxis de objeto](#41-sintaxis-de-objeto)
      + [4.2 Sintaxis de array](#42-sintaxis-de-array)
      + [4.3 Asignar clases a un componente](#43-asignar-clases-a-un-componente)
      + [4.4 Asignar estilos directamente](#44-asignar-estilos-directamente)
   * [5. Ciclo de vida del componente](#5-ciclo-de-vida-del-componente)
      + [5.1 El ciclo de vida de un componente](#51-el-ciclo-de-vida-de-un-componente)
   * [6. Componentes asíncronos](#6-componentes-asíncronos)
   * [7. Custom Directives](#7-custom-directives)
   * [8. Imágenes](#8-imágenes)
   * [9. Transiciones](#9-transiciones)
   * [10. Entornos](#10-entornos)
   * [11. Guards del router](#11-guards-del-router)
- [Bibliografía](#bibliografía)



## 1. Computed
Cuando se crea un componente de Vue (o el componente raíz) se le pasa como parámetro un objeto con las opciones con que se creará. Entre ellas tenemos _props_, _ data_, _methods_, y también otras como _computed_ y _watch_.

Hemos visto que en una interpolación o directiva podemos poner una expresión javascript. Pero si la expresión es demasiado compleja hace que nuestro HTML sea más difícil de leer. La solución es crear una expresión calculada que nos permite tener "limpio" el HTML. Por ejemplo un código con expresiones complejas como:

```vue
<template>
  <p>Autor: { { author.name + ' ' + author.surname }}</p>
  <p>Ha publicado libros: { { author.books.length > 0 ? 'Sí' : 'No' }}</p>
</template>

<script>
export default {
  name: 'author-item',
  data() {
    return {
      author: {
        name: 'John',
        surname: 'Doe',
        books: [
          'Vue 2 - Advanced Guide',
          'Vue 3 - Basic Guide',
          'Vue 4 - The Mystery'
        ]
      }
    }
  }
}
</script>
```

se puede simplificar creando propiedades calculadas:

```vue
<template>
  <p>Autor: { { fullName }}</p>
  <p>Ha publicado libros: { { hasPublished }}</p>
</template>

<script>
export default {
  name: 'author-item',
  data() {
    return {
      author: {
        name: 'John',
        surname: 'Doe',
        books: [
          'Vue 2 - Advanced Guide',
          'Vue 3 - Basic Guide',
          'Vue 4 - The Mystery'
        ]
      }
    }
  },
  computed: {
    fullName() {
      return this.name + ' ' + this.surname;
    },
    hasPublished() {
      return this.author.books.length > 0 ? 'Sí' : 'No'
    }
  }
})
</script>
```

En lugar de definir _computed_ podríamos haber obtenido el mismo resultado usando métodos, pero la ventaja de las propiedades calculadas es que se cachean por lo que si se vuelven a tener que renderizar en el DOM no vuelven a evaluarse, a menos que cambie el valor de alguna de las variables reactivas que use.

| Haz el ejercicio del tutorial de [Vue.js](https://vuejs.org/tutorial/#step-4)

Por defecto las propiedades _computed_ sólo hacen un _getter_, por lo que no se puede cambiar su valor. Pero podemos si queremos hacerlo definir métodos _getter_ y _setter_:

```javascript
  computed: {
    fullName:
      // getter
      get() {
        return this.name + ' ' + this.surname;
      },
      // setter
      set(newValue) {
        const names = newValue.split(' ');
        this.name = names[0];
        this.surname = names[names.length - 1];
      }
    },
  },
})
```

Si hacemos `this.fullName = 'John Doe'` estaremos asignando los valores adecuados a las variables _name_ y _surname_.

## 2. Watchers
Vue proporciona una forma genérica de controlar cuándo cambia el valor de una variable reactiva para poder ejecutar código en ese momento poniéndole un _watch_:

```javascript
  data() {
    return {
      name: 'John',
      surname: 'Doe',
      fullName: 'John Doe',
    }
  },
  watch: {
    name(newValue, oldValue) {
      this.fullName = newValue + ' ' + this.surname;
    },
    surname(newValue, oldValue) {
      this.fullName = this.name + ' ' + newValue;
    },
  },
})
```

En este caso no tiene mucho sentido y es más fácil (y más eficiente) usar una propiedad _computed_ como hemos visto antes, pero hay ocasiones en que necesitamos ejecutar código al cambiar una variable y es así donde se usan. Veremos su utilidad cuando trabajemos con _vue-router_.

NOTA: los _watcher_ son costosos por lo que no debemos abusar de ellos

| Haz el ejercicio del tutorial de [Vue.js](https://vuejs.org/tutorial/#step-10)

## 3. Acceder al DOM: 'ref'
Aunque Vue se encarga de la vista por nosotros en alguna ocasión podemos tener que acceder a un elemento del DOM. En ese caso no haremos un `document.getElement...` sino que le ponemos una referencia al elemento con el atributo `ref` para poder acceder al mismo desde nuestro script:
```vue
<template>
  <form ref="myForm">
    ...
  </form>
</template>

<script>
export default {
  mounted() {
    this.$refs.myForm.setAttribute('novalidate', true)
  }
}
</script>
```

Desde el código tenemos acceso a todas las referencias desde `this.$refs`. Hay que tener en cuenta que sólo se puede acceder a un elemento después de montarse el componente (en el _hook_ **mounted()** o después).

| Haz el ejercicio del tutorial de [Vue.js](https://vuejs.org/tutorial/#step-9)

### 3.1 nextTick
Si modificamos una variable reactiva el cambio se refleja automáticamente en el DOM, pero no inmediatamente sino que se espera hasta el evento _nextTick_ en el ciclo de modificación para asegurarse de no cambiar algo que quizá va a volverse a cambiar en este ciclo.

Si accedemos al DOM antes de que se produzca este evento el valor aún será el antiguo. Para obtener el nuevo valor hemos de esperar al _nextTick_:
```vue
<template>
  <p>Contador: <span ref="contador">{ { count }}</span></p>
  <button @click="increment">Incrementa</button>
</template>

<script>
export default {
  data() {
    return {
      count: 0
    }
  },
  methods: {
    increment() {
      count.value++
      console.log('Contador en el DOM: ' + this.$refs.contador.textContent)
      // Devolverá el valor sin actualizar aún
      nextTick(() => {
        console.log('Contador en el DOM tras nextTick: ' + this.$refs.contador.textContent)
        // Devolverá el valor actualizado
      })
    }
  }
}
</script>
```

Realmente es algo que seguramente nunca necesitemos pero así conocemos un poco más cómo funciona Vue internamente.

## 4. Clases HTML
Ya hemos visto que en Javascript usamos las clases con mucha frecuencia, normalmente para asignar a elementos estilos definidos en el CSS, pero también para identificar elementos sin usar una _id_ (como hacíamos poniendo a los botones de acciones de los productos las clases _subir_, _bajar_, _editar_ o _borrar_).

En Vue tenemos diferentes formas de asignar clases. La más simple sería _bindear_ el atributo _class_ y gestionarlas directamente en el código, pero no es lo más cómodo:
```html
<div :class="clasesDelDiv"></div>
```

En este caso tendríamos que asignar a la variables _clasesDelDiv_ las diferentes clases separadas por espacio, lo que es engorroso de mantener.

### 4.1 Sintaxis de objeto
Una forma más sencilla es _bindear_ un objeto donde cada propiedad es el nombre de una posible clase y su valor es un booleano que indica si tendrá o no dicha clase, por ejemplo:
```html
<div 
    class="static"
    :class="{ active: isActive, 'text-danger': hasError }"
></div>
```

En este caso el \<DIV> tendrá las clases:
- static: siemrpe tendrá esta clase. Como véis puede coexistir la directiva _:class_ con el atributo _class_ y se suman ambos
- active: tendrá esta clase si el valor de la variable _isActive_ es _true_
- text-danger: ídem para la variable _hasError_. Si el nombre de una clase tiene más de una palabra hay que entrecomillarla

Para mejorar la legibilidad del HTML podemos poner el objeto de las clases en el Javascript
```html
<div 
    class="static"
    :class="classObject"
></div>
```

```javascript
data() {
  return {
    classObject: {
      active: true,
      'text-danger': false
    }
  }
}
```

### 4.2 Sintaxis de array
Podemos indicar las clases en forma de array de variables que contienen la clase a asignar:
```html
<div :class="[activeClass, errorClass]"></div>
```

```javascript
data() {
  return {
    activeClass: 'active',
    errorClass: 'text-danger'
  }
}
```

En este caso el \<DIV> tendrá las clases `active` y `text-danger`. 

Y es posible incluir sintaxis de objeto dentro de la sintaxis de array:
```html
<div :class="[{ active: isActive}, errorClass]"></div>
```

### 4.3 Asignar clases a un componente
En la etiqueta de un componente podemos ponerle un atributo _class_ que le asignará las clases incluidas y que se sumaran a las que se le asignen dentro del propio componente. Por ejemplo, si el \<DIV> del ejemplo anterior es el _template_ de un componente llamado MyComponent puedo poner:
```html
<my-component class="main highligth"></my-component>
```

En este caso el \<DIV> tendrá las clases `main`, `highligth`, `active` si la variable _isActive_ vale _true_ y `text-danger`. 

En Vue3 el _template_ de un componente puede tener varios elementos raíz. En ese caso para indicar a cuál se aplicarán las clases definidas en el padre se usa la propiedad `$attr.class`:
```html
<template>
    <p :class="$attrs.class">Hi!</p>
    <span>This is a child component</span>
</template>
```

### 4.4 Asignar estilos directamente
Aunque no es lo recomendable, podemos asignar directamente estilos CSS igual que asignamos clases y también podemos usar la sintaxis de objeto o la de array.
```html
<div :style="{ color: activeColor, fontSize: fontSize + 'px' }"></div>
```

```javascript
data() {
  return {
    activeColor: 'red',
    fontSize: 30'
  }
}
```

## 5. Ciclo de vida del componente
### 5.1 El ciclo de vida de un componente
Al crearse la instancia de Vue o un componente la aplicación debe realizar unas tareas como configurar la observación de variables, compilar su plantilla (_template_), montarla en el DOM o reaccionar ante cambios en las variables volviendo a renderizar las partes del DOM que han cambiado. Además ejecuta funciones definidas por el usuario cuando sucede alguno de estos eventos, llamadas _hooks_ del ciclo de vida.

En la siguiente imagen podéis ver el ciclo de vida de la instancia Vue (y de cualquier componente) y los eventos que se generan y que podemos interceptar:

![Ciclo de vida de Vue](https://vuejs.org/assets/lifecycle.DLmSwRQE.png)

**IMPORTANTE**: no debemos definir estas funciones como _arrow functions_ porque en estas funciones se enlaza en la variable _this_ el componente donde se definen y si hacemos una _arrow function_ no tendríamos _this_:

```javascript
// MAL, NO HACER ASÍ
created: () => {
    console.log('instancia creada'); 
}
```

```javascript
// BIEN, HACER ASÍ
created() {
    console.log('instancia creada'); 
}
```

Los principales _hooks_ son:
- **beforeCreate**: aún no se ha creado el componente (sí la instancia de Vue) por lo que no tenemos acceso a sus variables, etc
- **created**: se usa por ejemplo para realizar peticiones a servicios externos lo antes posible
- **beforeMount**: ya se ha generado el componente y compilado su _template_
- **mounted**: ahora ya tenemos acceso a todas las propiedades del componete. Es el sitio donde hacer una patición externa si el valor devuelto queremos asignarlo a una variable del componente
- **beforeUpdate**: se ha modificado el componente pero aún no se han renderizado los cambios
- **updated**: los cambios ya se han renderizado en la página
- **beforeUnmount**: antes de que se destruya el componente (en versiones anteriores a Vue3 **beforeDestroy**)
- **unmounted**: ya se ha destruido el componente (en versiones anteriores a Vue3 **destroyed**)

| Haz el ejercicio del tutorial de [Vue.js](https://vuejs.org/tutorial/#step-9)

## 6. Componentes asíncronos
En proyectos grandes con centenares de componentes podemos hacer que en cada momento se carguen sólo los componentes necesarios de manera que se ahorra mucho tiempo de carga de la página.

Para que un componente se cargue asíncronamente al registrarlo se hace como un objeto que será una función que importe el componente. Un componente normal (síncrono) se registraría así:
```vue
<script>
import ProductItem from './ProductItem.vue'

export default {
    name: 'products-table',
    components: {
        ProductItem,
    },
...
}
</script>
```

Si queremos que se cargue asíncronamente no lo importamos hasta se registra:
```vue
<script>
export default {
    name: 'products-table',
    components: {
        ProductItem: () => import('./ProductItem.vue'),
    },
...
}
</script>
```

También podemos decirle que espere un tiempo a cargar el componente (delay) e incluso qué componente queremos cargar mientras está cargando el componente o cuál cargar si hay un error al cargarlo:
```vue
<script>
export default {
    name: 'products-table',
    components: {
        ProductItem: () => ({
            component: import('./ProductItem.vue'),
            delay: 500,       // en milisegundos
            timeout: 6000,
            loading: compLoading,   // componente que cargará mientras se está cargando
            error: compError,       // componente que cargará si hay un error,
        })
    },
...
}
</script>
```

## 7. Custom Directives
Podemos crear nuestras propias directivas para usar en los elementos que queramos. Se definen en un fichero .js con `Vue.directive` y le pasamos su nombre y un objeto con los estados en que queremos que reaccione. Por ejemplo vamos a hacer una directiva para que se le asigne el foco al elemento al que se la pongamos, que será de tipo _input_:
```javascript
import Vue from 'vue'

Vue.directive('focus', {
  mounted(el) {
    el.focus();
  }
})
```

Para usarla en un componente la importamos y ya podemos usarla en el _template_:
```vue
<template>
  ...
  <input v-focus type="text" name="nombre">
  ...
</template>

<script>
import focus from './focus.js'
...
}
</script>
```

Si queremos utilizarla en muchos componentes podemos importarla en el _main.js_ y así estará disponible para todos los componentes.

Los estados de la directiva en los que podemos actuar son:
- **mounted**: cuando se inserte la directiva
- **updated**: cuando se actualice el componente que contiene la directiva
- **beforeMount**: cuando se enlaza la directiva al componente por primera vez, antes de montar el componente
- ...

## 8. Imágenes
Lo habitual es guardar las imágenes en la carpeta `assets`. Para que se carguen correctamente usaremos en su atributo `src` la función `require` con la URL de la imagen. Ejemplo:
```html
<td>  
  <img :src="require('../assets/elPatitoFeo.jpeg')" height="100px" alt="El Patito Feo">
</td>
```

## 9. Transiciones
Vue permite controlar transiciones en nuestra aplicación poniendo el código CSS correspondiente y añadiéndole al elemento el atributo _transition_. Podemos encontrar más información en la [documentación oficial de Vue](https://vuejs.org/v2/guide/transitions.html).

## 10. Entornos
En Vue tenemos normalmente 3 entornos o _modos_, el de **development**, el de **test** y el de **production**. Las variables de entorno las guardaremos en uno de los siguientes ficheros:
- **.env**: se cargan en todos los modos
- **.env.local**: se cargan en todos los modos pero son ignordas por git
- **.env.[modo]**: se cargan sólo en todos el modo indicado 
- **.env.[modo].local**: ídem pero son ignordas por git

En contenido de estos ficheros son variables en forma `clave=valor`:
```javascript
// fichero .env
TITULO=Mi proyecto
VUE_APP_API=https://localhost/api
```

Si el nombre de la variable comienza por `VUE_APP_` será accesible desde el código con `process.env.nombreVariable`:
```javascript
// <script> de componente
console.log(process.env.VUE_APP_API);
```

Podemos saber en qué entorno se está ejecutando la aplicación consultando el valor de la variable `process.env.NODE_ENV`.

## 11. Guards del router
Son _hooks_ que podemos controlar en distintos momentos, algunos desde el componente y otros desde el _router_. Podemos ponerlos para todas las rutas, para una ruta en concreto o en el componente.

La mayoría reciben 3 parámetros:
- **to**: ruta a la que se va a saltar
- **from**: ruta de la que se viene
- **next**: función para que continue la carga del router. Siempre tras ejecutar el código que deseemos pondremos `netx()`.

En el router tenemos estos _guards_:
- **router.beforeEach(to, from, next)**: se ejecuta antes de que vaya a cambiarse la ruta
- **router.afterEach(to, from)**: se ejecuta una vez cambiada la ruta (por eso no tiene next, porque ya ha acabado)
- **ruta.beforeEnter(to, from, next)**: se pone como propiedad de una ruta y se ejecuta antes de entrar a ella

Para aplicarlos en nuestro router lo asignamos a una variable que exportamos:
```javascript
let router = new Router({
  routes: [
    {
      path: '/',
      component: 'MyComponent',
      beforeEnter(to, from, next) {
        console.log('Vengo de ' + from + ' y voy a ' + to);
        next();
      },
...
})

router.beforeEach(to, from, next) {
  console.log('Vengo de ' + from + ' y voy a ' + to);
  next();
}

export default router
```

En un componente también puedo definir los _hooks_:
- **beforeRouteEnter(to, from, next)**
- **beforeRouteUpdate(to, from, next)**
- **beforeRouteLeave(to, from, next)**



# Bibliografía

* Curso 'Programación con JavaScript'. CEFIRE Xest. Arturo Bernal Mayordomo
* [Curso de JavaScript y TypeScript](https://www.youtube.com/playlist?list=PLiZCpIzKtvqvt4tcQV4SAvaJn7QMdwUbd) de Arturo Bernal en Youtube
* [MDN Web Docs](https://developer.mozilla.org/es/docs/Web/JavaScript). Moz://a. https://developer.mozilla.org/es/docs/Web/JavaScript
* [Introducción a JavaScript](http://librosweb.es/libro/javascript/). Librosweb. http://librosweb.es/libro/javascript/
* [Curso de Javascript (Desarrollo web en entorno cliente)](https://www.youtube.com/playlist?list=PLI7nHlOIIPOJtTDs1HVJABswW-xJcA7_o). Ada Lovecode - Didacticode (90 vídeos)
* [Apuntes Desarrollo Web en Entorno Cliente (DWEC)](https://github.com/sergarb1/ApuntesDWEC). García Barea, Sergi
* [Apuntes Desarrollo Web en Entorno Cliente (DWEC)](https://github.com/cipfpbatoi/materials). Segura Vasco, Juan. CIPFP Batoi.
