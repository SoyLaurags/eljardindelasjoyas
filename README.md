# eljardindelasjoyas
Página de control de estocaje tanto en almacén como en tienda
# El Jardín · Stock

Aplicación **privada** de control de stock para [El Jardín Atelier](https://eljardindelasjoyas.lovable.app).
Interfaz en español, pensada primero para el móvil y con dos únicas ubicaciones: **Almacén** y **Tienda**.

- Todo el stock nuevo entra en el **Almacén**.
- Pasar una pieza a **Tienda** no cambia el total; venderla o darle de baja sí lo reduce.
- Ninguna operación deja cantidades negativas.
- Cada movimiento queda registrado con el antes y el después de ambas ubicaciones, quién lo hizo y cuándo.

---

## Índice

- [Pantallas](#pantallas)
- [Roles y permisos](#roles-y-permisos)
- [Cómo funciona el stock](#cómo-funciona-el-stock)
- [Precios y ventas](#precios-y-ventas)
- [Tecnologías](#tecnologías)
- [Modelo de datos](#modelo-de-datos)
- [Estructura del proyecto](#estructura-del-proyecto)
- [Puesta en marcha](#puesta-en-marcha)
- [Pruebas](#pruebas)
- [Convenciones](#convenciones)
- [Notas](#notas)

---

## Pantallas

| Ruta | Qué hace |
| --- | --- |
| `/auth` | Entrar con correo y contraseña, y recuperar la contraseña. |
| `/inicio` | Portada: Collares y Pendientes arriba, Pulseras y Anillos abajo, Resina y el resto de accesos centrados. |
| `/seccion/:section` | Listado de una familia. Buscador por nombre o referencia, contadores por joya y acciones rápidas. |
| `/resina` | Familias de resina: Collares y Pulseras, Anillos y Pendientes, y Materiales centrado. |
| `/nuevo` | Alta de una joya: foto, nombre, referencia, familia, unidades iniciales de almacén, precios y aviso de stock bajo. |
| `/ficha/:id` | Detalle de una joya: Tienda, Almacén, Total stock y Total vendidos, precios, movimientos, ventas y historial. |
| `/resumen` | Resumen de ventas por mes, trimestre, semestre, año, todo el histórico o fechas concretas. Ranking de joyas más vendidas, importe y quién vendió cada pieza. |
| `/reposicion` | Vista rápida de stock bajo, agotados en tienda, almacén vacío y joyas sin aviso, con reposición ágil. |
| `/descatalogados` | Joyas retiradas del catálogo. Conservan sus ventas y su historial y se pueden reactivar. |
| `/equipo` | Solo la dueña: dar de alta personas con su nombre, correo y contraseña, y cambiar nombres existentes. |

**Cabecera.** Está siempre disponible el botón dorado **«Stock»**: abre una ventana con el buscador rápido para sumar unidades al almacén de una joya que ya existe o para crear la ficha de una nueva. Con el icono de las flechas se amplía a pantalla completa.

---

## Roles y permisos

Los roles se guardan en `user_roles` y se asignan automáticamente al crear la cuenta.

| | Dueña (`admin`) | Equipo (`equipo`) |
| --- | --- | --- |
| Ver joyas, stock e historial | Sí | Sí |
| Crear fichas y añadir stock | Sí | Sí |
| Trasladar, vender, anular ventas y dar bajas | Sí | Sí |
| Activar o cambiar el aviso de stock bajo | Sí | Sí |
| Corregir stock a mano | Sí | No |
| Descatalogar o reactivar, y borrar | Sí | No |
| Gestionar el equipo | Sí | No |

El **registro público está cerrado**: las cuentas se crean desde el panel de usuarios de Lovable Cloud o desde la pantalla **Equipo**.

---

## Cómo funciona el stock

| Movimiento | Efecto |
| --- | --- |
| `entrada` | Sube el Almacén y el total. |
| `traslado_a_tienda` | Almacén → Tienda. El total no cambia. |
| `devolucion_almacen` | Tienda → Almacén. El total no cambia. |
| `venta` | Baja la Tienda y el total. Pide motivo si no hay precio registrado. |
| `venta_anulada` | Devuelve la unidad a Tienda y suma el total otra vez. |
| `baja` | Baja la Tienda y el total. Motivo obligatorio. |
| `correccion` | Fija valores de almacén y tienda a mano. Solo la dueña, con motivo. |

El aviso de **stock bajo** se decide **por joya**: se activa escribiendo un número en su ficha y cuenta el **total** (Almacén + Tienda). Si el campo queda vacío, la joya no avisa. El aviso de **agotado en tienda** salta cuando en Tienda queda a 0.

---

## Precios y ventas

- Cada joya puede tener **precio de coste** y **precio de venta**; la ficha muestra el margen entre ambos.
- Al registrar una venta se abre una ventana con tres opciones: **mantener el precio habitual**, **aplicar un descuento en porcentaje** o **escribir el precio final**.
- El importe queda en el movimiento (`unit_price` y `discount_percent`), y se ve en el historial, en «Ventas de esta joya» y en el Resumen de ventas, que suma el **importe vendido** del periodo.
- Las ventas anuladas restan su importe.
- Los precios solo pueden guardarse en los 10 minutos siguientes a registrar la venta y por la misma persona que la registró.

---

## Tecnologías

- [TanStack Start](https://tanstack.com/start) (React 19 + Vite) con enrutado por archivos.
- TanStack Router y TanStack Query.
- Tailwind CSS v4 con tokens de color propios (fondo blanco, mosaicos beige, verde salvia, granate, mostaza y dorado suave).
- Componentes sobre Radix UI (shadcn/ui) y lucide-react.
- Zod para validar las entradas.
- Lovable Cloud para usuarios, base de datos y almacenamiento de fotos.
- Vitest y Testing Library para las pruebas.

---

## Modelo de datos

**Tablas**

| Tabla | Contenido |
| --- | --- |
| `profiles` | Nombre y correo de cada persona. |
| `user_roles` | Rol de cada cuenta (`admin` / `equipo`). |
| `products` | Ficha de la joya: nombre, familia, referencia, foto, unidades, precio de coste y de venta, aviso de stock bajo y si está descatalogada. |
| `stock_movements` | Historial completo: tipo, cantidad, antes y después de Almacén y Tienda, motivo, precio, descuento, autor y fecha. |

**Funciones de base de datos** (todas `security definer`, con RLS activado en las tablas)

| Función | Para qué sirve |
| --- | --- |
| `apply_stock_movement` | Aplica cualquier movimiento de forma atómica, comprueba que no queden negativos y escribe el historial. |
| `create_product` | Crea la ficha y su entrada inicial de almacén en la misma operación. |
| `set_product_threshold` | Activa o cambia el aviso de stock bajo de una joya. |
| `set_product_prices` | Guarda el precio de coste y el de venta. |
| `set_movement_price` | Guarda el precio final y el descuento de una venta recién registrada. |
| `set_product_archived` | Descataloga o reactiva una joya (solo la dueña). |
| `has_role` / `is_member` | Consultan el rol de la persona conectada. |
| `match_products` | Búsqueda por parecido visual (desactivada por ahora). |

Las fotos se guardan en un cajón **privado** y se muestran con enlaces temporales firmados.

Las migraciones están en `drizzle/migrations`, numeradas de `0000` a `0009`: tipos y tablas base, umbral global (luego retirado), venta anulada, aviso de stock bajo por joya, precios de producto, precios de venta y los cambios de correo de la dueña.

---

## Estructura del proyecto

```text
src/
├─ routes/                pantallas (enrutado por archivos)
│  ├─ __root.tsx          shell común
│  ├─ auth.tsx            entrar / recuperar contraseña
│  └─ _authenticated/     pantallas que piden sesión
├─ components/            piezas de interfaz
│  ├─ AppShell.tsx        cabecera y navegación
│  ├─ QuickStoreTransfer.tsx  ventas y traslados rápidos por joya
│  ├─ QuickStockEntry.tsx buscador de «añadir stock o joya nueva»
│  ├─ HeaderStockButton.tsx   botón «Stock» de la cabecera
│  ├─ StockBadges.tsx     Tienda, Almacén, Total stock y Total vendidos
│  ├─ PriceControl.tsx    precio de coste y de venta
│  ├─ SaleDialog.tsx      precio final o descuento al vender
│  ├─ LowStockControl.tsx aviso de stock bajo por joya
│  ├─ ProductSales.tsx    ventas de una joya por año y mes
│  └─ PhotoZoom.tsx       foto ampliada
├─ lib/                   reglas de negocio puras
│  ├─ stock-rules.ts      preview de movimientos y estado de tienda
│  ├─ sale-price.ts       precio final, descuento e importe
│  ├─ sales-period.ts     periodos y ranking de ventas
│  ├─ sections.ts         familias de joyas y de resina
│  └─ team.functions.ts   gestión del equipo (servidor)
├─ hooks/use-session.ts   persona conectada, rol y fotos
└─ integrations/supabase/ cliente generado de Lovable Cloud (no tocar)
```

---

## Puesta en marcha

Necesitas **Node.js 20+** y **bun** (o npm).

```sh
bun install
bun run dev      # http://localhost:8080
```

La aplicación necesita las credenciales del backend de Lovable Cloud. En este proyecto las aporta la propia plataforma y se recogen en `.env` con estos nombres:

```text
VITE_SUPABASE_URL=...
VITE_SUPABASE_PUBLISHABLE_KEY=...
VITE_SUPABASE_PROJECT_ID=...
```

Ese archivo está en el repositorio y solo lleva valores públicos: la dirección del backend y su clave de lectura, las mismas que usa el navegador de cualquiera que entre en la app. Las claves privadas nunca se guardan ahí.

Para entrar hay que tener cuenta: la dueña se crea desde el panel de usuarios de Lovable Cloud, y el resto del equipo desde la pantalla **Equipo** o desde ese mismo panel.

```sh
bun run build     # compilar
bun run preview   # servir la compilación
bun run lint      # reglas de código
bun run format    # dar formato
```

---

## Pruebas

```sh
bun run test      # todas
bunx vitest run src/lib/stock-rules.test.ts
```

| Archivo | Qué comprueba |
| --- | --- |
| `src/lib/stock-rules.test.ts` | Que ningún movimiento deje negativos, que el traslado no altere el total y que el aviso de stock bajo cuente el total. |
| `src/lib/sale-price.test.ts` | Precio mantenido, descuento en porcentaje y precio final, e importe con ventas anuladas. |
| `src/lib/sales-period.test.ts` | Los rangos de mes, trimestre, semestre, año, fechas a medida y histórico completo. |
| `src/lib/visual-search-policy.test.ts` | Que la búsqueda por parecido visual siga apagada. |
| `src/test/app-routing.test.tsx` | Que las rutas principales existan. |

---

## Convenciones

Están recogidas en [`AGENTS.md`](AGENTS.md). Las más importantes:

- El stock **solo** cambia con `apply_stock_movement` o `create_product`; nunca se tocan las columnas de cantidades a mano. Así el historial y la regla de no negativos siguen siendo la única referencia.
- El aviso de stock bajo es **por joya** (`products.low_stock_threshold`, vacío = sin aviso). `app_settings` está retirado.
- Los roles viven en `user_roles`, nunca en el perfil de la persona.
- Las fotos son privadas y se enseñan con enlaces firmados.
- Las joyas se muestran siempre como **filas alargadas de ancho completo** (móvil, tablet y escritorio), nunca como tarjetas.
- Los archivos de `src/integrations/supabase` y `.env` se generan: no se editan a mano.

---

## Notas

- **Búsqueda por foto:** desactivada. El código de preparación de imágenes y la función `match_products` se conservan para poder activarla más adelante.
- **Datos de ejemplo:** hay 16 joyas inventadas con foto, precios y unos 8 meses de historial para poder probar el Resumen de ventas y Reposición. Se pueden quitar cuando se quiera.
- **En producción:** https://eljardindelasjoyas.lovable.app

---

Proyecto creado y mantenido con [Lovable](https://lovable.dev).
