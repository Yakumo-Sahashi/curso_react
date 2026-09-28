# ECMAScript

## 1. Variables, constantes y template strings

### El problema de `var`

`var` tiene **ámbito de función** (*function scope*) y sufre *hoisting*: su declaración se eleva al inicio del ámbito con valor `undefined`. Esto produce comportamientos que contradicen la intuición del lector.

```js
function demo() {
  console.log(x); // undefined, no lanza error
  var x = 10;

  if (true) {
    var y = 20;
  }
  console.log(y); // 20 — "escapó" del bloque if
}
```

El caso clásico de examen:

```javascript
for (var i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 100);
}
// Imprime: 3, 3, 3
```

Existe **una sola** variable `i` compartida por las tres funciones. Cuando los temporizadores ejecutan, el ciclo ya terminó e `i` vale 3.

---

### Ámbito de bloque: `let` y `const`

Ambas tienen **ámbito de bloque** (`{ }`) y viven en la *Temporal Dead Zone* (TDZ): existen desde el inicio del bloque pero no son accesibles hasta su línea de declaración, por lo que acceder antes lanza `ReferenceError` en lugar de devolver `undefined`.

```javascript
for (let i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 100);
}
// Imprime: 0, 1, 2 — let crea un binding nuevo por iteración
```

---

### La confusión más común con `const`

`const` impide **reasignar la variable**, no congelar el contenido del objeto.

```js
const persona = { nombre: 'Ana' };
persona.nombre = 'Luis';   // válido: se muta la propiedad
persona = { nombre: 'Eva' }; // TypeError: Assignment to constant variable

const numeros = [1, 2, 3];
numeros.push(4);           //válido
```

Para congelar de verdad (superficialmente):

```js
const config = Object.freeze({ apiUrl: 'https://api.ejemplo.com' });
config.apiUrl = 'otra';   // silenciosamente ignorado (o TypeError en modo estricto)
```

---

### Template String

