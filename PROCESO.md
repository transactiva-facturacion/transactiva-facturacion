# Validación y registro de facturas 3.0TD — Abril 2026

Documento de proceso de la validación de las **27 facturas de venta 3.0TD de abril 2026**
(`Validacion_Facturas_3TD_202604.xlsx`). Recoge fuentes, hallazgos, reglas de cálculo,
estructura del entregable y cómo regenerarlo. Sirve de referencia para repetir el ciclo
cada mes y para continuar el trabajo.

> Principio rector del proyecto: **no fabricar, no estimar**. Si un dato no está en la
> fuente, no se inventa. Todo PC (precio de compra) procede de facturas de compra reales.

---

## 1. Objetivo

Transactiva 2000 S.L. revende electricidad: compra a la comercializadora (Naturgy, Repsol,
Iberdrola) y refactura a ~185 puntos de suministro (CUPS) con margen objetivo ≥30 %.
El objetivo es **validar el margen real** de cada factura de venta sin registrar antes de
darla de alta en Business Central (BC), comparando precio de venta (PV) contra precio de
compra (PC) **real** leído de las facturas de compra.

Lote de este ciclo: **27 facturas 3.0TD de abril 2026** (18 Naturgy + 9 Repsol; una de ellas,
8547, resultó ser **6.1TD** colada en el lote).

---

## 2. Fuentes de datos

| Fuente | Uso |
|---|---|
| `Facturas_Venta_SinRegistrar_20260601.xlsx` | Líneas de venta sin registrar (BC). Fuente limpia. |
| `datos/hist_ref_invoices.json` | Histórico BC (mes anterior / año anterior) por factura. |
| `datos/validation_results_3td.json` | Las 27 con sus refs y checks. |
| OneDrive `TRANSACTIVA2000/CONTABILIDAD ANALÍTICA__/{NN}_{CUPS}/` | **Facturas de compra reales (PDF)** → PC real. |
| `HOJA_TTVA_LUZ_1.xlsx` (hoja "PAGOS vs COBROS") | Histórico mensual cobrado/pagado por CUPS (2023‑01 → 2026‑06). |
| `CLAUDE.md` | Constantes fiscales, códigos de producto BC, reglas de negocio. |

**Códigos de producto BC:** 1004–1009 = potencia P1–P6 · 1100–1105 = energía P1–P6 ·
1003 = alquiler equipos medida · 1041 = bono social · 1012 = tasa eléctrica ·
1001 = compensación excedentes.

El conector de Microsoft 365 **no puede leer `.xlsm`** (formato con macros: error
"MIME type not allowed"). Por eso HOJA_TTVA_LUZ hubo que aportarla como **`.xlsx`**.

---

## 3. Hallazgos clave sobre el PC real (leyendo facturas de compra)

- **El maestro de precios `Precios_PC_PV_por_CUPS.xlsx` NO es fiable.** Tenía valores
  fabricados (P1/P2/P3 inventados, varios P4/P5/P6 a cero). El PC real se tomó de las
  facturas de compra originales.
- **Naturgy "Plan Fijo Luz Best"**: precio fijo idéntico en todos los CUPS de ese plan:
  P1=0,183593 · P2=0,158014 · P3=0,134925 · P4=0,121236 · P5=0,114444 · P6=0,106808 €/kWh.
  Confirmado en PIZZERIA (8737), ORENCO (8544), FUNDACIÓN FW (8566), CINDY (8732).
  Potencia uniforme (€/kW·día): 0,057928 / 0,031190 / 0,014379 / 0,012748 / 0,008988 / 0,006052.
  Alquiler 0,197918 €/día, bono 0,019121 €/día.
- **Repsol NO siempre es plano.** TINTORERÍA (8649) y SANTIAGO (8731) tienen el **P6 al
  doble** del precio diurno (estructura real del contrato, no un error): TINTORERÍA
  P4=P5=0,10995 / P6=0,2199; SANTIAGO P4=P5=0,110688 / P6=0,221376. El bono Repsol varía
  (0,019121 ó 0,012742 €/día) y las tasas de potencia varían por contrato.
- **8547 es 6.1TD** (no 3.0TD): "Plan Fijo Luz", P4=0,097753 · P5=0,091419 · P6=0,084842;
  potencia 0,081083/0,042506/0,018635/0,014778/0,005822/0,002751; 40 kW; alquiler 0,394521.
