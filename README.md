# Listado Completo de Variables - Scoring vs NBO-CRC

## Fecha de Análisis
- Enero 2026

## Fuente
- Documento "Variables Consolidado" compartido por el equipo de Scoring (Nico)

---

## Leyenda

| Símbolo | Significado |
|---------|-------------|
| ✅ | **Ya la usamos** en el modelo NBO-CRC |
| ⚠️ | **Parcialmente** implementada |
| ❌ | **No la tenemos** |

---

## BLOQUE 0: Magnitud y Volumen

Mide el tamaño absoluto del compromiso financiero del cliente en el sistema y con la entidad.

| # | Variable | Fórmula/Descripción | Estado | Variable NBO |
|---|----------|---------------------|--------|--------------|
| 0.1 | **Deuda Total Real** | `Dactiva + Dincobrable + Dvendida + Dfideicomiso` | ✅ | `crc_deuda_total` |
| 0.2 | **Deuda Activa** | Suma de deuda vigente | ✅ | `crc_deuda_activa_total` |
| 0.3 | **Deuda Vendida** | Deuda castigada y vendida | ✅ | `crc_deuda_vendida_total` |
| 0.4 | **Deuda Incobrable** | Deuda castigada | ✅ | `crc_deuda_incobrable_total` |
| 0.5 | **Deuda en Tarjetas (DTC)** | `Dactiva WHERE TipoOperacion = 'TARJETA DE CRÉDITO'` | ❌ | - |
| 0.6 | **Deuda en Préstamos** | `Dactiva WHERE TipoOperacion = 'PRÉSTAMOS AMORTIZABLES'` | ❌ | - |
| 0.7 | **Exposición Total** | `Dtotal + Contingencia` | ⚠️ | `crc_total_riesgo` (parcial) |
| 0.8 | **Deuda Interna (UENO/VISION)** | `Dtotal WHERE CodEntidad IN ('UENO', 'VISION')` | ❌ | - |
| 0.9 | **Deuda Externa** | `Dtotal - Dinterna` | ❌ | - |
| 0.10 | **Exposición Contingente** | Líneas de crédito disponibles no utilizadas | ❌ | - |

---

## BLOQUE 1: Morosidad y Calidad de Cartera

Unifica el comportamiento de pago del cliente a través de todos sus productos para identificar su nivel de riesgo real.

| # | Variable | Fórmula/Descripción | Estado | Variable NBO |
|---|----------|---------------------|--------|--------------|
| 1.1 | **Máximo Atraso Histórico (6M)** | `MAX(diasatraso)` en ventana 6 meses | ✅ | `crc_max_dias_atraso` |
| 1.2 | **Peor Clasificación Regulatoria** | `MAX(clasificacionsistema)` en ventana | ✅ | `crc_peor_clasif_3M/6M` |
| 1.3 | **Ratio de Toxicidad** | `(Dincobrable + Dvendida) / Dtotal` | ❌ | - |
| 1.4 | **Flag de Peor Momento** | `1 si Atraso_t >= MAX(Atraso_t-1:t-5)` | ❌ | - |
| 1.5 | **Flag Atraso > 0** | `1 si diasatraso > 0` | ✅ | `crc_flag_dias_atraso` |
| 1.6 | **Flag Atraso > 30** | `1 si diasatraso > 30` | ✅ | `crc_flag_atraso_30d` |
| 1.7 | **Flag Atraso > 60** | `1 si diasatraso > 60` | ✅ | `crc_flag_atraso_60d` |
| 1.8 | **Flag Atraso > 90** | `1 si diasatraso > 90` | ✅ | `crc_flag_atraso_90d` |

---

## BLOQUE 2: Estructura y Share of Wallet

Aprovecha el desglose por producto del consolidado para entender la composición de la deuda.

| # | Variable | Fórmula/Descripción | Estado | Variable NBO |
|---|----------|---------------------|--------|--------------|
| 2.1 | **Share of Wallet TC** | `DTC / Dtotal` | ❌ | - |
| 2.2 | **Share of Wallet Préstamos** | `DPréstamos / Dtotal` | ❌ | - |
| 2.3 | **Exposición Total Real** | `Dvigente + Dvencida + Dcastigada + Contingente` | ⚠️ | Parcial |
| 2.4 | **Flag Tenencia TC** | `1 si DTC > 0` | ❌ | - |
| 2.5 | **Flag Tenencia Préstamos** | `1 si DPréstamos > 0` | ❌ | - |
| 2.6 | **Índice Diversidad (Entropía Shannon)** | `H = -Σ (Di/Dtotal) * ln(Di/Dtotal)` | ❌ | - |

