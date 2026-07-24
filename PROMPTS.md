# Banco de Prompts — Lesli

## Índice

- [Resumen de textos](#resumen-de-textos)
- [Extracción de información](#extracción-de-información)
- [Clasificación de texto](#clasificación-de-texto)
- [Generación de código](#generación-de-código)
- [Explicación de errores de consola](#explicación-de-errores-de-consola)

---

## Resumen de textos

**Objetivo:** Condensar un artículo o documentación técnica larga en sus puntos clave, para uso interno del equipo de desarrollo.

**Elementos aplicados:** rol, tarea, formato, restricciones

**Prompt (plantilla):**
```text
Actúa como editor/a técnico especializado en documentación de software. Resume el siguiente texto en no más de 5 líneas, conservando únicamente la información técnica relevante para un desarrollador. Devuelve el resumen como una lista de viñetas.

Texto:
"""
{{texto}}
"""
```

**Ejemplo probado:**
- Entrada 

```
Summary
The Node.js project will release new versions of the 26.x, 24.x, 22.x releases lines on or shortly after, Monday, July 27, 2026 in order to address:

The highest severity issue fixed in this release is HIGH.

Impact
The highest severity issue fixed in the 26.x release line is HIGH. The highest severity issue fixed in the 24.x release line is HIGH. The highest severity issue fixed in the 22.x release line is HIGH.

It's important to note that End-of-Life versions are always affected when a security release occurs. To ensure your system's security, please use an up-to-date version as outlined in our Release Schedule.

Release timing
Releases will be available on, or shortly after, Monday, July 27, 2026.

Contact and future updates
The current Node.js security policy can be found at https://nodejs.org/en/security/. Please follow the process outlined in https://github.com/nodejs/node/blob/master/SECURITY.md if you wish to report a vulnerability in Node.js.

Subscribe to the low-volume announcement-only nodejs-sec mailing list at https://groups.google.com/forum/#!forum/nodejs-sec to stay up to date on security vulnerabilities and security-related releases of Node.js and the projects maintained in the nodejs GitHub organization.
```
  
- Salida obtenida:
  - Versiones objetivo: Publicación de parches de seguridad para las líneas activas 26.x, 24.x y 22.x de Node.js.
  - Fecha de despliegue: Programado para el lunes 27 de julio de 2026 (o fecha próxima posterior).
  - Impacto: Soluciona vulnerabilidades con un nivel de gravedad máximo ALTO (HIGH) en todas las ramas mencionadas.
  - Versiones EOL: Las versiones fuera de soporte (End-of-Life) también son vulnerables y no recibirán parches; se requiere migrar a una rama activa.

**Notas de iteración:** La v1 del prompt no pedía formato de lista y el modelo devolvía un párrafo largo; agregar "Devuelve el resumen como una lista de viñetas" resolvió el problema.

---

## Extracción de información

**Objetivo:** Extraer entidades específicas (nombres, fechas, montos) de un texto no estructurado y formatearlo en una estructura de datos clara.

**Elementos aplicados:** rol, tarea, formato, restricciones

**Prompt (plantilla):**
```text
Actúa como un analista de datos. Analiza el siguiente texto y extrae todos los nombres de empresas, fechas y montos de dinero mencionados. Devuelve la información extraída estrictamente en formato JSON, usando las claves "empresas", "fechas" y "montos". No agregues ningún texto explicativo fuera del bloque JSON.

Texto:
"""
{{texto}}
"""
```

**Ejemplo probado:**
- Entrada (`{{texto}}`): "El 15 de marzo de 2023, la empresa TechCorp adquirió a InnovateSolutions por un monto de 50 millones de dólares."
- Salida obtenida: 
```json
{
  "empresas": [
    "TechCorp",
    "InnovateSolutions"
  ],
  "fechas": [
    "15 de marzo de 2023"
  ],
  "montos": [
    "50 millones de dólares"
  ]
}
```

**Notas de iteración:** Al principio el modelo agregaba texto conversacional como "Aquí tienes el JSON solicitado". Al agregar la restricción "No agregues ningún texto explicativo fuera del bloque JSON" se logró una salida automatizable y limpia.

---

## Clasificación de texto

**Objetivo:** Clasificar el sentimiento de los comentarios o tickets de soporte de los clientes para facilitar el análisis de feedback.

**Elementos aplicados:** rol, tarea, formato, ejemplos, restricciones

**Prompt (plantilla):**
```text
Actúa como un especialista en atención al cliente. Tu tarea es clasificar el sentimiento del siguiente comentario de un cliente. 
Las únicas categorías permitidas son: POSITIVO, NEGATIVO, NEUTRAL.

Ejemplos:
- "Me encantó el servicio, muy rápido." -> POSITIVO
- "El paquete llegó tarde y roto." -> NEGATIVO
- "El producto es tal cual la descripción." -> NEUTRAL

Solo responde con la palabra de la categoría correspondiente, sin justificación ni puntuación adicional.

Comentario:
"""
{{texto}}
"""
```

**Ejemplo probado:**
- Entrada (`{{texto}}`): "No volveré a comprar aquí, la atención fue pésima y nadie me resolvió el problema."
- Salida obtenida: NEGATIVO

**Notas de iteración:** Se agregaron ejemplos (few-shot prompting) para asegurar que el modelo entienda exactamente la tarea y no se desvíe del formato de salida esperado.

---

## Generación de código

**Objetivo:** Generar fragmentos de código, funciones o scripts con buenas prácticas, incluyendo documentación y tipos de datos.

**Elementos aplicados:** rol, tarea, formato, restricciones

**Prompt (plantilla):**
```text
Actúa como un desarrollador Senior en Python. Escribe una función que cumpla con el siguiente requerimiento: {{texto}}.

La función debe incluir obligatoriamente:
- Type hints (anotaciones de tipo).
- Docstring en formato Google.
- Manejo de excepciones (try/except) básico si aplica.

Solo devuelve el bloque de código, sin saludos, despedidas ni explicaciones adicionales.
```

**Ejemplo probado:**
- Entrada (`{{texto}}`): "una función que tome una lista de diccionarios y devuelva una nueva lista ordenada por la clave 'edad' de forma descendente."
- Salida obtenida: 
```python
from typing import List, Dict, Any

def ordenar_por_edad(usuarios: List[Dict[str, Any]]) -> List[Dict[str, Any]]:
    """
    Ordena una lista de diccionarios por la clave 'edad' de forma descendente.

    Args:
        usuarios (List[Dict[str, Any]]): Lista de diccionarios con información de usuarios.

    Returns:
        List[Dict[str, Any]]: Nueva lista ordenada por edad (mayor a menor).
    """
    try:
        return sorted(usuarios, key=lambda x: x.get('edad', 0), reverse=True)
    except Exception as e:
        print(f"Error al ordenar la lista: {e}")
        return []
```

**Notas de iteración:** La versión inicial devolvía código funcional pero sin comentarios ni tipos. Se ajustó el prompt para forzar explícitamente la inclusión de type hints y un estilo de docstring (formato Google) para mantener un estándar de calidad.

---

## Explicación de errores de consola

**Objetivo:** Ayudar a desarrolladores a entender errores complejos de terminal o consola de manera simple y sugerir soluciones prácticas.

**Elementos aplicados:** rol · contexto · tarea · formato

**Prompt (plantilla):**
```text
Actúa como un mentor de programación amable y con mucha experiencia. Un desarrollador junior de tu equipo se encontró con el siguiente error en su consola al intentar ejecutar su código.

Explica qué significa este error en términos sencillos, por qué suele ocurrir y sugiere 2 o 3 pasos claros (con ejemplos de código si es necesario) para solucionarlo.

Error:
"""
{{texto}}
"""
```

**Ejemplo probado:**
- Entrada (`{{texto}}`): "TypeError: Cannot read properties of undefined (reading 'map')"
- Salida obtenida:
```text
¡Hola! No te preocupes, este es uno de los errores más comunes en JavaScript/React.

**¿Qué significa?**
Básicamente, estás intentando usar el método `.map()` sobre una variable que actualmente es `undefined` (no tiene ningún valor asignado todavía), en lugar de ser un arreglo (array).

**¿Por qué suele ocurrir?**
Generalmente pasa cuando intentas renderizar o procesar datos que provienen de una API (y aún no han cargado), o si olvidaste inicializar un estado en React como un arreglo vacío.

**¿Cómo solucionarlo?**
1. **Verifica la inicialización:** Asegúrate de que tu estado comience como un arreglo vacío: `const [datos, setDatos] = useState([])`.
2. **Usa Optional Chaining (?):** Puedes agregar un signo de interrogación antes del punto para que JavaScript no intente mapear si es undefined: `datos?.map(item => ...)`
3. **Renderizado condicional:** Asegúrate de que tienes los datos antes de hacer el map: `datos && datos.map(item => ...)`
```

**Notas de iteración:** Originalmente la respuesta del modelo era demasiado abstracta y técnica. Al agregar el rol de "mentor amable" y especificar que va dirigido a un "desarrollador junior", el tono se volvió más didáctico, empático y mucho más útil.
