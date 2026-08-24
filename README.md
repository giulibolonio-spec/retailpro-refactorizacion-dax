# retailpro-refactorizacion-dax
Refactorización y validación de medidas DAX con variables en Power BI.
Refactorización de medidas DAX en Power BI
Descripción

Proyecto desarrollado para RetailPro con el objetivo de transformar medidas DAX funcionales, pero repetitivas y difíciles de mantener, en expresiones profesionales mediante el uso de variables VAR y RETURN.

La refactorización conserva exactamente la lógica y los resultados originales, mejorando la legibilidad, la mantenibilidad y la capacidad de depuración del código.

Dataset

Se utilizó el dataset Sample Superstore, que contiene información histórica de ventas, ganancias, clientes, productos y regiones.

Durante la preparación se configuraron correctamente los tipos de datos según la configuración regional de origen:

Order Date y Ship Date: fecha.
Sales, Profit y Discount: número decimal.
Quantity: número entero.
Modelo de datos

Se creó una dimensión de fechas denominada dim_fechas, con fechas continuas para todos los años del dataset.

La relación principal del modelo es:

dim_fechas[Date] (1) → Ventas[Order Date] (*)

La relación se configuró como activa, con cardinalidad 1:N y dirección de filtro única desde dim_fechas hacia Ventas.

Las medidas se centralizaron en la tabla _Medidas.

1. Crecimiento anual
Problemas identificados

La medida original calcula las ventas del año anterior dos veces: una en el numerador y otra en el denominador. Si cambiara la lógica temporal, sería necesario modificarla en más de un lugar, aumentando el riesgo de inconsistencias.

Las variables propuestas son:

VentasActuales
VentasAnoAnterior
Versión original
Crecimiento Anual =
DIVIDE(
    SUM(Ventas[Sales])
        - CALCULATE(
            SUM(Ventas[Sales]),
            PREVIOUSYEAR(dim_fechas[Date])
        ),
    CALCULATE(
        SUM(Ventas[Sales]),
        PREVIOUSYEAR(dim_fechas[Date])
    )
)
Versión refactorizada
Crecimiento Anual_v2 =
VAR VentasActuales =
    SUM(Ventas[Sales])
VAR VentasAnoAnterior =
    CALCULATE(
        SUM(Ventas[Sales]),
        PREVIOUSYEAR(dim_fechas[Date])
    )

RETURN
    DIVIDE(
        VentasActuales - VentasAnoAnterior,
        VentasAnoAnterior
    )
Mejora realizada

La versión _v2 calcula las ventas del año anterior una sola vez y reutiliza el resultado. Los nombres de las variables permiten comprender inmediatamente qué representa cada valor.

2. Margen de ganancia
Problemas identificados

La versión original concentra toda la lógica en una sola expresión. Aunque no presenta una repetición importante, resulta menos clara y sería más difícil de extender si se incorporaran nuevos componentes al cálculo.

Las variables propuestas son:

GananciaTotal
VentasTotales
MargenCalculado
Versión original
Margen % =
DIVIDE(
    SUM(Ventas[Profit]),
    SUM(Ventas[Sales])
) * 100
Versión refactorizada
Margen %_v2 =
VAR GananciaTotal =
    SUM(Ventas[Profit])
VAR VentasTotales =
    SUM(Ventas[Sales])
VAR MargenCalculado =
    DIVIDE(
        GananciaTotal,
        VentasTotales
    ) * 100

RETURN
    MargenCalculado
Mejora realizada

La versión _v2 separa los componentes del cálculo y asigna nombres descriptivos a los resultados intermedios. Se conservó la multiplicación por 100 para devolver exactamente el mismo valor que la medida original.

Ambas medidas se configuraron como número decimal con dos decimales.

3. Clasificación de rendimiento
Problemas identificados

La medida original calcula el margen completo dos veces y utiliza varios IF anidados. Además, los límites de clasificación están insertados directamente dentro de las condiciones, dificultando su modificación.

Las variables propuestas son:

MargenActual
UmbralAlto
UmbralMedio
Versión original
Clasificacion Rendimiento =
IF(
    DIVIDE(
        SUM(Ventas[Profit]),
        SUM(Ventas[Sales])
    ) * 100 >= 20,
    "Alto",
    IF(
        DIVIDE(
            SUM(Ventas[Profit]),
            SUM(Ventas[Sales])
        ) * 100 >= 10,
        "Medio",
        "Bajo"
    )
)
Versión refactorizada
Clasificacion Rendimiento_v2 =
VAR MargenActual =
    DIVIDE(
        SUM(Ventas[Profit]),
        SUM(Ventas[Sales])
    ) * 100
VAR UmbralAlto =
    20
VAR UmbralMedio =
    10

RETURN
    SWITCH(
        TRUE(),
        MargenActual >= UmbralAlto, "Alto",
        MargenActual >= UmbralMedio, "Medio",
        "Bajo"
    )
Mejora realizada

El margen se calcula una sola vez y los umbrales quedan identificados mediante variables descriptivas. SWITCH(TRUE(), ...) presenta las reglas en orden y facilita futuros cambios en los límites del negocio.

Verificación de resultados

Se creó una matriz con dim_fechas[Año] en las filas y las seis medidas en los valores.




Los resultados originales y refactorizados coincidieron en todos los años:

Año	Crecimiento original y v2	Margen original y v2	Clasificación original y v2
2014	Sin año anterior	10,23	Medio
2015	-2,83 %	13,10	Medio
2016	29,47 %	13,43	Medio
2017	20,36 %	12,74	Medio

La comparación confirma que la refactorización mejoró la estructura del código sin modificar los resultados.

Análisis con DAX Studio

DAX Studio no fue utilizado en esta práctica. La función Server Timings permitiría medir:

El tiempo total de ejecución de una consulta.
El tiempo utilizado por el Formula Engine.
El tiempo utilizado por el Storage Engine.
La cantidad de consultas enviadas al motor de almacenamiento.
La existencia de posibles cuellos de botella.

Es importante medir antes de optimizar porque una fórmula extensa no necesariamente es lenta y una medida aparentemente sencilla puede estar afectada por el modelo, la cardinalidad o el contexto de filtros. La medición permite identificar el problema real antes de realizar cambios innecesarios.

Conclusión

Las variables permitieron eliminar cálculos repetidos, asignar nombres claros a los resultados intermedios y simplificar la modificación de las reglas de negocio.

Las tres medidas refactorizadas devuelven exactamente los mismos resultados que sus versiones originales. El modelo conserva las medidas anteriores para facilitar la validación y centraliza todas las expresiones dentro de _Medidas.

Archivos del repositorio
retailpro-refactorizacion-dax/
├── README.md
├── RetailPro_Refactorizacion_DAX.pbix
└── evidencias/
    ├── README.md
    └── matriz_verificacion.png
