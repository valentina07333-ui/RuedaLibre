# Rueda Libre: bitácora del proyecto

Rueda Libre es un sitio web para un servicio de alquiler de bicicletas y rutas ciclistas guiadas. Tiene una página de inicio, una de tarifas, una de reservas y una versión nocturna hecha con Tailwind. En esta bitácora voy dejando lo que fui revisando y aprendiendo mientras armaba el sitio: cómo responde el servidor, qué hace cada tecnología, cómo se conectan mis páginas, cómo está organizada cada una, qué tan accesible es, cómo se comportan mis estilos, cómo se reacomoda todo en pantallas de distinto tamaño y qué cambió al pasar de mi versión sin Bootstrap a la actual.

- **Repositorio:** https://github.com/valentina07333-ui/RuedaLibre
- **Sitio publicado:** _por agregar cuando lo publique_
- **Última actualización:** 21 de noviembre de 2026

---

## Estructura del proyecto

```
RuedaLibre/
├── index.html                          # Inicio
├── styles.css                          # Estilos propios (los usan las páginas claras)
├── pages/
│   ├── reservas.html                   # Formulario de reserva
│   └── tarifas.html                    # Tablas de tarifas y recorridos
├── RuedaLibre_Nocturna_Tailwind/
│   └── RuedaNocturna.html              # Versión nocturna con Tailwind
├── img/                                # Logos, portadas y fotos de las bicicletas
└── README.md                           # Esta bitácora
```

Para verlo en mi computador lo abro con Live Server desde VS Code. Hace falta internet porque Bootstrap, Tailwind y la tipografía *Black Ops One* se cargan desde CDN.

---

## 1. La petición al servidor

Abrí `index.html` con Live Server, entré a DevTools, pestaña **Network**, recargué la página y busqué la petición del documento (la primera de la lista, de tipo *document*). Esto es lo que quedó:

| Dato | Valor |
|---|---|
| URL completa | `http://127.0.0.1:5500/index.html` |
| Protocolo | `http` |
| Host | `127.0.0.1` (mi propio computador, también conocido como localhost) |
| Puerto | `5500` (el que usa Live Server por defecto) |
| Ruta | `/index.html` |
| Método | `GET` |
| Código de estado | `200 OK` |

El `200` quiere decir que el servidor encontró el archivo y me lo devolvió sin problema. Si en alguna recarga aparece `304`, es porque el navegador reutilizó la copia que ya tenía guardada, y también es normal.

El protocolo es `http` y no `https` porque es un servidor local de pruebas; no hay certificado de por medio.

### Probando una ruta que no existe

En la barra de direcciones escribí `http://127.0.0.1:5500/promociones.html`, un archivo que no hice. El servidor respondió **404 Not Found**.

Un 404 significa que el servidor sí está funcionando y entendió la petición, pero en esa ruta no hay nada. Es un error del lado del cliente (los códigos 4xx): la dirección está mal escrita o el archivo no existe. Se diferencia de un 200, que es éxito, y de un 5xx, que sería el servidor fallando por dentro.

---

## 2. Qué hace cada tecnología en Rueda Libre

**HTML: la estructura y el contenido.** Define qué hay en cada página. Un ejemplo concreto es el formulario de `reservas.html`: usa `fieldset` y `legend` para separar los tres pasos (contacto, detalles de la reserva y confirmación), `label` para cada campo y atributos como `required` o `type="email"` para que el navegador valide los datos antes de enviarlos.

**CSS: la apariencia.** Define cómo se ve todo eso. Por ejemplo, en `styles.css` guardé la paleta en variables (`--primary-dark: #6096BA`, `--text-primary: #253746`, etc.) y las uso en todo el sitio. También tengo `.bike-card:hover`, que levanta la tarjeta de cada bicicleta unos píxeles cuando paso el mouse por encima.

