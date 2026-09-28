# Instalacion de Dependencias en React 
---
## Dependencias JS 

### Bootstrap

Libreria CSS de código abierto, muy popular, que facilita el desarrollo rápido de interfaces web responsivas y adaptables a diferentes dispositivos. 

Proporciona un conjunto de componentes y estilos predefinidos en HTML, CSS y JavaScript que se pueden utilizar para crear sitios web y aplicaciones web de manera eficiente. 
```bash
npm i bootstrap @popperjs/core 
```

* Importacion (index.js):
```js
import 'bootstrap/dist/css/bootstrap.min.css';
import 'bootstrap/dist/js/bootstrap.bundle.min.js';
```
---

### SweetAlert 2 
Libreria de JavaScript que permite crear ventanas emergentes (modales) atractivas y personalizables para reemplazar las alertas y confirmaciones estándar de JavaScript en aplicaciones web. Ofrece una alternativa visualmente más atractiva y funcional a las alertas tradicionales, con opciones para personalizar iconos, colores, posiciones y más. 
```bash
npm i sweetalert2
```

* Importacion (archivo actual.jsx):
```js
import Swal from "sweetalert2";
```

---

### Font Awesome
Libreria de iconos vectoriales y una herramienta para diseñadores y desarrolladores web. Funciona como una fuente tipográfica, lo que permite escalar los iconos a cualquier tamaño sin perder calidad, y personalizarlos con CSS. 
```bash
npm i --save @fortawesome/fontawesome-svg-core @fortawesome/react-fontawesome
npm i --save @fortawesome/free-brands-svg-icons @fortawesome/free-solid-svg-icons
```

* Importacion (archivo actual.jsx):
```js
import {FontAwesomeIcon}  from '@fortawesome/react-fontawesome';
import { faKey} from "@fortawesome/free-solid-svg-icons";
```
* Ejemplo:
```js
<FontAwesomeIcon icon={faKey}/>
```

---

### AOS | Animation On Scroll
Libreria  de JavaScript que permite agregar animaciones a elementos HTML cuando el usuario se desplaza por la página web. En otras palabras, AOS facilita la creación de efectos visuales atractivos que se activan a medida que el usuario interactúa con el contenido desplazándose hacia arriba o hacia abajo en la página.
```bash
npm i aos
```

* Importacion (App.jsx):
```js
import AOS from 'aos';

AOS.init({
    duration: 1000,
    once: true, 
});
```

---

## Dependencias de React

### React Router DOM
Libreria de JavaScript para React que facilita la implementación de la navegación en aplicaciones web de una sola página (SPA). Permite definir rutas y asociarlas a componentes, lo que posibilita la navegación entre diferentes vistas sin necesidad de recargar la página completa, mejorando la experiencia del usuario. 
```bash
npm i react-router-dom
```

* Importacion (App.jsx):
```js
import {BrowserRouter,Routes,Route} from "react-router-dom";
```
* Ejemplo:
```js
<BrowserRouter>
  <Routes>
    <Routes path="/" element={<Home/>}/>
    <Route path="*" element={<Error/>}/>
  </Routes>
</BrowserRouter> 
```

---

### React Hook Form
Libreria de React que facilita la gestión y validación de formularios. Se destaca por su alto rendimiento, bajo peso y la ausencia de dependencias externas. Utiliza un enfoque no controlado, lo que significa que se apoya en el DOM nativo para manejar los valores de los campos, reduciendo así la necesidad de re-renders innecesarios. 
 
```bash
npm i react-hook-form
```
* Importacion (Actual.jsx):
```js
import { useForm } from "react-hook-form";
```

---

## Dependencia para Tablas

### React Data Table Component
Librería para React que permite crear tablas interactivas y personalizables sin necesidad de depender de jQuery ni de DataTables clásico.

Solución hecha 100% en React para mostrar datos en forma de tabla con características modernas como: 

* Paginación automática
* Ordenamiento (sorting) por columnas
* Búsqueda / filtrado de datos
* Selección de filas (checkboxes)
* Diseño responsive
* Temas y estilos personalizables
* Renderizado condicional y celdas personalizadas

