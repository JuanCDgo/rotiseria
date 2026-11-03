# 🛠️ Manual Técnico - Rotisería & Minutas

Documentación técnica y arquitectura de la micro-aplicación web para rotiserías y locales gastronómicos.

---

## 🏗️ 1. Arquitectura y Stack Tecnológico

- **Frontend:** Single-Page Application (SPA) en Vanilla HTML5, CSS3 y JavaScript moderno (ES6+).
- **Estilos:** Tailwind CSS (vía CDN oficial de producción) con esquema de color gastronómico (`stone-900`, `amber-500`, `orange-600`).
- **Librería QR:** `qrcode.min.js` (local empaquetada + fallback CDN cdnjs + fallback redundante con QuickChart.io API).
- **Persistencia de Datos:** `LocalStorage` del navegador (cero dependencias de backend o bases de datos pesadas).
- **Hosting:** GitHub Pages con despliegue continuo desde rama `main`.
- **Integraciones:** WhatsApp Universal Link (`https://wa.me/...`) y Mercado Pago deeplink.

---

## 🗄️ 2. Modelo de Datos (LocalStorage)

Los datos se guardan bajo tres llaves principales:

### `rotiseria_settings_v1`
```json
{
  "businessName": "Rotisería · Don Pepe",
  "businessSubtitle": "Comidas caseras para llevar y delivery",
  "businessAddress": "Av. San Martín 1420",
  "whatsappPhone": "5491123456789",
  "paymentAlias": "rotiseria.donpepe.mp",
  "deliveryFee": 1500,
  "isOpen": true,
  "businessLogo": "data:image/jpeg;base64,...",
  "targetStamps": 6,
  "rewardDescription": "1 Pizza Muzzarella o 1 Docena de Empanadas Gratis"
}
```

### `rotiseria_dishes_v1`
Array de objetos para cada plato:
```json
[
  {
    "id": "pizz-01",
    "name": "Pizza Muzzarella",
    "category": "pizzas",
    "price": 8500,
    "description": "Salsa de tomate casera, muzzarella abundante, orégano y aceitunas.",
    "isAvailable": true
  }
]
```

### `rotiseria_empanadas_v1`
Variedades de empanadas con control de stock rápido:
```json
[
  { "id": "emp-01", "name": "Carne Suave", "available": true },
  { "id": "emp-02", "name": "Carne a Cuchillo", "available": true },
  { "id": "emp-03", "name": "Jamón y Queso", "available": true }
]
```

---

## ⚡ 3. Componentes Clave

### A. Doble Motor de Generación de QR
Para evitar pantallas o contenedores blancos si falla la carga del script `QRCode.js`:
```javascript
const qrPayload = 'https://mpago.la/transfer?alias=' + encodeURIComponent(alias);
let success = false;
try {
  if (typeof QRCode !== 'undefined') {
    new QRCode(container, {
      text: qrPayload,
      width: 170,
      height: 170,
      colorDark: '#1C1917',
      colorLight: '#ffffff',
      correctLevel: QRCode.CorrectLevel.M
    });
    success = true;
  }
} catch (err) {
  console.warn('QRCode JS error:', err);
}

if (!success) {
  container.innerHTML = '<img src="https://quickchart.io/qr?text=' + encodeURIComponent(qrPayload) + '&size=170" alt="QR Cobro" class="w-[170px] h-[170px] rounded-xl shadow-sm mx-auto block">';
}
```

### B. Compresión de Logo con HTML5 Canvas
Al subir una imagen de logo, se dibuja en un canvas off-screen y se exporta a JPEG con compresión `0.82`, limitando el tamaño a un máximo de 360x360 px. Esto garantiza que el tamaño en base64 no exceda los 50 KB, preservando la cuota de LocalStorage (5 MB disponibles).

### C. Smart Cap en Docena / Media Docena de Empanadas
El selector valida en tiempo real:
```javascript
const currentTotal = Object.values(selectedEmpanadas).reduce((a, b) => a + b, 0);
if (currentTotal >= targetLimit) {
  // Deshabilitar botones de incremento y mostrar animación de docena lista
}
```

---

## 🚀 4. Despliegue y Mantenimiento

1. **Repositorio:** `https://github.com/JuanCDgo/rotiseria`
2. **Branch de Producción:** `main`
3. **GitHub Pages URL:** `https://juancdgo.github.io/rotiseria/`
4. **Respaldo en la Nube:** Copia sincronizada en Google Drive en carpeta `Mini Apps / rotiseria`.
