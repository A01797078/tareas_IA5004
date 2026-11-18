# Experimentos y decisiones de diseño

Este documento registra los experimentos realizados con el simulador del capstone de Distribuidora Cuauhtémoc, sus resultados, hallazgos, limitaciones y las decisiones de diseño que surgen de su análisis.

El objetivo es mantener la trazabilidad entre las configuraciones, las ejecuciones y las conclusiones del equipo, además de facilitar la reproducción de los experimentos y la preparación de la presentación final.

Se distingue entre:

- **Evidencia:** información obtenida directamente de las ejecuciones o del código y los archivos de configuración.
- **Interpretación:** análisis realizado por el equipo a partir de esa evidencia.
- **Hipótesis:** supuesto que aún debe comprobarse o confirmarse con el responsable correspondiente.
- **Decisión de diseño:** modificación o característica que el equipo propone, adopta o descarta.

---

# 1. Experimentos

## E1 — Inventario suficiente

### 1.1 Objetivo

Ejecutar el escenario E1 con la configuración original del notebook para comprender el flujo de comunicación de los cinco agentes, el uso de herramientas, la integración de recomendaciones y el comportamiento de los controles y del supervisor.

Esta ejecución establece una referencia inicial para los experimentos posteriores.

### 1.2 Configuración

No se realizaron modificaciones en las instrucciones de los agentes, datos, herramientas ni lógica del simulador.

| Parámetro | Valor |
|---|---|
| Escenario | E1 |
| Criterio | `cobertura_equilibrada` |
| Versión de datos | `datos_1.0` |
| Versión del motor | `capstone_base_1.5` |
| Horizonte | 3 días |
| Inventario del centro | 160 unidades |
| Consultas realizadas | 6 |
| Tokens de entrada | 4,982 |
| Tokens de salida | 2,440 |
| Resultado final | `requiere_correccion` |

**Pregunta original:**

¿Qué resultado obtendré de ejecutar un flujo con inventario suficiente?

**Predicción original:**

El resultado no requerirá resurtir el producto en ninguna tienda.

**Cambio declarado:**

Referencia inicial.

### 1.3 Análisis del experimento

#### Interpretación del escenario

La predicción inicial fue incorrecta debido a una interpretación distinta del concepto de inventario suficiente.

El escenario E1 no establece que las tiendas tengan existencias suficientes para satisfacer su demanda. Establece que el centro de distribución dispone de unidades suficientes para cubrir las necesidades de reposición estimadas de las tres tiendas.

Las necesidades se calculan considerando las existencias actuales y la demanda estimada para un horizonte de tres días.

#### Necesidades y distribución

| Tienda | Existencias | Demanda estimada | Necesidad | Capacidad de envío | Envío propuesto |
|---|---:|---:|---:|---:|---:|
| T01 - Centro | 15 | 75 | 60 | 70 | 60 |
| T02 - Norte | 20 | 60 | 40 | 60 | 40 |
| T03 - Sur | 20 | 45 | 25 | 50 | 25 |
| **Total** | **55** | **180** | **125** | **180** | **125** |

El orquestador propuso distribuir 125 unidades de las 160 disponibles en el centro de distribución, dejando 35 unidades sin asignar.

La propuesta permite alcanzar una cobertura estimada del 100 % de la demanda de las tres tiendas durante el horizonte definido.

#### Comportamiento de los especialistas

**Inventario:** identificó correctamente las necesidades de reposición de las tres tiendas y una necesidad total de 125 unidades.

**Logística:** identificó los límites de capacidad, pero propuso como referencia distribuir 54, 53 y 53 unidades entre las tiendas. Esta distribución excedería las necesidades estimadas de T02 y T03, por lo que no sería válida como propuesta final.

**Comercial:** recomendó atender primero T02, después T01 y finalmente T03, siguiendo sus prioridades de servicio. Sin embargo, no se encontró una regla explícita que indique cómo debe utilizarse esta prioridad al aplicar el criterio `cobertura_equilibrada`.

**Orquestador:** integró los análisis y descartó explícitamente la propuesta logística por exceder las necesidades de dos tiendas. Propuso distribuir únicamente las unidades necesarias para cubrir la demanda estimada.

Este comportamiento constituye evidencia de que el orquestador identificó y resolvió una discrepancia entre recomendaciones de los especialistas.

#### Validaciones y supervisión

Las comprobaciones deterministas de Python fueron satisfactorias:

- Se incluyeron las tres tiendas sin duplicados.
- Las cantidades propuestas fueron enteras y no negativas.
- El total enviado no superó el inventario disponible.
- Ningún envío excedió la capacidad o necesidad de su tienda.

`comprobaciones.conforme = true`

A pesar de ello, el supervisor emitió el dictamen `requiere_correccion`, principalmente por dos observaciones:

1. No se definió explícitamente qué hacer con las 35 unidades restantes en el centro.
2. No existe una regla operacional verificable para `cobertura_equilibrada` ni para su relación con `prioridad_servicio`.

**Interpretación del equipo (preliminar):**

La propuesta es factible según las restricciones programadas. El inventario remanente no representa por sí mismo una violación, ya que las reglas permiten enviar una cantidad menor o igual al inventario disponible y prohíben exceder las necesidades de las tiendas.

Sin embargo, la falta de una definición operacional de los criterios de negocio sí constituye una ambigüedad que debe resolverse antes de evaluar escenarios con inventario insuficiente.

El dictamen del supervisor es una salida del modelo y no demuestra, por sí solo, que la propuesta incumpla las restricciones obligatorias.

### 1.4 Hallazgos y limitaciones

| ID | Hallazgo | Tipo | Implicación |
|---|---|---|---|
| H-E1-01 | Inventario suficiente se refiere a disponibilidad en el centro, no a existencias suficientes en las tiendas. | Interpretación del escenario | Ajustar el entendimiento de los escenarios antes de formular hipótesis. |
| H-E1-02 | Logística propuso cantidades superiores a las necesidades de dos tiendas. | Comportamiento del agente | Evaluar si las instrucciones deben enfatizar la restricción de necesidad. |
| H-E1-03 | El orquestador descartó una recomendación que excedía necesidades. | Comportamiento satisfactorio | Conservar como evidencia de coordinación. |
| H-E1-04 | No existe una definición operacional completa de los dos criterios de asignación. | Ambigüedad de especificación | Establecer hipótesis de negocio sujetas a confirmación. |
| H-E1-05 | El supervisor solicitó corregir una propuesta que superó las validaciones de Python. | Discrepancia entre evaluaciones | Revisar las condiciones de emisión de los dictámenes. |
| H-E1-06 | El tratamiento del inventario remanente no está descrito explícitamente. | Ambigüedad de especificación | Aclarar si basta con dejarlo sin asignar o si se requiere otra política. |
| H-E1-07 | No hay condiciones suficientemente explícitas para distinguir los tres dictámenes del supervisor. | Ambigüedad de supervisión | Proponer criterios de clasificación y evaluarlos. |

