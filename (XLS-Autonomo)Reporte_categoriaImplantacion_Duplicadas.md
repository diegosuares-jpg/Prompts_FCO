# PROMPT 1 — Reporte de calidad de datos: "Categorías de Implantación Repetidas por Tienda"
### Versión autónoma (sin NetSuite / sin MCP — solo Excel adjunto)

## CONTEXTO DE NEGOCIO

Central Oeste es una cadena de farmacias (Argentina) que usa NetSuite como ERP. Cada
artículo (ítem) tiene una "categoría de implantación" (una clasificación de producto).
Para cada combinación (categoría de implantación, tienda/sucursal) existe una tabla de
configuración que define qué "nivel de surtido" se exige en esa tienda para esa
categoría (los niveles posibles son: Ampliado, Medio, Troncal, Exclusivo Ecommerce, X,
No Aplica). Esa tabla de configuración es la fuente de este reporte.

Este reporte es un CHEQUEO DE CALIDAD DE DATOS sobre esa tabla de configuración: detecta
combinaciones (categoría, tienda) que tienen 2 o más filas cargadas cuando deberían
tener como máximo 1. No es el reporte de artículos, es un control sobre la tabla fuente.

**NO TENÉS ACCESO A NETSUITE NI A NINGÚN CONECTOR.** Vas a trabajar ÚNICAMENTE con el
archivo Excel que se adjunta a este mensaje. No inventes, no completes ni asumas datos
que no estén en ese archivo.

## ARCHIVO DE ENTRADA

Un archivo Excel (nombre esperado: algo como "ListadeTKConfiguracióndeSurtidoporPDV...xlsx";
si el nombre difiere, identificalo por tener estas columnas). Tiene una sola hoja de
datos con una fila de encabezado y luego una fila por registro de configuración
(aprox. 2.400 filas en la última exportación; tratá ese número como referencia, no como
requisito exacto). Columnas, en este orden:

1. ID interno (número entero, identificador único de la fila de configuración)
2. TK - Subsidiaria (texto jerárquico, ver nota de formato abajo)
3. TK - Tienda (PDV) (texto jerárquico, ver nota de formato abajo)
4. TK - Categoría de Implantación (texto, nombre de la categoría)
5. TK - Nivel de Surtido (texto: uno de "Ampliado", "Medio", "Troncal",
   "Exclusivo Ecommerce", "X", "No Aplica")
6. TK - Estado del envío a BRIDGE (texto: típicamente "Enviado sin Errores",
   "Creado y Pendiente de Envío" o "Actualizado y Pendiente de Envío" — puede haber
   variantes menores de redacción/acentuación, tratalas como equivalentes por contenido)

**Nota de formato — campos jerárquicos (columnas 2 y 3):** cada celda contiene una ruta con
segmentos separados por " : " (dos puntos entre espacios), por ejemplo algo como
"Central Oeste : STONE S.A. : ST2". El CÓDIGO CORTO de tienda o subsidiaria es el
ÚLTIMO segmento de esa ruta, recortando espacios sobrantes. En el ejemplo, el código de
tienda sería "ST2".

Este archivo NO tiene columnas de "Inactivo", "Creado" ni "Última modificación" — no
están disponibles en esta exportación. Seguí adelante sin ellas (no hace falta pedir
otro archivo ni completarlas).

## ALCANCE

- Sucursales a EXCLUIR del análisis (compará por el código corto ya extraído; tolerá
  variantes con/sin tilde):
  ACE (Centro de distribución) [también puede aparecer como "ACE (Centro de distribucion)"],
  COF, FAN, LKV, PFM,
  Producto terminado Marca Propia (35),
  ROSETO, STO.
  Con esta exclusión deberían quedar 38 tiendas distintas (de un total de 46 en el
  archivo completo).
- Incluir TODAS las filas restantes, sin filtrar por estado ni por ninguna otra
  condición.

## LÓGICA A APLICAR

1. Para cada fila, extraé el código corto de tienda (columna 3) como se explicó arriba,
   y descartá la fila si ese código está en la lista de exclusión.
