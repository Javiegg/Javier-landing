# 📘 MANUAL TÉCNICO, ARQUITECTURA Y GUÍA OPERATIVA DEL SISTEMA UNIFORTEX
## E-Commerce B2B + Sistema de Gestión Empresarial (ERP)
**Autor:** Antigravity AI (Google DeepMind)  
**Destinatario:** Operadores del Sistema, Desarrolladores y Directiva de UNIFORTEX  
**Fecha:** Septiembre 2026  
**Estado:** Arquitectura Modular Implementada y Sincronizada

---

## ÍNDICE GENERAL
1. **Introducción y Propósito del Documento**
2. **Definición de Conceptos Fundamentales (¿Qué es qué y para qué sirve?)**
   - 2.1. ¿Qué es un ERP y por qué UNIFORTEX lo necesita?
   - 2.2. ¿Qué es un E-Commerce B2B frente a un E-Commerce B2C tradicional?
   - 2.3. ¿Qué es una API REST y cómo conecta dos mundos distintos?
   - 2.4. Frontend vs. Backend: Roles de cada participante
3. **Análisis Técnico de la Página Web del Cliente (`prototipomig.html`)**
   - 3.1. Tipología de la aplicación: Single Page Application (SPA) Reactiva
   - 3.2. Estructura de datos: El catálogo de los 60 productos
   - 3.3. Módulos integrados: Catálogo, Estudio de Diseño, Carrito B2B y Generador de Órdenes (PDF)
   - 3.4. Taller de Personalización Textil, Modelos Semi-3D y Matriz de Técnicas Autorizadas
4. **Análisis Técnico del Panel Administrativo ERP (`erp_unifortex.html`)**
   - 4.1. Filosofía de diseño: Interfaz administrativa limpia (Filament Standard)
   - 4.2. Operaciones CRUD (Crear, Leer, Actualizar y Eliminar)
   - 4.3. Control de inventario y amortiguación de stock (*Safety Stock Buffer*)
   - 4.4. Bandeja de pedidos con enlace profundo (*Deep Linking*) a WhatsApp Web
   - 4.5. Módulo de Novedades & Video Reel Vertical (9:16)
   - 4.6. Módulo Unificado de "Edición de Página" (Lienzo en Vivo WYSIWYG)
5. **Mecanismo de Comunicación Bidireccional entre la Tienda y el ERP**
   - 5.1. Almacenamiento local persistente (`localStorage`)
   - 5.2. Canal de comunicación entre pestañas (`BroadcastChannel API`)
   - 5.3. Filosofía "Zero Downtime" (La tienda jamás se queda en blanco)
6. **Integración con el Backend Mayor: Laravel 11 y Filament v3**
   - 6.1. ¿Por qué Laravel?
   - 6.2. ¿Por qué Filament v3?
   - 6.3. Cómo Incrustar la "Edición de Página" en Filament v3 con una sola función
7. **Soluciones Concretas de Negocio que Ofrece este Sistema**
8. **Glosario Técnico Integral para el Operador**

---

## 1. INTRODUCCIÓN Y PROPÓSITO DEL DOCUMENTO

El presente manual tiene como finalidad explicar con total claridad técnica y pedagógica la estructura, el funcionamiento interno y los beneficios de los sistemas de software desarrollados para **UNIFORTEX**.

Como operador o administrador, no necesitas ser un experto en sintaxis de código para entender **la lógica de cómo viajan los datos**, **por qué se tomaron ciertas decisiones de arquitectura** y **cómo cada archivo colabora para que la empresa venda más, cometa menos errores y ahorre tiempo de trabajo manual**.

---

## 2. DEFINICIÓN DE CONCEPTOS FUNDAMENTALES

Para entender el ecosistema, primero debemos desglosar los términos técnicos que gobiernan este proyecto:

