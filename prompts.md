# Biblioteca de prompts — Laboratorio 07

## Tarea 1 — Clasificación de issues

**Herramienta:** Gemini

### Versión A — Sin ejemplos (zero-shot)
**Prompt:**
> Clasifica cada issue de GitHub indicando su tipo, prioridad y módulo.
> 1. "Al pagar con tarjeta aparece error 500 y el pedido queda duplicado."
> 2. "Agregar modo oscuro en la pantalla de perfil."
> 3. "¿El endpoint /api/pedidos acepta paginación?"
> 4. "El login tarda 8 segundos cuando hay más de 100 usuarios conectados."
> 5. "Actualizar la librería axios por una alerta de seguridad."

**Resultado (resumen):** 
La IA devolvió categorías heterogéneas (combinaciones como "Error / Bug", "Mejora / Feature", "Seguridad / Mantenimiento") y el formato no fue consistente en todos los puntos. Además, asignó prioridades arbitrarias sin un criterio unificado.

### Versión B — Few-shot (3 ejemplos)
**Prompt:**
> # Instrucción
> Clasifica cada issue siguiendo exactamente el formato de los ejemplos.
> Tipos válidos: BUG, FEATURE, CONSULTA, RENDIMIENTO, SEGURIDAD.
> Prioridad: ALTA (afecta dinero, datos o seguridad), MEDIA o BAJA.
> 
> # Ejemplos
> Issue: "La app se cierra al subir una foto de perfil."
> Tipo: BUG | Prioridad: MEDIA | Módulo: perfil
> 
> Issue: "Permitir exportar el historial de pedidos a Excel."
> Tipo: FEATURE | Prioridad: BAJA | Módulo: pedidos
> 
> Issue: "Las contraseñas se guardan en texto plano en la tabla usuarios."
> Tipo: SEGURIDAD | Prioridad: ALTA | Módulo: autenticación
> 
> # Issues a clasificar
> 1. "Al pagar con tarjeta aparece error 500 y el pedido queda duplicado."
> 2. "Agregar modo oscuro en la pantalla de perfil."
> 3. "¿El endpoint /api/pedidos acepta paginación?"
> 4. "El login tarda 8 segundos cuando hay más de 100 usuarios conectados."
> 5. "Actualizar la librería axios por una alerta de seguridad."

**Resultado (resumen):** 
Gemini respondió siguiendo estrictamente la estructura dada:
- Issue 1: Tipo: BUG | Prioridad: ALTA | Módulo: pagos
- Issue 2: Tipo: FEATURE | Prioridad: BAJA | Módulo: perfil
- Issue 3: Tipo: CONSULTA | Prioridad: BAJA | Módulo: pedidos
- Issue 4: Tipo: RENDIMIENTO | Prioridad: MEDIA | Módulo: autenticación
- Issue 5: Tipo: SEGURIDAD | Prioridad: ALTA | Módulo: dependencias

**Mejora observada:** 
1. **Tipos válidos:** Se limitó exclusivamente a la lista de tipos permitidos (BUG, FEATURE, CONSULTA, RENDIMIENTO, SEGURIDAD).
2. **Criterio de prioridad:** Aplicó correctamente la regla de asignación de prioridades (por ejemplo, asignó ALTA al error de pago y a la alerta de seguridad).
3. **Formato uniforme:** Mantuvo la estructura `Tipo: ... | Prioridad: ... | Módulo: ...` idéntica en los 5 casos.

---

## Tarea 2 — Razonamiento lógico

**Herramienta:** Gemini

### Versión A — Sin ejemplos (zero-shot)
**Prompt:**
> Un sistema tiene 7 microservicios que se despliegan por rondas.
> Un servicio solo se despliega cuando todas sus dependencias ya están desplegadas.
> Los servicios sin dependencias pendientes se despliegan en paralelo en la misma ronda.
> - usuarios depende de auth
> - pagos depende de usuarios
> - pedidos depende de usuarios, pagos y catalogo
> - notificaciones depende de pedidos
> - gateway depende de auth y notificaciones
> - auth y catalogo no tienen dependencias
> ¿Cuántas rondas se necesitan como mínimo? Responde solo con el número.

**Resultado (resumen):** 
Gemini respondió solo con el número `6`.