---

## BLOQUE 3: Variables de Velocidad (Deltas Granulares)

Incorpora la dinámica de cambio mes a mes. Mientras el saldo dice cuánto debe, la velocidad dice hacia dónde va.

| # | Variable | Fórmula/Descripción | Estado | Variable NBO |
|---|----------|---------------------|--------|--------------|
| 3.1 | **Variación % Deuda MoM** | `(Dt - Dt-1) / Dt-1` | ❌ | - |
| 3.2 | **Delta Absoluto Días Atraso** | `Atraso_t - Atraso_t-1` | ❌ | - |

---

## BLOQUE 4: Aceleración del Deterioro (Segunda Derivada)

Detecta si el deterioro se está acelerando o desacelerando.

| # | Variable | Fórmula/Descripción | Estado | Variable NBO |
|---|----------|---------------------|--------|--------------|
| 4.1 | **Aceleración de la Mora** | `(Atraso_t - Atraso_t-1) - (Atraso_t-1 - Atraso_t-2)` | ❌ | - |
| 4.2 | **Aceleración de Deuda** | `(Dt - Dt-1) - (Dt-1 - Dt-2)` | ❌ | - |

**Interpretación:**
- Si > 0: El atraso/deuda crece cada vez más rápido (exponencial)
- Si < 0: El atraso/deuda crece pero se está frenando

---

## BLOQUE 5: Momentum Financiero (Indicador RSI)

Aplica el Relative Strength Index para medir la fuerza de la tendencia del comportamiento.

| # | Variable | Fórmula/Descripción | Estado | Variable NBO |
|---|----------|---------------------|--------|--------------|
| 5.1 | **RSI de Morosidad (6M)** | `100 - 100/(1 + RS)` sobre diasatraso | ❌ | - |
| 5.2 | **RSI de Agotamiento de Línea** | RSI sobre montocontingencia (invertido) | ❌ | - |

**Fórmula RSI:**
```
Gain = Promedio de cambios positivos en la ventana
Loss = Promedio de valores absolutos de cambios negativos
RS = Gain / Loss
RSI = 100 - 100/(1 + RS)
```

**Interpretación:**
- RSI > 70: Sobre-apalancamiento, espiral de endeudamiento
- RSI < 30: Desapalancamiento, reduciendo exposición

---

## BLOQUE 6: Ratios de Estrés y Saturación

Variables adimensionales que contextualizan la deuda respecto a la capacidad aparente del cliente.

| # | Variable | Fórmula/Descripción | Estado | Variable NBO |
|---|----------|---------------------|--------|--------------|
| 6.1 | **Ratio Saturación Total (Utilization)** | `Dactiva / (Dactiva + Contingencia)` | ❌ | - |
| 6.2 | **Uso Préstamos** | `Dpréstamos / (Dpréstamos + ContingenciaPréstamos)` | ❌ | - |
| 6.3 | **Uso TC** | `DTC / (DTC + ContingenciaTC)` | ❌ | - |
| 6.4 | **FMX de Deuda (Fraction of Max)** | `DeudaMes1 / Σ(DeudaMes1:6)` | ❌ | - |
| 6.5 | **Caída de Tendencia %** | `(TotalDeuda_t - Promedio_ant) / Promedio_ant * 100` | ❌ | - |
| 6.6 | **Z-Score Deuda** | `(TotalDeuda_t - Promedio_ant) / StdDev_ant` | ❌ | - |

**Nota:** El Ratio de Saturación es la variable predictora más crítica. Si tiende a 1.0 (100%), el cliente está "ahorcado" financieramente.

---

## BLOQUE 7: Alertas Tempranas y Eventos Críticos

Variables sintéticas diseñadas para capturar combinaciones de alto riesgo.

| # | Variable | Fórmula/Descripción | Estado | Variable NBO |
|---|----------|---------------------|--------|--------------|
| 7.1 | **Alerta Default Inminente** | `1 si (RSImora > 70) AND (Uso > 0.90)` | ❌ | - |
| 7.2 | **Flag Venta Cartera Reciente** | `1 si deudavendida pasó de 0 a positivo` | ⚠️ | Tenemos flag, no el cambio |
| 7.3 | **Flag de Fusión** | Detecta registros de fusión bancaria (Ueno/Visión, Río/Continental) | ❌ | - |

