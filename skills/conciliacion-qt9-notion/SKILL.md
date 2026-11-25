---
name: conciliacion-qt9-notion
description: Concilia el export CSV del listado maestro de documentos de QT9 con la base "Documentos SST — Maestro QT9" del Repositorio Documental SST de Plantabal en Notion. Actualiza revisiones, fechas de aprobación y estado, recalcula la Próxima Revisión según el criterio CE-IF-D §5.5 Rev.15, crea los documentos nuevos y marca los que ya no aparecen en QT9. Activar SIEMPRE que Héctor suba un CSV o Excel exportado de QT9, o diga "actualiza el repositorio", "concilia con QT9", "cruce con QT9", "actualiza las fechas de los documentos", "sincroniza QT9 con Notion" o cualquier variante en la que quiera llevar a Notion el estado documental de QT9. No esperar a que diga "skill": si hay un export de QT9 y el contexto es el repositorio documental, esta skill aplica.
---

# Conciliación QT9 → Notion (Repositorio Documental SST)

QT9 QMS es la **fuente oficial**. Notion es un espejo de consulta y control. Esta skill lleva el estado del CSV de QT9 a Notion sin perder información y deja trazabilidad de cada cruce.

## Destinos en Notion

| Elemento | Referencia |
|---|---|
| Base de datos | **Documentos SST — Maestro QT9** · data source `collection://011b9fa1-2a54-44e1-8675-f3e845c0f02d` |
| Página contenedora | Repositorio Documental SST — Plantabal · `https://app.notion.com/p/3f1d5a472f6c81ec8fd4ea2dd09a4680` |
| Página que se actualiza sola con los datos | Centro de Control — E&S Plantabal · `https://app.notion.com/p/41dd5a472f6c81708ff6e671439c1201` |

Antes de escribir, haz `fetch` del data source para confirmar los nombres exactos de las columnas: pueden haber cambiado desde que se escribió esta skill.

## Columnas de la base y origen del dato

| Columna Notion | Tipo | Origen |
|---|---|---|
| Título del Documento | título | CSV · Title |
| Código | texto | CSV · Doc. Reference (ej. `GE-SO-D14`). **Clave de emparejamiento.** |
| Código QT9 | texto | CSV · Document Code (ej. `PB-GE-PRO-193`) |
| Tipo QT9 | texto | CSV · Doc. Type, tal cual |
| Tipo | selección | Derivado de Tipo QT9 (ver mapeo) |
| Revisión | texto | CSV · Revision |
| Fecha Aprobación | fecha | CSV · Date Approved |
| Estado en QT9 | selección | Derivado de CSV · Status (ver mapeo) |
| Próxima Revisión | fecha | **Calculada** (ver criterio) |
| Vigencia | fórmula | Automática: no se escribe |
| Categoría | selección | Derivada del prefijo del código (ver mapeo) |
| Conciliación QT9 | selección | Resultado del cruce |
| Última Verificación QT9 | fecha | Fecha del cruce (hoy) |
| Lote | selección | **No tocar**: lo gestiona Héctor (Plan de Actualización) |
| Sitio Aplicable, Observaciones | — | **No tocar** salvo que Héctor lo pida |

## Criterio de Próxima Revisión: CE-IF-D §5.5 Rev.15 (vigente desde el 20/08/2026)

`Próxima Revisión = Fecha Aprobación + periodo`

| Tipo | Periodo |
|---|---|
| Manual, Procedimiento, Instructivo | 4 meses |
| Matriz (identificación de peligros / evaluación de riesgos) | 6 meses |
| Formato, Registro, Documento, Caracterización, Política, Indicador | 2 años |
| Plan, Programa | No están en Rev.15: **preguntar a Héctor** antes de calcular. Si no responde, usa 2 años y márcalo en el informe |

Si el procedimiento CE-IF-D cambia de revisión, pide el nuevo criterio antes de calcular.

## Mapeos

**Categoría, por el prefijo del código:**
- `GE-SO-…` (incluye `GE-SO-RL-…`, matrices) → Seguridad Industrial
- `GE-GA-…` → Gestión Ambiental
- `GE-SF-…` → Salud Ocupacional
- `GE-D…`, `GD-…`, `CE-MC-…` → Sistema de Gestión
- Prefijo desconocido → consulta en la base cómo están clasificados los códigos parecidos y, si sigue sin quedar claro, pregunta.

**Tipo, a partir de Tipo QT9:** antes de mapear, consulta en la base los pares existentes `Tipo QT9 → Tipo` (`SELECT DISTINCT "Tipo QT9", "Tipo"`) y reutiliza ese mapeo. Opciones válidas de Tipo: Procedimiento, Instructivo, Formato, Registro, Política, Plan, Matriz, Programa, Caracterización, Documento, Manual, Indicador. Para un Tipo QT9 nuevo, propón el mapeo y confírmalo con Héctor.