Las *template literals* usan acentos graves (`` ` ``) y permiten interpolación, saltos de línea reales y expresiones arbitrarias.

```javascript
const nombre = 'Ana';
const edad = 22;

// Antes
const viejo = 'Hola, ' + nombre + '. Tienes ' + edad + ' años.';

// Moderno
const nuevo = `Hola, ${nombre}. Tienes ${edad} años.`;

// Multilínea sin \n
const html = `
  <article>
    <h2>${nombre}</h2>
    <p>Edad: ${edad}</p>
  </article>
`;

// Cualquier expresión es válida dentro de ${}
const mayor = `Es mayor de edad: ${edad >= 18 ? 'sí' : 'no'}`;
const total = `Total: $${(120 * 1.16).toFixed(2)}`;
```

---

### Ejemplos - Variables, constantes y template strings

Archivo `variables.js`:

**1. scope de bloque**

```js
function demostrarScope() {
  if (true) {
    var conVar = 'soy var';
    let conLet = 'soy let';
    console.log(conLet); // ok, estamos dentro del bloque
  }
  console.log(conVar);   // "soy var" — escapó del bloque
  // console.log(conLet); // ReferenceError: conLet is not defined
}
demostrarScope();
```

**2. el ciclo con setTimeout**

```js
console.log('--- con var ---');
for (var i = 0; i < 3; i++) {
  setTimeout(() => console.log('var:', i), 0);
}

console.log('--- con let ---');
for (let j = 0; j < 3; j++) {
  setTimeout(() => console.log('let:', j), 0);
}
```

**3. const no congela objetos**

```js
const alumno = { nombre: 'Ana', calificacion: 8 };
alumno.calificacion = 10;
console.log(alumno); // { nombre: 'Ana', calificacion: 10 }
```
**4. template strings**
```js
const { nombre, calificacion } = alumno;
const estatus = calificacion >= 7 ? 'Aprobado' : 'Reprobado';
console.log(`
  Alumno:       ${nombre}
  Calificación: ${calificacion}
  Estatus:      ${estatus}
`);
```

---

### Ejercicio 1

**Consigna:** Refactoriza el siguiente código legacy usando `const`/`let` y template strings. El resultado debe imprimir exactamente lo mismo.

```javascript
var producto = 'Laptop';
var precio = 15000;
var descuento = 0.15;
var precioFinal = precio - (precio * descuento);
var mensaje = 'Producto: ' + producto + '\n' +
              'Precio original: $' + precio + '\n' +
              'Descuento: ' + (descuento * 100) + '%\n' +
              'Precio final: $' + precioFinal.toFixed(2);
console.log(mensaje);
```

**Solución:**

```javascript
const producto = 'Laptop';
const precio = 15000;
const descuento = 0.15;
const precioFinal = precio - precio * descuento;

console.log(`Producto: ${producto}
Precio original: $${precio}
Descuento: ${descuento * 100}%
Precio final: $${precioFinal.toFixed(2)}`);
```

---

### Ejercicio 2

Escribe una función `generarTicket(articulos)` que reciba un arreglo de objetos `{ nombre, cantidad, precioUnitario }` y retorne (no imprima) una cadena con formato de ticket usando template strings, incluyendo subtotal por artículo, total e IVA del 16%.


---

## 2. Funciones

### Funciones flecha

- Declaración tradicional
```js
function sumar(a, b) {
  return a + b;
}
```
- Expresión de función
```js
const sumar2 = function (a, b) {
  return a + b;
};
```
- Función flecha con cuerpo
```js
const sumar3 = (a, b) => {
  return a + b;
};
```

- Retorno implícito (una sola expresión)
```js
const sumar4 = (a, b) => a + b;
```
- Un solo parámetro: paréntesis opcionales
```js
const doble = n => n * 2;
```
- Sin parámetros: paréntesis obligatorios
```js
const saludar = () => 'Hola';
```

---


### Parámetros por defecto

- Antes de ES6
```js
function saludar(nombre) {
  nombre = nombre || 'invitado'; // bug: saludar('') da 'invitado'
  return `Hola ${nombre}`;
}
```
- ES
```js
const saludar2 = (nombre = 'invitado') => `Hola ${nombre}`;

saludar2();          // "Hola invitado"
saludar2(undefined); // "Hola invitado"
saludar2('');        // "Hola " — solo undefined dispara el default
saludar2(null);      // "Hola null" — null NO dispara el default
```

Los defaults pueden referenciar parámetros previos:

```javascript
const crearRango = (inicio, fin = inicio + 10) => ({ inicio, fin });
crearRango(5); // { inicio: 5, fin: 15 }
```

---

### Rest y spread

**Rest (`...`) en parámetros — *empaquetar*:**

```js
const sumarTodos = (...numeros) => numeros.reduce((acc, n) => acc + n, 0);
sumarTodos(1, 2, 3, 4); // 10

const registrar = (nivel, ...mensajes) => {
  console.log(`[${nivel}]`, mensajes.join(' | '));
};
registrar('ERROR', 'fallo de red', 'código 500');
```

Restricción: el parámetro rest debe ser **el último**.

**Spread (`...`) en llamadas y literales — *expande*:**

```js
const nums = [5, 1, 9, 3];
Math.max(...nums); // 9  (equivale a Math.max(5, 1, 9, 3))

// Copiar y combinar arreglos
const a = [1, 2];
const b = [3, 4];
const combinado = [...a, ...b, 5]; // [1, 2, 3, 4, 5]

// Copiar y combinar objetos
const base = { id: 1, nombre: 'Ana' };
const actualizado = { ...base, nombre: 'Luis', activo: true };
// { id: 1, nombre: 'Luis', activo: true }
```

**Advertencia — copia superficial:**

```javascript
const original = { nombre: 'Ana', direccion: { ciudad: 'CDMX' } };
const copia = { ...original };
copia.direccion.ciudad = 'Monterrey';
console.log(original.direccion.ciudad); // 'Monterrey' se afectó el original
```

El spread copia un solo nivel. Para anidados hay que hacer spread en cada nivel:

```javascript
const copiaSegura = {
  ...original,
  direccion: { ...original.direccion, ciudad: 'Monterrey' }
};
```

O usar `structuredClone(original)` en entornos modernos.

---

### Ejemplos - Funciones

Archivo `funciones.js`:

**1. Conversión progresiva**
```js
function esPar(n) { return n % 2 === 0; }
const esPar2 = (n) => n % 2 === 0;
console.log(esPar(4), esPar2(4)); // true true
```
**2. El error del objeto implícito**
```js
const malo = (n) => { nombre: n };
const bueno = (n) => ({ nombre: n });
console.log(malo('Ana'));  // undefined
console.log(bueno('Ana')); // { nombre: 'Ana' }

```
**3. this**
```js
const cronometro = {
  segundos: 0,
  arrancar() {
    const id = setInterval(() => {
      this.segundos++;
      console.log(`⏱  ${this.segundos}s`);
      if (this.segundos === 3) clearInterval(id);
    }, 500);
  }
};
cronometro.arrancar();
```
**4. Rest**
```js
const promedio = (...calificaciones) => {
  if (calificaciones.length === 0) return 0;
  const suma = calificaciones.reduce((acc, c) => acc + c, 0);
  return suma / calificaciones.length;
};
console.log(promedio(10, 8, 9, 7)); // 8.5
```
**5. Spread: patrón de actualización estilo React**
```js
const estadoInicial = {
  usuario: { id: 1, nombre: 'Ana' },
  tema: 'claro',
  notificaciones: 0
};

const nuevoEstado = {
  ...estadoInicial,
  tema: 'oscuro',
  notificaciones: estadoInicial.notificaciones + 1
};

console.log(estadoInicial); // intacto
console.log(nuevoEstado);
console.log(estadoInicial === nuevoEstado); // false → son objetos distintos
```

---

### Ejercicio 1

**Consigna:** Implementa `actualizarProducto(producto, cambios)` que retorne un **nuevo** objeto con los cambios aplicados, sin mutar el original. Debe funcionar también con la propiedad anidada `stock`.

```javascript
const producto = {
  id: 'P-001',
  nombre: 'Teclado mecánico',
  precio: 1200,
  stock: { almacen: 15, tienda: 4 }
};
```

**Solución:**

```javascript
const actualizarProducto = (producto, cambios = {}) => ({
  ...producto,
  ...cambios,
  stock: { ...producto.stock, ...(cambios.stock ?? {}) }
});

const p2 = actualizarProducto(producto, { precio: 999, stock: { tienda: 2 } });

console.log(producto.precio);      // 1200 — intacto
console.log(p2.precio);            // 999
console.log(p2.stock);             // { almacen: 15, tienda: 2 }
console.log(producto === p2);      // false
```

---

### Ejercicio 2

Implementa una función `crearLogger(prefijo)` que retorne otra función. La función retornada debe aceptar un número variable de mensajes y mostrarlos con el prefijo y una marca de tiempo.

```javascript
const logApi = crearLogger('API');
logApi('GET /usuarios', 'status 200');
// [14:32:05][API] GET /usuarios | status 200
```

---

## 3. Arreglos e inmutabilidad

### Dos familias de métodos

Esta clasificación es el concepto central del bloque y conviene escribirla en el pizarrón:

| **Mutan el arreglo original** | **Retornan uno nuevo** |
|---|---|
| `push`, `pop`, `shift`, `unshift` | `map`, `filter`, `concat`, `slice` |
| `splice`, `sort`, `reverse`, `fill` | `toSorted`, `toReversed`, `with` (ES2023) |

---

### Funcion `map`

Aplica una función a cada elemento y retorna un arreglo **de la misma longitud**.

```js
const precios = [100, 250, 80];
const conIva = precios.map(p => p * 1.16);
// [116, 290, 92.8]

const usuarios = [
  { id: 1, nombre: 'ana' },
  { id: 2, nombre: 'luis' }
];

const normalizados = usuarios.map(u => ({
  ...u,
  nombre: u.nombre.toUpperCase()
}));
```

El callback recibe `(elemento, indice, arregloCompleto)`.

---

### Funcion `filter`

Retorna un arreglo con los elementos que cumplen la condición. La longitud cambia.

```js
const numeros = [1, 2, 3, 4, 5, 6];
const pares = numeros.filter(n => n % 2 === 0); // [2, 4, 6]

const productos = [
  { nombre: 'Laptop', precio: 15000, disponible: true },
  { nombre: 'Mouse', precio: 300, disponible: false },
  { nombre: 'Monitor', precio: 4000, disponible: true }
];

const disponiblesBaratos = productos
  .filter(p => p.disponible)
  .filter(p => p.precio < 10000);
```

**Patrón de eliminación inmutable** (muy usado en React):

```js
const eliminarPorId = (lista, id) => lista.filter(item => item.id !== id);
```

### Funcion `reduce`

El más general y el que más cuesta. Reduce el arreglo a **un solo valor**, que puede ser número, string, objeto o incluso otro arreglo.

```javascript
// Firma: array.reduce((acumulador, elemento, indice, arreglo) => nuevoAcumulador, valorInicial)

const nums = [1, 2, 3, 4];
const suma = nums.reduce((acc, n) => acc + n, 0); // 10
```

Casos más interesantes:

```js
// Total de un carrito
const carrito = [
  { nombre: 'Laptop', precio: 15000, cantidad: 1 },
  { nombre: 'Mouse', precio: 300, cantidad: 2 }
];
const total = carrito.reduce((acc, item) => acc + item.precio * item.cantidad, 0);

// Agrupar por categoría → retorna un objeto
const items = [
  { nombre: 'Manzana', tipo: 'fruta' },
  { nombre: 'Zanahoria', tipo: 'verdura' },
  { nombre: 'Pera', tipo: 'fruta' }
];

const porTipo = items.reduce((acc, item) => {
  const grupo = acc[item.tipo] ?? [];
  return { ...acc, [item.tipo]: [...grupo, item.nombre] };
}, {});
// { fruta: ['Manzana', 'Pera'], verdura: ['Zanahoria'] }

// Convertir arreglo a diccionario indexado por id (patrón frecuente en estado)
const usuarios = [{ id: 'a', n: 1 }, { id: 'b', n: 2 }];
const indice = usuarios.reduce((acc, u) => ({ ...acc, [u.id]: u }), {});
```

**Olvidar el valor inicial** es el error más común: sin él, `reduce` usa el primer elemento como acumulador inicial y falla en arreglos vacíos (`TypeError`).

---

### Métodos de búsqueda y verificación

```javascript
const usuarios = [
  { id: 1, nombre: 'Ana', activo: true },
  { id: 2, nombre: 'Luis', activo: false }
];

usuarios.find(u => u.id === 2);        // { id: 2, ... } — el elemento o undefined
usuarios.findIndex(u => u.id === 2);   // 1 — el índice o -1
usuarios.some(u => !u.activo);         // true — ¿al menos uno?
usuarios.every(u => u.activo);         // false — ¿todos?
usuarios.includes('Ana');              // false — compara con ===, no sirve con objetos
```

---

### El problema de `sort`

`sort` **muta** el arreglo y, sin comparador, convierte todo a string:

```javascript
[10, 9, 100].sort();               // [10, 100, 9] 
[10, 9, 100].sort((a, b) => a - b); // [9, 10, 100]
```

Para ordenar sin mutar:

```javascript
const ordenado = [...original].sort((a, b) => a.precio - b.precio);
// o, en entornos ES2023:
const ordenado2 = original.toSorted((a, b) => a.precio - b.precio);
```

---

### Ejemplos

Archivo `arreglos.js`:

```js
const inventario = [
  { id: 1, nombre: 'Laptop',   categoria: 'computo',    precio: 15000, stock: 3 },
  { id: 2, nombre: 'Mouse',    categoria: 'accesorios', precio: 300,   stock: 25 },
  { id: 3, nombre: 'Teclado',  categoria: 'accesorios', precio: 1200,  stock: 0 },
  { id: 4, nombre: 'Monitor',  categoria: 'computo',    precio: 4500,  stock: 7 },
  { id: 5, nombre: 'Webcam',   categoria: 'accesorios', precio: 850,   stock: 12 }
];
```
**1. Agregar propiedad calculada `map`**
```js
const conValorTotal = inventario.map(p => ({
  ...p,
  valorInventario: p.precio * p.stock
}));
console.table(conValorTotal);
```
**2. Solo lo que hay en existencia `filter`**
```js
const disponibles = inventario.filter(p => p.stock > 0);
console.log(`Disponibles: ${disponibles.length} de ${inventario.length}`);
```
**3. Valor total del inventario `reduce`**
```js
const valorTotal = inventario.reduce((acc, p) => acc + p.precio * p.stock, 0);
console.log(`Valor total: $${valorTotal.toLocaleString('es-MX')}`);
```
**4. Agrupar por categoría `reduce`**
```js
const porCategoria = inventario.reduce((acc, p) => {
  const actuales = acc[p.categoria] ?? [];
  return { ...acc, [p.categoria]: [...actuales, p.nombre] };
}, {});
console.log(porCategoria);
```
**5. Accesorios disponibles, ordenados por precio `Encadenamiento`**
```js
const resultado = inventario
  .filter(p => p.categoria === 'accesorios' && p.stock > 0)
  .map(p => ({ nombre: p.nombre, precio: p.precio }))
  .sort((a, b) => a.precio - b.precio);
console.log(resultado);
```

---

### Ejercicio 1

**Consigna:** Con el arreglo `inventario`, produce un reporte por categoría con esta forma:

```js
{
  computo:    { totalProductos: 2, valorTotal: 76500, precioPromedio: 9750 },
  accesorios: { totalProductos: 3, valorTotal: 17700, precioPromedio: 783.33 }
}
```

**Solución:**

```js
const reportePorCategoria = (productos) => {
  const agrupado = productos.reduce((acc, p) => {
    const grupo = acc[p.categoria] ?? [];
    return { ...acc, [p.categoria]: [...grupo, p] };
  }, {});

  return Object.entries(agrupado).reduce((acc, [categoria, items]) => {
    const valorTotal = items.reduce((s, p) => s + p.precio * p.stock, 0);
    const sumaPrecios = items.reduce((s, p) => s + p.precio, 0);

    return {
      ...acc,
      [categoria]: {
        totalProductos: items.length,
        valorTotal,
        precioPromedio: Number((sumaPrecios / items.length).toFixed(2))
      }
    };
  }, {});
};

console.log(reportePorCategoria(inventario));
```

### Ejercicio 2

Dado un arreglo de calificaciones:

```js
const alumnos = [
  { matricula: 'A01', nombre: 'Ana',   parciales: [8, 9, 10] },
  { matricula: 'A02', nombre: 'Luis',  parciales: [6, 5, 7]  },
  { matricula: 'A03', nombre: 'María', parciales: [10, 10, 9] },
  { matricula: 'A04', nombre: 'Jorge', parciales: [4, 6, 5]  }
];
```

Implementa, **sin mutar el arreglo original** y usando solo `map`, `filter`, `reduce` y `sort`:

1. `conPromedio(alumnos)` → agrega la propiedad `promedio` a cada alumno.
2. `aprobados(alumnos)` → solo los que tienen promedio ≥ 7.
3. `mejorAlumno(alumnos)` → el objeto del alumno con mayor promedio.
4. `estadisticas(alumnos)` → `{ promedioGrupal, aprobados, reprobados, calificacionMasAlta }`.
5. `cuadroHonor(alumnos)` → los 2 mejores, ordenados de mayor a menor.

---

## 4. Desestructuración

La desestructuración extrae valores de objetos y arreglos hacia variables individuales con una sintaxis declarativa.

### Objetos

```js
const usuario = {
  id: 1,
  nombre: 'Ana',
  email: 'ana@ejemplo.com',
  direccion: { ciudad: 'CDMX', cp: '06700' }
};

// Básico
const { nombre, email } = usuario;

// Renombrar
const { nombre: nombreUsuario } = usuario;

// Valor por defecto
const { telefono = 'No registrado' } = usuario;

// Anidado
const { direccion: { ciudad } } = usuario;

// Anidado + renombrado + default
const { direccion: { pais = 'México' } } = usuario;

// Rest: "el resto de las propiedades"
const { id, ...datosPublicos } = usuario;
// datosPublicos = { nombre, email, direccion }
```

El patrón `const { id, ...resto } = obj` es la forma idiomática de **quitar una propiedad de un objeto sin mutarlo**.

---

### Arreglos

Se desestructura por **posición**, no por nombre:

```js
const coordenadas = [19.43, -99.13];
const [latitud, longitud] = coordenadas;

// Saltar posiciones
const colores = ['rojo', 'verde', 'azul'];
const [, , tercero] = colores; // 'azul'

// Con default
const [a, b, c = 0] = [1, 2];

// Rest
const [primero, ...demas] = [1, 2, 3, 4]; // demas = [2, 3, 4]

// Intercambio sin variable temporal
let x = 1, y = 2;
[x, y] = [y, x];
```

---

### Desestructuración en parámetros

Es donde más impacto tiene en legibilidad:

```javascript
// Antes
function mostrarUsuario(usuario) {
  console.log(`${usuario.nombre} <${usuario.email}>`);
}

// Después
const mostrarUsuario = ({ nombre, email }) => {
  console.log(`${nombre} <${email}>`);
};

// Con defaults y objeto de configuración
const conectar = ({ host = 'localhost', puerto = 3000, seguro = false } = {}) =>
  `${seguro ? 'https' : 'http'}://${host}:${puerto}`;

conectar();                              // http://localhost:3000
conectar({ host: 'api.com', seguro: true }); // https://api.com:3000
```

El `= {}` final permite llamar la función sin argumentos. Sin él, `conectar()` lanzaría error al intentar desestructurar `undefined`.

---

### Ejemplos

Archivo `desestructuracion.js`:

```js
const respuestaApi = {
  status: 200,
  data: {
    usuarios: [
      { id: 1, nombre: 'Ana',  rol: 'admin',   perfil: { avatar: 'a.png', bio: 'Dev' } },
      { id: 2, nombre: 'Luis', rol: 'usuario', perfil: { avatar: 'l.png' } }
    ],
    total: 2
  },
  meta: { pagina: 1, porPagina: 10 }
};
```
**1. Extracción profunda en una línea**
```js
const { data: { usuarios, total }, meta: { pagina } } = respuestaApi;
console.log(`Página ${pagina}: ${total} usuarios`);
```
**2. Desestructurar dentro de map**
```js
const resumen = usuarios.map(({ nombre, rol, perfil: { bio = 'Sin biografía' } }) =>
  `${nombre} (${rol}): ${bio}`
);
console.log(resumen);
```
**3. Omitir una propiedad sin mutar (patrón rest)**
```js
const [usuarioAna] = usuarios;
const { perfil, ...anaSinPerfil } = usuarioAna;
console.log(anaSinPerfil); // { id: 1, nombre: 'Ana', rol: 'admin' }
console.log(usuarioAna);   // intacto, sigue teniendo perfil
```
**4. Parámetros desestructurados con defaults**
```js
const construirUrl = ({
  base = 'https://api.ejemplo.com',
  recurso,
  pagina = 1,
  limite = 10
}) => `${base}/${recurso}?_page=${pagina}&_limit=${limite}`;

console.log(construirUrl({ recurso: 'usuarios' }));
console.log(construirUrl({ recurso: 'productos', pagina: 3, limite: 25 }));
```
---

### Ejercicio 1

**Consigna:** Refactoriza esta función para que use desestructuración en el parámetro, valores por defecto y template strings:

```javascript
function formatearDireccion(pedido) {
  var envio = pedido.envio;
  var calle = envio.calle;
  var numero = envio.numero;
  var ciudad = envio.ciudad;
  var cp = envio.cp;
  var pais = envio.pais ? envio.pais : 'México';
  return calle + ' ' + numero + ', ' + ciudad + ', CP ' + cp + ', ' + pais;
}
```

**Solución:**

```javascript
const formatearDireccion = ({
  envio: { calle, numero, ciudad, cp, pais = 'México' }
}) => `${calle} ${numero}, ${ciudad}, CP ${cp}, ${pais}`;
```

---

### Ejercicio 2

Dada la respuesta:

```javascript
const clima = {
  ciudad: 'Ciudad de México',
  actual: { temp: 22, sensacion: 20, humedad: 45, viento: { velocidad: 12, direccion: 'NE' } },
  pronostico: [
    { dia: 'Lunes',    max: 24, min: 12 },
    { dia: 'Martes',   max: 26, min: 14 },
    { dia: 'Miércoles', max: 21, min: 11 }
  ]
};
```

Escribe `resumenClima(clima)` que use **solo desestructuración** (nada de `clima.actual.temp`) y retorne una cadena formateada con la ciudad, la temperatura actual, la velocidad del viento y el pronóstico del primer día.

---

## 5. Clases, objetos y módulos

### Objetos

```js
const nombre = 'Ana';
const edad = 22;

// Propiedades abreviadas (shorthand)
const usuario = { nombre, edad };  // en vez de { nombre: nombre, edad: edad }

// Métodos abreviados
const servicio = {
  url: '/api',
  obtener() { return `GET ${this.url}`; }   // en vez de obtener: function() {...}
};

// Claves computadas
const campo = 'email';
const formulario = { [campo]: 'ana@ejemplo.com', [`${campo}Valido`]: true };
// { email: '...', emailValido: true }
```

---

### Métodos estáticos de `Object`

```js
const obj = { a: 1, b: 2, c: 3 };

Object.keys(obj);    // ['a', 'b', 'c']
Object.values(obj);  // [1, 2, 3]
Object.entries(obj); // [['a',1], ['b',2], ['c',3]]

// Recorrer un objeto como si fuera arreglo
Object.entries(obj).map(([clave, valor]) => `${clave}=${valor}`);

// Reconstruir objeto desde pares
Object.fromEntries([['x', 10], ['y', 20]]); // { x: 10, y: 20 }
```

---

### Clases

Las clases de ES son *azúcar sintáctica* sobre el sistema de prototipos de JavaScript. No introducen un modelo de objetos nuevo: `class` sigue creando funciones constructoras y enlazando `prototype`.

```javascript
class Producto {
  // Campos privados (ES): solo accesibles dentro de la clase
  #costoInterno;

  // Campo público con valor inicial
  categoria = 'general';

  constructor(nombre, precio, costoInterno = 0) {
    this.nombre = nombre;
    this.precio = precio;
    this.#costoInterno = costoInterno;
  }

  // Método de instancia
  aplicarDescuento(porcentaje) {
    this.precio = this.precio * (1 - porcentaje / 100);
    return this; // permite encadenar
  }

  // Getter: se accede como propiedad, no como método
  get margen() {
    return this.precio - this.#costoInterno;
  }

  // Setter con validación
  set precioConIva(valor) {
    this.precio = valor / 1.16;
  }

  // Método estático: pertenece a la clase, no a las instancias
  static desdeJson(json) {
    const { nombre, precio, costo } = JSON.parse(json);
    return new Producto(nombre, precio, costo);
  }

  // Sobrescribir toString
  toString() {
    return `${this.nombre}: $${this.precio.toFixed(2)}`;
  }
}

const p = new Producto('Laptop', 15000, 11000);
console.log(p.margen);        // 4000 — sin paréntesis, es getter
p.aplicarDescuento(10);
console.log(`${p}`);          // "Laptop: $13500.00"
// console.log(p.#costoInterno); // SyntaxError: campo privado
```

---

### Herencia

```javascript
class ProductoDigital extends Producto {
  constructor(nombre, precio, urlDescarga) {
    super(nombre, precio, 0);   // super() debe ir ANTES de usar this
    this.urlDescarga = urlDescarga;
    this.categoria = 'digital';
  }

  // Sobrescritura
  toString() {
    return `${super.toString()} [descargable]`;
  }
}

const ebook = new ProductoDigital('Curso React', 799, '/d/react.zip');
console.log(`${ebook}`);              // "Curso React: $799.00 [descargable]"
console.log(ebook instanceof Producto); // true
```

---

### Módulos ES

Archivo `utilidades.js`
```js
export const PI = 3.14159;                          // exportación nombrada
export const areaCirculo = (r) => PI * r ** 2;

export default class Calculadora { /* ... */ }      // exportación por defecto (una por archivo)
```

Archivo `main.js`

```js
import Calculadora from './utilidades.js';                  // default: nombre libre
import { PI, areaCirculo } from './utilidades.js';          // nombradas: nombre exacto
import { PI as piValor } from './utilidades.js';            // renombrada
import * as utils from './utilidades.js';                   // todo como espacio de nombres
import Calculadora, { PI } from './utilidades.js';          // combinado
```

---

## 6. Promesas y Fetch API

### El modelo de ejecución de JavaScript

JavaScript es *single-threaded*: hay un solo hilo de ejecución y una sola pila de llamadas. Las operaciones lentas (red, temporizadores, lectura de archivos) no se ejecutan en ese hilo; se delegan al entorno (navegador o Node), que las devuelve mediante el **event loop** cuando terminan.

Conviene dibujar el esquema en el pizarrón:

```
   Call Stack          Web APIs              Task Queues
  ┌──────────┐      ┌─────────────┐      ┌────────────────┐
  │ función  │ ───► │ fetch       │ ───► │ Microtask Q    │  ← promesas
  │ función  │      │ setTimeout  │      │ (prioridad)    │
  └──────────┘      └─────────────┘      ├────────────────┤
        ▲                                 │ Macrotask Q    │  ← setTimeout
        └──────── Event Loop ─────────────┘
```

Ejemplo:

```js
console.log('1');
setTimeout(() => console.log('2'), 0);
Promise.resolve().then(() => console.log('3'));
console.log('4');
// Salida: 1, 4, 3, 2
```

Las promesas van a la cola de microtareas, que el event loop vacía **antes** que la de macrotareas.

### Del callback hell a las promesas

```javascript
// Callback hell
obtenerUsuario(1, (usuario) => {
  obtenerPedidos(usuario.id, (pedidos) => {
    obtenerDetalle(pedidos[0].id, (detalle) => {
      console.log(detalle);        // 3 niveles de anidación
    }, manejarError);
  }, manejarError);
}, manejarError);
```

Una **promesa** es un objeto que representa el resultado futuro de una operación asíncrona. Tiene tres estados: `pending`, `fulfilled` (con valor) y `rejected` (con razón). Una vez resuelta, es inmutable.

```javascript
const promesa = new Promise((resolve, reject) => {
  setTimeout(() => {
    const exito = Math.random() > 0.3;
    exito ? resolve({ id: 1, nombre: 'Ana' }) : reject(new Error('Falló la red'));
  }, 1000);
});

promesa
  .then(datos => console.log('Éxito:', datos))
  .catch(error => console.error('Error:', error.message))
  .finally(() => console.log('Terminó, con éxito o sin él'));
```

**Encadenamiento:** lo que retorne un `.then()` se convierte en el valor del siguiente. Si retorna una promesa, se espera a que se resuelva antes de continuar. Esto convierte la anidación en una secuencia plana.

```javascript
obtenerUsuario(1)
  .then(usuario => obtenerPedidos(usuario.id))
  .then(pedidos => obtenerDetalle(pedidos[0].id))
  .then(detalle => console.log(detalle))
  .catch(error => manejarError(error));   // un solo catch para toda la cadena
```

---

### `async` / `await`

Azúcar sintáctica sobre promesas que permite escribir código asíncrono con apariencia secuencial.

- Una función `async` **siempre** retorna una promesa.
- `await` solo puede usarse dentro de una función `async` (o en el nivel superior de un módulo ES).
- Los errores se capturan con `try/catch` convencional.

```javascript
const obtenerDatosCompletos = async (idUsuario) => {
  try {
    const usuario = await obtenerUsuario(idUsuario);
    const pedidos = await obtenerPedidos(usuario.id);
    const detalle = await obtenerDetalle(pedidos[0].id);
    return { usuario, detalle };
  } catch (error) {
    console.error('Falló la cadena:', error.message);
    throw error;                 // re-lanzar si quien llama debe enterarse
  } finally {
    console.log('Proceso concluido');
  }
};
```
---

### Combinadores de promesas

| Método | Comportamiento |
|---|---|
| `Promise.all([...])` | Resuelve con todos los valores; **falla si una falla** |
| `Promise.allSettled([...])` | Espera todas y retorna `{status, value/reason}` de cada una; nunca falla |
| `Promise.race([...])` | Se resuelve/rechaza con la primera que termine (útil para timeouts) |
| `Promise.any([...])` | Se resuelve con la primera que tenga éxito; ignora fallos |

```javascript
// Timeout manual con race
const conTimeout = (promesa, ms) => Promise.race([
  promesa,
  new Promise((_, reject) =>
    setTimeout(() => reject(new Error('Timeout')), ms))
]);
```

---

### Fetch API

`fetch` retorna una promesa que resuelve con un objeto `Response`.

**Las dos trampas fundamentales:**

1. **`fetch` no rechaza en errores HTTP.** Un 404 o un 500 resuelven normalmente. La promesa solo se rechaza ante fallos de red, CORS o cancelación. **Siempre hay que verificar `response.ok`.**
2. **El cuerpo requiere una segunda promesa.** `response.json()` es asíncrono porque el cuerpo puede llegar en fragmentos.

```javascript
const obtenerUsuarios = async () => {
  const respuesta = await fetch('https://jsonplaceholder.typicode.com/users');

  if (!respuesta.ok) {
    throw new Error(`HTTP ${respuesta.status}: ${respuesta.statusText}`);
  }

  return respuesta.json();   // segunda promesa
};
```

**Peticiones con cuerpo:**

```javascript
const crearPost = async (datos) => {
  const respuesta = await fetch('https://jsonplaceholder.typicode.com/posts', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      // 'Authorization': `Bearer ${token}`
    },
    body: JSON.stringify(datos)     // el cuerpo debe ser string
  });

  if (!respuesta.ok) throw new Error(`Error ${respuesta.status}`);
  return respuesta.json();
};
```

**Cancelación con `AbortController`:**

```javascript
const controlador = new AbortController();

