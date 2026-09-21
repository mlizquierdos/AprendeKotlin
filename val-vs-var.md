# Diferencia entre `val` y `var`

En Kotlin, ambas palabras clave sirven para **declarar variables**, pero se diferencian en si podemos **reasignar su valor** después de la inicialización.

| Característica | `val` | `var` |
|---|---|---|
| Significado | Variable de solo lectura | Variable mutable |
| ¿Se puede reasignar? | No | Sí |
| ¿Es necesario indicar el tipo? | No, se puede inferir | No, se puede inferir |
| Uso habitual | Cuando el valor no necesita cambiar | Cuando el valor debe cambiar |

---

## `val`: una referencia que no se puede reasignar

Cuando declaramos una variable con `val`, **no** podemos asignarle otro valor después de su inicialización.

### Ejemplo

```kotlin
fun main() {
    val nombre = "Ana"
    println(nombre)
    nombre = "Luis"   // ERROR
}
```

**¿Qué sucede?** La variable `nombre` se ha inicializado con el valor `"Ana"`; cuando intentamos asignarle `"Luis"`, Kotlin muestra un error de compilación porque `val` no permite reasignar la variable:

```
Val cannot be reassigned
```

### Forma correcta

```kotlin
fun main() {
    val nombre = "Ana"
    println(nombre)
}
```

Salida:

```
Ana
```

### ¿Por qué es útil `val`?

Porque indica que esa variable no necesita cambiar de referencia durante su vida útil. Por ejemplo:

```kotlin
val nombreAplicacion = "Mi aplicación"
val numeroMaximo = 100
val precioProducto = 19.99
```

Si no necesitamos reasignar esos valores, podemos declararlos con `val`.

---

## `var`: una variable que puede cambiar

Cuando utilizamos `var`, podemos asignar un nuevo valor a la variable, siempre que sea compatible con su tipo.

### Ejemplo

```kotlin
fun main() {
    var edad = 20
    println(edad)
    edad = 21
    println(edad)
}
```

Salida:

```
20
21
```

En este caso:

1. Se declara `edad` con el valor 20.
2. Se modifica su valor a 21.
3. Se muestran ambos valores.

### Otro ejemplo

```kotlin
var contador = 0
contador = contador + 1
contador = contador + 1
println(contador)
```

Salida:

```
2
```

Este patrón es habitual en contadores, acumuladores y estados que cambian.

---

## ¿Qué ocurre con el tipo de dato?

Tanto `val` como `var` utilizan **tipado estático**.

```kotlin
var edad = 20
edad = 25      // Correcto
edad = "Hola"  // ERROR
```

Aunque `edad` sea mutable, **no podemos cambiar su tipo de `Int` a `String`**.

Lo mismo sucede con `val`:

```kotlin
val edad = 20
edad = 25   // ERROR: no se puede reasignar
```

Es decir:

- `val` → no permite reasignar.
- `var` → permite reasignar.
- Ambos mantienen el tipo de la variable.

---

## ¿Qué me interesa utilizar habitualmente?

> **Recomendación: utiliza `val` por defecto.**

En Kotlin es una buena práctica declarar las variables con `val` siempre que no necesites reasignarlas. Emplea `var` cuando realmente necesites que la variable cambie.

### Ejemplo práctico

Imagina que estás calculando el área de un rectángulo:

```kotlin
fun main() {
    val base = 5.0
    val altura = 3.0
    val area = base * altura
    println("Área: $area")
}
```

Todas las variables se declaran con `val`. ¿Por qué?

- La base no cambia durante el cálculo.
- La altura no cambia durante el cálculo.
- El área se calcula una vez y no necesitamos reasignarla.

No tiene sentido utilizar `var` si no necesitamos modificar los valores.

### Ejemplo donde sí interesa `var`

Imagina una aplicación Android que tiene un contador de pulsaciones de un botón. Cada vez que el usuario pulsa el botón, el contador aumenta.

```kotlin
var contador = 0

fun incrementarContador() {
    contador++
}
```

Aquí necesitamos `var` porque el valor cambia.

### Ejemplo en Jetpack Compose

```kotlin
@Composable
fun Contador() {
    var contador by remember { mutableStateOf(0) }

    Column {
        Text("Pulsaciones: $contador")
        Button(onClick = { contador++ }) {
            Text("Incrementar")
        }
    }
}
```