2. Agrupá las filas restantes por la combinación (TK - Categoría de Implantación,
   código de tienda).
3. Quedate solo con los grupos que tengan 2 O MÁS filas. Esos son los "duplicados".
4. Para cada grupo duplicado, clasificalo:
   - "CONFLICTO (niveles distintos)" si dentro del grupo hay más de un valor distinto de
     "TK - Nivel de Surtido".
   - "REDUNDANTE (mismo nivel repetido)" si todas las filas del grupo tienen el mismo
     nivel.
5. Para cada grupo, determiná cuál fila es "la vigente":
   - Preferí la fila cuyo "TK - Estado del envío a BRIDGE" sea "Enviado sin Errores".
   - Si ninguna fila del grupo tiene ese estado, usá la de mayor "ID interno" entre las
     pendientes, pero marcala como una aproximación NO confirmada (no hay forma de saber
     cuál es la realmente vigente en ese caso).

## ENTREGABLE

Un archivo Excel con DOS hojas.

**Hoja 1, "Resumen" (debe ser la hoja activa al abrir el archivo):**
- Qué es este reporte y qué chequea (2-3 frases, en base al contexto de arriba).
- Alcance: 38 tiendas (listar cuáles 8 se excluyeron), se incluyen todas las filas sin
  filtrar por estado.
- Estadísticas globales: cantidad total de filas de configuración leídas, cantidad de
  filas tras excluir las 8 tiendas, cantidad de combinaciones (categoría, tienda)
  distintas, cantidad de esas combinaciones que resultaron duplicadas, cuántas son
  CONFLICTO vs cuántas REDUNDANTE, cantidad total de filas involucradas en duplicados.
- Una tabla de "repeticiones por tienda": cantidad de combinaciones duplicadas por cada
  tienda, ordenada de mayor a menor.
- Una explicación en texto de cómo se define "la vigente" (la regla del punto 5 de
  arriba).
- Una nota aclarando que esta tabla de origen no tiene un campo de "mensaje de error de
  integración", solo el estado (campo 6).

**Hoja 2, "Categorías repetidas por tienda":**
- Una fila por cada FILA de configuración que forme parte de un grupo duplicado (si un
  grupo tiene 3 filas, van las 3 filas, no una fila resumen).
- Ordenado por tienda y luego por categoría.
- Columnas: Tienda, Categoría de implantación, ID fila config, Nivel de surtido, Estado
  envío a BRIDGE, Tipo de duplicado (CONFLICTO/REDUNDANTE), ¿Es la vigente? (Sí/No).
- Formato condicional: fondo rojo claro (código de color FFC7CE) para las filas de tipo
  CONFLICTO; fondo amarillo (FFEB9C) para las REDUNDANTE; texto verde y negrita
  (por ejemplo color 006100) en la fila marcada como vigente.
- Autofiltro en la fila de encabezados y esa misma fila congelada (freeze panes).

## CIFRAS DE REFERENCIA PARA AUTOVALIDAR TU RESULTADO

(de una corrida anterior sobre prácticamente el mismo archivo; los tuyos pueden variar
levemente si el export cambió, pero deberían ser del mismo orden de magnitud):

- Filas totales en el archivo: ~2.403. Tras excluir las 8 tiendas: ~2.341 filas, ~2.240
  combinaciones (categoría, tienda) distintas.
- ~54 combinaciones duplicadas (39 CONFLICTO + 15 REDUNDANTE), ~155 filas involucradas
  en total.
- Las tiendas con más duplicados concentran la mayoría de los casos (una tienda con
  código "FLB" sola explica buena parte, seguida de una con código "ASM"); el resto son
  casos aislados.
- Prácticamente todas las combinaciones duplicadas tienen alguna fila "Enviado sin
  Errores" (muy pocas, del orden de 5, no tienen ninguna y requieren el criterio de
  respaldo).

Si tu resultado difiere en 1-2 órdenes de magnitud de estas cifras, revisá la lógica de
agrupación y de exclusión de tiendas antes de entregar.

Guardá el Excel resultante y entregáselo al usuario.
