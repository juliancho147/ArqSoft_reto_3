# Memoria del proyecto — Reto 3: Facilidad de Integración y Modificación

> Archivo de memoria/trabajo para este reto. Lo iremos actualizando a medida que avancemos (decisiones, diseños, pendientes). Curso: ARTI4109 - Arquitectura de Software, Maestría en Arquitecturas de TI (MATI), Universidad de los Andes.

## 1. Enunciado (resumen del PDF `ARTI4109-reto3-v2.pdf`)

**Objetivo del reto:** Diseñar la arquitectura de software del sistema propuesto e implementar las decisiones de arquitectura definidas, validando el cumplimiento de los requerimientos de calidad.

**Contexto — Holding XYZ:**
- Holding de empresas de salud compuesta por:
  - 1 EPS (aseguradora de salud obligatoria)
  - 5 IPS (3 clínicas + 2 hospitales)
  - 1 aseguradora de salud voluntaria (medicina prepagada)
  - 1 empresa de seguros de vida
- Cada empresa fue adquirida en momentos distintos y tiene sus propios sistemas (ERP, Callcenter, CRM/manejo de clientes, etc.) → **sistemas heterogéneos, difíciles de integrar**.
- Hoy la integración es manual: las empresas envían **archivos CSV** y la holding los procesa a mano → alto costo y tiempos largos de procesamiento.
- Cada cambio en los sistemas de análisis de la holding toma más tiempo/costo del esperado.

**El reto (requerimientos funcionales/negocio):**
- Integrar la mayor cantidad posible de sistemas empresariales existentes, a nivel de:
  - Clientes
  - Contratos
  - Proveedores
  - Ventas
  - Ingresos
  - Egresos
  - Productos adquiridos por clientes
- Debe soportar la adquisición futura de nuevas compañías (sector turismo y salud), es decir, **extensibilidad** para integrar sistemas de diferentes proveedores que ya están en producción.
- La integración es crítica porque se deben poder **reemplazar componentes tecnológicos en producción en cualquier momento** (bajo acoplamiento / intercambiabilidad).
- Capacidades de negocio deseadas, lo más rápido posible:
  - Base de datos unificada
  - Ventas cruzadas
  - Información unificada de proveedores y contratos
  - Ingresos y egresos unificados de la holding

**Requerimientos de modificabilidad (calidad):**
- Cambios **frecuentes** en los algoritmos que interpretan los datos provenientes de las diferentes empresas.
- Cambios **frecuentes** en la presentación de informes para los directivos de XYZ.
- Las empresas de la holding seguirán cambiando sus propios sistemas de información → esto impacta directamente los **datos y la lógica de negocio** de las aplicaciones de la holding que interpretan esos datos.

**Atributos de calidad centrales del reto:**
1. **Integrabilidad** (integration): conectar sistemas heterogéneos, actuales y futuros, con bajo costo de integración.
2. **Modificabilidad**: tolerar cambios frecuentes en algoritmos de interpretación de datos, en presentación de reportes, y en los sistemas fuente de cada empresa, sin alto impacto/costo.
3. Reemplazabilidad de componentes en producción (deploy sin downtime / bajo acoplamiento).

## 2. Estado del trabajo

- [x] Leído y resumido el enunciado del reto (este documento).
- [x] Definidos enunciado general, alcance, propósito, stakeholders y estructura Épicas/Features/HUs (ver sección 3).
- [~] Definir escenarios de calidad (QAS): ASR-MOD-1, MOD-2, INT-1 e INT-2 confirmados (4 ASRs cerrados) (ver sección 4).
- [ ] Proponer estilo(s) de arquitectura (candidatos a explorar: integración vía ESB/mensajería, microservicios, arquitectura basada en eventos, patrón adapter/ETL por empresa fuente, modelo de datos canónico).
- [~] Diseñar vistas de arquitectura: las cinco pestañas hechas: componentes, secuencias INT-1/INT-2, información y módulos (ver 9.3 a 9.7).
- [ ] Validar decisiones con tácticas de arquitectura (p. ej. tácticas de modificabilidad: encapsular, usar interfaz intermediaria, restringir dependencias; tácticas de integrabilidad: adaptadores, formato de datos canónico, orquestación/coreografía).
- [ ] Implementación/prototipo que valide las decisiones.

## 3. Definiciones acordadas

