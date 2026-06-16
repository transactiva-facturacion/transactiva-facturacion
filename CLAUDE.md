# Transactiva 2000 S.L. — Contexto del Proyecto

> Este fichero es el contexto permanente del proyecto. Claude Code lo lee automáticamente
> al arrancar en esta carpeta. Contiene todo lo necesario para continuar el trabajo
> sin explicaciones previas.

---

## Negocio

Comercializadora eléctrica que compra energía de red (Naturgy, Iberdrola, Repsol) y
refactura a **185 clientes** propios con margen. El margen se genera entre el precio
de compra (PC) y el precio de venta (PV).

---

## Repositorio GitHub

**URL**: `github.com/transactiva-facturacion/transactiva-facturacion` (privado)

**Ficheros en GitHub**:
| Fichero | Descripción |
|---------|-------------|
| `CLAUDE.md` | Este fichero — contexto permanente |
| `Facturas_Venta_SinRegistrar_20260601.xlsx` | Facturas pendientes de registro (exportación BC) |
| `Facturas_Venta_Registradas_Historico.xlsx` | Histórico ventas BC desde 01/01/2025 (15,4 MB) |
| `Precios_PC_PV_por_CUPS.xlsx` | TABLA_MAESTRA — PC y PV por producto y CUPS |
| `Margenes_Cartera_185CUPS.xlsx` | 185 CUPS con márgenes y segmentación |
| `Precios_Venta_Plantilla_BC.xlsx` | Máscara de precios para importar en BC |
| `validacion/2026-06/` | Ficheros de validación junio 2026 |

---

## Estructura de Carpetas (OneDrive)

```
OneDrive-TRANSACTIVA2000,SL/TRANSACTIVA2000/
├── CLAUDE.md
├── CONTABILIDAD ANALÍTICA_/           ← facturas de COMPRA por CUPS
│   ├── ES0021000001197056KH/
│   │   ├── FE26390018030427.pdf
│   │   └── ...
│   └── ...
├── Facturas_Venta_SinRegistrar_YYYYMMDD.xlsx
├── Facturas_Venta_Registradas_Historico.xlsx
├── Precios_PC_PV_por_CUPS.xlsx
├── Margenes_Cartera_185CUPS.xlsx
└── Precios_Venta_Plantilla_BC.xlsx
```

### Cómo acceder a facturas de compra desde Claude Code
```python
import os, glob
BASE = os.path.expanduser(
    "~/Library/CloudStorage/OneDrive-TRANSACTIVA2000,SL/TRANSACTIVA2000"
)
FACTURAS_COMPRA = os.path.join(BASE, "CONTABILIDAD ANALÍTICA_")
pdfs = glob.glob(f"{FACTURAS_COMPRA}/{cups}/*.pdf")
```

---

## Tarifas

| Tarifa | Tipo | Períodos energía | Períodos potencia |
|--------|------|-----------------|------------------|
| 2.0TD | Residencial/pequeño comercio | P1, P2, P3 | P1, P2 |
| 3.0TD | Comercial | P1–P6 | P1–P6 |
| 6.1TD | Industrial | P1–P6 | P1–P6 |

---

## Productos Business Central (BC)

| Código | Concepto | Unidad | Observación |
|--------|----------|--------|-------------|
| 1001 | Excedentes solares | €/kWh | Solo CUPS con solar |
| 1003 | Alquiler contador | €/mes | Fijo |
| 1004 | P1 Potencia | €/kW×días | |
| 1005 | P2 Potencia | €/kW×días | |
| 1006 | P3 Potencia | €/kW×días | |
| 1007 | P4 Potencia | €/kW×días | |
| 1008 | P5 Potencia | €/kW×días | |
| 1009 | P6 Potencia | €/kW×días | |
| 1012 | Tasa eléctrica | € | Base × 5,11269% |
| 1041 | Bono social | €/día | 0,019121 €/día × días |
| 1100 | P1 Consumo energía | €/kWh | |
| 1101 | P2 Consumo energía | €/kWh | |
| 1102 | P3 Consumo energía | €/kWh | |
| 1103 | P4 Consumo energía | €/kWh | |
| 1104 | P5 Consumo energía | €/kWh | |
| 1105 | P6 Consumo energía | €/kWh | |