**JavaScript: el comportamiento.** Todavía no escribí JavaScript propio. Lo único que se mueve hoy es el menú que se abre y se cierra en el celular, y eso lo hace el JavaScript de Bootstrap (`data-bs-toggle="collapse"`). La idea es agregar uno mío en la página de reservas: que el campo "Nombre del recorrido" aparezca solo cuando el usuario responde que sí quiere un recorrido guiado, y que se muestre un precio estimado según la bicicleta y el horario elegidos.

---

## 3. Navegación y rutas relativas

Quise recorrer el sitio usando solo el menú, pasando de cada página a las otras. Para eso revisé todos los enlaces de las cuatro páginas y los probé contra un servidor local: todos los archivos a los que apuntan responden `200`. Pero al hacer el recorrido aparecieron problemas que no esperaba.

### Tabla de rutas

| Archivo de origen | Destino | Ruta escrita |
|---|---|---|
| `index.html` | Versión nocturna | `./RuedaLibre_Nocturna_Tailwind/RuedaNocturna.html` |
| `index.html` | Tarifas (botón "Rutas y tarifas") | `./pages/tarifas.html` |
| `index.html` | Reservas (botones "Reservar bicicleta") | `./pages/reservas.html` |
| `pages/reservas.html` | Inicio | `../index.html` |
| `pages/reservas.html` | Versión nocturna | `../RuedaLibre_Nocturna_Tailwind/RuedaNocturna.html` |
| `pages/reservas.html` | Tarifas (botón "Consultar tarifas") | `./tarifas.html` |
| `pages/tarifas.html` | Inicio | `../index.html` |
| `pages/tarifas.html` | Versión nocturna | `../RuedaLibre_Nocturna_Tailwind/RuedaNocturna.html` |
| `pages/tarifas.html` | Reservas (botón "Haz una reserva") | `reservas.html` |
| `RuedaLibre_Nocturna_Tailwind/RuedaNocturna.html` | Inicio | `../index.html` |
| `RuedaLibre_Nocturna_Tailwind/RuedaNocturna.html` | Reservas | `../pages/reservas.html` |

La regla que fui entendiendo: `./` es la carpeta donde está el archivo actual, `../` sube una carpeta y el resto baja hasta el destino. Por eso desde `pages/` necesito `../index.html` para volver a la raíz, y desde la raíz necesito `./pages/tarifas.html` para entrar.

### Lo que encontré

- **Con el menú solo no se llega a todas las páginas.** El menú del inicio tiene "Tarifas" y "Reservar ahora", pero son anclas dentro de la misma página (`#tarifas` y `#reservas`), no llevan a `pages/tarifas.html` ni a `pages/reservas.html`. Y entre Tarifas y Reservas no hay ningún enlace en el menú; solo se pasa de una a otra con los botones del contenido.
- **El botón "Reservar ahora" de `reservas.html` apunta a `#reservas`, y esa página no tiene ningún elemento con ese id**, así que no hace nada. En `tarifas.html` sí existe `#reservas`, pero es la sección de información importante, no el formulario.

**Cómo lo voy a corregir:** poner en el menú de las tres páginas enlaces reales a Inicio, Tarifas y Reservas, y hacer que "Reservar ahora" lleve siempre a `reservas.html`.

---

## 4. Esquema de regiones de cada página

Antes de revisar el código dibujé en texto cómo está organizada cada página.

**Inicio (`index.html`)**

```
header
└─ nav (logo, enlaces, botón "Reservar ahora")
main
├─ section #inicio      → h1, texto, dos botones e imagen
├─ section #nosotros    → h2 y tarjeta "Nuestra filosofía" (h3)
├─ section #rutas       → h2 y 3 article (rutas)
├─ section #tarifas     → h2 y 3 article (bicicletas con precio)
└─ section #reservas    → llamado a la acción
footer
```

**Reservas (`pages/reservas.html`)**