### 3.1 Enunciado general del problema
La holding de salud XYZ (EPS, 5 IPS, medicina prepagada y seguros de vida) está formada por empresas adquiridas en distintos momentos, cada una con sistemas empresariales heterogéneos (ERP, Callcenter, manejo de clientes, etc.) difíciles de integrar; hoy la información se intercambia mediante archivos CSV procesados manualmente, lo que genera altos costos de integración y largos tiempos de procesamiento, y cada cambio en los sistemas de análisis toma más tiempo y costo del esperado. La arquitectura debe integrar la mayor cantidad posible de estos sistemas (clientes, contratos, proveedores, ventas, ingresos, egresos y productos adquiridos), preparándose además para adquisiciones futuras en turismo y salud y para reemplazar componentes tecnológicos en producción en cualquier momento, habilitando tan rápido como sea posible una base de datos unificada, ventas cruzadas, e información unificada de proveedores, contratos, ingresos y egresos.

Adicionalmente, debe soportar cambios frecuentes en los algoritmos que interpretan los datos de las diferentes empresas, en la presentación de informes para los directivos de XYZ, y en los datos y lógica de negocio de las aplicaciones de la holding cada vez que las empresas modifiquen sus sistemas. En síntesis, debe resolver simultáneamente un problema de **integrabilidad** y uno de **modificabilidad**.

### 3.2 Alcance
**Dentro del alcance**
- Diseñar una arquitectura de software para la holding XYZ que satisfaga los atributos de calidad de **modificabilidad** e **integración/interoperabilidad**.
- Validar la capacidad de modificación e integración de la arquitectura propuesta mediante una **revisión por arquitectos externos al equipo de desarrollo** (evaluación arquitectónica), y no mediante validación a nivel de código.

**Fuera del alcance**
- Los demás atributos de calidad (latencia, escalabilidad, seguridad, disponibilidad, entre otros).
- La construcción de una demo o implementación de código que valide la arquitectura; la validación es exclusivamente por evaluación de arquitectos externos.

### 3.3 Propósito del diseño
Definir una arquitectura de software para la holding XYZ que satisfaga los ASRs asociados a los atributos de calidad de **modificabilidad** e **integración/interoperabilidad**, definidos como el alcance de este reto, de manera que la solución propuesta sea validada mediante una evaluación realizada por arquitectos externos al equipo de desarrollo.

### 3.4 Stakeholders (extraídos del enunciado)
| Stakeholder | Rol | Descripción |
|---|---|---|
| Holding XYZ | Organización solicitante | Agrupa las empresas y encarga/requiere la arquitectura. |
| Directivos de XYZ | Consumidor de información / tomador de decisiones | Reciben los informes y requieren cambios frecuentes en su presentación; buscan una visión integral de todos los negocios y maximizar ingresos mediante venta cruzada. |
| EPS | Empresa de la holding / fuente de datos | Sistemas empresariales propios que deben integrarse. |
| IPS (3 clínicas y 2 hospitales) | Empresa de la holding / fuente de datos | Sistemas empresariales propios que deben integrarse. |
| Aseguradora de salud voluntaria (medicina prepagada) | Empresa de la holding / fuente de datos | Sistemas empresariales propios que deben integrarse. |
| Empresa de seguros de vida | Empresa de la holding / fuente de datos | Sistemas empresariales propios que deben integrarse. |
| Clientes | Beneficiario / sujeto de la información | Clientes de todas las empresas; se gestiona información de productos adquiridos y se busca habilitar ventas cruzadas. |
| Proveedores | Tercero relacionado | Se requiere tener su información y contratos de forma unificada. |
| Futuras empresas adquiridas (turismo y salud) | Futura empresa de la holding / fuente de datos | Aportarán sistemas en producción de diferentes proveedores tecnológicos que deberán integrarse. |

### 3.5 Estructura Épicas / Features / HUs (cada HU = un ASR)
**Épica 1: Modificabilidad**
> Como holding XYZ, queremos que la arquitectura permita modificar fácilmente los algoritmos de interpretación de datos y la presentación de informes, para reducir el tiempo y costo de los cambios frecuentes que hoy afectan al sistema de análisis.
- **Feature 1: Cambios aislados en el sistema de análisis de la holding.** Permite que los cambios frecuentes en los algoritmos que interpretan los datos de cada empresa y en la presentación de los informes para los directivos de XYZ se realicen de forma aislada, modificando solo el componente afectado, reduciendo tiempo y costo de los cambios.
  - HU-MOD-1 (ASR-MOD-1): modificar el algoritmo de transformación de datos de una empresa fuente.
  - HU-MOD-2 (ASR-MOD-2): modificar la presentación de informes para directivos.

