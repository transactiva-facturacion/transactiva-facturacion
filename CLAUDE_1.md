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

## Estructura de Carpetas (OneDrive)

```
OneDrive-TRANSACTIVA2000,SL/TRANSACTIVA2000/
├── CLAUDE.md                          ← este fichero
├── CONTABILIDAD ANALÍTICA_/           ← repositorio facturas de COMPRA
│   ├── ES0021000001197056KH/          ← una carpeta por CUPS
│   │   ├── FE26390018030427.pdf       ← PDFs de facturas de compra
│   │   └── ...
│   ├── ES0021000001198420NW/
│   └── ...
├── Gene_rico_YYYYMMDD.xlsx            ← exportación BC facturas de VENTA registradas
├── Compra_vs_Venta_por_CUPS.xlsx      ← Excel 185 CUPS con precios PC/PV y márgenes
├── Mascara_BC_Transactiva.xlsx        ← máscara de precios para subir a BC
├── TABLA_MAESTRA.xlsx                 ← precios PC y PV por producto y CUPS
└── [fichero_pendientes].xlsx          ← exportación BC facturas nuevas sin registrar
```

### Cómo acceder a una factura de compra
```python
import os, glob
ONEDRIVE = os.path.expanduser(
    "~/Library/CloudStorage/OneDrive-TRANSACTIVA2000,SL/TRANSACTIVA2000"
)
FACTURAS_COMPRA = os.path.join(ONEDRIVE, "CONTABILIDAD ANALÍTICA_")

# Listar facturas de un CUPS
cups = "ES0021000001197056KH"
pdfs = glob.glob(f"{FACTURAS_COMPRA}/{cups}/*.pdf")

# Leer un PDF concreto
# → usar Read tool de Claude Code sobre la ruta completa
```

### Cómo encontrar la factura de compra equivalente a una de venta
1. Tomar el período de la factura de venta (ej. 01/03/26–31/03/26)
2. `ls "CONTABILIDAD ANALÍTICA_/{CUPS}/"` → listar PDFs disponibles
3. Leer cada PDF hasta encontrar el que cubre ese período
4. El período de compra NO siempre coincide con el de venta — leer el PDF para confirmar

---

## Tarifas

| Tarifa | Tipo | Períodos energía | Períodos potencia |
|--------|------|-----------------|------------------|
| 2.0TD | Residencial/pequeño comercio | P1, P2, P3 | P1, P2 (P1=punta, P2=valle) |
| 3.0TD | Comercial | P1–P6 | P1–P6 |
| 6.1TD | Industrial | P1–P6 | P1–P6 |

---

## Productos Business Central (BC)

| Código | Concepto | Unidad | Observación |
|--------|----------|--------|-------------|
| 1001 | Excedentes solares | €/kWh | Solo CUPS con solar. PV < PC por diseño |
| 1003 | Alquiler contador | €/mes | Fijo |
| 1004 | P1 Potencia | €/kW×días | qty = kW contratados |
| 1005 | P2 Potencia | €/kW×días | qty = kW contratados |
| 1006 | P3 Potencia | €/kW×días | qty = kW contratados |
| 1007 | P4 Potencia | €/kW×días | qty = kW contratados |
| 1008 | P5 Potencia | €/kW×días | qty = kW contratados |
| 1009 | P6 Potencia | €/kW×días | qty = kW contratados |
| 1012 | Tasa eléctrica | € | Base × 5,11269% (ver fórmula) |
| 1041 | Bono social | €/día | 0,019121 €/día × días |
| 1100 | P1 Consumo energía | €/kWh | qty = kWh |
| 1101 | P2 Consumo energía | €/kWh | qty = kWh |
| 1102 | P3 Consumo energía | €/kWh | qty = kWh |
| 1103 | P4 Consumo energía | €/kWh | qty = kWh |
| 1104 | P5 Consumo energía | €/kWh | qty = kWh |
| 1105 | P6 Consumo energía | €/kWh | qty = kWh |

---

## Normativa Vigente

