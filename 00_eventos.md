# DWEC UT06: Conceptos avanzados de React.

## Manejo de eventos

Ya hemos mencionado los **controladores de eventos** con anterioridad, que están registrados para ser llamados cuando ocurren eventos específicos. Por ejemplo, la interacción de un usuario con los diferentes elementos de una página web puede provocar que se active una colección de diferentes tipos de eventos.

Los elementos de botón admiten los llamados eventos de mouse, de los cuales `click` es el evento más común. Simularemos un componente sencillo que contien un botón con el que interactuaremos.

```jsx
export default function Button() {
  return (
    <button>
      No hago nada
    </button>
  );
}
```

Para agregar un controlador de evento, primero definirás una función y luego la pasarás como una `prop` a la etiqueta JSX apropiada.

```jsx
export default function Button() {
  function handleClick() {
    alert('¡Me hiciste clic!');
  }

  return (
    <button onClick={handleClick}>
      Hazme clic
    </button>
  );
}
```

Las funciones controladoras de evento:

* Usualmente están definidas dentro de tus componentes.
* Tienen nombres que empiezan con `handle`, seguido del nombre del evento.

Por convención, es común llamar a los controladores de eventos como handle seguido del nombre del evento. A menudo verás `onClick={handleClick}`, `onMouseEnter={handleMouseEnter}`, etcétera.

Si la función no es muy compleja tambien se puede pasar directamente y simplificarla con una arrow function.

```jsx
<button onClick={() => {
  alert('¡Me hiciste clic!');
}}>
```

> #### *Tener en cuenta que ...*
> Las funciones que se pasan a los controladores de eventos deben ser pasadas, no llamadas.
>```jsx
> // Esta alerta se ejecuta cuando el componente se renderiza, no cuando ¡hiciste clic!
> <button onClick={alert('¡Me hiciste clic!')}>
> ```
> Si quieres definir un controlador de evento en línea, envuélvelo en una función anónima de esta forma:
> ```jsx
> <button onClick={() => alert('¡Me hiciste clic!')}>
> ```

### Leyendo las `props` en el controlador de eventos

Como los controladores de eventos son declarados dentro de un componente, tienen acceso a las `props` del componente. Este es  un botón que, cuando se hace clic, muestra una alerta con su prop `message`:

```jsx
function AlertButton({ message, children }) {
  return (
    <button onClick={() => alert(message)}>
      {children}
    </button>
  );
}

export default function Toolbar() {
  return (
    <div>
      <AlertButton message="¡Reproduciendo!">
        Reproducir película
      </AlertButton>
      <AlertButton message="¡Subiendo!">
        Subir imagen
      </AlertButton>
    </div>
  );
}
```

Esto le permite a estos dos botones mostrar diferentes mensajes. Intenta cambiar los mensajes que se les pasan.

### Nombrar `props` de controladores de eventos 

Componentes integrados como `<button>` y `<div>` solo admiten nombres de eventos del navegador como `onClick`. Sin embargo, cuando estás creando tus propios componentes, puedes nombrar sus props de controlador de evento como quieras.

Por convención, las `props` de controladores de eventos deberían empezar con `on`, seguido de una letra mayúscula.

Por ejemplo, la propiedad `onClick` del componente Button pudo haberse llamado `onSmash`.

```jsx
function Button({ onSmash, children }) {
  return (
    <button onClick={onSmash}>
      {children}
    </button>
  );
}

export default function App() {
  return (
    <div>
      <Button onSmash={() => alert('¡Reproduciendo!')}>
        Reproducir película
      </Button>
      <Button onSmash={() => alert('¡Subiendo!')}>
        Subir imagen
      </Button>
    </div>
  );
}
```

### Propagación de eventos

Los controladores de eventos también atraparán eventos de cualquier componente hijo que tu componente pueda tener. Decimos que un evento «se expande» o «se propaga» hacia arriba en el árbol de componentes cuando: empieza donde el evento sucedió, y luego sube en el árbol.

Este `<div>` contiene dos botones. Tanto el `<div>` como cada botón tienen sus propios controladores `onClick`. ¿Qué controlador crees que se activará cuando hagas clic en un botón?

```jsx 
export default function Toolbar() {
  return (
    <div className="Toolbar" onClick={() => {
      alert('¡Hiciste clic en la barra de herramientas!');
    }}>
      <button onClick={() => alert('¡Reproduciendo!')}>
        Reproducir película
      </button>
      <button onClick={() => alert('¡Subiendo!')}>
        Subir imagen
      </button>
    </div>
  );
}
```

Si haces clic en cualquiera de los botones, su onClick se ejecutará primero, seguido por el `onClick` del `<div>`. Así que dos mensajes aparecerán. Si haces clic en el propio toolbar, solo el `onClick` del `<div>` padre se ejecutará.