---

## Normativa Vigente

| Norma | Contenido | Fecha |
|-------|-----------|-------|
| RD 7/2026 | Tasa eléctrica 0,5% para Iberdrola y Repsol | 20/03/2026 |
| Orden TED/2026 | Bono social: 0,019121 €/día | 2026 |
| RD 244/2019 | Excedentes solares: cap PC ≤ 0,06 €/kWh | 2019 |

---

## Fórmulas Fiscales

### Tasa eléctrica
```
# VENTA (siempre 5,11% independiente del proveedor)
Tasa_PV = (Σ importe_potencia_PV + Σ importe_energia_PV) × 5,1126963%

# COMPRA
Tasa_PC = base_PC × 5,1126963%   # Naturgy y otros
Tasa_PC = base_PC × 0,5%         # Iberdrola y Repsol (RD 7/2026)
```

### Bono social
```
Importe = 0,019121 €/día × días_facturados
```

### IVA
```
Base 21%: potencia + energía + bono + alquiler
Base  0%: tasa eléctrica
Total c/IVA = base_21% × 1,21 + base_0%
```

---

## Tabla Discriminación Horaria 3.0TD

| Hora | Ene/Feb/Dic | Mar/Nov | Abr/May/Oct | Jun | Jul | Ago/Sep | Sáb/Dom/Fest |
|------|-------------|---------|-------------|-----|-----|---------|--------------|
| 0-8h | P6 | P6 | P6 | P6 | P6 | P6 | P6 |
| 8-9h | P2 | P3 | P5 | P4 | P2 | P4 | P6 |
| 9-14h | P1 | P2 | P4 | P3 | P1 | P3 | P6 |
| 14-18h | P2 | P3 | P5 | P4 | P2 | P4 | P6 |
| 18-22h | P1 | P2 | P4 | P3 | P1 | P3 | P6 |
| 22-24h | P2 | P3 | P5 | P4 | P2 | P4 | P6 |

---

## Sistema de Validación de Facturas de Venta (3.0TD)

### Objetivo
Validar las facturas pendientes de registro en BC antes de registrarlas.
Resultado: semáforo de confianza ALTA / MEDIA / BAJA por factura.

### Vista de revisión
Para cada factura 3.0TD sin registrar se presentan **3 columnas en paralelo**:
1. **PENDIENTE** (azul) — la factura sin registrar
2. **MES ANTERIOR** (marrón) — factura del período inmediatamente anterior
3. **MISMO PERÍODO AÑO ANTERIOR** (verde) — misma factura del año anterior

Reglas de referencia:
- El **período de referencia** es el **período facturado** (no la fecha de registro BC)
- Histórico válido **solo desde 01/01/2025**
- Período anterior = último período registrado antes del actual
- Mismo período año anterior = mismo mes, año -1

### Los 5 Checks

**C1 — Completitud de líneas (peso 25%)**
- Obligatorio siempre: potencia P1-P6 + al menos 1 período energía + alquiler + tasa + bono
- No es obligatorio tener los 6 períodos de energía (solo los que BC genere)
- 🔴 VETO automático si falta alguna línea obligatoria

**C2 — Energía a cero (peso 25%)**
- Cualquier línea de energía **presente** con qty=0 es siempre error
- 0 kWh nunca es estacionalidad — es error de lectura o importación SIPS
- 🔴 VETO automático

**C3a — Variación vs mismo período año anterior (peso 20%)**
- Compara kWh totales vs misma factura año anterior
- ✅ <25% | 🟡 25-50% | 🔴 >50% | ⚪ sin dato