**Limitaciones de la ejecución:**

- Solo se ejecutó una vez la configuración original de E1.
- No se evaluó la estabilidad de las respuestas ante ejecuciones independientes.
- E1 no permite comparar el efecto de los criterios de prioridad cuando existe escasez, ya que todas las necesidades pueden cubrirse.
- El cumplimiento de las restricciones programadas no demuestra que la propuesta sea óptima para el negocio.
- No se realizaron movimientos reales de inventario ni se midieron resultados operativos.

### 1.5 Conclusión del experimento

La ejecución inicial permitió comprender el funcionamiento general del simulador y observar cómo los especialistas, el orquestador, Python y el supervisor participan en la elaboración y evaluación de una propuesta.

El sistema produjo una distribución que cubre el 100 % de la demanda estimada y satisface las restricciones numéricas implementadas. El orquestador resolvió adecuadamente una discrepancia con la recomendación logística.

Sin embargo, se identificaron ambigüedades en las definiciones de los criterios de asignación y supervisión. Estas ambigüedades limitan la interpretación del resultado y justifican realizar experimentos posteriores con reglas más explícitas.

No se considera necesario modificar todavía el motor del simulador. Se propone continuar con la definición provisional y evaluación de criterios de negocio y supervisión, dejando su confirmación definitiva pendiente de un responsable autorizado.

### 1.6 Referencia de ejecución

[E1_intento_001_57a980ee06f9](./historial/intento_001_57a980ee06f9.json)

Identificador: `57a980ee06f9`

---

## E1-B — Criterios explícitos del supervisor

### 1.1 Objetivo

Evaluar si definir explícitamente los criterios de clasificación del supervisor permite distinguir mejor entre incumplimientos obligatorios, omisiones corregibles y decisiones que requieren intervención humana.

Se utiliza el escenario E1 para comparar el comportamiento del supervisor con la configuración original y con instrucciones modificadas, manteniendo los mismos datos y restricciones del simulador.

### 1.2 Configuración

Se modificaron únicamente las instrucciones del supervisor, incorporando definiciones explícitas para los tres dictámenes posibles:

- `conforme_para_revision_humana`: propuesta que cumple las restricciones obligatorias y cuenta con evidencia suficiente para revisión humana, aunque existan limitaciones no bloqueantes.
- `requiere_correccion`: errores, inconsistencias u omisiones corregibles con los datos y reglas autorizados.
- `escalar`: ambigüedades de negocio, conflictos de autoridad o decisiones que requieren una definición humana antes de continuar.

Adicionalmente, se indicó al supervisor que no debe considerar obligatorio utilizar todo el inventario disponible ni convertir preferencias de negocio en restricciones obligatorias.

No se modificaron las instrucciones de los otros cuatro agentes, los datos, las herramientas ni la lógica del simulador.

| Parámetro | Valor |
|---|---|
| Escenario | E1 |
| Criterio | `cobertura_equilibrada` |
| Versión de datos | `datos_1.0` |
| Versión del motor | `capstone_base_1.5` |
| Horizonte | 3 días |
| Inventario del centro | 160 unidades |
| Consultas realizadas | 6 |
| Tokens de entrada | 5,275 |
| Tokens de salida | 2,635 |
| Resultado final | `propuesta_para_revision_humana` |

**Pregunta original:**

¿Cómo afecta la definición explícita de criterios de clasificación al dictamen y la justificación del supervisor?

**Predicción original:**

El supervisor distinguirá mejor entre incumplimientos, omisiones corregibles y decisiones que requieren autoridad humana.

**Cambio declarado:**

Únicamente las instrucciones del supervisor.

### 1.3 Análisis del experimento

#### Comparación de las propuestas

En ambas ejecuciones, el orquestador propuso distribuir 125 unidades entre las tres tiendas, conservando 35 unidades en el centro de distribución.

| Tienda | E1 - Configuración original | E1-B - Supervisor modificado |
|---|---:|---:|
| T01 - Centro | 60 | 60 |
| T02 - Norte | 40 | 40 |
| T03 - Sur | 25 | 25 |
| **Total enviado** | **125** | **125** |
| **Inventario remanente** | **35** | **35** |

Las dos propuestas permiten alcanzar una cobertura estimada del 100 % de la demanda durante el horizonte de tres días.

Aunque las instrucciones del orquestador no cambiaron, su explicación en la segunda ejecución señaló explícitamente que las 35 unidades restantes no se distribuyen porque hacerlo excedería las necesidades registradas.

Las validaciones deterministas de Python fueron satisfactorias en ambos intentos.

#### Comportamiento de los especialistas y del orquestador

**Inventario:** identificó nuevamente una necesidad total de 125 unidades, distribuida en 60 para T01, 40 para T02 y 25 para T03.

**Logística:** volvió a proponer una distribución orientativa de 54, 53 y 53 unidades para utilizar las 160 disponibles en el centro. La propuesta excede las necesidades estimadas de T02 y T03, así como la capacidad máxima de envío de T03.

Esta observación resulta relevante porque el agente de Logística dispone de información sobre el inventario central y las capacidades, pero no necesariamente de las necesidades calculadas por Inventario. Por ello, exceder necesidades puede explicarse por información parcial, mientras que exceder la capacidad de T03 representa una inconsistencia respecto a los datos que sí tiene disponibles.

**Comercial:** mantuvo el orden de atención T02, T01 y T03, siguiendo las prioridades de servicio. La interpretación operacional de `cobertura_equilibrada` continuó sin estar definida.

**Orquestador:** propuso nuevamente una distribución ajustada a las necesidades de las tiendas. No incorporó la distribución orientativa de Logística y justificó la existencia de inventario remanente.

