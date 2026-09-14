# INF331 — Pruebas de Software: testing aumentado con IA

Primera versión del material introductorio para el curso INF331 de la UTFSM.

El caso práctico transversal es un **sistema de reserva de salas de estudio universitarias**. Sus reglas son ficticias y tienen fines docentes; no representan políticas de la UTFSM.

Recorrido: **Necesidad → requerimientos → historias de usuario → criterios de aceptación → casos de prueba → exploración y automatización con IA.**

## Índice

- [Prompt engineering](#prompt-engineering)
  - [Primeros prompts](#primeros-prompts)
  - [Multiprompts](#multiprompts)
- [Context Engineering](#context-engineering)
- [Requerimientos](#requerimientos)
  - [Requerimientos funcionales](#requerimientos-funcionales)
  - [Requerimientos no funcionales](#requerimientos-no-funcionales)
  - [Reglas de negocio](#reglas-de-negocio)
- [Historias de usuario](#historias-de-usuario)
- [Criterios de aceptación](#criterios-de-aceptación)
- [Casos de prueba](#casos-de-prueba)
- [MCP — Model Context Protocol](#mcp--model-context-protocol)
  - [Playwright MCP](#playwright-mcp)
- [Trabajo integrador](#trabajo-integrador)

## Prompt engineering

Prompt engineering es el diseño y evaluación de instrucciones para orientar las respuestas de un modelo de lenguaje hacia una tarea concreta.

En pruebas de software podemos utilizarlo para identificar ambigüedades, proponer escenarios, diseñar datos de prueba, revisar cobertura y generar borradores de pruebas automatizadas.

**Una respuesta convincente no demuestra que una prueba sea correcta.** Debemos comprobar que sus resultados esperados corresponden a la especificación y que sus pasos permiten verificar el comportamiento relevante.

| Elemento del prompt | Pregunta que responde |
|---|---|
| Objetivo | ¿Qué tarea debe realizar el modelo? |
| Contexto | ¿Qué sistema y comportamiento estamos analizando? |
| Información de entrada | ¿Qué requerimientos, reglas o código debe utilizar? |
| Restricciones | ¿Qué debe respetar y qué información no puede inventar? |
| Formato de salida | ¿Cómo necesitamos recibir el resultado? |
| Criterios de calidad | ¿Cómo evaluaremos la respuesta? |

### Primeros prompts

Consideremos esta instrucción:

```text
Genera pruebas para reservar una sala.
```

Es demasiado abierta. No especifica restricciones de duración, disponibilidad, usuarios autorizados ni resultados esperados. El modelo puede completar esos vacíos con supuestos que parecen razonables, pero que nadie ha aprobado.

Una versión más útil sería:

```text
Diseña casos de prueba para un sistema de reserva de salas universitarias.

Especificación del ejercicio:
- Solo estudiantes autenticados y habilitados pueden reservar.
- La duración debe ser un número entero entre 30 y 120 minutos,
  incluyendo ambos extremos.
- No se permiten reservas que se superpongan para una misma sala.
- Cada estudiante puede mantener como máximo 2 reservas activas.
- Una reserva está activa si no está cancelada y aún no ha finalizado.

Tarea:
Propón casos positivos, negativos y de valores límite.

Formato:
ID | Objetivo | Precondiciones | Datos | Pasos |
Resultado esperado | Regla cubierta

Restricciones:
- Basa cada resultado esperado en la especificación.
- No inventes mensajes exactos, endpoints ni elementos de interfaz.
- Separa las preguntas pendientes de los casos verificables.
```

La mejora consiste en hacer explícitas las condiciones que determinan si una respuesta es útil y comprobable.

Podemos practicar dos modalidades:

- **Zero-shot:** solicitar la tarea sin ejemplos de respuesta.
- **Few-shot:** incluir algunos ejemplos representativos del formato y nivel de detalle esperado.

Los ejemplos deben ser correctos: un ejemplo defectuoso puede propagar el mismo error al resto de la respuesta.

**Actividad:** ejecutar el prompt inicial y el mejorado sobre la misma especificación. Comparar cantidad de supuestos inventados, claridad de resultados esperados y cobertura de límites. Registrar el modelo, la fecha y los prompts utilizados.

### Multiprompts

En este curso utilizaremos *multiprompts* para referirnos a una **secuencia de prompts que divide una tarea compleja en etapas verificables**.

| Etapa | Objetivo | Resultado |
|---|---|---|
| 1. Analizar | Detectar información faltante y contradicciones | Preguntas pendientes |
| 2. Especificar | Organizar los acuerdos del sistema | Requerimientos identificados |
| 3. Diseñar | Derivar escenarios de prueba | Casos candidatos |
| 4. Revisar | Buscar omisiones, duplicados y expectativas incorrectas | Observaciones |
| 5. Consolidar | Incorporar correcciones justificadas | Casos revisados |

Ejemplo de secuencia:

```text
Prompt 1:
Analiza esta descripción del sistema.
Identifica ambigüedades, contradicciones y decisiones pendientes.
No completes los vacíos con reglas inventadas.
```

```text
Prompt 2:
Usa la descripción y las respuestas validadas que adjunto.
Redacta requerimientos con ID, descripción, fuente
y método de verificación.
Mantén las decisiones no resueltas en una sección aparte.
```

```text
Prompt 3:
A partir de estos requerimientos revisados, diseña casos
con particiones de equivalencia y valores límite.
Vincula cada caso con los requerimientos que verifica.
```

```text
Prompt 4:
Revisa los casos contra la especificación original.
Identifica expectativas sin respaldo, omisiones y duplicados.
Para cada observación, cita el ID del caso y la regla relevante.
```

Cada etapa debe recibir los artefactos necesarios; no debemos asumir que el modelo conserva correctamente toda la conversación.

La revisión con IA es una ayuda, pero **la coincidencia entre dos respuestas no constituye validación independiente**. Ambas pueden reproducir el mismo supuesto erróneo.

**Actividad:** comparar un flujo de un solo prompt con esta secuencia. Evaluar calidad de casos, esfuerzo de revisión y errores que llegaron a la última etapa.

## Context Engineering

Context Engineering consiste en seleccionar, organizar y mantener la información que el modelo necesita para trabajar: instrucciones, documentos, ejemplos, historial relevante y resultados de herramientas. Complementa el diseño del prompt al ocuparse del contexto completo disponible en cada interacción. [Referencia de Anthropic](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents)

En testing, una misma instrucción puede producir resultados muy distintos según reciba:

- Una descripción informal del producto.
- Requerimientos identificados y revisados.
- Reglas de negocio vigentes.
- Contratos de API.
- Código relevante.
- Reportes de defectos y evidencia de ejecución.

Para el caso práctico, podemos preparar un paquete de contexto:

```text
contexto/
  descripcion-sistema.md
  glosario.md
  requerimientos.md
  reglas-negocio.md
  decisiones-pendientes.md
  datos-prueba.md
```

Cada documento debería indicar su versión o fecha y distinguir información aprobada de propuestas.

```text
Utiliza los documentos adjuntos para diseñar pruebas de reservas.

La especificación aprobada define el comportamiento esperado.
El código describe la implementación actual y puede contener defectos.

Si encuentras discrepancias:
- Identifica los documentos o fragmentos involucrados.
- Explica cómo afectan al diseño de pruebas.
- No resuelvas el conflicto inventando una decisión de negocio.

Devuelve:
1. Fuentes utilizadas.
2. Conflictos o vacíos.
3. Casos derivados de reglas explícitas.
```

Un error frecuente es agregar todo el repositorio sin seleccionar lo relevante. Otro es conservar documentos antiguos que contradicen decisiones recientes.

También debemos distinguir las instrucciones del trabajo de los datos analizados: una página o archivo recibido como evidencia no debería cambiar por sí mismo el objetivo de la tarea.

**Actividad:** diseñar casos primero con una descripción breve y después con el paquete de contexto. Identificar qué casos mejoraron y qué información produjo esa mejora.

## Requerimientos

Un requerimiento expresa una necesidad, capacidad o restricción que el sistema debe satisfacer.

Para probarlo necesitamos saber qué significa cumplirlo. Por eso conviene redactar requerimientos identificables, comprensibles, consistentes y verificables.

```text
ID:
Tipo:
Descripción:
Fuente:
Justificación:
Prioridad:
Método de verificación:
Preguntas pendientes:
```

“Verificable” no significa necesariamente “automatizable”: algunos aspectos requieren inspección, mediciones especializadas o evaluación con personas.

### Requerimientos funcionales

Describen **qué comportamiento debe ofrecer el sistema**, incluyendo respuestas ante situaciones válidas y errores.

| ID | Requerimiento funcional |
|---|---|
| RF-01 | El sistema debe mostrar las salas disponibles para un intervalo solicitado. |
| RF-02 | El sistema debe permitir crear una reserva cuando se cumplen las reglas de elegibilidad, duración, disponibilidad y cantidad máxima. |
| RF-03 | El sistema debe rechazar una solicitud que incumpla esas reglas, informar la causa y no crear la reserva. |
| RF-04 | El sistema debe permitir que un estudiante cancele una reserva propia antes de su inicio. |

“Gestionar reservas” resulta demasiado amplio para orientar pruebas. Conviene separar creación, consulta, cancelación y tratamiento de errores.

```text
Revisa estos requerimientos funcionales.

Para cada uno:
- Identifica el actor, la acción y el resultado observable.
- Señala términos ambiguos.
- Evalúa si se puede verificar.
- Propón una redacción alternativa sin agregar funcionalidades.
- Separa toda decisión que requiera validación.
```

**Actividad:** transformar cinco frases informales en requerimientos verificables. Intercambiarlos con otro equipo para detectar interpretaciones diferentes.

### Requerimientos no funcionales

Describen atributos de calidad y restricciones bajo los cuales debe operar el sistema.

Para verificarlos es necesario establecer **métrica, umbral, condiciones y método de evaluación**. “El sistema debe ser rápido” no permite decidir objetivamente si cumple.

Ejemplos propuestos para el laboratorio:

| ID | Requerimiento | Verificación |
|---|---|---|
| RNF-01 | El percentil 95 de la latencia de consulta de disponibilidad será ≤ 2 s, con 100 usuarios virtuales, 10.000 reservas precargadas y 10 minutos de medición después de 2 minutos de calentamiento. | Prueba de carga en un entorno documentado y con una mezcla de solicitudes fija. |
| RNF-02 | El flujo de reserva podrá completarse usando solo teclado, con foco visible y sin bloqueos de navegación. | Recorrido manual del flujo completo. |
| RNF-03 | Los registros de aplicación no almacenarán contraseñas ni tokens de sesión en texto legible. | Inspección de registros generados durante autenticación y errores. |

Estos valores son decisiones didácticas, no exigencias institucionales.

Algunos aspectos de calidad también producen funciones concretas. Por ejemplo, una política de seguridad puede exigir controles específicos de autorización. La clasificación ayuda a organizar el trabajo, pero no debe ocultar esas relaciones.

```text
Revisa los siguientes requerimientos no funcionales.
Identifica métricas, umbrales, condiciones y métodos faltantes.
Puedes sugerir valores para discusión, pero márcalos como propuestas.
No los presentes como acuerdos aprobados.
```

**Actividad:** reescribir “rápido”, “seguro” y “fácil de usar” como condiciones evaluables.

### Reglas de negocio

Definen políticas y restricciones del dominio. Un requerimiento funcional establece cómo debe responder el sistema para hacerlas cumplir.

| ID | Regla |
|---|---|
| RN-01 | Solo estudiantes autenticados y habilitados pueden reservar. |
| RN-02 | La duración es un número entero entre 30 y 120 minutos, inclusive. |
| RN-03 | No pueden existir reservas no canceladas superpuestas para una misma sala. |
| RN-04 | Un estudiante puede mantener como máximo 2 reservas activas. |
| RN-05 | Una reserva está activa si no está cancelada y su hora de término es posterior al instante actual. |

Para RN-03 utilizaremos intervalos **[inicio, fin)**: incluyen el inicio y excluyen el fin. Una reserva de 10:00 a 11:00 permite que otra comience a las 11:00.

Dos intervalos se superponen cuando:

```text
inicioA < finB Y inicioB < finA
```

Esta definición permite derivar pruebas precisas de solapamiento.

**Actividad:** construir una tabla de decisión que combine elegibilidad, duración válida, disponibilidad y cantidad de reservas. Indicar cuándo se acepta la solicitud. Si fallan varias reglas, dejar pendiente la prioridad de los mensajes hasta que se defina.

## Historias de usuario

Una historia de usuario describe una necesidad desde la perspectiva de quien obtiene valor del sistema.

```text
Como [tipo de usuario],
quiero [capacidad],
para [beneficio].
```

Ejemplo:

```text
HU-01 — Reservar una sala

Como estudiante habilitado,
quiero reservar una sala disponible para un intervalo,
para organizar una sesión de estudio.
```

La historia inicia una conversación. Para implementarla y probarla debemos complementarla con reglas, ejemplos y criterios de aceptación.

```text
Requerimientos relacionados: RF-01, RF-02, RF-03.
Reglas relacionadas: RN-01 a RN-05.
Fuera de alcance: reservas recurrentes.
```

Una historia útil expresa valor, tiene un alcance manejable y puede verificarse. “Como desarrollador, quiero crear una tabla SQL” describe una tarea técnica; normalmente conviene vincularla a la capacidad de usuario que permite implementar.

```text
A partir de estos requerimientos, propone historias de usuario.

Para cada historia incluye:
- ID.
- Actor, capacidad y beneficio.
- Requerimientos relacionados.
- Reglas aplicables.
- Preguntas pendientes.

No agregues perfiles ni funcionalidades ausentes en las fuentes.
Separa las tareas técnicas de las historias.
```

**Actividad:** dividir “Como estudiante quiero gestionar mis reservas” en historias que entreguen resultados observables y puedan evaluarse por separado.

## Criterios de aceptación

Los criterios de aceptación establecen condiciones observables que debe satisfacer una historia para considerarse aceptada.

Para HU-01:

| ID | Criterio |
|---|---|
| CA-01 | Una solicitud que cumple todas las reglas crea exactamente una reserva y muestra su confirmación. |
| CA-02 | Una duración fuera del rango permitido se rechaza sin crear una reserva. |
| CA-03 | Una solicitud superpuesta con una reserva existente de la misma sala se rechaza. |
| CA-04 | Un estudiante con 2 reservas activas no puede crear una tercera. |
| CA-05 | Una reserva puede comenzar exactamente cuando termina otra en la misma sala. |

Podemos expresarlos mediante ejemplos en Gherkin. `Dado` describe el estado inicial, `Cuando` la acción y `Entonces` el resultado esperado. Un esquema de escenario permite repetir el mismo comportamiento con distintos datos. [Referencia de Gherkin](https://cucumber.io/docs/gherkin/reference/)

```gherkin
# language: es
Característica: Reserva de salas de estudio

  Escenario: Crear una reserva válida
    Dado un estudiante autenticado y habilitado
    Y que no tiene reservas activas
    Y una sala disponible entre las 10:00 y las 11:00
    Cuando solicita reservar esa sala para ese intervalo
    Entonces se crea exactamente una reserva a su nombre
    Y se muestra una confirmación

  Escenario: Permitir reservas consecutivas
    Dado una reserva de la sala A entre las 10:00 y las 11:00
    Y otro estudiante autenticado y habilitado sin reservas activas
    Y que la sala A está disponible entre las 11:00 y las 12:00
    Cuando solicita reservar la sala A entre las 11:00 y las 12:00
    Entonces se crea la nueva reserva
```

Los criterios indican qué debe cumplirse; los casos de prueba añaden datos, preparación y pasos específicos. Un criterio puede requerir varios casos. Los escenarios anteriores son ejemplos de especificación: requieren implementación de sus pasos para ejecutarse con Cucumber.

**Actividad:** redactar criterios para la cancelación de reservas. Detectar decisiones todavía ausentes, como el comportamiento de una segunda solicitud de cancelación.

## Casos de prueba

Un caso de prueba describe cómo verificar un comportamiento mediante precondiciones, datos, acciones y resultados esperados.

```text
ID:
Objetivo:
Requerimientos y criterios relacionados:
Precondiciones:
Datos:
Pasos:
Resultado esperado:
Postcondiciones o limpieza:
Prioridad:
```

Ejemplo desarrollado:

```text
ID: CP-01
Objetivo: Verificar el rechazo de una tercera reserva activa.
Trazabilidad: RF-03, RN-04, CA-04.

Precondiciones:
- Estudiante autenticado y habilitado.
- Tiene exactamente 2 reservas activas.
- La sala solicitada está disponible.
- La duración solicitada es de 60 minutos.

Pasos:
1. Consultar la disponibilidad de la sala.
2. Seleccionar el intervalo disponible.
3. Enviar la solicitud.
4. Consultar nuevamente las reservas del estudiante.

Resultado esperado:
- Se rechaza la solicitud por alcanzar el máximo permitido.
- No se crea una nueva reserva.
- Las 2 reservas existentes permanecen sin cambios.

Limpieza:
Restaurar los datos de prueba del estudiante.
```

Técnicas que aplicaremos:

| Técnica | Aplicación |
|---|---|
| Particiones de equivalencia | Duraciones inválidas inferiores, válidas e inválidas superiores. |
| Valores límite | Probar 29, 30, 31, 119, 120 y 121 minutos. |
| Tablas de decisión | Combinar elegibilidad, disponibilidad y cantidad de reservas. |
| Transiciones de estado | Verificar cambios entre reserva activa, cancelada y finalizada. |
| Pruebas basadas en riesgos | Priorizar reservas duplicadas y accesos no autorizados. |

Matriz inicial, suponiendo que las demás condiciones son válidas:

| Caso | Condición | Resultado esperado |
|---|---|---|
| CP-02 | Duración de 29 minutos | Rechazo |
| CP-03 | Duración de 30 minutos | Aceptación |
| CP-04 | Duración de 120 minutos | Aceptación |
| CP-05 | Duración de 121 minutos | Rechazo |
| CP-06 | Reserva existente 10:00–11:00; solicitud 10:30–11:30 | Rechazo |
| CP-07 | Reserva existente 10:00–11:00; solicitud 11:00–12:00 | Aceptación |
| CP-08 | Estudiante con 1 reserva activa | Aceptación; queda con 2 |
| CP-09 | Dos estudiantes solicitan simultáneamente la misma sala e intervalo | Se acepta exactamente una solicitud; queda una sola reserva |

CP-09 verifica que RN-03 se mantenga también ante concurrencia. Conviene probar este riesgo en API o integración, con control de sincronización, además del flujo visible en la interfaz.

El **oráculo de prueba** es la referencia utilizada para decidir si el resultado es correcto. Aquí corresponde a las reglas y criterios revisados. Copiar el comportamiento actual de la aplicación como resultado esperado puede convertir un defecto en una expectativa de prueba.

```text
Deriva casos desde estos criterios y reglas.

Aplica particiones de equivalencia, valores límite
y tablas de decisión cuando corresponda.

Para cada caso:
- Identifica la técnica utilizada.
- Cita el criterio o regla que respalda la expectativa.
- Indica el nivel de prueba más apropiado.
- Separa las decisiones pendientes.

No declares que las pruebas pasaron: esta tarea solo diseña casos.
```

**Actividad:** entregar una matriz de trazabilidad entre requerimientos, criterios y casos. Tener un caso asociado a cada requerimiento mide trazabilidad, pero no demuestra cobertura exhaustiva.

## MCP — Model Context Protocol

MCP es un protocolo para conectar aplicaciones de IA con herramientas y fuentes de contexto mediante una interfaz común.

Su arquitectura distingue:

- **Host:** aplicación de IA que coordina la interacción.
- **Cliente MCP:** componente que mantiene la conexión con un servidor.
- **Servidor MCP:** componente que expone capacidades.

Entre esas capacidades están las **herramientas**, que permiten ejecutar operaciones; los **recursos**, que proporcionan contenido; y los **prompts**, que ofrecen plantillas reutilizables. [Arquitectura oficial de MCP](https://modelcontextprotocol.io/docs/learn/architecture)

```text
Estudiante solicita una verificación
        ↓
Aplicación de IA interpreta el objetivo
        ↓
Cliente MCP invoca una herramienta del servidor
        ↓
La herramienta interactúa con el sistema
        ↓
La aplicación recibe resultados y evalúa la evidencia
```

MCP facilita la conexión. La calidad de una prueba sigue dependiendo de su especificación, de las acciones ejecutadas y de la interpretación del resultado.

**Actividad:** dibujar este flujo para consultar un requerimiento y verificarlo en una aplicación web. Identificar qué componente aporta contexto, cuál realiza acciones y dónde se conserva la evidencia.

### Playwright MCP

Playwright MCP es un servidor que permite a una aplicación de IA interactuar con un navegador mediante Playwright. Utiliza representaciones estructuradas de accesibilidad para identificar e interactuar con elementos de páginas web.

La documentación presenta esta configuración general para clientes compatibles:

```json
{
  "mcpServers": {
    "playwright": {
      "command": "npx",
      "args": ["@playwright/mcp@latest"]
    }
  }
}
```

La ubicación y estructura exacta de configuración dependen del cliente. El proyecto declara Node.js 18 o superior como requisito; para el laboratorio conviene utilizar una versión soportada y registrar las versiones efectivamente utilizadas. [Documentación de Playwright MCP](https://github.com/microsoft/playwright-mcp)

Para repetir el laboratorio durante el semestre, se debería fijar una versión validada del paquete en lugar de mantener `latest`.

**Laboratorio propuesto**

Preparación: disponer de una aplicación de reservas de prueba, usuarios ficticios, datos reiniciables y una dirección de acceso definida por el docente. Este material describe el ejercicio; no incluye todavía la aplicación de reservas.

```text
Usa Playwright MCP para verificar CA-01 en la aplicación de laboratorio.

Precondiciones:
- Usuario de prueba autenticado y habilitado.
- Sin reservas activas.
- Sala disponible para un intervalo de 60 minutos.

Procedimiento:
1. Inspecciona la página e identifica los controles disponibles.
2. Ejecuta el flujo de reserva.
3. Comprueba la confirmación.
4. Consulta las reservas y verifica que se creó exactamente una.

Restricciones:
- Opera solo sobre la aplicación y los datos del laboratorio.
- No inventes controles ni resultados.
- Si una precondición no se cumple, informa el bloqueo.

Informe:
Acciones realizadas | Resultado esperado | Resultado observado |
Evidencia | Estado: aprobado, fallido o bloqueado
```

Después de explorar, el estudiante debe transformar el flujo relevante en una prueba automatizada revisable. Para ello se utilizarán localizadores basados en roles o etiquetas, aserciones que esperen el estado requerido y datos aislados entre pruebas. [Buenas prácticas de Playwright](https://playwright.dev/docs/best-practices)

| Entregable | Propósito |
|---|---|
| Sesión de exploración con Playwright MCP | Investigar y verificar un flujo con asistencia de IA. |
| Prueba versionada con Playwright Test | Repetir una verificación mediante código y aserciones explícitas. |

Una navegación exitosa no demuestra por sí sola que se verificó un criterio. El informe debe mostrar qué se comprobó y con qué evidencia.

**Actividad:** explorar un caso positivo y uno negativo, documentar los hallazgos y automatizar uno de ellos. La automatización debe poder ejecutarse de nuevo desde un estado conocido.

## Trabajo integrador

Cada equipo desarrollará el diseño de pruebas del sistema de reservas y entregará:

1. Contexto del sistema y decisiones pendientes.
2. Requerimientos funcionales, no funcionales y reglas de negocio.
3. Historias de usuario con criterios de aceptación.
4. Casos de prueba y matriz de trazabilidad.
5. Registro de prompts y correcciones humanas.
6. Evidencia de exploración con Playwright MCP.
7. Una prueba automatizada y su resultado real de ejecución.

Rúbrica inicial propuesta:

| Dimensión | Peso |
|---|---:|
| Claridad y verificabilidad de requerimientos | 20 % |
| Coherencia de historias y criterios | 20 % |
| Diseño de casos y cobertura de riesgos | 25 % |
| Trazabilidad | 15 % |
| Evaluación crítica de las respuestas de IA | 10 % |
| Evidencia y reproducibilidad | 10 % |

La evaluación debe valorar especialmente la capacidad de **detectar y corregir errores de la IA**, justificar los resultados esperados y distinguir entre pruebas propuestas, ejecutadas y aprobadas.