fetch(url, { signal: controlador.signal })
  .then(r => r.json())
  .catch(err => {
    if (err.name === 'AbortError') return; // cancelación esperada, no es error
    console.error(err);
  });

controlador.abort(); // cancela la petición
```

### Ejemplos

Archivo `async.js`:

**1. Orden de ejecución: microtareas vs macrotareas**
```js
const BASE = 'https://jsonplaceholder.typicode.com';

console.log('A - síncrono');
setTimeout(() => console.log('D - macrotarea'), 0);
Promise.resolve().then(() => console.log('C - microtarea'));
console.log('B - síncrono');
```
**2. Crear una promesa manualmente**
```js
const esperar = (ms) => new Promise(resolve => setTimeout(resolve, ms));

const operacionRiesgosa = (probabilidadExito = 0.7) =>
  new Promise((resolve, reject) => {
    setTimeout(() => {
      Math.random() < probabilidadExito
        ? resolve('Operación completada')
        : reject(new Error('La operación falló'));
    }, 500);
  });
```
**3. Helper de fetch con verificación de estado**
```js
const obtenerJson = async (url, opciones = {}) => {
  const respuesta = await fetch(url, opciones);
  if (!respuesta.ok) {
    throw new Error(`HTTP ${respuesta.status} en ${url}`);
  }
  return respuesta.json();
};
```
**4. Comparación secuencial vs paralelo**
```js
const secuencial = async () => {
  console.time('secuencial');
  const usuarios = await obtenerJson(`${BASE}/users`);
  const posts    = await obtenerJson(`${BASE}/posts`);
  const comments = await obtenerJson(`${BASE}/comments`);
  console.timeEnd('secuencial');
  return { usuarios: usuarios.length, posts: posts.length, comments: comments.length };
};