```
header
└─ nav
main
├─ section              → h1 de bienvenida
└─ section
   ├─ form              → 3 fieldset (contacto, reserva, confirmación)
   └─ aside #tarifas    → guía "¿Cómo reservar?" con 4 pasos
footer
```

**Tarifas (`pages/tarifas.html`)**

```
header
└─ nav
main
├─ section              → h1
├─ section              → tabla de alquiler por tiempo
├─ section              → tabla de recorridos nocturnos
└─ section #reservas    → información importante y botón de reserva
footer
```

**Rueda Nocturna (`RuedaNocturna.html`)**

```
header
└─ nav
main
├─ section #home        → h1
├─ section #about
├─ section #events      → 3 article
├─ section #tarifas     → 3 article de bicicletas
├─ section              → 3 article con las portadas
└─ section #contact
footer
```

### Comprobación en DevTools (Elements)

Revisé que el árbol coincida con el esquema. En las cuatro páginas, cada `div` que se abre tiene su cierre (inicio 44/44, reservas 39/39, tarifas 26/26 y nocturna 35/35), así que ningún elemento queda metido en uno que no le corresponde. La única diferencia con el esquema es que en `reservas.html` el `aside` cumple la función de contenido complementario pero lleva el id `tarifas`, que es un nombre confuso porque ahí no hay tarifas, solo la guía de pasos.

---

## 5. Auditoría de accesibilidad

Revisé las páginas con la lista de la actividad: textos alternativos, jerarquía de encabezados, textos de enlace, idioma y teclado.

| # | Hallazgo | Corrección |
|---|---|---|
| 1 | **Idioma:** las cuatro páginas tienen `lang="es"`, título y descripción propios. | Nada que corregir. |
| 2 | **Alt incorrecto:** en `index.html`, la bicicleta Gravel usa `bike_urban.jpg` con el alt "Bicicleta deportiva", que no dice qué bicicleta es. | Cambiarlo a "Bicicleta Gravel Aventura". |
| 3 | **Alt incorrecto:** en `RuedaNocturna.html`, la primera imagen después del logo es `03_bicicleta_urbana.png` pero el alt dice "Bicicleta de montaña". | Cambiarlo a "Bicicleta urbana". |
| 4 | **Logos:** el isotipo y el logo oscuro llevan el alt "Rueda Libre" (o "Logo de Rueda Libre") y están dentro de un enlace al inicio; el alt sí aporta, porque es lo que describe el enlace. | Dejarlos así. |
| 5 | **Jerarquía de encabezados en `index.html`:** después de un `h2` las tarjetas de rutas y bicicletas usan `h5`, saltándose `h3` y `h4`. | Pasarlas a `h3` y ajustar el tamaño visual con CSS. |
| 6 | **Encabezados del footer:** el nombre "Rueda Libre" del pie de página es un `h5` en las cuatro páginas, aunque no es un título de contenido. | Cambiarlo a un `p` con negrita. |
| 7 | **Texto de enlace repetido:** en la flota hay tres botones que dicen solo "Reservar bicicleta"; fuera de contexto no se sabe cuál es cuál. | Cambiarlos a "Reservar la Urbana Clásica", "Reservar la Mountain Bike" y "Reservar la Gravel Aventura" (o agregar un `aria-label`). |
| 8 | **Ancla rota:** "Reservar ahora" en `reservas.html` apunta a `#reservas`, que no existe (ver sección 3). | Apuntarlo a `reservas.html`. |
| 9 | **Teclado:** el botón del menú es un `button` real y todos los enlaces son `a` con `href`, así que se pueden recorrer con Tab. No quité el contorno de foco en ningún estilo. | Falta hacer el recorrido completo con Tab en el navegador para confirmar el orden y que el menú desplegable se pueda abrir con Enter. |

### Contraste de colores

