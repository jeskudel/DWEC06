# DWEC UT06: Conceptos avanzados de React.

Necesitas entender cómo son renderizadas las páginas en una aplicación de React antes de zambullirte en el enrutamiento. Cuando pulsas un enlace en el navegador, en aplicaciones con más de una página, es enviada una petición al servidor antes de que la página HTML se muestre. 

En React, el contenido de la página es creado a partir de nuestros componentes. Así que lo que hace `React Router` es interceptar la petición que se envía al servidor y luego inyectar el contenido dinámicamente desde los componentes que hemos creado. Esta es la idea general detrás de las **SPA**, que permiten que el contenido se muestre más rápido sin que la página sea actualizada.

Así que, en una aplicación de página única, cuando se navega a un nuevo componente utilizando React Router, el archivo indice.html será reescrito en función de la lógica del componente. 

## Instalación de React Router

Como toda librería de programación, `React Router` funciona por medio de una serie de archivos que nos facilitan determinada acción. Esta librería en particular nos da una serie de componentes mediante los que podemos hacer la navegación por las distintas URL de nuestra aplicación de modo declarativo.

Todo lo que tienes que hacer para instalar `React Router` es ejecutar el comando de instalación de la librería en la terminal de tu proyecto y esperar a que se complete la instalación. 

```node
npm install react-router-dom@6
//tambien podemos utilizar la última versión
npm install react-router-dom@latest
```

Lo primero que hay que hacer, una vez completada la instalación, es conseguir que `React Router` esté disponible en cualquier parte de tu aplicación.

Para hacer esto, abre el archivo `index.js` de la carpeta **src**  e importa `BrowserRouter` desde `react-router-dom`, y luego, rodea el componente raíz (habitualmente `App`) con él.

Deberíamos pasar de algo asi:

```jsx
import React from 'react';
import ReactDOM from 'react-dom';
import App from './App.jsx';

ReactDOM.render (
  <React.StrictMode>
    <App />
  </React.StrictMode>,
  document.getElementById('root')
);
```

A esto otro:

```jsx
import React from 'react';
import ReactDOM from 'react-dom';
import App from './App.jsx';
import { BrowserRouter } from "react-router-dom";

ReactDOM.render (
  <BrowserRouter>
    <App />
  </BrowserRouter>,
  document.getElementById('root')
);
```

## Redigiendo a otros componentes

Vamos a crear varios componentes que nos serviran de muestra y los vamos a llamar `Inicio`, `Nosotros` y `Contacto`.

Van a ser muy sencillos ya que solo van a tener un `h1` y un texto identificativo del componente. Seguiran el patron de este ejemplo:

```jsx
function Inicio() {
  return (
    <div>
      <h1>Esta es la página de inicio</h1>
    </div>
  );
}

export default Inicio;
```
### Definición de rutas

Una vez tengamos los componentes creados, tendremos que definir las rutas. Dado que el componente `App` actúa como el componente raíz, dónde se renderiza al principio nuestro código React, vamos a crear todas nuestras rutas en él. 

Para ello importaremos las funcionalidades que emplearemos: `Routes` y `Route`. A continuación, importaremos todos los componentes a los que necesitamos proporcionar una ruta.

`Routes` actúa como contenedor padre de todas las rutas individuales que se crearán en nuestra aplicación.

`Route` se utiliza para crear una única ruta.

```jsx
import { Routes, Route } from "react-router-dom"
import Inicio from "./Inicio.jsx"
import Nosotros from "./Nosotros.jsx"
import Contacto from "./Contacto.jsx"

function Aplicacion() {
  return (
    <div className="Aplicacion">
      <Routes>
        <Route path="/" element={ <Inicio /> } />
        <Route path="sobre-nosotros" element={ <SobreNosotros /> } />
        <Route path="contacto" element={ <Contacto /> } />
      </Routes>
    </div>
  )
}

export default Aplicacion
```

`Route` recibe dos atributos:

* `path`, que especifica la ruta URL del componente deseado. Puedes llamar a esta ruta de la manera que quieras. Arriba, habrás notado que el primer nombre de ruta es raíz(`/`). Cualquier componente cuya nombre de ruta sea una barra diagonal se mostrará primero, siempre que la aplicación se cargue por vez primera. Esto implica que el componente `Inicio` será el primero en mostrarse.  
* `element`, que especifica el componente que debe renderizar el enrutador.