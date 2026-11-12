# Guía breve de integración UX/Frontend con Backend

## Contexto actual

- El frontend es un prototipo independiente en `ux/prototype/cajaclara-ui/`, hecho con Vue 3, Vue Router y Vite. Sus vistas y componentes están implementados, pero los datos provienen de mocks locales y no persisten.
- Django y la app `finanzas` viven en la raíz del repositorio. Los modelos actuales son `Categoria`, `Movimiento` y `CuentaPorCobrar`.
- El único endpoint disponible es `GET /api/finanzas/resumen-mensual/`. Devuelve `mes`, `anio`, `total_ingresos`, `total_gastos` y `flujo_caja` para el periodo actual. Aún no acepta periodo como parámetro ni filtra por el usuario autenticado; no debe considerarse listo para producción.
- Las pantallas principales del prototipo son `/` (dashboard), `/operaciones`, `/cobros` y `/resumen`. La vista espera más detalle que el entregado por el endpoint actual.

## Acuerdos necesarios antes de conectar

1. **Resumen:** acordar filtros de periodo (`mes`/`anio` o fechas), zona horaria `America/Santiago` y una respuesta que alimente tanto los indicadores como el resumen por semana. El endpoint actual solo cubre los totales del mes en curso.
2. **Operaciones:** definir endpoints y contratos para listar y registrar ventas pagadas, ventas pendientes, compras y otros gastos. El formulario puede incluir cliente/proveedor, productos y cantidades, categoría, canal, fecha, monto y documento; confirmar qué campos y tipos de operación soportará el MVP.
3. **Cuentas por cobrar:** definir listado/detalle, creación asociada a una venta pendiente y acción de pago completo. Acordar cómo se calcula `VENCIDA` y qué fecha se considera vencimiento.
4. **Datos compartidos del dashboard:** confirmar si productos destacados, actividad reciente y cobros prioritarios se incluirán en el resumen o tendrán endpoints separados.
5. **Contrato común:** usar identificadores estables, fechas ISO (`YYYY-MM-DD`), montos CLP enteros (sin decimales) y estados acordados. DRF puede entregar `DecimalField` como texto; el frontend debe convertir montos a número antes de calcular y formatearlos con `es-CL`.
6. **Acceso y errores:** definir autenticación y formato de errores. En desarrollo, configurar CORS para el origen del Vite; restringir orígenes antes de producción.

## Pasos de integración sugeridos

1. Integrar la rama `ux` a `main` preservando el frontend en `ux/prototype/cajaclara-ui/`; no mover ni reemplazar el proyecto Django de la raíz.
2. Acordar primero los contratos y priorizar el endpoint de resumen mensual. Configurar `VITE_API_BASE_URL` (por ejemplo, `http://localhost:8000/api`) en el entorno local, sin fijar URLs de producción en el código.
3. Sustituir mocks gradualmente por llamadas a la API, manteniendo el formato de respuesta detrás de una capa de adaptación para no acoplar los componentes directamente al modelo Django.
4. Añadir y validar estados de carga, error, vacío y éxito; no mostrar datos mock como si fueran datos persistidos.
5. Verificar los dos proyectos: `npm run build` desde `ux/prototype/cajaclara-ui/`; en la raíz, ejecutar migraciones y pruebas Django. Comprobar navegación responsive, formularios, filtros y que las operaciones sobrevivan una recarga cuando estén conectadas.

## Límite importante del modelo actual

`Movimiento` representa monto, categoría, fecha y descripción, pero no modela por sí solo el detalle de una venta con varios productos, clientes, proveedores o pagos. `CuentaPorCobrar` no guarda un historial de pagos. No asumir que las pantallas de operaciones y cobros pueden conectarse completamente sin acordar o ampliar esos contratos/modelos.
