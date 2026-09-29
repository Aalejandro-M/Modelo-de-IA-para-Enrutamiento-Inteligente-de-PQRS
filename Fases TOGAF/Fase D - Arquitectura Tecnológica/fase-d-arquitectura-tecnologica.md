# Fase A: Visión de la Arquitectura
## Proyecto: Modelo de IA para Enrutamiento Inteligente de PQRS
**Empresa de Referencia:** TelcoLatam (Empresa de Telecomunicaciones)

> **Nota de versión.** Este documento es la revisión arquitectónica de la Fase A original. Los cambios frente a la versión previa están marcados con 🔧 y resumidos en la sección 0. Los valores marcados como **(supuesto)** son cifras de ejemplo, razonables pero ilustrativas, que deben validarse con datos reales de TelcoLatam antes de la aprobación formal del documento.

---

## 0. Registro de cambios aplicados en esta revisión

| # | Cambio | Motivo |
|---|---|---|
| 1 | Se añadieron los Riesgos 3, 4 y 5 (integración con el CRM heredado, resistencia al cambio, cumplimiento de la Ley 1581 de 2012) | Estaban mencionados como preocupaciones de las partes interesadas pero no se reflejaban en el registro de riesgos |
| 2 | Se reconciliaron las métricas de cobertura (85%), exactitud (88%) y umbral de confianza (75%) | Eran tres cifras relacionadas sin una explicación de cómo se conectan entre sí |
| 3 | Se explicó la relación entre el objetivo de negocio (–40% en tiempo de primera respuesta) y el KPI de triage (–90%) | El objetivo de negocio y el KPI operativo miden cosas distintas y no estaban conectados |
| 4 | Se definió el modelo de respuesta cuando un ticket se divide entre varias divisiones: **respuestas independientes, una por división** | Decisión pendiente, confirmada por el patrocinador |
| 5 | Se especificó la jurisdicción de protección de datos: **Colombia — Ley 1581 de 2012 (Habeas Data)**, Decreto 1377 de 2013, y el marco regulatorio de tiempos de respuesta de PQR (Resolución CRC 5050 de 2016, art. 2.1.24.3) | La versión original mencionaba "la Ley de Protección de Datos Personales" sin especificar país |
| 6 | Se añadió un concepto de solución (*Solution Concept*) en diagrama | La Visión de Arquitectura solo tenía descripción narrativa |
| 7 | Se amplió la Evaluación de Preparación con factores de gobernanza, capacidad de las áreas receptoras y preparación legal/cumplimiento | Solo cubría TI y Cultura |
| 8 | Se añadieron métricas base **(supuestos ilustrativos)**: CSAT actual y tasa de reasignaciones por mala clasificación | No existía un punto de partida cuantificado para medir la mejora prometida |
| 9 | Se reescribieron los principios con la estructura completa de TOGAF (Nombre, Enunciado, Justificación, Implicaciones) y se agregaron un principio de Tecnología y uno de Privacidad | Faltaban implicaciones concretas y no había principio para el dominio Tecnología, pese a estar en el alcance |
| 10 | Se incluyó explícitamente en el Alcance la interfaz de validación/corrección para agentes | Era un requisito de un stakeholder pero no aparecía en la sección de alcance |
| 11 | Se agregó un mapa de partes interesadas (poder/interés) con roles adicionales | Faltaban el Oficial de Protección de Datos, los líderes de las divisiones receptoras y el cliente final |
| 12 | Se agregó una memoria de cálculo ilustrativa **(supuesto)** para el ROI de 8 meses | La cifra no tenía sustento numérico |
| 13 | Se agregó la sección 12 con la arquitectura de sistemas (GCP) y el presupuesto desarrollados en la propuesta técnica posterior | El presupuesto de USD 50.000 (sección 3) tampoco tenía desglose de a qué se destinaría |

---

## 1. Establecer el Proyecto de Arquitectura

La arquitectura empresarial es una capacidad de negocio, por lo que este ciclo del ADM (Architecture Development Method) se manejará como un proyecto formal utilizando el marco de gestión de proyectos de TelcoLatam.

* **Nombre del Proyecto:** IA-PQRS Smart Routing.
* **Alineación corporativa:** Este proyecto apoya el marco de gestión estratégica de Servicio al Cliente, buscando el respaldo de la gerencia corporativa y el compromiso de la gerencia de línea.
* 🔧 **Patrocinio formal:** Director de Servicio al Cliente (patrocinador de negocio) y Gerente de TI (patrocinador tecnológico), quienes firman la aprobación de esta fase (sección 11).

## 2. Identificar las Partes Interesadas, las Preocupaciones y los Requisitos del Negocio

El compromiso de las partes interesadas en esta etapa tiene tres objetivos: identificar componentes de la visión y requisitos, identificar los límites del alcance y reconocer las preocupaciones y factores culturales que darán forma a la comunicación de la arquitectura.

### 2.1 Partes interesadas principales

* **Director de Servicio al Cliente (Stakeholder principal):**
  * *Preocupación:* Tiempos de respuesta lentos y alta insatisfacción del cliente.
  * *Requisito:* Clasificación precisa de al menos el 85% de las PQRS sin intervención humana.
* **Gerente de TI (Stakeholder técnico):**
  * *Preocupación:* Integración del modelo de IA con el CRM heredado (legacy) de la empresa.
  * *Requisito:* Arquitectura basada en microservicios y APIs RESTful.
* **Agentes de Soporte (Usuarios finales):**
  * *Preocupación:* Miedo al reemplazo laboral y fatiga por reasignación manual.
  * *Requisito:* Interfaz clara donde la IA sugiera el área (Facturación, Soporte Técnico, Retención) y permita corrección manual si es necesario.

### 2.2 🔧 Mapa de partes interesadas (poder / interés)

| Parte interesada | Interés | Poder / influencia | Estrategia de involucramiento |
|---|---|---|---|
| Director de Servicio al Cliente | Alto | Alto | Gestionar de cerca — patrocinador |
| Gerente de TI | Alto | Alto | Gestionar de cerca — patrocinador |
| Agentes de Soporte | Alto | Medio | Mantener informados; involucrar en el diseño de la interfaz de validación |
| 🔧 Líderes de Facturación / Soporte Técnico / Retención (nuevo) | Medio | Medio | Mantener satisfechos — deben validar su capacidad de recibir casos ya enrutados, incluyendo respuestas independientes por división (sección 8) |
| 🔧 Oficial de Protección de Datos / Cumplimiento (nuevo) | Medio | Alto | Consultar antes de usar el histórico de PQRS para entrenar el modelo (ver Principio de Privacidad, sección 7, y Riesgo 5, sección 10) |
| 🔧 Cliente final / ciudadano (nuevo) | Alto (impactado por el resultado) | Bajo (influencia directa) | Monitorear a través del CSAT y del tiempo de respuesta |

## 3. Confirmar y Definir los Objetivos del Negocio, los Impulsores del Negocio y las Limitaciones

Es necesario identificar los objetivos comerciales y los impulsores estratégicos de la organización, asegurando que las definiciones existentes estén actualizadas.

* **Objetivo del Negocio:** Reducir el tiempo promedio de primera respuesta a las PQRS en un 40% durante el primer trimestre de implementación.
* **Impulsores del Negocio (Drivers):** Alto volumen de solicitudes diarias (más de 10.000 PQRS) y penalizaciones regulatorias por incumplimiento en tiempos de respuesta. 🔧 En Colombia, el proveedor de servicios de comunicaciones debe responder toda PQR dentro de los 15 días hábiles siguientes a su presentación (Resolución CRC 5050 de 2016, art. 2.1.24.3, prorrogable 15 días hábiles adicionales para práctica de pruebas); si no hay respuesta dentro del término aplica el Silencio Administrativo Positivo, que resuelve la PQR a favor del usuario. Esta es la penalización regulatoria concreta que motiva el proyecto.
* **Limitaciones (Constraints):** Presupuesto estricto de $50.000 USD, límite de tiempo de 3 meses para el MVP, y cumplimiento obligatorio de la **Ley 1581 de 2012 (Ley Estatutaria de Habeas Data)** y su Decreto reglamentario 1377 de 2013 en la manipulación de los textos de las quejas. 🔧 *(Jurisdicción especificada: Colombia, confirmada por el patrocinador.)*

### 3.1 🔧 Nota de trazabilidad: objetivo de negocio vs. KPI operativo

El objetivo de negocio (–40% en el tiempo total de primera respuesta) y el KPI de la sección 9 (–90% en el tiempo de *triage*) miden cosas distintas y no se puede asumir que uno implique el otro automáticamente. El razonamiento que los conecta es el siguiente:

* En la línea base, el cuello de botella declarado no es solo el tiempo de lectura del ticket (3–5 min), sino la **cola de hasta 24 horas** que se forma porque la capacidad de triage manual no alcanza a absorber el volumen (>10.000 PQRS/día).
* Si el motor de IA elimina prácticamente ese tiempo de servicio de triage, la cola generada por saturación de capacidad debería reducirse drásticamente (en términos de teoría de colas: al aumentar la tasa de servicio muy por encima de la tasa de llegada, el tiempo de espera tiende a colapsar), lo que hace plausible el 40% de reducción total.
* Sin embargo, los pasos posteriores (proyección, revisión y firma de la respuesta) **permanecen manuales** y no están cubiertos por esta automatización; si estos pasos resultan ser el nuevo cuello de botella, el 40% podría no alcanzarse solo con la mejora del triage.
* **Recomendación:** medir, además del tiempo de triage, el **tiempo de ciclo total** end-to-end como KPI complementario desde el piloto (sección 8), y tratar el 40% como una meta a validar, no como un hecho garantizado por el 90%.

## 4. Evaluar las Capacidades del Negocio

Este paso busca comprender el nivel de capacidad base y objetivo de la empresa. Es fundamental identificar las capacidades comerciales requeridas que la empresa debe poseer para actuar sobre las prioridades estratégicas.

* **Capacidad Base (Baseline):** Actualmente, la redirección de PQRS es 100% manual. Los agentes leen cada ticket y lo asignan en el CRM, tomando un promedio de 3 a 5 minutos por ticket.
* 🔧 **Métricas base adicionales (supuesto, a validar con datos reales de TelcoLatam):**
  * CSAT actual del proceso de PQRS: **61/100** *(supuesto)*.
  * Tasa de reasignaciones por clasificación inicial incorrecta ("¿Es de nuestra competencia? → No"): **≈ 18% de los tickets** *(supuesto)*.
  * Estas dos cifras deben reemplazarse por datos reales del CRM antes de fijar la línea base definitiva; se incluyen aquí solo para que el caso de negocio tenga un punto de partida cuantificable.
* **Capacidad Objetivo (Target):** Capacidad de procesamiento de lenguaje natural (NLP) integrada en el CRM que pre-clasifica y enruta el ticket en milisegundos.
* **Brechas (Gaps):** La empresa carece de talento interno especializado en IA (brecha de habilidades) y de infraestructura de procesamiento en la nube nativa.

## 5. Evaluar la Preparación para la Transformación del Negocio

Se utiliza una Evaluación de Preparación para la Transformación del Negocio para evaluar y cuantificar la preparación de la organización para someterse a un cambio.

* **Factores de preparación:**
  * *TI:* Preparación media (el CRM tiene APIs, pero la infraestructura es On-Premise).
  * *Cultura Organizacional:* Preparación baja/media. Hay resistencia al cambio por parte del nivel operativo ante la automatización.
  * 🔧 *Gobernanza y patrocinio (nuevo):* Preparación alta — existe patrocinio formal de dos directivos senior (sección 1), pero **falta constituir un comité de gobierno de datos/IA** que apruebe el uso del histórico de PQRS para entrenamiento (ver Riesgo 5).
  * 🔧 *Capacidad de las áreas receptoras — Facturación, Soporte Técnico, Retención (nuevo):* Sin evaluar formalmente. Al haberse definido que cada división responde de forma **independiente** (sección 8), el riesgo de descoordinación entre áreas se reduce, pero debe confirmarse que cada división tiene capacidad operativa para atender casos que hoy le llegan tras una cola de 24 horas y que, en el escenario objetivo, le llegarán casi en tiempo real.
  * 🔧 *Preparación legal / cumplimiento (nuevo):* Media — TelcoLatam ya opera bajo la Ley 1581 de 2012, pero no se ha evaluado si la política de tratamiento de datos vigente cubre el uso de texto histórico de PQRS para entrenar modelos de IA; probablemente se requiera anonimización o una autorización específica del titular (Decreto 1377 de 2013).
* Estos resultados se utilizan para dar forma al alcance de la arquitectura e identificar áreas de riesgo.

## 6. Definir el Alcance

Se debe definir qué está dentro y qué está fuera del alcance de los esfuerzos de la Arquitectura Base y la Arquitectura Objetivo.

* **Dentro del alcance (In-scope):**
  * Desarrollo del modelo de IA (clasificación de texto).
  * Creación del microservicio de inferencia.
  * Integración con el módulo de enrutamiento del CRM actual.
  * 🔧 **Interfaz de validación y corrección de la sugerencia de la IA para los Agentes de Soporte** (antes implícita, ahora explícita, ya que es un requisito directo del stakeholder "Agentes de Soporte" en la sección 2).
* **Fuera del alcance (Out-of-scope):**
  * La resolución automática o respuesta automatizada (Chatbot) hacia el cliente final; el alcance se limita solo al enrutamiento al área responsable.
  * 🔧 **Consolidación automática de respuestas entre divisiones.** Cuando un ticket involucre a más de una división (por ejemplo, Facturación y Soporte Técnico), **cada división elabora y envía su propia respuesta de forma independiente**, siguiendo el proceso ya vigente de proyección, revisión y firma; no se construye un mecanismo de consolidación entre áreas en esta iteración (decisión del patrocinador).
* **Dominios de arquitectura a cubrir:** Negocio, Datos (datasets de entrenamiento de PQRS), Aplicación (APIs e integración CRM) y Tecnología (Infraestructura Cloud).

## 7. Confirmar y Definir los Principios de la Arquitectura, incluyendo los Principios del Negocio

🔧 Los principios se reescriben con la estructura completa de TOGAF (Nombre, Enunciado, Justificación, Implicaciones) y se agregan un principio de Tecnología y uno de Privacidad, dado que ambos dominios están en el alcance pero no tenían principio propio.

**Principio de Negocio 1 — El cliente es el centro de la operación**
* *Enunciado:* Toda decisión de diseño debe priorizar el enrutamiento ágil de la PQRS para mejorar la experiencia del cliente.
* *Justificación:* La insatisfacción del cliente y los tiempos de respuesta lentos son la preocupación explícita del Director de Servicio al Cliente y el driver principal del proyecto.
* *Implicaciones:* Cualquier decisión técnica que agregue latencia perceptible al cliente (por ejemplo, revisiones adicionales no esenciales) debe justificarse explícitamente frente a este principio.

**Principio de Arquitectura (Datos) — Los datos son un activo**
* *Enunciado:* Los históricos de PQRS deben estar limpios, clasificados y gobernados para entrenar modelos efectivos.
* *Justificación:* La calidad del modelo de clasificación depende directamente de la calidad del dataset de entrenamiento.
* *Implicaciones:* Se requiere un proceso de curaduría, etiquetado, auditoría y balanceo del dataset antes del entrenamiento (ver Riesgo 2, sección 10).

**Principio de Arquitectura (Aplicación) — Independencia tecnológica**
* *Enunciado:* El modelo de IA debe estar desacoplado del CRM para permitir escalabilidad futura.
* *Justificación:* Acoplar el modelo directamente al CRM heredado limitaría la escalabilidad y dificultaría futuras migraciones tecnológicas.
* *Implicaciones:* El modelo se expone como microservicio con API REST; el CRM lo consume como cliente, no como componente interno.

**🔧 Principio de Arquitectura (Tecnología) — Preferencia por servicios administrados en la nube (*cloud-first*)** *(nuevo)*
* *Enunciado:* La infraestructura del microservicio de inferencia debe desplegarse preferentemente sobre servicios administrados en la nube, en lugar de ampliar la huella on-premise del CRM heredado.
* *Justificación:* La capacidad objetivo exige clasificar en milisegundos y escalar ante picos de más de 10.000 PQRS/día; la infraestructura on-premise actual no está diseñada para esa elasticidad (brecha de infraestructura, sección 4).
* *Implicaciones:* Se requiere presupuesto y habilidades de operación cloud (brecha ya identificada); la integración con el CRM debe hacerse vía API, no por acceso directo a su base de datos.

**🔧 Principio de Arquitectura (Datos / Privacidad) — Privacidad desde el diseño** *(nuevo)*
* *Enunciado:* El tratamiento del texto de las PQRS para entrenar o ejecutar el modelo de IA debe cumplir la Ley 1581 de 2012 y su Decreto 1377 de 2013 desde la etapa de diseño, no como control posterior.
* *Justificación:* Las PQRS pueden contener datos personales, y ocasionalmente datos sensibles; procesarlos sin las autorizaciones o la anonimización adecuadas expone a TelcoLatam a sanciones de la Superintendencia de Industria y Comercio y a pérdida de confianza del cliente (Riesgo 5, sección 10).
* *Implicaciones:* El dataset de entrenamiento debe anonimizarse o seudonimizarse; la finalidad del tratamiento debe documentarse en la política de datos vigente; el Oficial de Protección de Datos debe validar el proceso antes de iniciar el entrenamiento.

## 8. Desarrollar la Visión de la Arquitectura

Basándose en las preocupaciones de las partes interesadas, los requisitos, el alcance y los principios, se crea una vista de alto nivel de las arquitecturas base y objetivo.

* **Escenario de Negocio** 🔧 *(corregido para reflejar la decisión de respuestas independientes):* Un cliente envía una queja extensa sobre una caída de internet y un cobro injustificado. En la *Arquitectura Base*, el ticket espera en una cola general hasta 24 horas hasta que un agente humano lo lee y lo envía primero a Soporte Técnico y, después, de forma secuencial, a Facturación — duplicando la espera del cliente. En la *Arquitectura Objetivo*, el modelo de IA extrae la intención y las entidades del texto, identifica que el caso corresponde a dos áreas (Soporte Técnico y Facturación) y notifica a ambas **simultáneamente**; cada división recibe su propio caso y **elabora y envía su respuesta de forma independiente**, siguiendo el mismo proceso de proyección, revisión y firma ya vigente, sin necesidad de coordinación ni consolidación entre áreas.
* Estas definiciones iniciales de la arquitectura deben almacenarse en el Repositorio de Arquitectura.

### 8.1 🔧 Concepto de solución (*Solution Concept*) *(nuevo)*

```mermaid
flowchart LR
    A[Cliente] --> B[Canal de entrada]
    B --> C[Motor IA: clasificacion NLP]
    C -->|confianza mayor o igual 75%| D[Agente valida o corrige]
    C -->|confianza menor 75%| E[Cola manual - Repartidor]
    D --> F[Distribucion simultanea a division-es responsable-s]
    E --> F
    F --> G1[Facturacion]
    F --> G2[Soporte Tecnico]
    F --> G3[Retencion]
    G1 --> H[Respuesta independiente por division]
    G2 --> H
    G3 --> H
    H --> A
```

*Cada rectángulo Gx representa una división que, si recibe el caso, sigue su propio ciclo de proyección–revisión–firma ya vigente en el Procedimiento PQRSD, sin depender de las demás.*

## 9. Definir las propuestas de valor de la arquitectura objetivo y los KPI

Es fundamental desarrollar el caso de negocio, definir las propuestas de valor para los grupos de partes interesadas y establecer métricas de rendimiento.

