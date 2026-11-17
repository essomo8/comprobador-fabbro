# Comprobador de códigos y precio de FABBRO · SOLA

Herramienta web para revisar un proyecto exportado de Coohom (JSON) contra el maestro de artículos de FABBRO:

- productos sin código,
- códigos que no existen en FABBRO,
- descripciones de FABBRO que no coinciden con el nombre de Coohom,
- artículos sin precio de compra o sin precio de venta.

Muestra el resultado en pantalla, deja ver el Excel antes de descargarlo y genera el `.xlsx` en el propio navegador. No necesita servidor ni instalación.

## Archivos

| Archivo | Para qué sirve |
|---|---|
| `index.html` | La aplicación completa (un solo archivo). |
| `config.json` | Configuración compartida: URL de la API, campos, criterios. **Nunca lleva la clave.** |

## Publicarla en GitHub Pages

1. Crea un repositorio en GitHub (por ejemplo `comprobador-fabbro`) y sube `index.html`, `config.json` y este `README.md`.
2. En el repositorio: **Settings → Pages → Build and deployment → Source: Deploy from a branch**, rama `main`, carpeta `/ (root)`. Guarda.
3. Al cabo de un minuto la app estará en `https://<tu-usuario>.github.io/comprobador-fabbro/`.

> En las cuentas gratuitas, GitHub Pages solo funciona con repositorios **públicos**: cualquiera con el enlace puede abrir la app y leer `config.json` (la URL de la API y los nombres de los campos). Sin la clave no puede leer datos de FABBRO. Con GitHub Pro, Team o Enterprise se puede publicar desde un repositorio privado.

## API de FABBRO

La app ya viene configurada para la API pública de FABBRO (`https://api.fabbroapp.es/ext/v1`): solo hace falta la clave.

**Crear la clave:** FABBRO → Ajustes → API y webhooks → **Nueva clave**, con estos permisos (solo lectura):

| Permiso | Para qué |
|---|---|
| `products:read` | Leer el catálogo: si el código existe, descripción, precio de venta de la ficha, si está activo. |
| `sales-orders:read` | Enviar el JSON de Coohom a `kitchen-orders/preview` (valora el diseño sin crear pedido). |
| `costs:read` | Leer el precio de compra (`unitCost`). Sin este permiso, FABBRO lo devuelve siempre vacío y la app avisa de que no se ha comprobado. |

La clave va en la cabecera `x-api-key`, sin prefijo.

**Cómo trabaja (modo «Enviar el JSON a FABBRO», por defecto):**

1. Envía el JSON a `POST /kitchen-orders/preview` con `{ filename, format: "applet-json", design }` (si FABBRO rechaza ese formato, prueba `param-model`).
2. Toma cada componente de `data.price.breakdown`: tableros, cantos, herrajes… con su código, nombre en el diseño, tipo, unidad, cantidad, precio de la tarifa de venta y el mueble en el que aparece.
3. Descarga el catálogo `GET /products?limit=200` siguiendo `meta.nextCursor` y cruza cada código: existe o no, descripción, precio de compra, activo o inactivo.

Los otros dos modos (catálogo completo leyendo el JSON con la app, o código a código con `/products?code={codigo}`) siguen disponibles en Configuración.

## Configurarla (administrador)

1. Abre la app publicada y entra en **Configuración**.
2. Escribe la clave en «Clave de acceso». Las URL y los campos ya vienen puestos para FABBRO.
3. Pulsa **Probar conexión** (lee el catálogo) y **Probar la vista previa con un JSON…** (envía un JSON de Coohom).
4. Opcional: fija los campos del JSON de Coohom, ajusta los porcentajes de similitud y pon una contraseña para entrar en Configuración.
5. Pulsa **Guardar** y después **Descargar config.json**.
6. Sustituye `config.json` en el repositorio por el descargado (en GitHub: abrir el archivo → *Upload files* o editar y pegar). Todos los que abran la app usarán esa configuración.

Los ajustes guardados en tu navegador tienen prioridad sobre `config.json`. Para volver a la configuración publicada: **Configuración → Volver a la configuración publicada**.

## La clave de acceso

- La clave **no va en `config.json` ni en el repositorio**.
- Cada compañero la escribe una vez (la app la pide en la pantalla principal) y queda guardada en su navegador.
- O bien: en **Configuración → Copiar enlace con clave**, genera un enlace que guarda la clave al abrirse. Envíalo solo por un canal privado. El enlace se limpia de la barra de direcciones en cuanto se abre.
- Si la clave cambia, cada compañero puede actualizarla en Configuración → Clave de acceso, o abrir un enlace nuevo.

## Permitir las llamadas desde el navegador (CORS)

La app llama a la API de FABBRO directamente desde el navegador. El servidor de FABBRO tiene que permitir el origen de la app. Pide al desarrollador de FABBRO que responda con estas cabeceras para `https://<tu-usuario>.github.io`:

```
Access-Control-Allow-Origin: https://<tu-usuario>.github.io
Access-Control-Allow-Headers: x-api-key, Accept, Content-Type
Access-Control-Allow-Methods: GET, POST, OPTIONS
```

y que la petición previa `OPTIONS` responda `204` sin pedir clave. Si se usa la clave como parámetro en la URL (en lugar de cabecera), basta con `Access-Control-Allow-Origin`.

Mientras tanto, se puede trabajar con una exportación de artículos de FABBRO (JSON o CSV): **Configuración → Usar una exportación de FABBRO…**.

## Trabajar las correcciones

1. Carga el JSON y pulsa **Comprobar productos**.
2. Revisa por estado, por incidencia o por tipo (tablero, canto, herraje). Abre cada ficha para ver la comparación de textos, los precios y el artículo sugerido (con botón para copiar su código).
3. Corrige en FABBRO o en Coohom.
4. Pulsa **Volver a comprobar**: vuelve a leer FABBRO con el mismo proyecto, mantiene abiertas las fichas y te dice cuántas incidencias quedan y cuántas se han corregido.

## Criterio de las descripciones

La similitud entre el nombre de Coohom y la descripción de FABBRO va de 0 a 100 %. No tiene en cuenta mayúsculas, acentos, el orden de las palabras ni palabras como «de», «con» o «cm».

- Coincide: 85 % o más (configurable).
- A revisar: entre 60 % y 84 % (configurable).
- No coincide: por debajo de 60 %.

Cuando falta el código o no existe, la app propone el artículo de FABBRO con la descripción más parecida (solo en el modo de catálogo completo).

## Seguridad

- La contraseña de Configuración solo evita cambios por error: cualquiera con conocimientos técnicos puede saltársela. La protección real de los datos es la clave de la API.
- Los JSON de Coohom y los resultados se procesan en el navegador; no se envían a ningún sitio salvo las consultas a FABBRO.
