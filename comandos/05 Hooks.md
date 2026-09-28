# Hooks

## ¿Qué es un hook?

Un **hook** es una función que permite "engancharse" a características internas de React (estado, ciclo de vida, contexto) desde un componente de función. Todos empiezan con `use` por convención, y esa convención la aplica el linter para verificar las reglas.

---

## ¿Por qué existen?

Antes de React 16.8, solo las clases podían tener estado. Esto producía tres problemas:

1. **Lógica difícil de reutilizar.** Compartir comportamiento entre componentes exigía patrones complejos (HOCs, render props) que anidaban el árbol hasta hacerlo ilegible.
2. **Componentes gigantes.** Un mismo método de ciclo de vida (`componentDidMount`) mezclaba suscripciones, peticiones y temporizadores sin relación entre sí.
3. **`this` confuso.** Los estudiantes perdían más tiempo con `this.handleClick = this.handleClick.bind(this)` que con la lógica real.

Los hooks agrupan el código por **preocupación** en vez de por momento del ciclo de vida.

### Las dos reglas, y por qué existen

**Regla 1: llama a los hooks solo en el nivel superior.** Nunca dentro de condicionales, ciclos, funciones anidadas o después de un `return` temprano.

**Regla 2: llámalos solo desde componentes de función o desde otros hooks.** Nunca desde funciones JavaScript comunes.

La razón de la Regla 1 es el mecanismo interno: React **no conoce los nombres de tus variables de estado**. Asocia cada llamada a un hook con una posición en una lista interna del componente, y depende de que el orden de las llamadas sea idéntico en cada render.

```jsx
//Error: ROMPE EL ORDEN
const Componente = ({ mostrar }) => {
  if (mostrar) {
    const [a, setA] = useState(0);   // a veces se llama, a veces no
  }
  const [b, setB] = useState('');    // ¡cambia de posición!
};

//CORRECTO: el condicional va adentro
const Componente = ({ mostrar }) => {
  const [a, setA] = useState(0);
  const [b, setB] = useState('');
  if (!mostrar) return null;         // el return temprano va DESPUÉS de los hooks
};
```

---

## Hook: `useState`

Una variable local no sirve como estado:

```jsx
const Contador = () => {
  let cuenta = 0;                      // se reinicia en cada render
  return <button onClick={() => cuenta++}>{cuenta}</button>;
  // El clic incrementa la variable, pero React nunca se entera → no re-renderiza.
};
```

`useState` hace dos cosas que la variable local no puede: **persiste el valor entre renders** y **avisa a React que debe re-renderizar**.

### Sintaxis

```jsx
const [valor, setValor] = useState(valorInicial);
```

Esto es desestructuración de arreglo. `useState` retorna un arreglo de dos posiciones; los nombres los eliges tú.

**Convención:** `[algo, setAlgo]`.

```jsx
import { useState } from 'react';

const Contador = () => {
  const [cuenta, setCuenta] = useState(0);

  return (
    <div>
      <p>Valor: {cuenta}</p>
      <button onClick={() => setCuenta(cuenta + 1)}>+1</button>
      <button onClick={() => setCuenta(0)}>Reiniciar</button>
    </div>
  );
};
```

---

### Cuatro cosas que hay que entender

**1. El estado es inmutable: nunca lo modifiques directamente.**

```jsx
const [usuario, setUsuario] = useState({ nombre: 'Ana', edad: 22 });

usuario.edad = 23;                          //no re-renderiza
setUsuario({ ...usuario, edad: 23 });       // re-renderiza
const [tareas, setTareas] = useState([]);

tareas.push(nueva);                         // no añade
setTareas([...tareas, nueva]);              // si añade
```

React compara la **referencia** anterior con la nueva (`Object.is`). Si mutas, la referencia es la misma y React concluye que nada cambió.

**2. Las actualizaciones son asíncronas y se agrupan.**

```jsx
const manejarClic = () => {
  setCuenta(cuenta + 1);
  console.log(cuenta);   // imprime el valor VIEJO
};
```

`setCuenta` no modifica la variable `cuenta`; programa un nuevo render. Dentro del render actual, `cuenta` sigue siendo la constante que se capturó al ejecutar la función.

**3. La forma funcional del actualizador.**

```jsx
// Error: Los tres usan el MISMO valor de `cuenta` → incrementa solo 1
setCuenta(cuenta + 1);
setCuenta(cuenta + 1);
setCuenta(cuenta + 1);

// Correcto: Cada uno recibe el valor más reciente → incrementa 3
setCuenta(prev => prev + 1);
setCuenta(prev => prev + 1);
setCuenta(prev => prev + 1);
```

