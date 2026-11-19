# Estructura de la página semanal

Este es el esqueleto en Notion-flavored Markdown que replica la Semana 46, la versión aprobada por el usuario. Reemplaza el texto entre `«»` y quita las secciones que no apliquen esa semana. Si una planta no tuvo temas, quita su sección en vez de dejarla vacía.

Las vistas de Acciones (Seguimiento y Acciones de esta reunión) **no** se escriben aquí: se crean después con `create-view` y se mueven a su sitio (ver el paso 5 de SKILL.md). Deja los encabezados y las líneas grises como ancla.

````markdown
<callout icon="📌" color="gray_bg">
**Reporte semanal — Seguridad en el Trabajo** · Semana «N» · «DD» al «DD» de «mes» de «AAAA» · «Planta Quevedo y Planta Santo Domingo»
</callout>

## 📊 Resumen de la semana

<columns>
<column ratio="25">
<callout icon="«emoji»" color="«color»_bg">
**«Tema 1»**

### «Dato clave: número o estado»

«Línea de detalle»

«Línea de detalle»

<span color="gray">«Fuente o nota»</span>
</callout>
</column>
<column ratio="25">
«… misma forma para cada tema, hasta 4 tarjetas; con 3 temas usa ratio 33 …»
</column>
</columns>

---

## 🔁 Seguimiento — acciones abiertas de reuniones anteriores

<span color="gray">Se llena sola: muestra todas las acciones que aún no están en Hecho.</span>

---

## 🏭 Planta Quevedo

### «emoji» «Título del tema»

«1–3 frases de contexto. Lo importante en **negrita**.»

**Qué se hizo:**
- «hecho concreto»
- «hecho concreto»

«Opcional: tabla con | Fecha | Actividad | Detalle | o dos columnas de callouts para listas paralelas, como las brigadas.»

<callout icon="➡️" color="blue_bg">
**Siguiente paso:** «lo que sigue; también se crea como registro en Acciones»
</callout>

---

## 🏭 Planta Santo Domingo — «tema principal»

<columns>
<column ratio="50">
<callout icon="«emoji»" color="«color»_bg">
**«Indicador»**

### «número»

«desglose»

🔴 «lo urgente»
</callout>
</column>
<column ratio="50">
«segunda tarjeta»
</column>
</columns>

<callout icon="⚠️" color="red_bg">
**«Inconsistencia o riesgo a revisar»:** «detalle»
</callout>

<toggle heading="h3">
Listado de «equipos» («total»)

| «Col 1» | «Col 2» | «Col 3» | «Vencimiento» |
| --- | --- | --- | --- |
| … | … | … | 🔴 «MM/AAAA» por vencer |

<span color="gray">Fuente: «registro de origen» («n» registros).</span>
</toggle>

---

## 🚑 Incidentes de la semana

<callout icon="✅" color="green_bg">
Sin incidentes ni accidentes reportados.
</callout>

«Si hubo: un callout pink_bg por evento con tipo, fecha, área, descripción breve y acción inmediata, y su acción correctiva en la base de Acciones.»

---

## ✅ Acciones de esta reunión

<span color="gray">Se gestionan en la base Acciones — Reuniones E&S. Las abiertas aparecen también en la reunión siguiente.</span>
````

## Emojis y colores usados

| Tema | Emoji | Color |
|---|---|---|
| Brigadas / emergencias | 🚒 🔥 🚪 | red_bg / green_bg |
| EPP | 🧤 🦺 | blue_bg |
| SCI (piscina, extintores, gabinetes) | 💧 🧯 🗄️ | orange_bg / yellow_bg |
| Capacitación | 🎓 | purple_bg |
| Proveedores / visitas | 🤝 | brown_bg |
| Incidentes | 🚑 | pink_bg |
| Riesgo o alerta | ⚠️ | red_bg |
