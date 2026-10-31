# Integración con Claude Design

## Referencia visual aprobada

| Campo | Valor |
|---|---|
| Proyecto | `5d994040-ff6b-472b-9ca0-cbce5b162fcc` |
| URL | https://claude.ai/design/p/5d994040-ff6b-472b-9ca0-cbce5b162fcc?file=FINORA.dc.html |
| Archivos seleccionados | `FINORA.dc.html`, `support.js` |
| Estado de importación | ✅ **Importado y analizado** el 29/10/2026 (lectura completa de ambos archivos). Ver [Informe de importación](#informe-de-importación). |
| Versión leída | `FINORA.dc.html` etag `1793323287746030` (314 412 bytes, 1 466 líneas) · `support.js` etag `1793287293161578` (69 150 bytes, 1 915 líneas) |

## Cómo habilitar la importación

1. Registrar el MCP:
   ```bash
   claude mcp add --transport http claude_design https://api.anthropic.com/v1/design/mcp
   ```
2. Autenticarse con `/design-login`.
3. Reiniciar la sesión de Claude Code para que las herramientas del MCP estén disponibles.

## Proceso de importación (una vez habilitado)

1. Leer completos `FINORA.dc.html` y `support.js`.
2. Elaborar un informe de importación: archivos, páginas, componentes, tokens de diseño, tipografía, assets, dependencias (y cuáles faltan) e interacciones.
3. Reportar los conflictos con la identidad visual (`CLAUDE.md` §10) sin corregirlos en silencio.
4. Documentar los tokens de diseño en esta guía.
5. Proponer la integración en `finora-web` (componentes reutilizables vs. de página) dentro del `plan.md` de cada feature afectada.

## Reglas

- El diseño se **adapta** a la arquitectura de `finora-web`; no se copia el prototipo como código independiente.
- Se preservan layout, tipografía (Inter), espaciado, colores, interacciones y comportamiento responsive.
- No se sustituye por plantillas genéricas ni se introducen estilos ajenos.

## Informe de importación

> Este informe describe el prototipo; **no aprueba requisitos**. Todo comportamiento funcional que aparece en el diseño debe confirmarse en el `spec.md` de su feature antes de implementarse.

### 1. Archivos

| Archivo | Naturaleza | Uso en `finora-web` |
|---|---|---|
| `FINORA.dc.html` | Prototipo de una sola página: plantilla con `<sc-if>`/`<sc-for>` y `{{ }}` (157 condicionales, 99 bucles), estilos en línea, un bloque `<style>` con tokens y un `<script data-dc-script>` (`class Component extends DCLogic`) con datos de demostración y lógica de vistas. | Referencia visual y de interacción. Se adapta, no se copia. |
| `support.js` | Runtime de Claude Design generado (`dc-runtime`), "do not edit". Carga React 18.3.1, ReactDOM 18.3.1 y Babel standalone 7.29.0 desde unpkg y renderiza la plantilla. | **No se usa.** Es infraestructura del prototipo, no del producto. |

### 2. Pantallas (15 rutas, todas diseñadas)

| Grupo de navegación | Ruta del prototipo | Título | Feature en `specs/features/` |
|---|---|---|---|
| General | `home` | Inicio ("Hola, {nombre}") | `dashboard` |
| General | `movements` | Movimientos | `transactions` |
| General | `accounts` | Cuentas y tarjetas | `accounts` |
| General | `budgets` | Presupuestos | `budgets` |
| General | `goals` | Ahorro y metas | `savings-goals` |
| General | `debts` | Deudas y créditos | `debts` |
| Planificación | `calendar` | Calendario | `calendar` |
| Planificación | `recurring` | Pagos recurrentes | `recurring-payments` |
| Planificación | `predictions` | Predicciones | `predictions-alerts` |
| Planificación | `analytics` | Analítica e informes | `analytics` |
| Herramientas | `assistant` | Asistente financiero | `ai-assistant` |
| Herramientas | `capture` | Centro de captura | `capture-center` |
| Herramientas | `alerts` | Alertas | `predictions-alerts` |
| Herramientas | `documents` | Documentos | `documents` |
| (pie de menú) | `settings` | Configuración (Perfil, Tema, Idioma y región, Notificaciones, Privacidad, Seguridad, Integraciones, Preferencias de captura, Automatizaciones de categoría) | `settings` |

No hay pantallas de inicio de sesión ni registro: `authentication` **no tiene diseño**; sus pantallas se derivan de los tokens existentes (CD-04, resuelto).

Contenido destacado por pantalla:

- **Inicio**: resumen del periodo, evolución ingresos/gastos (líneas + tabla "Datos mensuales"), donut de gastos por categoría, próximos pagos, presupuestos del mes, metas, alertas relevantes, sugerencias.
- **Movimientos**: filtros (tipo, periodo, búsqueda), tabla con columnas adaptadas al ancho, estados `cargando` / `error` / `vacío` / `sin resultados`.
- **Cuentas**: tarjetas de crédito (deuda, cupo, % de uso), "Cuentas y efectivo", "Ahorro e inversión", alta de cuenta.
- **Presupuestos**: selector de mes, barras de progreso por categoría (`role="progressbar"`), historial de periodos, gestión de categorías y subcategorías.
- **Deudas**: detalle, proyección de amortización, próximas 6 cuotas estimadas, historial de pagos.
- **Calendario**: vista mensual (lunes primero), detalle del día, próximos 14 días, marcas de vencidos.
- **Predicciones**: proyección del saldo disponible con banda de incertidumbre, comparador de escenarios de compra (contado vs. cuotas con tarjeta), metas, simulación de deuda, posibles gastos de 30 días.
- **Centro de captura**: métodos manual, texto natural, voz (Web Speech API, `es-CO`), CSV, recibos (OCR), extractos PDF/Excel, sincronización bancaria; bandeja de revisión.
- **Componentes transversales**: modal de movimiento (`role="dialog"`, `aria-modal`), diálogo de confirmación, panel de alertas en el encabezado, menú de perfil, toast, hoja móvil "Todas las secciones", enlace "Saltar al contenido".

### 3. Layout y responsive

| Rango | Nombre | Navegación | Padding del contenido |
|---|---|---|---|
| `< 768px` | Móvil | Barra inferior + hoja "Todas las secciones" | `20px 16px 112px` |
| `768–1199px` | Tablet | Barra lateral contraída (72px, solo íconos) | `24px 24px 48px` |
| `≥ 1200px` | Escritorio | Barra lateral expandida (252px), contraíble a 72px | `28px 32px 56px` |

Encabezado fijo con buscador, botón de alertas (contador) y perfil. Tablas anchas con desplazamiento horizontal (`min-width` 960px escritorio / 640px resto).

### 4. Tipografía e iconografía

- **Inter** (Google Fonts) pesos 400, 500, 600, 650, 700. Base `400 15px/1.5`; `font-variant-numeric: tabular-nums` en todo el documento.
- Escala observada: h1 de página, h2 de tarjeta `600 16px/1.35`, etiquetas `500 13px/1.3`, sobretítulos `600 11–12px` en mayúsculas con `letter-spacing .06–.08em`, logotipo `700 16px` con `letter-spacing .16em`.
- Íconos: **Material Symbols Rounded** (fuente de íconos de Google, `opsz 20–24`, `wght 300–500`).
- Logotipo: SVG en línea (cuadro violeta redondeado con órbita y planeta).

### 5. Formatos (es-CO)

- Moneda: `Intl.NumberFormat('es-CO', {style:'currency', currency:'COP', maximumFractionDigits:0})`.
- Formato compacto: `$ 1,2 M`, `$ 350 mil`. Negativos con el signo `−` (U+2212).
- Porcentajes: `"12 %"` (con espacio). Tasas: `"19,8 % EA"` y equivalente mensual `"≈ 1,5 % MV"`.
- Fechas `DD/MM/AAAA`; meses en minúscula (`octubre de 2026`); semana inicia el lunes.

### 6. Interacciones y movimiento

- Animaciones: `fn-in` (entrada con desplazamiento de 6px), `fn-pop` (modales), `fn-fade`, `fn-shimmer` (esqueletos de carga), `fn-dot` (indicador "escribiendo" del asistente), `fn-grow` (barras), `fn-pulse` (grabación de voz). Curva `--ease: cubic-bezier(.2,.7,.2,1)`, transiciones de 160–200ms.
- `prefers-reduced-motion: reduce` desactiva animaciones.
- Tooltips de gráficas al pasar el cursor; cada gráfica tiene `aria-label` y alternativa "Ver datos como tabla".
- Tema `oscuro` / `claro` / `sistema` (sigue `prefers-color-scheme`).
- Datos de demostración persistidos en `localStorage` (`finora-demo-v1`) y píldora "Datos de demostración".

### 7. Dependencias

| Dependencia | En el prototipo | Necesaria en `finora-web` |
|---|---|---|
| Inter | Google Fonts | Sí (definir si se sirve desde Google Fonts o autoalojada). |
| Material Symbols Rounded | Google Fonts | Sí, o un set de íconos equivalente (decisión pendiente). |
| React 18 + Babel standalone | Vía `support.js` desde unpkg | No: el framework se decide en DP-ARQ-01. |
| Web Speech API | API del navegador | Solo si se aprueba la captura por voz. |
| Assets externos (imágenes, ilustraciones) | **Ninguno**: todo es SVG en línea | — |

## Tokens de diseño

Extraídos literalmente de `FINORA.dc.html` (`:root` = tema oscuro por defecto, `[data-theme="claro"]` = tema claro).

### Superficies y fondos

| Token | Oscuro | Claro |
|---|---|---|
| `--bg` | `#090D1B` | `#F5F3FB` |
| `--surface` | `#141A30` | `#FFFFFF` |
| `--elev` | `#1B223B` | `#EEEBF8` |
| `--input-bg` | `#0F1428` | `#FFFFFF` |
| `--side-bg` | `#0C1122` | `#FBFAFE` |
| `--header-bg` | `rgba(9,13,27,.86)` | `rgba(245,243,251,.88)` |
| `--hero-bg` | `linear-gradient(135deg,#1F2648 0%,#141A30 64%)` | `linear-gradient(135deg,#ECE6FF 0%,#FFFFFF 66%)` |
| `--card-bg` | `linear-gradient(140deg,#2B2558 0%,#1B223B 58%,#141A30 100%)` | `linear-gradient(140deg,#3A2F7A 0%,#262E52 60%,#1B223B 100%)` |
| `--ambient` | `radial-gradient(900px 520px at 88% -14%,rgba(155,131,255,.10),transparent 62%), radial-gradient(700px 480px at -10% 40%,rgba(141,175,255,.05),transparent 60%)` | `radial-gradient(900px 520px at 88% -14%,rgba(106,80,216,.08),transparent 62%)` |
| `--overlay` | `rgba(4,6,14,.68)` | `rgba(20,24,46,.45)` |
| `--shadow` | `0 12px 32px rgba(2,4,12,.45)` | `0 12px 32px rgba(40,36,90,.14)` |

### Marca y texto

| Token | Oscuro | Claro |
|---|---|---|
| `--violet` | `#9B83FF` | `#6A50D8` |
| `--violet-hover` | `#B3A3FF` | `#5A41C6` |
| `--violet-pressed` | `#7861D8` | `#4B34AD` |
| `--on-violet` | `#090D1B` | `#FFFFFF` |
| `--on-neg` | `#1B0A10` | `#FFFFFF` |
| `--lav` | `#C5B7FF` | `#5B45C2` |
| `--lav-soft` | `rgba(155,131,255,.15)` | `rgba(106,80,216,.10)` |
| `--text` | `#F1EEFF` | `#14182E` |
| `--text2` | `#B3BBD0` | `#434A66` |
| `--muted` | `#8D96AE` | `#5C6482` |
| `--border` | `#39415E` | `#DCD8EB` |
| `--border-soft` | `rgba(57,65,94,.55)` | `#E8E5F2` |
| `--border-input` | `#646E8E` | `#8A90AA` |
| `--grid` | `rgba(141,150,174,.16)` | `rgba(92,100,130,.16)` |

### Estados semánticos

| Token | Oscuro | Claro | Uso |
|---|---|---|---|
| `--pos` / `--pos-soft` | `#48D6B0` / `rgba(72,214,176,.12)` | `#0B7F61` / `rgba(11,127,97,.10)` | Ingresos, éxito |
| `--neg` / `--neg-soft` | `#FF8296` / `rgba(255,130,150,.12)` | `#C2304B` / `rgba(194,48,75,.09)` | Gastos, error |
| `--warn` / `--warn-soft` | `#FFD18B` / `rgba(255,209,139,.12)` | `#8A5A00` / `rgba(138,90,0,.10)` | Advertencias, insignias |
| `--info` / `--info-soft` | `#8DAFFF` / `rgba(141,175,255,.12)` | `#2F5BC4` / `rgba(47,91,196,.09)` | Información |

### Colores de categoría

| Token | Oscuro | Claro |
|---|---|---|
| `--c-vivienda` | `#9B83FF` | `#6A50D8` |
| `--c-mercado` | `#8DAFFF` | `#3D68C9` |
| `--c-transporte` | `#66C6E0` | `#1E7F9A` |
| `--c-restaurantes` | `#E79AD0` | `#B0458F` |
| `--c-servicios` | `#D9B97E` | `#8C6A1F` |
| `--c-salud` | `#A6D47E` | `#4F7F23` |
| `--c-entretenimiento` | `#F2B6A0` | `#B0573A` |
| `--c-suscripciones` | `#C9A0F5` | `#8649C9` |
| `--c-deudas` | `#A8B3CF` | `#5D6787` |
| `--c-salario` | `#48D6B0` | `#0B7F61` |
| `--c-freelance` | `#7FE3C8` | `#12977A` |
| `--c-otros_ing` | `#9FE0D0` | `#2E8F7A` |
| `--c-sin` | `#8D96AE` | `#5C6482` |

### Otros

| Token | Valor |
|---|---|
| `--font` | `'Inter', system-ui, -apple-system, 'Segoe UI', sans-serif` |
| `--ease` | `cubic-bezier(.2,.7,.2,1)` |
| Foco | `outline: 2px solid var(--lav); outline-offset: 2px` |
| Radios observados | 9–10px (botones de navegación, logotipo), 12px (controles, grupos segmentados), 14–16px (tarjetas), 99px (píldoras) |
| Altura mínima táctil | 40px (navegación), 44px (botones principales) |

## Conflictos y observaciones

Se reportan sin corregir (`CLAUDE.md` §11). Requieren decisión.

| ID | Tipo | Observación | Pregunta / propuesta |
|---|---|---|---|
| CD-01 | Identidad visual | El diseño incluye **tema claro** y opción "sistema". `CLAUDE.md` §10 solo describe el fondo azul marino oscuro. | ¿Se aprueba el tema claro como parte de la identidad (con el oscuro como predeterminado)? |
| CD-02 | Accesibilidad (WCAG 2.2 AA) | Tema claro: `--pos` sobre `--elev` = **4,23:1** (< 4,5:1 para texto normal). | Oscurecer `--pos` en claro (p. ej. hacia `#0A7358`) o no usarlo sobre `--elev`. |
| CD-03 | Accesibilidad (1.4.11) | Tema claro: `--border-input` sobre `--bg` = **2,87:1** y sobre `--elev` = **2,69:1** (< 3:1 para bordes de controles). Tema oscuro cumple (≥ 3,11:1). | Oscurecer `--border-input` en claro. |
| CD-04 | Alcance | No hay diseño de inicio de sesión, registro ni recuperación de cuenta. | **Resuelto (29/10/2026):** se derivan de los tokens, colores, tipografía y estilos de componentes existentes, y se revisan después de implementarlas (ver `authentication/plan.md`). |
| CD-05 | Reglas financieras | El prototipo calcula montos con `number` de JavaScript y `Math.round` (punto flotante), y estima cuotas con conversión EA→MV (`(1+EA)^(1/12)−1`). | Es solo presentación: en el producto los cálculos van al backend con precisión segura (DP-ARQ-09). La fórmula EA→MV debe confirmarse en `debts/spec.md`. |
| CD-06 | Reglas financieras (conforme) | `stats()` solo suma `type === 'income'` / `'expense'`; los movimientos `transfer` no cuentan como ingreso ni gasto. | Coincide con `CLAUDE.md` §9. Se trasladará como criterio de aceptación a `transactions`. |
| CD-07 | Arquitectura | Persistencia en `localStorage` y "asistente" basado en reglas locales con expresiones regulares (no IA). | Son simulaciones del prototipo; no se trasladan. El asistente real depende de DP-ARQ-07. |
| CD-08 | Dependencias | Íconos con la fuente **Material Symbols Rounded** (Google). | ¿Se mantiene, o se usa un set de íconos SVG equivalente? |
| CD-09 | Alcance funcional | El diseño muestra funcionalidades que aún no tienen requisitos aprobados: voz, OCR de recibos, extractos PDF/Excel, sincronización bancaria, simulación de escenarios de compra, automatizaciones de categoría. El propio diseño las marca como "Requiere backend" / "Requiere configuración" / "No conectado". | Definir en cada `spec.md` qué entra en la primera versión. |
| CD-10 | Datos | Categorías predefinidas (vivienda, mercado, transporte, restaurantes, servicios, salud, entretenimiento, suscripciones, deudas, salario, freelance, otros ingresos, sin categoría) y tipos de cuenta (`bank`, `cash`, `credit`, `wallet`, `savings`, `investment`). | Confirmar como catálogo inicial en `transactions` / `accounts` y en `database.md`. |

## Propuesta de integración en `finora-web`

Se concreta en el `plan.md` de cada feature una vez decidido DP-ARQ-01. Propuesta general:

- **Tokens** (DP-ARQ-11): un único archivo de variables CSS con los dos temas, tal como en el prototipo; el resto de estilos consumen solo variables.
- **Componentes reutilizables**: `AppShell` (barra lateral, encabezado, barra inferior móvil), `NavItem`, `Card`, `PageHeader`, `SegmentedControl`, `Button`, `IconButton`, `TextField`/`Select`, `Modal`/`ConfirmDialog`, `Toast`, `Badge`/`Pill`, `ProgressBar`, `EmptyState`/`ErrorState`/`Skeleton`, `DataTable`, `Money` (formato COP), `DateText` (DD/MM/AAAA).
- **Gráficas reutilizables**: `LineAreaChart` (con banda y tooltip), `BarChart`, `DonutChart`, cada una con su alternativa en tabla.
- **Componentes de página**: uno por ruta de la tabla de pantallas, dentro de su feature.
