# 📋 GUÍA DE INTEGRACIÓN Y ESPECIFICACIÓN TÉCNICA (LARAVEL + FILAMENT)
## Para el Desarrollador del Sistema de Gestión Empresarial (ERP / Panel de Administración)
**Proyecto:** Conexión Interna E-Commerce B2B / Catálogo Web ➔ Sistema de Gestión UNIFORTEX  
**Stack Backend / Panel:** **PHP 8.2+ / Laravel 11 / Filament v3** (TALL Stack: Tailwind, Alpine, Laravel, Livewire)  
**Objetivo:** Permitir que desde el panel administrativo desarrollado con Filament se gestione el catálogo de productos (CRUD completo, fotos con editor, stock en tiempo real, precios detal/mayorista), combos y ofertas, datos de contacto/WhatsApp de la empresa y la recepción automática de pedidos/cotizaciones con alertas y notificaciones en tiempo real.

---

## 1. ARQUITECTURA GENERAL CON LARAVEL + FILAMENT

```
┌────────────────────────────────────────────────────────────────────────┐
│             PANEL ADMINISTRATIVO FILAMENT (SISTEMA ERP)                │
│  - ProductResource: Stock, Fotos (Upload/Editor), Precios, Tallas     │
│  - PackageOfferResource: Combos dotación industrial con Repeater       │
│  - OrderResource: Bandeja de Cotizaciones con estados y botón WhatsApp │
│  - CompanySettingsPage: Teléfonos, tasa BCV, correo y descuentos       │
│  - Filament Notifications: Campana de alertas en vivo por nuevos pedidos│
│  - Filament Widgets: Métricas de ventas, stock crítico, órdenes del mes │
└───────────────────────────────────┬────────────────────────────────────┘
                                    │ (Eloquent ORM / MySQL / PostgreSQL)
                                    ▼
┌────────────────────────────────────────────────────────────────────────┐
│                   LARAVEL API REST (routes/api.php)                    │
│  - GET  /api/v1/productos          ➔ ProductApiController@index        │
│  - GET  /api/v1/paquetes           ➔ PackageApiController@index        │
│  - GET  /api/v1/configuracion      ➔ SettingApiController@show         │
│  - POST /api/v1/pedidos            ➔ OrderApiController@store          │
│  - Storage Disk 'public'           ➔ Fotos servidas via asset('storage')│
└───────────────────┬───────────────────────────────────▲────────────────┘
                    │                                   │
       JSON público │                      Recepción de │ POST Pedidos /
    (Catálogo, Info │                      Cotizaciones │ Órdenes B2B
     Precios, Stock)│                                   │
                    ▼                                   │
┌───────────────────────────────────────────────────────┴────────────────┐
│                     PÁGINA WEB CLIENTE (FRONTEND)                      │
│  - Catálogo Interactivo con 60+ Productos y Filtros Reactivos          │
│  - Bucle Infinito de Ofertas y Paquetes de Dotación Industrial         │
│  - Estudio de Diseño Textil (Personalización y Subida de Logotipos)   │
│  - Carrito B2B y Generador de Órdenes de Compra en Formato PDF         │
└────────────────────────────────────────────────────────────────────────┘
```

---

## 2. CONFIGURACIÓN INICIAL DEL PROYECTO LARAVEL & FILAMENT

### 2.1. Comandos de Instalación Base
```bash
# 1. Crear proyecto Laravel
composer create-project laravel/laravel unifortex-erp
cd unifortex-erp

# 2. Instalar Filament v3
composer require filament/filament:"^3.2" -W

# 3. Instalar panel Filament (por defecto en /admin)
php artisan filament:install --panels

# 4. Vincular el almacenamiento público para fotos
php artisan storage:link

# 5. Paquetes complementarios recomendados para Filament
composer require filament/spatie-laravel-settings-plugin -W
composer require pxlrbt/filament-excel -W
```

---

## 3. MIGRACIONES Y MODELOS ELOQUENT

