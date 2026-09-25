# PLANIFICACIÓN INTEGRAL DE ARQUITECTURA API RESTful
## Comunicación Bidireccional: Tienda Web UNIFORTEX ↔ Sistema de Gestión ERP (Laravel 11 + Filament v3)

---

### ÍNDICE DE CONTENIDO
1. [Objetivo y Topología de la Arquitectura](#1-objetivo-y-topología-de-la-arquitectura)
2. [Datos Oficiales Existentes de UNIFORTEX](#2-datos-oficiales-existentes-de-unifortex)
3. [Matriz de Endpoints de la API RESTful (v1)](#3-matriz-de-endpoints-de-la-api-restful-v1)
4. [Contratos de Datos JSON (Request & Response)](#4-contratos-de-datos-json-request--response)
5. [Estructura de Base de Datos y Migraciones Laravel 11](#5-estructura-de-base-de-datos-y-migraciones-laravel-11)
6. [Implementación de Controladores y FormRequests](#6-implementación-de-controladores-y-formrequests)
7. [Comunicación en Tiempo Real: WebSockets con Laravel Reverb](#7-comunicación-en-tiempo-real-websockets-con-laravel-reverb)
8. [Cliente Frontend Híbrido (`UnifortexApiClient.js`)](#8-cliente-frontend-híbrido-unifortexapiclientjs)
9. [Seguridad, CORS, Autenticación y Tolerancia a Fallos](#9-seguridad-cors-autenticación-y-tolerancia-a-fallos)

---

## 1. OBJETIVO Y TOPOLOGÍA DE LA ARQUITECTURA

El propósito de esta planificación es establecer el estándar de ingeniería definitivo para conectar la **Tienda Web Pública** ([prototipomig.html](file:///c:/Users/Grupo%20UNIFORTEX/Desktop/Nueva%20carpeta/prototipomig.html)) con el **Sistema de Gestión Empresarial ERP** ([erp_unifortex.html](file:///c:/Users/Grupo%20UNIFORTEX/Desktop/Nueva%20carpeta/erp_unifortex.html) / Laravel 11 + Filament v3).

### Diagrama de Flujo de Datos

```
+-----------------------------------------------------------------------------------+
|                            TIENDA WEB PÚBLICA B2B                                 |
|                         (prototipomig.html - Cliente)                             |
+-----------------------------------------------------------------------------------+
       ▲                                                                │
       │ (1) Consulta catálogo, video, combos y tasa                    │ (2) Envía Cotización
       │     GET /api/v1/productos                                      │     POST /api/v1/cotizaciones
       │     GET /api/v1/novedades                                      │
       │     GET /api/v1/config                                         ▼
+──────┴────────────────────────────────────────────────────────────────────────────+
|                               API GATEWAY RESTful                                  |
|                             (Laravel 11 / Sanctum / CORS)                         |
+──────┬────────────────────────────────────────────────────────────────────┬───────+
       │                                                                    │
       │ (3) Sincronización Push WebSockets                                 │ (4) CRUD & Uploads
       │     (Laravel Reverb / SSE en tiempo real)                          │     (Filament v3)
       ▼                                                                    ▼
+--------------------------------------+   +----------------------------------------+
|       BASE DE DATOS RELACIONAL       |   |             SISTEMA ERP                |
|        (MySQL / PostgreSQL)          |   |        (Panel de Administración)       |
|  - products      - packages          |   |  - Módulo "Edición de Página"          |
|  - quotations    - settings          |   |  - Bandeja de Cotizaciones Web         |
|  - reels         - quotation_items   |   |  - Control de Inventario y Producción  |
+--------------------------------------+   +----------------------------------------+
```

---

## 2. DATOS OFICIALES EXISTENTES DE UNIFORTEX

Para evitar discrepancias en producción, la API y la base de datos se alimentan con la información corporativa oficial ya validada en el prototipo:

| Parámetro | Valor Oficial de Negocio | Propósito en el Sistema |
| :--- | :--- | :--- |
| **WhatsApp Principal (Ventas)** | `+58 414-4938316` (`584144938316`) | Destinatario principal de cotizaciones y pedidos automáticos de la web. |
| **WhatsApp Asesoría Técnica** | `+58 424-3409898` (`584243409898`) | Soporte en normas técnicas COVENIN, OSHA y especificaciones de telas. |
| **WhatsApp B2B Corporativo** | `+58 424-3056068` (`584243056068`) | Atención a compras corporativas de licitación y grandes lotes industriales. |
| **Teléfono PBX / Llamadas** | `+58 (414) 493-8316` | Línea de atención telefónica visual en cabecera y pie de página. |
| **Correo Electrónico Oficial** | `grupounifortexventas@gmail.com` | Redacción directa de correos y respaldo de cotizaciones formales. |
| **Sede Maracay (Tienda & Almacén)** | `C.C. Ciudad Jardín, Local 20, Maracay, Edo. Aragua` | Despacho centro-occidente y atención al cliente presencial. |
| **Sede Caracas (Administrativa/B2B)**| `Av. Libertador, Edif. Fertec, Piso 4, Caracas` | Oficinas administrativas corporativas y contratación mayorista. |
| **Redes Sociales Oficiales** | Instagram: `@grupounifortex`<br>TikTok: `@grupounifortex`<br>Facebook: `@grupounifortex` | Presencia digital y enlaces oficiales del footer. |
| **Tasa Oficial BCV** | `36.85` Bs./USD (Dinámica) | Base para la conversión automática de precios en Bs. |
| **Descuento Mayorista B2B** | `18%` | Aplicable automáticamente a partir de 12 unidades por producto. |
| **Catálogo Base Inicial** | 60 Modelos Industriales | Dividido en 12 categorías normadas (Pantalones, Calzado, Bragas, EPP, etc.). |

---

## 3. MATRIZ DE ENDPOINTS DE LA API RESTFUL (v1)

Todos los endpoints están prefijados por `/api/v1/` y devuelven respuestas estructuradas en formato JSON (`Content-Type: application/json`).

| Método | Endpoint | Acceso | Función Técnica |
| :--- | :--- | :--- | :--- |
| **GET** | `/api/v1/config` | Público | Devuelve teléfonos, WhatsApps, sedes, correos y tasa BCV actual. |
| **PUT** | `/api/v1/config` | Privado (ERP) | Actualiza teléfonos, WhatsApp, tasa BCV o datos de sedes. |
| **GET** | `/api/v1/novedades` | Público | Devuelve video reel 9:16, producto vinculado, badge y vitrina. |
| **PUT** | `/api/v1/novedades` | Privado (ERP) | Actualiza el video reel, textos, precios y prendas del video. |
| **GET** | `/api/v1/productos` | Público | Lista los 60 productos con filtros por categoría, búsqueda y stock. |
| **GET** | `/api/v1/productos/{id}` | Público | Ficha técnica detallada de un producto específico. |
| **POST** | `/api/v1/productos` | Privado (ERP) | Registra un nuevo producto en el catálogo. |
| **PUT** | `/api/v1/productos/{id}` | Privado (ERP) | Modifica la ficha completa (precios, tallas, material, foto). |
| **PATCH**| `/api/v1/productos/{id}/quick` | Privado (ERP) | Cambio rápido in situ de stock, visibilidad o precio. |
| **DELETE**| `/api/v1/productos/{id}` | Privado (ERP) | Elimina o desactiva un producto del catálogo web. |
| **GET** | `/api/v1/paquetes` | Público | Obtiene los combos y paquetes activos para la página de inicio. |
| **POST** | `/api/v1/paquetes` | Privado (ERP) | Crea un nuevo combo corporativo. |
| **PUT** | `/api/v1/paquetes/{id}` | Privado (ERP) | Edita precio, título o ítems de un combo. |
| **DELETE**| `/api/v1/paquetes/{id}` | Privado (ERP) | Elimina un combo. |
| **POST** | `/api/v1/cotizaciones` | Público | Registra una orden de cotización enviada por un cliente web. |
| **GET** | `/api/v1/cotizaciones` | Privado (ERP) | Bandeja de cotizaciones recibidas con paginación y filtros. |
| **PATCH**| `/api/v1/cotizaciones/{id}/estado` | Privado (ERP) | Actualiza el estado (*Pendiente, Aprobado, Producción, Despachado*). |
| **POST** | `/api/v1/media/upload` | Privado (ERP) | Subida multipart de videos 9:16 o fotos de productos a almacenamiento. |

---

## 4. CONTRATOS DE DATOS JSON (REQUEST & RESPONSE)

### 4.1 Configuración de Empresa (`GET /api/v1/config`)
#### Respuesta (200 OK):
```json
{
  "success": true,
  "data": {
    "empresa": "UNIFORTEX INDUSTRIAL",
    "rif": "J-30491829-1",
    "whatsapp_ventas": "584144938316",
    "whatsapp_asesoria": "584243409898",
    "whatsapp_b2b": "584243056068",
    "telefono_llamadas": "+58 (414) 493-8316",
    "correo_oficial": "grupounifortexventas@gmail.com",
    "tasa_bcv": 36.85,
    "descuento_mayorista_pct": 18,
    "pedido_minimo_mayor": 12,
    "sedes": [
      {
        "ciudad": "Maracay",
        "tipo": "Tienda & Almacén Principal",
        "direccion": "C.C. Ciudad Jardín, Local 20, Maracay, Edo. Aragua",
        "horario": "Lunes a Viernes: 8:00 AM - 5:00 PM"
      },
      {
        "ciudad": "Caracas",
        "tipo": "Oficinas Administrativas & Atención Mayorista",
        "direccion": "Av. Libertador, Edif. Fertec, Piso 4, Caracas",
        "horario": "Lunes a Viernes: 8:30 AM - 4:30 PM"
      }
    ],
    "redes": {
      "instagram": "https://instagram.com/grupounifortex",
      "tiktok": "https://tiktok.com/@grupounifortex",
      "facebook": "https://facebook.com/grupounifortex"
    }
  }
}
```

---

### 4.2 Novedad Destacada & Video Reel 9:16 (`GET /api/v1/novedades`)
#### Respuesta (200 OK):
```json
{
  "success": true,
  "data": {
    "badge": "🔥 LANZAMIENTO EXCLUSIVO 2026",
    "titulo": "Novedades & Lanzamientos de Temporada",
    "subtitulo": "Descubre en video real el rendimiento, ajuste y resistencia de nuestras prendas de dotación y EPP.",
    "video_url": "https://assets.mixkit.co/videos/preview/mixkit-man-working-in-a-warehouse-inspecting-boxes-42797-large.mp4",
    "video_aspect_ratio": "9:16",
    "video_caption": "Demostración de resistencia al agua, viento y visibilidad reflectiva 3M en condiciones reales.",
    "producto_vinculado": {
      "id": 58,
      "sku": "COD - BMO01",
      "nombre": "Conjunto Motorizado Impermeable Negro 3M",
      "subtitulo": "Protección hidrostática certificada, doble solapa cortaviento y bandas reflectivas microprismáticas",
      "categoria": "Impermeables & Vial",
      "precio_regular": 34.00,
      "precio_oferta": 28.00,
      "ahorro_texto": "Ahorras $6.00/ud a partir de 12 dotaciones"
    },
    "prendas_en_video": [
      { "icono": "🏍️", "nombre": "Conjunto 2 Piezas Lona Impermeable 3M", "detalle": "Impermeabilidad 10,000mm" },
      { "icono": "🧤", "nombre": "Guantes de Precisión Poliuretano", "detalle": "Agarre antideslizante en aceites" },
      { "icono": "🥾", "nombre": "Botas de Seguridad Cuero Vacuno", "detalle": "Puntera de acero norma ANSI" }
    ]
  }
}
```

---

### 4.3 Envío de Cotización desde la Web (`POST /api/v1/cotizaciones`)
#### Petición (Request Payload):
```json
{
  "cliente": {
    "nombre": "Ing. Carlos Mendoza",
    "empresa": "Constructora Andina C.A.",
    "rif": "J-30491829-1",
    "telefono": "+584144938316",
    "correo": "cmendoza@andina.com"
  },
  "items": [
    {
      "producto_id": 1,
      "sku": "COD - PC28",
      "nombre": "Pantalón / Blue Jeans 3 Costuras",
      "talla": "34",
      "cantidad": 24,
      "precio_unitario": 20.09
    },
    {
      "producto_id": 4,
      "sku": "COD - CH01",
      "nombre": "Chemise Corporativa Institucional",
      "talla": "L",
      "cantidad": 24,
      "precio_unitario": 12.14
    }
  ],
  "tasa_bcv_aplicada": 36.85,
  "nota_adicional": "Solicitamos cotización con bordado del logo de la empresa en el bolsillo izquierdo.",
  "origen": "Tienda Web B2B"
}
```

#### Respuesta (201 Created):
```json
{
  "success": true,
  "message": "Cotización registrada exitosamente.",
  "data": {
    "id": 1042,
    "numero_orden": "COT-2026-1042",
    "fecha": "2026-09-25 14:15:00",
    "total_usd": 773.52,
    "total_bs": 28504.21,
    "estado": "Pendiente",
    "enlace_whatsapp_directo": "https://wa.me/584144938316?text=Hola%20UNIFORTEX%2C%20he%20generado%20la%20cotizaci%C3%B3n%20COT-2026-1042..."
  }
}
```

---

## 5. ESTRUCTURA DE BASE DE DATOS Y MIGRACIONES LARAVEL 11

Tu compañero desarrollador solo debe crear estas 6 migraciones estándar en Laravel:

### 5.1 Migración de Productos (`database/migrations/2026_01_01_000001_create_products_table.php`)
```php
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    public function up(): void
    {
        Schema::create('products', function (Blueprint $table) {
            $table->id();
            $table->string('code')->unique(); // SKU ej: COD - PC28
            $table->string('name');
            $table->string('category');
            $table->decimal('price_retail', 10, 2); // Precio Detal USD
            $table->decimal('price_wholesale', 10, 2); // Precio Mayor USD
            $table->integer('stock_quantity')->default(0);
            $table->integer('safety_stock')->default(10); // Alerta stock mínimo
            $table->string('sizes')->nullable();
            $table->string('size_short')->nullable();
            $table->text('material_description')->nullable();
            $table->string('material_short')->nullable();
            $table->string('icon_emoji', 10)->default('👕');
            $table->string('image_url')->nullable(); // Foto del producto
            $table->boolean('is_visible_web')->default(true);
            $table->boolean('is_featured_home')->default(false); // Estrella
            $table->timestamps();
        });
    }

    public function down(): void
    {
        Schema::dropIfExists('products');
    }
};
```

### 5.2 Migración de Cotizaciones (`database/migrations/2026_01_01_000002_create_quotations_table.php`)
```php
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    public function up(): void
    {
        Schema::create('quotations', function (Blueprint $table) {
            $table->id();
            $table->string('order_number')->unique(); // ej: COT-2026-1042
            $table->string('client_name');
            $table->string('company_name')->nullable();
            $table->string('tax_id')->nullable(); // RIF
            $table->string('phone');
            $table->string('email')->nullable();
            $table->decimal('total_usd', 12, 2);
            $table->decimal('exchange_rate_bcv', 10, 2);
            $table->decimal('total_bs', 14, 2);
            $table->text('customer_notes')->nullable();
            $table->enum('status', ['Pendiente', 'En Revisión', 'Aprobado', 'En Producción', 'Despachado', 'Cancelado'])->default('Pendiente');
            $table->string('channel_origin')->default('Web B2B');
            $table->timestamps();
        });

        Schema::create('quotation_items', function (Blueprint $table) {
            $table->id();
            $table->foreignId('quotation_id')->constrained('quotations')->onDelete('cascade');
            $table->foreignId('product_id')->nullable()->constrained('products')->nullOnDelete();
            $table->string('sku');
            $table->string('product_name');
            $table->string('size')->default('Única');
            $table->integer('quantity');
            $table->decimal('unit_price', 10, 2);
            $table->decimal('subtotal', 12, 2);
            $table->timestamps();
        });
    }

    public function down(): void
    {
        Schema::dropIfExists('quotation_items');
        Schema::dropIfExists('quotations');
    }
};
```

### 5.3 Migración de Video Novedad & Configuración (`database/migrations/2026_01_01_000003_create_reels_and_settings_table.php`)
```php
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    public function up(): void
    {
        Schema::create('featured_reels', function (Blueprint $table) {
            $table->id();
            $table->string('badge')->default('🔥 LANZAMIENTO EXCLUSIVO 2026');
            $table->string('title')->default('Novedades & Lanzamientos de Temporada');
            $table->string('subtitle')->nullable();
            $table->string('video_url');
            $table->text('video_caption')->nullable();
            $table->foreignId('featured_product_id')->nullable()->constrained('products')->nullOnDelete();
            $table->string('product_display_title');
            $table->string('product_display_subtitle')->nullable();
            $table->decimal('price_regular', 10, 2);
            $table->decimal('price_sale', 10, 2);
            $table->string('saving_badge_text')->nullable();
            $table->text('product_description')->nullable();
            $table->json('video_vitrina_items')->nullable(); // Array de items mostrados
            $table->boolean('is_active')->default(true);
            $table->timestamps();
        });

        Schema::create('company_settings', function (Blueprint $table) {
            $table->id();
            $table->string('key')->unique();
            $table->text('value')->nullable();
            $table->timestamps();
        });
    }

    public function down(): void
    {
        Schema::dropIfExists('featured_reels');
        Schema::dropIfExists('company_settings');
    }
};
```

---

## 6. IMPLEMENTACIÓN DE CONTROLADORES Y FORMREQUESTS

### 6.1 Controlador de Cotizaciones (`app/Http/Controllers/Api/QuotationController.php`)
```php
<?php

namespace App\Http\Controllers\Api;

use App\Http\Controllers\Controller;
use App\Models\Quotation;
use App\Models\QuotationItem;
use App\Events\NewQuotationReceived;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\DB;
use Illuminate\Support\Facades\Validator;

class QuotationController extends Controller
{
    public function store(Request $request)
    {
        $validator = Validator::make($request->all(), [
            'cliente.nombre' => 'required|string|max:150',
            'cliente.telefono' => 'required|string|max:30',
            'cliente.empresa' => 'nullable|string|max:150',
            'cliente.rif' => 'nullable|string|max:30',
            'items' => 'required|array|min:1',
            'items.*.nombre' => 'required|string',
            'items.*.cantidad' => 'required|integer|min:1',
            'items.*.precio_unitario' => 'required|numeric|min:0',
        ]);

        if ($validator->fails()) {
            return response()->json([
                'success' => false,
                'errors' => $validator->errors()
            ], 422);
        }

        return DB::transaction(function () use ($request) {
            $bcvRate = (float) ($request->input('tasa_bcv_aplicada') ?? 36.85);
            $orderNumber = 'COT-' . date('Y') . '-' . str_pad((Quotation::max('id') + 1), 4, '0', STR_PAD_LEFT);

            $totalUsd = 0;
            foreach ($request->input('items') as $it) {
                $totalUsd += ($it['cantidad'] * $it['precio_unitario']);
            }
            $totalBs = $totalUsd * $bcvRate;

            $cotizacion = Quotation::create([
                'order_number' => $orderNumber,
                'client_name' => $request->input('cliente.nombre'),
                'company_name' => $request->input('cliente.empresa'),
                'tax_id' => $request->input('cliente.rif'),
                'phone' => $request->input('cliente.telefono'),
                'email' => $request->input('cliente.correo'),
                'total_usd' => $totalUsd,
                'exchange_rate_bcv' => $bcvRate,
                'total_bs' => $totalBs,
                'customer_notes' => $request->input('nota_adicional'),
                'status' => 'Pendiente',
                'channel_origin' => $request->input('origen', 'Web B2B'),
            ]);

            foreach ($request->input('items') as $it) {
                QuotationItem::create([
                    'quotation_id' => $cotizacion->id,
                    'product_id' => $it['producto_id'] ?? null,
                    'sku' => $it['sku'] ?? 'GEN',
                    'product_name' => $it['nombre'],
                    'size' => $it['talla'] ?? 'Única',
                    'quantity' => $it['cantidad'],
                    'unit_price' => $it['precio_unitario'],
                    'subtotal' => $it['cantidad'] * $it['precio_unitario'],
                ]);
            }

            // Despacha evento WebSocket para que el ERP suene la campana en tiempo real
            event(new NewQuotationReceived($cotizacion));

            return response()->json([
                'success' => true,
                'message' => 'Cotización creada exitosamente.',
                'data' => [
                    'id' => $cotizacion->id,
                    'numero_orden' => $cotizacion->order_number,
                    'total_usd' => $cotizacion->total_usd,
                    'total_bs' => $cotizacion->total_bs,
                    'estado' => $cotizacion->status
                ]
            ], 201);
        });
    }

    public function index()
    {
        $quotes = Quotation::with('items')->latest()->paginate(20);
        return response()->json(['success' => true, 'data' => $quotes]);
    }

    public function updateStatus(Request $request, $id)
    {
        $request->validate(['status' => 'required|in:Pendiente,En Revisión,Aprobado,En Producción,Despachado,Cancelado']);
        $quote = Quotation::findOrFail($id);
        $quote->update(['status' => $request->status]);
        return response()->json(['success' => true, 'message' => 'Estado actualizado.', 'data' => $quote]);
    }
}
```

### 6.2 Controlador de Carga de Multimedia (`app/Http/Controllers/Api/MediaUploadController.php`)
```php
<?php

namespace App\Http\Controllers\Api;

use App\Http\Controllers\Controller;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Storage;

class MediaUploadController extends Controller
{
    public function upload(Request $request)
    {
        $request->validate([
            'file' => 'required|file|mimes:jpeg,png,webp,mp4,webm|max:51200', // Hasta 50MB
            'type' => 'required|in:product_image,video_reel'
        ]);

        $folder = $request->type === 'video_reel' ? 'videos/reels' : 'productos/fotos';
        $path = $request->file('file')->store($folder, 'public');
        $fullUrl = asset('storage/' . $path);

        return response()->json([
            'success' => true,
            'url' => $fullUrl,
            'path' => $path
        ]);
    }
}
```

---

## 7. COMUNICACIÓN EN TIEMPO REAL: WEBSOCKETS CON LARAVEL REVERB

Laravel 11 incluye **Laravel Reverb**, un servidor de WebSockets de primera clase, ultra-rápido y nativo.

### 7.1 Definición de Canales (`routes/channels.php`)
```php
use App\Models\User;
use Illuminate\Support\Facades\Broadcast;

// Canal público de la tienda: Cualquier cliente escuchando recibe actualizaciones de precios y catálogo
Broadcast::channel('unifortex-public-catalog', function () {
    return true;
});

// Canal privado administrativo: Solo los usuarios con rol operador en Filament escuchan nuevas cotizaciones
Broadcast::channel('unifortex-erp-orders', function (User $user) {
    return $user->can('view_erp');
});
```

### 7.2 Evento de Nueva Cotización (`app/Events/NewQuotationReceived.php`)
```php
namespace App\Events;

use App\Models\Quotation;
use Illuminate\Broadcasting\Channel;
use Illuminate\Broadcasting\InteractsWithSockets;
use Illuminate\Contracts\Broadcasting\ShouldBroadcastNow;

class NewQuotationReceived implements ShouldBroadcastNow
{
    use InteractsWithSockets;

    public $quotation;

    public function __construct(Quotation $quotation)
    {
        $this->quotation = $quotation->load('items');
    }

    public function broadcastOn()
    {
        return new Channel('unifortex-erp-orders');
    }

    public function broadcastAs()
    {
        return 'new_quotation_alert';
    }
}
```

---

## 8. CLIENTE FRONTEND HÍBRIDO (`UnifortexApiClient.js`)

Para garantizar que la web funcione tanto conectada al servidor como en modo local/offline sin romperse nunca, se proporciona este conector JavaScript drop-in:

```javascript
/**
 * UnifortexApiClient.js - Conector Oficial Web ↔ Laravel API
 * Soporta Zero Downtime: Si la API no responde, opera automáticamente con LocalStorage.
 */
class UnifortexApiClient {
    constructor(config = {}) {
        this.baseUrl = config.baseUrl || 'http://127.0.0.1:8000/api/v1';
        this.timeout = config.timeout || 4000;
        this.isOnline = true;
    }

    async request(endpoint, options = {}) {
        const url = `${this.baseUrl}${endpoint}`;
        const controller = new AbortController();
        const timer = setTimeout(() => controller.abort(), this.timeout);

        try {
            const res = await fetch(url, {
                ...options,
                signal: controller.signal,
                headers: {
                    'Accept': 'application/json',
                    'Content-Type': 'application/json',
                    ...(options.headers || {})
                }
            });
            clearTimeout(timer);
            if (!res.ok) throw new Error(`HTTP Error ${res.status}`);
            this.isOnline = true;
            return await res.json();
        } catch (err) {
            clearTimeout(timer);
            console.warn(`[UNIFORTEX API OFFLINE]: ${url} no disponible. Activando respaldo local.`);
            this.isOnline = false;
            return null; // El frontend tomará los datos de LocalStorage
        }
    }

    // PRODUCTOS
    async getProductos(filters = {}) {
        const params = new URLSearchParams(filters).toString();
        const data = await this.request(`/productos?${params}`);
        if (data && data.success) return data.data;
        // Respaldo
        return JSON.parse(localStorage.getItem('unifortex_catalogo_productos') || '[]');
    }

    // NOVEDAD & VIDEO REEL
    async getNovedades() {
        const data = await this.request('/novedades');
        if (data && data.success) return data.data;
        return JSON.parse(localStorage.getItem('unifortex_novedad_destacada') || '{}');
    }

    // ENVIAR COTIZACIÓN
    async enviarCotizacion(payload) {
        const res = await this.request('/cotizaciones', {
            method: 'POST',
            body: JSON.stringify(payload)
        });

        if (res && res.success) {
            return res.data;
        }

        // Si la API falla, guardar en la cola local para no perder la venta
        const offlineQueue = JSON.parse(localStorage.getItem('unifortex_offline_orders') || '[]');
        const localOrder = {
            ...payload,
            numero_orden: `OFFLINE-${Date.now()}`,
            fecha: new Date().toLocaleString()
        };
        offlineQueue.push(localOrder);
        localStorage.setItem('unifortex_offline_orders', JSON.stringify(offlineQueue));
        
        // Notificar por BroadcastChannel al ERP local si está abierto
        try {
            const bc = new BroadcastChannel('unifortex_sync_channel');
            bc.postMessage({ type: 'new_web_order', data: localOrder });
        } catch (e) {}

        return localOrder;
    }
}

// Instancia global
window.unifortexApi = new UnifortexApiClient();
```

---

## 9. SEGURIDAD, CORS, AUTENTICACIÓN Y TOLERANCIA A FALLOS

### 9.1 Configuración de CORS en Laravel (`config/cors.php`)
Permite que el archivo `prototipomig.html` o el dominio web de producción consulte la API sin bloqueos del navegador:

```php
return [
    'paths' => ['api/*', 'sanctum/csrf-cookie'],
    'allowed_methods' => ['*'],
    'allowed_origins' => [
        'http://localhost:*',
        'http://127.0.0.1:*',
        'https://unifortex.com',
        'https://*.unifortex.com',
    ],
    'allowed_origins_patterns' => [],
    'allowed_headers' => ['*'],
    'exposed_headers' => [],
    'max_age' => 86400,
    'supports_credentials' => true,
];
```

### 9.2 Rate Limiting (Protección Anti-Ataques) en `app/Providers/RouteServiceProvider.php`
* **Tienda Pública:** 60 peticiones por minuto por IP para evitar scraping abusivo.
* **Envío de Cotizaciones:** Máximo 5 cotizaciones por minuto por IP para evitar spam en WhatsApp.
* **Panel ERP:** Sin limitación estricta para usuarios autenticados con token Sanctum.

### 9.3 Ventajas de esta Arquitectura para el Compañero Desarrollador
1. **Desacoplamiento Absoluto:** El compañero puede trabajar en el backend Laravel 11 sin romper la tienda web ni el diseño visual.
2. **Cero Dependencia de Migración Abrupta:** La web puede consumir la API o los archivos estáticos indistintamente.
3. **Persistencia Garantizada:** Las fotos y videos subidos desde el ERP van directamente al disco público de Laravel (`storage/app/public`) y quedan disponibles en URLs CDN de alta velocidad.
4. **Cumplimiento Integral de Requerimientos:** Soporta la edición en vivo desde la pestaña única `Edición de Página`, los 60 productos, el reel 9:16 y el canal directo a WhatsApp.