Los colores de mi sitio no son los de la lista de la actividad (Verde Ruta, Naranja Pedal, etc.): mi paleta es azul, así que medí las combinaciones que realmente uso. Calculé la relación de contraste con la fórmula de WCAG. Para texto normal se necesita al menos 4.5:1; para texto grande (desde 24 px, o 18.66 px en negrita), 3:1.

| Combinación | Relación | Clasificación | Dónde la uso |
|---|---|---|---|
| `#253746` sobre blanco | 12.26 | Apta para texto normal | Texto general y títulos |
| `#253746` sobre `#F4F9FD` | 11.56 | Apta para texto normal | Secciones con fondo suave |
| `#253746` sobre `#EAF4FB` | 10.99 | Apta para texto normal | Tarjeta "Nuestra filosofía" |
| `#253746` sobre `#A2D2FF` | 7.70 | Apta para texto normal | Insignia del inicio |
| `#253746` sobre `#BDE0FE` | 8.90 | Apta para texto normal | Insignias de las bicicletas |
| Blanco sobre `#253746` | 12.26 | Apta para texto normal | Pie de página |
| `#6C757D` sobre blanco | 4.69 | Apta para texto normal | Textos secundarios de Bootstrap |
| `#6C757D` sobre `#F4F9FD` | 4.42 | Solo texto grande (por muy poco) | Textos secundarios en "Nuestras rutas" |
| `#607D8B` sobre blanco | 4.37 | Solo texto grande (por muy poco) | "/ hora" junto al precio |
| `#6096BA` sobre blanco | 3.20 | Solo texto grande | Etiquetas, fechas, precios, enlaces |
| `#6096BA` sobre `#F4F9FD` | 3.02 | Solo texto grande (al límite) | Etiquetas de sección sobre fondo suave |
| `#6096BA` sobre `#EAF4FB` | 2.87 | No apta | Poco uso, pero hay que evitarla |
| Blanco sobre `#6096BA` | 3.20 | Solo texto grande | Botones principales |
| Blanco sobre `#4F83A5` | 4.10 | Solo texto grande | Botones al pasar el mouse |
| Blanco sobre `#8ECAE6` | 1.79 | No apta | Final del degradado de "¿Listo para comenzar?" |

**Colores de la guía de marca:** _pendiente. Me falta agregar los códigos de Verde Ruta, Naranja Pedal, Azul Noche, Crema y Gris Asfalto para medir esas ocho combinaciones._

### Mapa de uso de color

- **Texto:** `#253746` para todo el texto importante. El gris secundario (`#6C757D`) solo sobre fondo blanco.
- **Fondos:** blanco, `#F4F9FD` y `#EAF4FB` para dar variedad sin perder lectura. El `#253746` queda para el pie de página.
- **Botones:** hoy son texto blanco sobre `#6096BA` (3.20), que solo sirve para texto grande. Voy a oscurecer ese azul hasta pasar 4.5:1.
- **Enlaces y etiquetas:** `#6096BA` solo en tamaños grandes (precios, títulos). Las etiquetas pequeñas en mayúsculas (`.section-label`) y las fechas de las rutas deberían pasar a un azul más oscuro o al `#253746`.
- **Degradado de la sección final:** el texto blanco no se lee sobre el extremo claro (`#8ECAE6`). Voy a oscurecer el degradado o cambiar el color del texto.

---

## 6. Herencia y especificidad en DevTools

### Herencia

En **Styles** y **Computed** miré un párrafo del sitio (el texto de "Nuestra filosofía") y qué le llega desde `body`. En mi `styles.css` el `body` define `font-family`, `color` y `font-weight`.

- **Se hereda:** `font-family`. El párrafo no tiene ninguna regla de fuente propia y aun así se ve en *Black Ops One*, porque la toma de `body`. Pasa lo mismo con el `color`.
- **No se hereda:** `padding`. La sección tiene `.section-padding` con 90px arriba y abajo, pero los párrafos de adentro no heredan ese espacio. Otro ejemplo es `margin-bottom`: el 1rem que tiene el `p` lo pone Bootstrap directamente sobre `p`, no viene del `body`.

