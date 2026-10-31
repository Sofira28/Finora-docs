# Proyecto FINORA

| Campo | Valor |
|---|---|
| Estado | `Draft` |
| Última actualización | 30/10/2026 |

## 1. Visión

**FINORA** es una plataforma premium de gestión de finanzas personales, enfocada inicialmente en usuarios de Colombia.

**Eslogan:** *"Tu dinero, más claro."*

Su propósito es darle a cada persona una visión clara y confiable de sus finanzas: lo que tiene, lo que debe, en qué gasta y hacia dónde va.

## 2. Público objetivo

- ✅ Personas en Colombia que gestionan sus finanzas personales.
- ❓ Segmentos específicos (edad, nivel de ingresos, independientes vs. asalariados, familias): **pendiente de definir**.

## 3. Localización

| Aspecto | Valor | Certeza |
|---|---|---|
| Idioma de la interfaz | Español de Colombia (`es-CO`) | ✅ |
| Moneda inicial | Peso colombiano (COP) | ✅ |
| Formato de fecha | DD/MM/AAAA | ✅ |
| Formato numérico | Colombiano (separador de miles `.`, decimal `,`; p. ej. `$ 1.250.000`) | ✅ formato colombiano · 🟡 detalle exacto de símbolo, espaciado y decimales visibles para COP |
| Zona horaria | ❓ Pendiente (¿`America/Bogota` por defecto?) | ❓ |
| Multimoneda / otros idiomas | Fuera del alcance inicial | 🟡 |

## 4. Módulos planificados

| Módulo | Feature (`specs/features/`) |
|---|---|
| Autenticación y perfiles de usuario | `authentication` |
| Dashboard | `dashboard` |
| Transacciones | `transactions` |
| Cuentas y tarjetas | `accounts` |
| Presupuestos y categorías | `budgets` |
| Metas de ahorro | `savings-goals` |
| Deudas y préstamos | `debts` |
| Calendario | `calendar` |
| Analítica y reportes | `analytics` |
| Asistente financiero con IA | `ai-assistant` |
| Centro de captura | `capture-center` |
| Pagos recurrentes | `recurring-payments` |
| Predicciones y alertas | `predictions-alerts` |
| Documentos | `documents` |
| Configuración | `settings` |

## 5. Reglas financieras transversales (confirmadas)

- Las transferencias entre cuentas propias del usuario **no** cuentan como ingreso ni gasto.
- **Saldo disponible** y **patrimonio neto** son conceptos distintos.
- Ingresos, gastos, ahorro y transferencias se clasifican correctamente.
- Los cálculos financieros usan precisión monetaria segura.
- Los datos financieros se aíslan por usuario.
- La validación en el backend es obligatoria.

## 6. Identidad visual

Ver `CLAUDE.md` §10–11. La fuente visual de verdad es el proyecto de Claude Design (pendiente de importación).

## 7. Preguntas abiertas

| ID | Pregunta |
|---|---|
| PA-PRJ-01 | ¿Modelo de negocio (gratuito, freemium, suscripción)? |
| PA-PRJ-02 | ¿Plataformas objetivo: solo web responsive, o también apps nativas o PWA? |
| PA-PRJ-03 | ¿Integración con bancos colombianos (open finance) o solo registro manual e importación? |
| PA-PRJ-04 | ¿Alcance del MVP: qué módulos entran en la primera versión? |
| PA-PRJ-05 | ~~¿Definición precisa de "saldo disponible" y "patrimonio neto"?~~ Resuelta el 30/10/2026 en [`features/accounts/spec.md`](features/accounts/spec.md) RN-ACC-03 a 05. |
