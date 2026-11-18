# PROMPT 3 — "Reporte Artículo x Sucursal - Habilitación de Surtido"
### Versión autónoma (sin NetSuite / sin MCP — solo los dos Excel adjuntos)

## CONTEXTO DE NEGOCIO

Central Oeste es una cadena de farmacias (Argentina) que usa NetSuite como ERP. La
pregunta de negocio que resuelve este reporte es: "¿este artículo puede venderse/estar
en stock en esta sucursal, o no?" (eso es lo que acá se llama "habilitación de
surtido"). Para responderla hay que cruzar dos cosas:

1. El NIVEL DE SURTIDO PROPIO de cada artículo (una clasificación que indica qué tan
   masiva es la distribución prevista de ese artículo: Ampliado, Medio, Troncal, o los
   niveles especiales Exclusivo Ecommerce, X o No Aplica).
2. El NIVEL DE SURTIDO REQUERIDO por cada sucursal para la "categoría de implantación"
   a la que pertenece ese artículo (una configuración por combinación categoría+tienda).

**NO TENÉS ACCESO A NETSUITE NI A NINGÚN CONECTOR.** Vas a trabajar ÚNICAMENTE con los
DOS archivos Excel que se adjuntan a este mensaje. No inventes, no completes ni asumas
datos que no estén en esos archivos.

## ARCHIVO 1 — ÍTEMS

Nombre esperado: algo como "VistabásicadeArtículo...xlsx" (si difiere, identificalo por
estas columnas). Una hoja de datos, encabezado en la primera fila, luego una fila por
artículo (aprox. 34.600 filas en la última exportación). Columnas, en este orden:

1. ID interno (entero, identificador único del artículo)
2. Nombre — ⚠ OJO: a pesar del nombre de la columna, este campo es en realidad el
   CÓDIGO/SKU del artículo (lo que en otros reportes se llama "Código").
3. Nombre para mostrar — ⚠ OJO: este es el que en realidad es el NOMBRE descriptivo del
   artículo (lo que en otros reportes se llama "Nombre"). Es decir, las columnas 2 y 3
   están invertidas respecto a lo que sus encabezados sugieren — no asumas, usá la
   columna 2 como código y la 3 como nombre.