| Norma | Contenido | Fecha |
|-------|-----------|-------|
| RD 7/2026 | Tasa eléctrica 0,5% para Iberdrola y Repsol (antes 5,11%) | 20/03/2026 |
| RD 7/2026 | IVA electricidad reducido al 10% | 20/03/2026 |
| Orden TED/2026 | Bono social: 0,019121 €/día | 2026 |
| Orden TED/133/2026 | FNEE: 0,002658 €/kWh | 2026 |
| RD 244/2019 | Excedentes solares: cap PC ≤ 0,06 €/kWh | 2019 |

---

## Fórmulas Fiscales

### Tasa eléctrica
```
# VENTA (siempre 5,11% independiente del proveedor)
Base_PV  = Σ importe_potencia_PV + Σ importe_energia_PV
Tasa_PV  = Base_PV × 5,1126963%

# COMPRA (depende del proveedor)
Base_PC  = Σ importe_potencia_PC + Σ importe_energia_PC
Tasa_PC  = Base_PC × 5,1126963%   # Naturgy y otros
Tasa_PC  = Base_PC × 0,5%         # Iberdrola y Repsol (RD 7/2026)
```

### Bono social
```
Importe = 0,019121 €/día × días_facturados
Ejemplo 31 días: 0,019121 × 31 = 0,59275 €
```

### IVA
```
Base imponible 21%: potencia + energía + bono + alquiler
Base exenta 0%:     tasa eléctrica
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

## Reglas del Motor de Facturación

### Margen y ajuste de kWh
| ID | Regla | Parámetro |
|----|-------|-----------|
| R01 | Margen mínimo global s/ventas ≥ 20% | 20% |
| R02 | Margen objetivo 3.0TD/6.1TD ≥ 30% | 30% |
| R03 | Si margen < mínimo → ajustar kWh por periodo (LP) | ±15% vs histórico mensual |
| R04 | Si LP no alcanza margen → ACEPTAR y flag. **NO tocar potencia** | Sin cambio en pot |
| R05 | Precio €/kWh energía: **NO se modifica dinámicamente** | Fijo en contrato |
| R06 | Distribución kWh 3.0TD: respetar tabla horaria oficial | θ(mes,periodo) × 1.4 máx |
| R07 | Potencia: **NUNCA bajar. No subir para tapar margen** | KW intocable |
| R08 | Armonía presupuestaria: factura coherente con histórico | CV < 0.4 en ventana 3m |

### Alertas de calidad / lectura
| ID | Regla | Umbral |
|----|-------|--------|
| R09 | Variación kWh vs período anterior | > ±50% → revisar |
| R10 | Variación kWh vs mismo período año anterior | > ±50% → revisar |
| R11 | Residencial verano: caída sospechosa | > 70% caída jul-ago |
| R12 | 3.0TD: todos los kWh en un solo periodo | Error lectura probable |

### Regulatorio (nunca modificar para ajustar margen)
| ID | Regla | Valor |
|----|-------|-------|
| R13 | Tasa PV = base × 5,11269% | Siempre |
| R14 | Tasa PC Iberdrola/Repsol = 0,5% | RD 7/2026 |
| R15 | Tasa PC Naturgy y otros = 5,11269% | Regulatorio |
| R16 | Bono social = 0,019121 €/día × días | TED/2026 |
| R17 | Cap excedentes PC ≤ 0,06 €/kWh | RD 244/2019 |
| R18 | Tasa y bono NUNCA se tocan para ajustar margen | Fijo |
| R19 | Facturar antes del día 5 de cada mes | Operativo |
| R20 | Sandbox siempre antes de producción BC | Obligatorio |

---

## Proceso de Revisión de Facturas (Mensual)

### Criterio de comparación
Para cada factura nueva pendiente, buscar en el histórico de ventas BC:
1. **Período anterior**: factura del mismo CUPS cuyo período termina justo antes
2. **Mismo período año anterior**: factura del mismo CUPS con el mismo mes de período pero del año anterior
- NO usar fecha de registro — usar período facturado
- Histórico válido desde 01/01/2025 únicamente

### Cómo leer el histórico de ventas BC
```python
# Fichero: Gene_rico_YYYYMMDD.xlsx (exportación BC tabla 113)
# Estructura: fila 0=paquete, fila 1=vacía, fila 2=cabeceras, filas 3+=datos
# Columnas clave:
#   'Nº documento'              → nº factura
#   'Venta a-Nº cliente'        → código cliente (C02330, etc.)
#   'Nº'                        → código producto (1004, 1100, etc.)
#   'Cantidad'                  → qty (kWh o kW)
#   'Precio venta'              → PV unitario
#   'Importe'                   → importe línea
#   '% IVA'                     → tipo IVA (0 o 21)
#   'Cód. dim. acceso dir. 1'   → CUPS
#   'Descripción'               → texto libre (contiene "Período de Lectura: DD/MM/AA a DD/MM/AA")
```

### Cálculo de margen sobre compra
```python
# Usar TABLA_MAESTRA.xlsx → hoja "Compra vs Venta"
# Estructura: sección por CUPS, filas por producto con PC y PV
# Margen = (Σ PV×qty - Σ PC×qty) / (Σ PV×qty)
# El cálculo se hace por TOTALES de factura, no línea a línea
```

### Señales de error de lectura (NO registrar sin corrección)
- kWh de energía solo en P6 y cero en el resto (3.0TD)
- kWh total < 15% del histórico del mismo período
- Variación > 80% vs período anterior sin justificación

---

## Proceso de Facturación Mensual (5 Fases)

### Fase 1 — Lectura (días 1-3)
1. Obtener fichero lecturas SIPS del distribuidor
2. Motor: alerta si variación > ±50% vs período anterior
3. Motor: alerta si variación > ±50% vs mismo período año anterior
4. Revisión manual alertas: ¿baja, vacaciones, contrato cancelado?
5. Suavizar curva: sin dientes de sierra (CV < 0.4 en ventana 3 meses)

### Fase 2 — Precios (días 3-4)
1. Cargar `Mascara_BC_Transactiva.xlsx` en BC **SANDBOX primero**
2. Verificar margen ≥ 20% en todos los CUPS
3. Para 3.0TD: verificar margen ≥ 30% o flag para renovación
4. Confirmar tasa: 0,5% Iberdrola/Repsol | 5,11% resto
5. Confirmar bono: 0,019121 €/día × días facturados

### Fase 3 — 3.0TD Manual (día 4)
1. Revisar clientes Tier A individualmente (> 800 kWh/mes)
2. Redistribuir kWh P1/P2 → P6 donde posible (tabla horaria)
3. No superar: θ(mes, periodo) × total_kWh × 1.4
4. Comparar con período anterior y mismo período año anterior
5. Leer factura de compra equivalente de `CONTABILIDAD ANALÍTICA_/`

### Fase 4 — Validación (día 4-5)
1. 0 alertas críticas → pasar SANDBOX a PRODUCCIÓN
2. Revisar total factura vs período anterior (coherencia)
3. Verificar excedentes solares PV ≥ 0,055 €/kWh
4. Emitir facturas **antes del día 5**

### Fase 5 — Control
1. Registrar margen real vs plan LP
2. Alerta si margen real < 85% del previsto
3. Re-segmentación trimestral

---

## Modelo LP (Programación Lineal kWh)

```
MAX  Σₚ (PVₚ − PCₚ) · qₚ   [solo energía — potencia fija]

