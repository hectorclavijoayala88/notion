---
name: reunion-semanal-notion
description: Arma la reunión semanal de Seguridad (E&S) de Plantabal como una página de Notion dentro de la base "Reuniones Semanales E&S", con tarjetas de resumen, temas por planta (Quevedo / Santo Domingo), incidentes y acciones de seguimiento enlazadas. Úsala siempre que el usuario hable de la reunión semanal, el reporte semanal de SST o E&S, "armemos la semana 47", pasar el PPT semanal a Notion, cargar los temas de la semana, revisar acciones abiertas de reuniones anteriores o preparar lo que se presenta a gerencia cada semana, aunque no diga "Notion". No la uses para generar el PPT del reporte (para eso está reporte-semanal-v2), salvo que pidan exportar la página de Notion.
---

# Reunión semanal E&S en Notion

Reemplaza el reporte semanal en PowerPoint por una página de Notion por semana. La ventaja clave frente al PPT es el **seguimiento**: cada "siguiente paso" es un registro en la base de Acciones que sigue abierto, y aparece solo en la reunión siguiente, hasta que se cierra. Cuida esa parte más que la estética.

## Dónde está todo

| Qué | Enlace / ID |
|---|---|
| Base **Reuniones Semanales E&S** | https://app.notion.com/p/41dd5a472f6c81d9b78afbfa973191ab · data source `collection://41dd5a47-2f6c-81c8-a46b-000b8b89e8ae` |
| Base **Acciones — Reuniones E&S** | https://app.notion.com/p/41dd5a472f6c81c0924ec4b7d722435a · data source `collection://41dd5a47-2f6c-81f8-b21f-000b3a170032` |
| Página plantilla | `41dd5a47-2f6c-8156-903c-fe4694ae93d2` (Estado = Plantilla) |
| Ejemplo real terminado (Semana 46) | `41dd5a47-2f6c-8190-9d91-ce853e3730a9` |
| Centro de Control (hub) | `41dd5a47-2f6c-8170-8ff6-e671439c1201` |

Si un ID no responde, búscalo por nombre con la búsqueda de Notion antes de crear nada nuevo: duplicar las bases rompería el historial.

**Propiedades de Reuniones:** Reunión (título), Semana (número), Período (fecha inicio–fin), Planta (multi: Quevedo, Santo Domingo), Temas (multi: Brigadas, EPP, SCI, Capacitación, Inspecciones, Incidentes, Proveedores, Documentación), Estado (Borrador, Presentado, Plantilla), Resumen (texto), Acciones (relación).

**Propiedades de Acciones:** Acción (título), Reunión (relación), Tema (select, mismas opciones), Planta (Quevedo, Santo Domingo, Ambas), Estado (status: Pendiente, En curso, En espera de terceros, Hecho), Responsable (texto), Fecha compromiso (fecha), Notas, ⏰ Plazo (fórmula automática; no la escribas).

## Flujo de cada semana

### 1. Recoger la información

El usuario está ocupado: pide todo en **un solo mensaje** y acepta lo que te dé en cualquier formato (notas sueltas, fotos, el PPT de siempre, un audio transcrito).

> "Para la semana N necesito: el período (ej. 17 al 23 nov), los temas de cada planta con sus datos clave y siguiente paso, si hubo incidentes, y qué cambió en las acciones abiertas que te muestro abajo."

Antes de preguntar, consulta las acciones abiertas (`Estado != 'Hecho'`) y muéstralas en una lista corta para que diga qué se cerró o avanzó. Así la reunión arranca con el seguimiento, que es para lo que existe.

Si el usuario sube un **.pptx**, extrae el texto leyendo el XML con `zipfile` (en `ppt/slides/slideN.xml`, etiquetas `a:t`). `python-pptx` puede fallar por dependencias en algunos entornos, y el XML siempre funciona. Las fotos están en `ppt/media/`.

No inventes datos ni fechas. Si falta algo importante (por ejemplo, una fecha compromiso), déjalo vacío y menciónalo al final.

### 2. Actualizar las acciones existentes

Cambia el Estado y agrega notas a las acciones que el usuario reporte. Hazlo antes de crear la página nueva, para que la vista de seguimiento ya salga al día.

### 3. Crear la página de la semana