### Versión B — Few-shot (3 ejemplos)
**Prompt:**
> # Instrucción
> Resuelve el problema razonando ronda por ronda, igual que en los ejemplos.
> 
> # Ejemplo 1
> Servicios: A, B. B depende de A.
> Razonamiento: Ronda 1: A (sin dependencias). Ronda 2: B (A ya está desplegado).
> Respuesta: 2 rondas.
> 
> # Ejemplo 2
> Servicios: A, B, C. C depende de A y B.
> Razonamiento: Ronda 1: A y B en paralelo. Ronda 2: C.
> Respuesta: 2 rondas.
> 
> # Ejemplo 3
> Servicios: A, B, C, D. B y C dependen de A. D depende de B y C.
> Razonamiento: Ronda 1: A. Ronda 2: B y C en paralelo. Ronda 3: D.
> Respuesta: 3 rondas.
> 
> # Problema
> Un sistema tiene 7 microservicios que se despliegan por rondas.
> Un servicio solo se despliega cuando todas sus dependencias ya están desplegadas.
> Los servicios sin dependencias pendientes se despliegan en paralelo en la misma ronda.
> - usuarios depende de auth
> - pagos depende de usuarios
> - pedidos depende de usuarios, pagos y catalogo
> - notificaciones depende de pedidos
> - gateway depende de auth y notificaciones
> - auth y catalogo no tienen dependencias
> ¿Cuántas rondas se necesitan como mínimo?

**Resultado (resumen):** 
Razonamiento generado por Gemini:
- Ronda 1: auth y catalogo (sin dependencias).
- Ronda 2: usuarios (depende de auth).
- Ronda 3: pagos (depende de usuarios).
- Ronda 4: pedidos (depende de usuarios, pagos y catalogo).
- Ronda 5: notificaciones (depende de pedidos).
- Ronda 6: gateway (depende de auth y notificaciones).
Respuesta final: 6 rondas.

**Mejora observada:** 
1. **Explicabilidad:** Ambas versiones llegaron al número correcto (6), pero la Versión B desglose el proceso paso a paso (Chain of Thought), lo que facilita enormemente auditorías y detección de errores lógicos.
2. **Detección de dependencias circulares:** Al probar agregar la dependencia "catalogo depende de gateway", la Versión B permitió identificar explícitamente el ciclo formado entre catalogo -> pedidos -> notificaciones -> gateway -> catalogo.

---

## Tarea 3 — Extracción estructurada

**Herramienta:** Gemini

### Versión A — Sin ejemplos (zero-shot)
**Prompt:**
> Extrae los datos importantes de estos reportes de error en formato JSON.
> Reporte 1: "Desde ayer, en la versión 2.3.1 de Android, cuando un cliente aplica el cupón BIENVENIDA20 el total no se actualiza. Lo reportó Ana Díaz de soporte."
> Reporte 2: "En la web el botón Descargar factura no hace nada y no muestra ningún error. Reportado por Luis Paredes el 3 de octubre."

**Resultado (resumen):** 
Gemini generó una estructura JSON inconsistente entre ambos reportes. Nombres de claves variables (ej. `plataforma` vs `sistema`, `reportero` vs `autor`), ausencia de tratamiento estandarizado para campos faltantes (omitía campos o usaba cadenas vacías) y la respuesta incluyó texto explicativo antes del JSON.

### Versión B — Few-shot (3 ejemplos)
**Prompt:**
> # Instrucción
> Convierte cada reporte en un objeto JSON con el mismo esquema de los ejemplos.
> Si un dato no aparece, usa null. Devuelve solo un arreglo JSON válido.
> 
> # Ejemplo 1
> Reporte: "En iOS, versión 2.2.0, la app se congela al abrir el mapa. Reportó Carla Soto."
> {"plataforma": "iOS", "version_app": "2.2.0", "modulo": "mapa", "sintoma": "la app se congela", "reportado_por": "Carla Soto", "fecha": null}
> 
> # Ejemplo 2
> Reporte: "El 12 de septiembre, en la web, el formulario de registro acepta correos sin @."
> {"plataforma": "web", "version_app": null, "modulo": "registro", "sintoma": "acepta correos sin @", "reportado_por": null, "fecha": "12 de septiembre"}
> 
> # Ejemplo 3
> Reporte: "Jorge Ruiz indica que en Android 2.3.0 las notificaciones llegan duplicadas."
> {"plataforma": "Android", "version_app": "2.3.0", "modulo": "notificaciones", "sintoma": "notificaciones duplicadas", "reportado_por": "Jorge Ruiz", "fecha": null}
> 
> # Reportes a convertir
> Reporte 1: "Desde ayer, en la versión 2.3.1 de Android, cuando un cliente aplica el cupón BIENVENIDA20 el total no se actualiza. Lo reportó Ana Díaz de soporte."
> Reporte 2: "En la web el botón Descargar factura no hace nada y no muestra ningún error. Reportado por Luis Paredes el 3 de octubre."

**Resultado (resumen):** 
Devolvió un arreglo JSON perfectamente estructurado respetando exactamente los nombres de las claves (`plataforma`, `version_app`, `modulo`, `sintoma`, `reportado_por`, `fecha`), asignando `null` de forma limpia a `version_app` en el segundo reporte y conservando expresiones relativas como "Desde ayer" en el campo fecha.

**Mejora observada:** 
1. **Esquema JSON rígido:** Todos los objetos dentro del arreglo comparten las mismas claves y orden.
2. **Mapeo de valores faltantes:** Asignó explícitamente `null` cuando faltaban atributos como la versión de la app.
3. **Sintaxis válida:** La salida devuelta pasa la validación sin errores en herramientas como `jsonlint.com`.

