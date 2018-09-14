# Redux

### Porque Redux?

Las paginas y aplicaciones web construidas con JavaScript van haciendose cada vez mas complejas. Esto hace que la manipulacion de nuestro **state** sea mas complicada que nunca. Este **state** incluye respuestas del servidor, data en cache, e informacion generada localmente que todavia no se ha mandado al servidor. El **UI State** se hace mas complejo tambien, ya que tenemos que manejar rutas activas, paginacion, tabs seleccionadas, etc.

Con una pagina mas profunda se ven casos de un componente cambiando otro, una view cambiando un componente, lo cual cambia otro componente diferente, y asi sucesivamente. Llega un punto en el cual ya no entendemos el donde, cuando, ni el porque de nuestro state, lo cual hace muy dificil el proceso de debugguear nuestra aplicacion.

Para hacerlo aun peor, los requisitos del desarrollo front-end se van complicando mas cada dia. Tenemos que optimizar nuestra aplicacion, implementar server-side rendering, y obtener data antes de hacer transiciones de ruta.

Muchas librerias de desarrollo front-end, como React, tratan de resolver el problema de mutacion y asincronicidad al quitar elementos como manipulacion directa del DOM de la ecuacion. Lo unico que queda es manipular la data de la aplicacion. Es aqui donde entra **Redux**.

### Conceptos Basicos

Para entender como Redux maneja nuestro state tenemos que pensar sobre el como un simple objeto. Si el state contuviera clases de la universidad el objeto se veria algo asi: 

```javascript
{
    clases: [{
    nombre: 'Teoria de Bases de Datos',
    aprobada: false
    },{
    nombre: 'Experiencia de Usuario',
    aprobada: false
    }],
    visibilidad: 'MOSTRAR_APROBADAS'
}
```

Este objeto es como un modelo o una clase, solo que sin los setters. Es asi para que diferentes partes del codigo puedan manipular directamente el state, causando bugs dificiles de solucionar.

Para que algo cambie en el state tenemos que despachar una accion. Una accion es un objeto simple de JavaScript que describe lo que cambia en el state. Algunos ejemplos de acciones: 

```javascript
{type: 'ADD_CLASE', text: 'Teoria de la Computacion'}
{type: 'TOGGLE_APB', index: 1}
{type: 'TOGGLE_VISIBILIDAD', filter: 'MOSTRAR_TODAS'}
```

Forzando que cada cambio sea una accion nos permite tener un claro entendimiento de que esta pasando con el state de la app. Si algo cambio, sabemos porque cambio. Las acciones son como migajas de pan de todo lo que ha cambiado.

Finalmente, escribimos una funcion **reducer** para conectar las acciones con el state. Esta funcion recibe un state y una accion como parametros, y devuelve el nuevo state de la aplicacion. Para aplicaciones complejas seria muy dificil escribir una reducer function, asi que la dividimos en funciones mas pequenas.

```javascript
function visibilidad(state = 'MOSTRAR_APROBADAS', accion) {
    if(accion.type === 'TOGGLE_VISIBILIDAD') {
    	return accion.filter
    } else {
    	return state
    }
}

function clases(state = [], accion) {
	switch(accion.type) {
    	case 'ADD_CLASE':
    	    return state.concat([{clase: accion.text, aprobada: false}])
        case: 'TOGGLE_APB':
            return state.map(
            	(clase, index) =>
                    accion.index === index
                        ? {nombre: accion.text, aprobada: !clase.aprobada}
                        : clase
            )
        default:
         return state
    }
}
```

Despues escribimos otra funcion que maneja el state completo de nuestra app al llamar a ambas funciones para cada variable en el state:

```javascript
function appClases(state = {}, accion) {
    return {
        clases: clases(state.clases, accion),
        visibilidad: visibilidad(state.visibilidad, accion)
    }
}
```

Esta es la idea principal detras de Redux: describir como va cambiando el state con acciones, y la mayoria del codigo es en simple JavaScript.

### Tres Principios de Redux

#### Unica fuente de verdad

El state de toda la aplicacion es almacenada en un objeto dentro de una **store**.

Esto hace crear aplicaciones universales un proceso facil, ya que el state del servidor se puede cargar en el cliente sin mucho esfuerzo. Un unico objeto de state nos permite debuggear e inspeccionar la aplicacion mucho mas facilmente.

Un store esta estructurado asi: 

```javascript
type Store = {
    dispatch: Dispatch //Es la funcion principal que recibe acciones
    getState () => State //Retorna el state actual
    subscribe: (listener: () => void) => () => void //Registra una funcion para ejecutarse cada vez que se cambie el state
    replaceReducer: (reducer: Reducer) => void //Usada para hot reloading y code splitting. Poco usada
}
```

#### El state es read-only

La unica manera de cambiar el state es emitiendo una **accion**, nosotros solo escribimos **reducers**

Los reducers son funciones que toman un state y una accion, y devuelven el state nuevo. Es importante recordar que tenemos que retornar un state nuevo en lugar de mutar el state recibido. Podemos empezar con un solo reducer e ir dividiendolo mientras nuestra aplicacion crece. Ya que los reducers son solo funciones, podemos controlar el orden en el cual se ejecutan, pasarles data adicional, o incluso reusar reducers para tareas como paginacion.

```javascript
function visibilidad(state = 'MOSTRAR_APROBADAS', accion) {
    if(accion.type === 'TOGGLE_VISIBILIDAD') {
        return accion.filter
    } else {
    	return state
    }
}

function clases(state = [], accion) {
	switch(accion.type) {
    	case 'ADD_CLASE':
    	    return state.concat([{clase: accion.text, aprobada: false}])
        case: 'TOGGLE_APB':
            return state.map(
            	(clase, index) =>
                    accion.index === index
                    	? {nombre: accion.text, aprobada: !clase.aprobada}
                        : clase
            )
        default:
         	return state
    }
}

import { combineReducers, createStore } from 'redux'
const reducer = combineReducers({ visibilidad, clases })
const store = createStore(reducer)
```