- **Excedentes solares (1001):** ninguna de las 27 **ventas** los compensa, pero la
  **compra** de CINDY sí (−1.204 kWh × 0,06 €/kWh) y 8547 está acogido → posible importe no
  trasladado al cliente (revisar con contabilidad). El único histórico de venta con
  excedentes es la factura 3975 (año anterior de CASA LILI): −27,356 kWh × 0,15 €/kWh.

**Bug de parser (corregido):** el parser original dejaba de acumular líneas de energía al
encontrar la primera línea no‑energía, perdiendo periodos. Afectó a 21/27 facturas
(~55.544 kWh menos). Todos los outputs generados con esa versión se invalidaron y se
reconstruyeron. Confirmado que las 27 tienen exactamente 3 lecturas de energía.

---

## 4. Reglas de cálculo

- **Consolidación de periodos 3.0TD:** abril válido = **P4/P5/P6** (calendario horario
  peninsular; P1/P2/P3 a cero horas). El consumo bruto de BC (a veces etiquetado P1/P2/P6)
  se consolida a P4/P5/P6 por orden de PV (mayor→P4, medio→P5, menor→P6). No cambian
  cantidades ni importes, solo la etiqueta. No se usa "energía fuera de periodo".
- **Margen** = (BasePV − CostePC) / BasePV, **solo sobre energía + potencia** (alquiler,
  bono y tasa son pass‑through, margen 0).
  - Energía coste = Σ(PC_€kWh × Cant).
  - Potencia coste = Σ(tasa_€kW·día × kW × días). Para los CUPS leídos se usan las tasas
    reales; para el resto, ratio PC/PV del maestro verificado.
- **Tasa eléctrica (1012):** queda **fuera del IVA**. Total c/IVA = (Σ importes − tasa) × 1,21 + tasa.
- **Alquiler / bono / tasa:** PC = PV (pass‑through).
- **Veredicto:** ≥30 % APROBAR · 20–30 % REVISAR · <20 % RECHAZAR.

Ejemplo (8544 ORENCO, 30 kW, 30 días):
- Energía PV 1.151,35 / PC 969,19 · Potencia PV 173,05 / PC 118,16.
- Base 1.324,40 / Coste 1.087,35 → **margen 17,9 % → RECHAZAR**.

---

## 5. Estructura del entregable (`Validacion_Facturas_3TD_202604.xlsx`)

### Hoja **Detalle** (un bloque por factura)
Layout comparativo de 3 columnas:
- **PENDIENTE (sin registrar):** Cant · Días · **PC real** · PV · Importe.
- **MES ANTERIOR · Fact.{nº BC}:** Cant · PV · Importe (azul→naranja).
- **AÑO ANTERIOR · Fact.{nº BC}:** Cant · PV · Importe (verde).

Filas: potencia P1–P6, energía **P4/P5/P6** (sin etiqueta punta/llano/valle),
alquiler/bono/tasa (en las 3 columnas), compensación excedentes (donde la hay),
TOTAL c/IVA y MARGEN.

- **Importes y totales son fórmulas vivas de Excel** (`=REDONDEAR(Cant×PV;2)`,
  total `=REDONDEAR((SUMA−tasa)×1,21+tasa;2)`).
- **Colores:** mes anterior en naranja, año anterior en verde (mismo tono que su cabecera).
- **Carmesí (`DC143C`):** celda de lectura **ausente** → editable.

### Recuadro por factura (datos del **propio CUPS** desde HOJA_TTVA_LUZ)
- Cobrado total · Pagado total · DIF · **Margen histórico** (DIF/Cobrado).
- **Mejor mes** y **peor mes** (por importe cobrado) con su **margen** de ese mes.
- Línea de **verificación** (ver §8).
- **Mini gráfico**: importe mensual cobrado de los **últimos 24 meses** del CUPS.
  Los datos viven en la hoja oculta `_series` (1 fila por factura, 24 columnas de mes).

### Hoja **Resumen**
Una fila por factura con: Margen abril (vivo), Veredicto, **Margen histórico**,
**Cobrado histórico** y "Qué revisar". Semáforo por **formato condicional** (rojo <20 %,
ámbar 20–30 %, verde ≥30 %). El margen y el veredicto son **fórmulas que leen el margen del
Detalle**, así que se actualizan y recolorean solos al editar el objetivo.

---

## 6. Margen objetivo editable (todas las facturas)