**Épica 2: Integración / Interoperabilidad**
> Como holding XYZ, queremos que la arquitectura permita integrar los datos de los sistemas empresariales heterogéneos de las empresas de salud en un modelo unificado, para tener una visión integral del negocio (clientes, facturación y pagos).
- **Feature 2: Integración y consolidación de los datos de las empresas en un modelo unificado.** Permite recibir e interpretar la información que envían las empresas de la holding desde sus sistemas heterogéneos y consolidarla en un modelo de datos unificado, sin perder información y reconociendo a una misma persona reportada por distintas empresas.
  - HU-INT-1 (ASR-INT-1): recibir e interpretar la facturación y los pagos de una empresa de salud.
  - HU-INT-2 (ASR-INT-2): consolidar un afiliado/paciente presente en varias empresas de salud.

## 4. ASRs

Se definieron 4 ASRs (2 de modificabilidad, 2 de integración/interoperabilidad).

### ASR-MOD-1 — Cambio en el algoritmo de transformación de datos de una empresa fuente (CONFIRMADO)
| Elemento | Descripción |
|---|---|
| Fuente del estímulo | Holding XYZ. |
| Estímulo | Una empresa de la holding cambia su sistema de información (nuevos campos o lógica) y se solicita modificar el algoritmo que transforma sus datos. |
| Patrón del estímulo | Ocurre de manera frecuente, como parte de la operación normal de la holding. |
| Artefacto | Transformación de los datos entrantes de cada empresa fuente. |
| Entorno | Desarrollo. |
| Respuesta | Se modifican el código fuente, las bases de datos (incluidos sus DDL) y los demás elementos de software necesarios para transformar los nuevos campos y lógica de la empresa afectada, sin modificar los de las demás empresas ni los de la reportería. |
| Medida de respuesta | Cambio completado en máximo 60 horas hombre, sin afectar a las demás empresas ni la reportería. |

Nota: las 60 horas hombre son una meta que se impone a la arquitectura (reto de diseño), no una línea base medida; se verificará más adelante. (Versión reescrita por el usuario; reemplaza la anterior.)

### ASR-MOD-2 — Cambio en la presentación de informes, dashboards y reportes para directivos (CONFIRMADO)
| Elemento | Descripción |
|---|---|
| Fuente del estímulo | Directivos de XYZ. |
| Estímulo | Solicitud de modificar la presentación (formato o contenido) de un informe, dashboard o reporte dirigido a los directivos de XYZ. |
| Patrón del estímulo | Ocurre de manera frecuente, como parte de la operación normal de la holding. |
| Artefacto | Generación y presentación de informes, dashboards y reportes del sistema. |
| Entorno | Desarrollo. |
| Respuesta | Se modifica el código fuente o el proceso que genera el informe, dashboard o reporte, para incluir nuevos campos y fuentes o para soportar cambios de formato en los datos existentes que llegan de las empresas de la holding. |
| Medida de respuesta | Cambio completado en máximo 40 horas hombre, sin modificar la transformación de datos ni la integración. |

Nota: las 40 horas hombre son una meta de diseño para la arquitectura, no una línea base medida. Criterio acordado: los ASRs no deben mencionar "componentes", porque la arquitectura aún no existe (ya aplicado en los 4 ASRs).

### ASR-INT-1 — Recepción e interpretación de la facturación y los pagos de una empresa de salud (CONFIRMADO)
| Elemento | Descripción |
|---|---|
| Fuente del estímulo | Empresas de la holding. |
| Estímulo | Envía a la holding la información de los servicios de salud facturados (ventas e ingresos) y de los pagos a proveedores (egresos) de un periodo, desde su sistema empresarial. |
| Patrón del estímulo | Ocurre de forma periódica y de manera independiente para cada empresa de la holding. |
| Artefacto | Recepción e integración de datos de la holding. |
| Entorno | Operación normal. |
| Respuesta | Los datos se interpretan y quedan disponibles en el modelo de datos unificado de la holding; si un registro no valida, se aparta para su revisión sin perderse y sin detener el procesamiento de los demás. |
| Medida de respuesta | 100 % de los registros válidos interpretados correctamente y 100 % de los registros inválidos apartados, sin pérdida de datos. |