La distribución final permaneció sin cambios, aunque las respuestas de los agentes presentaron diferencias de redacción y argumentación respecto a E1.

#### Comparación de los dictámenes del supervisor

| Elemento | E1 - Original | E1-B - Modificado |
|---|---|---|
| Validaciones Python | Conformes | Conformes |
| Dictamen del supervisor | `requiere_correccion` | `conforme_para_revision_humana` |
| Inventario remanente | Considerado un pendiente que requería corrección | Reconocido como permitido por las restricciones |
| Criterio de distribución | Falta de definición considerada motivo de corrección | Limitación pendiente de confirmación |
| Revisión humana | No se presentó como conforme | Propuesta disponible para revisión humana |

En la ejecución original, el supervisor consideró que la falta de definición del inventario remanente y del criterio `cobertura_equilibrada` impedía aceptar la propuesta para revisión humana.

En la segunda ejecución, el supervisor reconoció que:

- Las cantidades propuestas cumplen las restricciones programadas.
- Las 35 unidades sin asignar no representan un incumplimiento.
- La cobertura estimada de las tres tiendas es del 100 %.
- La definición operacional del criterio y las restricciones logísticas no modeladas siguen pendientes de confirmación.
- Estas limitaciones no impiden presentar la propuesta para revisión humana bajo las condiciones del escenario.

**Interpretación preliminar del equipo:**

El cambio observado sugiere que las instrucciones explícitas ayudaron al supervisor a distinguir entre restricciones obligatorias y asuntos pendientes de negocio.

El nuevo dictamen resulta más coherente con las validaciones del programa y con el alcance de una propuesta sujeta a revisión humana.

Sin embargo, no puede afirmarse que el cambio se deba exclusivamente a las instrucciones modificadas, ya que se ejecutó nuevamente todo el flujo y las respuestas de los demás agentes también presentaron variaciones.

#### Consumo de tokens

| Métrica | E1 - Original | E1-B - Modificado |
|---|---:|---:|
| Tokens de entrada | 4,982 | 5,275 |
| Tokens de salida | 2,440 | 2,635 |
| **Total** | **7,422** | **7,910** |
| Consultas | 6 | 6 |

La segunda ejecución consumió 488 tokens adicionales, aproximadamente un 6.6 % más que la primera.

Este incremento corresponde a las ejecuciones observadas y no permite establecer todavía un costo promedio ni afirmar que las instrucciones modificadas siempre incrementarán el consumo.

### 1.4 Hallazgos y limitaciones

| ID | Hallazgo | Tipo | Implicación |
|---|---|---|---|
| H-E1B-01 | El supervisor cambió su dictamen de `requiere_correccion` a `conforme_para_revision_humana`. | Comportamiento del agente | Las instrucciones explícitas pueden contribuir a una clasificación más alineada con los criterios propuestos. |
| H-E1B-02 | El supervisor distinguió el cumplimiento de restricciones de las limitaciones pendientes de validación. | Comportamiento satisfactorio | Conservar la distinción como parte de los criterios de supervisión propuestos. |
| H-E1B-03 | La distribución integrada fue idéntica en ambas ejecuciones. | Resultado de comparación | El cambio de dictamen no estuvo acompañado de un cambio cuantitativo en el plan. |
| H-E1B-04 | Logística volvió a proponer una asignación que supera límites de capacidad y necesidad. | Comportamiento del agente | Revisar si su responsabilidad debe limitarse a identificar restricciones, sin proponer cantidades de distribución. |
| H-E1B-05 | La interpretación operacional de los criterios de distribución continúa pendiente. | Ambigüedad de especificación | Mantener la necesidad de definir y validar las políticas de negocio. |
| H-E1B-06 | El consumo total aumentó un 6.6 % respecto a la primera ejecución. | Observación de consumo | Considerar el costo de las instrucciones adicionales, sin generalizar el resultado. |

**Limitaciones de la comparación:**

- Se realizó una ejecución por configuración, por lo que no se evaluó la estabilidad del comportamiento.
- Aunque solo se modificaron las instrucciones del supervisor, todo el flujo se ejecutó nuevamente y las respuestas de otros agentes variaron.
- No se establecieron todavía definiciones operacionales para `cobertura_equilibrada` y `prioridad_servicio`.
- El escenario E1 no permite evaluar conflictos reales entre prioridades cuando el inventario es insuficiente.
- El cambio en el dictamen no demuestra que el supervisor clasifique correctamente otros escenarios, especialmente aquellos con incumplimientos o riesgos.
- El resultado `conforme_para_revision_humana` no constituye una autorización para ejecutar movimientos de inventario.

### 1.5 Conclusión del experimento

La segunda ejecución mantuvo la misma distribución y las validaciones deterministas conformes de E1, pero produjo un dictamen diferente del supervisor.

Con las instrucciones modificadas, el supervisor distinguió entre el cumplimiento de las restricciones obligatorias y los aspectos pendientes de confirmación por el negocio. Esto permitió clasificar la propuesta como `conforme_para_revision_humana`, conservando las observaciones sobre las políticas no definidas y las limitaciones del escenario.

El resultado respalda de manera preliminar la hipótesis de que establecer criterios explícitos mejora la justificación y clasificación de las propuestas. Sin embargo, una comparación de dos ejecuciones independientes no permite demostrar que el comportamiento será consistente ni atribuir todas las diferencias exclusivamente al cambio realizado.

Se propone conservar provisionalmente las nuevas instrucciones del supervisor para los siguientes experimentos, sujetas a evaluación en escenarios de escasez y riesgo.

Adicionalmente, se mantiene como hallazgo pendiente la tendencia de Logística a proponer distribuciones cuantitativas que exceden algunas restricciones. Antes de modificar sus instrucciones, conviene observar su comportamiento en escenarios con inventario insuficiente.

### 1.6 Referencia de ejecución

**Referencia inicial:**

[E1_intento_001_57a980ee06f9](./historial/intento_001_57a980ee06f9.json)

**Ejecución con supervisor modificado:**

[E1B_intento_002_d4950f7838a6](./historial/intento_002_d4950f7838a6.json)

Identificador: `d4950f7838a6`

**Decisiones relacionadas:** DD-02 — Criterios de clasificación del supervisor.

**Hallazgos anteriores relacionados:** H-E1-05 y H-E1-07.


---

# 2. Decisiones de diseño

Esta sección registra las decisiones identificadas durante el análisis y ejecución del prototipo.