* **Propuesta de Valor:** Transformar el centro de contacto de un cuello de botella reactivo a un despachador inteligente y automatizado, reduciendo costos operativos y mitigando el riesgo de multas regulatorias.
* **KPIs (Indicadores Clave de Rendimiento)** 🔧 *reescritos y reconciliados:*
  * **Cobertura de enrutamiento automático:** ≥ 85% de las PQRS deben quedar enrutadas sin intervención humana (requisito del Director de Servicio al Cliente), lograda cuando el modelo entrega una confianza ≥ 75% (umbral operativo del Riesgo 1).
  * **Exactitud del modelo:** > 88% de precisión en la clasificación del área responsable, medida sobre el subconjunto de PQRS que supera el umbral de confianza del 75% (es decir, sobre el tramo cubierto automáticamente).
    * 🔧 *Nota de reconciliación:* cobertura (85%) y exactitud (88%) son magnitudes independientes — no hay garantía teórica de que un único umbral del 75% las satisfaga a ambas simultáneamente. Se recomienda tratar el 75% como punto de partida y ajustarlo durante el piloto (Fase C/D) según el balance real cobertura/exactitud que arroje el modelo.
  * **Reducción de tiempo de triage:** 90% de disminución en el tiempo de clasificación inicial (de 3–5 minutos a milisegundos).
  * **Reducción de tiempo total de primera respuesta:** 40% en el primer trimestre — sujeta a la nota de trazabilidad de la sección 3.1; se recomienda monitorear también el tiempo de ciclo de los pasos manuales posteriores (proyección, revisión, firma) como KPI complementario.
  * **Retorno de Inversión (ROI):** Recuperación de la inversión en 8 meses mediante la optimización de horas-hombre.
    * 🔧 **Memoria de cálculo ilustrativa (supuesto, a validar con datos reales de TelcoLatam):**
      * Volumen: 10.000 PQRS/día; con 85% de cobertura automática ≈ 8.500 tickets/día dejan de requerir triage manual.
      * Ahorro estimado por ticket automatizado: ≈ 4 minutos (de ~4 min manual a prácticamente 0).
      * Ahorro de tiempo: 8.500 × 4 min ≈ 566 horas-agente/día.
      * Costo promedio hora-agente *(supuesto)*: USD 6/hora → ahorro ≈ USD 3.400/día ≈ USD 71.000/mes (22 días hábiles).
      * Con una inversión de USD 50.000, el punto de equilibrio aritmético se alcanzaría en menos de un mes de operación estable, lo que sugiere que el horizonte de 8 meses de la Fase A original es conservador y probablemente ya incorpora la curva de aprendizaje del modelo, el tiempo de estabilización y los costos de infraestructura/talento adicionales (brecha de la sección 4) que no estaban desglosados. Se recomienda documentar estos supuestos explícitamente en el caso de negocio formal.

## 10. Identificar los riesgos de la transformación empresarial y las actividades de mitigación

Se deben identificar los riesgos asociados con la Visión de la Arquitectura y evaluar su nivel inicial y su estrategia de mitigación. Se consideran dos niveles de riesgo: Nivel de Riesgo Inicial y Nivel de Riesgo Residual.

* **Riesgo 1 — Clasificación errónea del modelo de IA (falsos positivos):** puede retrasar aún más el ticket (Riesgo Inicial: Crítico).
  * *Mitigación:* Implementar un umbral de confianza (*confidence score*). Si la IA tiene menos del 75% de seguridad en la predicción, el ticket se envía a la cola manual. (Riesgo Residual: Marginal).
* **Riesgo 2 — Sesgo en los datos de entrenamiento históricos** que provoque mala atención a ciertas demografías (Riesgo Inicial: Marginal).
  * *Mitigación:* Auditoría y balanceo del dataset antes de la fase de entrenamiento. (Riesgo Residual: Insignificante).
* 🔧 **Riesgo 3 — Integración con el CRM heredado** *(nuevo):* Las limitaciones, límites de tasa o indisponibilidad de las APIs del CRM on-premise pueden impedir el enrutamiento en tiempo real, afectando directamente la preocupación ya declarada por el Gerente de TI (Riesgo Inicial: Alto).
  * *Mitigación:* Diseñar el microservicio de inferencia desacoplado (Principio de independencia tecnológica) con reintentos automáticos y una cola de contingencia hacia el Repartidor si el CRM no responde a tiempo. (Riesgo Residual: Moderado).
* 🔧 **Riesgo 4 — Resistencia al cambio / baja adopción por parte de los agentes** *(nuevo):* El temor al reemplazo laboral y la preparación cultural baja/media (sección 5) pueden generar baja adopción del paso de validación humana, o desconfianza sistemática hacia las sugerencias de la IA (Riesgo Inicial: Alto).
  * *Mitigación:* Comunicar explícitamente que el rol del agente es de validación y no de reemplazo; capacitación temprana; involucrar a los Agentes de Soporte en el diseño de la interfaz de validación (Fase C). (Riesgo Residual: Bajo).
* 🔧 **Riesgo 5 — Incumplimiento de la Ley 1581 de 2012 en el tratamiento de datos de entrenamiento** *(nuevo):* Usar el histórico de PQRS sin las autorizaciones o la anonimización requeridas expone a TelcoLatam a sanciones de la Superintendencia de Industria y Comercio y a daño reputacional (Riesgo Inicial: Alto).
  * *Mitigación:* Anonimización/seudonimización del dataset y validación previa con el Oficial de Protección de Datos, conforme al Principio de Privacidad desde el diseño (sección 7). (Riesgo Residual: Bajo).

## 11. Desarrollar la Declaración de Trabajo de Arquitectura y plantear su aprobación

Se evaluarán los productos de trabajo requeridos contra los requisitos de rendimiento comercial y se estimarán los recursos necesarios para desarrollar la hoja de ruta.

* **Documento de Aprobación:** Se elabora formalmente la Declaración de Trabajo de Arquitectura, que incluirá la visión general, el plan del proyecto y los roles de: científicos de datos, ingenieros de datos, arquitectos de TI, 🔧 y además un **responsable de cumplimiento/protección de datos** y un **líder de producto o gerente de proyecto**, dado el nuevo Principio de Privacidad y los Riesgos 3 a 5 identificados en esta revisión.
* **Comunicaciones:** Se desarrolla el Plan de Comunicaciones de Arquitectura Empresarial para interactuar con las partes interesadas —incluyendo ahora a los líderes de las divisiones receptoras y al Oficial de Protección de Datos (sección 2.2)— sobre el progreso del proyecto.
* **Aprobación:** Se revisan los planes con los patrocinadores (Director de Servicio al Cliente y Gerente de TI) para asegurar la aprobación formal bajo los procedimientos de gobernanza adecuados y obtener la firma (*sign-off*) para proceder a la Fase B (Arquitectura de Negocio).

## 12. Arquitectura de Sistemas y Presupuesto (actualización posterior)

### 12.1 Resumen del cambio

El presupuesto de USD 50.000 fijado en la sección 3 no tenía una memoria de cálculo que lo respaldara. Además, se decidió anteponer un filtro de clasificación sin IA (motor de reglas por palabras clave) antes de invocar el modelo de IA, para reducir la latencia y el costo, escalando a IA solo cuando el motor de reglas no puede determinar el área con certeza. Con base en esto se desarrolló, como propuesta técnica posterior a esta Fase A, la arquitectura de sistemas en GCP y el presupuesto desglosado que se resumen a continuación.

### 12.2 Arquitectura

La clasificación pasa a tener tres niveles: **(1) Motor de Reglas** (Cloud Run, sin IA, diccionario de palabras clave por división) → si no clasifica con certeza, **(2) Motor IA** (Cloud Run + Vertex AI, modelo Gemini 2.5 Flash-Lite) aplicando el umbral de confianza del 75% ya definido en la sección 10 → si tampoco alcanza el umbral, **(3) cola manual** del Repartidor, sin cambios frente a la línea base. Un motor de reglas no requiere dataset de entrenamiento, es inmediato de construir dentro del plazo de 3 meses del MVP, y sus decisiones son 100% explicables, algo relevante frente a la obligación regulatoria de justificar cómo se resuelve cada PQR (Resolución CRC 5050 de 2016, ya referenciada en la sección 3).

![Concepto de solución — arquitectura de sistemas en GCP](../images/fase_a_arquitectura_gcp.png)

*Figura. Concepto de solución de la arquitectura objetivo en GCP: clasificación en 3 niveles (reglas → IA → manual), con los componentes de datos y observabilidad como soporte transversal.*

| Componente | Servicio GCP | Rol |
|---|---|---|
| Ingesta de eventos | Pub/Sub | Desacopla el CRM heredado de los servicios de clasificación, con reintentos automáticos (mitiga el Riesgo 3, sección 10) |
| Motor de reglas | Cloud Run | Servicio sin estado; ejecuta el diccionario de palabras clave por división |
| Motor de IA | Cloud Run + Vertex AI (Gemini 2.5 Flash-Lite) | Cloud Run orquesta la llamada al modelo de lenguaje alojado en Vertex AI |
| Interfaz de validación del agente | Cloud Run + Firestore | El Agente de Soporte confirma o corrige la sugerencia; Firestore guarda el estado del caso |
| Archivo y anonimización | Cloud Storage + Cloud DLP | Cloud DLP redacta datos personales antes de reentrenar el modelo (Principio de Privacidad desde el Diseño, sección 7) |
| Analítica y KPI | BigQuery | Tablero de cobertura, exactitud y tiempo de ciclo (KPI de la sección 9) |
| Observabilidad | Cloud Monitoring / Cloud Logging | Alerta si la tasa de escalamiento a IA o a cola manual se dispara |
| Identidad y seguridad | IAM + VPC Service Controls | Un solo perímetro de cumplimiento; ningún dato de PQRS sale de GCP |

### 12.3 Presupuesto

**MVP (3 meses):**

| Ítem | Costo estimado |
|---|---|
| Científico de Datos / ML Engineer (contratista, perfil semi-senior en Colombia) **(supuesto)** | ≈ USD 7.400 |
| Ingeniero Cloud/Backend (recurso interno ya asignado; no es desembolso nuevo) | ≈ USD 2.600 (costo de oportunidad, referencial) |
| Revisión de cumplimiento (Ley 1581 / anonimización) **(supuesto)** | ≈ USD 1.500 |
| Infraestructura GCP durante el MVP (dentro de los niveles gratuitos a este volumen) | ≈ USD 100 |
| Capacitación a agentes **(supuesto)** | ≈ USD 800 |
| Contingencia (15%) | ≈ USD 1.470 |
| **Total incremental (efectivo nuevo)** | **≈ USD 11.270** |

**Operación en régimen estable (post-MVP, mensual):** infraestructura ≈ USD 50–80/mes (Pub/Sub, Cloud Run, Vertex AI, BigQuery, Monitoring) + ML Engineer a 20% de dedicación **(supuesto)** ≈ USD 490/mes → **total ≈ USD 550–570/mes (≈ USD 6.600–6.900/año)**, frente a un ahorro estimado en horas-agente de ≈ USD 71.000/mes (memoria de cálculo del ROI, sección 9).

**Conclusión:** el presupuesto real del MVP, desglosado, se ubica entre USD 11.000 y 14.000 — muy por debajo de los USD 50.000 originalmente asignados. Esto no implica que $50.000 fuera insuficiente; el problema señalado era la falta de desglose, no la magnitud. Con el desglose en mano, el comité de aprobación puede optar por mantener los USD 50.000 como margen de contingencia e iteración, o reasignar el excedente a otras iniciativas.

### 12.4 Riesgos y consideraciones de esta arquitectura

* El motor de reglas requiere mantenimiento continuo del diccionario de palabras clave a medida que cambian los productos/servicios de TelcoLatam; sin este mantenimiento, aumenta la carga sobre el Motor IA (sin impacto material en el costo, sí en la latencia).
* Los precios de Vertex AI cambian con frecuencia; el presupuesto de la sección 12.3 usa el escenario de "peor caso" (100% de los casos escalados a IA) para no depender de qué tan bien funcione el motor de reglas.
* El crédito de bienvenida de USD 300 de Google Cloud aplica solo a cuentas nuevas y por tiempo limitado; no debe tratarse como ahorro recurrente.
* Los Riesgos 3, 4 y 5 (sección 10 de este documento) siguen aplicando íntegramente a esta arquitectura.

---

### Aprobación

| Rol | Nombre | Firma | Fecha |
|---|---|---|---|
| Director de Servicio al Cliente | | | |
| Gerente de TI | | | |

# Fase B: Arquitectura de Negocio
## Marco de trabajo TOGAF — ADM (Architecture Development Method)
### Proyecto: IA-PQRS Smart Routing
**Enrutamiento inteligente de Peticiones, Quejas, Reclamos y Sugerencias (PQRS)**
**Empresa de referencia:** TelcoLatam

*Documento elaborado como continuación de la Fase A: Visión de la Arquitectura.*

---

## 1. Introducción y alcance de la Fase B

El objetivo de la Fase B del ADM de TOGAF es desarrollar la Arquitectura de Negocio objetivo que describe cómo TelcoLatam necesita operar para alcanzar las metas establecidas en la Fase A (Visión de la Arquitectura), documentando tanto la arquitectura de línea base (situación actual) como la arquitectura objetivo, e identificando las brechas, los requisitos de negocio y los componentes de la hoja de ruta necesarios para cerrarlas.

Este documento toma como entrada directa la Visión de la Arquitectura aprobada en la Fase A (proyecto "IA-PQRS Smart Routing") y el procedimiento operativo vigente "Procedimiento PQRSD", que documenta el proceso manual actual de gestión y respuesta a PQRS. A partir de ambos insumos se construye la arquitectura de negocio línea base y objetivo, los catálogos y matrices formales, el análisis de brechas, la especificación de requisitos y los componentes de la hoja de ruta que exige esta fase.

### 1.1 Trazabilidad con la Fase A

| Elemento de la Fase A | Contenido aprobado | Reflejo en esta Fase B |
|---|---|---|
| Objetivo de negocio | Reducir el tiempo promedio de primera respuesta a las PQRS en 40% en el primer trimestre | Motiva el rediseño del proceso de triage/enrutamiento (secciones 3 y 6) |
| Impulsores (drivers) | Alto volumen (>10.000 PQRS/día) y penalizaciones regulatorias por incumplimiento | Justifica la automatización de la clasificación (sección 3.1) |
| Limitaciones | Presupuesto de USD 50.000, MVP en 3 meses, cumplimiento de la Ley de Protección de Datos Personales | Delimita el alcance de la hoja de ruta (sección 8) y los requisitos RN-05 (sección 7) |
| Capacidad base / objetivo | De un proceso 100% manual (3–5 min/ticket) a un motor NLP que clasifica en milisegundos | Estructura la comparación línea base vs. objetivo (secciones 2 y 3) |
| Brechas identificadas | Falta de talento en IA e infraestructura cloud nativa | Insumo directo del análisis de brechas (sección 6) |
| Alcance (in/out) | Dentro: modelo de clasificación y enrutamiento. Fuera: respuesta automatizada al cliente | Delimita qué procesos de la línea base cambian y cuáles permanecen (sección 3.1) |
| Principios de negocio y arquitectura | El cliente es el centro de la operación; los datos son un activo; independencia tecnológica | Guían las decisiones de diseño del proceso objetivo (sección 3) |
| KPI | Accuracy > 88%, reducción de 90% en tiempo de triage, ROI en 8 meses | Métricas de éxito de la arquitectura objetivo (sección 3.1) |
| Riesgos y mitigación | Clasificación errónea (umbral de confianza 75%) y sesgo en los datos (auditoría/balanceo) | Incorporados como controles del proceso objetivo (secciones 3.4 y 7) |

## 2. Arquitectura de Negocio – Línea Base (Baseline)

### 2.1 Descripción general

Actualmente la gestión de PQRS en TelcoLatam es 100% manual: un agente lee cada ticket y lo asigna dentro del CRM, tomando en promedio de 3 a 5 minutos por caso (capacidad base declarada en la Fase A). Este proceso está formalizado en el "Procedimiento PQRSD" vigente, descrito a continuación.

### 2.2 Catálogo de Actor / Rol – Línea Base

| Rol | Responsabilidad principal | Aplicaciones requeridas |
|---|---|---|
| Repartidor | Distribuye las solicitudes internamente, hace seguimiento al vencimiento de términos, elabora el informe mensual y mantiene las plantillas de respuesta. | Suite ofimática, lector de PDF |
| Proyecta la respuesta | Analiza la solicitud, consolida la información, elabora el documento de respuesta con la plantilla oficial y lo remite a revisión. | Suite ofimática, lector de PDF |
| Revisor | Verifica que el documento cumpla los lineamientos institucionales, normativos y de forma; informa inconsistencias sin modificar el documento. | Editor de texto, lector de PDF, certificado de firma PDF |
| Jefe de División | Firma el documento final, validando que el contenido sea adecuado antes del envío al interesado. | Editor de texto, lector de PDF, certificado de firma PDF |

### 2.3 Catálogo de Proceso / Evento / Control / Producto – Línea Base

| Tipo | Elemento | Descripción |
|---|---|---|
| Evento | Recepción de PQRS | Ingreso de la solicitud por el canal de entrada |
| Proceso | Análisis y distribución | El repartidor analiza y distribuye la solicitud a la división encargada |
| Control | Verificación de competencia | Determina si la solicitud corresponde al área o debe reasignarse |
| Control | Verificación de reserva legal | Determina si procede solicitar información adicional o rechazar la petición |
| Proceso | Proyección de la respuesta | Elaboración del documento de respuesta con la plantilla oficial |
| Control | Revisión de forma y fondo | Validación de lineamientos institucionales antes de la firma |
| Control | Lineamientos de elaboración | Plantilla oficial, lenguaje neutral, dirección física, fuente Nunito 10.5–12 pt |
| Producto | Respuesta oficial firmada | Documento firmado por el Jefe de División, enviado por correo o físico |

### 2.4 Diagrama de Proceso de Negocio – Línea Base

Vista simplificada del flujo de extremo a extremo, tal como está documentada en el Procedimiento PQRSD:

![Flujo del proceso PQRSD, línea base, vista simplificada](../images/fase_b_linea_base_flujo.jpg)

*Figura 1. Flujo del proceso PQRSD (línea base) — vista simplificada.*

Vista del mismo proceso organizada por carriles de actor, que respalda el catálogo de la sección 2.2:

![Flujo del proceso PQRSD, línea base, vista por actor](../images/fase_b_linea_base_actores.jpg)

*Figura 2. Flujo del proceso PQRSD (línea base) — vista por actor (swimlane).*

### 2.5 Limitaciones observadas en la línea base

* Proceso 100% manual: cada ticket es leído y clasificado por una persona (3–5 minutos por caso).
* Sin mecanismo de priorización ni de clasificación automática de la intención del ciudadano/cliente.
* El volumen actual (>10.000 PQRS/día declarado en la Fase A) satura la capacidad de triage manual, generando los tiempos de respuesta lentos reportados por el Director de Servicio al Cliente.
* No existe un umbral de confianza ni una segunda validación automatizada; toda la trazabilidad depende de la disponibilidad del repartidor.

## 3. Arquitectura de Negocio – Objetivo (Target)

### 3.1 Descripción general

La arquitectura objetivo incorpora un motor de Procesamiento de Lenguaje Natural (NLP) que extrae la intención y las entidades del texto de la PQRS y sugiere, en milisegundos, la o las divisiones responsables (por ejemplo, Facturación y Soporte Técnico de forma simultánea cuando el caso lo requiere, como en el escenario de negocio descrito en la Fase A). El alcance aprobado en la Fase A automatiza únicamente el paso de clasificación y enrutamiento inicial; la elaboración, revisión y firma de la respuesta permanecen sin cambios, dado que la resolución o respuesta automatizada al cliente (chatbot) está explícitamente fuera de alcance.

Para mitigar el riesgo de clasificación errónea (riesgo crítico identificado en la Fase A) se incorpora un umbral de confianza del 75%: por debajo de ese umbral, el caso se enruta a la cola manual del Repartidor, tal como ocurre hoy. Adicionalmente, y para atender la preocupación de los Agentes de Soporte (miedo al reemplazo laboral y fatiga por reasignación manual) y el nivel de preparación cultural bajo/medio reportado en la Fase A, se mantiene un punto de validación humana en el que el agente puede corregir la sugerencia de la IA.

*Esta arquitectura objetivo persigue directamente los KPI definidos en la Fase A: exactitud de clasificación superior al 88%, reducción del 90% en el tiempo de triage y retorno de la inversión en 8 meses.*

🔧 **Actualización (propuesta de arquitectura de sistemas):** antes de invocar el Motor IA, un Motor de Reglas determinístico (palabras clave por división) intenta clasificar el caso. Solo cuando este primer nivel no puede determinar el área con certeza, el caso se escala al Motor IA. Esto reduce la latencia y el costo de inferencia sin cambiar el umbral de confianza del 75% ni el punto de validación humana ya descritos.

### 3.2 Catálogo de Actor / Rol – Objetivo


