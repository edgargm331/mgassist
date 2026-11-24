# Guardia Assist · núcleo multi-especialidad (v28)

Una sola base de código con la interfaz Base44 y una **app por especialidad**:

| App | Carpeta | Estado |
|-----|---------|--------|
| **TyO Assist** — Traumatología y Ortopedia | `dist/tyo/` | v27, contenido auditado por ti |
| **MG Assist** — Medicina General | `dist/mg/` | v1, **borrador para validar** (`perfiles/mg/VALIDAR.md`) |

Todas comparten interfaz, notas NOM-004, filtro de datos personales (LFPDPPP), grabadora e IA, historial, modo sin conexión y módulos activables. Una mejora al núcleo llega a todas las especialidades.

## Qué hay de nuevo en v28

- **Perfiles por especialidad** (`perfiles/<id>/`): nombre, icono, configuración y contenido clínico propios.
- **Motores de datos:** las calculadoras y los protocolos de receta nuevos se describen como datos (campos, cálculo, interpretación, pruebas) y la app los dibuja con el diseño Base44.
- **Dos tipos de nota nuevos en el núcleo:** Historia clínica (NOM-004 6.1) y Referencia/traslado (6.4). Cada especialidad elige qué tipos muestra.
- **Datos separados:** si publicas varias apps en el mismo dominio, cada una guarda su historial, borradores y ajustes por separado, y tiene su propio caché sin conexión.
- **TyO sin cambios para el usuario:** conserva sus datos guardados y pasa todas sus pruebas (29/29 en el autodiagnóstico, más 20 y 48 pruebas de interacción).

## MG Assist v1 (Medicina General)

- **Calculadoras:** Función renal, Glasgow, Centor/McIsaac, CURB-65 y qSOFA. En «+ Más escalas» están IMC/SC, líquidos de Holliday-Segar, edad gestacional/FPP y Wells para TVP.
- **Recetas (dosis por peso en niños):** fiebre y dolor, faringoamigdalitis, cistitis no complicada, diarrea con planes A/B/C, dispepsia/ERGE y lumbalgia aguda. Tienen alertas por alergia, embarazo, riesgo GI, hepatopatía y TFG. La TFG es opcional.
- **Notas:** Ingreso, Historia clínica, Evolución (consulta subsecuente), Referencia y Alta, con frases rápidas y glosario de medicina general.
- **Autodiagnóstico:** 50 pruebas, incluidas dosis pediátricas clave.

> ⚠️ El contenido de MG está basado en guías citadas, pero **debe validarlo un médico adscrito** antes de usarse con pacientes. Imprime o comparte `perfiles/mg/VALIDAR.md` (trae las salidas reales para adulto, niño y lactante) y aplica las correcciones en `perfiles/mg/contenido.js`.

## Estructura

```
├── src/                       NÚCLEO (compartido) + contenido de TyO
│   ├── config.js              configuración de TyO
│   ├── html/  css/  js/       interfaz y lógica
│   │   └── js/calculadoras/motor.js · js/recetas/motor.js   motores de datos
├── perfiles/
│   ├── tyo/perfil.json        TyO usa src/ tal cual
│   └── mg/
│       ├── perfil.json        qué archivos quita/agrega el build
│       ├── config.js          nombre, especialidad, tipos de nota, módulos
│       ├── contenido.js       calculadoras, protocolos, frases, glosario, pruebas
│       ├── html/              pantallas que reemplazan las de src/ (inicio, recetas, encabezado)
│       ├── icons/  manifest.json
│       └── VALIDAR.md         hoja de validación clínica (se genera)
├── build.py                   src/ + perfiles/ → dist/<id>/
├── dist/tyo/  dist/mg/        LO QUE SE PUBLICA
└── tools/                     pruebas en Chrome headless y generadores
```

## Publicar

Cada carpeta de `dist/` es una app completa. Con GitHub Pages desde el iPhone:

1. Crea un repositorio (p. ej. `guardia`) y sube las carpetas `tyo` y `mg` de `dist/` (*Add file → Upload files*).
2. Ve a *Settings → Pages → main / root*.
3. Abre en Safari `https://TU-USUARIO.github.io/guardia/mg/` (o `/tyo/`) y toca *Compartir → Agregar a inicio*.

Cada app se instala con su propio icono y conserva sus propios datos.

## Editar y probar

```
python3 build.py --check              # todas las especialidades
python3 build.py --perfil mg --check  # solo una
node tools/validacion.js mg           # regenera la hoja de validación
```

Para las pruebas, sirve `dist/<id>` con `python3 -m http.server` y corre `tools/smoke.js`, `tools/test_mg.js` (MG), y `tools/test_v26.js` y `tools/test_v27.js` (TyO). Requieren `npm i playwright`.

### Agregar una calculadora a MG
Agrega un objeto a `calculadoras` en `perfiles/mg/contenido.js` con `id`, `titulo`, `campos`, `calcular(v)` y `pruebas`. Copia uno existente como modelo. La tarjeta, el panel, la búsqueda, el interruptor en Módulos y su prueba en el autodiagnóstico aparecen solos.

### Agregar un protocolo de receta
Agrega un objeto a `protocolos` con `generar({ p, x, h })`. Ahí `h.porPeso(mg/kg, peso, máximo)` y `h.susp(mg, mg/mL)` calculan dosis y mL. Agrega su prueba en `pruebas()`.

### Crear otra especialidad (p. ej. Pediatría, GyO, Cirugía)
1. Copia `perfiles/mg/` como `perfiles/ped/`.
2. En `perfil.json` cambia `id` y `titulo`. En `config.js` cambia `especialidad`, `storagePrefix: "ped:"` y `notas.tipos`. En `manifest.json` cambia el nombre.
3. Reemplaza el contenido de `contenido.js`. Genera los iconos con `node tools/icons.js perfiles/ped/icons PED esteto`.
4. Ejecuta `python3 build.py --perfil ped --check` y publica `dist/ped/`.

## Mejoras posibles

| # | Mejora | Para qué |
|---|--------|----------|
| 1 | Validar MG con adscrito y marcarlo como «auditado» | Usarlo en guardia real |
| 2 | Más contenido MG: crisis asmática, HAS/DM2 en consulta, otitis, IVRS pediátrica, certificados | Cubrir la consulta diaria |
| 3 | Pasar las calculadoras y recetas de TyO al formato de datos | Un solo motor para todas |
| 4 | Siguiente especialidad (Pediatría o GyO) | Reutilizar el núcleo |
| 5 | Plantilla del Puente Backend (Cloudflare Worker) dentro del proyecto | IA segura en minutos |

Cada mejora clínica debe validarse con la bibliografía y la normativa vigentes antes de usarse con pacientes.
