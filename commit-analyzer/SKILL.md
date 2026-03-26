---
name: commit-analyzer
description: >
  Analiza cambios staged en git para detectar bugs, vulnerabilidades de seguridad, malas prácticas, y genera
  descripciones detalladas de commits con mensaje en formato Conventional Commits. Usa este skill siempre que
  el usuario quiera revisar cambios antes de commitear o pushear, analizar un diff staged, detectar bugs o
  malas prácticas en código que está por commitear, generar un mensaje o descripción de commit, o hacer code
  review previo al commit. Se activa con frases como "revisá mis cambios staged", "analiza mi commit",
  "qué bugs tiene lo que cambié", "generame el mensaje de commit", "review antes de push", "detecta errores
  en mis cambios", "haceme un análisis antes de commitear", o "necesito una descripción para mi commit".
  NO usar para: code review de archivos sueltos sin contexto de commit, configurar linters, escribir tests,
  debugging de producción, o crear código nuevo. Este skill es específicamente para el momento previo al commit.
---

# Commit Analyzer

Analiza cambios staged en un repositorio git para encontrar problemas y documentar el commit de forma profesional.

## Cuándo usar este skill

Cuando el usuario quiera revisar sus cambios antes de hacer commit. El skill trabaja con cambios **staged** (`git add`), así que si no hay cambios staged, indicale al usuario que primero debe agregar archivos al staging area.

## Flujo de análisis

Seguí estos pasos en orden. Cada paso depende del anterior.

### Paso 1 — Obtener los cambios

Ejecutá estos tres comandos para tener el panorama completo:

```bash
git diff --staged --name-status
git diff --staged --stat
git diff --staged
```

Si no hay cambios staged, avisale al usuario y sugerile que ejecute `git add` con los archivos que quiera incluir. No continúes con el análisis si el diff está vacío.

Leé también los archivos completos que fueron modificados (no solo el diff) cuando necesites contexto adicional para entender si un patrón es realmente problemático. Un diff aislado puede ser engañoso — por ejemplo, una variable que parece no usarse en el diff podría usarse más abajo en el archivo.

### Paso 2 — Detectar bugs y problemas potenciales

Examiná el diff línea por línea buscando defectos concretos. No reportes especulaciones vagas — cada hallazgo debe señalar código específico y explicar por qué es un problema.

Categorías principales:

- **Errores lógicos**: off-by-one, condiciones invertidas, comparaciones incorrectas de tipos (`==` vs `===` en JS/TS)
- **Null safety**: acceso a propiedades sin verificar null/undefined, optional chaining faltante
- **Manejo de errores**: bloques catch vacíos, promesas sin await, errores silenciados
- **Concurrencia**: race conditions, estado compartido sin protección
- **Seguridad**: secrets hardcodeados, inyección SQL, XSS, datos sensibles en logs
- **Recursos**: memory leaks, file handles sin cerrar, event listeners sin cleanup

Consultá `references/patterns.md` (en el directorio de este skill) para un catálogo detallado de patrones problemáticos por categoría.

### Paso 3 — Evaluar prácticas de código

Revisá la calidad del código contra principios de código limpio. El objetivo no es ser pedante sino señalar cosas que genuinamente dificultan el mantenimiento futuro.

Aspectos a evaluar:

- **Naming**: ¿los nombres de variables, funciones y clases comunican su intención?
- **Tamaño**: ¿hay funciones excesivamente largas o con demasiados parámetros (>4)?
- **Duplicación**: ¿se repite lógica que podría extraerse?
- **Principios SOLID**: ¿hay responsabilidades mezcladas, acoplamiento innecesario?
- **Tipado**: en lenguajes tipados, ¿se usan `any`, tipos demasiado amplios, o falta tipado?
- **Dead code**: imports no usados, variables declaradas pero nunca leídas, código comentado

### Paso 4 — Generar recomendaciones

Para cada problema encontrado en los pasos 2 y 3, generá una recomendación estructurada.

Usá este formato para cada hallazgo:

```
### [SEVERIDAD] Título descriptivo del problema
- **Archivo**: `ruta/al/archivo.ts` (línea ~N del diff)
- **Problema**: Explicación clara de qué está mal y por qué importa
- **Sugerencia**:
  ```[lenguaje]
  // código sugerido como reemplazo
  ```
```

Niveles de severidad:
- **CRITICO**: Bugs que van a causar errores en runtime, vulnerabilidades de seguridad, pérdida de datos
- **ADVERTENCIA**: Malas prácticas que dificultan el mantenimiento o pueden causar bugs futuros
- **SUGERENCIA**: Mejoras de estilo o legibilidad que harían el código más limpio

Ordená los hallazgos por severidad (críticos primero). Si no encontrás ningún problema, decilo explícitamente — no inventes problemas donde no los hay.

### Paso 5 — Describir el commit

Generá una descripción completa del commit en español con esta estructura:

```markdown
## Resumen
[Una línea que capture la esencia del cambio]

## Archivos modificados
- `ruta/archivo.ext` — [qué se cambió en este archivo y por qué]
- `ruta/otro.ext` — [ídem]

## Alcance de los cambios
[Qué módulos, features o flujos se ven afectados por estos cambios.
Mencioná dependencias upstream/downstream si las hay.
Indicá si los cambios son backwards-compatible o si rompen algo.]

## Beneficios
- [Qué mejora concreta aporta este cambio]
- [Otro beneficio si aplica]

## Mensaje de commit sugerido
[tipo]([scope]): descripción corta en imperativo

[Cuerpo detallado explicando el qué y el por qué.
Mencioná decisiones de diseño relevantes.]
```

Para el mensaje de commit sugerido, usá el formato **Conventional Commits**:
- `feat`: nueva funcionalidad
- `fix`: corrección de bug
- `refactor`: cambio de estructura sin cambiar comportamiento
- `docs`: documentación
- `style`: formato, espacios, puntos y comas
- `test`: agregar o modificar tests
- `chore`: tareas de mantenimiento, dependencias, config

## Formato de salida completo

El reporte final debe tener estas tres secciones en este orden:

```
# Análisis de Commit

## 1. Reporte de Análisis
[Todos los hallazgos del paso 4, agrupados por severidad]

## 2. Descripción del Commit
[Estructura del paso 5]

## 3. Mensaje de Commit Sugerido
[El conventional commit listo para copiar y usar]
```

## Notas importantes

- Siempre respondé en español
- Sé específico: citá el código exacto del diff, no hables en abstracto
- No seas excesivamente crítico con cambios pequeños o triviales — ajustá la profundidad del análisis al tamaño del cambio
- Si el commit es limpio y bien hecho, reconocelo. No todos los análisis tienen que encontrar problemas
- Cuando sugieras código de reemplazo, asegurate de que sea compatible con el contexto del archivo
