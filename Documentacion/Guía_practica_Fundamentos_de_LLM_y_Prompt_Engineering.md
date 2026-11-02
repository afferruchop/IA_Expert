# Guía práctica: Fundamentos de LLM y Prompt Engineering

_Ruta de aprendizaje en 3 bloques: entender cómo funciona un modelo, dominar los 5 elementos del prompt y comprobarlo con 10 experimentos comparando un prompt vago contra uno preciso._

## Cómo usar esta guía

| Bloque                        | Qué haces                                                                        | Tiempo estimado |
| ----------------------------- | -------------------------------------------------------------------------------- | --------------- |
| 1. Fundamentos                | Lees los conceptos y haces los módulos gratuitos                                 | 2 a 3 h         |
| 2. Los 5 elementos del prompt | Estudias cada elemento aplicado a un email, un resumen y una extracción de datos | 1 h             |
| 3. Laboratorio de 10 prompts  | Ejecutas cada par (vago vs. preciso) y comparas respuestas                       | 2 h             |

Regla de oro: **no leas los prompts, ejecútalos**. El aprendizaje real ocurre al ver la diferencia entre dos respuestas.

---

## Bloque 1. Fundamentos (nivel conceptual)

### 1.1 LLM (Large Language Model)

**Qué es:** un modelo entrenado con enormes cantidades de texto para una tarea muy concreta: **predecir cuál es la siguiente pieza de texto más probable** dado lo que ya está escrito. De esa tarea simple surge la capacidad de redactar, resumir, traducir, clasificar o razonar.

**Analogía:** el autocompletado de tu celular, pero entrenado con una porción gigantesca de la escritura humana y con miles de millones de parámetros (los "diales" internos que se ajustaron durante el entrenamiento).

**Lo que implica en la práctica:**

- No "busca" la respuesta en una base de datos: la **genera** palabra por palabra. Por eso puede sonar seguro y equivocarse (alucinación).
- Responde a **patrones**. Si tu prompt es ambiguo, completa con el patrón más común, que suele ser genérico.
- Cuanto más claro es tu pedido, más se reduce el espacio de respuestas posibles.

**Por qué ocurre la alucinación:** no es un fallo separado, sino la consecuencia directa de predecir el token más plausible. Si el contexto no contiene la respuesta, el modelo rellena con lo estadísticamente verosímil, no con lo verdadero. No tiene un verificador de hechos interno. Por eso el contexto que tú aportas es el ancla de verdad más confiable que tienes, y por eso conviene pedirle "si no está en el texto, responde que no lo sabes" y verificar cifras, nombres y fuentes.

### 1.2 Tokens

**Qué son:** las unidades en que el modelo divide el texto. No son exactamente palabras: pueden ser una palabra completa, un fragmento ("inteli" + "gencia"), un signo de puntuación o un espacio.

**Ejemplo ilustrativo** (la división exacta varía según el modelo): la frase "Hola, ¿cómo estás?" puede convertirse en 6 a 8 tokens. En español suele haber más tokens por palabra que en inglés.

**Por qué te importa:**

- Los modelos **leen y escriben tokens**, y los límites y costos se miden en tokens.
- Un prompt larguísimo gasta capacidad que podría usarse en la respuesta.
- Pequeños cambios de redacción (mayúsculas, ortografía) cambian los tokens y a veces el resultado.

### 1.3 Contexto (ventana de contexto)

**Qué es:** la cantidad máxima de tokens que el modelo puede "tener a la vista" a la vez: tu prompt, el historial de la conversación, los documentos que pegaste y la respuesta que está escribiendo.

**Analogía:** el escritorio de trabajo. Todo lo que está sobre el escritorio se puede consultar; lo que no cabe, no existe para el modelo.

**Lo que implica en la práctica:**

- El modelo **no tiene memoria entre chats** salvo que la plataforma la ofrezca; dentro de un chat solo "ve" lo que está en la ventana.
- En conversaciones muy largas, instrucciones del inicio pueden perder peso. Solución: repetir lo esencial o abrir un chat nuevo con un resumen.
- **El contexto que tú aportas es el elemento más poderoso del prompt**: audiencia, objetivo, datos, restricciones.

### 1.4 Transformer

**Qué es:** la arquitectura de red neuronal (presentada en 2017) en la que se basan los LLM modernos. Su idea central es el **mecanismo de atención (self-attention)**: al procesar cada token, el modelo calcula qué otros tokens de la entrada son más relevantes para interpretarlo.

