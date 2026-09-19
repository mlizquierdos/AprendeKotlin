# 01 · Entorno y primer programa

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

## 1.3 Estructura de un proyecto Gradle

```
mi-proyecto/
├── build.gradle.kts        # configuración del proyecto (escrita en Kotlin)
├── settings.gradle.kts     # nombre del proyecto y módulos
└── src/
    ├── main/kotlin/        # código fuente
    └── test/kotlin/        # pruebas unitarias
```

## 1.4 Hola mundo

Cread el fichero `src/main/kotlin/Main.kt`:

```kotlin
// En Kotlin la función main NO necesita estar dentro de una clase.
// El compilador genera por debajo una clase MainKt con un método static main.
fun main() {
    println("Hola, DAM 2º")
}
```

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

## 1.5 Entrada por consola

```kotlin
fun main() {
    print("¿Cómo te llamas? ")
    val nombre = readlnOrNull() ?: "invitado"   // readlnOrNull devuelve String?
    println("Bienvenida/o, $nombre")            // plantilla de cadena
}
```

- `readlnOrNull()` devuelve `null` si no hay entrada (fin de fichero). Volveremos a `?:` en la unidad 03.
- `"$nombre"` es una **plantilla de cadena**; con expresiones se usa `${...}`.

## Ejercicios

1. Crea un proyecto Gradle nuevo llamado `kotlin-fundamentos` y ejecuta el «hola mundo».
2. Escribe un programa que pida nombre y edad y muestre: `Hola Ana, dentro de 10 años tendrás 27.`
3. Explica por escrito (3–4 líneas) qué genera Kotlin cuando compila una función de nivel superior.

**Para profundizar:** abre `Tools → Kotlin → Show Kotlin Bytecode → Decompile` y observa el Java equivalente de tu `Main.kt`.