Crea la página directamente en el data source de Reuniones, con la estructura de `references/estructura-pagina.md`. Esa estructura replica la Semana 46, que el usuario ya aprobó; léela antes de escribir. Propiedades:

- **Reunión**: `Semana N · DD al DD mes AAAA`, sin emoji (el ícono `📅` va aparte, si no se duplica).
- **Semana**, **Período** (`date:Período:start` / `end`), **Planta**, **Temas**.
- **Estado**: `Borrador`. Pasa a `Presentado` cuando el usuario diga que ya se presentó.
- **Resumen**: 1–2 frases con lo esencial (se ve en la tabla de la base).
- Después de crearla, pon `full_width: true` para que las tarjetas en columnas se vean bien.

### 4. Crear las acciones nuevas

Cada "siguiente paso" o pendiente que salga de los temas es un registro en Acciones, con la relación `Reunión` apuntando a la página nueva. Una acción debe ser algo que alguien pueda marcar como hecho: empieza con un verbo ("Confirmar…", "Gestionar…"), no con un tema. Si depende de un tercero (bomberos, un proveedor, un contratista), usa el estado `En espera de terceros`.

Si encuentras inconsistencias en los datos del usuario (números duplicados, equipos asignados a la planta equivocada, totales que no cuadran), créalas como acción pendiente y destácalas en un callout rojo. En la Semana 46 así salieron PSD32 duplicado y PSD26 en la planta equivocada, y fue útil.

### 5. Insertar las vistas de acciones en la página

Con `create-view` (parent_page_id = página de la semana, `show_data_source_title: false`):

1. **Seguimiento**: tabla del data source de Acciones con `FILTER "Estado" != "Hecho"; SORT BY "Fecha compromiso" ASC`.
2. **Acciones de esta reunión**: `FILTER "Reunión" = "<id de la página>"`.

`create-view` siempre agrega la vista **al final** de la página. Para moverla a su sección, haz `fetch`, toma la etiqueta `<database url=...>` y usa `update_content` para quitarla del final e insertarla bajo el encabezado correcto. Después verifica con `fetch`: los reemplazos grandes a veces desordenan los bloques; si pasa, repetir el mismo `replace_content` una vez lo corrige.

### 6. Cerrar

- En el hub, actualiza la fila "📅 Reuniones Semanales E&S" de la tabla de acceso rápido ("última: Semana N") y el número de acciones abiertas.
- Responde con el enlace a la página, las acciones creadas o cerradas, y lo que quedó pendiente de datos.
- Si hay acciones con fecha compromiso, ofrece crear recordatorios en Google Calendar (sin invitados).

## Formato visual (lo que el usuario ya aprobó)

- **Columnas**: `<columns><column ratio="25">…</column></columns>`. La sintaxis `:::columns` **no** funciona en Notion y se guarda como texto plano.
- **Tarjetas**: un `<callout>` con color de fondo por columna: título en negrita, dato clave como `###` y 2–3 líneas cortas. Colores por tema: Brigadas `red_bg`, EPP `blue_bg`, SCI `orange_bg` / `yellow_bg`, Capacitación `purple_bg`, Incidentes `pink_bg`, logros `green_bg`.
- **Barras de progreso**: cuadros de emoji (🟩🟩🟩⬜⬜⬜⬜⬜⬜⬜ = 30 %). Los caracteres ▓░ se ven mal.
- **Gráficos**: no uses vistas tipo chart. El plan gratuito permite solo 1 en todo el espacio y aparecen bloqueadas. Usa tarjetas y tablas filtradas.
- **Listados largos** (inventarios de extintores, asistencia, etc.): tabla dentro de `<toggle heading="h3">`, para que la página no se haga eterna. Marca con 🔴 lo vencido o por vencer y con ⚠️ lo que hay que confirmar.
- **Fotos**: súbelas con `create-file-upload`. En sesiones de Claude Code en la nube, la subida a `api.notion.com` puede estar bloqueada por la red. En ese caso, deja un callout `📷` que diga "arrastra aquí las fotos" y avísale al usuario.

## Si gerencia pide un archivo

La página se exporta desde Notion (⋯ → Exportar → PDF). Si piden PPT, usa la skill `reporte-semanal-v2` con los datos de la página como fuente.