**Ejemplo:** en "El banco cerró porque el río creció y _este_ inundó la calle", la atención ayuda a decidir a qué se refiere "este" (el río) según el resto de la frase.

**Lo que implica en la práctica:**

- Procesa todo el texto en paralelo, lo que permitió entrenar modelos enormes.
- Tu prompt entero influye en cada palabra generada: **el orden, las palabras clave y los ejemplos importan**.
- Por eso un buen ejemplo dentro del prompt (few-shot) cambia tanto el estilo y la estructura de la salida.

**Resumen de una línea:** el texto se divide en _tokens_, el _Transformer_ usa _atención_ para relacionarlos dentro del _contexto_ y el _LLM_ predice el siguiente token, una y otra vez, hasta formar la respuesta.

### 1.5 Temperatura

**Qué es:** el parámetro que controla qué tan "aleatoria" es la elección del siguiente token. Con `0` el modelo es casi determinista (siempre elige la opción más probable); con `0.7` hay equilibrio; con `1.5` o más es creativo pero errático. La escala exacta depende del proveedor (algunos van de 0 a 1 y otros de 0 a 2).

**Cómo usarla:**

- Extracción de datos, clasificación y código: cerca de `0`.
- Redacción de emails y resúmenes: baja o media.
- Lluvia de ideas y escritura creativa: súbela.
- Agentes que deben ser reproducibles: temperatura baja por defecto.

**Por qué te importa:** explica por qué el mismo prompt da respuestas distintas en dos ejecuciones. En los chats web normalmente no puedes cambiarla; sí en las APIs, los playgrounds y los modelos locales. En el laboratorio, los prompts 3, 4 y 10 (extracción, clasificación y código) son los más sensibles a este parámetro.

### 1.6 Modelo base vs. modelo instruido

**Modelo base:** solo predice el siguiente token. Si le pides "resume esto", puede continuar con más texto en vez de obedecer.

**Modelo instruido (instruct / chat):** fue afinado con ejemplos de instrucción y respuesta para seguir órdenes. Es el que usas en cualquier chat comercial.

**Regla práctica:** para todo lo que hagas con prompts y agentes, usa siempre la variante instruct. Cuando descargues un modelo local (por ejemplo `llama3.2` en Ollama), el modelo por defecto suele ser instruct; verifícalo en la ficha del modelo antes de usarlo.

### 1.7 Cursos gratuitos recomendados

**Google, Machine Learning Crash Course, módulo de Large Language Models** (va desde tokens hasta Transformers):

- https://developers.google.com/machine-learning/crash-course/llm
- Curso completo: https://developers.google.com/machine-learning/crash-course
- Nota: Google recomienda haber visto antes los módulos de redes neuronales y embeddings; el módulo de LLM es conceptual pero se apoya en ellos.

**Microsoft Learn, Fundamentals of Generative AI** (ruta gratuita, conceptos de IA generativa y LLM):

- https://learn.microsoft.com/training/paths/introduction-generative-ai
- Serie en video: https://learn.microsoft.com/shows/generative-ai-for-beginners
- Repositorio del currículo (lecciones con ejemplos de prompts): https://github.com/microsoft/generative-ai-for-beginners

**Ruta sugerida:** primero la ruta de Microsoft Learn (más general), luego el módulo LLM de Google (más cercano a tokens y Transformer).

**Autoevaluación (responde sin mirar):**

1. ¿Por qué un LLM puede "inventar" un dato con total seguridad?
2. ¿Qué diferencia hay entre una palabra y un token?
3. ¿Qué ocurre con la información que no cabe en la ventana de contexto?
4. ¿Qué problema resuelve el mecanismo de atención?

### 1.8 Dónde ejecutar los experimentos

Para el laboratorio puedes usar cualquier chat gratuito (ChatGPT, Claude, Gemini) o un modelo local si ya lo tienes. Lo importante es usar **el mismo modelo para las dos versiones de cada par** (vago vs. preciso): si cambias de modelo entre una y otra, la comparación no vale. Abre un chat nuevo para cada versión y anota qué modelo usaste.

---

## Bloque 2. Los 5 elementos del prompt

Un prompt eficaz combina cinco elementos. No siempre necesitas todos, pero cuantos más incluyas con intención, más control tienes.

