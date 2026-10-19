# 01 · Entorno y primer programa

> **Convención de los ejemplos:** los ejemplos completos incluyen su propia función `main()` y van seguidos de su **Salida**. Las funciones y clases se declaran **fuera** de `main()`.

## 1.1 ¿Por qué Kotlin?

Kotlin es, desde 2019, el lenguaje **preferente para Android** según Google. Frente a Java aporta:

- **Seguridad frente a nulos** integrada en el sistema de tipos (menos `NullPointerException`).
- **Menos código repetitivo**: una `data class` sustituye a 60 líneas de POJO.
- **Interoperabilidad total** con Java: podéis usar cualquier librería Java desde Kotlin.
- **Corrutinas** para programación asíncrona legible.

Todo lo que sabéis de POO en Java sigue siendo válido. Cambia la sintaxis y aparecen ideas nuevas
(inmutabilidad por defecto, funciones fuera de clases, expresiones en vez de sentencias).

## 1.2 Instalación

1. Instalad el **JDK 21 LTS** (o 17). Comprobadlo en un terminal:
   ```bash
   java -version
   ```
2. Instalad **IntelliJ IDEA Community Edition**.
3. `File → New → Project` → seleccionad **Kotlin**, build system **Gradle**, JDK 21.

> El compilador de Kotlin viene incluido en IntelliJ: no hay que instalarlo aparte.

### Un proyecto, un módulo

Para las prácticas del módulo basta con un proyecto de **un solo módulo**. Si al crear el proyecto aparecen carpetas como `app/`, `utils/` o `buildSrc/`, se ha elegido una plantilla de varios módulos: cread uno nuevo. Esas carpetas añaden código de ejemplo que no es vuestro y complican la entrega.

Si al crear el proyecto se marca la opción de código de ejemplo, el `Main.kt` generado incluye un comentario como este:

```kotlin
//TIP To <b>Run</b> code, press <shortcut actionId="Run"/> or
// click the <icon src="AllIcons.Actions.Execute"/> icon in the gutter.
```

Es un aviso del IDE, no parte de vuestra solución: **borradlo** antes de entregar.

## 1.3 Estructura de un proyecto Gradle

```
mi-proyecto/
├── build.gradle.kts        # configuración del proyecto (escrita en Kotlin)
├── settings.gradle.kts     # nombre del proyecto y módulos
├── gradlew, gradlew.bat    # permiten compilar sin tener Gradle instalado
├── gradle/                 # versión de Gradle que usa el proyecto
└── src/
    ├── main/kotlin/        # código fuente
    └── test/kotlin/        # pruebas unitarias
```

IntelliJ crea además carpetas que **no son código** y que se regeneran solas al abrir o compilar el proyecto:

| Carpeta | Qué contiene |
|---|---|
| `.idea/` | Configuración del IDE en vuestro equipo |
| `.gradle/` | Caché de Gradle |
| `.kotlin/` | Datos temporales del compilador de Kotlin |
| `build/` | Ficheros compilados |

El código se organiza en `src/main/kotlin/`. Con varias clases, lo habitual es un fichero por clase (`Libro.kt`, `Revista.kt`) y otro con la función `main()` (`Main.kt`).

## 1.4 Hola mundo

Cread el fichero `src/main/kotlin/Main.kt`:

```kotlin
// En Kotlin la función main NO necesita estar dentro de una clase.
// El compilador genera por debajo una clase MainKt con un método static main.
fun main() {
    println("Hola, DAM 2º")
}
```

Salida:

```text
Hola, DAM 2º
```

Para ejecutarlo, pulsad el triángulo verde que aparece junto a `fun main()`.

Comparad con Java:

```java
public class Main {
    public static void main(String[] args) {
        System.out.println("Hola, DAM 2º");
    }
}
```

Diferencias visibles ya en cuatro líneas:

| Java | Kotlin |
|---|---|
| Clase contenedora obligatoria | Funciones de nivel superior |
| `System.out.println` | `println` (de la librería estándar) |
| `;` obligatorio | `;` innecesario |
| `String[] args` obligatorio | Parámetro opcional |

### Dónde va cada cosa en un fichero

Las clases y las funciones se declaran **a nivel de fichero**, fuera de `main()`. Dentro de `main()` solo se crean los objetos y se usan:

```kotlin
class Saludo(val nombre: String) {           // fuera de main
    fun texto() = "Hola, $nombre"
}

fun main() {
    val s = Saludo("Ana")                     // dentro de main: crear y usar
    println(s.texto())
}
```

Salida:

```text
Hola, Ana
```

## 1.5 Entrada por consola

```kotlin
fun main() {
    print("¿Cómo te llamas? ")
    val nombre = readlnOrNull() ?: "invitado"   // readlnOrNull devuelve String?
    println("Bienvenida/o, $nombre")            // plantilla de cadena
}
```

Ejemplo de ejecución (en negrita lo que escribe el usuario):

<pre>
¿Cómo te llamas? <b>Lucía</b>
Bienvenida/o, Lucía
</pre>

- `readlnOrNull()` devuelve `null` si no hay entrada (fin de fichero). Volveremos a `?:` en la unidad 03.
- `"$nombre"` es una **plantilla de cadena**; con expresiones se usa `${...}`.

