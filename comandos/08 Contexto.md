# Contexto en React

## 1. El problema: el "prop drilling"

En React los datos fluyen de padres a hijos mediante props. Pero cuando un dato (el usuario logueado, el tema, el idioma) lo necesita un componente muy profundo en el árbol, hay que pasarlo por **todos los componentes intermedios**, aunque ellos no lo usen:

```jsx
<App usuario={usuario}>
  <Layout usuario={usuario}>
    <Sidebar usuario={usuario}>
      <Avatar usuario={usuario} />   {/* el único que lo necesita */}
    </Sidebar>
  </Layout>
</App>
```

A esto se le llama **prop drilling**: es tedioso, ensucia el código y dificulta el mantenimiento.

---

## 2. La solución: Context

**Context** permite compartir un valor con todo un subárbol de componentes **sin pasarlo manualmente por props**. El hook **`useContext`** es la forma de leer ese valor desde cualquier componente hijo, por profundo que esté.

Se compone de tres piezas:

| Pieza | Qué hace |
|---|---|
| `createContext(valorPorDefecto)` | Crea el contexto |
| `<Contexto.Provider value={...}>` | Envuelve el subárbol y **provee** el valor |
| `useContext(Contexto)` | Lee el valor más cercano del Provider desde un componente |

---

## 3. Uso básico paso a paso

### 1. crear el contexto

Archivo `TemaContext.jsx`
```jsx
import { createContext } from "react";

export const TemaContext = createContext("claro"); // valor por defecto
```

El valor por defecto solo se usa si un componente lee el contexto **sin** tener un Provider por encima.

### 2. proveer el valor

Archivo `App.jsx`
```jsx
import { useState } from "react";
import { TemaContext } from "./TemaContext";

const App = () => {
  const [tema, setTema] = useState("claro");

  return (
    <TemaContext.Provider value={{ tema, setTema }}>
      <Pagina />
    </TemaContext.Provider>
  );
}

export default App;
```

### 3. consumir con `useContext`

```jsx
import { useContext } from "react";
import { TemaContext } from "./TemaContext";

const BotonTema = () => {
    const { tema, setTema } = useContext(TemaContext);

    return (
        <button onClick={() => setTema(tema === "claro" ? "oscuro" : "claro")}>
            Tema actual: {tema}
        </button>
    );
}
```

Ningún componente intermedio recibió props: `BotonTema` obtiene el valor directamente.

---

## 4. Cómo funciona por dentro

1. `useContext` busca hacia arriba en el árbol el **Provider más cercano** de ese contexto.
2. Devuelve el `value` que este tenga.
3. Cuando el `value` cambia, React **vuelve a renderizar todos los componentes** que consumen ese contexto con `useContext`.

Puedes anidar Providers del mismo contexto: los componentes leerán el más cercano, lo que permite "sobrescribir" valores en una sección concreta.

---

## 5. Patrón recomendado: Provider + hook personalizado

Repetir `useContext(TemaContext)` por todos lados es poco elegante y no avisa si olvidas el Provider. La práctica recomendada es encapsular todo en un archivo:

Archivo `TemaContext.jsx`
```jsx
import { createContext, useContext, useState } from "react";

const TemaContext = createContext(null);

const TemaProvider = ({ children }) => {
  const [tema, setTema] = useState("claro");

  const alternar = () => setTema((t) => (t === "claro" ? "oscuro" : "claro"));

  return (
    <TemaContext.Provider value={{ tema, alternar }}>
      {children}
    </TemaContext.Provider>
  );
}

// Hook personalizado con validación
const useTema = () => {
  const contexto = useContext(TemaContext);
  if (contexto === null) {
    throw new Error("useTema debe usarse dentro de un <TemaProvider>");
  }
  return contexto;
}

export {TemaProvider, useTema}
```

**Ventajas:** la lógica queda centralizada, los componentes solo llaman `useTema()`, y obtienes un error claro si falta el Provider.

## 6. Ejemplo completo: tema claro/oscuro

Archivo `index.js`
```jsx
import { createRoot } from "react-dom/client";
import { TemaProvider } from "./TemaContext";
import App from "./App";

createRoot(document.getElementById("root")).render(
  <TemaProvider>
    <App />
  </TemaProvider>
);
```

Archivo `App.jsx`
```jsx
import { useTema } from "./TemaContext";

const BotonTema = () => {
    const { tema, alternar } = useTema();
    return (
        <button onClick={alternar}>
        Cambiar a {tema === "claro" ? "oscuro" : "claro"}
        </button>
    );
}

const Tarjeta = () => {
    const { tema } = useTema();

    const estilos = {
        padding: 20,
        borderRadius: 8,
        background: tema === "claro" ? "#ffffff" : "#222222",
        color: tema === "claro" ? "#111111" : "#f5f5f5",
        border: "1px solid #888",
    };

    return (
        <div style={estilos}>
            <h2>Soy una tarjeta</h2>
            <p>Mi aspecto depende del contexto, no de props.</p>
        </div>
    );
}

// Componente intermedio: NO recibe ni pasa nada relacionado con el tema
const Contenido = () => {
    return (
        <section>
        <Tarjeta />
        </section>
    );
}

export default const App = () => {
  return (
    <main>
      <BotonTema />
      <Contenido />
    </main>
  );
}
```

`Contenido` no sabe nada del tema, pero `Tarjeta` (su hija) lo usa sin problema.

---

## 7. Ejemplo típico: autenticación

Archivo `AuthContext.jsx`
```jsx
import { createContext, useContext, useState } from "react";

const AuthContext = createContext(null);

export const AuthProvider ? ({ children }) => {
  const [usuario, setUsuario] = useState(null);

  const login = (nombre) => setUsuario({ nombre });
  const logout = () => setUsuario(null);

  return (
    <AuthContext.Provider value={{ usuario, login, logout }}>
      {children}
    </AuthContext.Provider>
  );
}

export const useAuth = () => {
  const ctx = useContext(AuthContext);
  if (!ctx) throw new Error("useAuth debe usarse dentro de <AuthProvider>");
  return ctx;
}
```

```jsx
const Cabecera = () => {
  const { usuario, login, logout } = useAuth();

  return usuario ? (
    <p>
      Hola, {usuario.nombre}
      <button onClick={logout}>Salir</button>
    </p>
  ) : (
    <button onClick={() => login("Ana")}>Iniciar sesión</button>
  );
}
```