Cada decisión conserva su justificación, trazabilidad y estado, incluso si posteriormente se descarta.

**Estados:**

- 🟡 **Propuesta:** identificada, pero todavía no adoptada o probada.
- ✅ **Adoptada:** aceptada por el equipo y, cuando corresponda, implementada.
- ❌ **Descartada:** evaluada y rechazada, conservando su justificación.
- 🔵 **Pendiente de validación externa:** requiere confirmación de un responsable del negocio o docente.

Las decisiones de diseño deben distinguirse de las políticas empresariales: una hipótesis del equipo no equivale a un requerimiento confirmado.

## DD-00 — Utilidad para configuración reproducible de experimentos

**Estado:** 🟡 Propuesta

**Motivación:**

Facilitar la ejecución y reproducción de experimentos por distintos integrantes del equipo, evitando modificaciones manuales de variables entre corridas.

**Propuesta:**

Crear una utilidad que permita seleccionar un experimento y cargar sus parámetros e instrucciones, partiendo siempre de una configuración base conocida.

**Alcance:**

- Centralizar las configuraciones de los experimentos.
- Evitar la acumulación accidental de modificaciones entre corridas.
- Mantener los mecanismos existentes de creación de intentos y registro de evidencia.
- No modificar la lógica de negocio, las herramientas ni el flujo de los agentes.

**Justificación:**

La utilidad busca mejorar la reproducibilidad de las configuraciones y la colaboración del equipo. No garantiza respuestas idénticas del modelo en ejecuciones independientes.

**Trazabilidad:** necesidad identificada durante la preparación del plan de experimentación.

**Validación pendiente:** implementar y comprobar que las configuraciones seleccionadas corresponden a las registradas en cada intento.

## DD-01 — Definición operacional de criterios de asignación

**Estado:** 🟡 Propuesta / 🔵 Pendiente de validación externa

**Hallazgo de origen:** H-E1-04

**Problema:**

La configuración permite seleccionar `cobertura_equilibrada` o `prioridad_servicio`, pero no establece un procedimiento completo para determinar la distribución correspondiente a cada criterio.

Esto permite que diferentes agentes interpreten una misma preferencia de formas distintas.

**Hipótesis de diseño:**

Definir provisionalmente:

- `cobertura_equilibrada`: buscar una distribución que reduzca las diferencias de cobertura entre las tiendas, respetando todas las restricciones obligatorias.
- `prioridad_servicio`: favorecer a las tiendas con mayor prioridad de servicio, donde el valor 1 representa la prioridad más alta, sin exceder necesidades, capacidades ni inventario disponible.

Estas definiciones son hipótesis iniciales. Todavía debe establecerse un procedimiento concreto para resolver empates y determinar las cantidades asignadas cuando el inventario es insuficiente.

**Justificación:**

Las reglas explícitas permitirían reducir diferencias de interpretación y evaluar las recomendaciones con criterios verificables.

**Validación pendiente:**

- Consultar al profesor si el ejercicio espera que el equipo proponga estas definiciones.
- Confirmar con el responsable del negocio las políticas aplicables antes de recomendar un piloto operativo.
- Comparar los resultados de escenarios con escasez utilizando las definiciones propuestas.

**Trazabilidad:** E1, H-E1-04.

## DD-02 — Criterios de clasificación del supervisor

**Estado:** ✅ Adoptada provisionalmente para experimentación / 🔵 Pendiente de validación externa

**Hallazgos de origen:** H-E1-05, H-E1-07

**Problema:**

El supervisor puede emitir tres dictámenes, pero sus condiciones de uso no están definidas de forma suficientemente precisa.

En E1 solicitó corrección a pesar de que la propuesta cumplió todas las comprobaciones deterministas.

**Hipótesis de diseño:**

| Dictamen | Condición propuesta |
|---|---|
| `conforme_para_revision_humana` | Propuesta factible, con evidencia y justificación suficientes para revisión humana. |
| `requiere_correccion` | Omisiones o inconsistencias corregibles dentro del alcance y las reglas ya establecidas. |
| `escalar` | Ambigüedad de política, conflicto de autoridad o condición que requiere una decisión humana adicional antes de continuar. |

Estos criterios no sustituyen los controles deterministas. Una propuesta con restricciones obligatorias incumplidas debe seguir siendo rechazada por Python.

**Justificación:**

Una separación explícita entre errores corregibles y decisiones que requieren autoridad externa permitiría interpretar mejor los dictámenes y evitar correcciones innecesarias.

**Validación pendiente:**

- Evaluar si las nuevas instrucciones modifican el dictamen del supervisor ante un caso equivalente.
- Analizar si el nuevo dictamen está justificado por la evidencia.
- Confirmar las condiciones de escalamiento antes de considerar una aplicación operativa.

**Alcance:**

No se propone desarrollar un proceso automático de escalamiento. El motor ya reconoce `escalar` como `revision_humana_requerida`; la gestión posterior queda fuera del alcance del prototipo.

**Trazabilidad:** E1, H-E1-05 y H-E1-07.

## DD-03 — Tratamiento del inventario remanente

**Estado:** 🟡 Propuesta / 🔵 Pendiente de validación externa

**Hallazgo de origen:** H-E1-06

**Problema:**

El supervisor observó que no se especificaba qué hacer con las 35 unidades restantes, aunque la distribución respetó las restricciones programadas.

**Hipótesis de diseño:**

El inventario del centro que no sea necesario o no pueda asignarse sin superar los límites permanecerá sin asignar.

La propuesta debe indicar la cantidad remanente para facilitar su revisión, sin asumir un destino posterior ni ejecutar movimientos adicionales.

**Justificación:**

Permite explicar el resultado sin introducir una obligación de distribuir todo el inventario disponible.

**Validación pendiente:**

Confirmar si la política del negocio exige una regla adicional para gestionar inventario remanente.

**Trazabilidad:** E1, H-E1-06.

---

## E2 — Escasez con cobertura equilibrada

### 1.1 Objetivo

Evaluar cómo distribuyen los agentes el inventario disponible cuando este resulta insuficiente para cubrir las necesidades de las tres tiendas, utilizando el criterio `cobertura_equilibrada`.

Se busca observar cómo interpretan este criterio sin una definición operacional explícita, cómo coordinan sus recomendaciones y si la propuesta resultante cumple las restricciones y es consistente con el objetivo de equilibrar la cobertura de demanda.

