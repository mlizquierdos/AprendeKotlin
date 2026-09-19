# 04 · Funciones y lambdas

## 4.1 Declaración

```kotlin
fun sumar(a: Int, b: Int): Int {
    return a + b
}

// Cuerpo de expresión: el tipo de retorno se infiere
fun sumar2(a: Int, b: Int) = a + b

// Sin retorno: el tipo es Unit (equivalente a void), y puede omitirse
fun saludar(nombre: String) {
    println("Hola, $nombre")
}
```

## 4.2 Parámetros por defecto y con nombre

```kotlin
fun crearUsuario(
    nombre: String,
    activo: Boolean = true,
    rol: String = "alumno"
) = "$nombre ($rol) activo=$activo"

crearUsuario("Ana")                       // usa los valores por defecto
crearUsuario("Luis", rol = "profesor")    // argumento con nombre: se salta 'activo'
```

Esto elimina la necesidad de la **sobrecarga de constructores** típica de Java.

## 4.3 Número variable de argumentos

```kotlin
fun media(vararg notas: Double): Double =
    if (notas.isEmpty()) 0.0 else notas.average()

println(media(7.0, 8.5, 6.0))
```

## 4.4 Lambdas

Una lambda es una función sin nombre que se puede guardar en una variable o pasar como argumento.

```kotlin
val doble: (Int) -> Int = { n -> n * 2 }
println(doble(5))               // 10

// Con un solo parámetro se puede usar el nombre implícito "it"
val triple: (Int) -> Int = { it * 3 }
```

Sintaxis del tipo función: `(TiposDeEntrada) -> TipoDeSalida`.

## 4.5 Funciones de orden superior

Son funciones que **reciben o devuelven** otras funciones.

```kotlin
fun operar(a: Int, b: Int, operacion: (Int, Int) -> Int): Int {
    return operacion(a, b)
}

println(operar(6, 3) { x, y -> x + y })    // 9
println(operar(6, 3) { x, y -> x * y })    // 18

// Convención: si la lambda es el ÚLTIMO parámetro, se saca fuera del paréntesis
```

Esta convención es la base de la sintaxis de Jetpack Compose que veréis en la unidad 10:

```kotlin
Button(onClick = { contador++ }) {
    Text("Pulsa aquí")     // esta lambda final es el contenido del botón
}
```

## 4.6 Referencias a funciones

```kotlin
fun esPar(n: Int) = n % 2 == 0

val numeros = listOf(1, 2, 3, 4, 5, 6)
println(numeros.filter(::esPar))     // [2, 4, 6]
```

## 4.7 Funciones locales

```kotlin
fun validarAlumno(nombre: String, email: String) {
    // función anidada: sólo tiene sentido dentro de esta validación
    fun validar(campo: String, valor: String) {
        if (valor.isBlank()) throw IllegalArgumentException("$campo vacío")
    }
    validar("nombre", nombre)
    validar("email", email)
}
```

## Ejercicios

1. Función `aplicarDescuento(precio: Double, porcentaje: Double = 10.0): Double`.
2. Función de orden superior `repetir(veces: Int, accion: (Int) -> Unit)` que ejecute `accion` pasándole el número de iteración.
3. Crea una calculadora que guarde las operaciones en un `Map<String, (Double, Double) -> Double>` y permita al usuario elegir por consola.
4. Reescribe con lambdas un `Comparator` anónimo de Java que hayas usado en 1º.

**Para profundizar:** lee sobre `inline fun` y por qué las funciones de orden superior de la librería estándar están marcadas como `inline`.