Variables: qₚ = kWh en periodo p

Restricciones:
  R1. Conservación total: Σqₚ ∈ [0.85, 1.15] × Q_referencia
  R2. Límite horario: qₚ ≤ Q_ref × θ(mes,p) × 1.4
  R3. No negatividad: qₚ ≥ 0
  R4. Tasa e impuestos: excluidos (regulatorio fijo)
  R5. Potencia: intocable — si margen no alcanza → flag, no forzar

Q_referencia = kWh del mismo período año anterior (desde 01/01/2025)
Solver: HiGHS via scipy.optimize.linprog
```

---

## Segmentación de Cartera (185 CUPS)

| Tier | N | Mg €/año | kWh/mes | Acción |
|------|---|----------|---------|--------|
| A — Cuidar (>800 kWh, margen OK) | 8 | 7.856 € | 1.970 | Contrato 24m, visita semestral |
| A — Reactivar (>800 kWh, margen peligroso) | 1 | 12 € | 2.134 | Subir PV urgente en renovación |
| B — Optimizar (LP rentable) | 8 | 3.879 € | 420 | LP redistribución periodos |
| B — Mantener (margen estable) | 26 | 5.541 € | 325 | Precio fijo 24m |
| C — Fidelizar (77% cartera) | 142 | 8.948 € | 66 | Pack precio fijo 12m |

**Margen total actual: 26.237 €/año**
**Potencial LP: 30.094 € (+3.858 €, +26%)**

### Casos críticos
- **C03270 LULU BEACH CLUB** (3.0TD, 2.134 kWh): margen 12 €/año → subir PV en renovación
- **ES0021000001209594ZK** (MEDITERRÁNEO 96): Plan Variable → sin precio fijo posible
- **C00790 CASA LILI** (3.0TD): LP detecta +1.182 €
- **C01770 CASA LILI JAVEA** (6.1TD): LP detecta +761 €

### Autoconsumo solar
- 62 CUPS con excedentes (prod 1001)
- Objetivo fidelización: PV ≥ 0,055 €/kWh
- Cap compra: ≤ 0,06 €/kWh (RD 244/2019)

---

## Estacionalidad Media Cartera

| Ene | Feb | Mar | Abr | May | Jun | Jul | Ago | Sep | Oct | Nov | Dic |
|-----|-----|-----|-----|-----|-----|-----|-----|-----|-----|-----|-----|
| 0.71 | 0.81 | 0.81 | 0.89 | 0.90 | 0.77 | 1.04 | 1.11 | 1.10 | 0.89 | 0.86 | 0.73 |

Temporada alta: julio–septiembre. Temporada baja: enero, diciembre.

---

## Archivos de Trabajo

| Archivo | Descripción | Actualizar |
|---------|-------------|-----------|
| `TABLA_MAESTRA.xlsx` | PC y PV por producto y CUPS. Hoja "Compra vs Venta" | Cuando cambien precios |
| `Mascara_BC_Transactiva.xlsx` | Líneas BC listas para importar. 4 hojas: máscara+alertas+reglas+checklist | Mensual |
| `Compra_vs_Venta_por_CUPS.xlsx` | Excel 185 CUPS con márgenes, segmentación, solar | Mensual |
| `Gene_rico_YYYYMMDD.xlsx` | Exportación BC ventas registradas (tabla 37) | Mensual |
| `[pendientes]_CORREGIDO.xlsx` | Facturas nuevas corregidas listas para importar BC | Por campaña |
| `CONTABILIDAD ANALÍTICA_/{CUPS}/*.pdf` | Facturas de compra por CUPS | Automático (llegan de distribuidoras) |

---

## Principios de Operación

1. **Potencia**: NUNCA bajar. No subir para tapar margen — el cliente lo nota.
2. **Precio €/kWh**: NO tocar. Solo ajustar kWh.
3. **Tasa y bono**: NUNCA modificar para ajustar margen. Son regulatorios.
4. **Armonía presupuestaria**: sin saltos injustificados vs histórico.
5. **Si LP no alcanza**: aceptar y flag para renovación. No forzar.
6. **3.0TD**: revisión manual obligatoria. Tier A siempre.
7. **Sandbox siempre** antes de producción BC.
8. **Histórico válido**: solo desde 01/01/2025.
9. **Referencia correcta**: período facturado, no fecha de registro BC.

---

## Estado del Proyecto (junio 2026)

### Completado ✅
- Excel 185 CUPS con índice, búsqueda, márgenes, solar
- Precios compra extraídos de 203 facturas (Naturgy, Iberdrola, Repsol)
- Precios venta actualizados con histórico real BC (517 registros)
- Tasa recalculada con RD 7/2026 (0,5% Iberdrola/Repsol)
- Modelo LP 28 clientes 3.0TD/6.1TD
- Segmentación 5 tiers (185 clientes)
- Máscara BC (1.554 líneas, potencia intocable)
- Motor de validación facturas (bono, tasa, kWh, coherencia)
- Visor de revisión de facturas (pendiente + mes anterior + mismo período año anterior)
- Margen s/compra calculado por factura desde TABLA_MAESTRA

### Pendientes ⚠️
- 2 CUPS con margen < 20% → flagueados para renovación
- C03270 LULU BEACH: subir PV en próxima renovación
- ES0021000001209594ZK: Plan Variable sin solución
- Motor de alertas automatizado (actualmente manual)
- Control desviación mensual automatizado
- Integración SIPS lecturas automáticas