### ASR-INT-2 — Consolidación de un afiliado/paciente presente en varias empresas de salud de la holding (CONFIRMADO)
| Elemento | Descripción |
|---|---|
| Fuente del estímulo | Empresas de la holding. |
| Estímulo | Envía a la holding la información de un paciente (documento de identidad) y de los servicios de salud que ha adquirido, cuando esa persona ya fue reportada por la EPS como afiliada, con el documento de identidad en un formato distinto. |
| Patrón del estímulo | Ocurre de forma periódica y de manera independiente para cada empresa de la holding. |
| Artefacto | Recepción e integración de datos de la holding. |
| Entorno | Operación normal. |
| Respuesta | Se reconoce que se trata de la misma persona y se consolida en un único registro de cliente del modelo de datos unificado, que reúne su afiliación y los servicios adquiridos en ambas empresas. |
| Medida de respuesta | Cero clientes duplicados en el modelo unificado y 100 % de los servicios adquiridos asociados a la persona correcta. |

Nota: se descartó el ASR de "integrar un nuevo sistema de una empresa adquirida" (por ahora) y el de "reemplazo de componentes en producción" (parecía modificabilidad; podría retomarse como ASR de modificabilidad). Los ASRs de interoperabilidad quedaron renumerados (antes INT-2 e INT-3).

## 5. Notas y decisiones
- Interpretación acordada: el atributo de calidad se organiza en 2 épicas, 1 feature por épica, y cada ASR se modela como una HU.
- Los 4 ASRs están cerrados. Pendiente: tácticas de arquitectura, diseño (vistas) y plan de validación por juicio de experto.

## 6. Plan de validación por juicio de experto (BORRADOR para retomar)

**Idea:** reunión con un grupo de expertos (3 a 5 arquitectos externos, con perfil de arquitectura, integración y dominio salud) donde se expone la arquitectura, seguida de una encuesta individual (antes de cualquier discusión grupal) sobre qué tan modificable e interoperable es.

**Estructura de la encuesta:** 8 preguntas cerradas (escala 1 a 5; 1 = totalmente en desacuerdo, 5 = totalmente de acuerdo), 2 por ASR (una sobre la respuesta, otra sobre la medida), cada una con campo opcional de justificación, más 2 preguntas abiertas.

**Criterio de aceptación (umbrales ajustables):** promedio de las 2 preguntas de cada ASR >= 4,0 y ninguna calificación individual < 3. Se define antes de la reunión.

**Preguntas de ejemplo (borrador; dependen de que los ASRs queden finales):**
- ASR-MOD-1: (1) La arquitectura permite modificar el algoritmo de transformación de datos de una empresa sin afectar los componentes de las demás empresas ni los de reportería. (2) Es razonable esperar que ese cambio se haga modificando un solo componente, con esfuerzo de implementación y pruebas bajo.
- ASR-MOD-2: (3) La arquitectura permite modificar la presentación de un informe sin tocar los componentes de transformación de datos ni la capa de integración. (4) Es razonable esperar que ese cambio se haga modificando un solo componente, con esfuerzo bajo.
- ASR-INT-1: (5) La arquitectura permite recibir e interpretar correctamente la facturación y los pagos enviados por una empresa, dejándolos en el modelo de datos unificado. (6) El mecanismo para apartar registros inválidos, sin perderlos y sin detener el procesamiento de los demás, es adecuado.
- ASR-INT-2: (7) La arquitectura permite reconocer a una misma persona reportada por distintas empresas y consolidarla en un único registro con sus servicios adquiridos. (8) Es razonable esperar que la consolidación no genere clientes duplicados y asocie correctamente los servicios adquiridos a cada persona.
- Abiertas: (A) ¿Cuáles considera que son los principales riesgos o puntos débiles de la arquitectura en modificabilidad e interoperabilidad? (B) ¿Qué cambiaría o mejoraría de la arquitectura?

**Pendiente:** redactar el plan completo (participantes, agenda de la reunión, material previo).

## 7. Tácticas de arquitectura

### 7.1 Modificabilidad (ASR-MOD-1 y ASR-MOD-2) — acordadas (por ahora)
Fuente de clasificación: documentación del curso; según ella, la parametrización hace parte de la táctica de **diferir (aplazar) el enlace / la selección tecnológica**. Se trata así por ahora.