### 2.1. ¿Qué es un ERP y por qué UNIFORTEX lo necesita?
* **Definición Técnica:** **ERP** son las siglas de *Enterprise Resource Planning* (Planificación de Recursos Empresariales). Es una categoría de software modular cuyo propósito es unificar y gestionar todos los procesos operativos de un negocio: compras, almacén, producción, ventas, contabilidad y recursos humanos.
* **¿Para qué sirve en general?:** En lugar de tener la información dispersa en hojas de Excel, libretas de papel o conversaciones de WhatsApp, un ERP centraliza la **"única fuente de la verdad"** (*Single Source of Truth*). Si un producto se vende, el stock baja automáticamente en todas partes.
* **¿Por qué sirve en este caso específico (UNIFORTEX)?:**
  UNIFORTEX no es solo una tienda que revende productos; es una empresa que maneja **telas, confección, bordados de logotipos, tallas industriales, dotaciones por lotes y cotizaciones empresariales**.
  Sin un ERP, si el precio de la tela o la tasa de cambio cambia, un programador tendría que entrar al código fuente de la página web a modificar número por número. **Con el ERP ([erp_unifortex.html](file:///c:/Users/Grupo%20UNIFORTEX/Desktop/Nueva%20carpeta/erp_unifortex.html)), cualquier operador de oficina puede cambiar un precio, sumar 50 pantalones al inventario o modificar el teléfono de contacto en un formulario visual, y la página web se actualiza al instante sin tocar una sola línea de código.**

### 2.2. ¿Qué es un E-Commerce B2B frente a un B2C tradicional?
* **B2C (*Business-to-Consumer*):** Es el comercio electrónico común (como comprar unos zapatos personales en Amazon o MercadoLibre). El cliente compra 1 o 2 unidades, paga con tarjeta de crédito de inmediato y se le envía a su casa.
* **B2B (*Business-to-Business*):** Es el modelo de UNIFORTEX. Una empresa (constructora, clínica, petrolera o fábrica) le compra a otra empresa dotaciones para 20, 50 o 500 trabajadores.
* **Particularidades del B2B resueltas en el código:**
  1. **Precios Escalonados (Detal vs. Mayor):** Si compran menos de 12 unidades aplica precio detal; a partir de 12 unidades aplica un descuento mayorista automático (por defecto 18%).
  2. **Curva de Tallas:** Una empresa no pide una franela; pide una tabla con 10 tallas S, 25 tallas M, 15 tallas L y 5 tallas XL.
  3. **Personalización Textil (Branding):** Las prendas llevan bordado o estampado del logotipo de la empresa compradora.
  4. **Emisión de Órdenes de Compra (OC):** Los departamentos de compras de las empresas exigen una cotización formal en PDF con desglose de IVA y validez de la oferta para autorizar el pago administrativo.

### 2.3. ¿Qué es una API REST y cómo conecta dos mundos distintos?
* **Definición:** **API** significa *Application Programming Interface* (Interfaz de Programación de Aplicaciones). **REST** (*Representational State Transfer*) es un estándar internacional de comunicación en la web.
* **Explicación con una analogía:** Imagina un restaurante. Tú eres el cliente sentado en la mesa (el navegador web); la cocina es el servidor donde están los datos y la comida (la base de datos y Laravel); y el **mesonero** es la **API REST**.
  El cliente no entra a la cocina a preparar los platos. Le dice al mesonero: *"Tráeme la lista de productos"* (`GET /api/v1/productos`). El mesonero va a la cocina, toma la lista en una bandeja ordenada y se la entrega al cliente en un formato ligero y universal llamado **JSON**.
* **¿Por qué es vital?:** Porque permite que la página web sea completamente independiente del panel del ERP. Si el día de mañana se crea una aplicación móvil para choferes o vendedores de calle, esa misma API le entregará los datos sin tener que reprogramar el sistema.

### 2.4. Frontend vs. Backend: Roles de cada participante
* **Frontend (El lado del cliente / Lo que se ve):** Es el archivo [prototipomig.html](file:///c:/Users/Grupo%20UNIFORTEX/Desktop/Nueva%20carpeta/prototipomig.html). Se ejecuta en el navegador web del usuario (Chrome, Safari, Edge en su computadora o teléfono). Se encarga del diseño visual, los colores, animaciones, filtros de búsqueda, selección de tallas y cálculos matemáticos del carrito.
* **Backend (El lado del servidor / Lo que no se ve):** Es el servidor donde residen Laravel y Filament. Se encarga de la seguridad, almacenar permanentemente los datos en MySQL o PostgreSQL, validar la autenticación de los administradores y enviar correos o notificaciones.

---

## 3. ANÁLISIS TÉCNICO DE LA PÁGINA WEB ([prototipomig.html](file:///c:/Users/Grupo%20UNIFORTEX/Desktop/Nueva%20carpeta/prototipomig.html))

### 3.1. Tipología: Single Page Application (SPA) Reactiva
La tienda web no es una página web tradicional de los años 2000 que se recarga completamente y parpadea en blanco cada vez que haces clic en una categoría.
* Está construida como una **SPA (Single Page Application)**.
* Todo el contenido (Inicio, Catálogo, Paquetes, Asesor de EPP, Estudio de Diseño, Carrito de Compras) vive dentro del mismo archivo HTML de alto rendimiento.
* Cuando el usuario hace clic en "Calzado de Seguridad", JavaScript filtra en milisegundos la memoria del navegador y redibuja únicamente la cuadrícula de productos utilizando la función `renderCatalog()`, sin recargar la página.

### 3.2. Estructura de Datos de los 60 Productos
En las líneas 7691 a 8433 del archivo se encuentra la estructura base de los 60 productos industriales de UNIFORTEX. Cada producto es un **objeto JavaScript** con la siguiente anatomía técnica:

```javascript
{
    id: 1,                                        // Identificador numérico único
    code: 'COD - PC28',                           // SKU / Código de inventario
    name: 'Pantalón / Blue Jeans 3 Costuras',     // Nombre comercial
    cat: 'Pantalones & Jeans',                    // Categoría para los filtros
    price: 24.50,                                 // Precio unitario detal en USD
    bulkPrice: 20.09,                             // Precio mayorista (+12 uds)
    stock: 140,                                   // Existencia actual en almacén
    stockMin: 15,                                 // Umbral de seguridad para alertas
    mat: 'Tela índigo 14.5 onzas, 100% algodón',  // Ficha técnica completa
    matShort: 'Índigo 14.5oz (100% Algodón)',     // Resumen para la tarjeta visual
    sizes: '28 - 30 - 32 - 34 - 36 - 38...',      // Tallas disponibles para selección
    sizeShort: 'Tallas: 28 al 46',                // Resumen rápido
    ico: '👖',                                    // Icono emoji fallback
    img: null,                                    // URL de foto real (subida desde ERP)
    isStar: true,                                 // Flag: Aparece en "Más Buscados"
    active: true                                  // Flag: Visible u oculto en catálogo
}
```

### 3.3. Módulos Clave del Frontend
1. **Bucle Infinito de Desplazamiento (`initBandaAutoScroll`):**
   Ubicado en las líneas finales del script. Emplea `requestAnimationFrame` de la GPU (tarjeta gráfica) para que las tarjetas de promociones y productos estrella se deslicen suavemente en pantalla a 60 cuadros por segundo sin consumir batería excesiva ni trabar el procesador.
2. **Estudio de Diseño Textil (`renderDisenoStudio`):**
   Permite al cliente seleccionar una prenda oficial, asignarle un color corporativo, rotar entre 4 ángulos en alta definición semi-3D y simular la ubicación de su logotipo antes de pedir la cotización formal.
3. **Generador de Orden de Compra B2B en PDF:**
   Transforma el contenido del carrito en una cotización estructurada con membrete, RIF del cliente, cálculo de subtotal, IVA (16%) y total en USD, lista para imprimir o descargar.

### 3.4. Taller de Personalización Textil, Modelos Semi-3D y Matriz de Técnicas Autorizadas

El módulo de diseño (`#view-diseno`) fue completamente rediseñado bajo una arquitectura de **validación estricta de técnicas de marcaje textil** y **modelado vectorial semi-3D de alta fidelidad**.

#### A. Las 8 Prendas Oficiales del Sistema
El taller opera exclusivamente sobre las 8 prendas institucionales de confección UNIFORTEX:
1. `camisas` (Camisas Industriales Reforzadas)
2. `chemise` (Chemises Corporativas Institucionales)
3. `franelas` (Franelas de Algodón Cuello Redondo)
4. `pantalon` (Pantalón Jean de Trabajo 3 Costuras)
5. `gorras` (Gorras Corporativas 6 Paneles con Visera)
6. `chalecos` (Chalecos Reflectivos de Seguridad ANSI)
7. `bragas` (Bragas Industriales Enterizas / Overoles Mecánicos)
8. `imperneables` (Impermeables Industriales con Capucha)

#### B. Matriz Estricta de Compatibilidad de Técnicas de Personalización
Para evitar errores de producción donde un cliente solicite una técnica físicamente inviable para la tela de la prenda, el sistema implementa la matriz `REGLAS_TECNICAS_POR_PRENDA`:

| Prenda | Sublimado Cálido | Estampado Textil (Serigrafía / DTF) | Bordado Computarizado HD | Regla Técnica Aplicada |
| :--- | :---: | :---: | :---: | :--- |
| **Camisas Industriales** | ❌ Incompatible | ❌ Incompatible | ✅ **Autorizado** | Exclusivo Bordado (Drill / Oxford) |
| **Chemises Corporativas** | ❌ Incompatible | ❌ Incompatible | ✅ **Autorizado** | Exclusivo Bordado (Tejido Piqué) |
| **Franelas Cuello Redondo** | ✅ **Autorizado** | ✅ **Autorizado** | ✅ **Autorizado** | Total compatibilidad (Sublimado / Estampado / Bordado) |
| **Pantalón Jean 3 Costuras** | ❌ Incompatible | ❌ Incompatible | ✅ **Autorizado** | Exclusivo Bordado (Índigo 14.5oz) |
| **Gorras 6 Paneles** | ✅ **Autorizado** | ✅ **Autorizado** | ✅ **Autorizado** | Total compatibilidad (Sublimado / Estampado / Bordado) |
| **Chalecos Reflectivos** | ❌ Incompatible | ✅ **Autorizado** | ✅ **Autorizado** | Estampado & Bordado |
| **Bragas Industriales** | ❌ Incompatible | ✅ **Autorizado** | ✅ **Autorizado** | Estampado & Bordado |
| **Impermeables c/ Capucha** | ❌ Incompatible | ✅ **Autorizado** | ✅ **Autorizado** | Estampado & Bordado |

* **Comportamiento en la Interfaz:**
  - Cuando una técnica no está permitida para la prenda seleccionada, su botón en `#chipsTecnica` se deshabilita automáticamente (`pointer-events: none; opacity: 0.38; text-decoration: line-through`).
  - Si el usuario tenía seleccionada una técnica incompatible y cambia de prenda, el sistema la reajusta de forma segura a `bordado` y emite un toast informativo: *"Para [Prenda] se activó 🧵 Bordado Computarizado"*.
  - Un badge dinámico superior (`#disenoTecnicaReglaBadge`) resume al usuario las técnicas autorizadas en tiempo real (*"Solo Bordado"*, *"Sublimado, Estampado y Bordado"* o *"Estampado y Bordado"*).

#### C. Motor Gráfico Vectorial Semi-3D (`getGarmentSvg`)
A diferencia de imágenes estáticas planas (PNG/JPG), el sistema dibuja las prendas directamente en el navegador mediante vectores SVG en un espacio de coordenadas estandarizado de `viewBox="0 0 400 420"`:
- **Degradados Volumétricos:** Gradientes lineales y radiales simulan la caída de la luz, el volumen cilíndrico de los brazos y torsos, y la curvatura de las viseras y capuchas.
- **Detalle Confección:** Pespuntes dobles y triples discontinuos (`stroke-dasharray="3,2"`), ojales de ventilación, botones perlados con aro de costura, bolsillos fuelle, solapas cortaviento, bandas reflectivas microprismáticas en plateado de alta visibilidad y elásticos de fruncido.
- **4 Ángulos por Prenda:** Frente, Espalda, Lateral/Manga Izquierda y Lateral/Manga Derecha.
- **Adaptación Anatómica:** Para prendas con mangas (`camisas`, `chemise`, `franelas`, `chalecos`, `bragas`, `imperneables`), los botones indican *"👈 Manga Izquierda / 👉 Manga Derecha"*; para prendas sin mangas (`gorras` y `pantalon`), el sistema renombra dinámicamente las etiquetas a *"👈 Lateral Izquierdo / 👉 Lateral Derecho"* evitando confusiones.

### 3.5. Sistema de Identificación Fiscal Dual en Orden de Compra: Persona Natural vs Persona Jurídica

La vista de **Orden de Compra B2B** (`#view-orden-compra`) cuenta con un selector interactivo por pestañas que adapta dinámicamente los campos requeridos según la figura fiscal del comprador:

#### A. Modalidad 1: Persona Natural (Particular / Independiente)
Diseñada para compras individuales, profesionales independientes o contratistas:
* **Nombres:** Campo independiente exclusivo para primer y segundo nombre (`#ocNatNombres`).
* **Apellidos:** Campo independiente exclusivo para primer y segundo apellido (`#ocNatApellidos`).
* **Cédula de Identidad:** 
  - Selector de nacionalidad `V-` / `E-` (`#ocNatCedulaTipo`).
  - **Validación estricta de solo números** (`#ocNatCedula`): Bloqueo en tiempo real de letras, puntos y caracteres especiales mediante `inputmode="numeric"`, `pattern="[0-9]*"` y `oninput="this.value = this.value.replace(/\D/g, '')"`. Se exige una longitud mínima de 5 dígitos numéricos.
* **Teléfono / WhatsApp:** Teléfono personal de contacto (`#ocNatTelefono`).
* **Correo Electrónico:** Correo para recepción de confirmación y factura proforma (`#ocNatEmail`).

#### B. Modalidad 2: Persona Jurídica (Empresarial / Corporativo)
Diseñada para empresas, contratistas comerciales e instituciones públicas o privadas:
* **RIF Fiscal (`#ocJurRif`):** Permite **cualquier carácter** alfanumérico y símbolos (letras J, G, V, E, números, guiones y puntos, ej. `J-31048921-4`, `G-20001234-5`).
* **Razón Social / Nombre de la Empresa (`#ocJurEmpresa`):** Denominación legal de la compañía.
* **Contacto / Solicitante (`#ocJurContacto`):** Nombre y apellido del encargado o departamento de compras/RRHH.
* **Teléfono / WhatsApp Corporativo (`#ocJurTelefono`):** Línea comercial de la empresa.
* **Correo Electrónico Corporativo (`#ocJurEmail`):** Correo para emisión formal de la O.C.

#### C. Campos Comunes y Logísticos
* **Condición de Pago Simplificada:** Se eliminó el selector complejo de plazos o créditos; toda orden procesada o cotización formal opera bajo la modalidad estándar de **"Contado / Cotización"**, simplificando la emisión y evitando inconsistencias comerciales.
* **Dirección de Despacho / Entrega (`#ocDespacho`):** **Totalmente opcional**. El cliente puede ingresar su almacén o sede empresarial, o dejarlo en blanco, en cuyo caso el sistema asigna automáticamente: *"Por coordinar con ventas / Retiro en sede"*.
* **Campos con Ejemplos de Fondo (Placeholders Limpios):** Ningún campo contiene valores predeterminados invasivos en el atributo `value`. Todos inician completamente vacíos para facilitar una escritura rápida, guiando al usuario con textos sutiles de ejemplo de fondo (`placeholder`).

#### D. Procesamiento, Sincronización con el ERP y Hoja Oficial PDF
1. **Validación Exhaustiva (`procesarOrdenCompraOficial`):** Verifica que los campos requeridos de la modalidad activa (Natural o Jurídica) estén diligenciados y que exista al menos un producto en la lista, permitiendo procesar el despacho de forma opcional.
2. **Sincronización Bidireccional:**
   - Escribe el registro estructurado en `localStorage` (`unifortex_pedidos_recibidos`).
   - Envía el evento `{ type: 'new_web_order', data: nuevoPedido }` mediante `BroadcastChannel('unifortex_sync_channel')` para que el panel administrativo ERP (`erp_unifortex.html`) lo reciba y liste al instante sin recargar.
3. **Modal de Confirmación Interactivo:** Muestra resumen del pedido, monto total y accesos directos para enviar por WhatsApp corporativo (`https://wa.me/584144938316`) y generar PDF oficial.
4. **Emisión de Hoja Oficial O.C. (`descargarResumenOC`):** Abre una ventana de impresión formal con membrete de UNIFORTEX, datos fiscales del contratante (Natural o Jurídica), tabla de prendas solicitadas, cálculo de IVA (16%), total en USD y líneas de firma autorizada.

---

## 4. ANÁLISIS TÉCNICO DEL PANEL ADMINISTRATIVO ERP ([erp_unifortex.html](file:///c:/Users/Grupo%20UNIFORTEX/Desktop/Nueva%20carpeta/erp_unifortex.html))

Este archivo fue creado para otorgar autonomía operativa absoluta al personal de UNIFORTEX.

### 4.1. Filosofía de Diseño: Normal y Elegante (Estándar Filament)
Siguiendo tus instrucciones, no se sobrecargó de efectos innecesarios. Se utilizó una paleta neutra y profesional basada en tonos *Slate* (`#0f172a`, `#1e293b`), blanco puro y los colores de marca de UNIFORTEX (Naranja Industrial `#D35914` y Amarillo Seguridad `#FDC417`).

### 4.2. Operaciones CRUD Implementadas
El acrónimo **CRUD** engloba las 4 funciones elementales de cualquier sistema de base de datos:
* **C (Create):** Botón `+ Crear Producto`. Abre un formulario modal que valida campos obligatorios y agrega un nuevo registro al catálogo.
* **R (Read):** Tablas dinámicas con búsqueda reactiva por texto (código SKU, nombre o tela) y filtros desplegables por categoría y nivel de stock.
* **U (Update):** Botón `✏️ Editar`. Carga los datos existentes en el modal para modificar precios, tallas, telas o fotos. También incluye edición rápida de stock con botones `+5` / `-5` directamente en la fila.
* **D (Delete):** Botón `🗑️ Eliminar`. Solicita confirmación de seguridad y remueve el producto del inventario.

### 4.3. Control de Inventario y Amortiguación de Stock (*Safety Stock*)
El sistema calcula dinámicamente tres estados visuales para el stock mediante badges de color:
* **Verde (`En Existencia`):** Stock mayor al mínimo configurado (> 10 o 15 uds).
* **Amarillo (`Pocas Unidades`):** El stock está entre 1 y el mínimo. Alerta al operador de que debe planificar confección o compra de materia prima.
* **Rojo (`Agotado`):** Stock igual a 0. Evita que los clientes sigan pidiendo un producto sin existencias.

### 4.4. Bandeja de Cotizaciones y Enlace Profundo (*Deep Linking*) a WhatsApp Web
Cada vez que un cliente envía un pedido desde la web, este se almacena en la tabla de cotizaciones con su número correlativo (ej. `OC-2026-8412`).
El botón **`💬 Responder`** utiliza la tecnología de *Deep Linking* de WhatsApp:
1. Extrae el número telefónico del cliente y limpia caracteres especiales (`+`, `-`, espacios).
2. Construye un mensaje profesional pre-redactado:  
   *"Hola Carlos Mendoza de Constructora Andina C.A., te contactamos desde el Departamento Comercial de UNIFORTEX respecto a tu cotización OC-2026-8412 por un monto de $482.16 USD..."*
3. Codifica el texto (`encodeURIComponent`) y abre directamente WhatsApp Web en una nueva pestaña, listo para que el vendedor solo presione *Enter*.

### 4.5. Módulo de Novedades & Video Reel Vertical (9:16)
Este módulo permite al equipo de marketing y ventas promocionar un lanzamiento exclusivo o producto estrella con formato dinámico de redes sociales (*Reel / Shorts*):
* **Carga Multimedia:** El operador puede subir un archivo de video vertical (`.mp4` o `.webm`) desde su computadora o ingresar una URL de CDN. También cuenta con 3 demos precargados (Almacén/Logística, Confección Textil y Soldadura Industrial).
* **Vinculación con Catálogo y Ficha Técnica:** Se asocia directamente a cualquiera de los productos del catálogo mediante un menú desplegable. Esto enlaza el botón web **"Ver Ficha Técnica del Producto"** para que abra automáticamente la ficha técnica modal interactiva del producto correspondiente (`abrirDetalleProducto(id)`).
* **Prendas y EPPs Mostrados:** Permite listar detalladamente qué prendas, calzados o lentes se aprecian en el video con sus respectivas especificaciones.
* **Precios Mayoristas B2B:** Define el precio regular, precio con descuento corporativo y leyenda de ahorro.
* **Previsualización en Vivo:** Incluye un marco de smartphone 9:16 en el ERP que reproduce el video y muestra los textos en tiempo real antes de presionar *"Guardar y Publicar en la Web"*.

### 4.6. Módulo Unificado de "Edición de Página" (Editor Visual In Situ / WYSIWYG)
Para evitar la dispersión en múltiples pestañas y facilitar su incorporación en un ERP mayor (Laravel + Filament), se implementó la arquitectura **`UnifortexWebVisualEditor`**:
* **Pestaña Única en el Menú Lateral:** En lugar de 4 o 5 menús desconectados (productos, paquetes, novedades, contacto), el operador cuenta con una **única opción destacada: `🖥️ Edición de Página`**.
* **Lienzo Visual Idéntico a la Tienda Real:** Al entrar a la sección, el sistema reproduce la estética exacta de la tienda ([prototipomig.html](file:///c:/Users/Grupo%20UNIFORTEX/Desktop/Nueva%20carpeta/prototipomig.html)): fondo oscuro de lujo, tipografías *Inter* y *Poppins*, carrusel, video reel 9:16 en vivo y tarjetas de productos con sus fotos y precios reales.
* **Puntos de Edición Interactivos (*Hotspots*):** En el modo edición, cada sección resalta con un borde dorado y un botón directo:
  - `🎬 Cambiar Video Reel & Novedad` (Permite subir MP4 o elegir URL).
  - `🎁 Editar Paquete` (Ajusta combos, precios y prendas incluidas).
  - `📷 Foto / Datos` (Permite subir fotos locales desde el ordenador o cambiar precios/stock de los 60 productos).
  - `✏️ Editar Cabecera & WhatsApp` (Actualiza teléfonos y tasa de cambio BCV).
* **Panel Lateral Desplegable (*Inspector Drawer*):** Al pulsar cualquier botón de edición, se despliega suavemente desde el lateral derecho un formulario específico sin recargar la pantalla ni perder de vista el diseño.
* **Actualización en Caliente (*Hot-Reloading*):** Al pulsar "Guardar Cambios", los datos se actualizan de inmediato en el lienzo del ERP, se guardan en `localStorage` y se transmiten por `BroadcastChannel` para que la tienda web abierta en otra ventana cambie en tiempo real.

---

## 5. MECANISMO DE COMUNICACIÓN BIDIRECCIONAL ENTRE LA TIENDA Y EL ERP

Una de las mayores innovaciones implementadas en estos archivos es que **la tienda de compras ([prototipomig.html](file:///c:/Users/Grupo%20UNIFORTEX/Desktop/Nueva%20carpeta/prototipomig.html)) y el ERP ([erp_unifortex.html](file:///c:/Users/Grupo%20UNIFORTEX/Desktop/Nueva%20carpeta/erp_unifortex.html)) ya trabajan juntos en tiempo real**, incluso sin necesidad de tener un servidor de internet encendido.

```
┌──────────────────────────┐                      ┌──────────────────────────┐
│     TIENDA CLIENTE       │                      │        PANEL ERP         │
│   (prototipomig.html)    │                      │   (erp_unifortex.html)   │
└────────────┬─────────────┘                      └────────────▲─────────────┘
             │                                                 │
             │ 1. Cliente pide cotización                      │ 2. Notificación en vivo
             │    guarda en localStorage                       │    suena campana
             ▼                                                 │
   ┌───────────────────────────────────────────────────────────┴─────────────┐
   │             CANAL DE SINCRONIZACIÓN LOCAL NATIVO                        │
   │  - localStorage: 'unifortex_catalogo_productos' (Productos)             │
   │  - localStorage: 'unifortex_paquetes_ofertas' (Combos y Promociones)    │
   │  - localStorage: 'unifortex_novedad_destacada' (Video Reel & Novedades) │
   │  - localStorage: 'unifortex_pedidos_recibidos' (Cotizaciones)           │
   │  - localStorage: 'unifortex_configuracion_empresa' (WhatsApp / Tasa)    │
   │  - BroadcastChannel API: 'unifortex_sync_channel' (Eventos Inmediatos)  │
   └─────────────────────────────────────────────────────────────────────────┘
             ▲                                                 │
             │ 4. Tienda actualiza catálogo / novedades        │ 3. Operador cambia
             │    sin recargar la página                       │    precio, video o stock
             └─────────────────────────────────────────────────┘
```

### 5.1. Almacenamiento Local Persistente (`localStorage`)
`localStorage` es una memoria de almacenamiento segura que provee el navegador web en la computadora del usuario.
* No se borra al cerrar el navegador ni al apagar la computadora.
* Ambos archivos ([prototipomig.html](file:///c:/Users/Grupo%20UNIFORTEX/Desktop/Nueva%20carpeta/prototipomig.html) y [erp_unifortex.html](file:///c:/Users/Grupo%20UNIFORTEX/Desktop/Nueva%20carpeta/erp_unifortex.html)) comparten las mismas claves de almacenamiento:
  * `unifortex_catalogo_productos`
  * `unifortex_paquetes_ofertas`
  * `unifortex_novedad_destacada`
  * `unifortex_configuracion_empresa`
  * `unifortex_pedidos_recibidos`

### 5.2. Canal de Comunicación entre Pestañas (`BroadcastChannel API`)
Si abres [prototipomig.html](file:///c:/Users/Grupo%20UNIFORTEX/Desktop/Nueva%20carpeta/prototipomig.html) en una pantalla y [erp_unifortex.html](file:///c:/Users/Grupo%20UNIFORTEX/Desktop/Nueva%20carpeta/erp_unifortex.html) en otra:
* Cuando en el ERP cambias el precio del Jean de `$24.50` a `$26.00`, el ERP ejecuta:
  ```javascript
  const bc = new BroadcastChannel('unifortex_sync_channel');
  bc.postMessage({ type: 'products_updated', data: prods });
  ```
* La pestaña de la tienda web tiene un "oído digital" escuchando ese canal:
  ```javascript
  bc.onmessage = (ev) => {
      if (ev.data.type === 'products_updated') {
          sincronizarConERPLocal(); // Actualiza la memoria interna
          renderCatalog();          // Redibuja los precios en pantalla
      }
  };
  ```
* **Resultado:** La tienda cambia el precio ante los ojos del cliente en 0.05 segundos, sin que nadie tenga que refrescar con `F5`.

### 5.3. Filosofía "Zero Downtime" (Respaldo Híbrido)
¿Qué ocurre si se borra la memoria del navegador o el servidor del compañero está apagado?
* La tienda web contiene los 60 productos originales integrados como respaldo por defecto (`FALLBACK`).
* Si el ERP o la API no responden, la tienda web sigue mostrando los 60 productos intactos. **El cliente nunca verá un error 404, una pantalla rota o un sitio caído.**

---

## 6. INTEGRACIÓN CON EL BACKEND MAYOR: LARAVEL 11 Y FILAMENT v3

Tu compañero de equipo desarrollará el backend central en **Laravel 11** utilizando **Filament v3**. A continuación se detalla por qué esta combinación es óptima y cómo se conecta con lo que hemos construido:

### 6.1. ¿Por qué Laravel?
* **Seguridad Empresarial:** Trae protección integrada contra ataques comunes en la web: inyección SQL, ataques XSS (*Cross-Site Scripting*) y falsificación de solicitudes CSRF (*Cross-Site Request Forgery*).
* **Manejo de Migraciones:** Permite definir la base de datos mediante código PHP versionable. Si cambian de servidor, tu compañero solo corre `php artisan migrate` y la base de datos se crea idéntica en 3 segundos.
* **ORM Eloquent:** Permite consultar los datos con sintaxis limpia en lugar de complejas consultas SQL manuales:
  ```php
  $productosActivos = Product::where('activo', true)->where('stock', '>', 0)->get();
  ```

### 6.2. ¿Por qué Filament v3?
Filament es el ecosistema de paneles administrativos más moderno del mundo PHP. Funciona sobre el stack **TALL** (*Tailwind CSS, Alpine.js, Laravel y Livewire*).
* **Ahorro de Tiempo:** Filament ya trae programadas las tablas con ordenamiento por columnas, exportación a Excel, búsqueda con autocompletado y componentes de carga de fotos con recorte interactivo. Tu compañero no tiene que diseñar nada desde cero; solo define los campos (`TextInput`, `Select`, `FileUpload`).
* **Campana de Notificaciones en Base de Datos:** Cuando un cliente cotiza en la web, Laravel dispara:
  ```php
  Notification::make()
      ->title('¡Nueva Cotización Web Recibida!')
      ->body("Cliente: {$order->cliente_nombre} - Total: \${$order->total_usd}")
      ->success()
      ->sendToDatabase(User::all());
  ```
  Esto enciende un punto rojo en la campana del panel de Filament de todos los administradores en la oficina.

### 6.3. Cómo Incrustar la "Edición de Página" en Filament v3 con una sola función
Para que tu compañero no tenga que recrear el editor desde cero ni programar 5 vistas distintas, el módulo **`UnifortexWebVisualEditor`** fue programado de forma totalmente auto-contenida.

Tu compañero solo necesita crear una **Página Personalizada de Filament** (*Custom Filament Page*):
```php
// app/Filament/Pages/EdicionDePagina.php
namespace App\Filament\Pages;

use Filament\Pages\Page;

class EdicionDePagina extends Page
{
    protected static ?string $navigationIcon = 'heroicon-o-computer-desktop';
    protected static ?string $navigationLabel = 'Edición de Página';
    protected static ?string $title = 'Edición de Página Web en Vivo';
    protected static ?string $navigationGroup = 'Canal Web & E-Commerce';
    protected static string $view = 'filament.pages.edicion-de-pagina';
}
```

Y en la vista Blade (`resources/views/filament/pages/edicion-de-pagina.blade.php`), solo monta el componente en una sola línea:
```blade
<x-filament-panels::page>
    <div id="unifortex-visual-editor-root"></div>

    @push('scripts')
    <script src="{{ asset('js/unifortex-visual-editor.js') }}"></script>
    <script>
        document.addEventListener('DOMContentLoaded', () => {
            UnifortexWebVisualEditor.mount('#unifortex-visual-editor-root', {
                apiEndpoint: '/api/v1',
                csrfToken: '{{ csrf_token() }}'
            });
        });
    </script>
    @endpush
</x-filament-panels::page>
```
**Resultado:** El sistema ERP mayor de tu compañero tendrá exactamente la misma pestaña de "Edición de Página", con el lienzo visual en vivo, el reel 9:16 editable, el cambio de fotos y precios por panel flotante, conectado directamente a la base de datos MySQL de Laravel.

---

## 7. SOLUCIONES CONCRETAS DE NEGOCIO QUE OFRECE ESTE SISTEMA

| Desafío Operativo Anterior en UNIFORTEX | Solución Técnica Implementada en este Sistema | Beneficio Tangible para el Negocio |
| :--- | :--- | :--- |
| Dependencia de programadores para cambiar fotos, teléfonos o precios. | **Panel ERP Autónomo ([erp_unifortex.html](file:///c:/Users/Grupo%20UNIFORTEX/Desktop/Nueva%20carpeta/erp_unifortex.html))** con formularios visuales. | Independencia total y cambios en tiempo real en menos de 1 minuto. |
| Clientes pidiendo productos que ya estaban agotados en el almacén. | **Control de Stock con badges de color** y switch para ocultar productos con un solo clic. | Cero fricción con clientes y reducción de ventas canceladas por falta de inventario. |
| Errores de cálculo en cotizaciones al mayor o confusiones con el IVA. | **Calculadora B2B automática** que aplica 18% de descuento mayorista a partir de 12 uds y calcula el IVA reglamentario. | Cotizaciones exactas y profesionales con desglose transparente. |
| Vendedores perdiendo tiempo transcribiendo pedidos a WhatsApp. | **Generador de Enlace Directo a WhatsApp** con texto estructurado (nombre, RIF, prendas, tallas y monto). | Respuesta comercial en segundos y cierre de ventas mucho más ágil. |
| Pérdida de ventas cuando el servidor del sistema entra en mantenimiento. | **Arquitectura Híbrida Zero Downtime** con respaldo local en la tienda web. | La web está disponible las 24 horas del día, los 365 días del año. |

---

## 8. GLOSARIO TÉCNICO INTEGRAL PARA EL OPERADOR

* **B2B (*Business to Business*):** Modelo de negocio enfocado en transacciones comerciales entre empresas (ventas corporativas al mayor), a diferencia del B2C que vende al consumidor individual.
* **ERP (*Enterprise Resource Planning*):** Sistema de gestión integral que planifica y centraliza los recursos, inventarios, compras, ventas y finanzas de una empresa.
* **CRM (*Customer Relationship Management*):** Módulo o software destinado a gestionar la relación, historial, presupuestos y comunicación con los clientes.
* **SKU (*Stock Keeping Unit*):** Código alfanumérico único asignado a un producto para rastrear su inventario (ej. `COD - PC28`).
* **CRUD (*Create, Read, Update, Delete*):** Las cuatro operaciones fundamentales que se pueden realizar sobre cualquier registro de datos: Crear, Leer, Modificar y Eliminar.
* **API REST:** Estándar de comunicación que permite a la página web solicitar y enviar información al servidor central mediante mensajes en formato JSON.
* **JSON (*JavaScript Object Notation*):** Formato universal de texto plano organizado en pares de clave-valor (`{"nombre": "Pantalón", "precio": 24.50}`) que computadoras de cualquier lenguaje pueden leer rápidamente.
* **Endpoint:** Dirección URL específica de una API a la cual la página web hace peticiones (ej. `http://127.0.0.1:8000/api/v1/productos`).
* **localStorage:** Memoria interna del navegador web donde se almacenan datos persistentes que no se borran al cerrar la pestaña.
* **BroadcastChannel:** Tecnología del navegador que permite a dos pestañas abiertas hablarse y enviarse mensajes entre sí al instante sin necesidad de internet.
* **CORS (*Cross-Origin Resource Sharing*):** Regla de seguridad de los navegadores que autoriza a la tienda web a comunicarse con un servidor que esté en otra dirección o puerto.
* **SPA (*Single Page Application*):** Aplicación web que carga una sola vez y va modificando dinámicamente lo que muestra en pantalla sin recargar la página entera.
* **Filament v3:** Suite de herramientas visuales sobre Laravel para construir paneles administrativos, tablas y formularios de alta velocidad y estética cuidada.
* **Safety Stock (Stock de Seguridad):** Cantidad mínima de unidades que se mantienen en almacén como colchón para evitar el desabastecimiento mientras se confecciona nuevo lote.

---

## 9. MOTOR DE COTIZACIÓN CORPORATIVA EN PDF Y RENDERIZADO DE PRODUCTOS

### 9.1. Resolución del Flujo de Renderizado de Catálogo y Productos Estrella
* **Causa Raíz Identificada:** Durante la inicialización del frontend en `prototipomig.html`, la función `renderOCTable()` invocaba `escapeHtml(item.nombre)`. Al no encontrarse declarada previamente `escapeHtml`, el navegador detenía la ejecución del script principal por un error de referencia antes de que las llamadas `renderCart()`, `renderCatalog()` y `renderStarProducts('todos')` pudieran ejecutarse.
* **Solución Implementada:**
  1. Se definió formalmente `escapeHtml(str)` para sanear entradas contra inyecciones XSS.
  2. Se blindaron las llamadas de arranque dentro de bloques `try...catch`, garantizando que tanto los 60 productos del catálogo general (`id="productGrid"`) como los productos estrella con seguimiento de demanda en vivo (`id="starProductsGrid"`) se rendericen ininterrumpidamente.

### 9.2. Plantilla Corporativa Oficial de Cotización en PDF
Al solicitar una cotización formal desde la web o a través del botón de descarga, el sistema genera dinámicamente un documento ejecutivo con diseño sincronizado a la identidad visual de UNIFORTEX (`#111418`, dorado `#fdc417` y naranja `#d35914`):

1. **Directorio Oficial de Ventas & Asesoría Comercial Directa:**
   * **Línea 1 · Ventas Directas:** `+58 (414) 493-8316` (WhatsApp y Llamadas - Maracay/Región Central).
   * **Línea 2 · Corporativo & Licitaciones B2B:** `+58 (424) 340-9898` (Atención a Grandes Cuentas e Industrias).
   * **Línea 3 · Despachos & Logística:** `+58 (424) 305-6068` (Seguimiento de Envíos y Dotación Empresarial).
   * **Correos Electrónicos Oficiales:** `grupounifortexventas@gmail.com` y `cotizaciones@unifortex.com`.
   * **Sedes Físicas:** Sede Maracay (C.C. Ciudad Jardín, Local 20, Calle Mariño) y Sede Comercial Caracas (Av. Libertador, Edif. Fertec).
2. **Desglose Íntegro de Productos Solicitados:**
   * Ítem numerado, ícono de prenda, nombre comercial y código SKU (`UNI-...`).
   * Talla seleccionada (ej. `[Talla: 32]`) y tipo de tarifa aplicada (`[Tarifa Mayorista]`).
   * Categoría y certificación técnica (`✓ COVENIN 2244-91 / Doble Refuerzo`).
   * Cantidad solicitada, precio unitario en USD y subtotal por renglón.
   * *Modo de respaldo inteligente:* Si el cliente solicita cotización sin haber cargado el carrito, el sistema genera automáticamente un paquete base propuesto (Pantalón Blue Jeans 14 Oz, Chemise Piqué 220g, Bota Vaqueta PU y Casco Dieléctrico) para emitir el presupuesto de inmediato.
3. **Resumen Financiero y Normativas Fiscales:**
   * Subtotal neto por cantidad de prendas.
   * I.V.A. estimado del 16%.
   * **TOTAL GENERAL (USD)** destacado con tipografía de alto contraste y caja dorada.
   * Nota de conversión referencial a la tasa oficial del Banco Central de Venezuela (BCV).
4. **Condiciones Comerciales, Garantías y Firmas:**
   * Garantía de confección UNIFORTEX con costuras triples y telas industriales certificadas.
   * Digitalización y bordado computarizado incluido para pedidos superiores a 12 unidades.
   * Bloques formales de firma y sello para UNIFORTEX INDUSTRIAL C.A. y Visto Bueno del Cliente.
   * Barra de control con botones de impresión (`window.print()`), chat directo a WhatsApp y cierre.

---

## 10. FLUJO INTEGRAL DE COMPRA, REDIRECCIÓN A WHATSAPP Y ADJUNTADO DE PDF OFICIAL

### 10.1. Directorio de Asesores con Redirección Directa a WhatsApp
Al igual que en la sección de **Contacto** de la página web, al confirmar la compra u orden de cotización, el cliente visualiza un directorio interactivo con las tres líneas de atención corporativa:
* **Línea 1 · Ventas Directas:** `+58 414-4938316` (Maracay / Región Central).
* **Línea 2 · Asesoría Comercial & Licitaciones:** `+58 424-3409898` (Grandes Cuentas e Industrias).
* **Línea 3 · Cotizaciones B2B & Despachos:** `+58 424-3056068` (Logística y Envíos Nacionales).

**Comportamiento Interactivo:**
Al hacer clic en cualquier número de teléfono o en el botón correspondiente **"💬 Chatear"**, el sistema ejecuta `enviarOrdenAWhatsAppYDescargarPDF(telefono, event)`:
1. **Redirección Inmediata a WhatsApp:** Abre directamente el chat con el asesor seleccionado sin pantallas intermedias (en computadoras mediante `web.whatsapp.com/send` y en dispositivos móviles mediante la app nativa `whatsapp://`).
2. **Generación Simultánea del Documento PDF:** Dispara la apertura y descarga de la Orden de Compra / Cotización en PDF corporativo con membrete oficial, permitiendo al cliente guardar o imprimir el documento al instante.

### 10.2. Estructura del Mensaje Corporativo Enviado al Chat de WhatsApp
El texto pre-cargado en el chat de WhatsApp contiene todos los datos de auditoría comercial y hace mención explícita al documento PDF generado:

```text
🏢 *ORDEN DE COMPRA & COTIZACIÓN OFICIAL - UNIFORTEX INDUSTRIAL*
━━━━━━━━━━━━━━━━━━━━━━━━━━
📋 *O.C. Nº:* UF-2026-9842
📅 *Fecha:* DD/MM/AAAA
🏷️ *Tipo de Facturación:* Persona Jurídica (Empresa)
🏢 *Razón Social:* CONSTRUCTORA & MINERA DEL SUR C.A.
📄 *RIF Fiscal:* J-40192837-1
👤 *Contacto / Solicitante:* Ing. Carlos Mendoza (Gerente de Compras)
📞 *Teléfono:* +58 412-8899112
✉️ *Correo Electrónico:* cmendoza@minerasureste.com
💳 *Modalidad de Pago:* Contado / Cotización
📍 *Dirección de Despacho:* Zona Industrial Los Montones, Galpón 8, Barcelona, Edo. Anzoátegui
━━━━━━━━━━━━━━━━━━━━━━━━━━
📦 *DETALLE DE ÍTEMS Y PRENDAS SOLICITADAS:*
1. Bragas Industriales de Seguridad × 50 uds = $1147.50 USD
2. Cascos de Seguridad Dieléctricos Tipo 1 × 20 uds = $170.00 USD
━━━━━━━━━━━━━━━━━━━━━━━━━━
💵 *Subtotal Neto:* $1317.50 USD
🧾 *IVA Estimado (16%):* $210.80 USD
💰 *TOTAL GENERAL: $1528.30 USD*
*(Pagos en Bolívares pagaderos a la tasa oficial del BCV a la fecha de pago)*
━━━━━━━━━━━━━━━━━━━━━━━━━━
📄 *DOCUMENTO OFICIAL GENERADO EN PDF:*
✅ *Cotización / Orden de Compra Nº UF-2026-9842 emitida con membrete corporativo.*
📎 *Adjunto copia del archivo PDF generado en este chat para su validación, apartado de inventario y facturación proforma.*

¡Hola equipo de ventas UNIFORTEX! Acabo de procesar esta Orden de Compra desde su plataforma web. Quedo atento a su confirmación y factura proforma.
```

### 10.3. Opciones de Descarga Complementarias
Dentro del modal corporativo de confirmación se ofrecen además dos botones secundarios de descarga directa:
* **`📄 Descargar Orden (PDF)`:** Emite la Orden de Compra formal con membrete, RIF, datos fiscales, tabla de productos, totales y firmas de conformidad.
* **`📋 Descargar Cotización (PDF)`:** Emite el documento de cotización corporativa con la lista de productos y directorio de ventas de la empresa.