---

## Tarea 4 — Revisión de código con Role Prompting y XML

**Herramienta:** Gemini

### Versión A — Sin rol (zero-shot)
**Prompt:**
> Revisa este código:
> ```python
> def login(usuario, clave):
>     query = "SELECT * FROM usuarios WHERE nombre='" + usuario + "' AND clave='" + clave + "'"
>     return db.execute(query).fetchone() is not None
> ```

**Resultado (resumen):** 
Gemini dio una respuesta general señalando que el código era vulnerable a inyección SQL y proporcionó un ejemplo básico de corrección. No estructuró la respuesta de forma clara ni profundizó en otros problemas de seguridad.

### Versión B — Con rol y estructura XML
**Prompt:**
> <rol>
> Eres un revisor de seguridad de aplicaciones web con experiencia en OWASP Top 10.
> </rol>
> <contexto>
> Función de login de una API en Python con SQLite. El equipo de desarrollo es junior.
> </contexto>
> <codigo>
> def login(usuario, clave):
>     query = "SELECT * FROM usuarios WHERE nombre='" + usuario + "' AND clave='" + clave + "'"
>     return db.execute(query).fetchone() is not None
> </codigo>
> <tarea>
> 1. Identifica las vulnerabilidades y explica el riesgo de cada una.
> 2. Propón el código corregido usando consultas parametrizadas.
> </tarea>
> <formato>
> Tabla Markdown: Vulnerabilidad | Riesgo | Corrección. Luego, el código corregido.
> </formato>

**Resultado (resumen):** 
Al asignarle el rol de experto en OWASP Top 10 y separar las secciones mediante etiquetas XML, Gemini:
1. Generó exactamente la tabla Markdown solicitada desglosando Vulnerabilidad, Riesgo y Corrección.
2. Identificó tanto la Inyección SQL como la falta de hashing de contraseñas.
3. Proporcionó el código corregido usando tuplas para la parametrización de SQLite (`?`) con explicaciones pedagógicas adaptadas al contexto del equipo junior.

**Mejora observada:** 
1. **Enfoque técnico profesional:** La respuesta adoptó la perspectiva y terminología de un auditor de seguridad (OWASP A03:2021-Injection).
2. **Estructura estricta:** Las etiquetas XML permitieron aislar claramente el contexto, la tarea y el formato deseado sin confusión.
3. **Presentación clara:** La tabla facilitó una lectura rápida y directa de los hallazgos y soluciones.

---

## FASE 5 — Evaluación comparativa

### Paso 5.1 — Rúbrica de evaluación

| Criterio     | Descripción                                    | Puntaje Máximo |
| ------------ | ---------------------------------------------- | :------------: |
| Exactitud    | La respuesta es correcta (verificada por ti)   |       5        |
| Formato      | Respeta el formato o esquema solicitado        |       5        |
| Consistencia | Aplica el mismo criterio en todos los casos    |       5        |
| Utilidad     | Se puede usar directamente en el proyecto      |       5        |

*Nota: La puntuación total por tarea se calcula sumando los 4 criterios (máximo 20 puntos).*

---

### Paso 5.2 — Comparación: sin ejemplos vs. técnica avanzada

| Tarea                       | Versión A | Versión B | Mejora principal |
| --------------------------- | :-------: | :-------: | ---------------- |
| 1. Clasificación de issues  |   12/20   |   20/20   | Estandarización de categorías y formato uniforme mediante few-shot. |
| 2. Razonamiento lógico      |   15/20   |   20/20   | Desglose paso a paso (Chain of Thought) que facilita auditorías y detección de errores/ciclos. |
| 3. Extracción estructurada  |   11/20   |   20/20   | Garantía de un esquema JSON estricto, sintaxis válida y asignación explícita de `null`. |
| 4. Revisión de código (rol) |   13/20   |   19/20   | Mayor nivel de detalle técnico con OWASP Top 10 y delimitación clara mediante XML. |

---

## FASE 6 — Biblioteca de prompts y publicación

### Paso 6.1 — Plantillas reutilizables

#### Plantilla 1: Clasificación few-shot
```text
Actúa como {{rol}}.
Clasifica cada {{elemento}} siguiendo el formato de los ejemplos.
Categorías válidas: {{categorias}}.
Prioridad: {{criterio_prioridad}}.

# Ejemplos
Ejemplo 1: {{entrada_1}} -> Tipo: {{tipo_1}} | Prioridad: {{prioridad_1}} | Módulo: {{modulo_1}}
Ejemplo 2: {{entrada_2}} -> Tipo: {{tipo_2}} | Prioridad: {{prioridad_2}} | Módulo: {{modulo_2}}
Ejemplo 3: {{entrada_3}} -> Tipo: {{tipo_3}} | Prioridad: {{prioridad_3}} | Módulo: {{modulo_3}}

# Elementos a clasificar
{{datos}}
```