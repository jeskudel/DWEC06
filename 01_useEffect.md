# DWEC UT06: Conceptos avanzados de React.

## El hook useEffect

Algunos componentes tienen la necesidad de sincronizarse con sistemas externos. Por ejemplo, es posible que desees controlar un componente que no sea de React en función a un estado de React, configurar una conexión de servidor, o enviar un registro de análisis cuando un componente se muestra en la pantalla. Los Efectos te permiten ejecutar código después del renderizado para que puedas sincronizar tu componente con un sistema fuera de React.

Podriamos resumir el hook de `useEffect` en que *nos permite ejecutar codigo arbitrario cada vez que se monta/renderiza el componente y cada vez que las dependencias que hemos definido cambien*.

Veamos un ejemplo partiendo de la aplicación que ya teníamos.

```jsx
import { useEffect } from 'react';

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

### Escribiendo un Efecto

Para escribir un Efecto, sigue los siguientes pasos:

1. **Declara** un Efecto. Por defecto, tu Efecto se ejecutará después de cada renderizado.

```jsx
import { useEffect } from 'react';

function MyComponent() {
  useEffect(() => {
    // El código aquí se ejecutará después de *cada* renderizado
  });
  return <div />;
}
```
2. Define las **dependencias** del Efecto. La mayoría de los Efectos solo deben volver a ejecutarse cuando sea necesario en lugar de hacerlo después de cada renderizado. 
Por ejemplo, una animación de desvanecimiento solo debe desencadenarse cuando aparece el componente. La conexión y desconexión a una sala de chat solo debe suceder cuando el componente aparece y desaparece, o cuando cambia la sala de chat. Aprenderás cómo controlar esto especificando las dependencias.
3. Añade **limpieza** si es necesario. Algunos Efectos necesitan especificar cómo detener, deshacer, o limpiar cualquier cosa que estaban haciendo. Por ejemplo, «conectar» necesita «desconectar», «suscribirse» necesita «anular suscripción» y «buscar» necesita «cancelar» o «ignorar». Aprenderás cómo hacer esto devolviendo una función de limpieza

```jsx
import { useEffect } from 'react';

