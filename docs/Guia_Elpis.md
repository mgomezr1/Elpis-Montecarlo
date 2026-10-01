# Elpís 1.0: guía técnica

**Simulación Monte Carlo para evaluar proyectos de inversión.** Un proyecto no tiene un solo futuro: Elpís simula miles y le dice cuántos crean valor.

Autor: Mario Sergio Gómez Rueda · mgomezr1@gmail.com
Uso: este aplicativo fue desarrollado por Mario Sergio Gómez Rueda. Su uso es de carácter académico y cualquier otro uso se regirá por el derecho de la propiedad intelectual.

## 1. Descripción

Elpís es una aplicación web académica, autocontenida en un solo archivo `index.html`, que evalúa un proyecto de inversión con simulación Monte Carlo (Metropolis y Ulam, 1949; Hertz, 1964):

- El flujo de caja libre se construye con **variables generadoras** que el usuario define: precio, cantidad, costos, inversión, valor de rescate o cualquier otra.
- Cada variable puede ser **fija** o seguir una distribución **uniforme, triangular, PERT, normal, lognormal o discreta**.
- Cada variable incierta se muestrea **una vez por iteración** o **una vez por periodo**, y puede **crecer** a una tasa fija o a la tasa que marque otra variable.
- Las variables inciertas pueden ser **independientes o correlacionadas** (correlación de rangos de Spearman).
- La tasa de descuento es **directa** o el **WACC**, con Ke ingresado o calculado con **CAPM**.
- En cada iteración se calculan el **VPN**, la **TIR**, la **TIRM** y el **periodo de recuperación descontado**.

El resultado es la distribución completa de cada indicador, la probabilidad de que el proyecto destruya valor, una recomendación según el criterio de riesgo del usuario y un análisis de sensibilidad y robustez.

Forma parte de la familia de aplicativos académicos con nombres griegos (Kairós AHP, Áristos ELECTRE, Éngista TOPSIS, Axía MAUT, Seismos, Klísis, entre otros). **ἐλπίς** (elpís) significa esperanza y también expectativa: en el mito de Pandora fue lo único que quedó dentro del ánfora.

## 2. Cómo guardarlo y ejecutarlo

1. Guarde `index.html` en una carpeta.
2. Ábralo con doble clic en Chrome, Edge, Firefox o Safari. No requiere instalación, servidor ni registro.
3. Para publicarlo en GitHub Pages, suba el contenido del paquete a un repositorio y active Pages sobre la rama principal, carpeta raíz.

Todos los cálculos ocurren en el navegador. La única dependencia externa es SheetJS (cdn.sheetjs.com), usada solo para generar el Excel. Sin conexión funcionan la simulación, los gráficos, el guardado del modelo y el PDF.

## 3. Estructura de la interfaz

| Paso | Pregunta que responde | Contenido |
|---|---|---|
| Portada | ¿Qué hace una simulación? | Histograma interactivo de un proyecto de ejemplo con su distribución exacta. |
| 00 Método | ¿Cómo funciona? | Pasos del método, estructura del flujo, distribuciones, muestreo, correlación y lectura de resultados. |
| 01 Modelo | ¿Qué proyecto, horizonte y tasa? | Nombre, moneda, periodicidad, N, tasa directa o WACC, impuesto, escudo fiscal, iteraciones, semilla y límite de riesgo. Se confirma antes de seguir. |
| 02 Variables y flujo | ¿Qué supuestos generan el flujo? | Fichas de variables, líneas del flujo, capital de trabajo, matriz de correlación y escenario base. Guardar y abrir el modelo en JSON. |
| 03 Simulación | ¿Cuál es la distribución del valor? | Decisión, cifras clave, histograma, probabilidad acumulada, flujo por periodo, convergencia y Tablas 1 a 3. |
| 04 Sensibilidad | ¿Qué tendría que cambiar? | Tornado (Tabla 4), puntos de equilibrio (Tabla 5), tasa de descuento (Tabla 6) y frontera de la decisión. |
| 05 Informe | ¿Cómo lo comunico? | Excel con fórmulas e informe ejecutivo en PDF. |
| i Acerca de | ¿Es confiable? | Nombre, autoría, pruebas automáticas, escenarios de prueba, referencias y cita. |

El riel lateral muestra el estado de cada paso y se convierte en barra superior en pantallas pequeñas.

## 4. Identidad visual

