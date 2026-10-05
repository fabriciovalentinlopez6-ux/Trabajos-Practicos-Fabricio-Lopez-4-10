# Manual

Matias campos, Fabricio lopez
link del video: https://youtu.be/uN19_H5n-Lo?si=QH4HUKnK5iCSZpuz

Los programas que necesitamos son:

- **Node.js**: Permite ejecutar JavaScript fuera del navegador para crear el servidor que recibe los datos y losguarda

- **DBeaver**: Programa para crear base de datos

- **Visual Studio Code**: Editor de texto html

Como se hace:

- **Paso 1 (Instalar programas)**: Instalar Node.js, DBeaver y Visual Studio Code. Para verificar Node, ejecutar `node -v` en la terminal.

- **Paso 2 (Crear carpeta)**: Crear carpeta `registro-alumnos` y abrirla en VS Code.

- **Paso 3 (Abrir terminal)**: Abrir la terminal en VS Code (Ctrl + ñ) o mediante cmd.

- **Paso 4 (Instalar librerías)**: Ejecutar `npm init -y` y luego `npm install express better-sqlite3`.

- **Paso 5 (Crear base de datos)**: En DBeaver, crear conexión a SQLite, indicar la ruta `alumnos.db`, abrir un Editor SQL y ejecutar el script de creación.

- **Paso 6 (Crear archivos)**: Crear la estructura con `server.js` en la raíz y la carpeta `formulario/` conteniendo `index.html` y `script.js`.

- **Paso 7 (Encender servidor)**: Ejecutar `node server.js` en la terminal.

- **Paso 8 (Abrir en el navegador)**: Ingresar a `http://localhost:3000`.

- `http://`: El protocolo de comunicación.

- `localhost`: Significa "esta misma computadora".

- `:3000`: El puerto configurado en el servidor.

- **Doble clic vs Localhost**: Hacer doble clic abre el HTML como un archivo aislado (`file:///`) sin servidor; usar `localhost:3000` permite la comunicación correcta con la base de datos.

- **Paso 9 (Probar y verificar)**: Registrar un usuario y actualizar la vista en DBeaver (F5).

En caso de experimentar problemas (como me paso) puede deberse a nombres de los archivos diferrentes con repecto al código o equivocarse al momento de los comandos en cmd

## Ejemplo de códigos:

### Base de datos básica:

```sql
CREATE TABLE alumnos (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  nombre TEXT,
  apellido TEXT,
  email TEXT,
  edad INTEGER,
  dni TEXT UNIQUE
);
```

### Servidor(que se creo como intermediario entre la pagina y la base de datos):

```javascript
const express = require('express');
const Database = require('better-sqlite3');

const app = express();
app.use(express.json());
app.use(express.static('formulario'));

const db = new Database('alumnos.db');

app.post('/alumnos', (req, res) => {
  const { nombre, apellido, email, edad, dni } = req.body;

  try {
    db.prepare(
      'INSERT INTO alumnos (nombre, apellido, email, edad, dni) VALUES (?, ?, ?, ?, ?)'
    ).run(nombre, apellido, email, edad, dni);

    res.json({ mensaje: 'Alumno guardado' });
  } catch (error) {
    res.status(400).json({ mensaje: 'No se pudo guardar (¿DNI repetido?)' });
  }
});

app.get('/alumnos', (req, res) => {
  const alumnos = db.prepare('SELECT * FROM alumnos').all();
  res.json(alumnos);
});

app.listen(3000, () => {
  console.log('Servidor listo en http://localhost:3000');
});
```

- `const express = require('express')`: Importa la librería Express.

- `const Database = require('better-sqlite3')`: Importa el conector de SQLite.

- `const app = express()`: Instancia la aplicación del servidor.

- `app.use(express.json())`: Habilita la lectura de formato JSON.

- `app.use(express.static('formulario'))`: Sirve los archivos estáticos de la carpeta formulario.

- `const db = new Database('alumnos.db')`: Conecta con el archivo de la base de datos.

- `app.post('/alumnos', ...)`: Ruta para recibir y guardar datos (INSERT).

- `const { nombre, ... } = req.body`: Extrae los datos recibidos del formulario.

- `try { ... } catch (error) { ... }`: Maneja errores para evitar la caída del servidor.

- `db.prepare(...).run(...)`: Ejecuta la inserción SQL de forma segura usando parámetros (?).

- `res.json(...)`: Envía la respuesta al cliente.

- `app.get('/alumnos', ...)`: Ruta para consultar y devolver todos los registros (SELECT).

- `db.prepare('SELECT * FROM alumnos').all()`: Obtiene todas las filas de la tabla.

- `app.listen(3000, ...)`: Inicia el servidor en el puerto 3000.

### Formulario html:

```html
<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8">
  <title>Registro de alumnos</title>
</head>
<body>
  <h1>Registro de alumnos</h1>

  <form id="formulario">
    <input id="nombre" placeholder="Nombre" required>
    <input id="apellido" placeholder="Apellido" required>
    <input id="email" type="email" placeholder="Email" required>
    <input id="edad" type="number" placeholder="Edad">
    <input id="dni" placeholder="DNI" required>
    <button type="submit">Registrar</button>
  </form>

  <p id="mensaje"></p>

  <h2>Alumnos registrados</h2>
  <table border="1">
    <tr>
      <th>Nombre</th><th>Apellido</th><th>Email</th><th>Edad</th><th>DNI</th>
    </tr>
    <tbody id="tabla"></tbody>
  </table>

  <script src="script.js"></script>
</body>
</html>
```

El script para hacer que funcione el formulario:

```javascript
const formulario = document.getElementById('formulario');

formulario.addEventListener('submit', async function (evento) {
  evento.preventDefault(); // evita que la página se recargue

  const alumno = {
    nombre: document.getElementById('nombre').value,
    apellido: document.getElementById('apellido').value,
    email: document.getElementById('email').value,
    edad: document.getElementById('edad').value,
    dni: document.getElementById('dni').value
  };

  const respuesta = await fetch('/alumnos', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(alumno)
  });

  const resultado = await respuesta.json();
  document.getElementById('mensaje').textContent = resultado.mensaje;

  formulario.reset();   // limpia el formulario
  mostrarAlumnos();     // actualiza la tabla
});
async function mostrarAlumnos() {
  const respuesta = await fetch('/alumnos');
  const alumnos = await respuesta.json();
  let filas = '';
  for (const a of alumnos) {
    filas += '<tr><td>' + a.nombre + '</td><td>' + a.apellido + '</td><td>' +
             a.email + '</td><td>' + a.edad + '</td><td>' + a.dni + '</td></tr>';
  }
  document.getElementById('tabla').innerHTML = filas;
}

mostrarAlumnos(); // se ejecuta al abrir la página
```

- `document.getElementById(...).value`: Obtiene el valor ingresado en un campo.

- `addEventListener('submit', ...)`: Escucha el evento de envío del formulario.

- `evento.preventDefault()`: Evita el recargado por defecto de la página.

- `fetch('/alumnos', ...)`: Realiza las peticiones HTTP al servidor.

- `method: 'POST'`: Especifica que el tipo de petición es para enviar datos.

- `JSON.stringify(alumno)`: Convierte un objeto JavaScript a texto JSON.

- `async / await`: Maneja las operaciones asíncronas para esperar la respuesta del servidor.

- `formulario.reset()`: Resetea los campos del formulario.

- `for (const a of alumnos)`: Recorre el listado de alumnos para generar el HTML.

- `innerHTML`: Inserta el marcado HTML generado dentro de la tabla.

- `mostrarAlumnos()`: Función que consulta y renderiza la lista de alumnos.