| Rol / Actor | Responsabilidad en la arquitectura objetivo | Aplicaciones / servicios |
|---|---|---|
|  Motor de Reglas | Servicio determinístico (sin IA) que compara el texto contra un diccionario de palabras clave por división; si la certeza es alta, enruta directo, sin invocar el Motor IA. | Microservicio de reglas (Cloud Run) |
|  Motor de Clasificación IA | Servicio automatizado que analiza el texto, extrae intención y entidades, y sugiere el área responsable con un puntaje de confianza, solo para los casos que el Motor de Reglas no resolvió. | Microservicio de inferencia NLP (API REST) |
|  Agente de Soporte | Revisa la sugerencia de la IA en los casos con confianza ≥ 75% y puede corregirla antes de la distribución. | Interfaz de validación integrada al CRM |
|  Repartidor | Atiende únicamente los casos con confianza < 75% (cola manual); conserva sus funciones de seguimiento e informes. | Suite ofimática, lector de PDF |
| Proyecta la respuesta / Revisor / Jefe de División | Sin cambios frente a la línea base: elaboran, revisan y firman la respuesta una vez recibido el caso ya enrutado. | Suite ofimática, editor de texto, certificado de firma PDF |
| Director de Servicio al Cliente | Patrocinador y stakeholder principal; valida que se cumpla el requisito de 85% de clasificación sin intervención humana. | Tablero de KPI |
| Gerente de TI | Stakeholder técnico; garantiza la integración por microservicios y APIs REST con el CRM heredado. | Plataforma de integración / APIs |
|  Científico de Datos / Ingeniero de Datos | Curan, etiquetan, auditan y balancean el dataset histórico de PQRS; entrenan y validan el modelo NLP. | Plataforma de datos / entrenamiento de modelos |
|  Arquitecto de TI | Diseña la arquitectura de microservicios desacoplada del CRM y su despliegue en la nube. | Repositorio de arquitectura |

### 3.3 Catálogo de Función y Servicio de Negocio

| Función de negocio | Servicio de negocio | Descripción |
|---|---|---|
| Gestión de PQRS | Recepción y registro de solicitudes | Ingreso del caso por el canal de entrada (sin cambios) |
| Clasificación y enrutamiento |  Clasificación por reglas (sin IA) | Servicio nuevo: primer nivel, determinístico, por palabras clave |
| Clasificación y enrutamiento |  Clasificación automática por IA | Servicio nuevo: segundo nivel, solo para lo que las reglas no resuelven |
| Clasificación y enrutamiento |  Validación humana de la sugerencia | Servicio nuevo: el agente confirma o corrige la sugerencia de la IA |
| Atención al cliente | Atención por Facturación / Soporte Técnico / Retención | Áreas receptoras del caso ya enrutado (sin cambios) |
| Elaboración y aprobación de respuestas | Proyección, revisión y firma de la respuesta | Sin cambios frente a la línea base |

### 3.4 Catálogo de Proceso / Evento / Control / Producto – Objetivo

| Tipo | Elemento | Descripción |
|---|---|---|
| Evento | Recepción de PQRS | Ingreso de la solicitud por el canal de entrada (sin cambios) |
|  Proceso | Clasificación por reglas | El motor de reglas compara el texto contra un diccionario de palabras clave por división |
|  Control | Certeza de la clasificación por reglas | Determina si el caso se enruta directo o se escala al Motor IA |
|  Proceso | Clasificación NLP | El motor de IA extrae intención y entidades y sugiere el área responsable (solo casos escalados) |
|  Control | Umbral de confianza (75%) | Determina si el caso se enruta automáticamente o pasa a cola manual |
|  Proceso | Validación del agente | El agente confirma o corrige la sugerencia antes de la distribución |
| Proceso | Distribución simultánea | Notificación a una o varias divisiones responsables en paralelo |
| Control | Verificación de competencia / reserva legal | Sin cambios frente a la línea base |
| Proceso | Proyección, revisión y firma de la respuesta | Sin cambios frente a la línea base |
| Producto | Respuesta oficial firmada | Sin cambios frente a la línea base |

### 3.5 Diagrama de Proceso de Negocio – Objetivo

![Flujo del proceso PQRS con clasificación en 3 niveles](../images/fase_b_objetivo_3_niveles.png)

*Figura 3. Flujo del proceso PQRS con clasificación en 3 niveles (arquitectura objetivo). Verde: reglas sin IA; azul: IA.*

## 4. Matriz de interacción Actor – Proceso

Participación de cada actor en los procesos de la arquitectura objetivo (X = participa):

| Actor / Rol | Recepción | Clasif. reglas | Clasif. IA | Valida agente | Distribución | Competencia/ reserva legal | Proyecta resp. | Revisión | Firma |
|---|---|---|---|---|---|---|---|---|---|
| Motor de Reglas | | X | | | | | | | |
| Motor de Clasificación IA | | | X | | | | | | |
| Agente de Soporte | | | | X | | | | | |
| Repartidor | X | | | | X | | | | |
| División responsable | | | | | X | X | | | |
| Proyecta la respuesta | | | | | | | X | | |
| Revisor | | | | | | | | X | |
| Jefe de División | | | | | | | | | X |

## 5. Matriz Función de Negocio – Organización

| Función de negocio | Área / unidad organizacional responsable |
|---|---|
| Recepción y registro de solicitudes | Canal de entrada / Atención al Cliente |
| Clasificación automática por IA | TI — Ciencia de Datos (operación del microservicio) |
| Validación humana de la sugerencia | Atención al Cliente (Agentes de Soporte) |
| Atención por área responsable | Facturación / Soporte Técnico / Retención |
| Elaboración y aprobación de respuestas | División responsable / Jefatura de División |
| Gobierno y KPI del proceso | Dirección de Servicio al Cliente |
| Integración e infraestructura | Gerencia de TI |

## 6. Análisis de brechas (Gap Analysis)

| Brecha | Estado actual (línea base) | Estado objetivo | Acción para cerrarla |
|---|---|---|---|
| Talento en IA | No existe equipo interno de ciencia/ingeniería de datos | Modelo NLP en producción, monitoreado y reentrenado | Incorporar científico(s) e ingeniero(s) de datos al proyecto (según Declaración de Trabajo de la Fase A) |
| Infraestructura | CRM on-premise, sin infraestructura cloud nativa | Microservicio de inferencia desacoplado, desplegable en la nube | Diseñar la arquitectura de microservicios con el Arquitecto de TI (Fase C/D) |
| Datos | Histórico de PQRS sin limpiar, clasificar ni gobernar | Dataset etiquetado, auditado y balanceado, gobernado como activo | Ejecutar el principio de arquitectura "los datos son un activo" antes del entrenamiento |
| Proceso | Clasificación 100% manual, sin umbral de confianza ni trazabilidad automática | Clasificación automática con umbral de confianza y validación humana | Implementar el flujo objetivo descrito en la sección 3 |
| Adopción / cultura | Preparación cultural baja/media; resistencia al cambio y temor al reemplazo laboral | Agentes con rol de validación y corrección, no de reemplazo | Mantener el punto de validación humana y capacitar a los agentes (hoja de ruta, sección 8) |

## 7. Especificación de requisitos de arquitectura (Business Requirements)

Requisitos de negocio derivados de las partes interesadas y restricciones definidas en la Fase A, y de los lineamientos vigentes del Procedimiento PQRSD:

| ID | Requisito | Fuente |
|---|---|---|
| RN-01 | El motor de clasificación debe alcanzar al menos 85% de exactitud en la asignación del área responsable sin intervención humana. | Director de Servicio al Cliente |
| RN-02 | El motor de clasificación debe exponerse como microservicio con APIs RESTful, integrado al CRM heredado. | Gerente de TI |
| RN-03 | La interfaz debe mostrar al agente el área sugerida por la IA y permitir su corrección manual. | Agentes de Soporte |
| RN-04 | Cuando la confianza del modelo sea inferior al 75%, el caso debe enrutarse a la cola manual del Repartidor. | Mitigación del Riesgo 1 (Fase A) |
| RN-05 | El tratamiento del texto de las PQRS debe cumplir la Ley de Protección de Datos Personales. | Restricciones del proyecto (Fase A) |
| RN-06 | El dataset de entrenamiento debe auditarse y balancearse para mitigar sesgos hacia ciertas demografías. | Mitigación del Riesgo 2 (Fase A) |
| RN-07 | Debe mantenerse el uso de la plantilla oficial, el lenguaje neutral y los demás lineamientos de elaboración de respuestas ya vigentes. | Procedimiento PQRSD |
| RN-08 | El desarrollo del MVP debe ejecutarse dentro de un presupuesto de USD 50.000 y un plazo de 3 meses. | Limitaciones del proyecto (Fase A) |

## 8. Componentes de la hoja de ruta de Arquitectura de Negocio

Hoja de ruta de alto nivel para el MVP, acorde con el plazo de 3 meses y el presupuesto de USD 50.000 definidos en la Fase A:

| Fase | Horizonte | Componentes principales |
|---|---|---|
| Fase 1 — Cimentación | Mes 1 | Recolección, limpieza, etiquetado, auditoría y balanceo del dataset histórico de PQRS; definición de la arquitectura del microservicio. |
| Fase 2 — Construcción | Mes 2 | Entrenamiento y validación del modelo NLP; desarrollo de la API de inferencia; diseño de la interfaz de validación para agentes. |
| Fase 3 — Integración y despliegue | Mes 3 | Integración con el CRM heredado; pruebas piloto con umbral de confianza del 75%; capacitación a agentes; despliegue del MVP y medición de KPI iniciales. |

## 9. Impacto en el panorama arquitectónico y gobernanza

### 9.1 Dominios de arquitectura impactados

* Negocio: rediseño del proceso de triage y enrutamiento de PQRS (este documento).
* Datos: gobierno, etiquetado y balanceo del histórico de PQRS (a detallar en la Fase C).
* Aplicación: microservicio de inferencia, interfaz de validación e integración con el CRM (a detallar en la Fase C).
* Tecnología: infraestructura cloud para el despliegue del microservicio (a detallar en la Fase D).

### 9.2 Riesgos heredados de la Fase A

| Riesgo | Nivel inicial | Mitigación | Nivel residual |
|---|---|---|---|
| Clasificación errónea del modelo de IA | Crítico | Umbral de confianza del 75%; por debajo, enrutamiento manual | Marginal |
| Sesgo en los datos de entrenamiento | Marginal | Auditoría y balanceo del dataset antes del entrenamiento | Insignificante |

### 9.3 Gobierno de arquitectura

Conforme a la Declaración de Trabajo de Arquitectura aprobada en la Fase A, esta Arquitectura de Negocio debe someterse a revisión y aprobación formal del Director de Servicio al Cliente y del Gerente de TI antes de dar paso a la Fase C (Arquitectura de Sistemas de Información), siguiendo el Plan de Comunicaciones de Arquitectura Empresarial definido para el proyecto.

## 10. Conclusión y aprobación para avanzar a la Fase C

Esta Fase B documenta la arquitectura de negocio línea base (proceso manual descrito en el Procedimiento PQRSD) y la arquitectura de negocio objetivo (enrutamiento asistido por IA en 3 niveles: reglas → IA → manual), junto con los catálogos, matrices, el análisis de brechas, los requisitos de negocio y la hoja de ruta necesarios para cerrar la distancia entre ambas, en línea con los objetivos, principios y KPI aprobados en la Fase A. Con esta base, el proyecto "IA-PQRS Smart Routing" queda en condiciones de avanzar hacia la Fase C: Arquitectura de Sistemas de Información.

### Aprobación

| Rol | Nombre | Firma | Fecha |
|---|---|---|---|
| Director de Servicio al Cliente | | | |
| Gerente de TI | | | |


# Fase C: Arquitectura de Sistemas de Información
## Marco de trabajo TOGAF — ADM (Architecture Development Method)
### Proyecto: IA-PQRS Smart Routing
**Enrutamiento inteligente de Peticiones, Quejas, Reclamos y Sugerencias (PQRS)**
**Empresa de referencia:** TelcoLatam

*Documento elaborado como continuación de la Fase B: Arquitectura de Negocio, e informado directamente por el concepto de solución técnica desarrollado en la Fase A: Visión de la Arquitectura (sección 12).*

> **Nota de versión.** Esta es una revisión de la Fase C que incorpora la arquitectura real en GCP y la clasificación en 3 niveles (reglas → IA → manual) confirmadas en la Fase A y en la actualización de la Fase B. Los cambios frente a la versión anterior de este documento están resumidos en la sección 0.

---

## 0. Registro de cambios aplicados en esta revisión

| # | Cambio | Motivo |
|---|---|---|
| 1 | Se incorporó el **Motor de Reglas** (Cloud Run, sin IA) como primer nivel de clasificación, antes del Motor IA | Confirmado en la Fase A (sección 12.1) y en la actualización de la Fase B (sección 3.1) |
| 2 | Se reemplazó la arquitectura "cloud-agnóstica" genérica por la arquitectura real en **Google Cloud Platform (GCP)**: Pub/Sub, Cloud Run, Vertex AI (Gemini 2.5 Flash-Lite), Firestore, Cloud Storage + Cloud DLP, BigQuery, Cloud Monitoring/Logging, IAM + VPC Service Controls | Especificada en la Fase A, sección 12.2 |
| 3 | Se reemplazó la estimación de costos propia (≈USD 43.500, basada en supuestos de mercado) por el **desglose oficial de la Fase A** (≈USD 11.270 de inversión incremental para el MVP) | La Fase A, sección 12.3, ya trae una memoria de cálculo validada internamente; mantener una estimación paralela habría sido incoherente |
| 4 | Se incorporaron los **Riesgos 3, 4 y 5** de la Fase A (integración con el CRM heredado, resistencia al cambio, cumplimiento de la Ley 1581 de 2012) al gobierno de esta fase | Estaban documentados en la Fase A pero no reflejados aún en la arquitectura de sistemas |
| 5 | Se especificó la jurisdicción legal — **Colombia: Ley 1581 de 2012 (Habeas Data), Decreto 1377 de 2013 y Resolución CRC 5050 de 2016** — en los requisitos de sistema y en el gobierno de datos | La versión anterior hablaba de "la Ley de Protección de Datos Personales" sin especificar país |
| 6 | Se actualizaron el catálogo de aplicaciones, el catálogo de interfaces y el diagrama de arquitectura objetivo para reflejar los componentes reales de GCP en lugar de nombres genéricos ("API Gateway", "microservicio de inferencia") | Consistencia con el concepto de solución de la Fase A |

---

## 1. Introducción y alcance de la Fase C

La Fase C del ADM de TOGAF se centra en la Arquitectura de Aplicaciones y Datos: establece cómo el software y la información habilitan las capacidades de negocio definidas en la Fase B. Su objetivo no es diseñar una única aplicación, sino construir el mapa completo del ecosistema de software que soporta el proceso de gestión de PQRS y permite la toma de decisiones estratégica.

La arquitectura de aplicaciones debe servir directamente a la Arquitectura de Negocio definida en la Fase B, y materializar el concepto de solución ya esbozado en la Fase A. Sin esta alineación, la TI se convierte en un bloqueador en lugar de un motor de transformación: por eso cada componente de este documento se conecta explícitamente con un rol, un proceso o un requisito ya aprobado en las fases anteriores.

### 1.1 Trazabilidad con las Fases A y B

| Elemento | Fuente | Contenido aprobado | Reflejo en esta Fase C |
|---|---|---|---|
| Catálogo de actores objetivo (incluye Motor de Reglas) | Fase B, sección 3.2 | Motor de Reglas, Motor de Clasificación IA, Agente de Soporte, Científico/Ingeniero de Datos, Arquitecto de TI | Motiva el catálogo de aplicaciones objetivo (sección 3) |
| Concepto de solución en GCP | Fase A, sección 12.2 | Clasificación en 3 niveles (reglas → IA → manual) sobre Pub/Sub, Cloud Run, Vertex AI, Firestore, Cloud Storage + DLP, BigQuery, Monitoring/Logging, IAM + VPC SC | Base directa de la arquitectura de aplicaciones objetivo (sección 3) y de datos (sección 4) |
| Presupuesto desglosado del MVP | Fase A, sección 12.3 | Inversión incremental ≈ USD 11.270; operación estable ≈ USD 550-570/mes | Anexo A de este documento |
| Riesgos 3, 4 y 5 | Fase A, sección 10 | Integración con el CRM heredado; resistencia al cambio; incumplimiento de la Ley 1581 de 2012 | Sección 9.3 (riesgos técnicos) |
| Jurisdicción legal | Fase A, secciones 3 y 7 | Ley 1581 de 2012, Decreto 1377 de 2013, Resolución CRC 5050 de 2016 | Sección 4.3 (gobierno de datos) y requisito RS-05 |
| Requisitos de negocio (RN-01 a RN-08) | Fase B, sección 7 | Exactitud, integración REST, umbral de confianza, protección de datos, presupuesto | Traducidos a requisitos de sistema RS-01 a RS-10 (sección 7) |
| Hoja de ruta de negocio (3 meses) | Fase B, sección 8 | Cimentación, construcción, integración y despliegue | Detallada con componentes técnicos de GCP (sección 8) |

---

## 2. Arquitectura de Aplicaciones — Línea Base

### 2.1 Descripción general

Los sistemas actuales soportan el proceso 100% manual descrito en el Procedimiento PQRSD y en la Fase B: un agente registra y asigna cada ticket en el CRM heredado, y el resto del ciclo (elaboración, revisión y firma) se apoya en herramientas ofimáticas independientes, sin ninguna integración automatizada entre ellas.

### 2.2 Catálogo de Aplicaciones — Línea Base

| Aplicación | Tipo | Función en el proceso | Usada por |
|---|---|---|---|
| CRM heredado (on-premise) | Sistema de registro | Registro y asignación manual de la PQRS | Repartidor |
| Suite ofimática | Productividad | Elaboración de la respuesta con la plantilla oficial | Proyecta la respuesta |
| Lector de PDF | Utilitario | Consulta de solicitudes y anexos | Todos los roles |
| Editor de texto + certificado de firma digital | Firma electrónica | Revisión y firma del documento final | Revisor, Jefe de División |
| Repositorio de plantillas | Almacenamiento documental | Custodia de la plantilla oficial de respuesta | Repartidor (mantiene), Proyectista (usa) |

### 2.3 Catálogo de Interfaces — Línea Base

No existen interfaces automatizadas entre los sistemas actuales; toda transferencia de información entre aplicaciones es manual.

| Origen | Destino | Mecanismo | ¿Automatizada? |
|---|---|---|---|
| CRM heredado | Suite ofimática | Copiar/pegar manual del caso asignado | No |
| Suite ofimática | Editor de texto / firma digital | Envío de archivo por correo interno | No |
| Repositorio de plantillas | Suite ofimática | Descarga manual de la plantilla en blanco | No |

### 2.4 Diagrama de Arquitectura de Aplicaciones — Línea Base

![Arquitectura de aplicaciones — línea base](../images/fase_c_linea_base.png)

*Figura 1. Arquitectura de aplicaciones de la línea base — sin integración automatizada entre sistemas.*

### 2.5 Limitaciones observadas en la línea base

* El CRM heredado no expone capacidad nativa de integración vía API (aunque la Fase A señala que sí cuenta con APIs, la infraestructura sigue siendo on-premise, sección 5).
* No existe un repositorio de datos gobernado: el histórico de PQRS está disperso y sin etiquetar.
* No hay ningún mecanismo de clasificación automática ni de monitoreo del desempeño del proceso.
* Cada paso depende de la ejecución manual de una persona, sin trazabilidad de sistema.

---

## 3. Arquitectura de Aplicaciones — Objetivo

### 3.1 Descripción general

La clasificación de una PQRS pasa a tener **tres niveles**, tal como se definió en la Fase A (sección 12.1) y se ratificó en la actualización de la Fase B (sección 3.1):

1. **Motor de Reglas** (Cloud Run, sin IA): compara el texto contra un diccionario de palabras clave por división. Si la certeza es alta, enruta directo, sin invocar el Motor IA.
2. **Motor IA** (Cloud Run + Vertex AI, modelo Gemini 2.5 Flash-Lite): solo se invoca cuando el Motor de Reglas no puede determinar el área con certeza. Aplica el umbral de confianza del 75% ya definido en la Fase A.
3. **Cola manual** (Repartidor): recibe los casos que ni las reglas ni la IA lograron clasificar con certeza suficiente, sin cambios frente a la línea base.

Este diseño en tres niveles no es una preferencia estética: un motor de reglas no requiere dataset de entrenamiento, es inmediato de construir dentro del plazo de 3 meses del MVP, reduce la latencia y el costo de inferencia, y sus decisiones son 100% explicables — algo relevante frente a la obligación regulatoria de justificar cómo se resuelve cada PQR (Resolución CRC 5050 de 2016, art. 2.1.24.3).

Las aplicaciones de elaboración, revisión y firma de la respuesta permanecen sin cambios, en línea con el alcance aprobado en la Fase A.

### 3.2 Modelos de Arquitectura de Aplicaciones aplicados al proyecto

