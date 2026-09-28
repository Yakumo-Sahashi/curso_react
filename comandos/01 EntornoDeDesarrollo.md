# Entorno de Desarrollo JavaScript Moderno

## ¿Qué es ECMAScript?

**ECMAScript** (o abreviado ES) es la especificación en la que se basa JavaScript. ES nace de la organización **ECMA** International, cuyo objetivo es desarrollar estándares y reportes técnicos para facilitar el uso de tecnologías de la información.

---

## Ventajas de ECMAScript

- Funciones de flecha (arrow functions) 
- Let y Const
- Clases
- Métodos Privados 
- FETCH
- Desestructuración
- Templete String
- Map y Set

---

## ¿Por Qué Es Necesario Transformar El Código Js?

**JavaScript** es un lenguaje que no para de evolucionar y que cada año agrega nuevas características a su estándar. El estándar de JavaScript, conocido como **ECMAScript**.

El problema histórico: los navegadores no adoptan las versiones al mismo ritmo. La solución de la industria fue el **transpilador**, un programa que traduce código moderno a una versión anterior compatible. El transpilador estándar es **Babel**, y viene incluido en Create React App. Esto significa que en este curso podemos escribir sintaxis moderna sin preocuparnos por compatibilidad.

---

## Componentes del entorno

| Herramienta | Función | Versión sugerida |
|---|---|---|
| **Node.js** | Runtime de JavaScript fuera del navegador. Ejecuta las herramientas de construcción. | LTS (18.x o superior) |
| **npm** | Gestor de paquetes. Se instala junto con Node. | 9.x o superior |
| **VS Code** | Editor. | Última estable |
| **Navegador** | Firefox Developer, por sus DevTools. | Última estable |

---

## Ejemplo:

### 1. Verificar instalaciones
```bash
node -v     # debe imprimir v18.x.x o superior
npm -v      # debe imprimir 9.x.x o superior
```

### 2. Crear la carpeta de trabajo
```bash
mkdir curso-react && cd curso-react
```

### 3. Inicializar un proyecto Node
```bash
npm init -y
```

Archivo `package.json`

```json
{
  "name": "proyecto1",
  "version": "1.0.0",
  "description": "Mi primer proyecto con node",
  "license": "MIT",
  "author": "diego",
  "main": "index.js",
  "scripts": {
    "build" "webpack",
    "dev" "webpack"
  }
}
```

---

## Preparación del entorno de desarrollo


### ¿Qué es babel?

Babel es un "compilador" (o transpilador) para JavaScript. Básicamente permite transformar código escrito con las últimas y novedosas características de JavaScript y transformarlo en un código que sea entendido por navegadores más antiguos.

---

### Instalación De Babel

```bash
npm i --save-dev @babel/core @babel/cli @babel/preset-env

#--save-dev  -> guarda los paquetes como dependencias de desarrollo
#@babel/core   -> paquete principal de babel
#@babel/cli     -> línea de comandos 
#@babel/preset-env  -> preset  (conjunto de reglas preconfiguradas)

```

---

### Configuración

- Añadir un nuevo objeto al archivo `package.json` con el siguiente contenido:

```json
"scripts": {
    "build": "babel src -d controller"
}
```

- Crear archivo de configuración de babel en la raíz del proyecto con el siguiente nombre: **`.babelrc`**
- Crear un objeto indicando el preset que se utilizara.
```js
{ 
    "presets": ["@babel/preset-env"] 
}
```

---

### ¿Qué es Webpack?

Webpack es un empacador de módulos (module bundler) para JavaScript, que toma los archivos y dependencias de un proyecto y los transforma en un paquete optimizado.

---

### ¿Para qué se usa?

- **Empaquetar archivos:** Junta todos los archivos (.js, .css, imágenes, etc.) en un solo archivo o en varios optimizados.
- **Optimización del código:** Minimiza archivos, elimina código no usado (tree shaking) y divide el código en partes más pequeñas (code splitting).
- **Compatibilidad con Babel:** Permite transformar código moderno a versiones más antiguas con babel-loader. 
- **Servidor de desarrollo:** Con webpack-dev-server, puedes hacer recarga en vivo (hot reload).
- **Carga de recursos:** Webpack puede procesar archivos CSS, imágenes y fuentes mediante loaders.

---

### Instalación de webpack

```bash
npm i --save-dev webpack webpack-cli webpack-dev-server babel-loader
#webpack  -> El core de Webpack, usado para empaquetar archivos.
#webpack-cli  -> Proporciona comandos en la terminal para ejecutar Webpack.
#webpack-dev-server  -> Un servidor de desarrollo con recarga en vivo.
#babel-loader  -> Permite a Webpack procesar archivos JS con Babel.
```

---

### Configuración

- Crea un archivo **`webpack.config.js`** en la raíz del proyecto y agrégale lo siguiente:

```js
const path = require('path');

module.exports = {
  mode: 'development', // Modo desarrollo
  entry: './src/index.js', // Archivo de entrada
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: 'bundle.js', // Archivo de salida
  },
  devServer: {
    static: './dist', // Carpeta donde lanzara los archivos
    port: 3000, // Puerto del servidor de desarrollo
    open: true, // Abre el navegador automáticamente
    hot: true, // Habilita recarga en vivo
  },
  module: {
    rules: [
      {
        test: /\.js$/, // Archivos JS
        exclude: /node_modules/,
        use: {
          loader: 'babel-loader',
          options: {
            presets: ['@babel/preset-env']
          }
        }
      },
      {
        test: /\.scss$/, // Procesar archivos SCSS de Bootstrap
        use: ['style-loader', 'css-loader', 'sass-loader']
      },
      {
        test: /\.css$/, // Procesar archivos CSS
        use: ['style-loader', 'css-loader']
      },
      {
        test: /\.(woff|woff2|eot|ttf|otf|svg)$/, // Carga fuentes de FontAwesome
        type: 'asset/resource',
      }
    ]
  }
};
```

- Modifica el archivo package.json para añadir scripts:
```json
"scripts": {  
    "build": "webpack",  
    "dev": "webpack serve --open"
}
```

---

### Arbol de proyecto

```json
dist/
    |-- public/
    |     └-- css/
    |         └-- main.css
    |     └-- js/
    |         └-- index.js
    |-- bundle.js
    └-- index.html
src/
    └-- index.js
packge.json
.babelrc
webpack.config.js
```

---

### Iniciar proyecto modo desarrollo

Crear `index.js`:

```js
console.log('Entorno listo');
```

Iniciar proyecto en modo desarrollo

```bash
npm run dev
```

Crear proyecto final `bundle.js`

```bash
npm run build
```