| Categoría | Táctica | Idea de uso |
|---|---|---|
| Incrementar la cohesión | Dividir módulos / aumentar coherencia semántica | Separar en módulos distintas funcionalidades no relacionadas entre sí (p. ej. transformación de datos de cada empresa vs. generación de reportes). |
| Reducir el acoplamiento | Abstraer servicios comunes | Extraer servicios comunes (p. ej. acceso a datos) para que varias partes no dependan unas de otras. |
| Reducir el acoplamiento | Usar intermediarios (cola o bus) | Comunicar dos partes a través de una cola o bus para que un cambio no se propague. |
| Diferir el enlace | Reemplazo de componentes | Depender de interfaces que pueden ser implementadas por distintos componentes concretos. |
| Diferir el enlace | Parametrización | Parametrizar para adaptarse a nuevas modificaciones solo cambiando parámetros (p. ej. reportes). Aplicada también (como complemento) a transformación, orquestación y consolidación; ver 9.7. |

**Cómo se relacionan con los ASRs (análisis, sin cerrar):**
- MOD-1: coherencia semántica (módulo por empresa), intermediario (aislar propagación), reemplazo de componentes (interfaz común por empresa). La parametrización sola no cubre "nuevos campos y lógica" (la lógica nueva suele requerir código), por lo que se combina con separación de módulos e interfaces.
- MOD-2: parametrización (táctica principal) y abstraer servicios comunes (acceso a datos único para informes y dashboards).
- Sugerencia pendiente de decidir: considerar también **encapsular** con una interfaz estable, habilitador del reemplazo de componentes.

### 7.2 Interoperabilidad (ASR-INT-1 y ASR-INT-2) — acordadas (por ahora)
Se usan solo tácticas de la documentación oficial del curso: **orquestar** más las de modificabilidad que también favorecen la interoperabilidad. No se incluyen otras tácticas fuera de esa documentación.

**Orquestar** = mecanismo de control que coordina y secuencia la ejecución de varios servicios.
- INT-1 (facturación y pagos): un orquestador coordina el flujo de cada envío: recibir los datos, validarlos, transformarlos, cargarlos en el modelo unificado y apartar los registros inválidos sin detener el resto.
- INT-2 (consolidación del paciente): un orquestador coordina la consolidación: recibir los datos del paciente, buscarlo entre los afiliados ya registrados, consolidar el registro único y asociarle los servicios adquiridos.

| ASR | Orquestar | Compartidas con modificabilidad |
|---|---|---|
| INT-1 | Coordina recepción, validación, carga y apartado de inválidos. | Intermediario (cola o bus), para que las empresas envíen sin depender del procesamiento; dividir módulos, uno por empresa; reemplazo de componentes mediante interfaces. |
| INT-2 | Coordina la búsqueda y consolidación de la persona. | Abstraer servicios comunes (p. ej. acceso a los datos de personas); reemplazo de componentes mediante interfaces. |

## 8. Contexto para retomar en un nuevo chat

**Siguiente paso:** diseñar los diagramas/vistas de arquitectura. Probablemente no baste el de componentes; las vistas necesarias (p. ej. despliegue, otras) se discutirán primero con el usuario.

**Forma de trabajo acordada con el usuario:**
- Ir paso por paso; mostrar el borrador en el chat y guardar en este archivo solo cuando el usuario confirme ("listo", "me gusta", "guárdalo").
- No inventar: extraer del texto del enunciado y del criterio del usuario. Si falta un dato (p. ej. cifras), preguntar.
- Los ASRs no mencionan "componentes" (aún no hay arquitectura); en el diseño sí se podrán nombrar.
- Usar solo tácticas de la documentación oficial del curso (las de la sección 7).
- Las metas en horas hombre de MOD-1 (60) y MOD-2 (40) son retos de diseño para la arquitectura, no líneas base medidas.
- Validación final: juicio de expertos externos (sección 6); sin demo de código.
- Los diagramas deben mostrar cómo la arquitectura satisface cada ASR y sus tácticas.
- El archivo `Arquitectura-Reto3.drawio` también se edita a mano (el usuario ajusta líneas y cajas): no regenerar pestañas completas sin avisar; editar solo los elementos afectados y leer el archivo actual antes de cambiar nada.

**Pendiente además:** plan completo de validación por juicio de experto (participantes, agenda, material previo); decidir si se agrega la táctica "encapsular" (sugerida, sin decidir).

## 9. Vistas de arquitectura y flujo de datos