**4. Inicialización perezosa.**

```jsx
const [datos, setDatos] = useState(calcularCostoso());    // se ejecuta en CADA render
const [datos, setDatos] = useState(() => calcularCostoso()); // solo en el primero
```

Si pasas una **función**, React la ejecuta una sola vez. Si pasas el **resultado**, la expresión se evalúa en cada render aunque React descarte el valor.

---

### Cuántos estados usar

```jsx
// Opción A: estados separados — preferible cuando cambian independientemente
const [nombre, setNombre] = useState('');
const [email, setEmail] = useState('');

// Opción B: un objeto — preferible cuando cambian juntos (formularios)
const [formulario, setFormulario] = useState({ nombre: '', email: '' });

const manejarCambio = ({ target: { name, value } }) => {
  setFormulario(prev => ({ ...prev, [name]: value }));
};
```

La opción B usa **claves computadas** y permite un solo manejador para todo el formulario, siempre que cada `<input>` tenga su atributo `name`.

---

### Estado derivado

Error muy común:

```jsx
// Error: Estado redundante que hay que mantener sincronizado
const [tareas, setTareas] = useState([]);
const [totalPendientes, setTotalPendientes] = useState(0);

// Correcto: Calcularlo durante el render
const [tareas, setTareas] = useState([]);
const totalPendientes = tareas.filter(t => !t.completada).length;
```

> **Regla: si un valor puede calcularse a partir de otro estado o de las props, no debe ser estado.**

---

### Inputs controlados

```jsx
const [texto, setTexto] = useState('');

<input
  type="text"
  value={texto}                                  // el estado manda
  onChange={(e) => setTexto(e.target.value)}     // el input avisa
/>
```

El valor que se ve en pantalla **siempre** proviene del estado de React. Si pones `value` sin `onChange`, el campo queda de solo lectura y React lo advierte.

### Ejemplo

`src/components/Contador/Contador.js`:

```jsx
import { useState } from 'react';

const Contador = ({ inicial = 0, paso = 1 }) => {
  const [cuenta, setCuenta] = useState(inicial);
  const [historial, setHistorial] = useState([]);

  const registrar = (accion, nuevoValor) =>
    setHistorial(prev => [...prev, `${accion} → ${nuevoValor}`].slice(-5));

  const incrementar = () => {
    setCuenta(prev => {
      const nuevo = prev + paso;
      registrar('+', nuevo);
      return nuevo;
    });
  };

  const decrementar = () => setCuenta(prev => Math.max(0, prev - paso));
  const reiniciar   = () => { setCuenta(inicial); setHistorial([]); };

  // DEMO: la diferencia entre forma directa y funcional
  const tripleMal  = () => { setCuenta(cuenta + 1); setCuenta(cuenta + 1); setCuenta(cuenta + 1); };
  const tripleBien = () => { setCuenta(p => p + 1); setCuenta(p => p + 1); setCuenta(p => p + 1); };

  return (
    <div className="contador">
      <h2>{cuenta}</h2>
      <button onClick={decrementar}>−{paso}</button>
      <button onClick={incrementar}>+{paso}</button>
      <button onClick={reiniciar}>Reiniciar</button>

      <hr />
      <button onClick={tripleMal}>+3 (mal: suma 1)</button>
      <button onClick={tripleBien}>+3 (bien: suma 3)</button>

      {historial.length > 0 && (
        <ul>{historial.map((h, i) => <li key={`${h}-${i}`}>{h}</li>)}</ul>
      )}
    </div>
  );
};

export default Contador;
```

`src/components/FormularioRegistro/FormularioRegistro.js`:

```jsx
import { useState } from 'react';

const ESTADO_INICIAL = { nombre: '', email: '', edad: '', acepta: false, plan: 'basico' };

const FormularioRegistro = ({ onRegistrar }) => {
  const [formulario, setFormulario] = useState(ESTADO_INICIAL);
  const [enviado, setEnviado] = useState(false);

  // UN solo manejador para todos los campos, usando clave computada
  const manejarCambio = ({ target: { name, value, type, checked } }) => {
    setFormulario(prev => ({
      ...prev,
      [name]: type === 'checkbox' ? checked : value
    }));
  };

  // Validación DERIVADA, no almacenada en estado
  const errores = {
    nombre: formulario.nombre.trim().length < 3 ? 'Mínimo 3 caracteres' : null,
    email: !formulario.email.includes('@') ? 'Correo inválido' : null,
    edad: Number(formulario.edad) < 18 ? 'Debes ser mayor de edad' : null,
    acepta: !formulario.acepta ? 'Debes aceptar los términos' : null
  };

  const esValido = Object.values(errores).every(e => e === null);

  const manejarEnvio = (e) => {
    e.preventDefault();          // evita la recarga del navegador
    if (!esValido) return;
    onRegistrar(formulario);
    setEnviado(true);
    setFormulario(ESTADO_INICIAL);
  };

  return (
    <form onSubmit={manejarEnvio} noValidate>
      <label>
        Nombre
        <input name="nombre" value={formulario.nombre} onChange={manejarCambio} />
        {formulario.nombre && errores.nombre && <small>{errores.nombre}</small>}
      </label>

      <label>
        Email
        <input name="email" type="email" value={formulario.email} onChange={manejarCambio} />
        {formulario.email && errores.email && <small>{errores.email}</small>}
      </label>

      <label>
        Edad
        <input name="edad" type="number" value={formulario.edad} onChange={manejarCambio} />
        {formulario.edad && errores.edad && <small>{errores.edad}</small>}
      </label>

      <label>
        Plan
        <select name="plan" value={formulario.plan} onChange={manejarCambio}>
          <option value="basico">Básico</option>
          <option value="pro">Pro</option>
        </select>
      </label>

      <label>
        <input name="acepta" type="checkbox" checked={formulario.acepta} onChange={manejarCambio} />
        Acepto los términos
      </label>

      <button type="submit" disabled={!esValido}>Registrar</button>

      {enviado && <p className="exito">Registro completado</p>}

      <pre>{JSON.stringify(formulario, null, 2)}</pre>
    </form>
  );
};

export default FormularioRegistro;
```

El `<pre>` al final es un recurso didáctico excelente: el grupo ve el estado cambiar letra por letra mientras escribe.

---

## Hook: `useEffect`

### Qué es un efecto

Un componente de React debe ser **puro durante el render**: calcular JSX a partir de props y estado, sin tocar nada externo. Pero las aplicaciones reales necesitan hacer cosas fuera de React: peticiones de red, temporizadores, suscripciones, modificar el `document.title`, leer el tamaño de la ventana.

Eso son **efectos secundarios**, y `useEffect` es donde viven. React los ejecuta **después** de que el render se pintó en pantalla.

### Sintaxis y el arreglo de dependencias

```jsx
useEffect(() => {
  // código del efecto

  return () => {
    // función de limpieza (opcional)
  };
}, [dependencias]);
```

El segundo argumento determina **cuándo** se re-ejecuta:

| Forma | Cuándo se ejecuta |
|---|---|
| `useEffect(fn)` | Después de **cada** render. Casi siempre es un error |
| `useEffect(fn, [])` | Solo una vez, al montar |
| `useEffect(fn, [a, b])` | Al montar y cada vez que `a` o `b` cambien |

React compara cada dependencia con la del render anterior usando `Object.is` (comparación de referencia). Por eso un objeto o arreglo literal en las dependencias dispara el efecto en cada render: `{}` !== `{}`.

---

### La función de limpieza

Se ejecuta **antes** de cada re-ejecución del efecto y **al desmontar** el componente. Su propósito es deshacer lo que el efecto creó.

```jsx
useEffect(() => {
  const id = setInterval(() => console.log('tick'), 1000);
  return () => clearInterval(id);     // sin esto, el intervalo sigue tras desmontar
}, []);
```

**Todo lo que se suscribe debe desuscribirse:** `setInterval`/`clearInterval`, `addEventListener`/`removeEventListener`, `fetch`/`AbortController`, conexiones WebSocket.

Olvidar la limpieza produce **fugas de memoria** y el error clásico "Can't perform a React state update on an unmounted component".

---

### StrictMode y la doble ejecución

En desarrollo con `React.StrictMode`, React monta, desmonta y vuelve a montar cada componente. Un efecto con `[]` se ejecutará **dos veces**.

Esto no es un bug: es un detector. Si tu efecto tiene limpieza correcta, la doble ejecución es inofensiva. Si no la tiene, verás el síntoma (dos peticiones, dos intervalos) y sabrás que hay que arreglarlo.

Conviene demostrarlo: agregar un `console.log` en el efecto y contar las apariciones en consola.

---

### Errores más frecuentes