### 3.1. Migración: `create_products_table.php`
```php
Schema::create('products', function (Blueprint $table) {
    $table->id();
    $table->string('codigo', 50)->unique(); // Ej: COD - PC28
    $table->string('nombre');               // Ej: Pantalón / Blue Jeans 3 Costuras
    $table->string('categoria');            // Ej: Pantalones & Jeans, Calzado
    $table->decimal('precio_detal', 10, 2); // Ej: 24.50
    $table->decimal('precio_mayor', 10, 2)->nullable(); // Si es nulo, la web calcula 18% desc.
    $table->integer('stock')->default(0);   // Ej: 150
    $table->integer('stock_minimo')->default(10); // Alerta para badge rojo
    $table->text('material');               // Descripción técnica completa
    $table->string('material_corto')->nullable(); // Ej: Índigo 14.5oz (100% Algodón)
    $table->json('tallas')->nullable();     // Arreglo JSON: ["28", "30", "32", "34", "36"]
    $table->string('tallas_resumen')->nullable(); // Ej: Tallas: 28 al 46
    $table->string('imagen_url')->nullable(); // Ruta en storage (ej: productos/jean.webp)
    $table->string('icono_emoji', 20)->default('👕'); // Emoji fallback
    $table->boolean('es_estrella')->default(false); // Para sección 'Más Buscados'
    $table->boolean('activo')->default(true);       // Visibilidad en catálogo web
    $table->integer('orden')->default(0);           // Orden de aparición
    $table->timestamps();
});
```

### 3.2. Migración: `create_package_offers_table.php` (Combos y Paquetes)
```php
Schema::create('package_offers', function (Blueprint $table) {
    $table->id();
    $table->string('slug_tipo', 50);          // cuadrilla, corporativo, epp_flash
    $table->string('titulo');                 // Ej: Cuadrilla Operativa Pro
    $table->string('etiqueta_badge')->nullable(); // Ej: COMBO INDUSTRIAL ESTÁNDAR
    $table->string('cinta_ribbon')->nullable();   // Ej: 🔥 15% OFF B2B
    $table->text('descripcion');
    $table->decimal('precio_oferta', 10, 2);  // Ej: 38.50
    $table->string('precio_anterior')->nullable(); // Ej: $45.00 USD
    $table->string('nota_condicion')->nullable();  // Ej: Aplica a partir de 12 dotaciones
    $table->json('items_incluidos');          // Arreglo de prendas y cantidades
    $table->string('categoria_filtro')->nullable(); // industrial, administrativo, etc.
    $table->boolean('mostrar_en_inicio')->default(true);
    $table->boolean('activo')->default(true);
    $table->timestamps();
});
```

### 3.3. Migración: `create_orders_table.php` y `create_order_items_table.php`
```php
Schema::create('orders', function (Blueprint $table) {
    $table->id();
    $table->string('numero_orden')->unique(); // Ej: OC-2026-0842
    $table->string('cliente_nombre');
    $table->string('cliente_empresa')->nullable();
    $table->string('cliente_rif')->nullable();
    $table->string('cliente_telefono');
    $table->string('cliente_email')->nullable();
    $table->decimal('total_usd', 10, 2)->default(0);
    $table->text('nota_adicional')->nullable();
    $table->string('metodo_origen')->default('web_catalogo'); // web_carrito, web_orden_compra, web_diseno
    $table->enum('estado', ['Pendiente', 'En Revisión', 'Aprobado', 'En Producción', 'Despachado', 'Cancelado'])
          ->default('Pendiente');
    $table->timestamps();
});

Schema::create('order_items', function (Blueprint $table) {
    $table->id();
    $table->foreignId('order_id')->constrained('orders')->cascadeOnDelete();
    $table->foreignId('product_id')->nullable()->constrained('products')->nullOnDelete();
    $table->string('producto_nombre');
    $table->string('talla')->nullable();
    $table->integer('cantidad');
    $table->decimal('precio_unitario', 10, 2);
    $table->decimal('subtotal', 10, 2);
    $table->json('detalles_personalizacion')->nullable(); // Info del estudio de diseño/bordado
    $table->timestamps();
});
```

---

## 4. FILAMENT RESOURCES (CONFIGURACIÓN DEL PANEL DE ADMINISTRACIÓN)

Para generar los recursos automáticamente en Filament:
```bash
php artisan make:filament-resource Product --generate
php artisan make:filament-resource PackageOffer --generate
php artisan make:filament-resource Order
```

### 4.1. Recurso de Productos (`App\Filament\Resources\ProductResource.php`)