### 9.1 Vistas acordadas (puntos de vista de Rozanski y Woods)
- **Funcional:** diagrama de componentes (tácticas: módulos por empresa, intermediario, orquestador, interfaces) y diagramas de secuencia de los flujos de INT-1 e INT-2 (incluido el apartado de registros inválidos).
- **Información:** modelo conceptual/lógico de datos y persistencia (modelo unificado con identificación única de la persona, para INT-2).
- **Desarrollo:** estructura de módulos y dependencias (muestra el aislamiento de MOD-1 y MOD-2). Referencia: Rozanski y Woods, *Software Systems Architecture* (2ª ed., 2011), punto de vista de desarrollo; revisar también cómo lo nombra la documentación del curso.
- **Fuera:** concurrencia, despliegue y operacional (ligadas a atributos fuera del alcance); despliegue solo si un experto lo pide.

### 9.2 Idea de estructura de la arquitectura (propuesta del usuario, en discusión)
Como la información viene de fuentes distintas, debe haber un proceso de integración de datos:
1. **Flujo de entrada** que lee la información en crudo de cada empresa y la almacena en un **repositorio inicial** (tipo data lake).
2. **Proceso de transformación** que normaliza los datos a un mismo lenguaje.
3. **Segundo repositorio estructurado** (tipo data warehouse) con el modelo unificado, consumido por los demás módulos para reportes, generación de código, automatizaciones, etc.

Opinión inicial de Claude: encaja con los ASRs. El repositorio crudo desacopla la recepción de la transformación y permite reprocesar cuando cambia el algoritmo (MOD-1) y no perder datos (INT-1); la transformación es donde ocurre la consolidación de personas (INT-2); los reportes leen solo del repositorio estructurado, aislados de los cambios en las fuentes (MOD-2). Riesgos a tratar: el modelo unificado debe ser estable y expuesto mediante interfaces, porque un cambio en él afecta a todos los consumidores; y conviene hablar de "zona cruda" y "zona estructurada" sin fijar tecnología todavía.

**Decisión resuelta (ver 9.3):** el repositorio crudo (`zona_cruda`) solo almacena; la validación va en `transformacion_datos`.

### 9.3 Vista funcional — diagrama de componentes (CONFIRMADO)
Archivo de trabajo: `Arquitectura-Reto3.drawio` (una pestaña por vista: 1 componentes, 2 secuencia INT-1, 3 secuencia INT-2, 4 información, 5 desarrollo). Solo la pestaña 1 está dibujada.

**Reglas de estilo:** notación UML 2.0 (componentes con estereotipos «component»/«datastore», interfaces como círculo, dependencias «use» punteadas, zonas como paquetes). Nombres de componentes `nombre_componente` (minúsculas, guion bajo, sin tildes); interfaces `i_nombre`. Sin ASR ni texto explicativo dentro de los componentes y sin leyenda; solo definiciones puntuales (p. ej. "cola o bus") y las tácticas entre « ». Las flechas punteadas NO indican asincronía; la sincronía se define en los diagramas de secuencia.

**Componentes:** `recepcion`, `intermediario` (cola o bus), `zona_cruda`, `orquestador_integracion` (contiene `orquestacion_facturacion_pagos` y `orquestacion_consolidacion_personas`), `transformacion_datos` (un módulo por empresa: `modulo_eps`, `modulo_ips`, `modulo_medicina_prepagada`, `modulo_seguros_vida`, `modulo_futuras_empresas`; interfaz `i_transformacion`), `bd_rechazados`, `consolidacion_personas`, `acceso_modelo_unificado` (interfaz `i_modelo_unificado`), `zona_estructurada`, `reporteria`; actor: directivos de XYZ.

**Decisiones:**
- `zona_cruda` solo almacena el original (más empresa de origen y periodo); no valida y no se expande en este diagrama (su detalle va en la vista de información, igual que `zona_estructurada` y `bd_rechazados`).
- `bd_rechazados` es aparte de `zona_cruda`.
- Un solo orquestador con dos orquestaciones.
- Opción A de escritura: `transformacion_datos` no escribe en ninguna base; recibe un registro y devuelve transformado o inválido. La orquestación de facturación y pagos carga los válidos y envía los inválidos a `bd_rechazados`. La `zona_estructurada` la escriben solo esa orquestación y `consolidacion_personas`, siempre vía `acceso_modelo_unificado`. `reporteria` solo lee.
- Formato: hoy CSV, abierto a nuevos formatos (módulos nuevos en `transformacion_datos`).
- Sincronía: la carga (`recepcion` → `intermediario` → `zona_cruda`) sería asíncrona por el intermediario (por confirmar en los diagramas de secuencia); las llamadas del orquestador serían solicitud-respuesta.
- Riesgo anotado: un campo nuevo (MOD-1) implica cambiar el modelo unificado (DDL); por eso `i_modelo_unificado` debe ser estable.