**1. Ciclo infinito por actualizar estado sin dependencias controladas.**

```jsx
//set → render → efecto → set → render → ...
useEffect(() => {
  setContador(contador + 1);
});
```

**2. Objeto literal en las dependencias.**

```jsx
//Error: `opciones` es un objeto nuevo en cada render
const opciones = { limite: 10 };
useEffect(() => { buscar(opciones); }, [opciones]);

//Correcto: depender de los valores primitivos
useEffect(() => { buscar({ limite }); }, [limite]);
```

**3. Omitir dependencias para "evitar" re-ejecuciones.** Esto produce *stale closures*: el efecto usa valores congelados de un render anterior. La solución no es mentirle al arreglo, sino reestructurar (forma funcional del setter, `useCallback`, o mover la función dentro del efecto). El linter de CRA advierte de esto.

**4. Usar `useEffect` para estado derivado.**

```jsx
//Error: render extra innecesario
const [total, setTotal] = useState(0);
useEffect(() => { setTotal(items.reduce((a,i) => a+i.precio, 0)); }, [items]);

//Correcto
const total = items.reduce((a, i) => a + i.precio, 0);
```

> **Regla: si puedes calcularlo durante el render, no uses un efecto.**

---

### Ejemplos

`src/components/DemoEfectos/DemoEfectos.js`:

**1. Efecto sin limpieza: sincronizar el título**
```jsx
import { useState, useEffect } from 'react';

const TituloDinamico = () => {
  const [cuenta, setCuenta] = useState(0);

  useEffect(() => {
    document.title = cuenta > 0 ? `(${cuenta}) Notificaciones` : 'Mi App';
  }, [cuenta]);

  return <button onClick={() => setCuenta(c => c + 1)}>Notificar ({cuenta})</button>;
};
```
**2. Efecto con limpieza: temporizador**
```jsx
const Cronometro = () => {
  const [segundos, setSegundos] = useState(0);
  const [corriendo, setCorriendo] = useState(false);

  useEffect(() => {
    if (!corriendo) return;                      // no crear el intervalo si está pausado

    console.log('▶ intervalo creado');
    const id = setInterval(() => {
      setSegundos(prev => prev + 1);             // forma funcional: evita stale closure
    }, 1000);

    return () => {
      console.log('intervalo limpiado');
      clearInterval(id);
    };
  }, [corriendo]);

  const formato = `${String(Math.floor(segundos / 60)).padStart(2, '0')}:${String(segundos % 60).padStart(2, '0')}`;

  return (
    <div>
      <h2>{formato}</h2>
      <button onClick={() => setCorriendo(c => !c)}>{corriendo ? 'Pausar' : 'Iniciar'}</button>
      <button onClick={() => { setCorriendo(false); setSegundos(0); }}>Reiniciar</button>
    </div>
  );
};
```
**3. Suscripción a evento del navegador**
```jsx
const TamanioVentana = () => {
  const [tamanio, setTamanio] = useState({ ancho: window.innerWidth, alto: window.innerHeight });

  useEffect(() => {
    const manejarResize = () =>
      setTamanio({ ancho: window.innerWidth, alto: window.innerHeight });

    window.addEventListener('resize', manejarResize);
    return () => window.removeEventListener('resize', manejarResize);
  }, []);

  const dispositivo = tamanio.ancho < 640 ? 'Móvil'
                    : tamanio.ancho < 1024 ? 'Tablet'
                    : 'Escritorio';

  return <p>{tamanio.ancho} × {tamanio.alto} — {dispositivo}</p>;
};
```
**4. Demostración de la fuga de memoria**
```jsx
const DemoEfectos = () => {
  const [mostrarCrono, setMostrarCrono] = useState(true);

  return (
    <div>
      <TituloDinamico />
      <TamanioVentana />
      <button onClick={() => setMostrarCrono(m => !m)}>
        {mostrarCrono ? 'Desmontar' : 'Montar'} cronómetro
      </button>
      {mostrarCrono && <Cronometro />}
    </div>
  );
};

export default DemoEfectos;
```

---

## Hook: `useFetch`

### Los tres estados de toda petición

Cualquier consumo de API tiene exactamente tres estados posibles, y **los tres deben representarse en la interfaz**:

1. **Cargando** — la petición está en curso.
2. **Éxito** — hay datos.
3. **Error** — algo falló.

El error más común de los estudiantes es manejar solo el caso feliz, lo que produce interfaces que parpadean o se quedan en blanco sin explicación.

