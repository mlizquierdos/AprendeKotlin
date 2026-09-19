# 10 · De Kotlin a Android

A partir de aquí cambiamos de herramienta: **Android Studio** en lugar de IntelliJ IDEA.

## 10.1 Primer proyecto

1. `New Project` → **Empty Activity** (plantilla con Jetpack Compose).
2. Nombre del paquete: `es.centro.dam.miapp`.
3. Minimum SDK: **API 24** (cubre en torno al 95 % de los dispositivos).
4. Esperad a que Gradle sincronice la primera vez (puede tardar varios minutos).

## 10.2 Dónde aparece cada cosa que habéis aprendido

| Concepto de Kotlin | Dónde lo veréis en Android |
|---|---|
| `data class` | Modelos de datos y estado de pantalla |
| Null safety | Vistas y datos que pueden no existir todavía |
| Lambdas finales | Todos los componentes de Compose |
| Funciones de extensión | Utilidades sobre `Context`, `View`, `String` |
| `sealed class` | Estado de la interfaz: cargando / datos / error |
| Corrutinas | Llamadas de red y base de datos |
| `object` | Repositorios y configuración global |

## 10.3 Una pantalla con Jetpack Compose

Compose describe la interfaz **en Kotlin**, no en XML. Una función `@Composable`
recibe datos y emite interfaz.

```kotlin
@Composable
fun Contador() {
    // remember + mutableStateOf: estado que sobrevive a los redibujados
    var cuenta by remember { mutableStateOf(0) }

    Column(
        modifier = Modifier.fillMaxSize().padding(16.dp),
        horizontalAlignment = Alignment.CenterHorizontally
    ) {
        Text(text = "Has pulsado $cuenta veces", fontSize = 20.sp)

        Button(onClick = { cuenta++ }) {    // lambda final: el contenido del botón
            Text("Pulsar")
        }
    }
}
```

Cuando `cuenta` cambia, Compose vuelve a ejecutar la función y redibuja sólo lo necesario.
Esto se llama **recomposición**, y es la razón de que los modelos sean inmutables:
el estado sólo cambia asignando un valor nuevo.

## 10.4 El patrón MVVM en dos pantallas

```
Vista (@Composable)  →  observa el estado  →  ViewModel  →  Repositorio  →  datos
        ↑                                           |
        └───────── nuevo estado (StateFlow) ────────┘
```

```kotlin
// Estado de la pantalla, modelado con la sealed class de la unidad 07
sealed class EstadoUi {
    object Cargando : EstadoUi()
    data class Datos(val alumnos: List<Alumno>) : EstadoUi()
    data class Error(val mensaje: String) : EstadoUi()
}

class AlumnosViewModel(private val repo: AlumnoRepository) : ViewModel() {

    private val _estado = MutableStateFlow<EstadoUi>(EstadoUi.Cargando)
    val estado: StateFlow<EstadoUi> = _estado      // sólo lectura hacia fuera

    fun cargar() {
        // viewModelScope se cancela solo cuando el ViewModel muere
        viewModelScope.launch {
            _estado.value = try {
                EstadoUi.Datos(repo.obtenerAlumnos())
            } catch (e: Exception) {
                EstadoUi.Error(e.message ?: "Error desconocido")
            }
        }
    }
}
```

```kotlin
@Composable
fun PantallaAlumnos(vm: AlumnosViewModel) {
    val estado by vm.estado.collectAsState()

    when (val e = estado) {
        is EstadoUi.Cargando -> CircularProgressIndicator()
        is EstadoUi.Datos    -> LazyColumn {
            items(e.alumnos) { alumno -> Text(alumno.nombre) }
        }
        is EstadoUi.Error    -> Text("Error: ${e.mensaje}")
    }
}
```

Reconoceréis aquí la misma separación por capas que trabajasteis con MVC y el patrón
Repository en 1º: la vista no sabe de dónde vienen los datos y el repositorio no sabe
cómo se pintan.

## 10.5 Estructura de paquetes recomendada

```
es.centro.dam.miapp/
├── data/
│   ├── model/          # data class
│   └── repository/     # acceso a datos
├── ui/
│   ├── screens/        # funciones @Composable
│   ├── components/     # piezas reutilizables
│   └── theme/          # colores y tipografía
└── viewmodel/
```

## Ejercicios

1. Crea el proyecto y ejecuta el contador en el emulador.
2. Añade un botón de reinicio que ponga el contador a cero.
3. Crea una pantalla que muestre una lista fija de alumnos con `LazyColumn`.
4. Extrae esa lista a un `object RepositorioAlumnos` y consúmela desde un `ViewModel`.

**Para profundizar:** investiga Room para persistencia local y Retrofit para consumo de APIs REST.