**Pendientes a resolver en las secuencias:** quién dispara el orquestador y cuándo lee de `zona_cruda`; si `transformacion_datos` interviene en INT-2; sincronía de cada mensaje.

### 9.4 Vista funcional — secuencia INT-1 (CONFIRMADO, pestaña 2 del .drawio)
Marco `sd facturacion_pagos`; líneas de vida: `empresa`, `recepcion`, `intermediario`, `zona_cruda`, `orquestacion_facturacion_pagos`, `transformacion_datos`, `bd_rechazados`, `acceso_modelo_unificado`, `zona_estructurada`. Mensajes con formato `verbo_sustantivo`.
- **Fase 1 (envío, asíncrona):** `enviar_facturacion_pagos` → `publicar_envio` → `guardar_original(empresa, periodo)`.
- **Fase 2 (procesamiento periódico):** lo dispara un **evento programado** (mensaje encontrado hacia la orquestación; periodicidad no definida). La orquestación hace `leer_envio` (sync) y, en un `loop` por registro, llama `validar_registro` (devuelve `resultado_validacion`). `alt`: válido → `transformar_registro` (devuelve `registro_transformado`), `cargar_registro` a `acceso_modelo_unificado` → `escribir_registro` en `zona_estructurada`; inválido → `guardar_rechazado` en `bd_rechazados`, sin detener el resto.
- **Decisiones:** validar y transformar son dos llamadas separadas a `transformacion_datos`; las llamadas de la fase 2 son síncronas (solicitud-respuesta).

### 9.5 Vista funcional — secuencia INT-2 (CONFIRMADO, pestaña 3 del .drawio)
Marco `sd consolidacion_personas`; líneas de vida: `zona_cruda`, `orquestacion_consolidacion_personas`, `transformacion_datos`, `consolidacion_personas`, `acceso_modelo_unificado`, `zona_estructurada`, `bd_rechazados`. Arranca en el procesamiento (el envío ya está en `zona_cruda`; la fase 1 es la misma de INT-1 y no se dibuja ni se referencia).
- Un **evento programado** inicia la orquestación; `leer_envio` → `registros_del_envio`.
- `loop` por paciente: `validar_registro` → `resultado_validacion`. `alt`: **válido** → `transformar_registro` (normaliza el documento de identidad al formato unificado) → `consolidar_persona` (a `consolidacion_personas`) → `buscar_persona(documento)` a `acceso_modelo_unificado` → `persona_encontrada`; `alt` anidado: **persona ya registrada** → `asociar_servicios` + `escribir_persona`; **persona nueva** → `crear_persona` + `escribir_persona` (a `zona_estructurada`); responde `persona_consolidada`. **Inválido** → `guardar_rechazado` en `bd_rechazados`, sin detener el resto.
- **Decisiones:** `transformacion_datos` sí interviene en INT-2 (normaliza el documento antes de consolidar); los inválidos de INT-2 también van a `bd_rechazados`; llamadas síncronas. No cubierto (no definido): caso de varias coincidencias posibles en la búsqueda.
- **Pendientes de 9.3 resueltos:** el orquestador lo dispara un evento programado; `transformacion_datos` interviene en INT-2; sincronía definida en las secuencias (carga asíncrona por el intermediario, resto síncrono).