### 1.2 Configuración

Se utilizó el escenario E2 con las instrucciones originales de los especialistas y del orquestador, conservando las modificaciones realizadas previamente a los criterios de clasificación del supervisor (DD-02).

No se modificaron los datos, herramientas ni lógica del simulador.

| Parámetro | Valor |
|---|---|
| Escenario | E2 |
| Criterio | `cobertura_equilibrada` |
| Versión de datos | `datos_1.0` |
| Versión del motor | `capstone_base_1.5` |
| Horizonte | 3 días |
| Inventario del centro | 90 unidades |
| Necesidad total | 125 unidades |
| Consultas realizadas | 6 |
| Tokens de entrada | 5,159 |
| Tokens de salida | 2,702 |
| Resultado final | `requiere_correccion` |

**Pregunta original:**

¿Cómo distribuyen los agentes el inventario disponible cuando no es suficiente para cubrir las necesidades de las tres tiendas, utilizando el criterio `cobertura_equilibrada`?

**Predicción original:**

Se espera que los agentes identifiquen la escasez de inventario y propongan una distribución que busque equilibrar la cobertura entre las tres tiendas, respetando las restricciones de capacidad y necesidad. Debido a que el criterio no tiene una definición operacional explícita, es posible que los agentes utilicen también las prioridades de servicio para justificar la asignación, generando diferencias de interpretación entre especialistas.

**Cambio declarado:**

Cambio de escenario E1 a E2 (escasez con `cobertura_equilibrada`). Se conservan las instrucciones modificadas del supervisor.

### 1.3 Análisis del experimento

#### Interpretación del escenario

El escenario E2 conserva las necesidades de reposición de E1, pero reduce el inventario del centro de 160 a 90 unidades.

Las necesidades estimadas de las tiendas suman 125 unidades, por lo que existe un faltante de 35 unidades que debe distribuirse entre las tiendas según el criterio seleccionado.

Esto introduce un conflicto de asignación que no existía en E1: ya no es posible cubrir completamente la demanda de las tres tiendas.

#### Necesidades y distribución

| Tienda | Existencias | Demanda estimada | Necesidad | Capacidad de envío | Envío propuesto |
|---|---:|---:|---:|---:|---:|
| T01 - Centro | 15 | 75 | 60 | 70 | 43 |
| T02 - Norte | 20 | 60 | 40 | 60 | 29 |
| T03 - Sur | 20 | 45 | 25 | 50 | 18 |
| **Total** | **55** | **180** | **125** | **180** | **90** |

El orquestador propuso distribuir las 90 unidades disponibles, respetando las capacidades y necesidades de las tres tiendas.

En su justificación indicó que la propuesta alcanzaba coberturas de aproximadamente 71.7 %, 72.5 % y 72.0 %. Sin embargo, estos porcentajes corresponden a la proporción de la necesidad de reposición atendida, no a la cobertura de demanda.

La cobertura de demanda debe considerar las existencias iniciales de cada tienda:

`Cobertura = (Existencias + Envío) / Demanda estimada`

| Tienda | Reposición atendida | Cobertura de demanda |
|---|---:|---:|
| T01 - Centro | 71.7 % | 77.3 % |
| T02 - Norte | 72.5 % | 81.7 % |
| T03 - Sur | 72.0 % | 84.4 % |
| **Global** | **72.0 %** | **80.6 %** |

La distribución presenta una diferencia de aproximadamente 7.1 puntos porcentuales entre las tiendas con mayor y menor cobertura.

Aunque la propuesta distribuye proporcionalmente las necesidades de reposición, no equilibra en la misma proporción la cobertura de demanda.

#### Comportamiento de los especialistas

**Inventario:** identificó correctamente las necesidades de reposición, con un total de 125 unidades. Señaló que T01 concentra el mayor faltante.

**Logística:** identificó las 90 unidades disponibles y las capacidades máximas por tienda. A diferencia de E1, no propuso una distribución cuantitativa específica. Reconoció que el criterio de cobertura equilibrada no incluía una fórmula de asignación.

**Comercial:** recomendó atender primero T02, después T01 y finalmente T03, siguiendo sus prioridades de servicio. Esta recomendación mantiene la ambigüedad sobre cómo deben relacionarse las prioridades comerciales con el criterio activo de cobertura equilibrada.

**Orquestador:** propuso distribuir las unidades proporcionalmente a las necesidades de reposición, sin utilizar la prioridad comercial como factor principal.

La propuesta respetó las restricciones programadas, pero su justificación confundió la cobertura de demanda con la proporción de reposición atendida.

#### Validaciones y supervisión

Las comprobaciones deterministas de Python fueron satisfactorias:

- Se incluyeron las tres tiendas sin duplicados.
- Las cantidades fueron enteras y no negativas.
- Las 90 unidades enviadas no excedieron el inventario del centro.
- Ningún envío superó la capacidad o necesidad estimada de su tienda.

`comprobaciones.conforme = true`

Sin embargo, el supervisor emitió `requiere_correccion`.

Su principal observación fue que el orquestador justificó el equilibrio utilizando porcentajes de reposición atendida en lugar de cobertura de demanda.

El supervisor identificó correctamente esta diferencia conceptual, pero sus propios cálculos también fueron incorrectos: reportó coberturas de 57.3 %, 48.3 % y 40.0 %, cuando los valores correctos son 77.3 %, 81.7 % y 84.4 %.

Los porcentajes reportados por el supervisor corresponden a los envíos divididos entre la demanda estimada, sin incluir las existencias iniciales.

**Interpretación preliminar del equipo:**

El resultado demuestra que las validaciones numéricas implementadas pueden confirmar la factibilidad de una distribución sin verificar que las métricas utilizadas para justificarla sean correctas.

El supervisor detectó una inconsistencia real, pero cometió otro error al intentar fundamentarla.

La ausencia de una definición operacional completa de `cobertura_equilibrada` dificulta establecer si la distribución obtenida corresponde al criterio deseado por el negocio.

### 1.4 Hallazgos y limitaciones

