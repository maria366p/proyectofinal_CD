# Detección de pagos y proveedores duplicados en las compras del Banco

Proyecto final, entrega 1 · Universidad de los Andes

## Integrantes

| Nombre | Código |
|---|---|
| María Paula Ospina | 202123208 |
| [Integrante 2] | [código] |
| [Integrante 3] | [código] |
| [Integrante 4] | [código] |

## Alcance

El área de Procurement S2PA del Banco revisa las facturas de proveedores de forma manual y después del pago. El proyecto construye un producto de datos con tres componentes:

1. Un **modelo** que asigna a cada factura aprobada un puntaje de probabilidad de pago duplicado y agrupa los proveedores que probablemente son la misma empresa.
2. Una **API REST** que expone los puntajes y los casos.
3. Un **dashboard** con la vista del gasto consolidado y la cola priorizada de casos.

La **entrega 1** cubre los puntos 1 a 7: problema, ideación, ética, enfoque analítico, recolección, entendimiento de los datos y conclusiones iniciales.

## Conclusiones principales (entrega 1)

* **Cambio de régimen en agosto de 2024.** El volumen de facturas se triplica, aparece la carga por sftp y la proporción de facturas con contrato pasa de estar entre 65% y 82% a cerca de 25%. Los años no son comparables, así que el análisis usa la ventana septiembre de 2024 a mayo de 2026.
* **Hay duplicados que evaden el control de SmartBuy.** Seis grupos de facturas aprobadas tienen un número casi idéntico (solo cambia un guion o un espacio) y el mismo monto, por un total de 120.379 CAD.
* **Mismo monto no es duplicado.** Hay picos de recurrencia en 30, 91 y 365 días, así que el modelo debe aprender la periodicidad de cada proveedor.
* **Los campos de aprobación filtran el resultado.** El número de aprobadores falta en el 99,3% de las facturas anuladas, porque se llena al aprobar. No sirve para predecir la fricción.
* **El gasto está muy concentrado.** 150 proveedores (5,4%) explican el 80% del gasto y los 10 más grandes el 33,0%. Además, 22 nombres de proveedor tienen varias llaves en la misma entidad.
* **Hipótesis:** H4 (montos atípicos) se apoya; H1 se apoya en duplicados; H2 se apoya parcialmente; H3 no se apoya.
* **Suficiencia de los datos.** Alcanzan para la vista del gasto: los proveedores con historia cubren el 93,1% del gasto. Para evaluar el modelo falta la etiqueta; se propone que el Banco etiquete una muestra estratificada de 202 casos.

## Organización del repositorio

| Ruta | Contenido |
|---|---|
| `README.md` | Este archivo |
| `USO_IA.md` | Declaración del uso de IA: herramientas, prompts exactos y cómo se verificó cada parte |
| `requirements.txt` | Dependencias de Python |
| `.gitignore` | Excluye `data/`, porque los datos del Banco no se publican |
| `data/raw/` | `Spend_FA_x_Transit_x_User_ID.xlsx`, la extracción de SmartBuy (no se sube al repositorio) |
| `data/processed/` | `facturas_anonimizadas.parquet`, que genera el notebook (no se sube al repositorio) |
| `notebooks/01_entendimiento_datos.ipynb` | Punto 6: EDA, calidad y suficiencia de los datos, ejecutado de principio a fin |
| `notebooks/01_entendimiento_datos.html` | El mismo notebook en HTML, para leerlo sin Jupyter |
| `docs/diccionario_datos.md` | Punto 5: diccionario de los 27 atributos y de las variables derivadas |
| `docs/Reporte_entrega1.docx` |Word con el informe de la entrega|
| `docs/Reporte_entrega1.PDF` |PDF con el informe de la entrega|
| `docs/mockup/mockup.html` | Fuente HTML del mockup del tablero |
| `docs/img/` | `mockup.png`, `fig_punto6.png` y `fig_hipotesis.png` |
| `presentacion/Presentacion_Entrega1.pdf` | La misma presentación en PDF |


**Sustentación en video (máximo 6 minutos):** [enlace a YouTube] para ver el video en youtube

## Instrucciones de ejecución

1. Crear un entorno con Python 3.11 o superior e instalar las dependencias:
   bash
   pip install -r requirements.txt
2. Copiar el archivo entregado por el Banco en `data/raw/Spend_FA_x_Transit_x_User_ID.xlsx`. No se publica por confidencialidad; ver el punto 3 del reporte.
3. Ejecutar el notebook de principio a fin. También se puede abrir en Jupyter y correr todas las celdas.

   bash
   cd notebooks
   jupyter nbconvert --to notebook --execute --inplace 01_entendimiento_datos.ipynb

   El notebook genera `data/processed/facturas_anonimizadas.parquet`. Parquet se usa porque es más eficiente cuando tenemos volumenes grandes de información

**Orden de ejecución:** correr el notebook. El notebook corre en unos 35 segundos y no requiere conexión a internet.

## Dependencias

Python 3.13 (probado), pandas 3.0, numpy 2.5, matplotlib 3.11, seaborn 0.13, scipy 1.18, statsmodels 0.15, scikit-learn 1.9, levenshtein 0.27, openpyxl 3.1, pyarrow y jupyter/nbconvert. Las versiones exactas están en `requirements.txt`.

## Confidencialidad

* Los datos son información interna del Banco y no se incluyen en este repositorio público.
* El notebook reemplaza los nombres de personas y proveedores por seudónimos (`EMP_xxxxx`, `PROV_xxxx`) y elimina `User Login`, `User ID` y `Original Approver ID`.
* Los números de factura solo se muestran como patrón (letra = A, dígito = 9).