#### Formulario de Edición y Creación (Filament Form):
```php
use Filament\Forms;
use Filament\Forms\Form;

public static function form(Form $form): Form
{
    return $form->schema([
        Forms\Components\Section::make('Información Básica')
            ->columns(2)
            ->schema([
                Forms\Components\TextInput::make('codigo')
                    ->label('Código / SKU')
                    ->required()
                    ->unique(ignoreRecord: true),
                Forms\Components\TextInput::make('nombre')
                    ->label('Nombre Comercial')
                    ->required(),
                Forms\Components\Select::make('categoria')
                    ->label('Categoría')
                    ->options([
                        'Pantalones & Jeans' => 'Pantalones & Jeans',
                        'Camisas & Chemises' => 'Camisas & Chemises',
                        'Bragas & Overoles' => 'Bragas & Overoles',
                        'Calzado de Seguridad' => 'Calzado de Seguridad',
                        'Protección Industrial' => 'Protección Industrial',
                        'Chaquetas & Chalecos' => 'Chaquetas & Chalecos',
                    ])
                    ->required()
                    ->searchable(),
                Forms\Components\TextInput::make('icono_emoji')
                    ->label('Emoji Representativo')
                    ->default('👕')
                    ->maxLength(5),
            ]),

        Forms\Components\Section::make('Precios e Inventario')
            ->columns(3)
            ->schema([
                Forms\Components\TextInput::make('precio_detal')
                    ->label('Precio Detal ($ USD)')
                    ->numeric()
                    ->prefix('$')
                    ->required(),
                Forms\Components\TextInput::make('precio_mayor')
                    ->label('Precio Mayorista ($ USD)')
                    ->numeric()
                    ->prefix('$')
                    ->helperText('Dejar vacío para calcular 18% desc. automáticamente'),
                Forms\Components\TextInput::make('stock')
                    ->label('Stock Actual')
                    ->numeric()
                    ->default(0)
                    ->required(),
                Forms\Components\TextInput::make('stock_minimo')
                    ->label('Alerta Stock Mínimo')
                    ->numeric()
                    ->default(10)
                    ->helperText('Activa badge de "Pocas Unidades"'),
                Forms\Components\Toggle::make('es_estrella')
                    ->label('Destacar en "Más Buscados"')
                    ->default(false),
                Forms\Components\Toggle::make('activo')
                    ->label('Visible en la Web')
                    ->default(true),
            ]),

        Forms\Components\Section::make('Ficha Técnica y Tallas')
            ->columns(2)
            ->schema([
                Forms\Components\TextInput::make('material_corto')
                    ->label('Resumen Material')
                    ->placeholder('Ej: Índigo 14.5oz (100% Algodón)'),
                Forms\Components\TextInput::make('tallas_resumen')
                    ->label('Resumen de Tallas')
                    ->placeholder('Ej: Tallas: 28 al 46'),
                Forms\Components\Textarea::make('material')
                    ->label('Descripción Completa de Composición y Uso')
                    ->columnSpanFull(),
                Forms\Components\TagsInput::make('tallas')
                    ->label('Tallas Disponibles')
                    ->placeholder('Escribe la talla y presiona Enter')
                    ->columnSpanFull(),
            ]),

        Forms\Components\Section::make('Fotografía del Producto')
            ->schema([
                Forms\Components\FileUpload::make('imagen_url')
                    ->label('Foto Oficial (Formato WebP o JPG)')
                    ->image()
                    ->imageEditor()
                    ->directory('productos')
                    ->disk('public')
                    ->visibility('public')
                    ->maxSize(3072), // 3MB máx
            ]),
    ]);
}
```