4. ID externo
5. Descripción
6. Negocio (nivel más alto del árbol de clasificación del producto)
7. Categoria (nivel del árbol de clasificación del producto, distinto de la "categoría
   de implantación" de la columna 11 — no confundirlos)
8. Subcategoria (árbol de clasificación)
9. Familia (árbol de clasificación)
10. Código de troquel
11. Categoría de implantación (texto — esta es la que se cruza contra el archivo 2)
12. TK - Clasificación de Surtido (texto: "Ampliado", "Medio", "Troncal",
    "Exclusivo Ecommerce", "X" o "No Aplica" — este es el NIVEL PROPIO del artículo)
13. Artículo suspendido (Sí/No)
14. Inactivo (Sí/No)

**Limitación de alcance conocida:** esta exportación de Excel contiene SOLO artículos
ACTIVOS (columna 14 = "No" en el 100% de las filas). Los artículos inactivos no están
incluidos. Esto es una limitación aceptada del archivo fuente, no algo que debas
corregir ni señalar como error — simplemente dejá constancia en el reporte de que el
alcance es "artículos activos" y seguí adelante.

## ARCHIVO 2 — CONFIGURACIÓN DE NIVEL DE SURTIDO POR TIENDA

Nombre esperado: algo como "ListadeTKConfiguracióndeSurtidoporPDV...xlsx" (si difiere,
identificalo por estas columnas). Una hoja de datos, encabezado en la primera fila,
luego una fila por registro de configuración (aprox. 2.400 filas). Columnas, en este
orden:

1. ID interno (entero)
2. TK - Subsidiaria (texto jerárquico, ver nota de formato abajo — NO se usa en este
   reporte, solo la columna 3 importa acá)
3. TK - Tienda (PDV) (texto jerárquico, ver nota de formato abajo)
4. TK - Categoría de Implantación (texto — se cruza contra la columna 11 del archivo 1)
5. TK - Nivel de Surtido (texto: "Ampliado", "Medio", "Troncal", "Exclusivo Ecommerce",
   "X" o "No Aplica" — este es el NIVEL REQUERIDO por esa tienda para esa categoría)
6. TK - Estado del envío a BRIDGE (texto, ver nota de estado abajo)

**Nota de formato — campo jerárquico (columna 3):** cada celda es una ruta con segmentos
separados por " : ". El código corto de tienda es el ÚLTIMO segmento, recortando
espacios. Ej.: "Central Oeste : STONE S.A. : ST2" → código de tienda "ST2".

Este archivo NO tiene columnas de "Inactivo", "Creado" ni "Última modificación" — seguí
sin ellas, no hace falta pedir otro archivo.

### ⚠ Campo de estado (columna 6) — CRÍTICO, no lo ignores

Puede haber más de una fila de configuración para la misma combinación (categoría de
implantación, tienda). Antes de usar el nivel requerido de una combinación, resolvé
cuál fila vale:
- Si existe alguna fila de esa combinación con estado "Enviado sin Errores", usá el
  nivel de esa fila (si hay más de una con ese estado, usá la de mayor ID interno entre
  ellas).
- Si NINGUNA fila de esa combinación tiene "Enviado sin Errores", usá el nivel de la
  fila de mayor ID interno entre las pendientes (no hace falta marcarla especialmente
  en el resultado final de este reporte en particular, a diferencia de otros reportes
  de este mismo proyecto; simplemente usá ese valor para la evaluación SI/NO).

## ALCANCE — SUCURSALES A EXCLUIR

Antes de construir la configuración, descartá del archivo 2 toda fila cuyo código corto
de tienda (columna 3, ya extraído) sea uno de estos 8 (tolerá variantes con/sin tilde):
ACE (Centro de distribución) [también "ACE (Centro de distribucion)"], COF, FAN, LKV,
PFM, Producto terminado Marca Propia (35), ROSETO, STO.

Deberían quedar 38 tiendas distintas en la configuración resultante. Esas 38 tiendas
son las columnas del reporte final (no proceses ni muestres las 8 excluidas).

## REGLA DE EVALUACIÓN — SI / NO / SIN CONFIG

Para cada artículo (archivo 1) y cada una de las 38 tiendas, calculá un resultado así:

1. Buscá en la configuración ya resuelta (archivo 2, tras excluir las 8 tiendas y
   resolver duplicados por estado) la fila cuya categoría coincida EXACTAMENTE con la
   "Categoría de implantación" del artículo (columna 11 del archivo 1) y cuya tienda sea
   la que estás evaluando. Llamá a ese valor "nivel requerido".
2. Si NO existe esa combinación en la configuración (no hay ninguna fila de esa
   categoría para esa tienda) → el resultado es "SIN CONFIG".
3. Si el "nivel requerido" es "No Aplica" → el resultado es "NO".
4. Si el "nivel requerido" es "Exclusivo Ecommerce" o "X" → el resultado es "SI" SOLO SI
   el nivel propio del artículo (columna 12 del archivo 1) es EXACTAMENTE ese mismo
   valor; en cualquier otro caso, "NO".
5. Si el "nivel requerido" es "Ampliado", "Medio" o "Troncal" → aplicá esta jerarquía
   anidada, con rangos Ampliado=1, Medio=2, Troncal=3:
   - Si el nivel propio del artículo es también "Ampliado", "Medio" o "Troncal",
     comparar los rangos: el resultado es "SI" si rango(nivel propio) >=
     rango(nivel requerido); si no, "NO".
     (Ejemplo: un artículo "Troncal" (rango 3) sale "SI" en tiendas que requieren
     Troncal, Medio o Ampliado. Un artículo "Ampliado" (rango 1) sale "SI" solo en
     tiendas que requieren Ampliado, y "NO" en las que requieren Medio o Troncal.)
   - Si el nivel propio del artículo NO es ninguno de esos tres (es "Exclusivo
     Ecommerce", "X" o "No Aplica") → el resultado es "NO" (no tiene un rango
     comparable contra esta jerarquía).

**Importante:** esta regla de evaluación es una interpretación de negocio que todavía no
fue confirmada formalmente por el dueño del proceso. Dejá esto advertido en rojo en el
reporte (ver entregable abajo), no lo presentes como una regla 100% oficial.

## ENTREGABLE

Un archivo Excel con DOS hojas.

**Hoja 1, "Resumen y metodología" (hoja activa al abrir):**
- Qué muestra el reporte (2-3 frases, en base al contexto de arriba).
- Fuente de datos usada (los dos archivos Excel, mencionar que no se usó ningún
  conector ni API).
- En rojo y destacado: aclaración de que esta corrida solo cubre artículos ACTIVOS (la
  cantidad exacta de filas del archivo 1), y que no incluye los artículos inactivos.
- La regla de evaluación completa, explicada en texto (puntos 2 a 5 de arriba).
- En rojo y destacado: advertencia de que la regla de evaluación no está confirmada
  formalmente.
- Cómo se resolvieron las filas de configuración duplicadas (explicación del criterio
  de estado), con la cantidad de combinaciones que necesitaron el criterio de respaldo
  por pendiente.
- Lista de las 8 sucursales excluidas.
- Resultados globales: cantidad de celdas SI, NO y SIN CONFIG, con sus porcentajes
  sobre el total (total = cantidad de artículos × 38).
- Un top 15 de categorías de implantación con mayor porcentaje de celdas "SIN CONFIG"
  (columnas: categoría, cantidad de artículos de esa categoría, celdas SIN CONFIG,
  celdas totales, % SIN CONFIG).
- Una breve explicación de cómo leer la hoja 2.

**Hoja 2, "Reporte SI-NO":**
- Una fila por artículo. Columnas iniciales: ID Item, Código (columna 2 del archivo 1),
  Nombre (columna 3 del archivo 1), Categoría de implantación, Negocio, Categoría
  (árbol), Subcategoría, Familia, Nivel propio del artículo, Inactivo.
- A partir de la columna siguiente, una columna por cada una de las 38 tiendas (código
  de tienda como encabezado), con el resultado SI/NO/SIN CONFIG para ese artículo en esa
  tienda.
- Formato condicional sobre todo el rango de columnas de tiendas: verde cuando el valor
  es "SI" (por ejemplo fondo C6EFCE, texto 006100), rojo cuando es "NO" (fondo FFC7CE,
  texto 9C0006), naranja cuando es "SIN CONFIG" (fondo FFEB9C, texto 9C6500).
- Autofiltro en la fila de encabezados, y esa fila congelada junto con las columnas
  identificatorias (freeze panes justo antes de la primera columna de tienda).

## CIFRAS DE REFERENCIA PARA AUTOVALIDAR

(de una corrida anterior sobre prácticamente los mismos archivos; esperá resultados del
mismo orden, no necesariamente idénticos):

- Deberían quedar exactamente 38 tiendas tras la exclusión, y entre 2.200 y 2.300
  combinaciones (categoría, tienda) resueltas en la configuración.
- Sobre el total de celdas (artículos activos × 38 tiendas): aproximadamente 55-60% SI,
  30-35% NO, y 8-10% SIN CONFIG. Si tus porcentajes están muy lejos de este rango (por
  ejemplo, SIN CONFIG por encima del 30% o por debajo del 2%), revisá primero si estás
  cruzando bien la categoría de implantación del artículo contra la de la
  configuración (son texto, tienen que coincidir carácter por carácter).
- Es esperable que algunas categorías de implantación (por ejemplo una llamada
  "insumos", y otra "Exclusivo Ecommerce") salgan en 100% SIN CONFIG para todas las
  tiendas — eso es un hallazgo real (esa categoría no tiene ninguna fila en el archivo
  2), no un bug tuyo.
- No debería haber artículos con la "Categoría de implantación" o el "Nivel propio"
  vacíos en el archivo 1; si encontrás filas así, revisá que estés leyendo bien las
  columnas (recordá la inversión de nombres en las columnas 2 y 3).

Guardá el Excel resultante y entregáselo al usuario.
