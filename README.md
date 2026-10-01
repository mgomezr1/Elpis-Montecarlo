# Elpís

Aplicación web académica de **simulación Monte Carlo para evaluar proyectos de inversión**: construya el flujo de caja libre con variables inciertas, simule miles de futuros posibles y obtenga la distribución del VPN, la TIR y la TIRM, la probabilidad de que el proyecto destruya valor y qué tendría que cambiar para que la recomendación fuera otra.

**Abrir la aplicación:** https://mgomezr1.github.io/Elpis-Montecarlo/

No requiere instalación ni registro. Funciona en cualquier navegador moderno, en computador, tableta o teléfono.

## Qué permite hacer

- Definir las variables que generan el flujo (precio, cantidad, costos, inversión, rescate) con distribución fija, uniforme, triangular, PERT, normal, lognormal o discreta.
- Elegir para cada variable si se muestrea una vez por iteración o una vez por periodo, y si crece a una tasa fija o según otra variable.
- Incluir o no correlación de rangos entre las variables, con ajuste automático a la matriz válida más cercana.
- Armar el flujo de caja libre con ingresos, costos variables y fijos, CAPEX con depreciación, capital de trabajo, impuestos con o sin escudo fiscal y valor de rescate.
- Descontar con una tasa directa o con el WACC, con Ke directo o por CAPM.
- Ver la decisión, el VPN esperado, la probabilidad de VPN negativo, los percentiles, el flujo por periodo y la convergencia.
- Analizar la sensibilidad con tornado, puntos de equilibrio y efecto de la tasa de descuento.
- Guardar el modelo, descargar un Excel con fórmulas y un informe ejecutivo en PDF.

## Cómo usarla

1. Pruebe la portada: mueva el número de futuros y la incertidumbre y vea cómo la simulación se acerca a la probabilidad exacta.
2. En **Modelo**, escriba el nombre, el horizonte, la tasa, el impuesto, las iteraciones y su límite de riesgo; confirme.
3. En **Variables y flujo**, agregue las variables y las líneas del flujo; revise el escenario base.
4. En **Simulación**, ejecute y lea la decisión y la distribución del VPN.
5. En **Sensibilidad**, vea qué variables pesan más y qué tendría que cambiar.
6. Descargue el **Excel** o el **PDF**.

Los **escenarios de prueba** (en «Acerca de Elpís») cargan modelos ficticios de tres tamaños para practicar; quedan marcados como datos de prueba y sus archivos llevan el prefijo PRUEBA_.

## Privacidad

Todos los cálculos se hacen en su navegador. Los datos no se envían a ningún servidor. Use «Guardar modelo» para conservar su trabajo en un archivo JSON.

## Requisitos

- Navegador actualizado (Chrome, Edge, Firefox o Safari) con JavaScript activo.
- Conexión a internet solo para exportar a Excel, porque la aplicación descarga la biblioteca SheetJS. La simulación y el PDF funcionan sin conexión.

## Documentación

La guía completa (método, fórmulas, funciones, ejemplo verificado, pruebas y limitaciones) está en [docs/Guia_Elpis.md](docs/Guia_Elpis.md).

## Cómo citar

Gómez Rueda, M. S. (2026). *Elpís: Simulación Monte Carlo para evaluar proyectos de inversión* (Versión 1.0) [Software]. https://mgomezr1.github.io/elpis/

## Autoría y uso

Este aplicativo fue desarrollado por Mario Sergio Gómez Rueda. Su uso es de carácter académico y cualquier otro uso se regirá por el derecho de la propiedad intelectual. Consulte los términos en [LICENSE.md](LICENSE.md). Sugerencias o dudas: mgomezr1@gmail.com

## Fundamento metodológico

- Metropolis, N., & Ulam, S. (1949). The Monte Carlo method. *Journal of the American Statistical Association, 44*(247), 335–341. https://doi.org/10.1080/01621459.1949.10483310
- Hertz, D. B. (1964). Risk analysis in capital investment. *Harvard Business Review, 42*(1), 95–106.
- Vose, D. (2008). *Risk analysis: A quantitative guide* (3rd ed.). Wiley.
- Iman, R. L., & Conover, W. J. (1982). A distribution-free approach to inducing rank correlation among input variables. *Communications in Statistics: Simulation and Computation, 11*(3), 311–334. https://doi.org/10.1080/03610918208812265

## Componentes de terceros

La exportación a Excel usa [SheetJS Community Edition](https://sheetjs.com), distribuida bajo la licencia Apache 2.0 y cargada desde cdn.sheetjs.com.