#### Tabla de Productos (Filament Table con Alertas de Stock):
```php
use Filament\Tables;
use Filament\Tables\Table;

public static function table(Table $table): Table
{
    return $table
        ->columns([
            Tables\Columns\ImageColumn::make('imagen_url')
                ->label('Foto')
                ->disk('public')
                ->square()
                ->defaultImageUrl(fn ($record) => 'https://ui-avatars.com/api/?name=' . urlencode($record->icono_emoji)),

            Tables\Columns\TextColumn::make('codigo')
                ->label('Código')
                ->searchable()
                ->weight('bold'),

            Tables\Columns\TextColumn::make('nombre')
                ->label('Producto')
                ->searchable()
                ->limit(30),

            Tables\Columns\TextColumn::make('categoria')
                ->badge()
                ->color('gray'),

            Tables\Columns\TextColumn::make('precio_detal')
                ->label('Detal ($)')
                ->money('USD')
                ->sortable(),

            Tables\Columns\TextColumn::make('stock')
                ->label('Stock')
                ->badge()
                ->color(fn ($state, $record): string => match (true) {
                    $state <= 0 => 'danger',
                    $state <= $record->stock_minimo => 'warning',
                    default => 'success',
                })
                ->sortable(),

            Tables\Columns\ToggleColumn::make('es_estrella')
                ->label('⭐ Estrella'),

            Tables\Columns\ToggleColumn::make('activo')
                ->label('Visible'),
        ])
        ->filters([
            Tables\Filters\SelectFilter::make('categoria'),
            Tables\Filters\TernaryFilter::make('activo')->label('Solo Activos'),
            Tables\Filters\Filter::make('bajo_stock')
                ->label('Bajo Stock / Agotados')
                ->query(fn ($query) => $query->whereColumn('stock', '<=', 'stock_minimo')),
        ])
        ->actions([
            Tables\Actions\EditAction::make(),
            Tables\Actions\DeleteAction::make(),
        ]);
}
```

---

### 4.2. Recurso de Pedidos / Cotizaciones Web (`OrderResource.php`)
Permite al compañero gestionar todas las cotizaciones y compras enviadas por los clientes desde la web.

```php
use Filament\Tables;
use Filament\Tables\Table;
use Filament\Tables\Actions\Action;

public static function table(Table $table): Table
{
    return $table
        ->columns([
            Tables\Columns\TextColumn::make('numero_orden')
                ->label('N° Cotización')
                ->weight('bold')
                ->searchable(),

            Tables\Columns\TextColumn::make('created_at')
                ->label('Fecha')
                ->dateTime('d/m/Y H:i')
                ->sortable(),

            Tables\Columns\TextColumn::make('cliente_empresa')
                ->label('Empresa')
                ->searchable()
                ->placeholder('Particular'),

            Tables\Columns\TextColumn::make('cliente_nombre')
                ->label('Contacto')
                ->searchable(),

            Tables\Columns\TextColumn::make('total_usd')
                ->label('Total ($)')
                ->money('USD')
                ->weight('bold'),

            Tables\Columns\BadgeColumn::make('estado')
                ->colors([
                    'warning' => 'Pendiente',
                    'primary' => 'En Revisión',
                    'success' => 'Aprobado',
                    'info' => 'En Producción',
                    'gray' => 'Despachado',
                    'danger' => 'Cancelado',
                ]),
        ])
        ->actions([
            Tables\Actions\ViewAction::make(),
            
            // Botón directo para responder al WhatsApp del cliente con 1 clic
            Action::make('whatsapp')
                ->label('WhatsApp')
                ->icon('heroicon-o-chat-bubble-left-ellipsis')
                ->color('success')
                ->url(fn ($record) => "https://wa.me/{$record->cliente_telefono}?text=" . urlencode("Hola {$record->cliente_nombre}, te contactamos de UNIFORTEX respecto a tu cotización {$record->numero_orden}."))
                ->openUrlInNewTab(),

            Tables\Actions\EditAction::make(),
        ]);
}
```

---

## 5. API REST EN LARAVEL PARA ALIMENTAR EL FRONTEND

El frontend estático/dinámico interactúa con el backend de Laravel a través de `routes/api.php`.

### 5.1. Definición de Rutas (`routes/api.php`)
```php
use Illuminate\Support\Facades\Route;
use App\Http\Controllers\Api\ProductApiController;
use App\Http\Controllers\Api\PackageApiController;
use App\Http\Controllers\Api\SettingApiController;
use App\Http\Controllers\Api\OrderApiController;

Route::prefix('v1')->group(function () {
    // Endpoints Públicos para el Catálogo Web
    Route::get('/productos', [ProductApiController::class, 'index']);
    Route::get('/paquetes', [PackageApiController::class, 'index']);
    Route::get('/configuracion', [SettingApiController::class, 'show']);

    // Endpoint de Recepción de Pedidos (Protegido por Rate Limiting)
    Route::post('/pedidos', [OrderApiController::class, 'store'])
        ->middleware('throttle:30,1'); // Máx 30 envíos por minuto anti-spam
});
```

