# Diccionario de datos

Los datos vienen de SmartBuy, la plataforma de compras del Banco basada en Coupa, y corresponden al reporte *Spend FA x Transit x User ID* que nos entregó el área de Procurement S2PA. El reporte trae todas las facturas de 10.000 CAD o más creadas entre el 01/06/2022 y el 20/07/2026, con una fila por factura. En total son 75.631 facturas y 27 atributos. El archivo es un Excel interno del Banco, así que no se publica en el repositorio, como se explica en el punto 3 del reporte.

## Atributos originales

En la tabla se describe cada atributo con su tipo, el porcentaje de nulos, el número de valores distintos, qué tan importante es para el proyecto y qué se hizo con él en el entendimiento de los datos.

| Atributo | Descripción | Tipo | % de nulos | Valores únicos | Importancia | Tratamiento |
|---|---|---|---|---|---|---|
| `Requested by Name` | Persona que pidió la compra o la factura | Texto (persona) | 16,0 | 2.848 | Media | Se cambia por un seudónimo (`EMP_xxxxx`) y se usa para saber si el solicitante también aprobó la factura |
| `User Login` | Usuario de red del solicitante | Texto (identificador) | 16,0 | 2.850 | Baja | Se elimina porque es un dato personal |
| `User ID` | Código numérico del solicitante en SmartBuy | Numérico (identificador) | 16,0 | 2.850 | Baja | Se elimina porque es un dato personal |
| `Finance Approver` | Primer aprobador financiero de la factura | Texto (persona) | 61,4 | 106 | Media | Se cambia por un seudónimo. Sus nulos dependen del proceso |
| `Finance Approver2` | Segundo aprobador financiero | Texto (persona) | 83,2 | 78 | Baja | Se cambia por un seudónimo |
| `Original Approver Name` | Aprobador original de la cadena de aprobación | Texto (persona) | 56,3 | 2.553 | Media | Se cambia por un seudónimo. Solo se llena cuando la factura termina de aprobarse, así que tiene fuga de información |
| `Number of Approvers` | Número de aprobadores de la factura | Cuantitativa discreta | 57,3 | 18 | Alta | El nulo quiere decir que no hay cadena de aprobación. Los valores en 0 que sí tienen aprobador se pasan a nulo |
| `Invoice Supplier Name` | Proveedor, escrito como NOMBRE_ENTIDAD_CÓDIGO | Texto compuesto | 0,1 | 3.481 | Alta | Se separa en la entidad, el código (que es la llave) y el nombre normalizado, y se cambia por un seudónimo (`PROV_xxxx`) |
| `Invoice Number` | Número de factura que pone el proveedor | Texto (identificador) | 0,4 | 75.343 | Alta | Se normaliza quitando espacios, signos y ceros a la izquierda para encontrar duplicados. No se publica |
| `Invoice Status` | Estado de la factura: approved, voided, disputed, rejected, `pending_*`, draft o new | Categórica nominal | 0,0 | 10 | Alta | Con este atributo se construye la fricción. Los estados sin resolver quedan por fuera de la ventana |
| `Invoice Created Date` | Fecha en que se creó la factura en SmartBuy | Fecha | 0,0 | 1.345 | Alta | Con ella se define la ventana de análisis, de septiembre de 2024 a mayo de 2026 |
| `Invoice Total with Tax Amount` | Monto total con impuestos en la moneda original | Cuantitativa continua | 0,0 | 50.461 | Alta | Se usa para encontrar montos idénticos dentro de un mismo proveedor |
| `Invoice Currency Code` | Código ISO de la moneda | Categórica nominal | 0,0 | 14 | Media | No tiene problemas. Hay 14 monedas |
| `Invoice Currency Name` | Nombre de la moneda | Categórica nominal | 0,0 | 14 | Baja | Repite la información del código, porque cada código tiene un solo nombre |
| `Contract Number` | Contrato asociado a la factura | Texto (identificador) | 63,6 | 1.420 | Media | El nulo quiere decir que la factura no tiene contrato, así que se crea la variable `tiene_contrato` |
| `Original Approver ID` | Código del aprobador original | Numérico (identificador) | 56,3 | 2.552 | Baja | Se elimina porque es un dato personal |
| `Commodity Level 1` | Categoría de compra de primer nivel, con 37 valores | Categórica nominal | 35,5 | 37 | Alta | Los nulos se reemplazan por la categoría `Sin categoría` |
| `Commodity Level 2` | Categoría de compra de segundo nivel | Categórica nominal | 35,9 | 57 | Media | Los nulos se reemplazan por `Sin categoría` |
| `Commodity Name` | Categoría de compra de tercer nivel | Categórica nominal | 35,5 | 327 | Baja | Los nulos se reemplazan por `Sin categoría` |
| `Account Segment 1` | Transit (el área del Banco, con 5 dígitos) o un código de proyecto | Texto (código) | 1,2 | 3.505 | Alta | Se lee como texto para no perder los ceros y se crea la bandera `seg1_es_transit` |
| `Account Segment 2` | Segmento contable 2, cuyo significado hay que confirmar con el Banco | Texto (código) | 1,6 | 94 | Baja | No se trata |
| `Account Segment 3` | Segmento contable 3, que podría ser el centro de costo (por confirmar) | Texto (código) | 4,4 | 873 | Media | No se trata |
| `Account Segment 4` | Segmento contable 4, que podría ser la cuenta contable (por confirmar) | Texto (código) | 39,5 | 640 | Baja | No se trata |
| `Creation Method` | Forma en que se creó la factura, por ejemplo csp_po_flip, buyer_load o enterprise_ui | Categórica nominal | 0,0 | 12 | Media | Es coherente con el canal, porque cada método pertenece a un solo canal |
| `Invoice Channel` | Canal por el que entró la factura: csp, sftp, ui, email, cxml o san | Categórica nominal | 0,0 | 6 | Alta | No tiene problemas y es una variable clave para las hipótesis 1 y 2 |
| `Invoice Created by Type` | Tipo de usuario que creó la factura | Categórica nominal | 26,8 | 3 | Baja | Está vacío siempre en email y ui, así que esos nulos se imputan como InternalUser |
| `Invoice total with tax (CAD)` | Monto total con impuestos convertido a CAD | Cuantitativa continua | 0,0 | 54.810 | Alta | Se corrigió la tasa de cambio en 3 facturas y un error de moneda pasó a nulo. El resultado queda en `monto_cad` |