| ID | Hallazgo | Tipo | Implicación |
|---|---|---|---|
| H-E2-01 | El orquestador distribuyó las unidades proporcionalmente a las necesidades de reposición. | Interpretación del criterio | El reparto no demuestra que se haya aplicado equilibrio de cobertura de demanda. |
| H-E2-02 | El orquestador confundió la proporción de reposición atendida con cobertura de demanda. | Error de interpretación de métricas | Definir explícitamente ambas métricas y su uso. |
| H-E2-03 | El supervisor identificó la confusión, pero calculó incorrectamente las coberturas. | Error de evaluación | Evaluar la necesidad de verificar los cálculos mediante código. |
| H-E2-04 | Python consideró conforme una propuesta cuya justificación contenía errores. | Límite de los controles | Las validaciones actuales comprueban restricciones, pero no todos los cálculos de las explicaciones. |
| H-E2-05 | Comercial continuó recomendando un orden basado en prioridad de servicio. | Ambigüedad de especificación | Definir cómo interactúan las prioridades cuando el criterio activo es cobertura equilibrada. |
| H-E2-06 | Logística no propuso cantidades específicas y reconoció la falta de una fórmula de distribución. | Comportamiento del agente | Observar si limitar su responsabilidad a identificar restricciones resulta más adecuado. |

**Limitaciones de la ejecución:**

- Solo se realizó una ejecución de E2.
- No existe una definición operacional confirmada de `cobertura_equilibrada`.
- No se puede establecer que la distribución propuesta sea óptima para el negocio.
- Las validaciones de Python no comprueban directamente los porcentajes utilizados en las justificaciones de los agentes.
- El supervisor también puede cometer errores numéricos.
- No se evaluó la estabilidad de las respuestas ante ejecuciones independientes.
- El escenario utiliza datos sintéticos y no incorpora restricciones adicionales de transporte o almacenamiento.

### 1.5 Conclusión del experimento

La ejecución E2 permitió observar cómo los agentes enfrentan una situación de escasez bajo el criterio `cobertura_equilibrada`.

El orquestador produjo una distribución factible que utiliza las 90 unidades disponibles sin incumplir las restricciones programadas. Sin embargo, interpretó el equilibrio como una distribución proporcional de las necesidades de reposición y utilizó esos porcentajes para justificar la cobertura.

El supervisor identificó la diferencia entre ambas métricas y solicitó una corrección, aunque sus propios cálculos de cobertura también fueron incorrectos.

La predicción se cumplió parcialmente: los agentes identificaron la escasez y buscaron distribuir el inventario de manera equilibrada, pero no aplicaron una definición verificable de cobertura de demanda.

Este resultado respalda la necesidad de definir operacionalmente los criterios de distribución y considerar mecanismos de verificación para los cálculos utilizados en las justificaciones.

No se propone modificar todavía el simulador. Los hallazgos se conservarán para comparar el comportamiento con E3 y posteriormente evaluar instrucciones más explícitas.

### 1.6 Referencia de ejecución

[E2_intento_003_a81d9ba3110b](./historial/intento_003_a81d9ba3110b.json)

Identificador: `a81d9ba3110b`

**Decisiones relacionadas:** DD-01 — Definición operacional de criterios de asignación.

---

## E3 — Escasez con prioridad de servicio

### 1.1 Objetivo

Evaluar cómo cambia la distribución de inventario respecto a E2 cuando se conservan las mismas necesidades, existencias, capacidades e inventario disponible, pero se utiliza el criterio `prioridad_servicio`.

Se busca observar si los agentes interpretan este criterio como un orden de atención, cómo resuelven la escasez y qué consecuencias tiene la distribución para las tiendas de menor prioridad.

### 1.2 Configuración

Se utilizó el escenario E3, que conserva los datos y restricciones de E2, pero cambia el criterio activo a `prioridad_servicio`.

Se mantuvieron las instrucciones originales de los especialistas y del orquestador, así como las modificaciones de supervisión adoptadas provisionalmente en DD-02.

No se modificaron herramientas ni lógica del simulador.

| Parámetro | Valor |
|---|---|
| Escenario | E3 |
| Criterio | `prioridad_servicio` |
| Versión de datos | `datos_1.0` |
| Versión del motor | `capstone_base_1.5` |
| Horizonte | 3 días |
| Inventario del centro | 90 unidades |
| Necesidad total | 125 unidades |
| Consultas realizadas | 6 |
| Tokens de entrada | 5,233 |
| Tokens de salida | 2,191 |
| Resultado final | `propuesta_para_revision_humana` |

**Pregunta original:**

¿Cómo cambia la propuesta de distribución respecto a E2 cuando se conserva la misma información, pero se utiliza el criterio `prioridad_servicio`?

**Predicción original:**

Se espera que los agentes favorezcan la asignación de inventario a las tiendas según su prioridad de servicio: primero T02, después T01 y finalmente T03. Debido a la disponibilidad limitada, es posible que las tiendas de menor prioridad reciban una proporción menor de su necesidad, particularmente T03. Sin embargo, al no existir una definición operacional explícita del criterio, la distribución dependerá de cómo los agentes interpreten la prioridad y su relación con la cobertura.

**Cambio declarado:**

Cambio de criterio de `cobertura_equilibrada` a `prioridad_servicio` respecto a E2, manteniendo los mismos datos y las instrucciones de los agentes.

### 1.3 Análisis del experimento

#### Interpretación del escenario

E3 utiliza las mismas condiciones de escasez de E2: 90 unidades disponibles en el centro frente a una necesidad total de 125 unidades.

La diferencia es que el criterio activo cambia a `prioridad_servicio`, donde las tiendas tienen las siguientes prioridades:

- T02 - Norte: prioridad 1.
- T01 - Centro: prioridad 2.
- T03 - Sur: prioridad 3.

El simulador establece que el valor 1 representa la prioridad más alta, pero no define un algoritmo obligatorio para convertir esos valores en cantidades de distribución.

#### Necesidades y distribución

| Tienda | Prioridad | Necesidad | Envío E2 | Envío E3 | Necesidad pendiente E3 |
|---|---:|---:|---:|---:|---:|
| T01 - Centro | 2 | 60 | 43 | 50 | 10 |
| T02 - Norte | 1 | 40 | 29 | 40 | 0 |
| T03 - Sur | 3 | 25 | 18 | 0 | 25 |
| **Total** | | **125** | **90** | **90** | **35** |

El orquestador propuso atender las tiendas de manera secuencial según su prioridad:

1. Asignar 40 unidades a T02 para cubrir completamente su necesidad.
2. Asignar las 50 unidades restantes a T01, cubriendo parcialmente su necesidad.
3. Dejar T03 sin reposición debido al agotamiento del inventario central.