### Un conflicto de selectores

El enlace activo del menú (`.nav-link.active`) tiene un conflicto con Bootstrap, que también le da color a ese mismo enlace:

| Regla | Selector | Especificidad (id, clase, etiqueta) |
|---|---|---|
| Bootstrap | `.navbar-nav .nav-link.active` | (0, 3, 0) |
| Mía | `.nav-link.active` | (0, 2, 0) |

Por especificidad, la de Bootstrap gana (3 clases contra 2), aunque mi hoja se cargue después. El orden solo desempata cuando la especificidad es igual. Por eso en mi regla puse `!important`, que se salta ese cálculo. Funciona, pero no es lo más limpio: si algún día necesito cambiar ese color, tendría que pelear contra mi propio `!important`. Una forma mejor sería escribir un selector más específico, como `.navbar-nav .nav-link.active`, y quitar el `!important`.

---

## 7. Cuántas columnas se ven según el ancho

Probé el sitio en modo dispositivo con tres anchos: 1280 px (computador), 900 px (tablet) y 500 px (celular). Los números salen de los puntos de quiebre de las clases que uso: en Bootstrap `md` empieza en 768 px y `lg` en 992 px; en Tailwind `sm` empieza en 640 px, `md` en 768 px y `lg` en 1024 px. Por eso 900 px queda en el medio en las páginas claras: ya pasó el `md` pero todavía no llega al `lg`.

| Página | Bloque | 1280 px | 900 px | 500 px |
|---|---|---|---|---|
| Inicio | Menú | Horizontal | Botón hamburguesa | Botón hamburguesa |
| Inicio | Portada (texto e imagen) | 2 columnas | 1 | 1 |
| Inicio | Nosotros | 2 columnas | 1 | 1 |
| Inicio | Rutas (tarjetas) | 3 columnas | 2 | 1 |
| Inicio | Flota (tarjetas) | 3 columnas | 2 | 1 |
| Inicio | Pie de página | 2 columnas | 2 | 1 |
| Reservas | Formulario y guía de pasos | 2 columnas | 1 | 1 |
| Reservas | Campos del formulario | 2 columnas | 2 | 1 |
| Tarifas | Tablas | 1 (con desplazamiento lateral si no caben) | 1 | 1 |
| Tarifas | Información y botón de reserva | 2 columnas | 1 | 1 |
| Rueda Nocturna | Menú de enlaces | Visible | Visible | Oculto |
| Rueda Nocturna | Portada | 2 columnas | 2 | 1 |
| Rueda Nocturna | Eventos y tarifas | 3 columnas | 3 | 1 |
| Rueda Nocturna | Galería | 3 columnas | 2 | 1 |

Lo que más me llamó la atención es que Bootstrap y Tailwind no cambian de layout en el mismo punto: en las páginas claras casi todo se apila hasta los 992 px, y en la nocturna ya se reparte en columnas desde los 768 px.

---

## 8. Flexbox o Grid en cada caso

El criterio que usé es sencillo: si estoy acomodando elementos en **una dimensión** (una fila o una columna) uso Flexbox; si necesito controlar **dos dimensiones** a la vez (filas y columnas alineadas entre sí) uso Grid.

Un detalle importante: en las páginas claras no escribí `display: flex` ni `display: grid` en mi `styles.css`. Eso lo hace Bootstrap, y su `.row` y `.col-*` funcionan con Flexbox por debajo. En la versión nocturna sí uso las dos cosas directamente, con clases de Tailwind.