const paralelo = async () => {
  console.time('paralelo');
  const [usuarios, posts, comments] = await Promise.all([
    obtenerJson(`${BASE}/users`),
    obtenerJson(`${BASE}/posts`),
    obtenerJson(`${BASE}/comments`)
  ]);
  console.timeEnd('paralelo');
  return { usuarios: usuarios.length, posts: posts.length, comments: comments.length };
};
```
**5. Cadena dependiente**
```js
const perfilCompleto = async (idUsuario) => {
  const usuario = await obtenerJson(`${BASE}/users/${idUsuario}`);
  // Estas dos no dependen entre sí → van en paralelo
  const [posts, todos] = await Promise.all([
    obtenerJson(`${BASE}/posts?userId=${usuario.id}`),
    obtenerJson(`${BASE}/todos?userId=${usuario.id}`)
  ]);

  return {
    nombre: usuario.name,
    empresa: usuario.company.name,
    totalPosts: posts.length,
    pendientes: todos.filter(t => !t.completed).length
  };
};
```
**6. Manejo de errores y allSettled**
```js
const robusto = async () => {
  const resultados = await Promise.allSettled([
    obtenerJson(`${BASE}/users/1`),
    obtenerJson(`${BASE}/ruta-inexistente`),   // fallará con 404
    obtenerJson(`${BASE}/posts/1`)
  ]);

  resultados.forEach((r, i) => {
    r.status === 'fulfilled'
      ? console.log(`[${i}] ok`)
      : console.log(`[${i}] ${r.reason.message}`);
  });
};
```
**7. POST**
```js
const crearPost = async () => {
  const nuevo = await obtenerJson(`${BASE}/posts`, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ title: 'Curso React', body: 'Sesión 1', userId: 1 })
  });
  console.log('Creado:', nuevo);
};

//Ejecutar
(async () => {
  await esperar(100);
  console.log(await secuencial());
  console.log(await paralelo());
  console.log(await perfilCompleto(3));
  await robusto();
  await crearPost();

  try {
    console.log(await operacionRiesgosa(0.2));
  } catch (e) {
    console.error('Capturado:', e.message);
  }
})();
```