**C3b — Variación vs período anterior (peso 10%)**
- Compara kWh totales vs período inmediatamente anterior
- ✅ <25% | 🟡 25-50% | 🔴 >50% | ⚪ sin dato

**C4 — Margen sobre compra + LP (peso 20%)**
- Margen = (Σ PV×qty − Σ PC×qty) / Σ PV×qty
- PV = histórico ventas BC (precio negociado real por CUPS)
- PC = Precios_PC_PV_por_CUPS.xlsx (TABLA_MAESTRA)
- Objetivo 3.0TD: ≥ 30%
- ✅ ≥30% | 🟡 20-30% | 🔴 <20%
- Si margen < 30% → motor LP redistribuye kWh para alcanzarlo
- Si LP no alcanza → acepta máximo posible + flag renovación

### Puntuación global
```
Score = C1×25 + C2×25 + C3a×20 + C3b×10 + C4×20   (0-100)

ALTA  ≥ 80 → registrar en BC directamente
MEDIA 50-79 → revisar en visor antes de registrar
BAJA  < 50 → NO registrar sin corrección

VETO: C1 o C2 en 🔴 → BAJA automático independientemente del score
```

### Fuente de precios PV
**CRÍTICO**: El PV no viene del tarifario BC sino del **último histórico de facturas
de venta registradas en BC** (`Facturas_Venta_Registradas_Historico.xlsx`).
Para cada CUPS se toma el PV más reciente registrado por producto.
Muchos precios son negociados individualmente.

### Cómo leer el histórico de ventas BC
```python
# Fichero: Facturas_Venta_Registradas_Historico.xlsx
# Estructura: fila 0=paquete, fila 1=vacía, fila 2=cabeceras, filas 3+=datos
# Columnas clave:
#   'Nº documento'              → nº factura
#   'Venta a-Nº cliente'        → código cliente (C02330, etc.)
#   'Nº'                        → código producto (1004, 1100, etc.)
#   'Cantidad'                  → qty (kWh o kW)
#   'Precio venta'              → PV unitario
#   'Importe'                   → importe línea (usar este, no recalcular)
#   '% IVA'                     → tipo IVA (0 o 21)
#   'Cód. dim. acceso dir. 1'   → CUPS
#   'Descripción'               → contiene "Período de Lectura: DD/MM/AA a DD/MM/AA"
```

**IMPORTANTE**: Usar siempre el campo `Importe` de BC, NO recalcular con PV×qty.
BC aplica sus propios redondeos que no coinciden con el cálculo directo.

---

## Motor LP (Programación Lineal kWh 3.0TD)

```
MAX  Σₚ (PVₚ − PCₚ) · qₚ   [solo energía — potencia intocable]

Restricciones:
  R1. Total kWh ∈ [0.85, 1.15] × kWh_lectura_contador
  R2. Por período: qₚ ≤ total × θ(mes,p) × 1.4  (tabla horaria)
  R3. qₚ ≥ 0 para todo p
  R4. Margen resultante ≥ 30%
  R5. Si R4 infactible → maximizar margen sin R4, flag renovación

PVₚ = precio histórico BC | PCₚ = TABLA_MAESTRA
Solver: HiGHS via scipy.optimize.linprog
```

---

## Reglas del Motor de Facturación

| ID | Regla | Parámetro |
|----|-------|-----------|
| R01 | Margen mínimo global ≥ 20% | 20% |
| R02 | Margen objetivo 3.0TD ≥ 30% | 30% |
| R03 | Si margen < mínimo → LP redistribuir kWh | ±15% |
| R04 | Si LP no alcanza → ACEPTAR y flag. NO tocar potencia | — |
| R05 | Precio €/kWh energía NO se modifica | Fijo contrato |
| R06 | Distribución kWh 3.0TD: tabla horaria oficial | θ(m,p)×1.4 máx |
| R07 | Potencia: NUNCA bajar. No subir para tapar margen | KW intocable |
| R08 | Armonía presupuestaria: coherente con histórico | CV < 0.4 |
| R13 | Tasa PV = base × 5,11269% | Siempre |
| R14 | Tasa PC Iberdrola/Repsol = 0,5% | RD 7/2026 |
| R15 | Tasa PC Naturgy = 5,11269% | Regulatorio |
| R16 | Bono social = 0,019121 €/día × días | TED/2026 |
| R18 | Tasa y bono NUNCA se tocan para ajustar margen | Fijo |
| R19 | Facturar antes del día 5 de cada mes | Operativo |
| R20 | Sandbox BC siempre antes de producción | Obligatorio |

