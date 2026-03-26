# Catálogo de Anti-patrones y Problemas Comunes

Referencia rápida para el análisis de commits. Cada sección lista patrones problemáticos concretos con ejemplos mínimos.

---

## Seguridad

### Secrets hardcodeados
Claves API, tokens, contraseñas, connection strings directamente en el código fuente.

```
// Problemático
const API_KEY = "sk-abc123def456";
const dbUrl = "postgres://admin:password@prod-db:5432/app";
```

Buscar: strings que parezcan tokens (prefijos como `sk-`, `ghp_`, `AKIA`, `Bearer`), variables con nombres como `password`, `secret`, `token`, `key`, `credential` asignadas a literales string.

### Inyección SQL
Queries construidas con concatenación o template literals usando input del usuario.

```
// Problemático
const query = `SELECT * FROM users WHERE id = ${userId}`;
db.query("SELECT * FROM users WHERE name = '" + name + "'");
```

### XSS (Cross-Site Scripting)
Inserción de contenido del usuario en el DOM sin sanitizar.

```
// Problemático
element.innerHTML = userInput;
dangerouslySetInnerHTML={{ __html: userContent }}
```

### Path traversal
Construcción de rutas de archivos usando input del usuario sin validar.

```
// Problemático
const filePath = `./uploads/${req.params.filename}`;
fs.readFile(path.join(baseDir, userInput));
```

---

## Errores lógicos

### Comparación laxa en JavaScript/TypeScript
Uso de `==` o `!=` en lugar de `===` o `!==`, que permite coerción de tipos.

```
// Problemático
if (value == null)     // captura null Y undefined
if (status == "200")   // "200" == 200 es true
if (arr == [])         // siempre false
```

Excepción: `== null` es un patrón aceptado para verificar null/undefined simultáneamente.

### Off-by-one
Errores en límites de iteración, índices de arrays, slicing.

```
// Problemático
for (let i = 0; i <= arr.length; i++)  // accede a arr[arr.length] que es undefined
items.slice(0, count - 1)  // omite el último elemento
```

### Condiciones invertidas o redundantes

```
// Problemático
if (!isValid || !isValid)   // condición duplicada
if (x > 0 && x > 10)       // primera condición es redundante
if (status === "active" || status !== "inactive")  // siempre true
```

### Asignación en lugar de comparación

```
// Problemático
if (user = getUser())    // asigna en vez de comparar
while (result = true)    // loop infinito
```

---

## Manejo de errores

### Catch vacío o silenciador
Captura excepciones sin hacer nada, escondiendo errores.

```
// Problemático
try { riskyOperation(); } catch (e) {}
try { riskyOperation(); } catch (e) { /* TODO */ }
.catch(() => {})
```

### Promesas sin await
Funciones async que llaman operaciones async sin await, causando ejecución no controlada.

```
// Problemático
async function save() {
  database.insert(data);   // falta await
  return "saved";          // retorna antes de que se complete el insert
}
```

### Throw de tipos no-Error

```
// Problemático
throw "something went wrong";   // pierde stack trace
throw 404;                       // no es informativo
throw { message: "error" };      // pierde stack trace
```

---

## Recursos y memoria

### Event listeners sin cleanup
Registrar listeners en mount sin removerlos en unmount.

```
// Problemático (React)
useEffect(() => {
  window.addEventListener('resize', handler);
  // falta return () => window.removeEventListener('resize', handler)
}, []);
```

### Timers sin cleanup

```
// Problemático
useEffect(() => {
  setInterval(pollData, 5000);
  // falta clearInterval en cleanup
}, []);
```

### File handles sin cerrar
Abrir archivos o conexiones sin asegurar que se cierren.

```
// Problemático (Python)
f = open("data.txt")
data = f.read()
# falta f.close() o usar context manager (with)
```

---

## Prácticas de código

### Funciones demasiado largas
Funciones con más de ~50 líneas o que hacen múltiples cosas no relacionadas. Señal: necesitás comentarios para separar secciones lógicas dentro de la función.

### Demasiados parámetros
Funciones con más de 4 parámetros. Considerar usar un objeto de opciones.

```
// Problemático
function createUser(name, email, age, role, department, manager, startDate) {}

// Mejor
function createUser(options: CreateUserOptions) {}
```

### Magic numbers y strings
Valores literales sin nombre que oscurecen su significado.

```
// Problemático
if (retries > 3) {}
setTimeout(callback, 86400000);
if (user.role === "admin_v2") {}
```

### Código muerto
- Variables declaradas pero nunca usadas
- Imports que no se referencian
- Funciones definidas pero nunca llamadas
- Bloques de código comentados (si es temporal, debería tener un TODO con contexto)

### Naming pobre
- Variables de una letra fuera de loops (`x`, `d`, `t`)
- Nombres genéricos (`data`, `result`, `temp`, `info`, `handler`)
- Booleanos sin prefijo interrogativo (`active` vs `isActive`)
- Funciones sin verbo (`userData()` vs `getUserData()`)

### any en TypeScript
Uso de `any` que anula el sistema de tipos.

```
// Problemático
function process(data: any): any {}
const items: any[] = [];
(value as any).method();
```

---

## Rendimiento

### Re-renders innecesarios (React)
- Objetos o arrays creados inline como props: `style={{ color: 'red' }}`
- Funciones creadas en cada render sin useCallback
- Falta de useMemo para cómputos costosos
- Componentes sin memo que reciben props que cambian por referencia

### N+1 queries
Queries en loops que deberían ser una sola query batch.

```
// Problemático
for (const userId of userIds) {
  const user = await db.query("SELECT * FROM users WHERE id = ?", userId);
}
```

### Operaciones síncronas bloqueantes
Uso de métodos `*Sync` de fs en código de servidor, o cómputos pesados en el main thread.

```
// Problemático en servidor
const data = fs.readFileSync(largePath);
const parsed = JSON.parse(hugeString);  // si hugeString es muy grande
```

---

## Patrones específicos de Python

### Mutable default arguments

```python
# Problemático
def append_to(element, target=[]):
    target.append(element)
    return target
```

### Bare except

```python
# Problemático
try:
    operation()
except:        # captura TODO, incluyendo KeyboardInterrupt y SystemExit
    pass
```

### String formatting inseguro para SQL

```python
# Problemático
cursor.execute(f"SELECT * FROM users WHERE name = '{name}'")
cursor.execute("SELECT * FROM users WHERE name = '%s'" % name)
```
