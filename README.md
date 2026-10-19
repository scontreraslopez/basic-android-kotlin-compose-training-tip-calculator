Tip Time - Solution Code
=================================

Starter code for the [Android Basics with Compose](https://developer.android.com/courses/android-basics-compose/course): Tip Time app.


Introduction
------------
The Tip Time app contains various UI elements for calculating a tip,
teaching about user input, and State in Compose.


Pre-requisites
--------------
* Experience with Kotlin syntax.
* How to create and run a project in Android Studio.


Getting Started
---------------
1. Install Android Studio, if you don't already have it.
2. Download the sample.
3. Import the sample into Android Studio.
4. Build and run the sample.

Guía de la clase (castellano)
----------------------------

### Objetivo

Al terminar, la app mostrará una propina sugerida a partir del importe de la cuenta que
introduzca el usuario. Por ahora el porcentaje está fijo al **15 %**; en el siguiente
codelab se añadirá un porcentaje personalizable.

### Antes de empezar

- Los textos están en el archivo de recursos `res/values/strings.xml` y se usan con `stringResource(...)`.
- La app usa un tema propio (`TipTimeTheme`); por ejemplo, el texto de la propina usa
  `MaterialTheme.typography.displaySmall`.

### Paso 1 — Añadir un `TextField`

En `EditNumberField()` (debajo de `TipTimeLayout()`), añadir un `TextField` con `value`
a cadena vacía y `onValueChange` a una lambda vacía:

```kotlin
@Composable
fun EditNumberField(modifier: Modifier = Modifier) {
    TextField(
        value = "",
        onValueChange = {},
        modifier = modifier
    )
}
```

> 💬 **Explicar:** `value` es lo que se *muestra*; `onValueChange` es el callback que se
> llama cuando el usuario escribe. Si no actualizamos `value`, el campo no cambia.

### Paso 2 — Scroll vertical en el `Column`

```kotlin
Column(
    modifier = Modifier
        .statusBarsPadding()
        .padding(horizontal = 40.dp)
        .verticalScroll(rememberScrollState())
        .safeDrawingPadding(),
    ...
)
```

> 💬 **Explicar:** permite desplazar el contenido cuando no cabe (p. ej. con el teclado
> abierto o en horizontal). `rememberScrollState()` recuerda la posición del scroll.

### Paso 3 — Llamar a `EditNumberField` desde `TipTimeLayout`

```kotlin
EditNumberField(
    modifier = Modifier
        .padding(bottom = 32.dp)
        .fillMaxWidth()
)
```

### Paso 4 — Estado: hacer que el campo sea editable

Primera aproximación:

```kotlin
var amountInput: MutableState<String> = mutableStateOf("0")
```

> 💬 **Explicar:** Compose solo *recompone* (redibuja) cuando cambia un `State<T>` /
> `MutableState<T>` que se está leyendo. Sin estado observable, no repinta.
>
> Pero así el valor se pierde en cada recomposición → hace falta `remember`.

Versión correcta:

```kotlin
var amountInput by remember { mutableStateOf("") }
```

> 💬 **Explicar:** `remember` conserva el valor entre recomposiciones, pero **no** ante
> cambios de configuración (girar la pantalla). Para eso existe `rememberSaveable`.

### Paso 5 — Mejorar el `TextField`

```kotlin
TextField(
    ...
    singleLine = true,
    label = { Text(stringResource(R.string.bill_amount)) },
    keyboardOptions = KeyboardOptions(keyboardType = KeyboardType.Number)
)
```

- `singleLine`: una sola línea, sin saltos.
- `label`: etiqueta flotante del campo.
- `keyboardOptions`: muestra el teclado numérico.

### Paso 6 — *State hoisting* al padre `TipTimeLayout`

Subir el estado al padre y convertir `EditNumberField` en un composable *stateless*:

```kotlin
@Composable
fun TipTimeLayout() {
    var amountInput by remember { mutableStateOf("") }
    ...
    EditNumberField(
        value = amountInput,
        onValueChange = { amountInput = it },
        modifier = ...
    )
}

@Composable
fun EditNumberField(
    value: String,
    onValueChange: (String) -> Unit,
    modifier: Modifier = Modifier
) { ... }
```

> 💬 **Explicar:** el padre es quien necesita el valor para calcular la propina; el hijo
> solo lo muestra y avisa de los cambios (estado baja, eventos suben).

### Paso 7 — Calcular la propina

`amountInput` es un `String`, así que hay que convertirlo a número (con valor por
defecto si no es válido):

```kotlin
val amount = amountInput.toDoubleOrNull() ?: 0.0
val tip = calculateTip(amount)
```

Y mostrarlo:

```kotlin
Text(
    text = stringResource(R.string.tip_amount, tip),
    style = MaterialTheme.typography.displaySmall
)
```

> 💬 **Explicar:** `toDoubleOrNull()` devuelve `null` si el texto no es un número, y el
> operador Elvis `?:` aporta el valor por defecto.