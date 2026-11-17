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

# 3. Resumen de experimentos

Esta tabla reúne los resultados principales. El detalle y la evidencia se conservan en cada sección de experimento.

| Experimento | Pregunta / hipótesis | Cambio realizado | Evidencia observada | Resultado | Conclusión y límites |
|---|---|---|---|---|---|
| **E1 — Referencia inicial** | Comprender el comportamiento con inventario suficiente. Predicción: no se requerirá reposición. | Ninguno. Configuración original. | Necesidad 125; envío 125; remanente 35; cobertura 100 %; Python conforme; supervisor `requiere_correccion`. | La predicción no se cumplió. Se identificaron discrepancias entre especialistas y ambigüedades en criterios de negocio y supervisión. | Se obtuvo una propuesta numéricamente factible. No se ha comprobado la estabilidad del comportamiento ni la interpretación de criterios bajo escasez. |
| E1-B — Criterios del supervisor | ¿Definir criterios explícitos permite distinguir mejor entre incumplimientos, omisiones corregibles y decisiones pendientes de autoridad humana? | Se modificaron únicamente las instrucciones del supervisor. Se conservaron los datos, el escenario E1 y la configuración de los demás agentes | Mismo reparto 60/40/25; cobertura 100 %; Python conforme. El supervisor reconoció que las 35 unidades remanentes no incumplen restricciones y mantuvo las limitaciones pendientes. | El dictamen cambió de requiere_correccion a conforme_para_revision_humana, sin modificar la distribución final. | La hipótesis recibió apoyo preliminar: el supervisor distinguió mejor entre restricciones y asuntos pendientes. No se demuestra consistencia general ni causalidad exclusiva, ya que se ejecutó nuevamente todo el flujo y solo hay una corrida por configuración. |

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