---

## BLOQUE 8: Comportamiento Revolvente vs Transaccional

Diferencia clientes que pagan mensualmente ("transactors") de los que mantienen saldos ("revolvers").

| # | Variable | Fórmula/Descripción | Estado | Variable NBO |
|---|----------|---------------------|--------|--------------|
| 8.1 | **Flag Revolver** | `1 si (DTC_t > 0) AND (DTC_t >= 0.9*DTC_t-1) AND (DTC_t-1 >= 0.9*DTC_t-2)` | ❌ | - |

---

## BLOQUE 9: Sensibilidad al Riesgo por Producto

Diferentes productos tienen diferentes sensibilidades al riesgo.

| # | Variable | Fórmula/Descripción | Estado | Variable NBO |
|---|----------|---------------------|--------|--------------|
| 9.1 | **Ratio Revolvente vs A Plazos** | `DeudaRevolvente / DeudaAPlazos` | ❌ | - |

**Nota:** Un ratio creciente a menudo señala problemas de flujo de efectivo.

---

## BLOQUE 10: Comportamiento Histórico

Probabilidades y patrones históricos de comportamiento.

| # | Variable | Fórmula/Descripción | Estado | Variable NBO |
|---|----------|---------------------|--------|--------------|
| 10.1 | **Roll Rate 30-60** | `1 si (1 <= Atraso_t-1 <= 30) AND (Atraso_t > 30)` | ❌ | - |
| 10.2 | **Tasa de Cura** | `Veces(Atraso_t-1>0 AND Atraso_t=0) / Veces(Atraso_t-1>0)` | ❌ | - |
| 10.3 | **Cant Días Atraso Histórico** | Suma histórica de días | ⚠️ | Tenemos max, no suma |
| 10.4 | **cant_veces_mora_1_30_ult_Km** | `Σ 1 if 1 <= D_t-i <= 30` | ❌ | - |
| 10.5 | **cant_rolls_30_a_60_ult_m** | `Σ 1 if (1 <= D_t-i-1 <= 30) AND (31 <= D_t-i <= 60)` | ❌ | - |
| 10.6 | **cant_curas_30_a_0_ult_Km** | `Σ 1 if (1 <= D_t-i-1 <= 30) AND (D_t-i = 0)` | ❌ | - |
| 10.7 | **meses_desde_ultimo_roll_30_60** | `min{k: transición 30→60 en t-k}` | ❌ | - |
| 10.8 | **meses_desde_ultima_mora_1_30** | `min{k: 1 <= D_t-k <= 30}` | ❌ | - |
| 10.9 | **max_saldo_en_mora_ult_12m** | `MAX{DeudaMora_t-k : 0 <= k <= 11}` | ❌ | - |
| 10.10 | **promedio_saldo_mora_ult_6m** | `AVG{DeudaMora_t-k : 0 <= k <= 5}` | ❌ | - |
| 10.11 | **ratio_saldo_mora_vs_limite** | `DeudaMora / Contingencia` | ❌ | - |
| 10.12 | **ratio_deuda_total_mora_vs_deuda_total** | `DeudaMora / DeudaTotal` | ❌ | - |

---

## VARIABLES DE DEPÓSITOS

Estadísticas y métricas relacionadas con los depósitos del cliente.

| # | Variable | Fórmula/Descripción | Estado | Variable NBO |
|---|----------|---------------------|--------|--------------|
| D.1 | **Total Depósitos Sistema** | Saldo total en depósitos | ✅ | `crc_depositos_sistema_total` |
| D.2 | **Promedio Depósitos 3M** | `AVG(Depósitos)` últimos 3 meses | ❌ | - |
| D.3 | **Promedio Depósitos 6M** | `AVG(Depósitos)` últimos 6 meses | ❌ | - |
| D.4 | **Máximo Depósitos 3M/6M** | `MAX(Depósitos)` en ventana | ❌ | - |
| D.5 | **StdDev Depósitos 3M/6M** | Desviación estándar (volatilidad) | ❌ | - |
| D.6 | **Variación % Depósitos** | `(Dep_t - Dep_t-1) / Dep_t-1 * 100` | ❌ | - |
| D.7 | **Variación Bruta Depósitos** | `Dep_t - Dep_t-1` | ❌ | - |
| D.8 | **Caída Tendencia Depósitos %** | `(Dep_t - Promedio_ant) / Promedio_ant * 100` | ❌ | - |
| D.9 | **Z-Score Depósitos** | `(Dep_t - Promedio_ant) / StdDev_ant` | ❌ | - |
| D.10 | **FMX Depósitos** | `Dep_t / (Dep_t + Dep_t-1 + Dep_t-2)` | ❌ | - |
| D.11 | **RSI Depósitos** | Momentum de acumulación/pérdida | ❌ | - |