> Para detener la propagación de un evento, no tenemos mas que llamar al método del evento `stopPropagation` que tiene la variable `e` o `event`. Se hace de la misma manera que lo haciamos en JS puro ([enlace](https://developer.mozilla.org/en-US/docs/Web/API/Event/stopPropagation)).

### Evitar comportamiento por defecto

Algunos eventos del navegador tienen comportamientos por defecto asociados a ellos. Por ejemplo, un evento submit de un `<form>`, que ocurre cuando se hace clic en un botón que está dentro de él, por defecto recargará la página completa. Puedes llamar `e.preventDefault()` en el objeto del evento para evitar que esto suceda:

```jsx
export default function Signup() {
  return (
    <form onSubmit={e => {
      e.preventDefault();
      alert('¡Enviando!');
    }}>
      <input />
      <button>Enviar</button>
    </form>
  );
}
```

## Modificando estados con eventos

Imaginemos que tenemos un componente muy sencillo que hace de contador. Queremos que cada vez que pulsemos el boton, este contador se incremente. Empezaremos por pasar una función simple para comprobar que podemos hacer `click` y nos loguea algo en la consola. Esto ya lo hemos visto antes.

```jsx
const App = () => {
  const [ counter, setCounter ] = useState(0)

  const handleClick = () => {    
    console.log('clicked')  
    }
  return (
    <div>
      <div>{counter}</div>
      <button onClick={handleClick}>
      plus
      </button>
    </div>
  )
}
```

Establecemos el valor del atributo `onClick` del botón para que sea una referencia a la función `handleClick` definida en el código.

Ahora, cada **clic** del botón plus hace que se llame a la función `handleClick`, lo que significa que cada evento de clic registrará un mensaje de clicked en la consola del navegador.

Ahora queremos utilizar la función  `setCounter` del objeto `useState()`. Lo haremos usando una arrow function ya que no es una función definida por nosotros, sino que queremos hacer uso de ella. Veamos el resultado.

```jsx
<button onClick={() => setCounter(counter + 1)}>
  plus
</button>
```

Además, agregaremos otro botón que nos resetee la cuenta que lleva el contador a `0`. Para ello solo tenemos que llamar a `setCounter` con el valor de reseteo.

```jsx
<button onClick={() => setCounter(0)}>
  plus
</button>
```

Por lo general, definir controladores de eventos dentro de las plantillas JSX no es una buena idea. Aquí está bien, porque nuestros controladores de eventos son muy simples. Vamos a separar a los controladores de eventos en funciones separadas de todas formas.

```jsx
const App = () => {
  const [ counter, setCounter ] = useState(0)

  const increaseByOne = () => setCounter(counter + 1)  
  const setToZero = () => setCounter(0)
  return (
    <div>
      <div>{counter}</div>
      <button onClick={increaseByOne}>        
      Incrementar
      </button>
      <button onClick={setToZero}>        
      Resetear
      </button>
    </div>
  )
}
```

## Cambios de estado y re-renderizado

Cuando se inicia la aplicación, se ejecuta el código en `App`. Este código usa un hook `useState` para crear el estado de la aplicación, estableciendo un valor inicial de la variable `counter`. Este componente contiene el componente `Display`, que muestra el valor del contador, `0`, y tres componentes `Button`. Todos los botones tienen controladores de eventos, que se utilizan para cambiar el estado del contador.

```jsx
const App = () => {
  const [ counter, setCounter ] = useState(0)

  const increaseByOne = () => setCounter(counter + 1)
  const decreaseByOne = () => setCounter(counter - 1)  
  const setToZero = () => setCounter(0)

  return (
    <div>
      <Display counter={counter}/>
      <Button onClick={increaseByOne} text='plus'/>
      <Button onClick={setToZero} text='zero'/>
      <Button onClick={decreaseByOne} text='minus'/>
    </div>
  )
}
```

Cuando se hace **clic** en uno de los botones, se ejecuta el controlador de eventos. El controlador de eventos cambia el estado del componente `App` con la función `setCounter`. Llamar a una función que cambia el estado hace que el componente se vuelva a procesar.

Entonces, si un usuario hace **clic** en el botón `plus`, el controlador de eventos del botón cambia el valor de `counter` a `1`, y el componente `App` se vuelve a generar. Esto hace que sus subcomponentes `Display` y `Button` también se vuelvan a renderizar. `Display` recibe el nuevo valor del contador, `1`, como `prop`. Los componentes `Button` reciben controladores de eventos que pueden usarse para cambiar el estado del contador.

Para asegurarnos de entender como funciona el programa, vamos a agregarle algunos `console.log`

```jsx
const App = () => {
  const [ counter, setCounter ] = useState(0)
  console.log('rendering with counter value', counter)

  const increaseByOne = () => {
    console.log('increasing, value before', counter)    
    setCounter(counter + 1)
  }
  const decreaseByOne = () => { 
    console.log('decreasing, value before', counter)    
    setCounter(counter - 1)
  }  
  const setToZero = () => {
    console.log('resetting to zero, value before', counter)    
    setCounter(0)
  }

  return (
    <div>
      <Display counter={counter}/>
      <Button onClick={increaseByOne} text='plus'/>
      <Button onClick={setToZero} text='zero'/>
      <Button onClick={decreaseByOne} text='minus'/>
    </div>
  )
}
```
> Para pobrar este ejemplo debereis crear los componentes `Button` y `Display`. Tambien, podeis sustituirlos por elementos HTML básicos.