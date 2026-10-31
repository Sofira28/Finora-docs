# ADR-0005: Modelo de movimientos y cálculo de saldos

| Campo | Valor |
|---|---|
| Estado | Aceptada (30/10/2026) |
| Fecha | 30/10/2026 |
| Feature | [`transactions`](../../specs/features/transactions/plan.md) |
| Resuelve | DP-DB-03 ([`specs/database.md`](../../specs/database.md)) |

## Contexto

`transactions` registra gastos, ingresos, transferencias entre cuentas propias y ajustes de saldo. Las transferencias no son ingreso ni gasto, pero mueven saldo entre dos cuentas, incluidas las tarjetas de crédito, donde el saldo es la deuda. El saldo actual de cada cuenta debe reflejar todos sus movimientos, incluso al editar o eliminar uno antiguo.

Hay que decidir dos cosas:

1. Cómo se guarda una transferencia (DP-DB-03).
2. Si el saldo de cada cuenta se guarda o se calcula.

## Decisión

1. **Un movimiento es una fila**, también las transferencias, con `account_id` (origen) y `destination_account_id` (destino). El efecto en cada cuenta se deduce del tipo de movimiento, del papel de la cuenta (origen o destino) y de si es tarjeta.
2. **Los saldos se calculan al leer**: saldo inicial + suma de los efectos de los movimientos de la cuenta, con una consulta agregada por usuario. No hay columnas de saldo.

## Alternativas consideradas

- **Dos movimientos enlazados por transferencia** (salida y entrada):
  - facilita sumar por cuenta;
  - pero puede quedar a medias si falla una de las dos escrituras o si se edita solo una;
  - obliga a mantener el enlace y duplica la fecha, el monto y la nota;
  - complica clasificar el ahorro, que depende de las dos cuentas a la vez.
- **Saldo guardado en `accounts`**, actualizado en cada escritura:
  - lectura más barata;
  - pero cada creación, edición o eliminación debe actualizar uno o dos saldos con bloqueo;
  - un error deja saldos desincronizados sin forma simple de detectarlo;
  - con el volumen esperado (≤ 100 cuentas y unos 10.000 movimientos por usuario), calcular al leer es suficiente.

## Consecuencias

- **Positivas**:
  - la integridad de una transferencia es la de una fila;
  - los `CHECK` y las FK compuestas expresan las reglas en la base de datos;
  - editar o eliminar un movimiento antiguo no requiere recalcular nada guardado.
- **Negativas**:
  - cada lectura de `/accounts` agrega los movimientos del usuario;
  - la regla de efecto existe en Python (pruebas unitarias) y en SQL (`CASE`), y las pruebas de integración comprueban que coinciden.
- **Revisión**: si la prueba de rendimiento de RNF-TRX-06 no se cumple, se evaluará guardar saldos con un ADR nuevo.