| Modelo | Qué evalúa | Aplicación al proyecto IA-PQRS |
|---|---|---|
| Modelo Funcional | Distribución de costos operativos y de capital; identifica aplicaciones que consumen recursos sin generar valor diferencial | El CRM heredado y la suite ofimática son de bajo valor diferencial: se conservan tal cual (costo hundido); la inversión incremental se concentra en el Motor de Reglas y el Motor IA |
| Modelo de Desarrollo | Enfoque de construcción: desarrollo interno, compra externa o integración de plataformas existentes | Vertex AI (Gemini 2.5 Flash-Lite) se **compra** como servicio administrado; el Motor de Reglas y la Interfaz de Validación se **desarrollan a medida** |
| Modelo de Integración | Nivel de rigidez o fluidez de las interfaces; capacidad de adaptación del sistema | Pub/Sub desacopla el CRM heredado de los servicios de clasificación, con reintentos automáticos ante indisponibilidad (mitiga el Riesgo 3 de la Fase A) |
| Modelo de Producto | Valor competitivo único; identifica aplicaciones que generan diferenciación | El activo diferenciador es el diccionario de reglas por división y el dataset propio con el que eventualmente se afine el modelo, no la infraestructura genérica que los rodea |

### 3.3 Estrategia de portafolio: comprar vs. desarrollar

* **Comprar (SaaS / servicio administrado):** Vertex AI con el modelo Gemini 2.5 Flash-Lite para la clasificación de segundo nivel — es una capacidad genérica de lenguaje, no diferenciadora por sí misma.
* **Desarrollar a medida:** el Motor de Reglas (diccionario de palabras clave propio de TelcoLatam) y la Interfaz de Validación embebida en el flujo del agente — aquí está el conocimiento específico del negocio.
* **Plataforma de integración (administrada):** Pub/Sub como capa de desacople entre el CRM heredado y los servicios de clasificación.

### 3.4 Catálogo de Aplicaciones — Objetivo

| Aplicación | Servicio GCP | Estado | Función en el proceso |
|---|---|---|---|
| Ingesta de eventos | Pub/Sub | Nuevo | Desacopla el CRM heredado de los servicios de clasificación, con reintentos automáticos |
| Motor de Reglas | Cloud Run | Nuevo | Servicio sin estado; ejecuta el diccionario de palabras clave por división (primer nivel) |
| Motor IA | Cloud Run + Vertex AI (Gemini 2.5 Flash-Lite) | Nuevo | Clasifica por NLP los casos que el Motor de Reglas no resolvió (segundo nivel) |
| Interfaz de Validación | Cloud Run + Firestore | Nuevo | El Agente de Soporte confirma o corrige la sugerencia; Firestore guarda el estado del caso |
| Archivo y anonimización | Cloud Storage + Cloud DLP | Nuevo | Cloud DLP redacta datos personales antes de usar el histórico para reentrenar el modelo |
| Analítica y KPI | BigQuery | Nuevo | Tablero de cobertura, exactitud y tiempo de ciclo |
| Observabilidad | Cloud Monitoring / Cloud Logging | Nuevo | Alerta si la tasa de escalamiento a IA o a cola manual se dispara |
| Identidad y seguridad | IAM + VPC Service Controls | Nuevo | Perímetro único de cumplimiento; ningún dato de PQRS sale de GCP |
| CRM heredado | (on-premise, existente) | Sin cambios (extendido) | Continúa como sistema de registro y distribución del caso |
| Suite ofimática + firma digital | (existente) | Sin cambios | Elaboración, revisión y firma de la respuesta |

### 3.5 Catálogo de Interfaces — Objetivo

| Interfaz | Origen → Destino | Mecanismo | Datos intercambiados |
|---|---|---|---|
| Evento de ingesta | CRM heredado → Pub/Sub | Mensajería asíncrona, con reintentos | Texto de la PQRS |
| Clasificación por reglas | Pub/Sub → Motor de Reglas (Cloud Run) | REST / evento | Texto de la PQRS |
| Escalamiento a IA | Motor de Reglas → Motor IA (Vertex AI) | REST / JSON | Texto de la PQRS (solo si las reglas no resuelven) |
| Resultado de clasificación | Motor IA → Interfaz de Validación | REST / JSON | Área sugerida + puntaje de confianza |
| Confirmación del agente | Interfaz de Validación (Firestore) → CRM heredado | REST / evento | Caso enrutado y división asignada |
| Entrenamiento | Cloud Storage (dataset anonimizado por DLP) → Motor IA | Batch / ETL | Dataset etiquetado y anonimizado |
| Monitoreo | Motor de Reglas y Motor IA → Cloud Monitoring/Logging | Streaming / logs | Métricas de desempeño y tasa de escalamiento |
| Analítica | Interfaz de Validación → BigQuery | Streaming / batch | Métricas de cobertura, exactitud y tiempo de ciclo |

### 3.6 Diagrama de Arquitectura de Aplicaciones — Objetivo

![Arquitectura de aplicaciones objetivo en GCP — clasificación en 3 niveles](../images/fase_c_objetivo_gcp.png)

*Figura 2. Arquitectura de aplicaciones objetivo en GCP. En ámbar, los componentes de clasificación (reglas e IA); en azul claro, los componentes de datos, analítica y observabilidad.*

---

## 4. Arquitectura de Datos

### 4.1 Catálogo de Entidades de Datos

| Entidad | Descripción | Aplicación de origen |
|---|---|---|
| PQRS (texto original) | Solicitud recibida del cliente | Canal de entrada / CRM heredado |
| Resultado del Motor de Reglas | Área sugerida (o "sin certeza") | Motor de Reglas (Cloud Run) |
| Clasificación del Motor IA | Área sugerida + puntaje de confianza | Motor IA (Vertex AI) |
| Decisión del agente | Confirmación o corrección de la sugerencia | Interfaz de Validación (Firestore) |
| Caso enrutado | PQRS con la división ya asignada | CRM heredado |
| Respuesta oficial | Documento firmado enviado al cliente | Suite ofimática / firma digital |
| Dataset de entrenamiento (anonimizado) | Histórico de PQRS etiquetado, auditado, balanceado y anonimizado | Cloud Storage + Cloud DLP |
| Registro de auditoría y métricas | Trazabilidad y desempeño del modelo en producción | BigQuery / Cloud Monitoring |

### 4.2 Matriz Entidad de Datos — Aplicación

C = Crea · L = Lee · A = Actualiza

| Entidad de datos | Motor de Reglas | Motor IA | Interfaz Validación (Firestore) | Cloud Storage + DLP | BigQuery |
|---|---|---|---|---|---|
| PQRS (texto original) | L | L | L | L | |
| Resultado del Motor de Reglas | C | L | L | | L |
| Clasificación del Motor IA | | C | L / A | | L |
| Decisión del agente | | | C | L | |
| Dataset de entrenamiento | | L | | C / A | |
| Registro de auditoría | | C | | | C / L |

### 4.3 Principios de gobierno de datos y cumplimiento

* **Jurisdicción y marco legal:** el tratamiento del texto de las PQRS debe cumplir la **Ley 1581 de 2012** (Habeas Data) y su **Decreto reglamentario 1377 de 2013** (Colombia). El texto debe anonimizarse o seudonimizarse mediante **Cloud DLP** antes de almacenarse en Cloud Storage para entrenamiento (RS-05).
* **Validación previa por el Oficial de Protección de Datos:** conforme al mapa de partes interesadas de la Fase A (sección 2.2), el Oficial de Protección de Datos/Cumplimiento debe validar el proceso de anonimización antes de iniciar cualquier entrenamiento o reentrenamiento del modelo.
* **Principio "los datos son un activo"** (heredado de la Fase A, sección 7): gobierno formal del dataset mediante auditoría y balanceo antes de cada reentrenamiento (RN-06).
* **Comité de gobierno de datos/IA:** la Fase A (sección 5) señala que aún falta constituir este comité, encargado de aprobar el uso del histórico de PQRS para entrenamiento; se recomienda constituirlo antes del Mes 2 de la hoja de ruta (sección 8).
* **Retención y trazabilidad:** el registro de auditoría en BigQuery debe conservarse el tiempo suficiente para sustentar decisiones de reentrenamiento y eventuales requerimientos de la Superintendencia de Industria y Comercio.

---

## 5. Matriz Aplicación — Función de Negocio

| Función de negocio (Fase B) | Aplicación que la soporta |
|---|---|
| Recepción y registro de solicitudes | CRM heredado → Pub/Sub |
| Clasificación por reglas (primer nivel) | Motor de Reglas (Cloud Run) |
| Clasificación automática por IA (segundo nivel) | Motor IA (Cloud Run + Vertex AI) |
| Validación humana de la sugerencia | Interfaz de Validación (Cloud Run + Firestore) |
| Atención por área responsable | CRM heredado (distribución) |
| Elaboración y aprobación de respuestas | Suite ofimática + firma digital |
| Gobierno y KPI del proceso | BigQuery (tablero de métricas) |
| Integración e infraestructura | Pub/Sub, IAM + VPC Service Controls |

---

## 6. Análisis de Brechas (Aplicaciones y Datos)

| Brecha | Estado actual | Estado objetivo | Acción para cerrarla |
|---|---|---|---|
| Integración | Sin integración automatizada entre sistemas | Pub/Sub desacopla el CRM de los servicios de clasificación, con reintentos automáticos | Diseñar los tópicos y suscripciones de Pub/Sub (Fase D detalla el dimensionamiento) |
| Clasificación automática | No existe ningún mecanismo de clasificación | Clasificación en 3 niveles: reglas → IA (Vertex AI) → manual | Construir el diccionario de reglas y configurar el Motor IA con el umbral de confianza |
| Gobierno de datos | Dataset disperso, sin etiquetar ni anonimizar | Dataset etiquetado, auditado, balanceado y anonimizado (Cloud DLP) en Cloud Storage | Implementar el pipeline de curaduría y anonimización de datos |
| Monitoreo | Sin trazabilidad automática del desempeño | Cloud Monitoring/Logging + BigQuery con métricas de cobertura, exactitud y tiempo de ciclo | Desplegar los tableros y alertas de observabilidad |
| Cumplimiento legal | Sin comité de gobierno de datos/IA ni proceso formal de anonimización | Comité constituido y pipeline de anonimización validado por el Oficial de Protección de Datos | Constituir el comité y ejecutar la validación antes del entrenamiento (Fase A, sección 5) |

---

## 7. Especificación de Requisitos de Sistemas (RS)

| ID | Requisito de sistema | Requisito de negocio relacionado |
|---|---|---|
| RS-01 | El Motor IA (Vertex AI, Gemini 2.5 Flash-Lite) debe responder en el orden de milisegundos y soportar un volumen superior a 10.000 solicitudes/día | RN-01 |
| RS-02 | El Motor de Reglas y el Motor IA deben exponerse como servicios Cloud Run, consumidos vía Pub/Sub con reintentos automáticos | RN-02 |
| RS-03 | La Interfaz de Validación (Cloud Run + Firestore) debe mostrar el área sugerida y el puntaje de confianza, y permitir la corrección manual en un clic | RN-03 |
| RS-04 | El sistema debe enrutar automáticamente a la cola manual del Repartidor cuando ni el Motor de Reglas ni el Motor IA alcancen la certeza/confianza requerida (< 75%) | RN-04 |
| RS-05 | El pipeline de datos debe anonimizar o seudonimizar los datos personales del texto de la PQRS mediante Cloud DLP, conforme a la Ley 1581 de 2012 y el Decreto 1377 de 2013 | RN-05 |
| RS-06 | BigQuery debe registrar métricas de sesgo por segmento y alertar desviaciones significativas antes de cada reentrenamiento | RN-06 |
| RS-07 | El CRM heredado y la suite ofimática deben conservar sin modificaciones la plantilla oficial y los lineamientos vigentes | RN-07 |
| RS-08 | La arquitectura debe operar dentro de servicios cloud de pago por uso (Cloud Run, Vertex AI, Pub/Sub, BigQuery), ajustándose al presupuesto y plazo del MVP (ver Anexo A) | RN-08 |
| RS-09 | El Motor de Reglas debe resolver el caso sin invocar el Motor IA cuando la certeza sea alta, para reducir la latencia y el costo de inferencia | Fase A, sección 12.1 |
| RS-10 | La ingesta vía Pub/Sub debe incluir reintentos automáticos y una cola de contingencia hacia el Repartidor si el CRM heredado no responde a tiempo | Mitigación del Riesgo 3 (Fase A) |

---

## 8. Componentes de la Hoja de Ruta (Aplicaciones y Datos)

| Fase | Horizonte | Componentes de aplicaciones y datos (GCP) |
|---|---|---|
| Fase 1 — Cimentación | Mes 1 | Diseño de tópicos/suscripciones de Pub/Sub; construcción del diccionario de reglas por división; diseño del esquema de BigQuery y del dataset en Cloud Storage; constitución del comité de gobierno de datos/IA |
| Fase 2 — Construcción | Mes 2 | Despliegue del Motor de Reglas (Cloud Run); configuración del Motor IA (Vertex AI, Gemini 2.5 Flash-Lite) con el umbral de confianza; desarrollo de la Interfaz de Validación (Cloud Run + Firestore); pipeline de anonimización con Cloud DLP |
| Fase 3 — Integración y despliegue | Mes 3 | Integración Pub/Sub ↔ CRM heredado; despliegue piloto con clasificación en 3 niveles; configuración de Cloud Monitoring/Logging e IAM + VPC Service Controls; capacitación a agentes; medición de KPI iniciales en BigQuery |

---

## 9. Rol del Arquitecto y Gobernanza

### 9.1 Roles en la Fase C

| Rol | Responsabilidades en esta fase |
|---|---|
| Arquitecto Empresarial | Crea marcos técnicos y patrones de diseño; establece estándares de desarrollo; define arquitecturas de referencia; gestiona el portafolio de aplicaciones; protege el valor empresarial |
| Arquitecto de Aplicaciones (Arquitecto de TI, catálogo Fase B) | Sirve como puente entre dominios; alinea la estrategia con la ejecución; diseña la arquitectura de microservicios en GCP definida en la Fase A |

### 9.2 Dominios de arquitectura impactados

* **Negocio:** sin cambios adicionales frente a lo definido en la Fase B.
* **Datos:** gobierno, etiquetado, balanceo y anonimización del histórico de PQRS — detallado en la sección 4.
* **Aplicación:** Motor de Reglas, Motor IA, Interfaz de Validación e integración vía Pub/Sub — detallado en la sección 3.
* **Tecnología:** el proveedor cloud ya está definido (GCP); queda para la Fase D el dimensionamiento detallado (regiones, políticas de escalamiento, continuidad/DR, hardening de seguridad).

### 9.3 Riesgos técnicos (heredados de la Fase A, sección 10)

| Riesgo | Nivel inicial | Mitigación | Nivel residual |
|---|---|---|---|
| Integración con el CRM heredado (límites de tasa o indisponibilidad de sus APIs) | Alto | Pub/Sub como capa desacoplada, con reintentos automáticos y cola de contingencia hacia el Repartidor | Moderado |
| Resistencia al cambio / baja adopción por parte de los agentes | Alto | Comunicar que el rol del agente es de validación, no de reemplazo; capacitación temprana; involucrar a los agentes en el diseño de la Interfaz de Validación | Bajo |
| Incumplimiento de la Ley 1581 de 2012 en el tratamiento de datos de entrenamiento | Alto | Anonimización con Cloud DLP y validación previa del Oficial de Protección de Datos | Bajo |
| Mantenimiento del diccionario de reglas (queda desactualizado si cambian productos/servicios) | Moderado | Revisión periódica del diccionario; monitoreo de la tasa de escalamiento al Motor IA como señal de alerta | Bajo |

### 9.4 Gobierno de arquitectura

Esta Arquitectura de Sistemas de Información debe someterse a revisión y aprobación formal del Gerente de TI y del Director de Servicio al Cliente antes de avanzar a la Fase D (Arquitectura Tecnológica), donde se detallará el dimensionamiento de la infraestructura ya definida en GCP (regiones, políticas de escalamiento, continuidad y hardening de seguridad), siguiendo el Plan de Comunicaciones de Arquitectura Empresarial definido para el proyecto.

---

## 10. Conclusión y Aprobación para Avanzar a la Fase D

Esta Fase C documenta la arquitectura de aplicaciones línea base (sistemas manuales y desconectados descritos en el Procedimiento PQRSD) y la arquitectura de aplicaciones y datos objetivo — clasificación en 3 niveles (reglas → IA → manual) sobre servicios administrados de GCP —, junto con los catálogos, matrices, el análisis de brechas, los requisitos de sistemas y la hoja de ruta necesarios para cerrar la distancia entre ambas, en línea con la Arquitectura de Negocio de la Fase B y el concepto de solución de la Fase A.

Con esta base, el proyecto "IA-PQRS Smart Routing" queda en condiciones de avanzar hacia la Fase D: Arquitectura Tecnológica, donde se detallará el dimensionamiento de la infraestructura GCP ya seleccionada.

### Aprobación

| Rol | Nombre | Firma | Fecha |
|---|---|---|---|
| Gerente de TI | | | |
| Director de Servicio al Cliente | | | |

---

## Anexo A. Estimación de Costos del MVP

Esta sección reemplaza la estimación independiente que traía una versión anterior de este documento (≈USD 43.500, basada en supuestos genéricos de mercado). En su lugar, se usa el desglose oficial ya desarrollado en la Fase A (sección 12.3), que es más preciso porque está anclado a los servicios reales de GCP definidos en la sección 3 de este documento.

### A.1 Desglose de costos — MVP (3 meses)

| Ítem | Costo estimado (USD) | Notas |
|---|---|---|
| Científico de Datos / ML Engineer (contratista, perfil semi-senior en Colombia) | ≈ 7.400 | (supuesto, Fase A) |
| Ingeniero Cloud/Backend | ≈ 2.600 | Recurso interno ya asignado; costo de oportunidad, no desembolso nuevo |
| Revisión de cumplimiento (Ley 1581 / anonimización) | ≈ 1.500 | (supuesto, Fase A) |
| Infraestructura GCP durante el MVP | ≈ 100 | Dentro de los niveles gratuitos a este volumen |
| Capacitación a agentes | ≈ 800 | (supuesto, Fase A) |
| Contingencia (15%) | ≈ 1.470 | |
| **Total incremental (efectivo nuevo)** | **≈ 11.270** | |

**Operación en régimen estable (post-MVP, mensual):** infraestructura ≈ USD 50-80/mes (Pub/Sub, Cloud Run, Vertex AI, BigQuery, Monitoring) + ML Engineer a 20% de dedicación (supuesto) ≈ USD 490/mes → total ≈ **USD 550-570/mes** (≈ USD 6.600-6.900/año).

### A.2 Verificación de coherencia con el presupuesto

| Concepto | Monto (USD) |
|---|---|
| Total estimado del MVP (desglose oficial, Fase A) | 11.270 |
| Presupuesto aprobado en la Fase A / RN-08 | 50.000 |
| Margen restante | 38.730 (≈ 77%) |

El presupuesto real del MVP, desglosado, se ubica entre USD 11.000 y 14.000 — muy por debajo de los USD 50.000 originalmente asignados. **Esto no implica que los USD 50.000 fueran insuficientes: el problema original era la falta de desglose, no la magnitud.** Con el desglose en mano, el comité de aprobación puede optar por: (a) mantener los USD 50.000 como margen de contingencia e iteración (por ejemplo, para ajustar el umbral de confianza durante el piloto o ampliar el alcance del diccionario de reglas), o (b) reasignar el excedente a otras iniciativas del portafolio de TI.

**Nota:** el escenario de costos usa el "peor caso" (100% de los casos escalados al Motor IA); en la práctica, cuanto mejor funcione el Motor de Reglas, menor será el consumo de Vertex AI y, por tanto, el costo real. El crédito de bienvenida de USD 300 de Google Cloud no debe tratarse como ahorro recurrente, ya que aplica solo a cuentas nuevas y por tiempo limitado.

# Fase D — Arquitectura Tecnológica

## 1. Información del documento

| Campo | Valor |
|---|---|
| Proyecto | IA-PQRS Smart Routing |
| Empresa de referencia | TelcoLatam |
| Fase ADM | Fase D — Arquitectura Tecnológica |
| Fases de entrada | Fase A (Visión de la Arquitectura), Fase B (Arquitectura de Negocio), Fase C (Arquitectura de Sistemas de Información) |
| Proveedor cloud | Google Cloud Platform (GCP) — decisión ya tomada en la Fase A, sección 12 |
| Estado | Borrador para revisión del Gerente de TI y el Director de Servicio al Cliente |
| Fecha | 2026-09-29 |
| Autor | Arquitectura Empresarial — continuación documental de las Fases A, B y C |

## 2. Objetivo