| #   | Elemento                | Pregunta que responde                                            | Ejemplo corto                                                                     |
| --- | ----------------------- | ---------------------------------------------------------------- | --------------------------------------------------------------------------------- |
| 1   | **Rol**                 | ¿Quién debe ser el modelo?                                       | "Actúa como ejecutivo de cobranza con 10 años de experiencia"                     |
| 2   | **Tarea**               | ¿Qué debe hacer, con un verbo claro?                             | "Redacta un email recordando un pago vencido"                                     |
| 3   | **Contexto**            | ¿Qué necesita saber (datos, audiencia, objetivo, restricciones)? | "Cliente desde 2021, factura de 1.200 USD vencida hace 15 días, relación cordial" |
| 4   | **Ejemplos (few-shot)** | ¿Cómo se ve una buena salida?                                    | Uno o dos modelos de entrada y salida                                             |
| 5   | **Formato de salida**   | ¿En qué estructura y longitud entrego?                           | "Asunto + 3 párrafos, máximo 120 palabras"                                        |

**Cómo los uso (de mayor a menor impacto):** contexto, formato, ejemplos, tarea, rol. El rol ayuda sobre todo con el tono y el nivel técnico; el contexto y el formato son los que más evitan respuestas genéricas.

### Caso A. Un email

**Prompt vago:**

> Escribe un email para cobrar una factura.

**Problema:** no hay destinatario, monto, tono ni longitud. El modelo rellena con lo más promedio: un texto genérico y frío.

**Prompt con los 5 elementos:**

> **Rol:** Eres un ejecutivo de cuentas por cobrar, cordial y profesional. **Tarea:** Redacta un email de recordatorio de pago. **Contexto:** Cliente: Laura Gómez, gerente de compras de Textiles Andina, cliente desde 2021. Factura F-2045 por 1.200 USD, venció hace 15 días. Es su primer retraso. Queremos mantener la relación y recibir el pago o una fecha de pago esta semana. **Ejemplo de tono deseado:** "Hola Carlos, te escribo para recordarte con cariño que la factura F-1980 está pendiente. Si ya la pagaste, ignora este mensaje y gracias." **Formato:** Asunto en una línea + saludo + 2 párrafos cortos + cierre. Máximo 120 palabras. Un único llamado a la acción al final.

**Qué observar:** personalización, tono, llamado a la acción claro, longitud controlada.

### Caso B. Un resumen

**Prompt vago:**

> Resume este texto.

**Problema:** no sabes para quién, para qué ni qué extensión. El modelo elige por ti.

**Prompt con los 5 elementos:**

> **Rol:** Eres un analista que prepara briefings para un director que tiene 2 minutos. **Tarea:** Resume el acta de reunión que pego abajo. **Contexto:** El lector no asistió a la reunión. Necesita saber qué se decidió, qué queda pendiente y quién es responsable. Ignora los comentarios laterales. **Ejemplo del formato esperado:** Decisiones: 1) Se aprueba el presupuesto de marketing de 5.000 USD. Pendientes: 1) Enviar contrato a legal (Ana, viernes). Riesgos: Retraso del proveedor. **Formato:** Tres secciones (Decisiones, Pendientes con responsable y fecha, Riesgos). Máximo 100 palabras en total. Si falta una fecha o responsable, escribe "sin definir".
>
> Acta: \[pegar texto\]

**Qué observar:** si el resumen es accionable, si respeta el límite y si marca lo que falta en vez de inventarlo.

### Caso C. Extracción de datos

**Prompt vago:**

> Saca los datos importantes de esta factura.

**Problema:** "importantes" es subjetivo y la salida no se puede usar en una hoja de cálculo ni en un programa.

**Prompt con los 5 elementos:**

> **Rol:** Eres un asistente de contabilidad que digita facturas con cero errores. **Tarea:** Extrae los campos solicitados del texto de la factura. **Contexto:** Las fechas deben ir en formato AAAA-MM-DD. Los montos son números sin símbolo de moneda. Si un campo no aparece, usa null. No infieras ni inventes datos. **Ejemplo:** Texto: "Factura N° 887 de Distribuciones Sol, 3 de marzo de 2025, total COP 450.000, NIT 900.123.456-7" Salida: {"numero": "887", "proveedor": "Distribuciones Sol", "fecha": "2025-03-03", "total": 450000, "moneda": "COP", "nit": "900.123.456-7"} **Formato:** Responde únicamente con un objeto JSON válido, con las claves: numero, proveedor, fecha, total, moneda, nit. Sin texto adicional.
>
> Texto de la factura: \[pegar texto\]