### 5.2. Controlador de Productos (`ProductApiController.php`)
```php
namespace App\Http\Controllers\Api;

use App\Http\Controllers\Controller;
use App\Models\Product;
use Illuminate\Http\JsonResponse;

class ProductApiController extends Controller
{
    public function index(): JsonResponse
    {
        // Solo traemos los activos, ordenados
        $productos = Product::where('activo', true)
            ->orderBy('orden')
            ->orderBy('id', 'desc')
            ->get();

        $formateados = $productos->map(function ($p) {
            return [
                'id' => $p->id,
                'code' => $p->codigo,
                'name' => $p->nombre,
                'cat' => $p->categoria,
                'price' => (float) $p->precio_detal,
                'bulkPrice' => $p->precio_mayor ? (float) $p->precio_mayor : round($p->precio_detal * 0.82, 2),
                'stock' => (int) $p->stock,
                'stockMin' => (int) $p->stock_minimo,
                'mat' => $p->material,
                'matShort' => $p->material_corto ?? $p->material,
                'sizes' => is_array($p->tallas) ? implode(' - ', $p->tallas) : ($p->tallas ?? 'Única'),
                'sizeShort' => $p->tallas_resumen ?? 'Tallas varias',
                'ico' => $p->icono_emoji ?? '👕',
                'img' => $p->imagen_url ? asset('storage/' . $p->imagen_url) : null,
                'isStar' => (bool) $p->es_estrella,
                'active' => (bool) $p->activo,
            ];
        });

        return response()->json([
            'success' => true,
            'count' => $formateados->count(),
            'data' => $formateados,
        ]);
    }
}
```

### 5.3. Controlador de Pedidos con Alerta en Filament (`OrderApiController.php`)
Al crearse un pedido desde la web, este controlador guarda la orden y emite una **notificación en vivo dentro del panel de Filament**:

```php
namespace App\Http\Controllers\Api;

use App\Http\Controllers\Controller;
use App\Models\Order;
use App\Models\OrderItem;
use App\Models\User;
use Illuminate\Http\Request;
use Illuminate\Http\JsonResponse;
use Filament\Notifications\Notification;
use Illuminate\Support\Str;

class OrderApiController extends Controller
{
    public function store(Request $request): JsonResponse
    {
        $validated = $request->validate([
            'cliente.nombre' => 'required|string|max:150',
            'cliente.empresa' => 'nullable|string|max:150',
            'cliente.rif' => 'nullable|string|max:50',
            'cliente.telefono' => 'required|string|max:50',
            'cliente.correo' => 'nullable|email|max:100',
            'items' => 'required|array|min:1',
            'totalUsd' => 'required|numeric',
            'notaAdicional' => 'nullable|string',
            'origen' => 'nullable|string',
        ]);

        $numeroOrden = 'OC-' . date('Y') . '-' . strtoupper(Str::random(5));

        $order = Order::create([
            'numero_orden' => $numeroOrden,
            'cliente_nombre' => $validated['cliente']['nombre'],
            'cliente_empresa' => $validated['cliente']['empresa'] ?? null,
            'cliente_rif' => $validated['cliente']['rif'] ?? null,
            'cliente_telefono' => $validated['cliente']['telefono'],
            'cliente_email' => $validated['cliente']['correo'] ?? null,
            'total_usd' => $validated['totalUsd'],
            'nota_adicional' => $validated['notaAdicional'] ?? null,
            'metodo_origen' => $validated['origen'] ?? 'web_orden_compra',
            'estado' => 'Pendiente',
        ]);

        foreach ($validated['items'] as $item) {
            OrderItem::create([
                'order_id' => $order->id,
                'product_id' => $item['productoId'] ?? null,
                'producto_nombre' => $item['nombre'],
                'talla' => $item['talla'] ?? 'Única',
                'cantidad' => $item['cantidad'],
                'precio_unitario' => $item['precioUnitario'],
                'subtotal' => $item['subtotal'] ?? ($item['cantidad'] * $item['precioUnitario']),
                'detalles_personalizacion' => $item['personalizacion'] ?? null,
            ]);
        }

        // 🔔 NOTIFICACIÓN EN TIEMPO REAL A TODOS LOS USUARIOS DE FILAMENT:
        $admins = User::all();
        Notification::make()
            ->title('¡Nueva Cotización Web Recibida!')
            ->body("Orden: {$numeroOrden} | Cliente: {$order->cliente_nombre} ({$order->cliente_empresa}) por $" . number_format($order->total_usd, 2))
            ->success()
            ->actions([
                \Filament\Notifications\Actions\Action::make('ver')
                    ->button()
                    ->url('/admin/orders/' . $order->id),
            ])
            ->sendToDatabase($admins);

        return response()->json([
            'success' => true,
            'orderNumber' => $numeroOrden,
            'message' => 'Cotización registrada exitosamente.',
        ], 201);
    }
}
```