---

## VARIABLES CROSS (Deuda vs Depósitos)

Variables que cruzan información de deuda con depósitos para medir capacidad de pago real.

| # | Variable | Fórmula/Descripción | Estado | Variable NBO |
|---|----------|---------------------|--------|--------------|
| C.1 | **Ratio Cobertura % (Deuda/Depósitos)** | `TotalDeuda / TotalDepósitos * 100` | ❌ | - |
| C.2 | **Liquidez Neta** | `TotalDepósitos - TotalDeuda` | ❌ | - |
| C.3 | **PLCR (Personal Liquidity Coverage Ratio)** | `(Depósitos + Contingencia) / (Deuda + Gastos)` | ❌ | - |
| C.4 | **Velocidad de Divergencia (Leverage Velocity)** | `TasaCrecimientoDeuda(6M) - TasaCrecimientoDepósitos(6M)` | ❌ | - |
| C.5 | **Ratio Cobertura Deuda Revolvente** | `TotalDepósitos / DeudaRevolvente` | ❌ | - |
| C.6 | **Ratio Cobertura Deuda Transaccional** | `TotalDepósitos / DeudaTransaccional` | ❌ | - |
| C.7 | **Ratio Dependencia de Deuda** | `Retiros / (NuevosDesembolsos + Depósitos)` | ❌ | - |
| C.8 | **Ratio Cobertura Simple** | `TotalDepósitos / DeudaActiva` | ❌ | - |

---

## VARIABLES AFECTADO (Codeudor/Garante)

Variables relacionadas con la exposición del cliente como codeudor o garante de terceros.

| # | Variable | Fórmula/Descripción | Estado | Variable NBO |
|---|----------|---------------------|--------|--------------|
| A.1 | **Flag es Codeudor** | `1 si tiene registros como afectado` | ✅ | `crc_flag_es_codeudor` |
| A.2 | **Conteo Registros Afectado** | Cantidad de operaciones como garante | ✅ | `crc_cnt_registros_afectado` |
| A.3 | **Exposición Sistema (EX_CRC)** | Suma de deuda + contingencia + indirecta | ❌ | - |
| A.4 | **Peor Calificación Sistema** | `MAX(clasificación)` en todo el sistema | ⚠️ | Tenemos por ventana |
| A.5 | **Variables Específicas UENO** | Aislamiento comportamiento con UENO (códigos 2007, 1046) | ❌ | - |
| A.6 | **Principalidad (MAX_BY)** | Banco con mayor deuda del cliente | ❌ | - |

---

## VARIABLES DE DINÁMICA Y VARIACIONES

Variables que miden cambios y velocidades en el tiempo.

| # | Variable | Fórmula/Descripción | Estado | Variable NBO |
|---|----------|---------------------|--------|--------------|
| V.1 | **Variación Lag-to-Lag (6M_2_1)** | Cambio entre meses específicos para shocks puntuales | ❌ | - |
| V.2 | **Promedios Móviles 3M/6M** | Volatilidad de cambios | ❌ | - |
| V.3 | **Hambre de Crédito (VAR_CNT_ENTIDADES)** | Velocidad apertura nuevas relaciones bancarias | ❌ | - |
| V.4 | **Var Deuda Vendida** | Cambio en deuda castigada | ❌ | - |
| V.5 | **Var Exposición Entidades** | Cambio en uso de líneas de crédito | ❌ | - |

**Nota:** Un aumento brusco en "Hambre de Crédito" es una Red Flag de búsqueda desesperada de liquidez.

---

## ESTADÍSTICAS DE VENTANA

Métricas de resumen para ventanas de 3 y 6 meses.