### 9.6 Vista de información (CONFIRMADO, pestañas 4a y 4b del .drawio)
Diagramas de clases UML 2.0 (modelo lógico). "Zona curada" = `zona_estructurada`. Los campos marcados como ejemplo son inventados mientras no se conozcan los sistemas reales.
- **4a. Zona cruda (modelo entidad-relación simple, sin esquemas):** la zona cruda guarda archivos (posible almacenamiento de objetos tipo S3, sin fijar tecnología) y solo sus metadatos; el contenido lo interpreta `transformacion_datos`. Paquete «datastore» `zona_cruda`: `entidad` (`id_entidad`, `tipo_entidad`, `nombre_entidad`; enumeración `tipo_entidad`: eps, ips, medicina_prepagada, seguros_vida; ej. de nombres: clinica_1, hospital_1, aseguradora_x) 1 — 0..* `archivo_recibido` (`id_archivo`, `nombre_archivo`, `ruta_almacenamiento`, `id_entidad`, `periodo`, `fecha_ingesta`, `formato`, `tamano_archivo`, `cantidad_registros`; enumeración `formato_archivo`: csv (actual), txt, excel, parquet, formato_aplicacion). Sin campo `estado`. Paquete «datastore» `bd_rechazados`: `registro_rechazado` (`id_rechazo`, `id_archivo`, `numero_registro`, `campo_con_error`, `motivo_rechazo`, `registro_original`) con referencia por `id_archivo` a `archivo_recibido`.
- **4b. Zona estructurada (opción B):** esquema `modelo_unificado` con `entidad` (`id_entidad`, `tipo_entidad`, `nombre_entidad`), `cliente` (`id_cliente`, `documento_identidad`), `afiliacion`, `producto_adquirido`, `venta`, `ingreso`, `egreso`, `proveedor`, `contrato`, más un esquema por tipo de entidad (`esquema_eps`, `esquema_ips`, `esquema_medicina_prepagada`, `esquema_seguros_vida`) con `datos_propios_<tipo>` para campos específicos, de modo que un campo nuevo de una entidad entra en su esquema sin modificar el modelo unificado. Asociaciones y atributos del modelo unificado son propuesta de Claude aceptada por el usuario ("por ahora").

### 9.7 Vista de módulos (CONFIRMADO, pestaña 5 del .drawio)
Nombre acordado: "vista de módulos" (corresponde a la vista de desarrollo de la sección 9.1). Diagrama de paquetes UML 2.0; alcance: solo módulos y dependencias (sin organización de código, construcción ni estándares; se puede ampliar si un experto lo pide).
- **Módulos:** `orquestador_integracion` (`orquestacion_facturacion_pagos`, `orquestacion_consolidacion_personas`, `parametros_orquestacion`); `consolidacion_personas` (`busqueda_persona`, `consolidacion_registro_unico`, `asociacion_servicios`, los tres pasos de la sección 7.2, y `parametros_consolidacion`); `transformacion_datos` (subpaquete `modulos_por_entidad` con `modulo_eps`, `modulo_ips`, `modulo_medicina_prepagada`, `modulo_seguros_vida`, `modulo_futuras_empresas`, y `parametros_transformacion` transversal); `reporteria` (`presentacion`, `preparacion_logica_informes`, `consulta_datos`, `parametros_informes`); `acceso_modelo_unificado`; `contratos` (solo las interfaces `i_transformacion` e `i_modelo_unificado`, sin tipos de datos).
- **Dependencias:** el orquestador usa `consolidacion_personas` y `contratos`; `consolidacion_personas` usa `contratos`; `transformacion_datos` y `acceso_modelo_unificado` realizan (`«realize»`) las interfaces de `contratos`; `consulta_datos` usa `contratos` (`i_modelo_unificado`); dentro de `reporteria`: `presentacion` → `preparacion_logica_informes` → `consulta_datos`, y `preparacion_logica_informes` → `parametros_informes`; las orquestaciones usan `parametros_orquestacion`, los tres submódulos de consolidación usan `parametros_consolidacion` y `modulos_por_entidad` (como grupo) usa `parametros_transformacion`. Los módulos por entidad no dependen entre sí (MOD-1); `reporteria` no depende de la transformación ni del orquestador (MOD-2); nadie toca `acceso_modelo_unificado` salvo por su interfaz.
- **Decisiones:** `recepcion` e `intermediario` y los almacenes (`zona_cruda`, `bd_rechazados`) no se dibujan en esta vista (el intermediario es tecnología; están en la vista funcional). La táctica de parametrización se materializa en un módulo de parámetros por componente (opción B: no un único módulo compartido, para no acoplar a todas las entidades; `parametros_transformacion` es transversal a los módulos por entidad pero guarda los valores de cada entidad por separado), todos de color turquesa y con la etiqueta «Parametrización»: `parametros_informes` (MOD-2), `parametros_transformacion` (MOD-1, como complemento: la lógica nueva sigue requiriendo código), `parametros_orquestacion` y `parametros_consolidacion`. Los parámetros viven dentro de su componente, sin almacén nuevo. No se dibuja su contenido (ejemplos conversados, sin decidir: mapeo de campos y lectura del archivo, periodicidad del evento programado, criterios de coincidencia de personas). La "lógica de preparación" de `preparacion_logica_informes` es solo para armar el informe, distinta de `transformacion_datos`. Layout ampliado para que los nombres quepan en sus cajas.