| Caso | Lo que usé | Por qué |
|---|---|---|
| Portada | Flexbox en el inicio (`row align-items-center`); Grid en la nocturna (`grid md:grid-cols-2`) | Es una sola fila con dos bloques, texto e imagen, centrados verticalmente: una dimensión, así que Flexbox es lo natural. En la nocturna usé Grid porque declara las dos columnas iguales de entrada, aunque aquí Flexbox también habría alcanzado. |
| Galería | Grid en la nocturna (`grid sm:grid-cols-2 lg:grid-cols-3`); en el inicio, las tarjetas de la flota con `row g-4` y `col-md-6 col-lg-4` | Las tarjetas se organizan en filas y columnas que tienen que quedar alineadas y cambian de 1 a 2 y a 3 columnas: son dos dimensiones, el caso de Grid. En el inicio Bootstrap lo resuelve con Flexbox que salta de línea, y por eso necesito `h-100` para que todas las tarjetas de una fila midan lo mismo. |
| Reservas (formulario y guía) | Flexbox (`row g-4 align-items-start` con `col-lg-8` y `col-lg-4`) | Son dos bloques lado a lado de alturas distintas, uno ancho y otro angosto. Solo importa el eje horizontal: una dimensión. |
| Fila del formulario | Flexbox (`row g-3` con `col-md-6`) | Cada fila es un par de campos que se parten solos en móvil. Los campos son independientes entre filas, así que no necesito que las columnas se alineen en una cuadrícula: una dimensión. |
| Encabezado | Flexbox (`navbar` en el inicio; `flex items-center justify-between` en la nocturna) | Logo a un lado y enlaces con botón al otro, alineados en una línea. Es una fila, una dimensión. |
| Pie | Flexbox (`row align-items-center` en el inicio; `flex flex-col md:flex-row` en la nocturna) | Dos bloques en una fila que en móvil se apilan. La nocturna cambia la dirección del eje (columna a fila), que es justo lo que hace bien Flexbox: una dimensión. |

---

## 9. Probando en 360, 768 y 1280 px

Revisé el sitio con el modo dispositivo a 360 px (celular pequeño), 768 px (tablet) y 1280 px (computador). Estos fueron los problemas que encontré:

| # | Problema | Dónde | Solución | Estado |
|---|---|---|---|---|
| 1 | **El menú desaparece en celular.** Los enlaces tienen `hidden ... md:flex` y no hay botón de menú, así que por debajo de 768 px solo queda el botón "Reservar ahora" y no hay cómo ir a Inicio o a Tarifas. | `RuedaNocturna.html`, a 360 px | Agregar un botón de hamburguesa que muestre y oculte la lista de enlaces (o dejar los enlaces en una fila que se pueda desplazar). | Por aplicar |
| 2 | **Las tablas no caben.** La de recorridos tiene cinco columnas y a 360 px se saldría de la pantalla y rompería el ancho de toda la página. | `tarifas.html`, a 360 px | Envolver cada tabla en `table-responsive`: la tabla se desplaza de lado dentro de su caja y la página no se rompe. | Aplicado |
| 3 | **Una regla móvil que no llega al inicio.** El `@media (max-width: 768px)` de mi CSS apunta a `.hero-section`, pero la portada del inicio usa la clase `.hero`, así que esos ajustes (tamaño del título y espacios) solo funcionan en Reservas y Tarifas. | `styles.css` e `index.html`, a 360 y 768 px | Cambiar el selector a `.hero, .hero-section` o usar la misma clase en las tres páginas. | Por aplicar |
| 4 | **La guía de reserva queda al final.** Por debajo de 992 px la guía "¿Cómo reservar?" se apila debajo del formulario, y en celular hay que recorrer todos los campos para encontrarla. | `reservas.html`, a 360 y 768 px | Subirla con `order-first order-lg-last`, o dejarla debajo y poner un enlace corto arriba. | Por decidir |

---

## 10. Conflictos entre mis estilos y los de Bootstrap

Abrí el panel Styles de DevTools buscando declaraciones tachadas. Estos son tres conflictos y cómo los resolví:

**1. Fuente y color del `body`.**
Bootstrap define `font-family` y `color` en `body` con sus variables (`--bs-body-font-family`, `--bs-body-color`), y yo defino los míos en el mismo selector. Tienen la misma especificidad (0, 0, 1), así que decide el orden: en Styles las de Bootstrap aparecen tachadas y gana la mía porque `styles.css` se carga después. **Solución:** dejar siempre el `<link>` de `styles.css` debajo del de Bootstrap, como está en las cuatro páginas.

**2. Los botones `.btn-primary`.**
Sobrescribí `--bs-primary` en `:root` y con eso cambiaron `.text-primary` y las insignias, pero los botones siguieron con el azul de Bootstrap, porque `.btn-primary` trae sus propias variables de color (`--bs-btn-bg`) y no lee `--bs-primary`. **Solución:** escribí `.btn-primary` y `.btn-primary:hover` con mi color. Tienen la misma especificidad que las de Bootstrap y ganan por llegar después. Me queda pendiente revisar el estado de botón presionado, que puede seguir mostrando el azul original.

**3. El gris de los textos secundarios.**
En mi paleta definí `--text-secondary: #607D8B`, pero en el HTML uso la clase `text-secondary` de Bootstrap, que lleva `!important` y pinta `#6C757D`. Por eso mi gris casi no se ve: solo aparece en el "/ hora" del precio. **Solución:** en vez de pelear con el `!important`, sobrescribir `--bs-secondary-rgb` en `:root` con el gris que quiero (ya oscurecido para que pase el contraste).

Además está el enlace activo del menú (`.nav-link.active`), que expliqué en la sección 6: ahí sí tuve que usar `!important`.

---

## 11. Comparativa: mi versión sin Bootstrap contra la actual

En el historial de GitHub no encontré una etiqueta `v1.5`, así que tomo como versión base el último commit antes de Bootstrap (`f3632c7`) y la comparo con la versión actual (`3c4be6e`). Todo el proyecto se hizo el 17 de noviembre; los tiempos salen de la hora de los commits, así que son una aproximación.

| | Sin Bootstrap (`f3632c7`) | Con Bootstrap y personalización (`3c4be6e`) |
|---|---|---|
| Tiempo | 49 min (de 19:58 a 20:47, cuando empieza Bootstrap) | 2 h 10 min con Bootstrap (de 20:47 a 22:57) y 29 min más con Tailwind (hasta 23:26) |
| `styles.css` | 41 líneas (33 con contenido), solo variables de color | 227 líneas (186 con contenido) |
| `index.html` | 93 líneas | 425 líneas |
| CSS de distribución de la flota | 0 líneas: las bicicletas eran una lista de `article` sin estilos | 0 líneas propias de distribución. La distribución sale de 10 clases de Bootstrap (`row`, `g-4`, `col-md-6`, `col-lg-4`, `h-100`, `d-flex`, `flex-column`, `mt-auto`, `w-100`, `align-self-start`). Mi CSS tiene 43 líneas (36 con contenido) solo de aspecto de las tarjetas (`.bike-*`). |
| Clases distintas en el HTML | Casi ninguna | 132 en las tres páginas claras; solo 24 tienen reglas en mi `styles.css` |

**Lo que me dio Bootstrap sin esfuerzo:**
- Cuadrícula responsive: con dos clases por tarjeta pasé de 1 a 2 y a 3 columnas.
- Menú que se colapsa en celular, con su botón y su comportamiento, sin escribir nada de JavaScript.
- Formularios, tablas, botones y tarjetas con un aspecto consistente desde el primer momento.
- Utilidades de espacio y alineación (`mt-auto`, `gap-3`, `align-items-center`) que me ahorraron decenas de líneas de CSS.

**El control que perdí:**
- Para cambiar un color a veces hay que ganarle a Bootstrap por especificidad, y terminé usando `!important` (en el menú).
- Mi paleta no llega a todo: el gris secundario y los estados de los botones siguen siendo los de Bootstrap.
- El HTML se llenó de clases y quedó mucho más largo, y casi todas son de Bootstrap, no mías.
- Dependo de un CDN: sin internet, la página se ve rota.
- Se parece a cualquier otro sitio hecho con Bootstrap si no lo personalizo a fondo.