Fondo petróleo profundo (#04232B) y un par de colores con significado: **oro** (#FFC857) para la esperanza, es decir, los futuros con VPN mayor o igual que cero, y **coral** (#FF6F59) para el riesgo, los futuros con VPN negativo. El ícono es el ánfora de Pandora con un pequeño histograma dentro. Tipografía del sistema; el bloque `@font-face` comentado permite usar fuentes locales.

## 5. Fundamento de cálculo

### 5.1 Generador aleatorio

xoshiro128** de Blackman y Vigna (2021), sembrado con splitmix32. Devuelve uniformes en el intervalo abierto (0, 1). La misma semilla reproduce exactamente la misma simulación.

### 5.2 Muestreo por inversión

Todas las distribuciones se muestrean como x = Q(u), donde Q es la función cuantil:

| Distribución | Cuantil Q(u) | Valor central | Media |
|---|---|---|---|
| Uniforme (a, b) | a + u (b − a) | (a + b) / 2 | (a + b) / 2 |
| Triangular (a, m, b) | a + √(u (b − a)(m − a)) si u < (m − a)/(b − a); si no, b − √((1 − u)(b − a)(b − m)) | m | (a + m + b) / 3 |
| PERT (a, m, b) | a + (b − a) · B⁻¹(u; α, β), con α = 1 + 4 (m − a)/(b − a), β = 1 + 4 (b − m)/(b − a) | m | (a + 4m + b) / 6 |
| Normal (μ, σ) | μ + σ Φ⁻¹(u) | μ | μ |
| Lognormal (m, s) | exp(μL + σL Φ⁻¹(u)), con σL² = ln(1 + s²/m²) y μL = ln m − σL²/2 | m | m |
| Discreta | primer valor cuya probabilidad acumulada alcanza u | valor esperado | valor esperado |

Φ⁻¹ es el algoritmo AS 241 de Wichura (1988). B⁻¹ se obtiene por Newton protegido con bisección sobre la beta incompleta regularizada, calculada con la fracción continua y el logaritmo de la gamma de Press et al. (2007). Las variables marcadas en % se ingresan en puntos porcentuales y se dividen por 100.

### 5.3 Crecimiento

Con tasa fija g: x(t) = x · (1 + g)^(t − 1) para t ≥ 2. Con otra variable G en %: x(t) = x · Π(1 + G(s)) para s = 2, …, t. La variable de crecimiento no puede tener a su vez crecimiento.

### 5.4 Correlación

El usuario ingresa la correlación de rangos de Spearman ρs. Elpís usa una cópula gaussiana: la convierte en la correlación de normales ρ = 2 sen(π ρs / 6) (Kruskal, 1958), obtiene L por Cholesky, genera z = L e con e normales independientes y transforma u = Φ(z). Así se conserva la forma de cada distribución y se logra la correlación de rangos pedida, con la misma idea de Iman y Conover (1982).

Las variables por iteración se ordenan antes que las por periodo. Como L es triangular inferior, las primeras dependen solo de normales fijas en la iteración y las segundas se renuevan cada periodo: dentro de un periodo todos los pares tienen la correlación pedida; entre periodos, las variables por periodo son independientes.

Si la matriz no es definida positiva, Elpís puede reemplazarla por la matriz de correlación válida más cercana en norma de Frobenius con proyecciones alternantes y corrección de Dykstra (Higham, 2002), y muestra la matriz ingresada junto a la usada.

### 5.5 Flujo de caja libre

Cada línea es el producto de dos factores: variable por variable, variable sola, o (solo en costos variables) variable en % por los ingresos totales del periodo.

```
Ingresos(t) = Σ líneas de ingreso
EBITDA(t)   = Ingresos(t) − Costos variables(t) − Costos fijos(t)
UAII(t)     = EBITDA(t) − Depreciación(t)
Impuesto(t) = t · UAII(t)        (si no hay escudo fiscal y UAII < 0, impuesto = 0)
NOPAT(t)    = UAII(t) − Impuesto(t)
KT(t − 1)   = %KT(t) · Ingresos(t);  KT(N) = 0 si se recupera
ΔKT(t)      = KT(t) − KT(t − 1)
FCL(t)      = NOPAT(t) + Depreciación(t) − CAPEX(t) − ΔKT(t) + Rescate neto(t)
```

Cada CAPEX del periodo p con vida útil v se deprecia en línea recta, CAPEX / v, en los periodos p + 1 a p + v dentro del horizonte. El valor de rescate se registra en N y se descuenta el impuesto sobre su diferencia con el valor en libros: Rescate neto = VR − t (VR − VL).

### 5.6 Tasa e indicadores

Tasa directa o WACC = E/V · Ke + D/V · Kd (1 − t), con Ke = rf + β · prima de mercado + prima adicional si se usa CAPM. La tasa efectiva anual se lleva a la periodicidad del flujo: r = (1 + tasa EA)^(1/k) − 1.

- VPN = Σ FCL(t) / (1 + r)^t.
- TIR: Newton desde 10 % (como Excel) y, si no converge, bisección sobre la primera raíz que aparezca en una rejilla de tasas. No existe si el flujo no cambia de signo. Se informa cuántas iteraciones tienen más de un cambio de signo.
- TIRM = (VF de los flujos positivos a r / VP de los negativos a r)^(1/N) − 1, equivalente a MIRR de Excel con financiamiento y reinversión a r.
- Periodo de recuperación descontado, interpolado dentro del periodo en que el acumulado se vuelve no negativo.

### 5.7 Estadísticos

Media, desviación estándar muestral, error estándar σ/√n, intervalo de confianza del 95 % de la media, mínimo, máximo, asimetría y curtosis en exceso ajustadas, y percentiles con la definición de PERCENTILE.INC de Excel. Probabilidad de VPN negativo con su error estándar √(p(1 − p)/n).

### 5.8 Decisión

| Resultado | Condición |
|---|---|
| Crea valor | VPN esperado > 0 y P(VPN < 0) ≤ límite del usuario |
| Crea valor con riesgo alto | VPN esperado > 0 y P(VPN < 0) > límite |
| No crea valor | VPN esperado ≤ 0 |

El límite es un criterio del usuario; Elpís no impone un valor de referencia.

### 5.9 Sensibilidad y robustez

- **Tornado:** correlación de rangos de Spearman entre el valor muestreado de cada variable (el promedio de los periodos si se muestrea por periodo) y el VPN. Contribución = ρ² / Σρ² (Vose, 2008). Incluye la media de cada variable en los futuros con VPN negativo y no negativo.
- **Puntos de equilibrio:** multiplicador de cada variable, con las demás en su valor central, que lleva el VPN base a cero; se busca entre 0 % y 500 % del valor central.
- **Tasa:** los flujos simulados se descuentan de nuevo con otras tasas. La tasa que anula el VPN esperado es la TIR del flujo medio, porque el VPN es lineal en los flujos. La tasa que iguala la probabilidad de pérdida con el límite se halla por bisección.

## 6. Explicación de las funciones (JavaScript)

| Sección | Funciones principales |
|---|---|
| 2 Utilidades | `leerNumero`, `formatearNumero`, `formatearDinero`, `percentilOrdenado`, `describir` |
| 3 Distribuciones | `crearGenerador`, `normalCDF`, `normalInv`, `lnGamma`, `betaRegularizada`, `betaInv`, `crearCuantil`, `momentosDistribucion`, `valorCentral`, `validarParametros` |
| 4 Correlación | `spearmanAPearson`, `cholesky`, `eigenSimetrica`, `correlacionMasCercana`, `rangos`, `correlacionSpearman` |
| 5 Modelo | `calcularTasa`, `vpn`, `tir`, `tirm`, `periodoRecuperacion`, `compilarModelo`, `aplicarCrecimiento`, `construirFlujo`, `indicadores` |
| 6 Simulación | `crearSimulacion`, `simularSincrono`, `simularAsincrono`, `evaluarConTasa`, `analizar` |
| 7 a 12 Interfaz | mensajes, pasos, fichas, líneas, correlación, escenario base, resultados, sensibilidad y gráficos SVG propios |
| 13 Excel | `crearHojaExcel`, `hojaSupuestos`, `hojaVariables`, `hojaFlujo`, `hojaSimulacion`, `hojaEstadisticos`, `construirLibroExcel` |
| 14 PDF | `DocumentoPDF` (generador propio de PDF 1.4), `histogramaPDF`, `generarInformePDF` |
| 15 a 18 | escenarios de prueba, `ejecutarPruebas`, portada interactiva e `iniciar` |

`compilarModelo` valida todo el modelo y lo traduce a índices; `construirFlujo` es la única función que arma el flujo y la usan la simulación, el escenario base, el Excel, las pruebas y el PDF, de modo que todos muestran el mismo cálculo.

## 7. Contenido del Excel

| Hoja | Contenido |
|---|---|
| Menu | Proyecto, fecha, origen de los datos, decisión e índice con enlaces a cada hoja. |
| Supuestos | Datos generales; E/V, Ke por CAPM, WACC, tasa EA y tasa por periodo con fórmulas. |
| Variables | Distribución y parámetros; valor central, media y desviación teóricas con fórmulas; matriz de correlación usada. |
| Escenario base | Flujo completo con fórmulas: valores centrales enlazados a Variables, factores de crecimiento, líneas, depreciación, impuestos, capital de trabajo, rescate, FCL, VPN (NPV), TIR (IRR) y TIRM (MIRR). |
| Iteracion 1 | El mismo flujo con los valores muestreados en la primera iteración; coincide con la fila 1 de Simulacion. |
| Simulacion | Hasta 10.000 iteraciones: flujos como valores, VPN, TIR y TIRM con fórmulas y el valor muestreado de cada variable incierta. |
| Estadisticos | AVERAGE, STDEV, MIN, MAX, PERCENTILE, COUNTIF y un histograma con COUNTIFS sobre las iteraciones exportadas. |
| Sensibilidad | Tornado, puntos de equilibrio, efecto de la tasa y frontera de la decisión (valores). |
| Resultado | Decisión, cifras clave sobre todas las iteraciones y conclusión. |

Verificación: los libros de los escenarios de prueba se recalcularon en LibreOffice; las más de 31.000 fórmulas de cada uno coinciden con los valores de Elpís con una diferencia relativa máxima del orden de 10⁻¹⁵.

## 8. Contenido del PDF

Portada con aviso de datos de prueba si aplica; 1 Recomendación; 2 Distribución de los resultados (histograma y Tablas 1 y 2); 3 Estructura del modelo (supuestos, Tablas 3 y 4); 4 Calidad de la simulación; 5 Robustez (frontera y Tablas 5 a 7); 6 Nota metodológica, referencias y autoría. Todas las páginas llevan marca de agua diagonal «USO ACADÉMICO» y pie con nombre, versión, autor, nota de uso, correo y numeración.

## 9. Ejemplo verificado: cafetería universitaria

Escenario de prueba pequeño: N = 5 años, tasa directa del 14 % EA, impuesto del 35 % con escudo fiscal, capital de trabajo del 8 % de los ingresos del año siguiente, 10.000 iteraciones, semilla 2026.

| Variable | Distribución | Muestreo | Crecimiento |
|---|---|---|---|
| Precio promedio por cliente | Triangular 7.000; 8.000; 9.500 | Por iteración | Ninguno |
| Clientes por año | PERT 30.000; 40.000; 48.000 | Por iteración | 3 % anual |
| Costo variable por cliente | Uniforme 3.000 a 3.800 | Por periodo | Ninguno |
| Costos fijos anuales | Normal 110 M; 8 M | Por periodo | 4 % anual |
| Adecuación y equipos | Triangular 120 M; 140 M; 170 M | Por iteración | Ninguno |
| Valor de rescate | Fija 20 M | | |

**Escenario base, año 1, a mano:** ingresos 8.000 × 40.000 = 320.000.000; costo variable 3.400 × 40.000 = 136.000.000; costos fijos 110.000.000; depreciación 140.000.000 / 5 = 28.000.000; UAII 46.000.000; impuesto 16.100.000; NOPAT 29.900.000; ΔKT = 0,08 × (329.600.000 − 320.000.000) = 768.000; FCL = 29.900.000 + 28.000.000 − 768.000 = **57.132.000**. Año 0: −140.000.000 − 25.600.000 = **−165.600.000**. Año 5: incluye la recuperación de 28.813.026 de capital de trabajo y el rescate neto 20.000.000 − 0,35 × (20.000.000 − 0) = 13.000.000, para un FCL de **102.578.992**. Elpís obtiene exactamente esas cifras.

| Indicador | Escenario base | Simulación |
|---|---|---|
| VPN | 56.869.730 | Media 66.764.339; desviación 62.172.755 |
| Error estándar de la media | | 621.728 |
| Percentiles 5 y 95 del VPN | | −30.407.469 y 173.769.374 |
| P(VPN < 0) | | 14,35 % |
| TIR | 26,34 % | Mediana 27,43 % |
| TIRM | 20,93 % | |
| Recuperación descontada | 3,90 años | |

Decisión con un límite del 10 %: **Crea valor con riesgo alto**. El precio explica el 59,0 % de la contribución y los clientes el 35,3 %; bastaría una caída del 7,69 % del precio central para anular el VPN base. El VPN esperado supera al del escenario base porque la triangular del precio es asimétrica a la derecha: su media (8.166,67) es mayor que su moda (8.000). Es una lección clásica: evaluar con el valor más probable no equivale a evaluar con el valor esperado.

## 10. Pruebas automáticas (17)

| # | Prueba | Criterio |
|---|---|---|
| 1 | Generador reproducible | Misma semilla, misma secuencia; media de 200.000 uniformes ≈ 0,5 |
| 2 | Inversa de la normal | Φ⁻¹(0,975) = 1,959963984540 y Φ(Φ⁻¹(p)) = p |
| 3 | Inversa de la beta | I(B⁻¹(p)) = p con error < 10⁻¹⁰ |
| 4 | Momentos | Media y desviación muestrales de 6 distribuciones frente a las teóricas |
| 5 | VPN de una anualidad | 137,236031 |
| 6 | TIR | 10 %, 15,2382 % y no existe sin cambio de signo |
| 7 | TIRM y recuperación | 14,0450 % y 1,9167 periodos |
| 8 | Modelo determinista | Desviación cero y VPN exacto |
| 9 | Modelo lineal normal | Media, desviación y P(VPN < 0) frente a la distribución normal exacta |
| 10 | Correlación inducida | Spearman muestral 0,70 ± 0,02 |
| 11 | Matriz no válida | Se detecta y se ajusta a una matriz válida con diagonal unitaria |
| 12 | Contabilidad | Σ ΔKT = 0, depreciación, impuesto sin escudo y rescate neto |
| 13 | Tamaños dinámicos | 1 a 20 periodos y 2 a 14 variables sin valores no finitos |
| 14 | Reproducibilidad | La misma semilla da los mismos VPN |
| 15 | Excel | 9 hojas en orden, menú con 8 enlaces, VPN por fórmula igual al simulado |
| 16 | PDF | Encabezado, cierre, páginas numeradas, marca de agua y aviso de prueba |
| 17 | Marcado de prueba | Prefijo PRUEBA_ solo para datos de prueba |

Las 17 pruebas pasan en Chromium. Además se verificaron Φ⁻¹, Φ, lnΓ y B⁻¹ contra SciPy (diferencias del orden de 10⁻¹⁵).

## 11. Limitaciones

- La correlación se induce con una cópula gaussiana, no con el reordenamiento exacto de Iman y Conover; la correlación de rangos lograda es la pedida en promedio, con error muestral.
- Las variables muestreadas por periodo son independientes entre periodos; no hay autocorrelación ni procesos estocásticos como el movimiento browniano.
- La tasa de descuento, la tasa de impuesto y la periodicidad son deterministas.
- La depreciación es lineal y el capital de trabajo es proporcional a los ingresos.
- El Excel incluye como máximo 10.000 iteraciones con detalle. IRR de Excel puede no converger en flujos poco usuales en los que Elpís sí encuentra una raíz.
- Los resultados dependen por completo de las distribuciones que ingresa el usuario.

## 12. Referencias (APA 7)

Blackman, D., & Vigna, S. (2021). Scaled linear pseudorandom number generators. *ACM Transactions on Mathematical Software, 47*(4), Article 36. https://doi.org/10.1145/3460772

Brealey, R. A., Myers, S. C., & Allen, F. (2020). *Principles of corporate finance* (13th ed.). McGraw-Hill Education.

Glasserman, P. (2003). *Monte Carlo methods in financial engineering*. Springer. https://doi.org/10.1007/978-0-387-21617-1

Hertz, D. B. (1964). Risk analysis in capital investment. *Harvard Business Review, 42*(1), 95–106.

Higham, N. J. (2002). Computing the nearest correlation matrix: A problem from finance. *IMA Journal of Numerical Analysis, 22*(3), 329–343. https://doi.org/10.1093/imanum/22.3.329

Iman, R. L., & Conover, W. J. (1982). A distribution-free approach to inducing rank correlation among input variables. *Communications in Statistics: Simulation and Computation, 11*(3), 311–334. https://doi.org/10.1080/03610918208812265

Kruskal, W. H. (1958). Ordinal measures of association. *Journal of the American Statistical Association, 53*(284), 814–861. https://doi.org/10.1080/01621459.1958.10501481

Law, A. M. (2015). *Simulation modeling and analysis* (5th ed.). McGraw-Hill Education.

Malcolm, D. G., Roseboom, J. H., Clark, C. E., & Fazar, W. (1959). Application of a technique for research and development program evaluation. *Operations Research, 7*(5), 646–669. https://doi.org/10.1287/opre.7.5.646

Metropolis, N., & Ulam, S. (1949). The Monte Carlo method. *Journal of the American Statistical Association, 44*(247), 335–341. https://doi.org/10.1080/01621459.1949.10483310

Press, W. H., Teukolsky, S. A., Vetterling, W. T., & Flannery, B. P. (2007). *Numerical recipes: The art of scientific computing* (3rd ed.). Cambridge University Press.

Vose, D. (2008). *Risk analysis: A quantitative guide* (3rd ed.). Wiley.

Wichura, M. J. (1988). Algorithm AS 241: The percentage points of the normal distribution. *Journal of the Royal Statistical Society. Series C (Applied Statistics), 37*(3), 477–484. https://doi.org/10.2307/2347330