Cada pulsación modifica el valor de `contador`. En este contexto, `var` es necesario porque la variable se reasigna mediante el mecanismo de estado de Compose. En Compose, `remember` y `mutableStateOf` permiten conservar y observar el estado para actualizar la interfaz cuando cambia.

---

## Una diferencia importante: `val` no significa que el objeto sea inmutable

Este punto es especialmente importante para el alumnado.

```kotlin
val numeros = mutableListOf(1, 2, 3)
numeros.add(4)
println(numeros)
```

Salida:

```
[1, 2, 3, 4]
```

Pero hemos utilizado `val`. **¿Por qué podemos añadir elementos?** El motivo es que `val` impide reasignar la referencia, pero no necesariamente modificar el objeto al que apunta.

### Lo que NO podemos hacer

```kotlin
val numeros = mutableListOf(1, 2, 3)
numeros = mutableListOf(4, 5, 6)   // ERROR
```

No podemos asignar otra lista a la variable.

### Lo que SÍ podemos hacer

```kotlin
val numeros = mutableListOf(1, 2, 3)
numeros.add(4)
numeros.remove(2)
println(numeros)
```

Salida:

```
[1, 3, 4]
```

Es decir:

```
val numeros ────────► Objeto mutable (lista)
                │
                ├── Añadir elementos: SÍ
                ├── Eliminar elementos: SÍ
                └── Reasignar numeros: NO
```

> **¡Importante!** `val` significa que no podemos reasignar la variable, pero no garantiza que el objeto referenciado sea inmutable.

---

## Comparación con Java

Como referencia para quien tiene experiencia con Java:

**Kotlin**
```kotlin
val nombre = "Ana"
```
Es una referencia de solo lectura.

**Java**
```java
final String nombre = "Ana";
```
La referencia se declara con `final`, por lo que no puede reasignarse.

Sin embargo, no son equivalentes en todos los aspectos: `val` no implica necesariamente la misma semántica de inmutabilidad profunda que una estructura inmutable. En Kotlin:

```kotlin
val lista = mutableListOf(1, 2, 3)
lista.add(4)
```

La lista sigue siendo modificable.

---

## ¿Qué ocurre con los arrays?

### Array con `val`

```kotlin
val numeros = arrayOf(1, 2, 3)
numeros[0] = 10
println(numeros.contentToString())
```

Salida:

```
[10, 2, 3]
```

Aunque el array se declare con `val`, sus elementos pueden modificarse. Pero no podemos reasignar el array:

```kotlin
val numeros = arrayOf(1, 2, 3)
numeros = arrayOf(4, 5, 6)   // ERROR
```

### Array con `var`

```kotlin
var numeros = arrayOf(1, 2, 3)
numeros[0] = 10
numeros = arrayOf(4, 5, 6)
println(numeros.contentToString())
```

Aquí sí podemos:

- Modificar los elementos.
- Reasignar el array completo.

---

## Regla práctica

**Utiliza `val` cuando:**

- El valor no necesita ser reasignado.
- Declaras datos de configuración.
- Almacenas resultados de cálculos.
- Guardas referencias que no necesitas cambiar.
- Defines parámetros o valores que permanecen constantes durante una operación.

```kotlin
val nombre = "Ana"
val edad = 20
val area = base * altura
```

**Emplea `var` cuando:**

- Necesitas reasignar el valor.
- Trabajas con contadores.
- Actualizas estados.
- Modificas valores durante un proceso.

```kotlin
var contador = 0
var mensaje = ""
var usuarioActivo = false
```

> **Recomendación para programar en Kotlin:** empieza declarando las variables con `val`. Utiliza `var` únicamente cuando necesites reasignar su valor. Esto favorece un código más predecible y facilita el mantenimiento.

---

## Ejercicio

¿Qué sucede en cada uno de estos casos?

```kotlin
val a = 10
a = 20

var b = 10
b = 20

val lista = mutableListOf(1, 2, 3)
lista.add(4)
```

### Soluciones

1. Error de compilación: no se puede reasignar un `val`.
2. Correcto: `var` permite reasignar.
3. Correcto: `val` permite modificar el objeto mutable.

---

## Referencias

- <https://kotlinlang.org/docs/home.html>