**Qué observar:** si el JSON es válido, si respeta los formatos y si usa null en vez de inventar.

### Lección clave del bloque

Un prompt vago deja **decisiones sin tomar**, y el modelo las toma por ti con el patrón más común. Un prompt preciso toma esas decisiones por ti de antemano.

---

## Bloque 3. Laboratorio: 10 prompts, versión vaga vs. versión precisa

**Cómo ejecutarlo:** abre un chat nuevo por cada versión (así el historial no contamina), pega el prompt, guarda la respuesta y compárala con la rúbrica del Bloque 4. En cada versión precisa señalo con etiquetas el **rol**, el **few-shot** y el **formato**. Donde veas \[corchetes\], reemplaza por tu propio material.

### Prompt 1. Email de cobranza

**Vaga:**

> Escribe un email para cobrar una factura.

**Precisa:**

> &#91;ROL\] Eres un ejecutivo de cuentas por cobrar cordial y profesional. Redacta un recordatorio de pago a Laura Gómez (Textiles Andina), factura F-2045 por 1.200 USD vencida hace 15 días, primer retraso, relación de 4 años. \[FEW-SHOT\] Ejemplo de tono: "Hola Carlos, te escribo para recordarte con cariño que la factura F-1980 está pendiente. Si ya la pagaste, ignora este mensaje." \[FORMATO\] Asunto + saludo + 2 párrafos + cierre. Máximo 120 palabras. Un solo llamado a la acción: confirmar fecha de pago esta semana.

**Qué observar:** datos concretos presentes, tono cálido, un único llamado a la acción.

### Prompt 2. Resumen de reunión

**Vaga:**

> Resume esta reunión: \[texto\]

**Precisa:**

> &#91;ROL\] Eres un asistente ejecutivo que prepara resúmenes para un director con 2 minutos disponibles. Resume la transcripción siguiente para alguien que no asistió. Ignora conversación social. \[FEW-SHOT\] Ejemplo de salida: Decisiones: Se aprueba lanzar la campaña el 15 de mayo. Pendientes: Diseño entrega piezas (Marta, 5 de mayo). Riesgos: Presupuesto sin confirmar. \[FORMATO\] Exactamente tres secciones (Decisiones, Pendientes, Riesgos). Máximo 100 palabras. Si falta responsable o fecha, escribe "sin definir". Transcripción: \[texto\]

**Qué observar:** estructura fija, responsables y fechas, honestidad ante datos faltantes.

### Prompt 3. Extracción de datos de una factura

**Vaga:**

> Saca los datos de esta factura: \[texto\]

**Precisa:**

> &#91;ROL\] Eres un asistente de contabilidad que digita datos sin cometer errores ni inventar. Extrae los campos de la factura. Fechas en AAAA-MM-DD, montos como número sin símbolo, campo ausente = null. \[FEW-SHOT\] Entrada: "Factura 887, Distribuciones Sol, 3 de marzo de 2025, total COP 450.000" → Salida: {"numero":"887","proveedor":"Distribuciones Sol","fecha":"2025-03-03","total":450000,"moneda":"COP"} \[FORMATO\] Solo un JSON válido con las claves numero, proveedor, fecha, total, moneda. Sin explicaciones. Factura: \[texto\]

**Qué observar:** si puedes pegar la salida directamente en un programa o una hoja de cálculo.

### Prompt 4. Clasificación de reseñas

**Vaga:**

> Dime qué opinan los clientes de estas reseñas: \[lista\]

**Precisa:**

> &#91;ROL\] Eres un analista de experiencia de cliente. Clasifica cada reseña por sentimiento (positivo, neutro, negativo) y tema principal (precio, calidad, envío, atención, otro). \[FEW-SHOT\] Reseña: "Llegó tarde pero el producto es excelente" → neutro | envío Reseña: "Carísimo para lo que ofrece" → negativo | precio \[FORMATO\] Tabla con columnas: N°, Reseña (máx. 8 palabras), Sentimiento, Tema. Al final, una línea con el tema más mencionado. Reseñas: \[lista\]

**Qué observar:** consistencia en las etiquetas (el few-shot define el criterio de los casos mixtos).

### Prompt 5. Plan de estudio

**Vaga:**

