# Stock PDI — PA Perú

Reporte de stock y movimientos del PDI, centro L003. Sitio estático: no necesita servidor,
base de datos ni cuenta de ningún servicio.

## Qué hay aquí

    index.html        la aplicación completa (~75 KB)
    data/base.json    corte de stock, descripciones de material y reglas de área
    data/mov.json     historial de movimientos, con su área ya calculada
    data/areas.json   órdenes conocidas e histórico de áreas resueltas
    .nojekyll         evita que GitHub Pages procese los archivos

La página carga `data/base.json` al abrir. `data/mov.json` se trae recién cuando alguien entra
a la ventana de movimientos o descarga el Excel, y `data/areas.json` solo al usar la ingesta.
Por eso el reporte abre rápido por más que crezca el historial.

## Publicar por primera vez

1. Crear un repositorio nuevo en GitHub.
2. Subir estos archivos respetando la carpeta `data/`.
3. En **Settings → Pages**, elegir *Deploy from a branch*, rama `main`, carpeta `/ (root)`, y guardar.
4. Al minuto queda en `https://USUARIO.github.io/REPOSITORIO/`.

## Actualizar el corte

1. Abrir el reporte y pulsar **Modo editor** (clave `PAPERU2026`).
2. En la ventana **Ingesta**, cargar los Excel de SAP: stock, movimientos y el export de
   órdenes IW39.
3. Pulsar **Generar archivos de datos**: se descargan `base.json`, `mov.json` y `areas.json`.
4. Subirlos a la carpeta `data/` del repositorio, reemplazando los actuales.

GitHub redespliega solo. Si el navegador muestra el corte viejo, un refresco forzado
(Ctrl+F5) lo resuelve.

## La columna Área

El reporte calcula el área de cada movimiento, sin fórmulas en Excel:

1. **Por orden.** Se busca la Orden en el export IW39. Su área sale del puesto de trabajo
   responsable; si ese puesto da **PREVENTIVO**, se mira el texto breve de la orden: si
   menciona ULE, COLGAJO, PODA, MENSULA o CRUCETA queda como **ULE**, si no como **CORTE**.
2. **Por documento.** Si el movimiento no tiene orden, o esa orden no está en el IW39, se
   busca el número de documento en el histórico de áreas ya resueltas. Ese histórico lo
   construye el propio reporte: cada vez que resuelve un documento por orden, lo congela.
   Por eso no hay que mantenerlo a mano.
3. **Por texto de cabecera.** Excepciones sueltas, con coincidencia exacta.

Las tres tablas de reglas —puesto de trabajo, palabras de ULE y textos de cabecera— se editan
en la misma ventana de ingesta.

Un movimiento cuya orden todavía no aparece en ningún IW39 queda sin área y se muestra con un
guión. Se resuelve solo en cuanto se suba un IW39 que la incluya: al publicar se recalcula
**todo** el historial, no solo lo nuevo.

## Reglas de la ingesta

- Se descarta toda fila cuyo código de material empiece con **S** (materiales de servicio),
  en stock y en movimientos.
- El **stock reemplaza**: cada corte pisa al anterior.
- Los **movimientos acumulan**: solo se agregan las filas que aún no están, comparando fila
  completa. Volver a subir el mismo archivo agrega cero.
- No hacen falta tablas dinámicas ni tablas de Excel: basta la hoja con los datos. El
  encabezado se busca en cualquier hoja, dentro de las primeras 30 filas.

## Columnas que deben venir en los Excel

**Stock:** Material · Centro · Almacén · Stock especial · Número de stock especial ·
Unidad medida base · Libre utilización · Valor libre util.

**Órdenes (IW39):** Orden · Texto breve · Pto.tbjo.responsable

**Movimientos:** Documento material · Fecha contabiliz. · Clase de movimiento · Centro ·
Almacén · Material · Descripción material · Un.medida de entrada · Ctd.en UM entrada ·
Impte.mon.local · Orden · Texto clase de mov. · Nombre del usuario · Referencia ·
Texto cab.documento

Si falta alguna, la ingesta dice cuál y no genera nada.

## Almacén → área

| Almacén | Área |
|---|---|
| C113 | COMERCIAL |
| C313 | COMERCIAL |
| E113 | EMERGENCIA BT/MT/AP |
| F113 | FÍSICO |
| M113 | MTTO BT/MT/AP |
| O113 | OBRAS BT/MT/AP |

## Advertencias

- **Un sitio en GitHub Pages es público.** Cualquiera con la URL ve los valores de stock,
  sin contraseña. GitHub Pages no admite control de acceso salvo en planes Enterprise, y la
  clave del modo editor solo oculta la ventana de ingesta: no protege los datos.
- La carga de archivos por la web de GitHub admite hasta **25 MB por archivo**. A unos
  120 bytes por movimiento, eso alcanza para cerca de 200 mil filas de historial. Pasado ese
  punto hay que subir con `git` desde la computadora.
- La página usa SheetJS desde cdnjs para leer y escribir Excel, así que la ingesta y la
  descarga de Excel necesitan internet. Ver el reporte no lo necesita.
