# PROMPT 2 — "Matriz Categoría de Implantación x Subsidiaria - Nivel de Surtido"
### Versión autónoma (sin NetSuite / sin MCP — solo Excel adjunto)

## CONTEXTO DE NEGOCIO

Central Oeste es una cadena de farmacias (Argentina) que usa NetSuite como ERP. Cada
artículo tiene una "categoría de implantación" (una clasificación de producto). Para
cada combinación (categoría de implantación, tienda/sucursal) existe una configuración
que define qué "nivel de surtido" se exige en esa tienda para esa categoría (los
niveles posibles son: Ampliado, Medio, Troncal, Exclusivo Ecommerce, X, No Aplica).

Este reporte arma una matriz visual: filas = categorías de implantación, columnas =
sucursales (agrupadas visualmente bajo la subsidiaria a la que pertenecen), e
intersección = el nivel de surtido configurado para esa combinación.

**NO TENÉS ACCESO A NETSUITE NI A NINGÚN CONECTOR.** Vas a trabajar ÚNICAMENTE con el
archivo Excel que se adjunta a este mensaje. No inventes, no completes ni asumas datos
que no estén en ese archivo.

## ARCHIVO DE ENTRADA

El mismo tipo de archivo que en el reporte de duplicados: un Excel (nombre esperado algo
como "ListadeTKConfiguracióndeSurtidoporPDV...xlsx"; si difiere, identificalo por estas
columnas) con una hoja de datos, encabezado en la primera fila, y luego una fila por
registro de configuración (aprox. 2.400 filas). Columnas, en este orden:

1. ID interno (entero)
2. TK - Subsidiaria (texto jerárquico, ver nota de formato)
3. TK - Tienda (PDV) (texto jerárquico, ver nota de formato)
4. TK - Categoría de Implantación (texto)
5. TK - Nivel de Surtido (texto: "Ampliado", "Medio", "Troncal", "Exclusivo Ecommerce",
   "X" o "No Aplica")
6. TK - Estado del envío a BRIDGE (texto, ver nota de estado abajo)

**Nota de formato — campos jerárquicos (columnas 2 y 3):** cada celda es una ruta con
segmentos separados por " : ". El código corto de tienda/subsidiaria es el ÚLTIMO
segmento, recortando espacios. Ej.: "Central Oeste : STONE S.A. : ST2" → código de
tienda "ST2", y esa fila pertenece a la subsidiaria cuyo nombre corto es "STONE S.A."
(extraído igual, último segmento de la columna 2).

## ⚠ CAMPO DE ESTADO — CRÍTICO, no lo ignores

Puede haber más de una fila de configuración para la misma combinación (categoría,
tienda). La columna 6 distingue:
- "Enviado sin Errores" → configuración confirmada y vigente.
- "Creado y Pendiente de Envío" / "Actualizado y Pendiente de Envío" → un cambio fue
  cargado pero todavía NO está aplicado/confirmado.

Regla de resolución para cada combinación (categoría, tienda):
- Si existe alguna fila con estado "Enviado sin Errores", usá el nivel de esa fila (si
  hubiera más de una con ese estado — no debería pasar, pero por las dudas — usá la de
  mayor ID interno entre ellas).
- Si NINGUNA fila de esa combinación tiene "Enviado sin Errores", usá el nivel de la
  fila de mayor ID interno entre las pendientes, pero MARCALA como no confirmada
  (en el Excel: texto en itálica + un asterisco al final, ej. "Troncal *").

## AGRUPACIÓN POR SUBSIDIARIA — IMPORTANTE

La subsidiaria NO es uno a uno con la tienda: varias tiendas pueden pertenecer a la
misma subsidiaria (por ejemplo, una subsidiaria con nombre corto "STONE S.A." agrupa
varias tiendas con códigos como ST1, ST2, ST3, ST4, STO; otra con nombre corto
"ELKAV S.R.L." agrupa varias tiendas más). Dentro de una misma subsidiaria, tiendas
distintas suelen tener niveles DISTINTOS configurados para la misma categoría. Por eso
NO colapses a una columna por subsidiaria: mantené UNA COLUMNA POR TIENDA (deberían ser
46 en total, sin excluir ninguna en este reporte — a diferencia de otros reportes de
este mismo proyecto, acá no se excluye ningún local), ordenadas por subsidiaria y
agrupadas visualmente bajo ella.

## ENTREGABLE

Un archivo Excel con DOS hojas.

**Hoja 1, "Notas y leyenda" (hoja activa al abrir):**
- Qué es la matriz y cómo leerla (2-3 frases).
- El hallazgo del campo de estado: cuántas filas totales tienen "Enviado sin Errores"
  vs. pendientes, y qué porcentaje de combinaciones (categoría, tienda) con datos solo
  tienen una fila pendiente (sin ninguna "Enviado").
- Por qué las columnas son tiendas agrupadas por subsidiaria y no subsidiarias puras
  (explicación del punto anterior).
- Leyenda de colores (ver abajo).
- Mención de las categorías que no tienen NINGUNA fila de configuración en todo el
  archivo (si las hay), listándolas.

**Hoja 2, "Categoría x Subsidiaria":**
- Fila 1: nombre de la subsidiaria, en celdas combinadas (merged) que abarcan todas las
  columnas de tienda que le pertenecen.
- Fila 2: código corto de cada tienda (una columna por tienda, 46 columnas en total),
  ordenadas por subsidiaria y luego por código de tienda.
- Columna A: nombre de la categoría de implantación (una fila por categoría distinta
  encontrada en el archivo).
- Desde la fila 3 en adelante, cada celda (categoría, tienda) muestra el nivel de
  surtido resuelto para esa combinación según la regla de arriba. Si no hay ninguna
  fila de configuración para esa combinación, dejar la celda en blanco.
- Colores de fondo/texto por nivel (fondo/texto en hex):
  - Troncal = fondo 1E7F3C, texto blanco (FFFFFF)
  - Medio = fondo 70AD47, texto blanco (FFFFFF)
  - Ampliado = fondo C6E0B4, texto verde oscuro (1E5E27)
  - Exclusivo Ecommerce = fondo 2E75B6, texto blanco (FFFFFF)
  - X = fondo 7030A0, texto blanco (FFFFFF)
  - No Aplica = fondo D9D9D9, texto gris (595959)
- Las celdas con valor pendiente no confirmado (ver regla de arriba) van en itálica con
  un asterisco al final del texto, independientemente del color de nivel que les
  corresponda.

## CIFRAS DE REFERENCIA PARA AUTOVALIDAR

(de una corrida anterior; orden de magnitud esperado, no exacto):

- Debería haber 46 tiendas y entre 65 y 75 categorías de implantación distintas.
- Alrededor de 2.300 combinaciones (categoría, tienda) con datos, de las cuales
  aproximadamente el 96% tienen una fila "Enviado sin Errores" y el resto (~4%) dependen
  del criterio de respaldo por pendiente.
- Debería haber 4 subsidiarias que agrupan más de una tienda (con 2, 3, 5 y 7 tiendas
  respectivamente) y el resto de las subsidiarias con una sola tienda cada una.
- Es esperable que 3 o 4 categorías no tengan ninguna fila de configuración (quedan con
  toda la fila en blanco) — esto es un hallazgo real, no un error tuyo.

Guardá el Excel resultante y entregáselo al usuario.