Definir la arquitectura tecnológica objetivo (infraestructura, plataformas, redes, seguridad, integración y operación) que soporta la clasificación de PQRS en 3 niveles (reglas → IA → manual) establecida en la Fase A (sección 12) y detallada como arquitectura de aplicaciones y datos en la Fase C. Esta fase traduce el concepto de solución en GCP ya aprobado en un diseño técnico dimensionado: selección de región, patrones de cómputo y almacenamiento, conectividad con el CRM heredado on-premise, controles de seguridad e identidad, observabilidad, CI/CD y continuidad operativa — de modo que el proyecto quede listo para pasar de la arquitectura a la implementación (Fase E).

## 3. Alcance

**Dentro del alcance de esta fase:**
* Dimensionamiento técnico de los componentes GCP ya catalogados en la Fase C (Pub/Sub, Cloud Run, Vertex AI, Firestore, Cloud Storage + Cloud DLP, BigQuery, Cloud Monitoring/Logging, IAM + VPC Service Controls).
* Diseño de la conectividad entre el CRM heredado on-premise y GCP.
* Arquitectura de seguridad, identidad y gestión de secretos.
* Estrategia de CI/CD, Infraestructura como Código (IaC) y gobierno de configuración.
* Estrategia de resiliencia, alta disponibilidad, continuidad y recuperación ante desastres (DR).
* Catálogo de estándares y principios tecnológicos, análisis de brechas técnicas y hoja de ruta de implementación.

**Fuera del alcance de esta fase:**
* Rediseño del CRM heredado o de las herramientas ofimáticas usadas por Proyectista/Revisor/Jefe de División (sin cambios, Fase C sección 3.1).
* Migración del CRM a la nube (permanece on-premise; solo se diseña su conectividad).
* Selección o entrenamiento detallado del modelo de IA (cubierto en la Fase C, aquí solo se dimensiona la plataforma que lo aloja: Vertex AI).
* Fase E (Oportunidades y Soluciones) y Fase F (Planificación de la Migración), que consumirán esta arquitectura como insumo.

## 4. Contexto arquitectónico

### 4.1 Arquitectura Baseline

La línea base tecnológica es la ya descrita en las Fases A (sección 5) y C (sección 2): un CRM heredado on-premise con capacidad de exponer APIs pero sin infraestructura cloud nativa, sin integración automatizada entre sistemas, sin gobierno de datos, sin observabilidad ni CI/CD formal. El detalle de hardware, topología de red y versiones de software de este entorno **no está disponible** en la documentación de fases anteriores (`TBD` — ver sección 21).

### 4.2 Arquitectura Target

La arquitectura objetivo despliega los servicios administrados de GCP ya seleccionados en la Fase A/C sobre una región primaria de Sudamérica, con conectividad privada hacia el CRM heredado, un perímetro de seguridad único (VPC Service Controls), IAM de mínimo privilegio, observabilidad extremo a extremo y una estrategia de resiliencia que se apoya explícitamente en el proceso manual ya existente como mecanismo de degradación segura ante fallas (sección 7.14). El detalle completo se desarrolla en la sección 7.

### 4.3 Dependencias con las fases anteriores

| Fase | Insumo que consume esta Fase D |
|---|---|
| Fase A, sección 12 | Selección de proveedor (GCP), concepto de solución en 3 niveles, presupuesto desglosado (≈USD 11.270 MVP / ≈USD 550-570 mes operación), Riesgos 3-5, jurisdicción legal (Ley 1581 de 2012, Decreto 1377 de 2013) |
| Fase B, secciones 3 y 7 | Catálogo de actores objetivo (Motor de Reglas, Motor IA, Agente de Soporte), requisitos de negocio RN-01 a RN-08 |
| Fase C, secciones 3, 4 y 7 | Catálogo de aplicaciones y de interfaces objetivo, catálogo de entidades de datos, requisitos de sistema RS-01 a RS-10, riesgos técnicos preliminares (sección 9.3), y el señalamiento explícito de que "queda para la Fase D el dimensionamiento detallado (regiones, políticas de escalamiento, continuidad/DR, hardening de seguridad)" (Fase C, sección 9.2) |

### 4.4 Principios arquitectónicos aplicables

Se heredan de la Fase A (sección 7) los principios de **Privacidad desde el Diseño**, **Independencia tecnológica** y **Preferencia por servicios administrados en la nube (*cloud-first*)**, y de la Fase C el criterio de **comprar vs. desarrollar** (sección 3.3). Esta fase añade los principios tecnológicos específicos detallados en la sección 11.

### 4.5 Restricciones

* Presupuesto: ≈USD 11.270 de inversión incremental para el MVP y ≈USD 550-570/mes en operación estable (Fase A, sección 12.3; Fase C, Anexo A) — toda decisión de esta fase debe caber dentro de ese marco o justificar explícitamente una excepción.
* Plazo: MVP en 3 meses (Fase A, sección 3; RN-08).
* Jurisdicción legal: Colombia — Ley 1581 de 2012, Decreto 1377 de 2013, y Resolución CRC 5050 de 2016 para tiempos de respuesta (Fase A/C).
* GCP no tiene una región en Colombia; debe seleccionarse una región sudamericana disponible (sección 7.3).
* El CRM heredado permanece on-premise; no se migra en este proyecto.

### 4.6 Supuestos

* **Supuesto:** el CRM heredado puede realizar llamadas salientes HTTPS (egress a internet) o, alternativamente, la red de TelcoLatam puede establecer un túnel VPN hacia GCP. No se confirma cuál de las dos condiciones aplica — ver ADR-02 (sección 15) y TBD (sección 21).
* **Supuesto:** TelcoLatam no cuenta hoy con un proveedor de identidad (IdP) corporativo federable a Google Cloud Identity; se asume Google Workspace/Cloud Identity como directorio para el acceso de agentes, sujeto a confirmación.
* **Supuesto:** el volumen de >10.000 PQRS/día (Fase A) se distribuye de forma no uniforme durante el horario laboral, sin picos que excedan varias veces ese promedio; no se dispone de un perfil de tráfico horario real.
* **Supuesto:** no existen hoy requisitos regulatorios de residencia de datos que exijan que el texto de las PQRS permanezca físicamente en territorio colombiano (la Ley 1581 regula el tratamiento, no exige residencia física); se recomienda validar este punto con el Oficial de Protección de Datos antes de la implementación.

## 5. Requisitos tecnológicos

### 5.1 Requisitos funcionales tecnológicos

| ID | Requisito | Origen |
|---|---|---|
| RT-01 | La plataforma debe recibir el texto de la PQRS desde el CRM heredado y publicarlo como evento en Pub/Sub, sin pérdida de mensajes | RS-02, RS-10 |
| RT-02 | La plataforma debe exponer un mecanismo de ingesta (`ingestion-gateway`) que traduzca la llamada del CRM heredado (que no habla nativamente el protocolo de Pub/Sub) en una publicación autenticada al tópico correspondiente | Nuevo — deriva de RS-02 |
| RT-03 | La plataforma debe permitir al Agente de Soporte confirmar o corregir la sugerencia de clasificación desde una interfaz web con autenticación corporativa | RS-03 |

### 5.2 Requisitos no funcionales

| ID | Requisito | Origen |
|---|---|---|
| RT-04 | Todo dato en tránsito debe cifrarse con TLS 1.2 o superior; todo dato en reposo debe cifrarse con las llaves administradas por Google (o CMEK si el Oficial de Protección de Datos lo exige) | RS-05, principio de Privacidad desde el Diseño (Fase A) |
| RT-05 | Todos los servicios deben desplegarse mediante Infraestructura como Código (Terraform), sin cambios manuales directos en la consola de GCP en producción | Principio tecnológico PT-04 (sección 11) |

### 5.3 Requisitos de disponibilidad

| ID | Requisito | Métrica objetivo | Origen |
|---|---|---|---|
| RT-06 | Los servicios Cloud Run que forman la ruta crítica de clasificación deben tener una disponibilidad mensual objetivo | ≥ 99.5% (heredada del SLA de Cloud Run regional) | RS-01 |
| RT-07 | Ante la indisponibilidad de cualquier componente de clasificación (reglas, IA o validación), el caso debe degradar de forma segura a la cola manual del Repartidor, sin bloquear la recepción de nuevas PQRS | RS-10, Riesgo 3 (Fase A) |

### 5.4 Requisitos de rendimiento

| ID | Requisito | Métrica objetivo | Origen |
|---|---|---|---|
| RT-08 | El Motor de Reglas debe responder en tiempo sub-segundo | p95 ≤ 300 ms | RS-01, RS-09 |
| RT-09 | El Motor IA (llamada a Vertex AI) debe responder dentro de un tiempo razonable para un flujo asíncrono, aunque no sea literalmente de "milisegundos" como sugiere RS-01 — se ajusta la expectativa a la naturaleza de un modelo de lenguaje | p95 ≤ 3 s | Refinamiento arquitectónico de RS-01 (ver ADR-09) |
| RT-10 | El tiempo de ciclo completo de clasificación automática (ingesta → reglas/IA → validación disponible para el agente) no debe exceder | p95 ≤ 8 s | KPI de la Fase A (reducción del 90% en tiempo de triage) |

### 5.5 Requisitos de escalabilidad

| ID | Requisito | Métrica objetivo | Origen |
|---|---|---|---|
| RT-11 | La plataforma debe soportar el volumen actual (>10.000 PQRS/día) con margen de crecimiento sin cambios arquitectónicos | ≥ 3x el volumen actual (≈30.000/día) | RS-01 |
| RT-12 | El escalamiento debe ser automático (sin aprovisionamiento manual de capacidad) para todos los componentes de la ruta de clasificación | Cloud Run y Pub/Sub escalan de forma nativa; Vertex AI gestionado | Principio *cloud-first* (Fase A) |

### 5.6 Requisitos de seguridad

| ID | Requisito | Origen |
|---|---|---|
| RT-13 | El acceso de los Agentes de Soporte a la Interfaz de Validación debe controlarse por identidad (Zero Trust), no por ubicación de red | Principio tecnológico PT-02 |
| RT-14 | Ningún dato de PQRS debe salir del perímetro de seguridad definido (VPC Service Controls) | RS-05, Fase A sección 12.2 |
| RT-15 | Las credenciales de servicio (API keys, tokens) deben gestionarse en Secret Manager con rotación; se evitan llaves estáticas de larga duración donde sea técnicamente posible | Principio tecnológico PT-06 |

### 5.7 Requisitos de continuidad

| ID | Requisito | Métrica objetivo | Origen |
|---|---|---|---|
| RT-16 | Los datos en Firestore, Cloud Storage y BigQuery deben respaldarse de forma que permitan una recuperación ante desastres | RPO ≤ 24 h | Sección 7.14 |
| RT-17 | El tiempo máximo aceptable para restablecer el servicio de clasificación automática tras un desastre regional | RTO ≤ 4 h (con degradación a cola manual mientras tanto) | Sección 7.14, ADR-06 |

### 5.8 Requisitos de interoperabilidad

| ID | Requisito | Origen |
|---|---|---|
| RT-18 | Todas las interfaces internas deben exponerse como REST/JSON documentado con OpenAPI 3.0 | Estándar TE-01 (sección 10) |
| RT-19 | El Motor IA debe poder sustituirse por otro proveedor de modelos de lenguaje sin rediseñar el resto de la arquitectura (principio de independencia tecnológica, Fase A) | Principio tecnológico PT-07 |

### 5.9 Requisitos de observabilidad y operación

| ID | Requisito | Origen |
|---|---|---|
| RT-20 | Cada componente de la ruta de clasificación debe emitir métricas, logs estructurados y trazas distribuidas correlacionables por un identificador único de caso | RS-06, sección 7.11 |
| RT-21 | Deben existir alertas automáticas ante: aumento anómalo de la tasa de escalamiento a IA, agotamiento de presupuesto mensual, y profundidad creciente de la cola de mensajes fallidos (*dead-letter*) | Sección 7.11 |

## 6. Arquitectura Tecnológica Baseline

### 6.1 Vista general

La línea base es deliberadamente simple: un sistema monolítico on-premise (el CRM) rodeado de herramientas ofimáticas de escritorio, sin capa de integración, sin nube y sin automatización. No existe una arquitectura tecnológica formalmente documentada previa a este proyecto.

### 6.2 Infraestructura

`Información no disponible.` Las Fases A-C no documentan servidores, hipervisores ni centros de datos específicos del CRM heredado. Se asume (Supuesto) que corresponde a infraestructura física o virtualizada dentro de un centro de datos propio o contratado de TelcoLatam.

### 6.3 Computación

`Información no disponible` en detalle (modelos de servidor, capacidad). Funcionalmente, el CRM ejecuta los módulos de registro y asignación de PQRS descritos en la Fase C (sección 2.2).

### 6.4 Almacenamiento

`Información no disponible` en detalle técnico. Funcionalmente, el histórico de PQRS está "disperso y sin etiquetar" (Fase C, sección 2.5), sin un repositorio de datos gobernado.

### 6.5 Redes y comunicaciones

`Información no disponible.` La Fase A (sección 5) indica que el CRM "tiene APIs, pero la infraestructura es On-Premise"; no se documenta topología, segmentación ni si existe salida a internet.

### 6.6 Seguridad

`Información no disponible` en el nivel de controles técnicos (firewalls, IAM interno, cifrado). No hay evidencia de un perímetro de seguridad formal ni de gestión de secretos.

### 6.7 Plataformas

CRM heredado (plataforma no especificada por nombre en la documentación de fases anteriores) + suite ofimática de escritorio + editor de texto con certificado de firma digital PDF (Fase C, sección 2.2).

### 6.8 Middleware

No existe middleware de integración; toda transferencia de datos entre aplicaciones es manual (copiar/pegar, envío de archivos por correo interno — Fase C, sección 2.3).

### 6.9 Integración

Ninguna interfaz automatizada (Fase C, sección 2.3): 0 integraciones documentadas entre el CRM, la suite ofimática y el repositorio de plantillas.

### 6.10 Operación y monitoreo

No existe observabilidad de sistema ni monitoreo formal del proceso; el seguimiento depende enteramente de la disponibilidad del Repartidor (Fase B, sección 2.5).

## 7. Arquitectura Tecnológica Target

### 7.1 Principios de diseño

Serverless-first, seguridad por diseño (Zero Trust + privacidad desde el diseño), automatización total de la infraestructura (IaC), observabilidad por defecto, resiliencia mediante degradación segura hacia el proceso manual existente, y costo variable (pago por uso) alineado al presupuesto ya aprobado. El detalle de cada principio está en la sección 11.

### 7.2 Vista general de la arquitectura

La arquitectura target despliega los componentes ya catalogados en la Fase C sobre un único proyecto GCP de producción (más proyectos de *staging*/desarrollo, sección 7.8), en la región **southamerica-east1 (São Paulo)** como región primaria (ADR-01, sección 15), con conectividad privada hacia el CRM heredado, un perímetro VPC Service Controls que envuelve todos los servicios de datos, e IAM de mínimo privilegio con una cuenta de servicio dedicada por componente. La vista conceptual completa está en la sección 8.1.

### 7.3 Arquitectura de infraestructura

| Decisión | Valor | Justificación |
|---|---|---|
| Región primaria | `southamerica-east1` (São Paulo, Brasil) | Región GCP más cercana a Colombia con el catálogo de servicios más maduro en Sudamérica; GCP no tiene región propia en Colombia (ver ADR-01) |
| Región secundaria (DR) | `southamerica-west1` (Santiago, Chile) — a confirmar | Alternativa sudamericana para redundancia de datos; no se activa cómputo aquí en el MVP (ver sección 7.14) |
| Disponibilidad de Vertex AI (Gemini) en la región primaria | **A validar (TBD)** | Los modelos generativos de Vertex AI no siempre están disponibles en todas las regiones; si el modelo no está disponible en `southamerica-east1`, el Motor IA deberá invocar el endpoint global/`us-central1` de Vertex AI, añadiendo latencia cross-region (ver Riesgo TR-03, sección 16) |
| Organización de proyectos GCP | `pqrs-prod`, `pqrs-staging`, `pqrs-shared-services` (CI/CD, logging centralizado) | Aísla producción de pruebas; sigue las prácticas recomendadas de Google Cloud (jerarquía de recursos) |

### 7.4 Arquitectura de computación

| Servicio Cloud Run | Función | Concurrencia | CPU siempre asignada | Instancias mín./máx. |
|---|---|---|---|---|
| `ingestion-gateway` | Recibe la llamada del CRM heredado y publica en Pub/Sub | 80 | Sí (reduce *cold start* para el CRM) | 1 / 10 |
| `motor-reglas` | Clasificación por diccionario de palabras clave (Fase C, sección 3.1) | 80 | No (escala a cero) | 0 / 10 |
| `motor-ia` | Orquesta la llamada a Vertex AI (Gemini 2.5 Flash-Lite) | 40 | No | 0 / 10 |
| `interfaz-validacion` | Aplicación web para el Agente de Soporte, respaldada por Firestore | 80 | No | 0 / 5 |

No se utiliza Google Kubernetes Engine (GKE): dado el volumen (>10.000 PQRS/día, tráfico bajo y no sostenido) y la ausencia de necesidades de orquestación multi-contenedor compleja, Cloud Run cubre el requisito de escalabilidad (RT-11, RT-12) con menor sobrecarga operativa y menor costo — decisión coherente con el principio *cloud-first* de la Fase A y con el presupuesto (ver ADR-04, sección 15).

### 7.5 Arquitectura de almacenamiento

| Componente | Servicio | Configuración objetivo |
|---|---|---|
| Estado de casos en validación | Firestore (modo Nativo, regional en `southamerica-east1`) | Recuperación a un punto en el tiempo (PITR) habilitada; reglas de seguridad restringidas a las cuentas de servicio de los Cloud Run autorizados |
| Texto crudo de la PQRS (pre-anonimización) | Cloud Storage — bucket `pqrs-raw` | Retención corta (ver política de retención, sección 7.7); cifrado con llaves administradas por Google (CMEK opcional, ADR-09) |
| Dataset anonimizado de entrenamiento | Cloud Storage — bucket `pqrs-training-anonymized` (poblado vía Cloud DLP) | Versionado habilitado; ciclo de vida: transición a *Coldline* tras 90 días |
| Analítica y KPI | BigQuery — dataset `pqrs_analytics` | Particionado por fecha de ingesta; *time travel* de 7 días + exportaciones periódicas a Cloud Storage para retención extendida |
| Auditoría | BigQuery — dataset `pqrs_audit` | Acceso restringido a nivel de columna para campos que pudieran contener texto no anonimizado; retención alineada a los requerimientos de trazabilidad de la Superintendencia de Industria y Comercio (Fase C, sección 4.3) |

### 7.6 Arquitectura de red y comunicaciones

* **VPC** dedicada por proyecto (`vpc-pqrs-prod`), con un conector de Acceso a VPC sin servidores (Serverless VPC Access) para que los servicios Cloud Run alcancen recursos privados.
* **Conectividad con el CRM heredado:** dos alternativas evaluadas (ADR-02, sección 15):
  1. **Cloud VPN de alta disponibilidad (HA VPN)** entre la red on-premise de TelcoLatam y la VPC de GCP — recomendado si el CRM no tiene salida a internet o si la política de seguridad de TelcoLatam exige conectividad privada.
  2. **Llamada saliente HTTPS autenticada con Workload Identity Federation** desde el CRM hacia el `ingestion-gateway` (expuesto con ingreso restringido) — recomendado si el CRM sí puede salir a internet, por ser más simple y económico.
  * **Decisión:** se documenta como condicional (TBD) hasta confirmar con el Gerente de TI la postura de red real del CRM (sección 21).
* **Cloud NAT** para that any Cloud Run/recurso privado que requiera salida a internet (por ejemplo, para acceder a Vertex AI si su endpoint no es accesible vía Private Service Connect en la región elegida).
* **Cloud Armor** (WAF) solo si `ingestion-gateway` requiere ingreso público (alternativa 2 anterior); no aplica si se opta por Cloud VPN puro.
* **API Gateway** como capa única de gestión de las APIs REST internas (autenticación por API key/OIDC, cuotas, versión `v1`), evitando el costo de una plataforma de gestión de APIs de nivel empresarial (Apigee) no justificada por el volumen ni el presupuesto del proyecto.

### 7.7 Arquitectura de seguridad