En el Detalle, cada factura tiene una **celda amarilla "Margen objetivo"** (columna P, en la
fila de P4 Energía). Al escribir un %:
- las cantidades de energía **P4/P5/P6 suben en la misma proporción** para cumplir ese margen,
- importes, total y margen se recalculan por fórmula,
- el semáforo se recolorea (Detalle y Resumen).

Mecánica (coste fijo, se escala el ingreso): `k = (C/(1−objetivo) − ingreso_potencia) / ingreso_energía`,
`cant_nueva = cant_orig × k`. Si la celda está vacía, k=1 (cantidades originales).
Es una simulación de "cuánto habría que facturar para llegar a ese margen".

---

## 7. Verificación del fichero de no registradas (cada mes)

Indicador en el recuadro de cada factura:
- **Lecturas:** deben ser 3. Si faltan, las celdas van en **carmesí** y quedan editables.
- **Productos:** potencia P1–P6, energía, alquiler, bono, tasa. Avisa si falta alguno.
- **Periodos (calendario horario):** avisa (ámbar) si hay consumo en periodos inactivos de
  abril (P1/P2/P3) → se consolidan a P4/P5/P6. En abril lo presentan 4 facturas
  (8544, 8545, 8567, 8733) — es el caso conocido del motor LP.

---

## 8. Cruce con HOJA_TTVA_LUZ (hoja "PAGOS vs COBROS")

Estructura: bloque de 4 filas por CUPS → [Nº consec · Nº cont · **CUPS**] /
[dirección + **fechas** mensuales] / [**Pagado**] / [**Cobrado**]. Meses en columnas pares
(2023‑01 → 2026‑06, 42 meses). 221 CUPS.

Cruce por código CUPS: **27/27 coinciden**. Por CUPS se calcula: cobrado total, pagado
total, DIF, margen histórico = DIF/cobrado, mejor/peor mes y serie de 24 meses para el
gráfico. Nota: el **margen mensual es ruidoso** (pagos y cobros no caen en el mismo mes), por
eso "mejor/peor mes" se eligen por importe y se muestra el margen de ese mes.

---

## 9. Veredictos del lote

7 APROBAR (≥30 %) · 8 REVISAR (20–30 %) · 12 RECHAZAR (<20 %) — coherente con los críticos
conocidos (LULU 3,4 %, TALLERES 4,7 %, MONTGO 7,2 %, 2 HANDS 6,9 %).

---

## 10. Incidencias abiertas / pendientes

- **Registrar en BC** las APROBAR tras revisión.
- **Renegociar** las 12 RECHAZAR (margen real bajo).
- **Excedentes no compensados** al cliente (CINDY, 8547): revisar con contabilidad.
- **CASA LILI año anterior (3975):** PC de compra del excedente no archivado en OneDrive
  (abril 2025 inexistente) → queda sin dato.
- **Motor LP** bloqueado: el calendario predice cero horas en P1/P2/P3 en abril pero las
  facturas reales traen consumo ahí (se consolida).
- **Tasa eléctrica (1012):** Iberdrola/Repsol pueden usar 0,5 % por RD 7/2026; la línea no
  siempre reconcilia contra base×tipo (incidencia abierta).

---

## 11. Cómo regenerar

Desde `validacion_3td_202604/`:

1. `build_detalle.py` genera la hoja **Detalle** (lee los JSON de `datos_generados/`,
   desde el directorio raíz del repo con `datos/` relativo).
2. Script de **Resumen** (inline) re‑adjunta la hoja Resumen con fórmulas + formato condicional.
3. Insertor de **gráficos** referencia la hoja `_series` (24 meses).
4. Recalcular con `scripts/recalc.py` (LibreOffice) — los gráficos sobreviven al recálculo.
5. `present_files` del `.xlsx` final.

Orden importante: **build → Resumen → gráficos → recalc** (el recálculo al final conserva
los valores de las fórmulas y los gráficos).

### Datos generados (`datos_generados/`)
`inv27.json` (27 facturas) · `inv_raw.json` (líneas crudas BC) ·
`combined_pc.json` (PC combinado) · `pc_real_extraido.json` (PC leído de PDFs) ·
`margenes.json` · `margin_rows.json` · `ref_full.json` ·
`ttva_data.json` (serie mensual por CUPS) · `ttva_box.json` (datos del recuadro) ·
`chart_specs.json` (anclas de gráficos).
