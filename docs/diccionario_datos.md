# Diccionario de datos

**Fuente:** SmartBuy (Coupa), reporte *Spend FA x Transit x User ID*, entregado por Procurement S2PA del Banco.  
**Cobertura:** facturas con monto ≥ 10.000 CAD creadas entre el 01/06/2022 y el 20/07/2026.  
**Tamaño:** 75.631 facturas (una fila por factura) × 27 atributos.  
**Acceso:** archivo Excel interno; no se publica en el repositorio (ver punto 3 del reporte).

## Atributos originales

| Atributo | Descripción | Tipo | % nulos | Valores únicos | Importancia | Tratamiento |
|---|---|---|---|---|---|---|
| `Requested by Name` | Persona que solicitó la compra/factura | Texto (persona) | 16.0 | 2.848 | Media | Seudonimizado como EMP_xxxxx; se usa para la bandera solicitante = aprobador |
| `User Login` | Usuario de red del solicitante | Texto (identificador) | 16.0 | 2.850 | Baja | Eliminado (dato personal) |
| `User ID` | ID numérico del solicitante en SmartBuy | Numérico (identificador) | 16.0 | 2.850 | Baja | Eliminado (dato personal) |
| `Finance Approver` | Primer aprobador financiero de la factura | Texto (persona) | 61.4 | 106 | Media | Seudonimizado; nulo según el proceso |
| `Finance Approver2` | Segundo aprobador financiero | Texto (persona) | 83.2 | 78 | Baja | Seudonimizado |
| `Original Approver Name` | Aprobador original de la cadena de aprobación | Texto (persona) | 56.3 | 2.553 | Media | Seudonimizado; se llena solo si la factura completa la aprobación (fuga de información) |
| `Number of Approvers` | Número de aprobadores de la factura | Cuantitativa discreta | 57.3 | 18 | Alta | Nulo = sin cadena de aprobación; valores 0 con aprobador → nulo |
| `Invoice Supplier Name` | Proveedor en formato NOMBRE_ENTIDAD_CÓDIGO | Texto compuesto | 0.1 | 3.481 | Alta | Se separa en entidad + código (llave) y nombre normalizado; seudonimizado como PROV_xxxx |
| `Invoice Number` | Número de factura asignado por el proveedor | Texto (identificador) | 0.4 | 75.343 | Alta | Se normaliza (sin espacios, guiones, ceros a la izquierda) para detectar duplicados; no se publica |
| `Invoice Status` | Estado de la factura (approved, voided, disputed, rejected, pending_*, draft, new) | Categórica nominal | 0.0 | 10 | Alta | Base de la variable de fricción; estados sin resolver fuera de la ventana |
| `Invoice Created Date` | Fecha de creación de la factura en SmartBuy | Fecha | 0.0 | 1.345 | Alta | Define la ventana analítica septiembre de 2024 a mayo de 2026 |
| `Invoice Total with Tax Amount` | Monto total con impuestos en la moneda original | Cuantitativa continua | 0.0 | 50.461 | Alta | Se usa para comparar montos idénticos dentro del proveedor |
| `Invoice Currency Code` | Código ISO de la moneda | Categórica nominal | 0.0 | 14 | Media | Sin problemas; 14 monedas |
| `Invoice Currency Name` | Nombre de la moneda | Categórica nominal | 0.0 | 14 | Baja | Redundante con el código (relación 1 a 1) |
| `Contract Number` | Contrato asociado a la factura | Texto (identificador) | 63.6 | 1.420 | Media | Nulo = sin contrato → variable tiene_contrato |
| `Original Approver ID` | ID del aprobador original | Numérico (identificador) | 56.3 | 2.552 | Baja | Eliminado (dato personal) |
| `Commodity Level 1` | Categoría de compra, nivel 1 (37 valores) | Categórica nominal | 35.5 | 37 | Alta | Nulo → 'Sin categoría' |
| `Commodity Level 2` | Categoría de compra, nivel 2 | Categórica nominal | 35.9 | 57 | Media | Nulo → 'Sin categoría' |
| `Commodity Name` | Categoría de compra, nivel 3 | Categórica nominal | 35.5 | 327 | Baja | Nulo → 'Sin categoría' |
| `Account Segment 1` | Transit (área, 5 dígitos) o código de proyecto | Texto (código) | 1.2 | 3.505 | Alta | Bandera seg1_es_transit; se lee como texto para conservar ceros |
| `Account Segment 2` | Segmento contable 2 (significado por confirmar con el Banco) | Texto (código) | 1.6 | 94 | Baja | Sin tratamiento |
| `Account Segment 3` | Segmento contable 3 (posible centro de costo; por confirmar) | Texto (código) | 4.4 | 873 | Media | Sin tratamiento |
| `Account Segment 4` | Segmento contable 4 (posible cuenta contable; por confirmar) | Texto (código) | 39.5 | 640 | Baja | Sin tratamiento |
| `Creation Method` | Mecanismo de creación (csp_po_flip, buyer_load, enterprise_ui, ...) | Categórica nominal | 0.0 | 12 | Media | Consistente con el canal (jerarquía exacta) |
| `Invoice Channel` | Canal de ingreso (csp, sftp, ui, email, cxml, san) | Categórica nominal | 0.0 | 6 | Alta | Sin problemas; variable clave de H1 y H2 |
| `Invoice Created by Type` | Tipo de usuario que creó la factura | Categórica nominal | 26.8 | 3 | Baja | Nulo en email/ui → imputado como InternalUser |
| `Invoice total with tax (CAD)` | Monto total con impuestos convertido a CAD | Cuantitativa continua | 0.0 | 54.810 | Alta | Corregido en 3 facturas (tasa de cambio) y 1 error de moneda a nulo → monto_cad |

## Variables derivadas en el entendimiento de los datos

| Variable | Definición |
|---|---|
| `proveedor_id` | Seudónimo de la llave entidad + código del proveedor |
| `grupo_nombre_id` | Seudónimo del nombre normalizado (para consolidar proveedores duplicados) |
| `prov_intragrupo` | El proveedor es una entidad del mismo grupo del Banco |
| `solicitante_es_aprobador` | El solicitante coincide con el aprobador original |
| `monto_cad` | Monto en CAD verificado (tasa corregida; error de moneda a nulo) |
| `tiene_contrato` | La factura tiene contrato asociado |
| `sin_cadena_aprobacion` | La factura no tiene número de aprobadores |
| `seg1_es_transit` | `Account Segment 1` tiene formato de transit (5 dígitos) |
| `num_factura_norm` / `num_factura_colision` | Número normalizado y bandera de colisión con otra factura del mismo proveedor |
| `extremo_no_verificado` / `error_monto` | Banderas de la verificación de montos extremos |
| `con_friccion` | Factura anulada, disputada o rechazada (variable objetivo provisional) |
| `candidato_duplicado` | Aprobada con otra del mismo proveedor y monto en ≤ 7 días o con número casi idéntico |