| # | Variable | Fórmula/Descripción | Estado | Variable NBO |
|---|----------|---------------------|--------|--------------|
| E.1 | **MAX Deuda 3M/6M** | Peor escenario en ventana | ❌ | - |
| E.2 | **AVG Deuda 3M/6M** | Tendencia estructural (suaviza ruido) | ⚠️ | Parcial (suma, no avg) |
| E.3 | **STDDEV Deuda 3M/6M** | Volatilidad (flujos erráticos = más riesgo) | ❌ | - |
| E.4 | **MAX Atraso 3M/6M** | Peor momento de mora | ✅ | `crc_max_dias_atraso` |

---

## RATIOS ESTRUCTURALES Y DE NEGOCIO

Añaden contexto relativo y proporción a los valores absolutos.

| # | Variable | Fórmula/Descripción | Estado | Variable NBO |
|---|----------|---------------------|--------|--------------|
| R.1 | **Share of Wallet UENO** | `DeudaUENO / DeudaSistema` | ❌ | - |
| R.2 | **Saturación** | `DeudaReal / (Deuda + Contingencia)` | ❌ | - |
| R.3 | **Tendencia Estructural (TEN)** | Comparación punto a punto Mes1 vs Mes6 | ❌ | - |
| R.4 | **Contribución Temporal / Shock (FMX)** | `ValorActual / TotalVentana` | ❌ | - |
| R.5 | **Flag Cambio Principalidad** | `1 si banco principal cambió en últimos 6M` | ❌ | - |

**Nota sobre Share of Wallet:** Si el ratio baja, estamos perdiendo al cliente frente a la competencia.

---

## 📊 RESUMEN CONSOLIDADO

| Categoría | Total Variables | ✅ Tenemos | ⚠️ Parcial | ❌ Falta |
|-----------|-----------------|------------|------------|----------|
| Magnitud y Volumen | 10 | 4 | 1 | 5 |
| Morosidad | 8 | 6 | 0 | 2 |
| Estructura/SoW | 6 | 0 | 1 | 5 |
| Velocidad (Deltas) | 2 | 0 | 0 | 2 |
| Aceleración | 2 | 0 | 0 | 2 |
| Momentum (RSI) | 2 | 0 | 0 | 2 |
| Ratios Estrés | 6 | 0 | 0 | 6 |
| Alertas | 3 | 0 | 1 | 2 |
| Revolvente | 1 | 0 | 0 | 1 |
| Sensibilidad | 1 | 0 | 0 | 1 |
| Comportamiento Hist | 12 | 0 | 1 | 11 |
| Depósitos | 11 | 1 | 0 | 10 |
| Cross | 8 | 0 | 0 | 8 |
| Afectado | 6 | 2 | 1 | 3 |
| Dinámica | 5 | 0 | 0 | 5 |
| Estadísticas Ventana | 4 | 1 | 1 | 2 |
| Ratios Estructurales | 5 | 0 | 0 | 5 |
| **TOTAL** | **~92** | **14 (15%)** | **6 (7%)** | **72 (78%)** |

---

## 🎯 TOP 10 Variables Recomendadas a Incorporar

Basándome en impacto potencial y facilidad de implementación:

| Prioridad | Variable | Impacto | Dificultad | Fórmula |
|-----------|----------|---------|------------|---------|
| 1 | **Ratio Saturación** | 🔥🔥🔥 | Fácil | `Deuda/(Deuda+Contingencia)` |
| 2 | **Ratio Toxicidad** | 🔥🔥🔥 | Fácil | `(Incobrable+Vendida)/Total` |
| 3 | **Ratio Deuda/Depósitos** | 🔥🔥🔥 | Fácil | `DeudaTotal/Depósitos` |
| 4 | **Share of Wallet TC** | 🔥🔥 | Media | Requiere desglose por producto |
| 5 | **Variación % Deuda** | 🔥🔥 | Media | Requiere historia mensual |
| 6 | **Delta Días Atraso** | 🔥🔥 | Media | Requiere historia mensual |
| 7 | **Liquidez Neta** | 🔥🔥 | Fácil | `Depósitos - Deuda` |
| 8 | **Flag Peor Momento** | 🔥🔥 | Media | Comparar actual vs histórico |
| 9 | **Contingencia** | 🔥🔥 | Fácil | Ya está en tablas CRC |
| 10 | **RSI Morosidad** | 🔥 | Difícil | Cálculo complejo |