### El patrón sin hook (para entender el problema)

```jsx
const ListaUsuarios = () => {
  const [usuarios, setUsuarios] = useState([]);
  const [cargando, setCargando] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    const controlador = new AbortController();

    const obtener = async () => {
      try {
        setCargando(true);
        setError(null);
        const res = await fetch('https://jsonplaceholder.typicode.com/users', {
          signal: controlador.signal
        });
        if (!res.ok) throw new Error(`HTTP ${res.status}`);
        setUsuarios(await res.json());
      } catch (err) {
        if (err.name !== 'AbortError') setError(err.message);
      } finally {
        setCargando(false);
      }
    };

    obtener();
    return () => controlador.abort();
  }, []);

  if (cargando) return <p>Cargando…</p>;
  if (error) return <p>Error: {error}</p>;
  return <ul>{usuarios.map(u => <li key={u.id}>{u.name}</li>)}</ul>;
};
```

Son 25 líneas de código idéntico que habría que repetir en cada componente que consuma una API. Eso justifica extraer un custom hook.

**Tres puntos técnicos a subrayar:**

1. **`if (!res.ok) throw`** — recordar de la Sesión 1 que `fetch` no rechaza ante 404 o 500.
2. **`AbortController`** — si el usuario navega a otra vista antes de que la respuesta llegue, el componente se desmonta y actualizar su estado provoca una advertencia. Cancelar la petición en la limpieza resuelve el problema, y además es lo que hace que StrictMode sea inofensivo.
3. **`finally`** — `setCargando(false)` debe ocurrir pase lo que pase.

---

### Por qué la función async va *dentro* del efecto

```jsx
//Error: useEffect no acepta una función async: retornaría una promesa,
//    y React espera que el retorno sea la función de limpieza.
useEffect(async () => { ... }, []);

//Correcto
useEffect(() => {
  const obtener = async () => { ... };
  obtener();
}, []);
```

---

### Ejemplo

`src/hooks/useFetch.js`:

```jsx
import { useState, useEffect, useCallback } from 'react';

const ESTADO_INICIAL = { datos: null, cargando: true, error: null };

export const useFetch = (url, opciones = {}) => {
  const [estado, setEstado] = useState(ESTADO_INICIAL);
  const [recarga, setRecarga] = useState(0);

  // Serializamos las opciones para poder usarlas como dependencia primitiva.
  // Si pasáramos el objeto directamente, cambiaría de referencia en cada render.
  const opcionesSerializadas = JSON.stringify(opciones);

  useEffect(() => {
    if (!url) {
      setEstado({ datos: null, cargando: false, error: null });
      return;
    }

    const controlador = new AbortController();

    const obtener = async () => {
      setEstado(prev => ({ ...prev, cargando: true, error: null }));

      try {
        const respuesta = await fetch(url, {
          ...JSON.parse(opcionesSerializadas),
          signal: controlador.signal
        });

        if (!respuesta.ok) {
          throw new Error(`Error ${respuesta.status}: ${respuesta.statusText}`);
        }

        const datos = await respuesta.json();
        setEstado({ datos, cargando: false, error: null });

      } catch (err) {
        if (err.name === 'AbortError') return;   // cancelación esperada, no es error
        setEstado({ datos: null, cargando: false, error: err.message });
      }
    };

    obtener();

    return () => controlador.abort();
  }, [url, opcionesSerializadas, recarga]);

  const refrescar = useCallback(() => setRecarga(n => n + 1), []);

  return { ...estado, refrescar };
};
```


`src/components/Usuarios/Usuarios.js`:

```jsx
import { useState } from 'react';
import { useFetch } from '../../hooks/useFetch';

const BASE = 'https://jsonplaceholder.typicode.com';

const Usuarios = () => {
  const [idSeleccionado, setIdSeleccionado] = useState(null);

  const { datos: usuarios, cargando, error, refrescar } = useFetch(`${BASE}/users`);

  // Petición dependiente: la URL es null hasta que haya selección
  const { datos: posts, cargando: cargandoPosts } = useFetch(
    idSeleccionado ? `${BASE}/posts?userId=${idSeleccionado}` : null
  );

  if (cargando) return <p className="skeleton">Cargando usuarios…</p>;

  if (error) {
    return (
      <div className="error">
        <p>{error}</p>
        <button onClick={refrescar}>Reintentar</button>
      </div>
    );
  }

  return (
    <div className="layout">
      <aside>
        <h2>Usuarios ({usuarios.length})</h2>
        <button onClick={refrescar}>↻ Refrescar</button>
        <ul>
          {usuarios.map(({ id, name, email }) => (
            <li key={id}>
              <button
                onClick={() => setIdSeleccionado(id)}
                className={id === idSeleccionado ? 'activo' : ''}
              >
                {name}
                <small>{email}</small>
              </button>
            </li>
          ))}
        </ul>
      </aside>

      <main>
        {!idSeleccionado && <p>Selecciona un usuario.</p>}
        {cargandoPosts && idSeleccionado && <p>Cargando publicaciones…</p>}
        {posts && (
          <>
            <h3>{posts.length} publicaciones</h3>
            <ul>{posts.map(p => <li key={p.id}>{p.title}</li>)}</ul>
          </>
        )}
      </main>
    </div>
  );
};

export default Usuarios;
```

