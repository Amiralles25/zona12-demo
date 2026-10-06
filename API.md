# Zona12 — Referencia de API

Documento complementario de [`README.md`](README.md). Cubre todos los endpoints del backend Express, con autenticación, entrada esperada y función de cada ruta.

### Convenciones

- Base URL: `/api`. Las respuestas son JSON salvo `GET /uploads/encargos/:nombre`.
- **Usuario** requiere cookie `accessToken`; **Admin** requiere además rol vigente en base de datos.
- Zod valida body y query, elimina campos extra y devuelve `400` con `{ error, detalles }` cuando falla.
- Los `:id` son enteros positivos parseados de forma estricta. Errores relevantes: `401`, `403`, `404`, `409`, `429` y `500`.

### Auth

| Método y ruta | Auth | Entrada | Respuesta / errores | Función |
| --- | --- | --- | --- | --- |
| `POST /auth/registro` | No; bloqueado demo | email, password min. 8, nombre, apellidos | Verificación; `400`, `409`, `403` demo | Inicia registro |
| `POST /auth/registro/verificar` | No; bloqueado demo | email + código 6 dígitos | Usuario/sesión; `400`, `403` | Confirma registro |
| `POST /auth/login` | No | email + password | Cookies; `401` | Autentica |
| `POST /auth/refresh` | Refresh cookie | — | Cookies renovadas; `401` | Rota sesión |
| `POST /auth/logout` | Opcional | — | Cookies limpias | Cierra sesión |
| `POST /auth/recuperar` | No; bloqueado demo | email | Genérica; `400`, `429`, `403` | Envía recovery |
| `POST /auth/restablecer` | No; bloqueado demo | token + password nueva min. 8 | Éxito; `400`, `403` | Restablece contraseña |

### Productos y tallas

| Método y ruta | Auth | Entrada | Respuesta / errores | Función |
| --- | --- | --- | --- | --- |
| `GET /products` | No | — | Catálogo activo | Lista tienda |
| `GET /products/:id` | No | id positivo | Producto; `400`, `404` | Ficha |
| `POST /products` | Admin | Producto, flags, tallas | `201`; `400` | Crea |
| `PATCH /products/:id` | Admin | Campos parciales | Actualizado; `400`, `404` | Edita metadatos |
| `PATCH /products/:id/stock` | Admin | Stock validado | Actualizado | Ajusta stock |
| `DELETE /products/:id` | Admin | id | Éxito; `404` | Elimina |
| `GET /products/admin/all` | Admin | búsqueda, filtros, orden, paginación | Lista admin | Opera catálogo |
| `GET /products/admin/:id` | Admin | id | Producto editable | Detalle admin |
| `PUT /products/admin/:id/tallas` | Admin | XS–XXL, fan/player/player_larga, stock ≥ 0 | Matriz actualizada | Mantiene variantes |
| `PUT /products/admin/:id/imagenes` | Admin | Hasta 20: tipo, orden, URL segura | Galería actualizada | Gestiona imágenes |
| `GET /tallas` | No | — | Guía de tallas | Datos de guía |

### Pedidos y checkout