const App = () => {
  const [ counter, setCounter ] = useState(0)

  useEffect(()=>{
    console.log('El código a ejecutar irá aquí')
  })

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

En este caso al no indicarle ninguna dependencia (el **parametro de dependencia es opcional**), este efecto se ejecutara cada vez que se renderiza el componente, teniendo como resultado que cada vez que pulsamos algún botón para cambiar el estado de `counter` producimos un re-renderizado y se ejecutara tambien el efecto.

Si le indicamos una dependencia en formato de una `array` vacio, este efecto solo se ejecutara la primera vez que se renderice el componente.

```jsx
useEffect(()=>{
  console.log('El código a ejecutar irá aquí')
}, [])
```

Imaginemos que solo queremos ejecutar ese efecto cuando el numero de `counter` supere el valor `10` o `-10` que vamos a marcar como límite. Para ello vamos a crear una variable que controle si excedemos ese limite por alguno de los extremos.

```jsx
import { useEffect } from 'react';

const App = () => {
  const [ counter, setCounter ] = useState(0)
  let limiteExcedido = counter > 10 || counter < -10 ? true : false

  useEffect(()=>{
    console.log('El código a ejecutar irá aquí')
  }, [limiteExcedido])

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

### Añadiendo limpieza a un efecto

Estás escribiendo un componente `ChatRoom` que necesita conectarse al servidor del chat cuando aparece. Se te da una API `createConnection()` que devuelve un objeto con los métodos `connect()` y `disconnect()`. ¿Cómo mantienes conectado el componente mientras este se muestra al usuario?
Comenzamos por escribir la lógica del efecto.

```jsx
useEffect(() => {
  const connection = createConnection();
  connection.connect();
}, []);
```
Sería lento conectarse al chat después de cada nuevo renderizado, así que añades el array de dependencias. El código dentro del Efecto no usa ninguna `prop` o estado, por lo que el array de dependencias está vacío `[]`. Esto le indica a React que solo ejecute este código cuando se «monta» el componente, es decir, aparece en pantalla por primera vez.

Este es el fichero que usaremos como módulo (`chat.js`).

```jsx
export function createConnection() {
  // Una implementación real se conectaría al servidor
  return {
    connect() {
      console.log('✅ Conectando...');
    },
    disconnect() {
      console.log('❌ Desconectado.');
    }
  };
}
```

Y este será el componente que renderizaremos.

```jsx
import { useEffect } from 'react';
import { createConnection } from './chat.js';

export default function ChatRoom() {
  useEffect(() => {
    const connection = createConnection();
    connection.connect();
  }, []);
  return <h1>¡Bienvenido al chat!</h1>;
}
```

Este efecto solo se ejecuta cuando se monta el componente, entonces podrías pensar que "✅ Conectando..." se imprime una vez en la consola. Sin embargo, si revisas la consola, "✅ Conectando..." se imprime dos veces. ¿Por qué sucede esto?

Imagina que el componente `ChatRoom` es parte de una gran aplicación con muchas pantallas diferentes. El usuario inicia su viaje en la página `ChatRoom`. El componente se monta y llama a `connection.connect()`. Entonces imagina que el usuario navega hacia otra pantalla, por ejemplo, a la página de Configuración. El componente `ChatRoom` se desmonta. Finalmente, el usuario hace clic en el botón de atrás y `ChatRoom` se monta nuevamente. Esto configuraría una segunda conexión ¡Pero la primera conexión nunca fue destruida! A medida que el usuario navega por la aplicación, las conexiones seguirían acumulándose.

Errores como este son fáciles de pasarlos por alto sin una extensa prueba manual. Para ayudarte a detectarlos rápidamente, en desarrollo, React vuelve a montar cada componente una vez inmediatamente después de su montaje inicial.

Ver en consola dos veces "✅ Conectando..." te ayuda a notar el problema real: tu código no cierra la conexión cuando el componente se desmonta.

Para solucionar este problema, devuelve una **función de limpieza (cleanup function)** desde el Efecto.

```jsx
  useEffect(() => {
    const connection = createConnection();
    connection.connect();
    return () => {
      connection.disconnect();
    };
  }, []);
```

React llamará a la función de limpieza antes que se ejecute el Efecto nuevamente (si tenemos alguna dependencia marcada), y una última vez cuando el componente se desmonta. Veamos qué sucede cuando se implementa la función de limpieza.

Este es el comportamiento correcto en modo de desarrollo. Al volver a montar el componente, React verifica que navegar a otro lado y luego volver, no romperá tu código. ¡Desconectar y luego conectar nuevamente es exactamente lo que debería suceder! Cuando implementas la limpieza adecuadamente, no debe haber ninguna diferencia visible para el usuario entre ejecutar el Efecto una vez o ejecutarlo, limpiarlo y volver a ejecutarlo. Hay llamadas adicionales a connect/disconnect porque React está explorando tu código en busca de errores en desarrollo. Esto es normal, ¡No intentes hacerlo desaparecer!.

En producción, solo verás "✅ Conectando..." una vez. Volver a montar componentes solo sucede en desarrollo para ayudarte a encontrar Efectos que necesitan limpieza. Puedes desactivar el **Modo Estricto** para optar por el comportamiento de producción, pero recomendamos dejarlo activado. Esto te permite encontrar muchos errores como el anterior.

### Efectos Secundarios 

Os dejo un video donde los chicos de "**Desarrollo Útil**" explicamen con varios ejemplos muy sencillo cuales son los efectos secundarios (sideEffects) que podemos tener cuando trabajamos con useEffect y eventos.

* Cuando se ejecuta la función de "cleanup" cuando tenemos dependencias.
* Cuando tenemos suscripciones a eventos con `addEventListener`.
* Cuando tenemos intervalos programados.

<p align="center"> 
<a href="https://www.youtube.com/watch?v=9GaCyok0DEU">
<img src="./img/video_side_effects.jpg" width="60%" height="60%" style="display: block; margin: 0 auto" />
</a>
</p>

### Otro ejemplo

Tambien os dejo un video del youtuber "**Midudev**" que en esta parte del video crea otro ejemplo (mas visual y diferente) utilizando useEffect. Además, nos habla de la inportancía de utilizar las funciones de `cleanup` cuando desmontamos un componente.

<p align="center"> 
<a href="https://www.youtube.com/watch?v=qkzcjwnueLA&t=4937s">
<img src="./img/video_useEffect_midu.jpg" width="60%" height="60%" style="display: block; margin: 0 auto" />
</a>
</p>

> #### *Tener en cuenta que ...*
> Ademas, recordad el consejo (mas bien obligación) que nos da Midu en esta parte del [video](https://www.youtube.com/watch?v=qkzcjwnueLA&t=5400s).