Esta distribución utiliza las 90 unidades disponibles y respeta las restricciones programadas.

#### Comparación de cobertura entre E2 y E3

| Tienda | Cobertura E2 | Cobertura E3 | Diferencia |
|---|---:|---:|---:|
| T01 - Centro | 77.3 % | 86.7 % | +9.3 pp |
| T02 - Norte | 81.7 % | 100 % | +18.3 pp |
| T03 - Sur | 84.4 % | 44.4 % | -40.0 pp |
| **Cobertura global** | **80.6 %** | **80.6 %** | **0 pp** |
| **Cobertura mínima** | **77.3 %** | **44.4 %** | **-32.9 pp** |

Ambas propuestas alcanzan la misma cobertura global, porque utilizan las mismas 90 unidades y parten de las mismas existencias.

Sin embargo, distribuyen de manera diferente el faltante.

E2 mantiene una cobertura superior al 77 % en las tres tiendas, mientras que E3 garantiza el 100 % de cobertura a T02, favorece a T01 y deja a T03 con una cobertura del 44.4 %, correspondiente únicamente a sus existencias iniciales.

La comparación demuestra que una misma cobertura global puede ocultar diferencias importantes en la atención individual de las tiendas.

#### Comportamiento de los especialistas

**Inventario:** identificó correctamente las necesidades de las tres tiendas, aunque indicó que no contaba con un identificador de producto, pese a que el caso incluye P01. Esta observación debe analizarse considerando el contexto específico proporcionado al agente.

**Logística:** identificó las restricciones de inventario y capacidad sin proponer cantidades específicas. Señaló que no disponía de las prioridades necesarias para determinar una distribución final.

**Comercial:** identificó correctamente el orden de atención T02, T01 y T03, y recomendó favorecer las tiendas de acuerdo con sus valores de prioridad.

**Orquestador:** integró las recomendaciones y resolvió explícitamente la diferencia entre la información disponible para Logística y Comercial. Utilizó las prioridades informadas por este último para proponer una distribución secuencial.

En esta ejecución, el criterio `prioridad_servicio` produjo una interpretación más directa entre los agentes, especialmente Comercial y Orquestador.

#### Validaciones y supervisión

Las comprobaciones deterministas de Python fueron satisfactorias:

- Se incluyeron las tres tiendas sin duplicados.
- Las cantidades fueron enteras y no negativas.
- El total enviado fue de 90 unidades, sin superar el inventario del centro.
- Ninguna tienda recibió más unidades de las que necesitaba o podía recibir.

`comprobaciones.conforme = true`

El supervisor emitió `conforme_para_revision_humana`.

En su revisión confirmó que la propuesta respetaba el orden de prioridad, las cantidades permitidas y las validaciones de Python.

También identificó que el orquestador había dejado pendiente repetir una comprobación que ya se encontraba ejecutada y conforme, por lo que consideró redundante esa observación.

Mantuvo como pendientes la aprobación humana y la revisión de restricciones de transporte no modeladas.

**Interpretación preliminar del equipo:**

El resultado coincide con la predicción: los agentes favorecieron a las tiendas de mayor prioridad y T03 quedó sin reposición.

No obstante, que el reparto sea consistente con el orden de prioridades no demuestra que esta sea la interpretación operacional que requiere el negocio.

El criterio sigue siendo flexible y podría admitir otras distribuciones que favorezcan a T02 sin dejar completamente sin reposición a T03.

#### Comparación de consumo con E2

| Métrica | E2 | E3 |
|---|---:|---:|
| Tokens de entrada | 5,159 | 5,233 |
| Tokens de salida | 2,702 | 2,191 |
| **Total** | **7,861** | **7,424** |
| Consultas | 6 | 6 |

E3 consumió 437 tokens menos que E2, aproximadamente un 5.6 % menos.

Esta diferencia corresponde únicamente a las ejecuciones observadas y no permite concluir que `prioridad_servicio` tenga un menor costo de operación.

### 1.4 Hallazgos y limitaciones

| ID | Hallazgo | Tipo | Implicación |
|---|---|---|---|
| H-E3-01 | El orquestador distribuyó el inventario secuencialmente según las prioridades. | Interpretación del criterio | Existe una interpretación operativa observable de `prioridad_servicio`, pero requiere confirmación del negocio. |
| H-E3-02 | T03 no recibió unidades adicionales y quedó con una cobertura del 44.4 %. | Consecuencia de negocio | Evaluar si resulta aceptable concentrar el faltante en tiendas de menor prioridad. |
| H-E3-03 | E2 y E3 alcanzaron la misma cobertura global, pero con diferencias importantes entre tiendas. | Resultado comparativo | La cobertura global no es suficiente para evaluar la conveniencia de una distribución. |
| H-E3-04 | El orquestador integró las prioridades comerciales que Logística no tenía disponibles. | Comportamiento satisfactorio | Conservar como evidencia de coordinación entre especialistas. |
| H-E3-05 | Python y el supervisor consideraron conforme la propuesta. | Comportamiento satisfactorio | La propuesta puede presentarse para revisión humana bajo las restricciones modeladas. |
| H-E3-06 | El escenario activa `prioridad_servicio`, mientras que la política conserva `cobertura_equilibrada` en `criterio_prioridad`. | Diferencia de configuración | Confirmar la precedencia entre el criterio activo del escenario y el valor predeterminado de la política. |

**Limitaciones de la ejecución:**

- Se realizó una sola ejecución de E3.
- El criterio `prioridad_servicio` no cuenta con una definición operacional completa validada por el negocio.
- El reparto secuencial es una interpretación del agente, no una política empresarial confirmada.
- No se evaluaron alternativas que favorezcan las prioridades manteniendo una cobertura mínima para todas las tiendas.
- Las diferencias entre E2 y E3 son compatibles con el cambio de criterio, pero la variabilidad del modelo impide atribuirlas exclusivamente a este sin pruebas adicionales.
- La propuesta es factible según las restricciones programadas, pero no se ha demostrado que sea la mejor alternativa comercial.
- No se ejecutaron movimientos reales de inventario.

### 1.5 Conclusión del experimento

La ejecución E3 mostró que los agentes interpretaron `prioridad_servicio` como un orden secuencial de atención, asignando primero las unidades necesarias para cubrir T02, después el inventario restante a T01 y dejando T03 sin reposición.