> Hazme un plan para aprender inteligencia artificial.

**Precisa:**

> &#91;ROL\] Eres un tutor de IA que diseña rutas para principiantes que trabajan tiempo completo. Diseña un plan de 4 semanas para aprender los fundamentos de LLM y prompting. Tengo 5 horas por semana, no sé programar y mi meta es usar IA en mi trabajo administrativo. \[FEW-SHOT\] Formato de una semana: "Semana 1 | Objetivo: entender qué es un LLM | Tareas: (1) módulo de Microsoft Learn, 90 min; (2) probar 3 prompts, 60 min | Entregable: resumen de 5 líneas". \[FORMATO\] Una tabla por semana con Objetivo, Tareas con minutos y Entregable. Cierra con 3 indicadores para saber si avanzo.

**Qué observar:** realismo del tiempo, adaptación a tu nivel, entregables medibles.

### Prompt 6. Reescritura de tono

**Vaga:**

> Mejora este mensaje: "Necesito el informe ya, siempre lo entregas tarde."

**Precisa:**

> &#91;ROL\] Eres un coach de comunicación para líderes de equipo. Reescribe el mensaje para un colega con quien tengo buena relación. Debe ser firme, sin acusaciones y proponer una solución. Conserva la fecha límite de mañana a las 9 a. m. \[FEW-SHOT\] Antes: "Tu código siempre tiene errores." Después: "Encontré tres errores en la última entrega; ¿revisamos juntos antes del próximo envío?" \[FORMATO\] Entrega 3 versiones (directa, cálida, formal) de máximo 40 palabras cada una, con una frase indicando cuándo usar cada una. Mensaje: "Necesito el informe ya, siempre lo entregas tarde."

**Qué observar:** desaparece la generalización ("siempre") y aparece una acción concreta.

### Prompt 7. Publicación para LinkedIn

**Vaga:**

> Escribe un post sobre inteligencia artificial.

**Precisa:**

> &#91;ROL\] Eres un redactor de contenido para profesionales de recursos humanos. Escribe un post sobre 3 errores al usar IA para filtrar hojas de vida. Audiencia: analistas de selección sin perfil técnico. Tono cercano, sin tecnicismos. \[FEW-SHOT\] Ejemplo de gancho: "El año pasado descarté 40 candidatos buenos por confiar ciegamente en un filtro automático." \[FORMATO\] Gancho de una línea, 3 errores numerados (máximo 2 líneas cada uno), cierre con una pregunta para generar comentarios. Máximo 150 palabras, sin hashtags.

**Qué observar:** gancho, estructura escaneable, tono acorde con la audiencia.

### Prompt 8. Explicar un concepto

**Vaga:**

> Explícame qué es un token.

**Precisa:**

> &#91;ROL\] Eres un profesor que explica conceptos técnicos con analogías cotidianas. Explica qué es un token en un modelo de lenguaje a una persona sin formación técnica. Que quede claro por qué importa para el costo y el límite de un chat. \[FEW-SHOT\] Ejemplo de estilo: "Un píxel es la unidad mínima de una imagen: no ves píxeles, ves la foto completa, pero la foto se construye con ellos." \[FORMATO\] 1) Analogía en 2 frases, 2) definición en 1 frase, 3) un ejemplo con la frase "Hola, ¿cómo estás?", 4) una pregunta de comprensión al final. Máximo 130 palabras.

**Qué observar:** analogía cotidiana, estructura numerada y verificación de comprensión.

### Prompt 9. Tabla comparativa para decidir

**Vaga:**

> Compárame Excel y Google Sheets.

**Precisa:**

> &#91;ROL\] Eres un consultor de productividad para pequeñas empresas. Compara Excel y Google Sheets para un equipo de 6 personas que trabaja a distancia, edita archivos en simultáneo y no tiene personal de TI. Presupuesto bajo. \[FEW-SHOT\] Ejemplo de fila: "Colaboración | Excel: posible con OneDrive, requiere configuración | Sheets: nativa, sin configuración | Ventaja: Sheets". \[FORMATO\] Tabla con 6 criterios (costo, colaboración, funciones avanzadas, curva de aprendizaje, funcionamiento sin conexión, integraciones). Después, una recomendación final de 2 frases y una condición bajo la cual cambiarías de opinión.

**Qué observar:** si compara según TU situación y si indica sus supuestos (verifica precios actuales por tu cuenta).