---

## 6. WIDGETS DE FILAMENT PARA EL DASHBOARD

El compañero puede crear métricas visuales clave en el Dashboard de Filament ejecutando:
```bash
php artisan make:filament-widget StatsOverview --stats-overview
```

Ejemplo de implementación (`App\Filament\Widgets\StatsOverview.php`):
```php
namespace App\Filament\Widgets;

use App\Models\Product;
use App\Models\Order;
use Filament\Widgets\StatsOverviewWidget as BaseWidget;
use Filament\Widgets\StatsOverviewWidget\Stat;

class StatsOverview extends BaseWidget
{
    protected function getStats(): array
    {
        return [
            Stat::make('Total Productos', Product::count())
                ->description('En catálogo activo')
                ->icon('heroicon-o-archive-box')
                ->color('success'),

            Stat::make('Stock Crítico', Product::whereColumn('stock', '<=', 'stock_minimo')->count())
                ->description('Productos por agotarse')
                ->icon('heroicon-o-exclamation-triangle')
                ->color('danger'),

            Stat::make('Cotizaciones Pendientes', Order::where('estado', 'Pendiente')->count())
                ->description('Recibidas desde la web')
                ->icon('heroicon-o-clock')
                ->color('warning'),

            Stat::make('Total Cotizado este Mes', '$' . number_format(Order::whereMonth('created_at', now()->month)->sum('total_usd'), 2))
                ->description('Volumen B2B estimado')
                ->icon('heroicon-o-currency-dollar')
                ->color('info'),
        ];
    }
}
```

---

## 7. CONFIGURACIÓN CORS Y ALMACENAMIENTO DE FOTOS

### 7.1. Habilitar CORS en Laravel (`config/cors.php`)
Para que la página web pueda consultar los productos y enviar pedidos sin bloqueos del navegador:
```php
return [
    'paths' => ['api/*', 'sanctum/csrf-cookie'],
    'allowed_methods' => ['*'],
    'allowed_origins' => ['*'], // O especificar el dominio final: ['https://unifortex.com']
    'allowed_headers' => ['*'],
    'exposed_headers' => [],
    'max_age' => 0,
    'supports_credentials' => false,
];
```

### 7.2. Almacenamiento de Archivos (`.env` y `config/filesystems.php`)
Asegurarse de que en el archivo `.env` esté configurado:
```env
FILESYSTEM_DISK=public
APP_URL=http://tu-dominio-o-ip:8000
```
Y haber ejecutado:
```bash
php artisan storage:link
```
Esto creará un enlace simbólico entre `storage/app/public` y `public/storage`, permitiendo que las fotos subidas con Filament sean accesibles directamente vía URL.

---

## 8. RESUMEN DE VENTAJAS DE ESTA ARQUITECTURA

1. **Desarrollo Ultra Rápido**: Con Filament v3, tu compañero no tiene que diseñar el panel desde cero (formularios, tablas, subida de fotos, paginación, filtros y búsquedas vienen listos y estilizados con Tailwind).
2. **Cero Mantenimiento Manual del Frontend**: Cada vez que cambie un precio, foto o stock en Filament, el catálogo de la web se actualiza al instante.
3. **Notificaciones Nativas**: Las cotizaciones generadas en la web encienden la campana de notificaciones de Filament sin necesidad de servicios externos costosos.
4. **Escalabilidad y Seguridad**: Laravel cuenta con protección CSRF, Rate Limiting nativo, transacciones de base de datos seguras y roles de usuario con Filament Shield si se requiere.