* **Perímetro VPC Service Controls** (ya definido en la Fase A/C) que envuelve Pub/Sub, Cloud Storage, BigQuery, Firestore, Cloud DLP y Vertex AI: ningún dato de PQRS puede exfiltrarse fuera del perímetro, incluso con credenciales válidas pero mal configuradas.
* **IAM de mínimo privilegio:** una cuenta de servicio por componente (`sa-ingestion-gateway`, `sa-motor-reglas`, `sa-motor-ia`, `sa-interfaz-validacion`, `sa-etl-dlp`), cada una con únicamente los roles necesarios (p. ej. `sa-motor-ia` solo tiene `roles/aiplatform.user` y el permiso de publicar en su tópico de salida, no acceso a BigQuery ni a Cloud Storage).
* **Zero Trust para agentes humanos:** la Interfaz de Validación se expone mediante **Identity-Aware Proxy (IAP)**, controlando el acceso por identidad corporativa (Google Workspace/Cloud Identity) y contexto (dispositivo, ubicación), en lugar de depender de que el agente esté conectado a la red corporativa o VPN — cumple RT-13.
* **Gestión de secretos:** todas las credenciales (API keys de contingencia, tokens) se almacenan en **Secret Manager**, con rotación programada; se prioriza **Workload Identity Federation** para evitar llaves de cuenta de servicio estáticas donde el CRM lo permita (ADR-07).
* **Cifrado:** TLS 1.2+ en tránsito (impuesto por defecto en los servicios GCP usados); en reposo, llaves administradas por Google por defecto, con **Cloud KMS (CMEK)** como mejora opcional para el bucket de datos anonimizados si el Oficial de Protección de Datos lo exige (ADR-09, no incluido en el presupuesto del MVP).
* **Gestión de certificados:** certificados TLS administrados automáticamente por Google (Certificate Manager) para cualquier dominio personalizado expuesto a través de API Gateway o un Load Balancer.
* **Gestión de vulnerabilidades:** análisis de imágenes de contenedor habilitado en Artifact Registry (Container Analysis) antes de cada despliegue; **Security Command Center** (nivel estándar/gratuito) habilitado para postura de seguridad continua sobre el perímetro.

### 7.8 Arquitectura de plataformas

Separación por proyecto GCP: `pqrs-dev` (desarrollo), `pqrs-staging` (pruebas de integración y de carga), `pqrs-prod` (producción), y `pqrs-shared-services` (Artifact Registry, Cloud Build, sumideros de logging centralizados). Esta separación es un requisito de gobierno de configuración (sección 12) que la línea base no tenía en absoluto.

### 7.9 Middleware

Pub/Sub actúa como el middleware de mensajería asíncrona entre el CRM heredado y los servicios de clasificación (ya definido en la Fase C); no se introduce un bus de mensajería adicional (se descartaron explícitamente RabbitMQ y Kafka en la Fase A por sobredimensionamiento frente al volumen del proyecto).

### 7.10 Integración tecnológica

Todas las integraciones internas son REST/JSON sobre HTTPS, documentadas con OpenAPI 3.0 (TE-01) y expuestas a través de API Gateway. La integración con Vertex AI usa el SDK/API REST nativo de Google. El detalle punto a punto ya está en el catálogo de interfaces de la Fase C (sección 3.5); esta fase añade el mecanismo de transporte (Pub/Sub, Cloud VPN/WIF) y los controles de autenticación.

### 7.11 Observabilidad

* **Métricas y paneles:** Cloud Monitoring, con paneles para tasa de escalamiento reglas→IA, tasa de escalamiento IA→manual, latencia p95 por componente (RT-08/RT-09/RT-10), tasa de error, y consumo de presupuesto de Vertex AI.
* **Logs:** Cloud Logging con logs estructurados (JSON) correlacionados por un identificador único de caso (*trace ID*) presente en todos los componentes (RT-20); sumidero (*sink*) hacia BigQuery (`pqrs_audit`) para retención extendida.
* **Trazas distribuidas:** Cloud Trace habilitado en la cadena `ingestion-gateway → motor-reglas → motor-ia → interfaz-validacion`, permitiendo diagnosticar en qué salto se concentra la latencia.
* **Reporte de errores:** Error Reporting integrado nativamente con Cloud Run.
* **Alertas (RT-21):** notificaciones (correo, y opcionalmente Slack/PagerDuty si se contratan) ante: aumento anómalo de la tasa de escalamiento a IA, profundidad creciente de la cola de mensajes fallidos (*dead-letter topic* de Pub/Sub), y alertas de presupuesto de facturación (Cloud Billing Budgets) dado el historial de falta de control presupuestal señalado en la Fase A original.

### 7.12 DevSecOps / CI-CD

