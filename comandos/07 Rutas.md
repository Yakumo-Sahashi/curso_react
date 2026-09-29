# Rutas React

## 1. ¿Qué es y para qué sirve?

React construye **aplicaciones de una sola página (SPA)**: el navegador carga un único HTML y React cambia el contenido sin recargar. **react-router-dom** es la librería estándar para tener **múltiples "páginas" (rutas)** dentro de esa SPA: sincroniza la URL con los componentes que se muestran.

- `/` → muestra el componente `Inicio`
- `/acerca` → muestra `Acerca`
- `/productos/42` → muestra `Producto` con el id 42

---

## 2. Instalación

```bash
npm i react-router-dom
```

---

## 3. Configuración básica

Envuelve tu aplicación con `BrowserRouter`
Archivo `App.jsx`

```jsx
import { BrowserRouter,Routes,Route } from "react-router-dom";
import Inicio from "./Inicio";
import Acerca from "./Acerca";
import Error from "./Error";

const App = () => {
    return (
        <BrowserRouter>
            <Routes>
            <Route path="/" element={<Inicio />} />
            <Route path="/acerca" element={<Acerca/>} />
            <Route path="*" element={<Error/>} />
            </Routes>
        </BrowserRouter>
    );
}

export default App;
```

React Router elige la ruta que mejor coincide con la URL actual y renderiza su `element`. El path `*` captura todo lo demás (página 404).

---

## 4. Navegación

### `Link` y `NavLink`

**Nunca uses `<a href>`** para navegar dentro de la app: recarga toda la página. Usa `Link`:

```jsx
import { Link, NavLink } from "react-router-dom";

<Link to="/acerca">Ir a Acerca</Link>

<NavLink
  to="/productos"
  className={({ isActive }) => (isActive ? "activo" : "")}
>
  Productos
</NavLink>
```

`NavLink` es igual que `Link`, pero sabe si su ruta está activa; ideal para menús.

### Navegación programática: `useNavigate`

Para redirigir desde código (por ejemplo, después de enviar un formulario):

```jsx
import { useNavigate } from "react-router-dom";

const Login = () => {
  const navigate = useNavigate();

  const handleSubmit = () => {
    // ...validar credenciales
    navigate("/dashboard");            // ir a otra ruta
    // navigate(-1);                   // volver atrás
    // navigate("/", { replace: true }); // sin dejar rastro en el historial
  }

  return <button onClick={handleSubmit}>Entrar</button>;
}
```

### Redirección declarativa: `Navigate`

```jsx
import { Navigate } from "react-router-dom";

<Route path="/antigua" element={<Navigate to="/nueva" replace />} />
```

---

## 5. Rutas dinámicas y parámetros

Un segmento que empieza con `:` es un **parámetro**. Se lee con `useParams`:

```jsx
<Route path="/productos/:id" element={<Producto />} />
```

```jsx
import { useParams } from "react-router-dom";

const Producto = () => {
  const { id } = useParams(); // siempre es un string
  return <h2>Producto #{id}</h2>;
}
```

Enlaces hacia ella: `<Link to="/productos/42">Ver producto 42</Link>`.

### Query strings: `useSearchParams`

Para URLs como `/productos?categoria=libros&pagina=2`:

```jsx
import { useSearchParams } from "react-router-dom";

const Productos() => {
  const [params, setParams] = useSearchParams();
  const categoria = params.get("categoria");

  return (
    <button onClick={() => setParams({ categoria: "libros" })}>
      Filtrar libros (actual: {categoria ?? "todas"})
    </button>
  );
}
```

---

## 6. Rutas anidadas y `Outlet`

Sirven para **layouts compartidos** (menú, cabecera, pie) con contenido interno cambiante.

```jsx
<Routes>
  <Route path="/" element={<Layout />}>
    <Route index element={<Inicio />} />          {/* "/" */}
    <Route path="acerca" element={<Acerca />} />  {/* "/acerca" */}
    <Route path="productos" element={<Productos />} />
    <Route path="productos/:id" element={<Producto />} />
    <Route path="*" element={<Error/>} />
  </Route>
</Routes>
```

```jsx
import { Outlet, NavLink } from "react-router-dom";

const Layout = () => {
  return (
    <>
      <nav>
        <NavLink to="/">Inicio</NavLink>
        <NavLink to="/acerca">Acerca</NavLink>
        <NavLink to="/productos">Productos</NavLink>
      </nav>

      <main>
        <Outlet /> {/* aquí se renderiza la ruta hija que coincida */}
      </main>

      <footer>© 2026</footer>
    </>
  );
}
```

- `index` marca la ruta hija que se muestra en la URL del padre exacta.
- Los paths hijos son **relativos** al padre (no llevan `/` inicial).

---

## 7. Rutas protegidas

Un patrón común: solo dejar pasar a usuarios autenticados.

```jsx
import { Navigate, Outlet } from "react-router-dom";

const RutaProtegida = ({ autenticado }) => {
  return autenticado ? <Outlet /> : <Navigate to="/login" replace />;
}
```

```jsx
<Routes>
  <Route path="/login" element={<Login />} />

  <Route element={<RutaProtegida autenticado={usuario !== null} />}>
    <Route path="/dashboard" element={<Dashboard />} />
    <Route path="/perfil" element={<Perfil />} />
  </Route>
</Routes>
```

Una ruta sin `path` (solo con `element`) actúa como **ruta de layout**: agrupa hijas sin afectar la URL.