## Variables creadas en el entendimiento de los datos

Durante el análisis se crearon estas variables a partir de los atributos originales.

| Variable | Qué significa |
|---|---|
| `proveedor_id` | Seudónimo de la llave del proveedor (entidad y código) |
| `grupo_nombre_id` | Seudónimo del nombre normalizado del proveedor, que sirve para juntar las llaves que son la misma empresa |
| `prov_intragrupo` | Indica si el proveedor es una empresa del mismo grupo del Banco |
| `solicitante_es_aprobador` | Indica si la persona que pidió la compra es la misma que aprobó la factura |
| `monto_cad` | Monto en CAD ya verificado, con la tasa corregida y el error de moneda en nulo |
| `tiene_contrato` | Indica si la factura tiene un contrato asociado |
| `sin_cadena_aprobacion` | Indica si la factura no tiene número de aprobadores |
| `seg1_es_transit` | Indica si `Account Segment 1` tiene el formato de un transit (5 dígitos) |
| `num_factura_norm` y `num_factura_colision` | El número de factura normalizado y una bandera que dice si choca con otra factura del mismo proveedor |
| `extremo_no_verificado` y `error_monto` | Banderas que salen de la revisión de los montos extremos |
| `con_friccion` | Indica si la factura terminó anulada, disputada o rechazada. Es una variable objetivo provisional |
| `candidato_duplicado` | Indica si una factura aprobada tiene otra del mismo proveedor con el mismo monto en 7 días o menos, o un número casi idéntico |
