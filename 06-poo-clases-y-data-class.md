# 06 · POO en Kotlin: clases y `data class`

## 6.1 Clase básica

```kotlin
// El constructor principal va en la propia cabecera de la clase.
// val/var en los parámetros crea directamente las propiedades.
class Alumno(val nombre: String, var nota: Double) {

    // bloque de inicialización: se ejecuta al construir el objeto
    init {
        require(nota in 0.0..10.0) { "Nota fuera de rango" }
    }

    fun estaAprobado(): Boolean = nota >= 5.0
}

val a = Alumno("Ana", 8.0)     // sin "new"
println(a.nombre)              // acceso por propiedad, no por getter
a.nota = 9.0
```

Ese código equivale a unas 30 líneas de Java con constructor, campos privados, *getters* y *setters*.

## 6.2 Propiedades con lógica

```kotlin
class Rectangulo(val base: Double, val altura: Double) {

    // propiedad calculada: no ocupa memoria, se evalúa al leerla
    val area: Double
        get() = base * altura

    // getter/setter personalizados
    var nombre: String = ""
        set(value) {
            field = value.trim().uppercase()   // "field" es el campo de respaldo
        }
}
```

## 6.3 Constructores secundarios

```kotlin
class Usuario(val nombre: String, val email: String) {
    constructor(nombre: String) : this(nombre, "$nombre@centro.es")
}
```

En la práctica se usan poco: los **parámetros por defecto** (unidad 04) los sustituyen casi siempre.

## 6.4 Visibilidad

| Modificador | Alcance |
|---|---|
| `public` | Por defecto. Visible desde cualquier sitio |
| `private` | Sólo dentro de la clase (o del fichero) |
| `protected` | Clase y subclases |
| `internal` | Dentro del mismo módulo |

```kotlin
class Cuenta(private var saldo: Double) {
    fun ingresar(cantidad: Double) {
        require(cantidad > 0) { "La cantidad debe ser positiva" }
        saldo += cantidad
    }
    fun consultar() = saldo
}
```

## 6.5 `data class`

Para clases cuyo cometido es **transportar datos** (modelos, respuestas de API, entidades):

```kotlin
data class Alumno(
    val nombre: String,
    val nota: Double,
    val curso: Int = 1
)
```

El compilador genera automáticamente:

- `equals()` / `hashCode()` basados en las propiedades del constructor principal
- `toString()` legible: `Alumno(nombre=Ana, nota=8.5, curso=2)`
- `copy()` para crear copias modificadas
- `componentN()` para el *destructuring*

```kotlin
val ana = Alumno("Ana", 8.5, 2)
val anaRevisada = ana.copy(nota = 9.0)     // se crea un objeto nuevo; ana no cambia

println(ana == Alumno("Ana", 8.5, 2))      // true: compara CONTENIDO
println(ana === Alumno("Ana", 8.5, 2))     // false: distinta referencia
```

> **Patrón clave para Android:** los modelos de datos son `data class` inmutables
> y los cambios se expresan con `.copy()`. Así el estado de la interfaz es siempre
> predecible: nadie puede modificar un objeto "por debajo".

> 📄 Antes de seguir, repasad [Diferencia entre val y var](val-vs-var.md): ahí se explica
> con detalle que `val` impide reasignar la referencia, pero no convierte el objeto en
> inmutable (por ejemplo, una `val lista = mutableListOf(...)` se puede seguir modificando).
> Esa distinción es la que hace necesario el patrón `copy()` que acabáis de ver.

## 6.6 Comparación rápida con Java

| Java | Kotlin |
|---|---|
| `new Alumno(...)` | `Alumno(...)` |
| `getNombre()` / `setNota()` | `alumno.nombre` / `alumno.nota = ...` |
| POJO con 60 líneas | `data class` de 4 líneas |
| `record` (Java 16+) | `data class` (con `copy()` además) |

## Ejercicios

1. Crea la clase `Libro` con título, autor, año y disponibilidad; añade un método `prestar()` que cambie el estado.
2. Convierte a `data class` un POJO `Producto` de tus prácticas de 1º y comprueba `equals`, `toString` y `copy`.
3. Clase `CuentaBancaria` con saldo privado, métodos `ingresar`/`retirar` y validación con `require`.
4. `data class Punto(val x: Int, val y: Int)` con una propiedad calculada `distanciaAlOrigen`.

**Para profundizar:** investiga `lateinit`, `by lazy` y en qué situaciones de Android se usa cada uno.