---

## Custom Hook: useCounter

### ¿Qué es y qué no es?

Un **custom hook** es simplemente una función que:
- empieza con `use`,
- llama a otros hooks en su interior,
- retorna lo que su consumidor necesite.

No es una característica especial de React: es una convención que permite **extraer lógica con estado** para reutilizarla.

**Lo que se comparte es la lógica, no el estado.** Si dos componentes usan `useCounter()`, cada uno tiene su propio contador independiente. Esta es la confusión más frecuente y conviene demostrarla en vivo con dos instancias en pantalla.

### Cuándo extraer un custom hook

- La misma lógica con estado aparece en dos o más componentes.
- Un componente tiene tanta lógica que el JSX se pierde entre el ruido.
- Quieres poder probar la lógica sin renderizar interfaz.

### Anatomía

```jsx
// src/hooks/useCounter.js
import { useState } from 'react';

export const useCounter = (valorInicial = 0, { paso = 1, min = -Infinity, max = Infinity } = {}) => {
  const [contador, setContador] = useState(valorInicial);

  const incrementar = (cantidad = paso) =>
    setContador(prev => Math.min(max, prev + cantidad));

  const decrementar = (cantidad = paso) =>
    setContador(prev => Math.max(min, prev - cantidad));

  const reiniciar = () => setContador(valorInicial);

  const establecer = (valor) =>
    setContador(Math.min(max, Math.max(min, valor)));

  // Valores derivados que el consumidor agradecerá
  const enMinimo = contador <= min;
  const enMaximo = contador >= max;

  return { contador, incrementar, decrementar, reiniciar, establecer, enMinimo, enMaximo };
};
```

### ¿Retornar objeto o arreglo?

| | Arreglo `[a, b]` | Objeto `{ a, b }` |
|---|---|---|
| Ventaja | El consumidor renombra libremente | Los nombres son explícitos; el orden no importa |
| Cuándo | 2 valores, como `useState` | 3 o más valores |

Para `useCounter`, que retorna seis cosas, el objeto es la elección correcta.

## Ejemplo

`src/components/DemoContadores/DemoContadores.js`:

```jsx
import { useCounter } from '../../hooks/useCounter';

const Carrito = () => {
  const { contador, incrementar, decrementar, enMinimo, enMaximo } =
    useCounter(1, { min: 1, max: 10 });

  return (
    <div>
      <h3>Cantidad de producto</h3>
      <button onClick={() => decrementar()} disabled={enMinimo}>−</button>
      <span> {contador} </span>
      <button onClick={() => incrementar()} disabled={enMaximo}>+</button>
      {enMaximo && <small>Máximo por cliente alcanzado</small>}
    </div>
  );
};

const Paginador = ({ totalPaginas = 5 }) => {
  const { contador: pagina, incrementar, decrementar, establecer, enMinimo, enMaximo } =
    useCounter(1, { min: 1, max: totalPaginas });

  return (
    <div>
      <h3>Página {pagina} de {totalPaginas}</h3>
      <button onClick={() => decrementar()} disabled={enMinimo}>Anterior</button>
      {Array.from({ length: totalPaginas }, (_, i) => i + 1).map(n => (
        <button key={n} onClick={() => establecer(n)} disabled={n === pagina}>{n}</button>
      ))}
      <button onClick={() => incrementar()} disabled={enMaximo}>Siguiente</button>
    </div>
  );
};

const DemoContadores = () => (
  <>
    <Carrito />
    <Carrito />     {/* ← estados INDEPENDIENTES, misma lógica */}
    <Paginador totalPaginas={7} />
  </>
);

export default DemoContadores;
```