**Todo lo que se lee por consola es texto.** Para usarlo como número hay que convertirlo, y el usuario puede escribir algo que no lo sea. La forma segura es `toIntOrNull()` o `toDoubleOrNull()`, que devuelven `null` en lugar de cerrar el programa:

```kotlin
fun main() {
    print("¿Cuántos años tienes? ")
    val edad = readlnOrNull()?.toIntOrNull()
    if (edad == null) println("Eso no es un número")
    else println("Dentro de 10 años tendrás ${edad + 10}")
}
```

Ejemplos de ejecución:

<pre>
¿Cuántos años tienes? <b>17</b>
Dentro de 10 años tendrás 27
</pre>

<pre>
¿Cuántos años tienes? <b>diecisiete</b>
Eso no es un número
</pre>

Lo veréis en detalle en la unidad 03, y en la 04 tenéis una función que vuelve a preguntar hasta recibir un número válido.

## 1.6 Leer los errores del compilador

Si el código no compila, IntelliJ subraya en rojo la línea y la pestaña **Build** muestra los errores con este formato:

```text
Main.kt:8:41: error: unresolved reference 'titulo'.
```

| Parte | Significado |
|---|---|
| `Main.kt` | Fichero |
| `8:41` | Línea 8, columna 41 |
| `unresolved reference 'titulo'` | Qué ocurre: se usa un nombre que no existe en ese punto |

Corregid **el primer error** de la lista y volved a compilar: a menudo los siguientes son consecuencia del primero y desaparecen solos.

**Un proyecto que no compila no se puede ejecutar ni comprobar.** Antes de entregar cualquier práctica, compilad el proyecto completo (`Build → Build Project`) y aseguraos de que no hay ningún error.

## 1.7 Cómo entregar un proyecto

Las prácticas y pruebas se entregan como un **ZIP del proyecto completo**, con el nombre que indique el enunciado (por ejemplo, `Apellido_Nombre_Prueba01_1P.zip`).

1. **Comprobad que compila y se ejecuta** (apartado 1.6).
2. **Borrad lo que no es vuestro:** el comentario `//TIP` de la plantilla, código comentado e `import` sin usar.
3. **Comprimid la carpeta raíz del proyecto**, la que contiene `build.gradle.kts` y `src/`. No basta con el fichero `Main.kt`: sin el resto del proyecto no se puede abrir ni compilar.
4. No hace falta incluir `.idea/`, `.gradle/`, `.kotlin/` ni `build/`: se regeneran solas y aumentan mucho el tamaño.
5. **Probad el ZIP:** descomprimidlo en otra carpeta, abridlo con IntelliJ (`File → Open`), compiladlo y ejecutadlo. Si funciona ahí, funcionará al corregirlo.

Contenido esperado del ZIP:

```
Apellido_Nombre_Prueba01_1P/
├── build.gradle.kts
├── settings.gradle.kts
├── gradlew, gradlew.bat
├── gradle/
└── src/
    └── main/kotlin/
        ├── Main.kt
        └── ... (resto de ficheros .kt)
```

> Si usáis Git, el fichero `.gitignore` que crea IntelliJ ya excluye las carpetas regenerables del repositorio.

## 1.8 Errores típicos

| Error | Qué ocurre | Solución |
|---|---|---|
| Entregar solo `Main.kt` | No se puede abrir ni compilar como proyecto | Comprimir la carpeta raíz del proyecto |
| Proyecto con `app/`, `utils/` y `buildSrc/` | Código de ejemplo ajeno y estructura más compleja | Crear un proyecto de un solo módulo |
| Entregar sin compilar | Errores que impiden ejecutar nada | `Build → Build Project` antes de comprimir |
| Dejar el comentario `//TIP` de la plantilla | Código sin limpiar | Borrarlo |
| Declarar clases dentro de `main()` | Código difícil de leer y de reutilizar | Clases a nivel de fichero |
| Convertir lo que escribe el usuario con `toInt()` | Una letra cierra el programa | `toIntOrNull()` |
| Corregir los errores del compilador empezando por el último | Se pierde tiempo con errores derivados | Corregir el primero y recompilar |

## Ejercicios

1. Crea un proyecto Gradle nuevo llamado `kotlin-fundamentos` y ejecuta el «hola mundo».
2. Escribe un programa que pida nombre y edad y muestre: `Hola Ana, dentro de 10 años tendrás 27.` Si la edad no es un número, debe mostrar un aviso en lugar de cerrarse.
3. Explica por escrito (3–4 líneas) qué genera Kotlin cuando compila una función de nivel superior.
4. **Lee el error.** Para cada mensaje, indica en qué línea está el problema y qué crees que lo causa:
   ```text
   Main.kt:3:5: error: unresolved reference 'printn'.
   Main.kt:7:5: error: 'val' cannot be reassigned.
   ```
5. **Simulacro de entrega.** Comprime el proyecto del ejercicio 2 siguiendo el apartado 1.7, descomprímelo en otra carpeta, ábrelo con IntelliJ y comprueba que compila y se ejecuta. Anota qué carpetas has dejado fuera del ZIP.

**Para profundizar:** abre `Tools → Kotlin → Show Kotlin Bytecode → Decompile` y observa el Java equivalente de tu `Main.kt`.