> **Nota:** `GET /orders/admin/all` y `GET /orders/admin/:id` están declarados después de `GET /orders/:id` en el código. El parser numérico de la ruta genérica oculta esos `GET` admin y responde `400` antes de llegar al handler. Se documentan porque existen y su lógica es correcta, pero requieren reordenarse en el router (ver [Limitaciones conocidas](README.md#limitaciones-conocidas) del README).

| Método y ruta | Auth | Entrada | Respuesta / errores | Función |
| --- | --- | --- | --- | --- |
| `POST /orders/verificacion-telefono` | No/opcional | `telefono_contacto` 9–20 | Estado SMS; `429`, `502` | Verifica invitado contra reembolso; demo puede incluir `codigo_demo` |
| `POST /orders` | Invitado/usuario | Líneas, método, dirección/recogida, datos invitado/código según caso | `201`; `400`, `401`, `409`, `429` | Calcula y crea pedido en transacción |
| `GET /orders` | Usuario | — | Lista propia | Pedidos del usuario |
| `GET /orders/:id` | Propietario/token | id + token público cuando aplique | `401`, `403`, `404` | Detalle seguro |
| `GET /orders/admin/all` | Admin | búsqueda, estado, método, fechas, orden, página | Lista | Cola declarada |
| `GET /orders/admin/:id` | Admin | id | Detalle | Ficha declarada |
| `PATCH /orders/admin/:id/estado` | Admin | Estado permitido | Pedido actualizado | Transiciona y ajusta stock |
| `PATCH /orders/admin/:id/seguimiento` | Admin | tracking máx. 100 | Actualizado | Mantiene tracking |

### Usuarios, direcciones y favoritos

| Método y ruta | Auth | Entrada | Función |
| --- | --- | --- | --- |
| `GET /users/me` / `PATCH /users/me` | Usuario | — / nombre y apellidos | Consulta/edita perfil |
| `PATCH /users/me/password` | Usuario | actual + nueva min. 8 | Cambia password e invalida otras sesiones; demo `403` |
| `GET/POST /users/me/direcciones` | Usuario | — / dirección validada | Lista/crea libreta |
| `PATCH/DELETE /users/me/direcciones/:id` | Usuario | Campos parciales / id | Edita o elimina; archiva si tiene pedidos históricos |
| `PATCH /users/me/direcciones/:id/principal` | Usuario | id | Marca dirección principal |
| `GET /users` / `GET /users/admin/:id` | Admin | filtros/página / id | Lista segura y detalle de cliente |
| `GET /favoritos/me` / `GET /favoritos/me/productos` | Usuario | — | IDs o productos favoritos completos |
| `POST/DELETE /favoritos/:productoId` | Usuario | id | Añade idempotentemente / elimina |
| `GET /favoritos/admin/estadisticas` | Admin | `limite` 1–50 | Ranking de demanda |

### Encargos

> **Nota:** `GET /encargos/admin` y `GET /encargos/admin/:id` se declaran después de `GET /encargos/:id`; el mismo problema de orden de rutas que en `orders` oculta esos `GET` administrativos hasta reordenarlos.

| Método y ruta | Auth | Entrada | Función |
| --- | --- | --- | --- |
| `POST /encargos` | Invitado/usuario | `multipart`: equipo, competición, talla, teléfono; identidad/email invitado; imagen JPG/PNG/WebP ≤ 5 MB opcional | Crea encargo y notifica |
| `GET /encargos/:id` | Propietario/token | id + token si aplica | Detalle seguro |
| `GET /encargos/admin` / `GET /encargos/admin/:id` | Admin | filtros/página / id | Cola y ficha declaradas |
| `PATCH /encargos/admin/:id/estado` | Admin | Estado válido | Cambia ciclo y libera cupo de imagen al cerrar |
| `GET /uploads/encargos/:nombre` | URL opaca | Nombre aleatorio | Sirve bytes desde BD, no un explorador de archivos |

### Recogida, configuración y métricas

| Método y ruta | Auth | Entrada | Función |
| --- | --- | --- | --- |
| `GET /recogida/disponibilidad` | No | — | Fechas disponibles |
| `GET /recogida/franjas` | No | `fecha=YYYY-MM-DD` | Franjas y plazas restantes |
| `GET/POST /recogida/admin/franjas` | Admin | — / día, etiqueta, activa | Lista/crea franjas; duplicado `409` |
| `PATCH/DELETE /recogida/admin/franjas/:id` | Admin | activa / id | Activa, desactiva o elimina si no hay pedidos vivos |
| `POST /recogida/admin/disponibilidad` | Admin | fecha, disponible, motivo | Upsert de cierre/apertura |
| `GET /recogida/admin/pedidos` | Admin | fecha | Recogidas del día |
| `GET/PATCH /config/envio` | No/Admin | — / configuración | Consulta/edita envío |
| `GET/PATCH /config/recargos` | No/Admin | — / importes ≥ 0 | Consulta/edita personalización, parches y versiones |
| `GET /config/demo` | No | — | Estado demo y credenciales solo si están completas |
| `GET /admin/metricas` | Admin | — | Dashboard |
| `GET /admin/ventas`, `/admin/horas` | Admin | `dias` 7–365 | Series de ventas |
| `GET /admin/actividad`, `/admin/top-productos` | Admin | `limite` 1–50 | Actividad y ranking |
| `GET /admin/proveedor` | Admin | — | Cola de proveedor |
| `GET /health` | No | — | `{ status: "ok" }` |