---

## Proceso de Revisión Mensual

### Flujo
```
Exportar BC (facturas sin registrar)
    ↓
Identificar 3.0TD/6.1TD
    ↓
Para cada factura → buscar:
  - Factura período anterior (mismo CUPS)
  - Factura mismo período año anterior (mismo CUPS, mismo mes, año-1)
    ↓
Ejecutar motor C1→C2→C3a→C3b→C4+LP
    ↓
Excel con vista comparativa 3 columnas + semáforo
    ↓
ALTA → BC Sandbox → Producción
MEDIA → Visor revisión manual → corrección → Sandbox → Producción
BAJA → Corrección obligatoria → re-validar
```

### Señales de error de lectura (NO registrar sin corrección)
- kWh de energía solo en P6 y cero en el resto (3.0TD)
- kWh total < 15% del histórico del mismo período
- Variación > 80% vs período anterior sin justificación
- Línea de energía presente con qty = 0

---

## Segmentación de Cartera (185 CUPS)

| Tier | N | Mg €/año | kWh/mes | Acción |
|------|---|----------|---------|--------|
| A — Cuidar (>800 kWh, margen OK) | 8 | 7.856 € | 1.970 | Contrato 24m |
| A — Reactivar (>800 kWh, margen peligroso) | 1 | 12 € | 2.134 | Subir PV urgente |
| B — Optimizar (LP rentable) | 8 | 3.879 € | 420 | LP redistribución |
| B — Mantener (margen estable) | 26 | 5.541 € | 325 | Precio fijo 24m |
| C — Fidelizar (77% cartera) | 142 | 8.948 € | 66 | Pack precio fijo 12m |

**Margen total actual: 26.237 €/año**
**Potencial LP: 30.094 € (+3.858 €, +26%)**

### Casos críticos
- **C03270 LULU BEACH CLUB** (3.0TD, 2.134 kWh): margen 7,2% → subir PV renovación
- **C02700 TALLERES MARTINEZ** (3.0TD): margen 3,2% → crítico
- **C03020 MONTGO SEGURIDAD** (3.0TD): margen 4,8%
- **ES0021000001209594ZK MEDITERRÁNEO 96**: Plan Variable → sin solución precio fijo

---

## Estado del Proyecto (junio 2026)

### Completado ✅
- Motor validación C1-C4 con LP construido y ejecutado sobre 27 facturas 3.0TD
- Vista comparativa 3 columnas (pendiente + mes anterior + mismo período año anterior)
- Semáforo ALTA/MEDIA/BAJA con score ponderado
- Todos los períodos P1-P6 mostrados siempre (gris si no aplica)
- Importe de BC directo (no recalculado)
- Colores confianza: verde=ALTA, amarillo=MEDIA, rojo=BAJA
- PV desde histórico BC real (no tarifario)
- Repositorio GitHub operativo con ficheros versionados

### Resultados validación junio 2026 (3.0TD)
- **ALTA: 9 facturas** → registrar directamente
- **MEDIA: 18 facturas** → revisión manual
- **BAJA: 0 facturas**

### Pendientes ⚠️
- Extender motor a clientes 2.0TD
- Automatizar pipeline completo
- C03270 LULU BEACH, C02700 TALLERES MARTINEZ: subir PV en renovación
- Control desviación mensual automatizado
- Integración SIPS lecturas automáticas
- Subir `Validacion_Facturas_3TD_202606.xlsx` a GitHub