**Ventajas y desventajas, en corto.** Con Bootstrap avancé mucho más rápido en lo difícil, que es la distribución responsive, pero mi `styles.css` igual creció de 41 a 227 líneas, porque personalizarlo toma su tiempo y exige entender la especificidad.

**Conclusión.** La comparación no es del todo justa: la versión sin Bootstrap era un HTML casi sin estilos, así que no sé cuánto me habría tomado hacer a mano la misma distribución. Lo que sí veo es que Bootstrap me dejó dedicar mi CSS a lo que le da identidad al sitio (paleta, tarjetas, sombras, tipografía) en lugar de resolver columnas. La próxima vez voy a empezar por sobrescribir las variables de Bootstrap en `:root` antes de escribir selectores propios, para pelear menos con él.

---

## 12. Cosas que me faltan por arreglar

Pendientes que fui anotando mientras revisaba:

- Varias clases que usé en `reservas.html` y `tarifas.html` no existen en mi `styles.css` (`reservation-card`, `reservation-info`, `step-number`, `pricing-card`, `pricing-table`, `difficulty`, `pricing-info`, `pricing-details`), así que esas partes solo muestran el estilo base de Bootstrap.
- Las bicicletas no coinciden entre páginas: el inicio ofrece Urbana, Montaña y Gravel, pero el formulario y las tarifas hablan de Urbana, Montaña y Eléctrica.
- Los problemas 1, 3 y 4 de la sección 9 y el estado de botón presionado de la sección 10 siguen sin aplicarse.
- El pie de página dice 2024.
- El formulario se envía con `mailto:`, o sea que solo abre el programa de correo; no hay un envío real.
- La imagen `gravel.jpg` solo se usa en la versión nocturna; en el inicio la Gravel usa `bike_urban.jpg`.

---

## Cierre

### Antes y después de personalizar

| Antes: Bootstrap sin personalizar (commit `1a2918c`) | Después: versión actual |
|---|---|
| ![Inicio antes de personalizar](img/capturas/antes-inicio.png) | ![Inicio después de personalizar](img/capturas/despues-inicio.png) |

### Resumen de la semana

**Lo que construí.** Un sitio de cuatro páginas con la misma identidad visual: un inicio con la marca, las rutas y la flota; una página de tarifas con dos tablas; un formulario de reservas con validación del navegador; y una versión nocturna hecha con Tailwind para comparar una forma de trabajar con otra.

**Lo que aprendí en cada parte de la semana.**
- *Cómo funciona la web:* qué le pide el navegador al servidor y qué responde (200 y 404), y cómo se escriben las rutas relativas entre carpetas.
- *Estructura y accesibilidad:* organizar cada página en regiones, revisar alt, encabezados y contraste, y entender por qué a veces mis estilos no ganan (herencia y especificidad).
- *Diseño responsive:* cuántas columnas se ven según el ancho, cuándo usar Flexbox y cuándo Grid, y los problemas que aparecen en 360 px.
- *Bootstrap:* qué me ahorra y a qué le tengo que ganar, y la comparación con mi versión sin él.

**Lo que mejoraría.**

1. Arreglar la navegación para que desde el menú se llegue a todas las páginas y agregar un menú de celular a la versión nocturna.
2. Oscurecer los azules de botones, etiquetas y enlaces para que cumplan el contraste.
3. Corregir los alt, la jerarquía de encabezados y el selector `.hero` que no recibe los ajustes móviles.
4. Terminar los estilos que faltan en reservas y tarifas.
5. Sobrescribir las variables de Bootstrap en `:root` y quitar el `!important` del menú.
6. Agregar un poco de JavaScript propio al formulario.
