# 🛠️ PLAN DE ADAPTACIÓN Y ESCALABILIDAD WEB (FRONTEND)
## Para el Desarrollador de la Página Web (Tú)
**Proyecto:** Adaptación Dinámica de [prototipomig.html](file:///c:/Users/Grupo%20UNIFORTEX/Desktop/Nueva%20carpeta/prototipomig.html) para Conexión con el Sistema de Gestión del Compañero  
**Fecha de Elaboración:** Septiembre 2026  
**Meta:** Desacoplar todos los datos que hoy están escritos dentro del código HTML/JS (productos, fotos, precios, stock, ofertas y datos de contacto) para que la página los consuma automáticamente desde el Sistema de Gestión, **conservando el 100% de las funciones actuales, estética visual y rendimiento fluido**.

---

## 1. PRINCIPIO FUNDAMENTAL: ARQUITECTURA "ZERO DOWNTIME" (FALLBACK HÍBRIDO)

Para que tu trabajo nunca dependa de que el servidor de tu compañero esté encendido, apagado o en pruebas:

> **Regla de Oro:** La web tendrá una **Estrategia Híbrida (Offline-First)**:
> 1. Al cargar la página, se comprueba si hay datos guardados en la memoria del navegador (`localStorage`). Si existen, se cargan de inmediato (0 milisegundos de espera).
> 2. En segundo plano, se hace una consulta asíncrona (`fetch`) al Sistema de Gestión del compañero.
> 3. Si el Sistema de Gestión responde: se actualizan los productos, precios, fotos y stock en pantalla y se actualiza la memoria local.
> 4. Si el Sistema de Gestión NO responde (porque está apagado, sin internet o en mantenimiento): la página utiliza los datos de respaldo locales (`FALLBACK_PRODUCTS`). **La web NUNCA se quedará en blanco ni mostrará pantallas de error.**

---

## 2. MAPA DE ELEMENTOS A DESACOPLAR EN EL CÓDIGO ACTUAL

Hoy en día, para hacer un cambio debes editar el archivo HTML directamente. Con esta adaptación, todos estos elementos se gobernarán desde el panel de tu compañero:

| Elemento en la Web | Ubicación Actual en el Código | Cómo se Gobernará desde el ERP |
| :--- | :--- | :--- |
| **Catálogo de 60 Productos** | `const PRODUCTS = [...]` (Línea ~7,568) | Se descargará vía `GET /api/v1/productos` |
| **Fotos de Productos** | Atributo `img` o emojis en `PRODUCTS` | Se cargará la URL de la foto subida desde el ERP |
| **Control de Stock** | No existe indicador en tiempo real | Se mostrarán badges de "Disponible", "Pocas Uds" o "Agotado" |
| **Precios Detal y Mayor** | Atributo `price` en `PRODUCTS` | Se actualizarán en la base de datos y se reflejarán al instante |
| **Tallas Disponibles** | Atributo `sizes` en `PRODUCTS` | El modal de tallas tomará las tallas dinámicamente |
| **Teléfonos y WhatsApp** | Hardcodeado `'584144938316'` en varias funciones | Se centralizará en una variable `CONFIG_EMPRESA` |
| **Promociones y Paquetes** | Escrito en HTML en `promosTrack` y `paquetesTrack` | Se renderizará con una plantilla JS desde la API |
| **Recepción de Pedidos** | Solo envía texto por WhatsApp | Guardará la cotización en la base de datos del ERP + WhatsApp |

---

## 3. PASO A PASO TÉCNICO DE LA ADAPTACIÓN

### PASO 1: Crear el Módulo Conector API (`apiClient`)
En la sección de scripts de [prototipomig.html](file:///c:/Users/Grupo%20UNIFORTEX/Desktop/Nueva%20carpeta/prototipomig.html) (o en un archivo separado `api.js`), se define el cliente de conexión con la dirección IP o dominio del servidor de tu compañero:

```javascript
// Configuración de la URL del servidor Laravel del compañero (ajustable fácilmente)
const API_CONFIG = {
    BASE_URL: 'http://127.0.0.1:8000/api/v1', // Servidor Laravel local (php artisan serve) o 'https://erp.tuempresa.com/api/v1'
    TIMEOUT_MS: 4000,
    CACHE_KEY_PROD: 'unifortex_cache_productos',
    CACHE_KEY_CONFIG: 'unifortex_cache_config',
    CACHE_KEY_PAQ: 'unifortex_cache_paquetes'
};

// Función auxiliar para peticiones con límite de tiempo
async function fetchConTimeout(url, opciones = {}) {
    const controller = new AbortController();
    const timeout = setTimeout(() => controller.abort(), API_CONFIG.TIMEOUT_MS);
    try {
        const respuesta = await fetch(url, { ...opciones, signal: controller.signal });
        clearTimeout(timeout);
        return respuesta;
    } catch (err) {
        clearTimeout(timeout);
        throw err;
    }
}
```

---

### PASO 2: Centralización de Datos de Contacto y WhatsApp
Reemplazar los números fijos por un objeto dinámico que se actualiza desde el backend:

```javascript
// Valores por defecto de respaldo (si el backend no responde, se usan estos)
let CONFIG_EMPRESA = {
    telefonoWhatsApp: '584144938316',
    telefonoLlamadas: '+58 (414) 493-8316',
    correoVentas: 'ventas@unifortex.com',
    direccion: 'Valencia, Carabobo, Venezuela',
    descuentoMayorista: 18,
    pedidoMinimoMayor: 12
};

async function sincronizarConfiguracionEmpresa() {
    try {
        const res = await fetchConTimeout(`${API_CONFIG.BASE_URL}/configuracion`);
        if (res.ok) {
            const json = await res.json();
            if (json.success && json.data) {
                CONFIG_EMPRESA = { ...CONFIG_EMPRESA, ...json.data };
                localStorage.setItem(API_CONFIG.CACHE_KEY_CONFIG, JSON.stringify(CONFIG_EMPRESA));
                actualizarTextosContactoEnDOM();
            }
        }
    } catch (e) {
        console.warn('⚠️ No se pudo conectar con el ERP para datos de empresa. Usando caché local.');
        const cached = localStorage.getItem(API_CONFIG.CACHE_KEY_CONFIG);
        if (cached) CONFIG_EMPRESA = JSON.parse(cached);
    }
}

function actualizarTextosContactoEnDOM() {
    // Actualiza automáticamente teléfonos y correos en header, footer y enlaces de WhatsApp
    document.querySelectorAll('.dinamico-telefono').forEach(el => el.textContent = CONFIG_EMPRESA.telefonoLlamadas);
    document.querySelectorAll('.dinamico-correo').forEach(el => el.textContent = CONFIG_EMPRESA.correoVentas);
}
```

---

### PASO 3: Catálogo Dinámico con Stock e Imágenes
Transformar la variable `PRODUCTS` para que se alimente desde la base de datos del compañero:

```javascript
let DYNAMIC_PRODUCTS = [...PRODUCTS]; // Inicia con los 60 productos actuales de respaldo

async function sincronizarCatalogoProductos() {
    try {
        const res = await fetchConTimeout(`${API_CONFIG.BASE_URL}/productos`);
        if (res.ok) {
            const json = await res.json();
            if (json.success && Array.isArray(json.data) && json.data.length > 0) {
                DYNAMIC_PRODUCTS = json.data;
                localStorage.setItem(API_CONFIG.CACHE_KEY_PROD, JSON.stringify(DYNAMIC_PRODUCTS));
                
                // Re-renderizar catálogo y productos estrella con los datos frescos
                renderCatalog();
                renderStarProducts('todos');
                mostrarToast('🔄 Catálogo e inventario sincronizados en tiempo real.');
            }
        }
    } catch (e) {
        console.warn('⚠️ Usando catálogo local/caché por falta de conexión al ERP.');
        const cached = localStorage.getItem(API_CONFIG.CACHE_KEY_PROD);
        if (cached) {
            DYNAMIC_PRODUCTS = JSON.parse(cached);
            renderCatalog();
            renderStarProducts('todos');
        }
    }
}
```

#### Adaptación en la Tarjeta del Producto para Stock y Fotos:
En la función `renderCatalog()`, incorporar el indicador visual de inventario:

```javascript
// Dentro del mapeo de productos en renderCatalog:
const stockDisponible = p.stock !== undefined ? p.stock : 50;
let stockBadge = '';
let btnDisabled = '';

if (stockDisponible <= 0) {
    stockBadge = '<span class="badge-stock agotado">❌ Agotado</span>';
    btnDisabled = 'disabled style="opacity:0.5;cursor:not-allowed;" title="Producto sin stock"';
} else if (stockDisponible <= 12) {
    stockBadge = `<span class="badge-stock pocas">⚠️ Últimas ${stockDisponible} uds</span>`;
} else {
    stockBadge = `<span class="badge-stock disponible">✅ Disponible (${stockDisponible} uds)</span>`;
}
```

---

### PASO 4: Renderizado Dinámico de Paquetes y Promociones
Actualmente las tarjetas de paquetes están escritas en código HTML fijo dentro de `#promosTrack` y `#paquetesTrack`.  
Al convertirlas a renderizado dinámico por JavaScript:
1. Se crea la función `renderPaquetesDinamicos(paquetesArray)`.
2. Se inyectan las tarjetas con las ofertas vigentes que tu compañero marque como `activo: true` en su sistema de gestión.
3. **Se ejecuta inmediatamente `setupInfiniteTrack()` e `initBandaAutoScroll()`**, de modo que el bucle infinito y la fluidez sigan funcionando a la perfección sin importar si hay 3, 7 o 15 ofertas configuradas.

---

### PASO 5: Envío de Órdenes de Compra y Cotizaciones al ERP
Cuando el cliente pulse **"Generar Orden de Compra"** o **"Enviar por WhatsApp"**, la web hará dos cosas simultáneas:
1. **Envío silencioso al ERP**: Envía el JSON con los productos, cantidades, tallas y datos del cliente mediante un `POST /api/v1/pedidos`. A tu compañero le aparecerá una alerta en su panel de administración: *"Nueva Cotización Web recibida"*.
2. **Apertura de WhatsApp**: Abre la conversación con el mensaje detallado exactamente como lo hace ahora.

```javascript
async function registrarPedidoEnERP(datosPedido) {
    try {
        await fetch(`${API_CONFIG.BASE_URL}/pedidos`, {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify(datosPedido)
        });
        console.log('✅ Cotización guardada con éxito en el Sistema de Gestión.');
    } catch (e) {
        console.warn('⚠️ No se pudo registrar en la base de datos del ERP, pero la cotización se enviará por WhatsApp.');
    }
}
```

---

## 4. CRONOGRAMA DE IMPLEMENTACIÓN RECOMENDADO

Para avanzar de forma ordenada y sin arriesgar nada de lo construido:

* **Semana 1: Establecer Contrato de API y Pruebas Locales**:
  * Tu compañero levanta su backend con los endpoints básicos (`GET /productos` y `GET /configuracion`).
  * Prueban las respuestas en Postman o en el navegador.
* **Semana 2: Integración de Catálogo y Datos de Contacto**:
  * Se implementa la carga asíncrona de `CONFIG_EMPRESA` y `DYNAMIC_PRODUCTS`.
  * Verifican que al cambiar un precio o un stock en el panel de tu compañero, al refrescar la web se vea el cambio.
* **Semana 3: Subida de Imágenes y Fichas Técnicas**:
  * Tu compañero habilita la subida de fotos en su sistema.
  * La web recibe las URLs de las fotos y las renderiza con respaldo en emojis si falta alguna imagen.
* **Semana 4: Paquetes y Sincronización de Órdenes**:
  * Se conectan los paquetes y ofertas.
  * Se prueba el envío de cotizaciones desde el carrito y tabla de órdenes directamente a la bandeja del ERP.

---

## 5. BENEFICIOS INMEDIATOS DE ESTA ARQUITECTURA

1. **Autonomía Total**: Ya no tendrás que abrir el archivo de código de más de 12,000 líneas cada vez que se cree una camisa nueva, cambie un precio o se agote una bota.
2. **Cero Errores de Sintaxis**: Se evitan roturas accidentales de etiquetas HTML o llaves JS por cambios manuales cotidianos.
3. **Escalabilidad**: El catálogo puede crecer de 60 a 500 productos sin que el archivo HTML se vuelva más pesado.
4. **Resiliencia**: Si el servidor del compañero está apagado, la web sigue funcionando al 100% con los datos de respaldo.

---
*Documento preparado como guía paso a paso para la adaptación del frontend web.*