**Estado en QT9, a partir de Status:** los valores exactos dependen del export, así que muestra a Héctor los valores distintos del CSV y confirma. Mapeo orientativo:
- Released / Approved / Effective → Vigente
- In Review / Under Review → En Revisión
- Draft / In Progress → En Elaboración
- Pending Approval → Pendiente de Aprobación
- Obsolete / Retired / Archived → Obsoleto

## Flujo de trabajo

### 1. Leer y validar el CSV
- Carga el archivo y muestra las columnas detectadas y el número de filas.
- Confirma qué columna corresponde a cada campo de la tabla anterior. Si falta **Doc. Reference** o **Date Approved**, detente y pide un export que las incluya.
- Normaliza los códigos: quita espacios y pásalos a mayúsculas. Revisa si hay códigos duplicados en el CSV. Si hay varias revisiones del mismo código, usa la revisión más alta o la fecha de aprobación más reciente, y avísalo.

### 2. Leer la base de Notion
- Haz una sola consulta SQL con todo lo necesario: `url, "Código", "Título del Documento", "Revisión", "date:Fecha Aprobación:start", "Estado en QT9", "Tipo", "Tipo QT9", "Categoría", "Lote", "Conciliación QT9"`.
- **Plan gratuito:** las consultas a la base tienen un tope por hora. Agrupa todo en la menor cantidad posible de consultas y no repitas consultas innecesarias. Las escrituras (actualizar o crear páginas) no consumen ese tope.

### 3. Clasificar cada documento
| Caso | Condición | Conciliación QT9 |
|---|---|---|
| **Actualizado** | Está en ambos y cambió la revisión, la fecha o el estado | Coincide |
| **Sin cambios** | Está en ambos y todo es igual | Coincide |
| **Nuevo** | Está en el CSV y no en Notion | Coincide (se crea) |
| **Solo en Notion** | Está en Notion y no en el CSV | Solo en Notion |

### 4. Mostrar la vista previa y pedir confirmación **antes de escribir**
Presenta un resumen con:
- Conteo por caso.
- Tabla de **actualizados**: código, título, revisión anterior → nueva, fecha anterior → nueva, nueva Próxima Revisión.
- Lista de **nuevos**, con su Tipo y Categoría propuestos.
- Lista de **solo en Notion**.
- Documentos del Plan de Actualización (columna Lote) que se revisaron en este cruce.

No escribas nada en Notion hasta que Héctor confirme.

### 5. Escribir en Notion
- **Actualizados:** Revisión, Fecha Aprobación, Estado en QT9, Tipo QT9, Código QT9, Próxima Revisión, Conciliación QT9 = Coincide, Última Verificación QT9 = hoy. Actualiza el título solo si cambió en QT9.
- **Sin cambios:** solo Conciliación QT9 = Coincide y Última Verificación QT9 = hoy.
- **Nuevos:** crea la página en el data source con todas las columnas derivadas. Lote vacío.
- **Solo en Notion:** Conciliación QT9 = Solo en Notion y Última Verificación QT9 = hoy. **Nunca los borres.**
- Fechas en formato ISO (`AAAA-MM-DD`), sin hora.
- Si una escritura falla, reintenta una vez. Si vuelve a fallar, anótala en el informe final y no la des por hecha.

### 6. Actualizar el texto de la página del Repositorio
En el Repositorio Documental, actualiza las frases que citan la fecha del último cruce ("conciliado contra el export de QT9 del …", "actualizadas el …") con la fecha de hoy. También actualiza la descripción del data source.

### 7. Informe final para Héctor
- Conteo de actualizados, sin cambios, nuevos y solo en Notion.
- **Vigencia tras el cruce:** cuántos documentos siguen con la revisión vencida, por categoría (consulta `"date:Próxima Revisión:start" < hoy`, excluyendo los No Elaborado).
- **Plan de Actualización por lote:** cuántos documentos de cada lote quedaron al día y cuántos siguen vencidos, y si alguna meta de lote ya pasó.
- Pendientes que requieren decisión: tipos nuevos, documentos solo en Notion, Plan/Programa sin criterio de revisión.
- Enlace al Centro de Control y al Repositorio.

## Reglas
- QT9 manda: ante una diferencia, el dato de QT9 reemplaza al de Notion.
- Nunca borres páginas ni columnas. Nunca cambies **Lote**, **Observaciones** ni **Sitio Aplicable** sin pedido explícito.
- No escribas cifras fijas en ninguna página: el Centro de Control y las vistas se calculan solos desde la base.
- Responde siempre en español, con tablas y listas claras.
