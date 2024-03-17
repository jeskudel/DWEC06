# DWEC UT06: Conceptos avanzados de React.

## El hook useRef

El hook `useRef` crea una referencia mutable que persiste a través de los renderizados de componentes. A diferencia de `useState`, que gestiona el estado y desencadenante un renderizado del componente, `useRef` se utiliza principalmente para acceder y manipular el DOM o para almacenar valores mutables que no desencadenen re-renderizados. 

La sintaxis seria algo asi: 
```jsx
const variabel = useRef(initialValue) 
```

* `initialValue`: El valor que quieres que tenga inicialmente la propiedad current del objeto ref. Puede ser un valor de cualquier tipo. Este argumento se ignora después del renderizado inicial.

Cuando utilizamos `useRef` nos devuelve un objeto con una sola propiedad.

* `current`: Inicialmente, se establece en el `initialValue` que has pasado. Más tarde puedes establecerlo a otra cosa. Si pasas el objeto ref a React como un atributo `ref` a un nodo **JSX**, React establecerá su propiedad `current`.

```jsx
import { useRef } from 'react';

function MyComponent() {
  const intervalRef = useRef(0);
  const inputRef = useRef();
  // ...
  <input ref={inputRef} type='text'/>
  // ...
```

> #### *Tener en cuenta que ...*
> * Se puede mutar la propiedad `current` pero si contiene un objeto que se utiliza para el renderizado (por ejemplo, una parte de tu estado), entonces no deberías mutar ese objeto.
> * Cuando cambias la propiedad `ref.current`, React no vuelve a renderizar tu componente. React no está al tanto de cuándo la cambias porque una ref es un objeto JavaScript plano.

Imaginemos que se necesita almacenar un `id` de intervalo y recuperarlo más tarde, puedes ponerlo en una `ref`. Para actualizar el valor dentro de la `ref`, es necesario cambiar manualmente supropiedad `current`. Con esto ya podemos crear un cronometro para empezar a contar el tiempo y pararlo cuando queramos.

```jsx
import { useRef } from 'react';

function Stopwatch() {
  const intervalRef = useRef(0);
  // ...
  
function handleStartClick() {
  const intervalId = setInterval(() => {
    // ...
  }, 1000);
  intervalRef.current = intervalId;
}

function handleStopClick() {
  const intervalId = intervalRef.current;
  clearInterval(intervalId);
}
```

Este sería el código completo.

```jsx
import { useState, useRef } from 'react';

export default function Stopwatch() {
  const [startTime, setStartTime] = useState(null);
  const [now, setNow] = useState(null);
  const intervalRef = useRef(null);

  function handleStart() {
    setStartTime(Date.now());
    setNow(Date.now());

    clearInterval(intervalRef.current);
    intervalRef.current = setInterval(() => {
      setNow(Date.now());
    }, 10);
  }

  function handleStop() {
    clearInterval(intervalRef.current);
  }

  let secondsPassed = 0;
  if (startTime != null && now != null) {
    secondsPassed = (now - startTime) / 1000;
  }

  return (
    <>
      <h1>Time passed: {secondsPassed.toFixed(3)}</h1>
      <button onClick={handleStart}>
        Start
      </button>
      <button onClick={handleStop}>
        Stop
      </button>
    </>
  );
}
```

> #### *Tener en cuenta que ...*
> No escribas ni leas `ref.current` durante el renderizado. React espera que el cuerpo de tu componente se comporte como una función pura.
> ```jsx
> function MyComponent() {
> // 🚩 No escribas una ref durante el renderizado
> myRef.current = 123;
> // 🚩 No leas una ref durante el renderizado
> return <h1>{myOtherRef.current}</h1>;
> ```
> Puedes, en su lugar, leer o escribir refs desde controladores de eventos o efectos.
> ```jsx
> function MyComponent() {
> useEffect(() => {
>   // ✅ Se pueden leer o escribir refs en efectos
>   myRef.current = 123;
> });
> //...
> function handleClick() {
>   // ✅ Puedes leer o escribir refs en los controladores de eventos
>   doSomething(myOtherRef.current);
> }
> ```

## Manipulando el DOM usando useRef

Es particularmente común utilizar una `ref` para manipular el DOM. React tiene soporte incorporado para esto. En primer lugar, declara una objeto `ref` con un valor inicial de `null`.

```jsx
import { useRef } from 'react';

function MyComponent() {
  const inputRef = useRef(null);
  // ...
```

Luego pasa tu objeto `ref` como el atributo `ref` al JSX del nodo DOM que quieres manipular.

```jsx
// ...
return <input ref={inputRef} />;
```

Después de que React cree el nodo DOM y lo ponga en la pantalla, React establecerá la propiedad current de tu objeto `ref` a ese nodo DOM. Ahora puedes acceder al nodo DOM de `<input>` y llamar a métodos como `focus()`.

```jsx
  function handleClick() {
    inputRef.current.focus();
  }
```

Veamos un ejemplo completo para seleccionar y enfocar un `input` de texto cada vez que pulsemos un botón.

```jsx
import { useRef } from 'react';

export default function Form() {
  const inputRef = useRef(null);

  function handleClick() {
    inputRef.current.focus();
  }

  return (
    <>
      <input ref={inputRef} />
      <button onClick={handleClick}>
        Focus the input
      </button>
    </>
  );
}
```