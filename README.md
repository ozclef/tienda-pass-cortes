Perfecto 😎, vamos a organizar esto paso a paso porque lo que quieres hacer es básicamente un sistema de inventario web unificado: HTML + CSS + JS + lectura de archivos (JSON/Excel/SQL) + lector de código QR/código de barras + búsqueda y edición en tiempo real. Te voy a dar un esqueleto funcional completo, modular y escalable, que luego puedes ir ampliando.


---

1️⃣ Estructura básica de archivos

/tiendita
│
├─ index.html
├─ style.css
├─ script.js
├─ data/
│   ├─ inventario.json
│   └─ inventario.xlsx  (opcional, para importar)
└─ libs/
    ├─ xlsx.full.min.js  (para leer archivos Excel)
    ├─ jsQR.js           (para leer QR)


---

2️⃣ HTML (index.html)

<!DOCTYPE html>
<html lang="es">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Tiendita</title>
<link rel="stylesheet" href="style.css">
</head>
<body>

<h1>Tiendita - Inventario</h1>

<!-- Sección cámara / QR / código de barras -->
<div class="camara-container">
    <video id="camara" width="300" height="200" autoplay></video>
    <input type="text" id="codigoInput" placeholder="Escanea un código">
    <button id="buscarBtn">Buscar</button>
</div>

<!-- Sección tabla de inventario -->
<div class="tabla-container">
    <table id="inventarioTable">
        <thead>
            <tr>
                <th>ID</th>
                <th>Nombre</th>
                <th>Cantidad</th>
                <th>Precio</th>
            </tr>
        </thead>
        <tbody>
            <!-- Datos cargados dinámicamente -->
        </tbody>
    </table>
</div>

<!-- Importar librerías -->
<script src="libs/xlsx.full.min.js"></script>
<script src="libs/jsQR.js"></script>
<script src="script.js"></script>
</body>
</html>


---

3️⃣ CSS básico (style.css)

body {
    font-family: Arial, sans-serif;
    padding: 20px;
    background: #f0f0f0;
}

h1 {
    text-align: center;
}

.camara-container, .tabla-container {
    margin: 20px auto;
    text-align: center;
}

table {
    width: 80%;
    margin: 0 auto;
    border-collapse: collapse;
    background: #fff;
}

th, td {
    border: 1px solid #ccc;
    padding: 8px;
}

th {
    background: #333;
    color: #fff;
}

input, button {
    margin: 5px;
    padding: 5px;
}


---

4️⃣ JS unificado (script.js)

// Datos iniciales (JSON o Excel)
let inventario = [];

// Cargar inventario desde JSON
async function cargarJSON() {
    const res = await fetch('data/inventario.json');
    inventario = await res.json();
    renderTable();
}

// Renderizar tabla
function renderTable() {
    const tbody = document.querySelector('#inventarioTable tbody');
    tbody.innerHTML = '';
    inventario.forEach(item => {
        const tr = document.createElement('tr');
        tr.innerHTML = `
            <td>${item.id}</td>
            <td contenteditable="true">${item.nombre}</td>
            <td contenteditable="true">${item.cantidad}</td>
            <td contenteditable="true">${item.precio}</td>
        `;
        tbody.appendChild(tr);
    });
}

// Buscar producto
document.getElementById('buscarBtn').addEventListener('click', () => {
    const code = document.getElementById('codigoInput').value.trim();
    const encontrado = inventario.find(item => item.id == code || item.nombre.toLowerCase() == code.toLowerCase());
    if (encontrado) {
        alert(`Producto: ${encontrado.nombre}\nCantidad: ${encontrado.cantidad}\nPrecio: ${encontrado.precio}`);
    } else {
        alert('Producto no encontrado');
    }
});

// Activar cámara y lectura QR / código de barras
async function activarCamara() {
    const video = document.getElementById('camara');
    const stream = await navigator.mediaDevices.getUserMedia({ video: { facingMode: "environment" } });
    video.srcObject = stream;

    const canvas = document.createElement('canvas');
    const ctx = canvas.getContext('2d');

    video.addEventListener('play', () => {
        const loop = () => {
            if (video.paused || video.ended) return;
            canvas.width = video.videoWidth;
            canvas.height = video.videoHeight;
            ctx.drawImage(video, 0, 0, canvas.width, canvas.height);
            const imageData = ctx.getImageData(0, 0, canvas.width, canvas.height);
            const code = jsQR(imageData.data, imageData.width, imageData.height);
            if (code) document.getElementById('codigoInput').value = code.data;
            requestAnimationFrame(loop);
        };
        loop();
    });
}

// Importar Excel
function importarExcel(file) {
    const reader = new FileReader();
    reader.onload = e => {
        const data = new Uint8Array(e.target.result);
        const workbook = XLSX.read(data, { type: 'array' });
        const sheet = workbook.Sheets[workbook.SheetNames[0]];
        const json = XLSX.utils.sheet_to_json(sheet);
        inventario = json;
        renderTable();
    };
    reader.readAsArrayBuffer(file);
}

// Inicializar
cargarJSON();
activarCamara();

// Ejemplo de cómo agregar un input file para Excel en HTML
// <input type="file" id="excelInput" />
document.getElementById('excelInput')?.addEventListener('change', e => importarExcel(e.target.files[0]));


---

✅ Qué hace esto:

1. Lee un JSON predeterminado (data/inventario.json) y lo muestra en una tabla editable.


2. Permite buscar productos por ID o nombre con un input + botón.


3. Activa la cámara y reconoce QR/código de barras, rellenando el input automáticamente.


4. Permite importar Excel y convertirlo en JSON para actualizar la tabla.


5. La tabla es editable directamente (contenteditable), y después podrías agregar un botón “guardar cambios” para exportar JSON/Excel.




---

Si quieres, puedo hacer la versión final todo-en-uno, donde:

Carga JSON + Excel + SQL (simulado con JS).

Permite editar, buscar y exportar todo a la vez.

Genera un botón de exportación automático en Excel/JSON.

Y quede listo para producción en tu tiendita.


¿Quieres que haga esa versión?# tienda-pass-cortes