### Prompt 10. Revisar una consulta SQL o fórmula

**Vaga:**

> Arréglame esta consulta: \[código\]

**Precisa:**

> &#91;ROL\] Eres un ingeniero de datos senior que hace revisión de código con enfoque didáctico. Revisa la consulta SQL siguiente. Debe devolver las ventas totales por cliente del último mes, pero muestra duplicados. Motor: PostgreSQL. \[FEW-SHOT\] Ejemplo de hallazgo: "Línea 3: JOIN sin condición genera producto cartesiano. Corrección: agregar ON c.id = v.cliente_id. Por qué: sin ella cada fila se multiplica." \[FORMATO\] 1) Diagnóstico en una frase, 2) lista de hallazgos (línea, problema, corrección, por qué), 3) consulta corregida en un bloque de código. No cambies nada que no esté relacionado con el problema. Consulta: \[código\]

**Qué observar:** explicación del porqué y cambios acotados.

---

## Bloque 4. Cómo comparar las respuestas

### Rúbrica (puntúa de 1 a 5 cada criterio)

| Criterio         | Pregunta                                      |
| ---------------- | --------------------------------------------- |
| Relevancia       | ¿Responde exactamente lo que necesitaba?      |
| Especificidad    | ¿Usa mis datos o suena genérico?              |
| Formato          | ¿Respeta la estructura y la longitud pedidas? |
| Tono y audiencia | ¿Suena como lo usaría realmente?              |
| Usabilidad       | ¿Puedo usarlo tal cual o debo reescribirlo?   |
| Fiabilidad       | ¿Inventa datos o indica lo que falta?         |

### Tabla de registro (cópiala en una hoja de cálculo)

| Prompt | Puntaje vago | Puntaje preciso | Diferencia | Elemento que más influyó (rol, contexto, few-shot, formato) | Observación |
| ------ | ------------ | --------------- | ---------- | ----------------------------------------------------------- | ----------- |
| 1      |              |                 |            |                                                             |             |
| 2      |              |                 |            |                                                             |             |
| ...    |              |                 |            |                                                             |             |

### Experimentos para entender cuál elemento pesa más

Elige 3 prompts precisos y repite cada uno **quitando un elemento a la vez**:

1. Sin rol.
2. Sin few-shot.
3. Sin formato de salida.
4. Sin contexto.

Anota cuál elemento, al faltar, empeora más la respuesta. En la mayoría de los casos verás que el formato y el contexto son los que más cambian el resultado, y que el few-shot es lo que más estabiliza el estilo.

### Dos experimentos adicionales

- **Repetibilidad:** ejecuta el mismo prompt vago 3 veces y el mismo prompt preciso 3 veces. ¿Cuál varía menos?
- **Modelo distinto:** prueba el mismo par en dos asistentes. Verás que un buen prompt mejora los resultados en ambos.

---

## Checklist antes de enviar un prompt

- [ ] ¿Dije quién debe ser el modelo y para qué audiencia escribe?
- [ ] ¿Usé un verbo de acción claro (redacta, extrae, clasifica, compara)?
- [ ] ¿Di los datos, el objetivo y las restricciones?
- [ ] ¿Incluí al menos un ejemplo de entrada y salida?
- [ ] ¿Definí formato, estructura y longitud?
- [ ] ¿Le pedí que no invente datos y que marque lo que falta?
- [ ] ¿Evité incluir información sensible o confidencial?

## Errores frecuentes

1. **Pedir varias cosas a la vez** sin orden. Divide en pasos o numera.
2. **Ejemplos contradictorios** con las instrucciones: el modelo suele seguir el ejemplo.
3. **Formato implícito:** si no lo pides, no lo obtienes.
4. **Confiar sin verificar:** revisa cifras, nombres, citas y fuentes.
5. **No iterar:** el primer resultado es un borrador; refina con "hazlo más corto", "cambia el tono" o "agrega un ejemplo".

## Entregable sugerido

Un documento o presentación con: (1) definiciones propias de LLM, token, contexto y Transformer en tus palabras, (2) los tres casos del Bloque 2 con tus propios datos, (3) la tabla de registro del Bloque 4 completa y (4) tres conclusiones sobre qué elementos del prompt te dieron más control.

_Nota: los ejemplos de tokenización son ilustrativos. Los cursos y enlaces fueron verificados al preparar esta guía; si alguno cambia, busca el título del curso en el sitio oficial._
