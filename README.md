# rl-inventario-qlearning
# Gestión de inventario mediante Q-Learning

Proyecto académico de aprendizaje por refuerzo que compara una política de reposición aprendida mediante **Q-Learning** con una política tradicional **(s, S)**, utilizando datos abiertos de ventas.

**Autor:** Ferley Antonio Moreno Cruz  
**Institución:** Universidad de Cundinamarca  
**Programa:** Especialización en Analítica y Ciencia de Datos  
**Asignatura:** Introducción al Machine Learning  
**Actividad:** Semana 14  
**Modalidad:** Individual  

## Problema y objetivo

La reposición de inventario requiere equilibrar la disponibilidad de productos y el costo de mantener existencias. Pedir cantidades insuficientes puede generar faltantes; acumular demasiadas unidades incrementa el almacenamiento.

El objetivo es evaluar qué resultados obtiene Q-Learning frente a una regla tradicional al decidir cuánto inventario solicitar en un entorno simulado.

## Datos abiertos

Se utiliza **Online Retail**, publicado en UCI Machine Learning Repository. El archivo original contiene **541.909 registros** de transacciones de un comercio británico, entre el 1 de diciembre de 2010 y el 9 de diciembre de 2011.

- [Fuente y descarga oficial](https://archive.ics.uci.edu/dataset/352/online+retail).
- [Identificador DOI](https://doi.org/10.24432/C5BW33).
- Licencia de los datos: **Creative Commons Attribution 4.0 International (CC BY 4.0)**.

Se utilizan las variables `InvoiceNo`, `StockCode`, `Description`, `Quantity`, `InvoiceDate` y `UnitPrice`.

**Producto seleccionado:** `22423 — REGENCY CAKESTAND 3 TIER`.

Las cantidades vendidas se utilizan como aproximación de la demanda. Los datos no permiten identificar las ventas que pudieron perderse por falta de existencias.

## Preparación y división temporal

El cuaderno documenta los filtros aplicados a facturas, productos, fechas, cantidades y precios. Las descripciones ausentes no causan exclusión y las filas repetidas se conservan porque no existe un identificador de línea que demuestre que sean errores.

Se excluye la última fecha del archivo por su posible carácter incompleto. Los días sin transacciones se representan como cero ventas registradas.

Los días se distribuyen cronológicamente:

| Periodo | Proporción | Uso |
|---|---:|---|
| Entrenamiento | 60 % | Selección del producto y aprendizaje |
| Validación | 20 % | Ajuste y selección de las políticas |
| Prueba | 20 % restante | Comparación final |

El producto y los parámetros del entorno se definen con entrenamiento. La selección de las políticas utiliza validación antes de consultar los resultados de prueba.

## Entorno simulado

| Componente | Definición |
|---|---|
| Agente | Responsable automático de reposición |
| Estado para Q-Learning | Inventario agrupado en intervalos de 10 unidades, día de la semana y nivel de ventas recientes |
| Acciones | Solicitar 0, 10, 20, 40 u 80 unidades |
| Capacidad | 260 unidades |
| Reposición | Inmediata al comienzo del día y limitada por la capacidad |
| Faltantes | Unidades no atendidas, sin acumulación de pedidos pendientes |
| Recompensa | Negativo del costo operativo diario |

La decisión se toma antes de revelar la demanda del día. El nivel de ventas recientes utiliza únicamente días anteriores disponibles.

### Costos asumidos

- Pedido efectivamente recibido: **2,00** por pedido.
- Almacenamiento: **0,10** por unidad al final del día.
- Faltante: **5,00** por unidad no atendida.

Son **unidades monetarias simuladas**. El costo de adquisición y los ingresos por ventas quedan fuera del objetivo del experimento.

## Algoritmo y comparación

Q-Learning utiliza:

- Tres semillas: `42`, `43` y `44`.
- 2.500 episodios por semilla.
- Ventanas de entrenamiento de 28 días.
- Inventarios iniciales aleatorios durante el entrenamiento.
- Tasa de aprendizaje: `alpha = 0.15`.
- Factor de descuento: `gamma = 0.95`.
- Exploración con descenso lineal de `1.00` a `0.05`.
- Evaluación en validación cada 500 episodios.

Se conserva la mejor versión de cada semilla según su costo de validación. El modelo final también se selecciona por validación.

La política tradicional solicita reposición cuando el inventario alcanza el umbral `s`, intentando aproximarse al nivel `S` mediante las mismas cantidades permitidas al agente. Sus parámetros se seleccionan en validación.

En prueba, ambas políticas comienzan con inventario cero y comparten demanda, costos, capacidad y acciones.

## Ejecución en Google Colab

1. Abre el archivo `.ipynb` del proyecto en [Google Colab](https://colab.research.google.com/).
2. Conecta una sesión de Python; no se requiere GPU.
3. Ejecuta las celdas en orden desde el principio.
4. Espera la finalización de los tres entrenamientos.
5. Revisa las tablas y figuras.
6. Ejecuta las celdas de generación del informe y descarga los archivos.

Las primeras celdas descargan los datos desde UCI. Se necesita conexión a internet y se instalan los complementos indicados en el cuaderno.

**Bibliotecas utilizadas:** NumPy, pandas, Matplotlib, Requests y openpyxl. Para los informes se utilizan ReportLab, PyMuPDF y python-docx.

## Archivos y resultados generados

El cuaderno crea una carpeta `rl_inventario/` en su directorio de trabajo:

| Ruta o archivo | Contenido |
|---|---|
| `data/raw/` | Archivo original descargado |
| `data/processed/` | Ventas diarias separadas por periodo |
| `results/auditoria_limpieza.csv` | Exclusiones aplicadas |
| `results/diseno_datos.json` | Producto y división temporal |
| `results/configuracion_entorno.json` | Supuestos y parámetros del simulador |
| `results/seleccion_modelo.json` | Política seleccionada e hiperparámetros |
| `results/comparacion_prueba.csv` | Métricas de todas las políticas evaluadas |
| `results/curva_validacion.csv` | Evolución del costo de validación |
| `results/q_seed_*.npy` | Tablas Q seleccionadas |
| `results/*_prueba.csv` | Trayectorias diarias de prueba |
| `results/figures/` | Figuras del experimento |
| `Informe_Q_Learning_APA.pdf` | Informe técnico |
| `Informe_Q_Learning_APA.docx` | Versión editable del informe |

Las métricas principales son costo total, unidades faltantes, inventario promedio, porcentaje de unidades atendidas y número de pedidos. Los valores y la interpretación se consultan en el cuaderno ejecutado y en el informe; no se presupone que Q-Learning supere a la política tradicional.

## Comprobaciones

El cuaderno incluye un ejemplo controlado para comprobar:

- Balance del inventario.
- Costos y recompensas.
- Límite de capacidad.
- Finalización de episodios.
- Estado inicial independiente de la demanda actual y futura.

## Limitaciones

El estudio considera un solo producto y un periodo histórico. Las ventas son una aproximación de demanda; los costos son supuestos y la reposición inmediata simplifica el abastecimiento.

La agregación del estado pierde información y no garantiza representar todos los factores que influyen en la demanda. El entrenamiento finito no demuestra convergencia ni optimalidad. Los resultados corresponden al simulador y no demuestran efectos reales sobre el comercio.

## Referencias

Chen, D. (2015). *Online Retail* [Conjunto de datos]. UCI Machine Learning Repository. https://doi.org/10.24432/C5BW33

Sutton, R. S., & Barto, A. G. (2018). *Reinforcement learning: An introduction* (2nd ed.). MIT Press. https://web.stanford.edu/class/psych209/Readings/SuttonBartoIPRLBook2ndEd.pdf