```bash
npm i react-data-table-component
```
* Ejemplo:
```js
import DataTable from "react-data-table-component";

const columnas = [
  {
    name: "ID",
    selector: row => row.id,
    sortable: true
  },
  {
    name: "Nombre",
    selector: row => row.nombre,
    sortable: true
  },
  {
    name: "Edad",
    selector: row => row.edad,
    sortable: true
  }
];

const datos = [
  { id: 1, nombre: "Juan", edad: 25 },
  { id: 2, nombre: "Ana", edad: 30 },
  { id: 3, nombre: "Pedro", edad: 22 }
];

const App = () => {
  return (
    <DataTable
      title="Lista de Usuarios"
      columns={columnas}
      data={datos}
      pagination
      highlightOnHover
    />
  );
}

export default App;
```
---

## Dependencias para Generacion de Archivos

### jsPDF 
Librería de JavaScript puro que permite crear archivos PDF directamente en el navegador (o en Node.js) sin necesidad de un servidor.

Su propósito principal es generar documentos PDF de forma dinámica, por ejemplo: facturas, reportes, recibos, tickets o certificados, y permitir que el usuario los descargue o los visualice en línea.
```bash
npm i jspdf
```
* Ejemplo:
```js
import jsPDF from "jspdf";

const doc = new jsPDF();

doc.text("Hola, este es un PDF generado con jsPDF", 10, 10);

doc.save("documento.pdf");
```

---

### jsPDF Auto Table
Librería jsPDF que permite generar tablas automáticas en documentos PDF directamente desde JavaScript.

Mientras que jsPDF sirve para crear PDFs desde cero (texto, imágenes, formas, etc.), jspdf-autotable se enfoca en dibujar tablas completas con encabezados, filas, estilos y formatos sin tener que calcular manualmente posiciones o alineaciones.
```bash
npm i jspdf-autotable
```
* Ejemplo: 
```js
import jsPDF from "jspdf";
import "jspdf-autotable";

const doc = new jsPDF();

const columnas = ["ID", "Nombre", "Edad"];
const filas = [
  [1, "Juan", 25],
  [2, "Ana", 30],
  [3, "Pedro", 22]
];

doc.autoTable({
  head: [columnas],
  body: filas,
});

doc.save("tabla.pdf");
```

---

### XLSX | SheetJS
Librería xlsx (también conocida como SheetJS) es una herramienta de JavaScript que permite leer, crear, modificar y exportar archivos de Excel (.xlsx, .xls, .csv) directamente desde el navegador o Node.js, sin necesidad de tener Excel instalado.
```bash
npm i xlsx
```
* Ejemplo:
```js
import * as XLSX from "xlsx";

const datos = [
  ["ID", "Nombre", "Edad"],
  [1, "Juan", 25],
  [2, "Ana", 30],
  [3, "Pedro", 22]
];

const hoja = XLSX.utils.aoa_to_sheet(datos);

const libro = XLSX.utils.book_new();
XLSX.utils.book_append_sheet(libro, hoja, "Usuarios");

XLSX.writeFile(libro, "usuarios.xlsx");
```

---

### File Saver
Librería de JavaScript que sirve para guardar o descargar archivos directamente desde el navegador, sin necesidad de depender de un backend para enviarlos como descarga.

En pocas palabras, permite que tu aplicación genere un archivo en memoria (PDF, Excel, imagen, texto, etc.) y lo guarde en la computadora del usuario.
```bash
npm i file-saver
```
* Ejemplo:
```js
import * as XLSX from "xlsx";
import { saveAs } from "file-saver";

const datos = [
  ["ID", "Nombre", "Edad"],
  [1, "Juan", 25],
  [2, "Ana", 30]
];

const hoja = XLSX.utils.aoa_to_sheet(datos);
const libro = XLSX.utils.book_new();
XLSX.utils.book_append_sheet(libro, hoja, "Usuarios");

// Convertir a binario y guardar
const excelBuffer = XLSX.write(libro, { bookType: "xlsx", type: "array" });
const blob = new Blob([excelBuffer], { type: "application/octet-stream" });
saveAs(blob, "usuarios.xlsx");
```

---

## Dependencia para Cuentas Regresivas

### React CountDown
Librería para React que permite crear cuentas regresivas (countdowns) de manera sencilla y totalmente personalizable.

Básicamente, le dices una fecha/hora objetivo, y la librería se encarga de calcular y actualizar automáticamente el tiempo restante hasta llegar a esa fecha.
```bash
npm i react-countdown
```
* Ejemplo:
```js
import Countdown from "react-countdown";

const App = () => {
  const fechaObjetivo = Date.now() + 1000 * 60 * 5; // 5 minutos desde ahora

  return (
    <div>
      <h2>Cuenta regresiva</h2>
      <Countdown date={fechaObjetivo} />
    </div>
  );
}
export default App;
```