* **Control de versiones:** el repositorio del proyecto ya existente en GitHub (`Modelo-de-IA-para-Enrutamiento-Inteligente-de-PQRS`), con una carpeta `infra/` para el código Terraform.
* **Integración continua:** Cloud Build (o GitHub Actions equivalente) ejecuta pruebas unitarias, análisis estático de código, escaneo de secretos (p. ej. *trufflehog*/*git-secrets*) y construcción de la imagen de contenedor en cada *pull request*.
* **Despliegue continuo:** Cloud Build publica la imagen en Artifact Registry (con escaneo de vulnerabilidades habilitado) y despliega a Cloud Run con **despliegue gradual (traffic splitting)**: 10% → 50% → 100% del tráfico a la nueva revisión, con reversión automática si las métricas de error superan un umbral.
* **Infraestructura como Código:** Terraform gestiona Pub/Sub, Cloud Run, IAM, VPC, Firestore, BigQuery y VPC Service Controls; ningún cambio manual en la consola de GCP en `pqrs-prod` (RT-05).
* **DevSecOps:** las puertas de calidad (*quality gates*) del pipeline incluyen el análisis estático, el escaneo de secretos y el escaneo de vulnerabilidades de contenedor como bloqueantes, no opcionales.

### 7.13 Resiliencia y alta disponibilidad

Cloud Run es multi-zona dentro de la región de forma nativa (sin configuración adicional); Pub/Sub replica los mensajes dentro de la región del tópico; Firestore y BigQuery ofrecen alta disponibilidad regional gestionada por Google. No se requiere diseño adicional de HA a nivel de infraestructura porque los servicios son completamente administrados — el foco de esta arquitectura está en la **resiliencia del proceso de negocio** (sección 7.14), no en la redundancia de servidores.

### 7.14 Continuidad y recuperación ante desastres

* **Principio de degradación segura (ADR-06):** si cualquier componente de clasificación automática (reglas o IA) falla o se degrada, el diseño ya contempla (RS-10, RT-07) el enrutamiento automático a la cola manual del Repartidor. Esto significa que, ante una falla total de la plataforma en GCP, TelcoLatam **no pierde la capacidad de atender PQRS**: simplemente vuelve al proceso 100% manual que ya opera hoy como línea base. Esta es la estrategia de continuidad de negocio principal del proyecto, y reduce sustancialmente la necesidad de una arquitectura de DR costosa.
* **Backup de datos:**
  * Firestore: exportaciones administradas diarias a Cloud Storage (además del PITR nativo).
  * BigQuery: *time travel* de 7 días + exportación mensual a Cloud Storage (*Coldline*) para retención de auditoría de largo plazo.
  * Cloud Storage: versionado de objetos habilitado en los buckets de datos anonimizados.
* **Objetivos de recuperación:** RPO ≤ 24 h (RT-16) y RTO ≤ 4 h (RT-17) para el restablecimiento de la clasificación automática; durante ese lapso, el sistema opera en modo degradado (100% cola manual) sin interrupción del servicio al cliente.
* **Región secundaria:** `southamerica-west1` queda documentada como región de redundancia de datos (replicación de backups), sin cómputo activo en el MVP — activarla implicaría una inversión adicional no incluida en el presupuesto actual (Anexo A, Fase C).

### 7.15 Estimación de costos, ubicación de los datos y especificación detallada por componente

> **Nota de fuentes y alcance.** El presupuesto operativo agregado (≈ US$550-570/mes) ya fue aprobado en la Fase A (sección 12.3) y desglosado por bloques en la *Propuesta de Arquitectura GCP y Presupuesto* (sección 7). Esta tabla lo desagrega componente por componente, incorporando los elementos de infraestructura, seguridad y CI/CD que esta Fase D añade y que aquellos documentos no costeaban individualmente (Firestore, Secret Manager, API Gateway, Artifact Registry, Cloud Build, Cloud VPN condicional). Los precios unitarios se tomaron de las páginas oficiales de precios de Google Cloud consultadas en septiembre de 2026 y deben reverificarse en la calculadora oficial de Google Cloud antes de la aprobación final; los rangos reflejan el volumen de referencia de la Fase A (**(supuesto)** >10.000 PQRS/día ≈ 300.000/mes) y no un perfil de tráfico horario confirmado (sección 4.6).

| ID | Componente | Servicio / modo específico de GCP usado | Ubicación (región y centro de datos) | Costo estimado (mensual) | Notas |
|---|---|---|---|---|---|
| COMP-01 | CRM heredado | N/A (on-premise, sin cambios funcionales) | Centro de datos de TelcoLatam en Colombia (dirección/proveedor exactos: `Información no disponible`) | US$0 (no es gasto de GCP) | Sección 21, punto 3 |
| COMP-02 | Ingestion Gateway | Cloud Run, 2ª generación, 1 vCPU / 512 MiB, CPU siempre asignada, mín. 1 / máx. 10 instancias | `southamerica-east1` (São Paulo, Brasil) | ≈ US$0-10 | Instancia mínima=1 sale del nivel gratuito de "escala a cero"; impacto marginal a este volumen |
| COMP-03 | Pub/Sub | 5 tópicos (`pqrs-recibido`, `pqrs-requiere-ia`, `pqrs-clasificado`, `pqrs-cola-manual`, `pqrs-enrutado`) + suscripciones push | `southamerica-east1` (política de almacenamiento de mensajes fijada a la región) | ≈ US$0-5 | Primeros 10 GiB de throughput/mes gratis; a 300.000 mensajes/mes de texto corto, prácticamente sin costo |
| COMP-04 | Motor de Reglas | Cloud Run, 2ª generación, 1 vCPU / 512 MiB, escala a 0, máx. 10 instancias | `southamerica-east1` | ≈ US$0-5 | Dentro del nivel gratuito (2M solicitudes + 360.000 GiB-s/mes) a este volumen |
| COMP-05 | Motor IA (orquestador) | Cloud Run, 2ª generación, 1 vCPU / 1 GiB (memoria adicional para el cliente SDK de Vertex AI), escala a 0 | `southamerica-east1` | ≈ US$0-10 | Cómputo de orquestación únicamente; el costo del modelo se factura aparte (fila COMP-06) |
| COMP-06 | Vertex AI — motor de inferencia | Vertex AI Generative AI API, modelo **Gemini 2.5 Flash-Lite** (modelo base de Google, sin *fine-tuning*), invocado por `generateContent` | `southamerica-east1` **si el modelo está disponible ahí (TBD, GAP-10)**; en caso contrario, endpoint de `us-central1` (EE. UU.) | ≈ US$15-20 (peor caso: 100% de los casos escalados a IA) | Cifra heredada de la *Propuesta de Arquitectura GCP*, sección 7; si aplica el endpoint de EE. UU., los datos de PQRS cruzan de región durante la inferencia — implicación de residencia de datos a validar (sección 21, punto 6) |
| COMP-07 | Interfaz de Validación | Cloud Run, 2ª generación, 1 vCPU / 512 MiB, escala a 0, máx. 5 instancias | `southamerica-east1` | ≈ US$0-5 | Tráfico bajo (solo Agentes de Soporte) |
| COMP-08 | Firestore | Modo Nativo, base de datos **regional** (single-region), con recuperación a un punto en el tiempo (PITR) | `southamerica-east1` | ≈ US$0-10 | Nivel gratuito diario: 1 GiB almacenado + 50.000 lecturas + 20.000 escrituras + 20.000 eliminaciones; a >10.000 casos/día con ~2 lecturas y ~2 escrituras por caso, el uso ronda el límite gratuito de escrituras — posible pequeño excedente |
| COMP-09 | Cloud Storage | 2 buckets Standard, regionales: `pqrs-raw` (retención corta) y `pqrs-training-anonymized` (versionado, ciclo de vida a *Coldline* a los 90 días) | `southamerica-east1` | ≈ US$5-10 | El nivel "Always Free" de 5 GiB de Cloud Storage solo aplica a buckets en `us-west1`/`us-central1`/`us-east1`; al ser un bucket regional en Sudamérica, no aplica ese nivel gratuito |
| COMP-10 | Cloud DLP | Trabajos de anonimización por lotes (no continuos) sobre el texto crudo antes de poblar el bucket de entrenamiento | `southamerica-east1` (o API global de DLP) | ≈ US$5-10 | Depende del volumen real de texto a inspeccionar/mes; estimación conservadora, validar con un piloto |
| COMP-11 | BigQuery | 2 *datasets*: `pqrs_analytics` (particionado por fecha) y `pqrs_audit` (acceso restringido por columna) | `southamerica-east1` (almacenamiento regional) | ≈ US$5-10 | Primer 1 TiB de consultas/mes gratis; a este volumen de datos, difícilmente se excede |
| COMP-12 | Cloud Monitoring / Logging / Trace | Métricas, logs estructurados y trazas de los 4 servicios Cloud Run | Global (con datos de origen procesados desde `southamerica-east1`) | ≈ US$0-5 | Niveles gratuitos mensuales: ~150 MiB de métricas, 50 GiB de logs, 2,5M *spans* de traza — suficiente a este volumen |
| COMP-13 | IAM | Cuentas de servicio de mínimo privilegio (una por componente) | Global (sin datos de PQRS) | US$0 | Sin costo directo |
| COMP-14 | VPC Service Controls | Perímetro de seguridad sobre Pub/Sub, Firestore, Cloud Storage, BigQuery, Cloud DLP y Vertex AI | `southamerica-east1` (recursos protegidos) | US$0 | Sin costo directo; es un control de gobierno, no un servicio facturado por uso |
| COMP-15 | Secret Manager | ≈5-8 secretos activos (uno por cuenta de servicio/credencial) | Global, con política de réplica fijada a `southamerica-east1` | ≈ US$0-2 | Nivel gratuito: 6 versiones activas + 10.000 operaciones de acceso/mes; a este número de secretos y de invocaciones, se mantiene cerca del nivel gratuito |
| COMP-16 | Cloud VPN (HA VPN) | 2 túneles (mínimo para 99,99% de disponibilidad) | `southamerica-east1` (extremo GCP); extremo on-premise en el datacenter de TelcoLatam | **Condicional (ADR-02):** ≈ US$72-90/mes si se adopta esta alternativa; **US$0 adicionales** si se adopta la alternativa de HTTPS + Workload Identity Federation (fila COMP-24) | US$0,05/túnel/hora × 2 túneles × 720 h ≈ US$72/mes, más tráfico IPsec — impacto de presupuesto real que depende de GAP-06 |
| COMP-17 | Serverless VPC Access | Conector para que Cloud Run alcance recursos privados (solo necesario si se adopta Cloud VPN) | `southamerica-east1` | ≈ US$5-15 (condicional, solo si aplica COMP-16) | Estimación conservadora por instancias mínimas del conector; validar en la calculadora oficial |
| COMP-18 | API Gateway | Gestión de las APIs REST internas (autenticación, cuotas, versión `v1`) | Global (con *backends* en `southamerica-east1`) | US$0 | Primeras 2.000.000 de llamadas/mes gratis; a ≈300.000 PQRS/mes con 3-4 llamadas internas cada una (≈900.000-1.200.000 llamadas/mes), se mantiene dentro del nivel gratuito |
| COMP-19 | Identity-Aware Proxy (IAP) | Control de acceso Zero Trust a la Interfaz de Validación | `southamerica-east1` (recurso protegido) | US$0 de GCP | El costo real no es de GCP sino de la licencia de Google Workspace/Cloud Identity por agente (fila COMP-23) — **pendiente, TBD sección 21 punto 4** |
| COMP-20 | Artifact Registry | Repositorio de imágenes de contenedor de los 4 servicios Cloud Run (con escaneo de vulnerabilidades) | `southamerica-east1` (o multi-región, a definir en Terraform) | ≈ US$1-5 | Nivel gratuito: 0,5 GiB/mes; con 4 servicios y varias versiones retenidas, el excedente es pequeño (pocos GiB) |
| COMP-21 | Cloud Build | Pipeline de CI/CD (lint, pruebas, escaneo de secretos, build de imagen) | Región del *pool* de ejecución (a definir, recomendado `southamerica-east1` o el más cercano disponible) | ≈ US$0-10 | Nivel gratuito: 2.500 minutos de build/mes; con el volumen de despliegues esperado en un equipo pequeño, se mantiene mayormente dentro del nivel gratuito |
| COMP-22 | Terraform (IaC) | Código de infraestructura, sin servicio alojado por GCP | N/A (repositorio Git) | US$0 de GCP | Costo ya cubierto como parte del tiempo del Ingeniero Cloud/Backend (Fase A, sección 12.3) |
| COMP-23 | Google Cloud Identity / Workspace | Directorio corporativo para habilitar IAP | Google gestiona la ubicación; TelcoLatam administra el dominio | **Pendiente (TBD):** rango de mercado orientativo ≈ US$7-12/usuario/mes si se adquiere, según el plan — no es una cifra confirmada | Depende del número de Agentes de Soporte con acceso y de si TelcoLatam ya tiene una suscripción activa (sección 21, punto 4) |
| COMP-24 | Workload Identity Federation | Autenticación del CRM sin llaves estáticas (alternativa a COMP-16) | Global | US$0 | Sin costo directo; es la alternativa económica en el ADR-02 |
| COMP-25 | Cloud KMS | 1-2 llaves de cifrado (CMEK opcional) | `southamerica-east1` | **Opcional, no incluido en el MVP:** ≈ US$1-5/mes si se activa | Solo se activa si el Oficial de Protección de Datos lo exige (ADR-09); la Ley 1581 no lo exige por sí sola |
| COMP-26 | Security Command Center | Postura de seguridad continua, nivel estándar | Global | US$0 | El nivel estándar es gratuito; el nivel Premium (no contemplado en este MVP) tiene costo adicional |
| COMP-27 | Cloud Billing Budgets & Alerts | Alertas de consumo frente al presupuesto aprobado | Global | US$0 | Sin costo directo |

**Resumen de costo operativo mensual refinado**

| Escenario | Costo estimado/mes |
|---|---|
| Infraestructura GCP — suma de las filas anteriores, escenario base (conectividad CRM vía HTTPS + Workload Identity Federation, sin Cloud VPN, sin CMEK) | ≈ US$40-100 |
| + Licencia de Google Workspace/Cloud Identity (IAP) | Pendiente — depende del número de agentes y de si TelcoLatam ya tiene la suscripción (TBD, sección 21, punto 4); no se suma al total hasta confirmarse |
| + Científico de Datos/ML Engineer (monitoreo y reentrenamiento, 20% dedicación — ya presupuestado en la *Propuesta de Arquitectura GCP*, sección 7) | ≈ US$490 |
| **Total operativo refinado — escenario base** | **≈ US$530-590/mes** — consistente con el rango ya aprobado en la Fase A, sección 12.3 (≈ US$550-570/mes) |
| **Escenario alterno:** si el ADR-02 se resuelve hacia Cloud VPN (HA VPN) en lugar de HTTPS + WIF (filas COMP-16 y COMP-17) | + ≈ US$77-105/mes adicionales → **≈ US$610-695/mes** |
| **Adición opcional:** Cloud KMS/CMEK si el Oficial de Protección de Datos lo exige (COMP-25) | + ≈ US$1-5/mes |

**Implicación para la toma de decisiones:** la resolución del ADR-02 (conectividad con el CRM heredado, GAP-06) no es solo una decisión de red — tiene un impacto directo de hasta ≈ US$80-105/mes sobre el presupuesto operativo (aprox. 15% del total). Se recomienda que esta confirmación del Gerente de TI ocurra antes de cerrar el presupuesto operativo definitivo, no después.

## 8. Vistas y diagramas arquitectónicos

### 8.1 Vista conceptual

```mermaid
flowchart TB
    subgraph L1["Usuarios"]
        A1["Cliente / Ciudadano"]
        A2["Agente de Soporte"]
        A3["Científico / Ingeniero de Datos"]
    end
    subgraph L2["Canales"]
        B1["Canal de entrada de PQRS"]
        B2["Interfaz de Validación (web, vía IAP)"]
    end
    subgraph L3["Aplicaciones"]
        C1["Motor de Reglas"]
        C2["Motor IA"]
        C3["Interfaz de Validación"]
    end
    subgraph L4["Capa de Integración"]
        D1["Ingestion Gateway"]
        D2["Pub/Sub"]
        D3["API Gateway"]
    end
    subgraph L5["Plataforma Tecnológica"]
        E1["Cloud Run"]
        E2["Vertex AI (Gemini 2.5 Flash-Lite)"]
        E3["IAM + VPC Service Controls"]
    end
    subgraph L6["Plataforma de Datos"]
        F1["Firestore"]
        F2["Cloud Storage + Cloud DLP"]
        F3["BigQuery"]
    end
    subgraph L7["Infraestructura"]
        G1["VPC — southamerica-east1"]
        G2["Cloud VPN / conectividad con CRM heredado"]
    end

    A1 --> B1 --> D1 --> D2 --> C1
    C1 -->|"sin certeza"| C2
    C1 -->|"certeza alta"| C3
    C2 --> C3
    C3 --> B2 --> A2
    C1 -.-> E1
    C2 -.-> E1
    C2 -.-> E2
    C3 -.-> F1
    F2 -.->|"dataset anonimizado"| C2
    C3 -.-> F3
    A3 -.-> F2
    D3 -.-> C1
    D3 -.-> C2
    D3 -.-> C3
    E1 --> G1
    E3 -.-> G1
    G1 --- G2
```

### 8.2 Vista lógica

```mermaid
flowchart LR
    CRM["CRM heredado\n(Canal de entrada)"] --> IG["Ingestion Gateway\n(Cloud Run)"]
    IG --> PS1[["Pub/Sub: pqrs-recibido"]]
    PS1 --> MR["Motor de Reglas\n(Cloud Run)"]
    MR -->|"certeza alta"| PS3[["Pub/Sub: pqrs-clasificado"]]
    MR -->|"sin certeza"| PS2[["Pub/Sub: pqrs-requiere-ia"]]
    PS2 --> MI["Motor IA\n(Cloud Run + Vertex AI)"]
    MI -->|"confianza ≥ 75%"| PS3
    MI -->|"confianza < 75%"| PS4[["Pub/Sub: pqrs-cola-manual"]]
    PS4 --> COLA["Cola manual\n(Repartidor)"]
    PS3 --> IV["Interfaz de Validación\n(Cloud Run + Firestore)"]
    IV --> PS5[["Pub/Sub: pqrs-enrutado"]]
    COLA --> PS5
    PS5 --> CRM2["CRM heredado\n(actualiza vía API)"]
    IG -.-> GCS["Cloud Storage"]
    GCS --> DLP["Cloud DLP"]
    DLP --> BQ["BigQuery"]
    MR -.-> MON["Cloud Monitoring / Logging / Trace"]
    MI -.-> MON
    IV -.-> MON
```

### 8.3 Vista física

```mermaid
flowchart TB
    subgraph OnPrem["Datacenter TelcoLatam (on-premise, Colombia)"]
        CRM["Servidor(es) CRM heredado"]
    end

    subgraph GCPProdProject["Proyecto GCP: pqrs-prod (southamerica-east1)"]
        subgraph VPCProd["VPC: vpc-pqrs-prod"]
            SVA["Conector Serverless VPC Access"]
            CR1["Cloud Run: ingestion-gateway"]
            CR2["Cloud Run: motor-reglas"]
            CR3["Cloud Run: motor-ia"]
            CR4["Cloud Run: interfaz-validacion"]
        end
        PS["Pub/Sub (tópicos y suscripciones)"]
        FS["Firestore (regional)"]
        GCS["Cloud Storage (raw + anonymized)"]
        BQ["BigQuery"]
        VAI["Vertex AI (Gemini 2.5 Flash-Lite)"]
        VPCSC["Perímetro VPC Service Controls"]
    end

    subgraph GCPDRProject["Región secundaria: southamerica-west1 (solo backups)"]
        BKP["Backups de Firestore / BigQuery / Storage"]
    end

    subgraph SharedSvc["Proyecto GCP: pqrs-shared-services"]
        AR["Artifact Registry"]
        CB["Cloud Build (CI/CD)"]
        LOGSINK["Sumidero centralizado de Logging"]
    end

    CRM ---|"Cloud VPN (HA) o HTTPS + WIF — TBD"| SVA
    SVA --> CR1 --> PS
    PS --> CR2 --> PS
    PS --> CR3 --> VAI
    CR3 --> PS
    PS --> CR4 --> FS
    CR1 -.-> GCS
    GCS -.-> BQ
    VPCSC -.-> PS
    VPCSC -.-> FS
    VPCSC -.-> GCS
    VPCSC -.-> BQ
    VPCSC -.-> VAI
    GCPProdProject -.->|"backups periódicos"| BKP
    CB -.->|"despliega revisiones"| CR1
    CB -.->|"despliega revisiones"| CR2
    CB -.->|"despliega revisiones"| CR3
    CB -.->|"despliega revisiones"| CR4
    LOGSINK -.-> BQ
```

### 8.4 Vista de despliegue

```mermaid
flowchart LR
    DEV["Desarrollador"] -->|"push / PR"| REPO["Repositorio GitHub"]
    REPO --> CB1["Cloud Build:\nlint + pruebas unitarias\n+ escaneo de secretos"]
    CB1 --> CB2["Build de imagen\nde contenedor"]
    CB2 --> AR["Artifact Registry\n(escaneo de vulnerabilidades)"]
    AR --> DEPLOY_DEV["Despliegue automático\na pqrs-dev"]
    DEPLOY_DEV --> TEST["Pruebas de integración\ny de carga en pqrs-staging"]
    TEST -->|"aprobación manual"| DEPLOY_PROD["Despliegue gradual a pqrs-prod\n(10% → 50% → 100%)"]
    DEPLOY_PROD --> MONITOR["Cloud Monitoring:\n¿tasa de error normal?"]
    MONITOR -->|"sí"| DONE["100% del tráfico\nen la nueva revisión"]
    MONITOR -->|"no"| ROLLBACK["Reversión automática\na la revisión anterior"]
```

### 8.5 Vista de integración

```mermaid
sequenceDiagram
    participant CRM as CRM heredado
    participant IG as Ingestion Gateway
    participant PS as Pub/Sub
    participant MR as Motor de Reglas
    participant MI as Motor IA (Vertex AI)
    participant IV as Interfaz de Validación
    participant AG as Agente de Soporte

    CRM->>IG: Nueva PQRS (HTTPS autenticado)
    IG->>PS: Publica evento pqrs-recibido
    PS->>MR: Entrega evento
    alt Certeza alta por reglas
        MR->>PS: Publica pqrs-clasificado
    else Sin certeza
        MR->>MI: Escala el caso
        MI->>MI: Clasificación NLP (Gemini 2.5 Flash-Lite)
        alt Confianza >= 75%
            MI->>PS: Publica pqrs-clasificado
        else Confianza < 75%
            MI->>PS: Publica pqrs-cola-manual
        end
    end
    PS->>IV: Entrega caso clasificado
    IV->>AG: Muestra sugerencia + confianza
    AG->>IV: Confirma o corrige
    IV->>PS: Publica pqrs-enrutado
    PS->>CRM: Actualiza el caso vía API
```

### 8.6 Vista de seguridad

```mermaid
flowchart TB
    subgraph Perimeter["Perímetro VPC Service Controls"]
        PS["Pub/Sub"]
        FS["Firestore"]
        GCS["Cloud Storage"]
        BQ["BigQuery"]
        DLP["Cloud DLP"]
        VAI["Vertex AI"]
    end

    subgraph Compute["Cloud Run (cuentas de servicio de mínimo privilegio)"]
        SA1["sa-ingestion-gateway"]
        SA2["sa-motor-reglas"]
        SA3["sa-motor-ia"]
        SA4["sa-interfaz-validacion"]
    end

    SM["Secret Manager"]
    KMS["Cloud KMS (CMEK opcional)"]
    IAP["Identity-Aware Proxy"]
    IDP["Google Cloud Identity / Workspace"]
    SCC["Security Command Center"]
    WIF["Workload Identity Federation"]

    CRM["CRM heredado (on-premise)"] -->|"autenticación federada, sin llaves estáticas"| WIF
    WIF --> SA1
    AGENTE["Agente de Soporte"] -->|"identidad corporativa + contexto"| IAP
    IDP --> IAP
    IAP --> SA4

    SA1 --> Perimeter
    SA2 --> Perimeter
    SA3 --> Perimeter
    SA4 --> Perimeter

    SA1 -.->|"credenciales"| SM
    SA2 -.->|"credenciales"| SM
    SA3 -.->|"credenciales"| SM
    SA4 -.->|"credenciales"| SM

    GCS -.-> KMS
    BQ -.-> KMS

    SCC -.->|"postura de seguridad"| Perimeter
    SCC -.->|"postura de seguridad"| Compute
```

## 9. Catálogo de componentes tecnológicos

| ID | Componente | Categoría | Baseline | Target | Propósito | Dependencias |
|---|---|---|---|---|---|---|
| COMP-01 | CRM heredado | Aplicación / plataforma | Sí (sin cambios funcionales) | Sí (extendido con conectividad) | Registro y distribución del caso | COMP-16 o COMP-24 |
| COMP-02 | Ingestion Gateway | Cómputo (Cloud Run) | No | Sí | Traduce la llamada del CRM a un evento Pub/Sub autenticado | COMP-03, COMP-16/24 |
| COMP-03 | Pub/Sub | Integración / mensajería | No | Sí | Desacopla el CRM de los servicios de clasificación, con reintentos | COMP-02, COMP-04, COMP-05, COMP-06 |
| COMP-04 | Motor de Reglas | Cómputo (Cloud Run) | No | Sí | Clasificación determinística de primer nivel | COMP-03 |
| COMP-05 | Motor IA | Cómputo (Cloud Run) + IA | No | Sí | Clasificación NLP de segundo nivel | COMP-03, COMP-06 |
| COMP-06 | Vertex AI (Gemini 2.5 Flash-Lite) | Plataforma de IA | No | Sí | Modelo de lenguaje para clasificación | COMP-05 |
| COMP-07 | Interfaz de Validación | Cómputo (Cloud Run) | No | Sí | Confirmación/corrección humana de la sugerencia | COMP-03, COMP-08, COMP-19 |
| COMP-08 | Firestore | Almacenamiento (datos operacionales) | No | Sí | Estado de los casos en validación | COMP-07 |
| COMP-09 | Cloud Storage | Almacenamiento (objetos) | No | Sí | Archivo del texto crudo y del dataset anonimizado | COMP-10 |
| COMP-10 | Cloud DLP | Datos / cumplimiento | No | Sí | Anonimización antes de reentrenar el modelo | COMP-09 |
| COMP-11 | BigQuery | Almacenamiento analítico | No | Sí | KPI, analítica y auditoría | COMP-09 |
| COMP-12 | Cloud Monitoring / Logging / Trace | Observabilidad | No | Sí | Métricas, logs y trazas de toda la ruta | Todos los COMP-0x de cómputo |
| COMP-13 | IAM | Seguridad / identidad | No | Sí | Cuentas de servicio de mínimo privilegio | Todos |
| COMP-14 | VPC Service Controls | Seguridad / perímetro | No | Sí | Evita exfiltración de datos de PQRS | COMP-03, 06, 08, 09, 10, 11 |
| COMP-15 | Secret Manager | Seguridad / secretos | No | Sí | Gestión y rotación de credenciales | Todos los COMP-0x de cómputo |
| COMP-16 | Cloud VPN (HA VPN) | Red | No | Condicional (ADR-02) | Conectividad privada con el CRM heredado | COMP-01, COMP-02 |
| COMP-17 | Serverless VPC Access | Red | No | Sí | Permite a Cloud Run alcanzar recursos privados | COMP-16 |
| COMP-18 | API Gateway | Integración / API management | No | Sí | Punto único de gestión de las APIs REST | COMP-02, 04, 05, 07 |
| COMP-19 | Identity-Aware Proxy (IAP) | Seguridad / Zero Trust | No | Sí | Acceso de agentes por identidad, no por red | COMP-07, COMP-23 |
| COMP-20 | Artifact Registry | CI/CD | No | Sí | Almacena y escanea imágenes de contenedor | COMP-21 |
| COMP-21 | Cloud Build | CI/CD | No | Sí | Integración y despliegue continuo | COMP-20 |
| COMP-22 | Terraform (IaC) | Gestión de configuración | No | Sí | Despliegue reproducible de toda la infraestructura | — |
| COMP-23 | Google Cloud Identity / Workspace | Identidad | No | Sí (supuesto, sección 4.6) | Directorio corporativo para IAP | COMP-19 |
| COMP-24 | Workload Identity Federation | Seguridad / identidad | No | Condicional (ADR-02) | Autenticación del CRM sin llaves estáticas | COMP-02 |
| COMP-25 | Cloud KMS | Seguridad / cifrado | No | Opcional (ADR-09) | CMEK para datos anonimizados | COMP-09, COMP-11 |
| COMP-26 | Security Command Center | Seguridad / postura | No | Sí (nivel estándar) | Gestión continua de vulnerabilidades y postura | COMP-14 |
| COMP-27 | Cloud Billing Budgets & Alerts | Gobernanza / FinOps | No | Sí | Alertas de consumo frente al presupuesto aprobado | — |

## 10. Estándares tecnológicos

| ID | Dominio | Estándar | Justificación | Obligatorio |
|---|---|---|---|---|
| TE-01 | APIs | REST + JSON, documentado con OpenAPI 3.0 | Interoperabilidad (RT-18) y consistencia con RS-02 | Sí |
| TE-02 | Autenticación de servicios | OAuth 2.0 / OIDC vía cuentas de servicio de GCP; Workload Identity Federation donde sea posible | Evita llaves estáticas (RT-15) | Sí |
| TE-03 | Cifrado en tránsito | TLS 1.2 o superior en todas las comunicaciones | RT-04 | Sí |
| TE-04 | Infraestructura como Código | Terraform (HCL) | RT-05, reproducibilidad y auditabilidad | Sí |
| TE-05 | Control de versiones y commits | Git con *Conventional Commits*, ramas protegidas en `main` | Trazabilidad de cambios en producción | Sí |
| TE-06 | Logging estructurado | JSON, con campo de identificador de caso (*trace ID*) obligatorio | RT-20 | Sí |
| TE-07 | Contenedores | Imágenes base mínimas (*distroless* o equivalente) | Reduce superficie de ataque (gestión de vulnerabilidades) | Recomendado |
| TE-08 | Cumplimiento normativo | Ley 1581 de 2012, Decreto 1377 de 2013, Resolución CRC 5050 de 2016 | Marco legal aplicable (Fase A/C) | Sí |
| TE-09 | Marco de arquitectura cloud | Google Cloud Architecture Framework (pilares de seguridad, confiabilidad, costo, rendimiento) | Alineación con las buenas prácticas del proveedor ya seleccionado | Recomendado |
| TE-10 | Versionado de APIs | Semántico, con prefijo de versión en la ruta (`/v1/...`) | RT-19, independencia tecnológica | Sí |

## 11. Principios tecnológicos

| ID | Principio | Descripción | Justificación | Implicaciones |
|---|---|---|---|---|
| PT-01 | Serverless-first | Preferir servicios completamente administrados (Cloud Run, Pub/Sub, Firestore, Vertex AI) sobre infraestructura auto-gestionada (VMs, Kubernetes) | Menor costo operativo y de mantenimiento, alineado al presupuesto y al principio *cloud-first* de la Fase A | No se despliega GKE ni VMs para los componentes de esta arquitectura (sección 7.4) |
| PT-02 | Zero Trust para acceso humano | El acceso de personas se controla por identidad y contexto, no por pertenecer a una red específica | Reduce el riesgo de accesos indebidos y elimina la dependencia de VPN para los agentes | La Interfaz de Validación se expone vía IAP, no vía IP interna abierta (sección 7.7) |
| PT-03 | Privacidad desde el diseño | Heredado de la Fase A: anonimización antes de cualquier uso secundario del dato | Cumplimiento de la Ley 1581 de 2012 | Cloud DLP es un paso obligatorio antes de que cualquier dato llegue al bucket de entrenamiento |
| PT-04 | Infraestructura reproducible (IaC) | Toda la infraestructura se declara en Terraform; no hay cambios manuales en producción | Auditabilidad, consistencia entre entornos, recuperación más rápida ante desastres | Cualquier cambio de infraestructura pasa por el pipeline de CI/CD (sección 7.12) |
| PT-05 | Observabilidad por defecto | Todo componente nuevo debe nacer con métricas, logs y trazas, no añadirse después | Detección temprana de degradación (p. ej. aumento de escalamiento a IA) | Se exige instrumentación de Cloud Trace desde el primer despliegue (RT-20) |
| PT-06 | Gestión centralizada de secretos | Ninguna credencial se almacena en código fuente o variables de entorno sin cifrar | Reduce superficie de fuga de credenciales | Secret Manager es la única fuente de credenciales en tiempo de ejecución |
| PT-07 | Independencia tecnológica del modelo de IA | El Motor IA debe poder cambiar de proveedor de modelo sin rediseñar el resto del sistema | Heredado de la Fase A; evita bloqueo (*vendor lock-in*) en el componente más propenso a cambiar de precio/rendimiento | La interfaz entre Motor de Reglas/Motor IA e Interfaz de Validación se mantiene estable aunque cambie el modelo subyacente |
| PT-08 | Degradación segura ante fallas | Ante cualquier falla del sistema automatizado, el proceso cae de vuelta al flujo manual ya existente, nunca se detiene la atención al cliente | Continuidad de negocio de bajo costo, aprovechando que el proceso manual ya es la línea base | Fundamenta la estrategia de DR de la sección 7.14 (ADR-06) |

## 12. Gap Analysis

| ID | Situación Baseline | Situación Target | Gap | Impacto | Prioridad | Acción |
|---|---|---|---|---|---|---|
| GAP-01 | Sin infraestructura como código; cambios manuales (si los hubiera) | Terraform gestiona toda la infraestructura | Ausencia total de IaC | Alto — riesgo de configuración inconsistente entre entornos | Alta | Construir los módulos Terraform desde la Fase 1 de la hoja de ruta (sección 18) |
| GAP-02 | Sin integración automatizada entre sistemas | Pub/Sub + Ingestion Gateway + API Gateway | Falta de capa de integración | Alto — es la base de todo el proyecto | Alta | Ya cubierto por el diseño de esta fase; ejecutar en el Mes 1-2 |
| GAP-03 | Sin observabilidad de sistema | Cloud Monitoring/Logging/Trace + alertas | Ausencia de visibilidad operativa | Alto — sin esto no se puede detectar degradación del modelo ni fallas | Alta | Instrumentar cada servicio desde su primer despliegue (PT-05) |
| GAP-04 | Sin gestión de secretos formal | Secret Manager + Workload Identity Federation | Riesgo de credenciales expuestas | Alto | Alta | Definir la política de secretos antes de escribir el primer servicio |
| GAP-05 | Sin perímetro de seguridad definido | VPC Service Controls | Datos de PQRS sin contención formal | Alto — riesgo de incumplimiento de la Ley 1581 | Alta | Definir el perímetro antes de mover cualquier dato real (Mes 1) |
| GAP-06 | Conectividad CRM–nube no diseñada | Cloud VPN o WIF (condicional, ADR-02) | Postura de red del CRM no confirmada | Medio — bloquea el diseño final de red | Alta | Confirmar con el Gerente de TI antes de finalizar el Mes 1 |
| GAP-07 | Sin estrategia de backup/DR | RPO ≤ 24h / RTO ≤ 4h + degradación segura a manual | Falta de objetivos de recuperación formales | Medio — mitigado por el propio diseño de degradación segura (PT-08) | Media | Configurar exportaciones automáticas desde el primer entorno productivo |
| GAP-08 | Sin gestión de vulnerabilidades | Artifact Registry (escaneo) + Security Command Center | Superficie de ataque no monitoreada | Medio | Media | Habilitar el escaneo antes del primer despliegue a `pqrs-prod` |
| GAP-09 | Sin catálogo de APIs ni gestión de cuotas | API Gateway | Falta de control de consumo y versión | Bajo-Medio | Media | Configurar junto con el despliegue del Ingestion Gateway |
| GAP-10 | Disponibilidad regional de Vertex AI (Gemini) no confirmada en `southamerica-east1` | Requiere validación técnica temprana | Riesgo de latencia cross-region no presupuestada | Medio | Alta | Validar en un *spike* técnico durante la primera semana del Mes 1 (sección 21) |

## 13. Dependencias arquitectónicas

* La arquitectura tecnológica depende de que la Fase C (aplicaciones y datos) no cambie sustancialmente el diseño de tres niveles (reglas → IA → manual); cualquier cambio ahí obliga a revisar el dimensionamiento de Cloud Run y Vertex AI de esta fase.
* Depende de la confirmación del Gerente de TI sobre la postura de red del CRM heredado (GAP-06) para cerrar el diseño final de conectividad (ADR-02).
* Depende de la constitución del comité de gobierno de datos/IA (ya señalada como pendiente en la Fase A, sección 5, y reiterada en la Fase C, sección 4.3) para aprobar formalmente el pipeline de anonimización antes de cualquier entrenamiento real.
* Depende de la disponibilidad confirmada de modelos Vertex AI en la región elegida (GAP-10); de no estar disponibles, esta fase debería revisarse para adoptar `us-central1` como región del componente de IA (con el resto de la plataforma permaneciendo en `southamerica-east1`), lo que introduciría latencia adicional a evaluar contra RT-09.
* Depende de que TelcoLatam mantenga (o adquiera) una suscripción a Google Workspace/Cloud Identity para habilitar IAP (sección 4.6); si no la tiene, el mecanismo de Zero Trust deberá sustituirse por una alternativa (p. ej., Cloud Identity Free tier o federación con un IdP externo vía SAML).

## 14. Matriz de trazabilidad

| Requisito | Capacidad | Componente tecnológico | Decisión arquitectónica | Evidencia |
|---|---|---|---|---|
| RN-01 / RS-01 | Clasificación automática ≥ 85% sin intervención humana | Motor de Reglas + Motor IA (Vertex AI) | ADR-03, ADR-09 | Sección 7.4, 5.4 |
| RN-02 / RS-02 | Integración por microservicios y APIs REST | Cloud Run + API Gateway + Pub/Sub | ADR-04, ADR-08 | Sección 7.4, 7.6 |
| RN-03 / RS-03 | Interfaz de validación con corrección manual | Interfaz de Validación (Cloud Run + Firestore) + IAP | ADR-03 | Sección 7.5, 7.7 |
| RN-04 / RS-04 | Enrutamiento a cola manual bajo el umbral de confianza | Pub/Sub (`pqrs-cola-manual`) | PT-08, ADR-06 | Sección 8.2, 8.5 |
| RN-05 / RS-05 | Cumplimiento de la Ley 1581 de 2012 | Cloud DLP + VPC Service Controls | ADR-09 (CMEK opcional) | Sección 7.7, 7.5 |
| RN-06 / RS-06 | Auditoría y balanceo del dataset | BigQuery (`pqrs_audit`) + Cloud Monitoring | — | Sección 7.5, 7.11 |
| RN-07 / RS-07 | Plantilla oficial y lineamientos sin cambios | CRM heredado + suite ofimática (sin cambios) | — | Sección 6, 4.1 |
| RN-08 / RS-08 | Presupuesto y plazo del MVP | Serverless-first (PT-01), niveles gratuitos de GCP | ADR-01, ADR-04, ADR-08 | Sección 7.4, Fase A §12.3 |
| RS-09 | El Motor de Reglas resuelve sin invocar IA cuando hay certeza | Motor de Reglas (Cloud Run) | — | Sección 7.4, 8.2 |
| RS-10 / Riesgo 3 (Fase A) | Reintentos y cola de contingencia ante indisponibilidad del CRM | Pub/Sub + Ingestion Gateway | ADR-02, ADR-06 | Sección 7.6, 7.14 |

## 15. Decisiones arquitectónicas

**ADR-01 — Selección de región GCP primaria**
- **Contexto:** GCP no tiene una región dentro de Colombia; hay dos regiones sudamericanas disponibles: `southamerica-east1` (São Paulo) y `southamerica-west1` (Santiago).
- **Alternativas consideradas:** (a) `southamerica-east1`, (b) `southamerica-west1`, (c) una región de Norteamérica (`us-central1`) con mayor disponibilidad de servicios pero mayor latencia y posible fricción regulatoria.
- **Decisión adoptada:** `southamerica-east1` como región primaria; `southamerica-west1` como destino de backups.
- **Justificación:** es la región sudamericana con el catálogo de servicios GCP históricamente más completo y madura en soporte, minimizando el riesgo de encontrar servicios no disponibles (aunque persiste el riesgo puntual de Vertex AI, ver GAP-10).
- **Consecuencias:** latencia moderada desde Colombia (a validar); mantiene el dato dentro de Sudamérica.
- **Riesgos:** disponibilidad de Vertex AI (Gemini) no confirmada en esta región (ver ADR-09 y sección 21).

**ADR-02 — Conectividad entre el CRM heredado y GCP**
- **Contexto:** no se sabe si el CRM on-premise tiene salida a internet o si la política de seguridad de TelcoLatam exige conectividad privada.
- **Alternativas consideradas:** (a) Cloud VPN (HA VPN) — privada, mayor costo y complejidad de configuración; (b) llamada HTTPS saliente con Workload Identity Federation — simple y económica, requiere que el CRM tenga egress a internet.
- **Decisión adoptada:** condicional — se recomienda (b) por simplicidad y costo si el CRM tiene egress a internet; de lo contrario, (a).
- **Justificación:** (b) es más económica y rápida de implementar dentro del plazo de 3 meses; (a) es más segura si la política de red de TelcoLatam lo exige.
- **Consecuencias:** el diseño final de red no puede cerrarse hasta confirmar con el Gerente de TI (GAP-06).
- **Riesgos:** un cambio de (b) a (a) durante la implementación añadiría tiempo y costo no presupuestado.

**ADR-03 — Acceso de agentes a la Interfaz de Validación mediante IAP (Zero Trust)**
- **Contexto:** los Agentes de Soporte necesitan acceder a la Interfaz de Validación de forma segura, potencialmente desde distintas ubicaciones.
- **Alternativas consideradas:** (a) acceso solo desde la red corporativa/VPN; (b) Identity-Aware Proxy (IAP) con identidad corporativa.
- **Decisión adoptada:** (b) IAP.
- **Justificación:** evita depender de la topología de red (más flexible para trabajo remoto) y es coherente con el principio Zero Trust (PT-02).
- **Consecuencias:** requiere que TelcoLatam tenga o adquiera Google Workspace/Cloud Identity (sección 4.6).
- **Riesgos:** si TelcoLatam no puede adoptar Cloud Identity a tiempo, se debe caer a la alternativa (a) como plan de contingencia.

**ADR-04 — Cloud Run en lugar de Kubernetes (GKE) para los servicios de clasificación**
- **Contexto:** se necesita una plataforma de cómputo para el Motor de Reglas, el Motor IA y la Interfaz de Validación.
- **Alternativas consideradas:** (a) Google Kubernetes Engine (GKE); (b) Cloud Run (serverless).
- **Decisión adoptada:** (b) Cloud Run.
- **Justificación:** el volumen (>10.000 PQRS/día, tráfico moderado) no justifica la complejidad operativa ni el costo base de un clúster GKE; Cloud Run escala a cero y cubre RT-11/RT-12 con menor esfuerzo.
- **Consecuencias:** menor control de bajo nivel sobre el entorno de ejecución, aceptable dado el perfil de carga.
- **Riesgos:** si el volumen creciera órdenes de magnitud por encima de lo proyectado, podría requerirse revisar esta decisión.

**ADR-05 — Terraform como herramienta de Infraestructura como Código**
- **Contexto:** se requiere una forma reproducible y auditable de desplegar la infraestructura GCP.
- **Alternativas consideradas:** (a) Terraform; (b) Google Cloud Deployment Manager (nativo, pero en desuso); (c) Pulumi.
- **Decisión adoptada:** (a) Terraform.
- **Justificación:** es el estándar de facto para IaC multi-nube, con amplio soporte de la comunidad y del propio proveedor GCP, y facilita una eventual migración parcial si se revisara el principio de independencia tecnológica.
- **Consecuencias:** requiere que el equipo (Arquitecto de TI / Ingeniero Cloud, ya presupuestado en la Fase A) tenga o adquiera competencia en Terraform.
- **Riesgos:** ninguno significativo dado el tamaño del proyecto.

**ADR-06 — Estrategia de continuidad basada en degradación segura hacia el proceso manual**
- **Contexto:** se necesita una estrategia de continuidad de negocio y recuperación ante desastres proporcional al presupuesto del proyecto.
- **Alternativas consideradas:** (a) arquitectura multi-región activa-activa de alto costo; (b) degradación segura hacia el proceso manual ya existente, con backups de datos y recuperación en una sola región.
- **Decisión adoptada:** (b).
- **Justificación:** el proceso manual (línea base) sigue siendo un canal de contingencia válido y ya operativo; invertir en una arquitectura multi-región activa-activa no es proporcional al riesgo ni al presupuesto (~USD 11.270 de inversión incremental).
- **Consecuencias:** RTO/RPO más laxos (sección 7.14) que en un diseño de alta disponibilidad multi-región, pero con cero riesgo de interrupción del servicio al cliente.
- **Riesgos:** si el volumen de PQRS creciera tanto que el proceso manual ya no pudiera absorber una caída prolongada del sistema automatizado, esta decisión debería revisarse.

**ADR-07 — Workload Identity Federation y Secret Manager para gestión de credenciales**
- **Contexto:** se requiere autenticar al CRM heredado y a los servicios internos sin exponer credenciales estáticas.
- **Alternativas consideradas:** (a) llaves de cuenta de servicio JSON estáticas; (b) Workload Identity Federation (sin llaves) + Secret Manager para lo que no pueda federarse.
- **Decisión adoptada:** (b), con (a) como respaldo solo si el CRM no soporta federación.
- **Justificación:** reduce drásticamente el riesgo de fuga de credenciales de larga duración (RT-15).
- **Consecuencias:** mayor complejidad de configuración inicial.
- **Riesgos:** si el CRM heredado no soporta ningún mecanismo de federación moderno, se debe usar (a) con rotación estricta vía Secret Manager.

**ADR-08 — API Gateway en lugar de Apigee para gestión de APIs**
- **Contexto:** se necesita un punto único de gestión de las APIs REST internas (cuotas, autenticación, versión).
- **Alternativas consideradas:** (a) Apigee (gestión de APIs de nivel empresarial); (b) API Gateway (GCP, más simple y económico).
- **Decisión adoptada:** (b).
- **Justificación:** el volumen y la complejidad del proyecto no justifican el costo ni las capacidades avanzadas de Apigee; API Gateway cubre TE-01 y RT-18 suficientemente.
- **Consecuencias:** menos funcionalidades de monetización/analítica de API avanzada, irrelevantes para este proyecto.
- **Riesgos:** ninguno significativo.

**ADR-09 — Ajuste de la expectativa de latencia del Motor IA y evaluación de CMEK**
- **Contexto:** RS-01 (Fase C) describe una respuesta "en el orden de milisegundos", una expectativa realista para el Motor de Reglas pero no necesariamente para una llamada a un modelo de lenguaje.
- **Alternativas consideradas:** (a) mantener la expectativa literal de milisegundos para todo el pipeline; (b) diferenciar la expectativa por componente (reglas vs. IA) con métricas realistas.
- **Decisión adoptada:** (b), formalizada en RT-08/RT-09/RT-10.
- **Justificación:** evita comprometer un SLA técnicamente inalcanzable para la llamada a Vertex AI, sin renunciar al objetivo de negocio de reducir el tiempo de triage en 90% (que se mide contra los 3-5 minutos de la línea base, no contra milisegundos absolutos).
- **Consecuencias:** ninguna sobre el KPI de negocio, que sigue siendo alcanzable.
- **Riesgos:** ninguno; es una aclaración, no un incumplimiento.
- **Nota adicional:** esta misma decisión evalúa Cloud KMS (CMEK) como mejora de cifrado opcional para los buckets de datos anonimizados y BigQuery — no se incluye en el MVP por no ser exigido explícitamente por el marco legal identificado (Ley 1581 no exige CMEK), pero queda documentado como mejora disponible si el Oficial de Protección de Datos lo requiere.

## 16. Riesgos tecnológicos

| ID | Riesgo | Probabilidad | Impacto | Mitigación | Responsable |
|---|---|---|---|---|---|
| TR-01 | El CRM heredado no permite ninguna de las dos opciones de conectividad diseñadas (ni egress a internet ni VPN) | Baja | Alto | Escalar a un adaptador intermedio (agente local que sí pueda alcanzar GCP) como plan C; validar con el Gerente de TI antes del Mes 1 | Gerente de TI / Arquitecto de TI |
| TR-02 | Vertex AI (Gemini 2.5 Flash-Lite) no está disponible en `southamerica-east1` | Media | Medio | Usar el endpoint global/`us-central1` para el Motor IA únicamente, aceptando latencia adicional (ver RT-09) | Arquitecto de TI |
| TR-03 | El diccionario de reglas se desactualiza y aumenta la carga (y el costo) sobre el Motor IA | Media | Bajo-Medio | Monitorear la tasa de escalamiento reglas→IA como señal de alerta (RT-21); revisión periódica del diccionario (heredado de la Fase C) | Científico de Datos |
| TR-04 | Configuración incorrecta de IAM o del perímetro VPC Service Controls expone datos de PQRS | Baja | Alto | Revisión de IAM en cada *pull request* de Terraform; Security Command Center activo | Arquitecto de TI |
| TR-05 | Escalamiento inesperado de costos de Vertex AI por un aumento no previsto en la tasa de casos ambiguos | Media | Medio | Alertas de presupuesto (Cloud Billing Budgets, RT-21); límites de cuota configurados en Vertex AI | Gerente de TI |
| TR-06 | Vulnerabilidades en imágenes de contenedor no detectadas antes de producción | Baja | Medio | Escaneo obligatorio en Artifact Registry como *quality gate* bloqueante | Ingeniero Cloud/Backend |
| TR-07 | Dependencia de un único proveedor cloud (vendor lock-in) | Media | Bajo | Uso de Terraform (portable) y de estándares abiertos (REST/OpenAPI) donde sea posible; el principio de independencia tecnológica (PT-07) ya limita el acoplamiento al modelo de IA específico | Arquitecto Empresarial |
| TR-08 | TelcoLatam no dispone de Google Workspace/Cloud Identity para habilitar IAP | Media | Bajo-Medio | Plan de contingencia: acceso vía VPN + red interna mientras se resuelve (ver ADR-03) | Gerente de TI |
| TR-09 | Retraso en la constitución del comité de gobierno de datos/IA (pendiente desde la Fase A) bloquea la validación del pipeline de anonimización | Media | Medio | Escalar la constitución del comité como hito explícito del Mes 1 de la hoja de ruta (sección 18) | Director de Servicio al Cliente |

## 17. Consideraciones de implementación

* **Orden de construcción recomendado:** (1) proyectos GCP y jerarquía de IAM base → (2) VPC, conectividad con el CRM (una vez resuelto GAP-06) y VPC Service Controls → (3) Pub/Sub y Ingestion Gateway → (4) Motor de Reglas → (5) pipeline de datos (Cloud Storage + Cloud DLP) y Motor IA → (6) Interfaz de Validación (Cloud Run + Firestore + IAP) → (7) observabilidad y alertas → (8) BigQuery y tableros de KPI.
* **Entornos:** `pqrs-dev` para desarrollo individual, `pqrs-staging` para pruebas de integración y de carga (incluida una prueba de carga que valide RT-11 antes del despliegue a producción), `pqrs-prod` para producción.
* **Pruebas de carga:** dado que no existe un perfil de tráfico horario real (sección 4.6), se recomienda instrumentar `pqrs-staging` con tráfico sintético antes del piloto, para validar RT-08/RT-09/RT-10 con datos reales de latencia de Vertex AI en la región elegida.
* **Bootstrap de secretos:** las primeras credenciales (p. ej., la clave inicial de Workload Identity Federation) deben crearse manualmente una sola vez por un administrador, y todo lo posterior debe fluir por el pipeline de CI/CD.
* **Capacitación:** el Ingeniero Cloud/Backend y el Científico de Datos ya presupuestados (Fase A, sección 12.3) deben adquirir o confirmar competencia en Terraform, Cloud Run y Vertex AI antes del inicio del Mes 2.

## 18. Impacto sobre la hoja de ruta

La hoja de ruta de 3 meses ya definida en la Fase B (sección 8) y detallada con componentes de aplicaciones/datos en la Fase C (sección 8) se mantiene sin cambios en su duración total; esta fase añade el detalle tecnológico por mes:

| Fase | Horizonte | Componentes tecnológicos añadidos por la Fase D |
|---|---|---|
| Fase 1 — Cimentación | Mes 1 | Confirmación de la postura de red del CRM (GAP-06); creación de proyectos GCP y jerarquía de IAM; diseño de la VPC y del perímetro VPC Service Controls; validación de disponibilidad regional de Vertex AI (GAP-10); constitución del comité de gobierno de datos/IA |
| Fase 2 — Construcción | Mes 2 | Módulos Terraform para Pub/Sub, Cloud Run e IAM; pipeline de CI/CD (Cloud Build + Artifact Registry); configuración de Secret Manager y Workload Identity Federation; instrumentación de observabilidad desde el primer despliegue |
| Fase 3 — Integración y despliegue | Mes 3 | Conectividad final CRM↔GCP (Cloud VPN o WIF); configuración de IAP para la Interfaz de Validación; pruebas de carga en `pqrs-staging`; despliegue gradual a `pqrs-prod`; alertas de presupuesto y de degradación activas antes del piloto |

## 19. Recomendaciones

1. Confirmar con el Gerente de TI, antes de iniciar la Fase 1, si el CRM heredado tiene salida a internet — esta única respuesta determina si se construye Cloud VPN o la ruta más simple de Workload Identity Federation (ADR-02, GAP-06).
2. Ejecutar un *spike* técnico de una semana para confirmar la disponibilidad de Vertex AI (Gemini 2.5 Flash-Lite) en `southamerica-east1` antes de comprometer el dimensionamiento final (GAP-10).
3. Adoptar Terraform desde el primer commit de infraestructura, no como una regularización posterior — evita deuda técnica de configuración manual.
4. Priorizar la constitución del comité de gobierno de datos/IA (pendiente desde la Fase A) en paralelo a la construcción técnica, no después de ella, para no bloquear el pipeline de anonimización en el Mes 2.
5. Tratar la estrategia de continuidad (degradación segura al proceso manual, ADR-06) como una ventaja competitiva a comunicar a los stakeholders, no como una limitación: reduce el costo de DR sin sacrificar la continuidad del servicio al cliente.

## 20. Conclusiones

Esta Fase D dimensiona técnicamente la arquitectura en GCP ya seleccionada en la Fase A y detallada como arquitectura de aplicaciones y datos en la Fase C: selecciona una región primaria sudamericana, diseña la conectividad con el CRM heredado bajo dos alternativas condicionadas a una confirmación pendiente, establece un perímetro de seguridad de mínimo privilegio con acceso Zero Trust para los agentes, define una estrategia de CI/CD e Infraestructura como Código, y adopta una estrategia de continuidad de bajo costo apoyada en el propio proceso manual existente como mecanismo de degradación segura. Con esta arquitectura tecnológica documentada, el proyecto "IA-PQRS Smart Routing" queda listo para avanzar hacia la Fase E (Oportunidades y Soluciones) y la Fase F (Planificación de la Migración), sujeto a resolver los puntos pendientes listados en la sección 21.

## 21. Información pendiente / TBD

| # | Punto pendiente | Por qué es necesario | Responsable sugerido |
|---|---|---|---|
| 1 | Postura de red del CRM heredado (¿tiene egress a internet? ¿su política de seguridad exige conectividad privada?) | Determina la decisión final del ADR-02 (Cloud VPN vs. Workload Identity Federation); tiene un impacto directo de ≈ US$80-105/mes sobre el presupuesto operativo (sección 7.15) | Gerente de TI |
| 2 | Disponibilidad de Vertex AI (Gemini 2.5 Flash-Lite) en `southamerica-east1` | Determina si el Motor IA opera en la misma región o requiere una llamada cross-region (impacta RT-09) | Arquitecto de TI |
| 3 | Especificaciones técnicas del hardware/infraestructura actual del CRM heredado | No documentadas en las Fases A-C; necesarias para completar la sección 6.2-6.6 con precisión | Gerente de TI |
| 4 | Disponibilidad de Google Workspace/Cloud Identity en TelcoLatam | Determina si IAP puede implementarse tal como se diseñó (ADR-03) o si se requiere una alternativa | Gerente de TI |
| 5 | Perfil real de tráfico horario de las PQRS (picos, estacionalidad) | Necesario para calibrar los límites de instancias de Cloud Run (sección 7.4) con precisión, más allá del supuesto de la sección 4.6 | Director de Servicio al Cliente |
| 6 | Requisitos de residencia de datos más allá de la Ley 1581 (p. ej., políticas internas de TelcoLatam) | Podría cambiar la decisión de región (ADR-01) o exigir CMEK (ADR-09) | Oficial de Protección de Datos |
| 7 | Fecha de constitución del comité de gobierno de datos/IA | Pendiente desde la Fase A (sección 5); bloquea la validación formal del pipeline de anonimización | Director de Servicio al Cliente |
| 8 | Tolerancia real de negocio a RTO/RPO (¿4 horas y 24 horas son aceptables para el Director de Servicio al Cliente?) | Los valores de la sección 5.7 son una propuesta técnica razonable, no una cifra confirmada por el negocio | Director de Servicio al Cliente |
