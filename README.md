Tip Time - Solution Code
=================================

Solution code for the [Android Basics with Compose](https://developer.android.com/courses/android-basics-compose/course): Tip Time app.


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

Partimos de la app del codelab anterior (propina fija al 15 %) y la ampliamos para que:

- El usuario introduzca un **porcentaje de propina** personalizado.
- Se pueda **redondear** la propina hacia arriba con un `Switch`.
- Los campos tengan una **acción de teclado** (Siguiente / Hecho) e **iconos**.

### Antes de empezar

- El estado ya está *elevado* (*state hoisting*) a `TipTimeLayout`: `EditNumberField` es
  *stateless* y recibe `value` y `onValueChanged`.
- `calculateTip(amount, tipPercent = 15.0)` ya acepta un porcentaje, pero aún no se usa.
- Los iconos `money.xml` y `percent.xml` ya están en `res/drawable`.

### Paso 1 — Nuevos recursos de texto

En `res/values/strings.xml`:

```xml
<string name="how_was_the_service">Tip Percentage</string>
<string name="round_up_tip">Round up tip?</string>
```

### Paso 2 — Etiqueta como parámetro de `EditNumberField`

Para reutilizar el composable en varios campos, la etiqueta deja de estar fija y pasa a
ser un parámetro:

```kotlin
@Composable
fun EditNumberField(
    @StringRes label: Int,
    value: String,
    onValueChanged: (String) -> Unit,
    modifier: Modifier
) {
    TextField(
        label = { Text(stringResource(label)) },
        ...
    )
}
```

Y en la llamada: `label = R.string.bill_amount`.

> 💬 **Explicar:** `@StringRes` indica que el `Int` debe ser un id de recurso de texto
> (`R.string.*`); Android Studio avisa si pasamos otro número cualquiera.

### Paso 3 — Campo para el porcentaje de propina

Nuevo estado en `TipTimeLayout` y un segundo `EditNumberField`:

```kotlin
var tipInput by remember { mutableStateOf("") }

val amount = amountInput.toDoubleOrNull() ?: 0.0
val tipPercent = tipInput.toDoubleOrNull() ?: 0.0

val tip = calculateTip(amount, tipPercent)
```

```kotlin
EditNumberField(
    label = R.string.how_was_the_service,
    value = tipInput,
    onValueChanged = { tipInput = it },
    modifier = Modifier.padding(bottom = 32.dp).fillMaxWidth()
)
```

> 💬 **Explicar:** aquí se ve la ventaja del *state hoisting*: el mismo composable sirve
> para dos campos distintos porque el estado vive en el padre.

### Paso 4 — Acción del teclado (IME)

`EditNumberField` recibe ahora `keyboardOptions` como parámetro:

```kotlin
@Composable
fun EditNumberField(
    @StringRes label: Int,
    keyboardOptions: KeyboardOptions,
    ...
) {
    TextField(
        ...
        keyboardOptions = keyboardOptions
    )
}
```

En el primer campo el botón del teclado será **Siguiente** y en el segundo **Hecho**:

```kotlin
keyboardOptions = KeyboardOptions.Default.copy(
    keyboardType = KeyboardType.Number,
    imeAction = ImeAction.Next   // ImeAction.Done en el campo de porcentaje
),
```

> 💬 **Explicar:** `ImeAction.Next` mueve el foco al siguiente campo; `ImeAction.Done`
> cierra el teclado.

### Paso 5 — Redondear la propina con un `Switch`

Nuevo estado y nuevo composable `RoundTheTipRow`:

```kotlin
var roundUp by remember { mutableStateOf(false) }
...
val tip = calculateTip(amount, roundUp = roundUp, tipPercent = tipPercent)
...
RoundTheTipRow(
    roundUp = roundUp,
    onRoundUpChanged = { roundUp = it },
    modifier = Modifier.padding(bottom = 32.dp)
)
```

```kotlin
@Composable
fun RoundTheTipRow(
    roundUp: Boolean,
    onRoundUpChanged: (Boolean) -> Unit,
    modifier: Modifier = Modifier
) {
    Row(
        modifier = modifier
            .fillMaxWidth()
            .size(48.dp),
        verticalAlignment = Alignment.CenterVertically
    ) {
        Text(text = stringResource(R.string.round_up_tip))
        Switch(
            checked = roundUp,
            onCheckedChange = onRoundUpChanged,
            modifier = Modifier
                .fillMaxWidth()
                .wrapContentWidth(Alignment.End),
        )
    }
}
```

Y `calculateTip` aplica el redondeo:

```kotlin
private fun calculateTip(
    amount: Double,
    roundUp: Boolean,
    tipPercent: Double = 15.0
): String {
    var tip = tipPercent / 100 * amount
    if (roundUp) {
        tip = kotlin.math.ceil(tip)
    }
    return NumberFormat.getCurrencyInstance().format(tip)
}
```

> 💬 **Explicar:** `RoundTheTipRow` sigue el mismo patrón que `EditNumberField`
> (estado baja, eventos suben). `fillMaxWidth()` + `wrapContentWidth(Alignment.End)`
> empuja el `Switch` al extremo derecho. `ceil` redondea hacia arriba.

### Paso 6 — Iconos en los campos

Nuevo parámetro `leadingIcon` en `EditNumberField`:

```kotlin
@Composable
fun EditNumberField(
    @StringRes label: Int,
    @DrawableRes leadingIcon: Int,
    ...
) {
    TextField(
        ...
        leadingIcon = {
            Icon(
                painter = painterResource(leadingIcon),
                contentDescription = null
            )
        },
        ...
    )
}
```

En las llamadas: `leadingIcon = R.drawable.money` y `leadingIcon = R.drawable.percent`.

> 💬 **Explicar:** `contentDescription = null` porque el icono es decorativo; la etiqueta
> del campo ya describe su contenido para los lectores de pantalla.