La distribución cumplió las restricciones programadas y el supervisor emitió `conforme_para_revision_humana`.

La predicción se cumplió en esta ejecución, ya que las tiendas de mayor prioridad recibieron mayor atención y la tienda de menor prioridad absorbió la mayor parte del faltante.

La comparación con E2 muestra que ambos criterios produjeron propuestas distintas con la misma cobertura global, pero consecuencias diferentes para cada tienda.

Aunque `prioridad_servicio` resultó más sencillo de interpretar para los agentes, todavía se requiere confirmar si el negocio espera una asignación secuencial o una distribución ponderada que considere también niveles mínimos de cobertura.

Se propone conservar este resultado como referencia para evaluar posteriormente definiciones operacionales explícitas de ambos criterios, sin modificar todavía la lógica del simulador.

### 1.6 Referencia de ejecución

[E3_intento_004_9750fd8fd220](./historial/intento_004_9750fd8fd220.json)

Identificador: `9750fd8fd220`

**Comparación principal:**

[E2_intento_003_a81d9ba3110b](./historial/intento_003_a81d9ba3110b.json)

**Decisiones relacionadas:** DD-01 — Definición operacional de criterios de asignación.

---

# 3. Resumen de experimentos

Esta tabla reúne los resultados principales. El detalle y la evidencia se conservan en cada sección de experimento.

| Experimento | Pregunta / hipótesis | Cambio realizado | Evidencia observada | Resultado | Conclusión y límites |
|---|---|---|---|---|---|
| **E1 — Referencia inicial** | Comprender el comportamiento con inventario suficiente. Predicción: no se requerirá reposición. | Ninguno. Configuración original. | Necesidad 125; envío 125; remanente 35; cobertura 100 %; Python conforme; supervisor `requiere_correccion`. | La predicción no se cumplió. Se identificaron discrepancias entre especialistas y ambigüedades en criterios de negocio y supervisión. | Se obtuvo una propuesta numéricamente factible. No se ha comprobado la estabilidad del comportamiento ni la interpretación de criterios bajo escasez. |
| **E1-B — Criterios del supervisor** | ¿Definir criterios explícitos permite distinguir mejor entre incumplimientos, omisiones corregibles y decisiones pendientes de autoridad humana? | Se modificaron únicamente las instrucciones del supervisor. Se conservaron los datos, el escenario E1 y la configuración de los demás agentes | Mismo reparto 60/40/25; cobertura 100 %; Python conforme. El supervisor reconoció que las 35 unidades remanentes no incumplen restricciones y mantuvo las limitaciones pendientes. | El dictamen cambió de requiere_correccion a conforme_para_revision_humana, sin modificar la distribución final. | La hipótesis recibió apoyo preliminar: el supervisor distinguió mejor entre restricciones y asuntos pendientes. No se demuestra consistencia general ni causalidad exclusiva, ya que se ejecutó nuevamente todo el flujo y solo hay una corrida por configuración. |
| **E2 — Escasez con cobertura equilibrada** | ¿Cómo distribuyen los agentes el inventario insuficiente utilizando cobertura_equilibrada? Se espera que intenten equilibrar coberturas, aunque podrían incorporar prioridades de servicio. | Escenario E2, con 90 unidades disponibles. Se conservaron las instrucciones originales de los especialistas y los criterios modificados del supervisor. | Reparto 43/29/18; cobertura global 80.6 %; Python conforme. El orquestador confundió reposición atendida con cobertura de demanda y el supervisor también calculó incorrectamente las coberturas. | requiere_correccion. El reparto es factible, pero su justificación no acredita correctamente el criterio seleccionado. | Los agentes identificaron la escasez y propusieron una distribución proporcional de reposición, no un equilibrio verificable de cobertura. Falta definir el criterio y validar los cálculos; solo se realizó una ejecución. |
| **E3 — Escasez con prioridad de servicio** | ¿Cómo cambia la distribución respecto a E2 con prioridad_servicio? Se espera favorecer T02, T01 y T03, posiblemente dejando a la última con menor reposición. | Cambio de criterio a prioridad_servicio, manteniendo los datos, restricciones e instrucciones utilizadas en E2. | Reparto 50/40/0; cobertura global 80.6 %; cobertura mínima 44.4 %; Python conforme. Se atendió primero T02 y después T01. | propuesta_para_revision_humana. El supervisor consideró conforme la distribución secuencial según prioridades. | La predicción se cumplió en esta ejecución. E2 y E3 muestran igual cobertura global, pero diferentes consecuencias por tienda. La interpretación secuencial requiere validación del negocio y no se ha comprobado estabilidad. |

---

# 4. Pendientes de investigación

| ID | Pregunta pendiente | Origen | Próximo paso |
|---|---|---|---|
| P-01 | ¿Cuál es la definición operacional de cada criterio de asignación? | E1 / DD-01 | Consultar al profesor y formular hipótesis verificables. |
| P-02 | ¿Cuándo debe el supervisor emitir cada dictamen? | E1 / DD-02 | Proponer criterios y realizar una comparación controlada. |
| P-03 | ¿Se requiere una política explícita para el inventario remanente? | E1 / DD-03 | Documentar una regla provisional y solicitar validación. |
| P-04 | ¿Cómo se comportan los agentes cuando el inventario es insuficiente? | E1 | Explorar escenarios de escasez. |
| P-05 | ¿Cuánto cambia el resultado al repetir la misma configuración? | E1 | Considerar repeticiones para observar variabilidad. |
| P-06 | ¿El supervisor distingue una ambigüedad de negocio de un defecto corregible? | E1 / DD-02 | Evaluar con instrucciones más explícitas. |

---

# 5. Consideraciones para la recomendación final

La ejecución E1 no permite todavía recomendar un piloto operativo, pero proporciona evidencia inicial del funcionamiento del prototipo y de sus controles.

Los experimentos posteriores deben permitir evaluar si las modificaciones propuestas mejoran la consistencia, la interpretación de los criterios y la calidad de las recomendaciones.

Antes de considerar una validación funcional con usuarios del negocio, será necesario confirmar las políticas de distribución y las condiciones de aceptación y escalamiento.

La recomendación final deberá distinguir entre resultados comprobados en el entorno sintético, decisiones de diseño del equipo y aspectos pendientes de validación externa.