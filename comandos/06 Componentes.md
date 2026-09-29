# Componentes React

## 1. ¿Qué es un componente?

Un **componente** es una pieza reutilizable e independiente de interfaz de usuario. En React, toda la aplicación se construye combinando componentes, como si fueran bloques de LEGO: un botón, una tarjeta, un formulario, una página completa.

Hoy en día, un componente es simplemente una **función de JavaScript que devuelve JSX** (una sintaxis parecida a HTML).

```jsx
const Saludo = () => {
  return <h1>¡Hola, mundo!</h1>;
}
```

**Reglas básicas:**

- El nombre debe empezar con **mayúscula** (`Saludo`, no `saludo`).
- Debe devolver un único elemento raíz (o un fragmento `<>...</>`).
- Se usa como una etiqueta HTML: `<Saludo />`.

---

## 2. JSX

JSX permite escribir la estructura de la UI dentro de JavaScript. Para insertar valores o expresiones se usan llaves `{}`:

```jsx
const nombre = "Ana";
return <p>Hola, {nombre}. 2 + 2 = {2 + 2}</p>;
```

Diferencias con HTML: se usa `className` en lugar de `class`, y `onClick` (camelCase) en lugar de `onclick`.

---

## 3. Props: pasar datos a un componente

Las **props** (propiedades) son los argumentos que recibe un componente desde su padre. Son **de solo lectura**: el componente hijo no debe modificarlas.

```jsx
const Saludo = ({ nombre, edad }) => {
  return <p>Hola, {nombre}. Tienes {edad} años.</p>;
}

// Uso:
<Saludo nombre="Ana" edad={28} />
```

También existe la prop especial `children`, que representa lo que se coloca entre las etiquetas de apertura y cierre:

```jsx
const Caja = ({ children }) => {
  return <div className="caja">{children}</div>;
}

<Caja>
  <p>Contenido dentro de la caja</p>
</Caja>
```

---

## 4. State: datos que cambian

El **estado** es la memoria interna de un componente. Cuando cambia, React **vuelve a renderizar** el componente para reflejar el nuevo valor. Se maneja con el hook `useState`:

```jsx
import { useState } from "react";

const [contador, setContador] = useState(0);
//      valor       función      valor inicial
//      actual      para cambiarlo
```

**Importante:** nunca modifiques el estado directamente (`contador = 5`). Siempre usa la función `setContador`.

| Props | State |
|---|---|
| Vienen de afuera (del padre) | Pertenecen al propio componente |
| Solo lectura | Se pueden modificar con su setter |
| Cambiarlas re-renderiza al hijo | Cambiarlo re-renderiza al componente |

---

## 5. Ciclo de renderizado

1. **Render:** React ejecuta tu función componente y obtiene el JSX resultante.
2. **Comparación:** React compara el resultado con el anterior (Virtual DOM).
3. **Actualización:** solo modifica en el DOM real las partes que cambiaron.

Un componente se vuelve a renderizar cuando cambia su **estado**, cambian sus **props**, o se re-renderiza su **padre**.

---

## 6. Buenas prácticas

- **Componentes pequeños** con una sola responsabilidad.
- **Un componente por archivo**, con el mismo nombre que el componente.
- **Nunca mutar** estado ni props; crear copias nuevas (`...spread`, `map`, `filter`).
- **Usar `key` estables** (ids reales, no el índice si la lista puede reordenarse).
- **Extraer lógica reutilizable** a *custom hooks* (`useMiHook`).
- Solo llamar a los hooks **en el nivel superior** del componente, nunca dentro de condicionales o bucles.

---