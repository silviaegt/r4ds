
# Funciones

## Introducción

Una de las mejores maneras de lograr tener mayor alcance como científica de datos es escribir funciones. Las funciones te permitirán automatizar algunas tareas comunes en una manera poderosa y general que copiar-y-pegar. Escribir funciones tiene 3 grandes ventajas sobre copiar-y-pegar:

1. Puedes dar a la función un nombre evocativo que hace to código más fácil de entender.

1. Como cambien los requerimientos, tu solo necesitas cambiar tu código en un solo lugar, en vez de varios lugares.

1. Tu eliminas las chances de hacer errores accidentales cuando copias y pegas (ej. actualizar el nombre de una variable en un lugar, pero no en otro).

Escribir funciones en un viaje de toda la vida. Incluso después de usar R por varios años yo aún aprendo nuevas técnicas y mejores formas de abordar viejos problemas. El objetivo de este capítulo no es enseñarte cada detalle esotérico de funciones pero sí introducirte con consejos pragmáticos que puedes aplicar inmediatamente.

Como así también en los avisos práctivos para escribir funciones, en este capítulo también te da consejos para cómo diseñar tu código. El buen diseño del código es la correcta puntuación. Tu puedes hacerlo sin eso, pero te hará las cosas más fácil de leer! Al igual que los estilos de puntuación, hay muchas posibles variaciones. Aquí presentamos el diseño que usamos en nuestro código, pero lo más importante es ser consistente.

### Prerequisitos

El objetivo de este capítulo es escribir funciones en R base, así no
necesitarás paquetes extra.

## ¿Cuándo deberías escribir una función?

Debes considerar escribir una función cuando has copiado y pegado un bloque de código más de dos veces (ej. ahora tienes tres copias del mismo código). Por ejemplo, mira a este código. Qué es lo que realiza?


```r
df <- tibble::tibble(
 a = rnorm(10),
 b = rnorm(10),
 c = rnorm(10),
 d = rnorm(10)
)

df$a <- (df$a - min(df$a, na.rm = TRUE)) /
 (max(df$a, na.rm = TRUE) - min(df$a, na.rm = TRUE))
df$b <- (df$b - min(df$b, na.rm = TRUE)) /
 (max(df$b, na.rm = TRUE) - min(df$a, na.rm = TRUE))
df$c <- (df$c - min(df$c, na.rm = TRUE)) /
 (max(df$c, na.rm = TRUE) - min(df$c, na.rm = TRUE))
df$d <- (df$d - min(df$d, na.rm = TRUE)) /
 (max(df$d, na.rm = TRUE) - min(df$d, na.rm = TRUE))
```

Es posible que puedas descifrar que esto reescala cada columna para tener un rango de 0 a 1. ¿Pero has visto el error? He cometido un error copiando-y-pegando el código para `df$b`: He olvidado de cambiar `a` a `b`. Extraer código repetido en una función es una buena idea porque te previene de cometer errores como este:

Para escribir una función primero necesitas analizar el código. Cuantos inputs tiene?



```r
(df$a - min(df$a, na.rm = TRUE)) /
 (max(df$a, na.rm = TRUE) - min(df$a, na.rm = TRUE))
```

Este código tiene un solo input `df$a`. (Si te sorprende que `TRUE` no es un input, puedes explorar el ejercicio de abajo). Para hacer los inputs mas claros, es buena idea reescribir el código usando variables temporadles con nombres generales. Aca el código solamente requiere un solo vector numérico, por lo que lo llamaré `x`:


```r
x <- df$a
(x - min(x, na.rm = TRUE)) / (max(x, na.rm = TRUE) - min(x, na.rm = TRUE))
#>  [1] 0.289 0.751 0.000 0.678 0.853 1.000 0.172 0.611 0.612 0.601
```

Hay algo de duplicación en este código. Estamos computando el rango de datos tres veces, así que tiene razón hacerlo en un solo paso:


```r
rng <- range(x, na.rm = TRUE)
(x - rng[1]) / (rng[2] - rng[1])
#>  [1] 0.289 0.751 0.000 0.678 0.853 1.000 0.172 0.611 0.612 0.601
```

Sacando cálculos intermedios en variables nombradas es una buena práctica porque deja más claro lo que está haciendo el código. Ahora que has simplificado el código, y chequeado de que aún funciona, puedo convertirlo en una función:


```r
rescale01 <- function(x) {
 rng <- range(x, na.rm = TRUE)
 (x - rng[1]) / (rng[2] - rng[1])
}
rescale01(c(0, 5, 10))
#> [1] 0.0 0.5 1.0
```

Hay 3 pasos clasves para crear una funció nueva:

1. Necesitas elegir un __nombre__ para la función. Aquí he usado `rescale01` poque esta función reescala un vector para que se encuentre entre 0 y 1.

1. Listas los inputs, o __argumentos__, a la función dentro de `function`.
Aquí solo tenemos un argumento. Si tenemos más, la llamada se vería como `function(x, y, z)`.

1. Situas el código que has creado en __cuerpo__ de una función, un
`{` bloque lo que inmediatamente sigue a `function(...)`.

Ten en cuenta el proceso general: yo solo creo la función después de darme cuenta como funciona con una entrada simple. Es más fácil de empezar con codigo funcionando y convertirlo en una función; es más dificil de crear una función y luego intentar de hacerlo trabajar.

En este punto es una buena idea controlar tu función con algunos inputs diferentes:


```r
rescale01(c(-10, 0, 10))
#> [1] 0.0 0.5 1.0
rescale01(c(1, 2, 3, NA, 5))
#> [1] 0.00 0.25 0.50   NA 1.00
```

A medida que escribas más y más funciones eventualmente querras convertir estos informales, tests interactivos en tests formales y automatizados. Este proceso se llama examen de la unidad. Desafortunadamente, esto está más allá del alcance de este libro, pero puede aprender sobre eso en <http://r-pkgs.had.co.nz/tests.html>.

Podemos simplificar el ejemplo original ahora que tenemos una función:


```r
df$a <- rescale01(df$a)
df$b <- rescale01(df$b)
df$c <- rescale01(df$c)
df$d <- rescale01(df$d)
```

Comparado al original, este código es fácil de entender y hemos eliminado erorres del tipo copiar-y-pegar-. Esto es aún un poco de duplicación ya que estamos relizando lo mismo a diferentes columnas. Aprenderemos como eliminar esta duplicación en [iteración], una vez que hayas aprendido más sobre las estructuras de R en [vectores].


Otra ventaja de las funcioens es que nuestros requerimientos camiban, solo necesitamos hacer cambios en un solo lugar. Por ejemplo, podríamos descubrir que algunas de nuestras variables incluyen valores infinitos y `rescale01()` falla:



```r
x <- c(1:10, Inf)
rescale01(x)
#>  [1]   0   0   0   0   0   0   0   0   0   0 NaN
```

Porque hemos extraído el código en una función, nosotros solo necesitamos corregirlo en un solo lugar:


```r
rescale01 <- function(x) {
 rng <- range(x, na.rm = TRUE, finite = TRUE)
 (x - rng[1]) / (rng[2] - rng[1])
}
rescale01(x)
#>  [1] 0.000 0.111 0.222 0.333 0.444 0.556 0.667 0.778 0.889 1.000   Inf
```

Esta es una importante parte de "no repetirse a uno mismo" (o DRY) principio. Cuanto más repetición tengas en tu código, la mayor parte de lugares que neceistas recordar de actualizar cuando las cosas cambian (¡y esto siempre sucede!), y es más probable que crees errores (bugs) a lo largo del tiempo.

### Práctica

1. ¿Por qué `TRUE` no es un parámetro para `rescale01()`? ¿Qué pasaría si `x` está contenido en un valor único perdido y `na.rm` fuese `FALSE`?

1. En la segunda variante de `rescale01()`, los valores infinitos se dejan sin cambio. Reescribe `rescale01()` así `-Inf` is convertido a 0, y `Inf` es convertido a 1.

1. Practica convertir los siguientes fragmentos de código en funciones. Piensa en lo que hace cada función. ¿Cómo lo llamarías? ¿Cuántos argumentos necesita? ¿Puedes reescribirlo para ser más expresivo o menos duplicado?

 
 ```r
 mean(is.na(x))
 
 x / sum(x, na.rm = TRUE)
 
 sd(x, na.rm = TRUE) / mean(x, na.rm = TRUE)
 ```

1. Sigue <http://nicercode.github.io/intro/writing-functions.html>
para escribir tus propias funciones para computar la variancia y el sesgo de un vector numérico.

1. Escribe `both_na()`, una función que toma dos vectores de la misma longitud y retorna el número de posiciones que tiene `NA` en ambos vectores.

1. ¿Qué hacen las siguientes funciones? ¿Por qué son tan útiles aún cuando son tan cortas?

 
 ```r
 is_directory <- function(x) file.info(x)$isdir
 is_readable <- function(x) file.access(x, 4) == 0
 ```

1. Lee el [complete lyrics](https://en.wikipedia.org/wiki/Little_Bunny_Foo_Foo)
 de "Pequeño conejito Foo Foo". There's a lot of duplication in this song.
 Extiende el ejemplo inicial de pipes para recrear la canción completa, usar las funciones para reducir la duplicación.


## Las funciones son para los humanos y las computadoras

Es importante recordar que las funciones no son solo para las computadoras, sino también para los seres humanos. A R no le importa el nombre de tu función ni los comentarios que tiene, pero si serán importantes para los humanos que la lean. En esta sección se discutirán algunas cosas que debes tener en mente a la hora de escribir funciones entendibles para los humanos.

The name of a function is important. Ideally, the name of your function will be short, but clearly evoke what the function does. That's hard! But it's better to be clear than short, as RStudio's autocomplete makes it easy to type long names.

Generalmente, los nombres de las funciones deberían ser verbos, y los argumentos sustantivos. Hay algunas excepciones: usar un sustantivo es correcto si la función computa el valor de un sustantivo muy conocido (ejemplo: mean() — (del inglés media) es mejor que compute_mean()— (del inglés computar media)), o accede a alguna propiedad del objeto (ejemplo: coef()— (abreviatura del inglés coeficientes) es mejor que get_coefficients()— (del inglés obtener coeficientes) ). Una buena señal de que un sustantivo puede ser una mejor elección es analizar si estás usando un verbo muy amplio como “obtener”, “computar”, “calcular” o “determinar”. Utiliza tu mejor criterio y no tengas miedo de renombrar tu función si encuentras un nombre mejor más tarde.


```r
# Too short
f()

# Not a verb, or descriptive
my_awesome_function()

# Long, but clear
impute_missing()
collapse_years()
```

Si el nombre de tu función está compuesto por múltiples palabras, recomiendo usar “caso\_serpiente”, donde cada palabra en minúscula está separada por un guión bajo. Otra alternativa popular es caso Camello. No importa realmente cual elijas, lo importante es que seas consistente: elije una o la otra y quédate con ella. R mismo no es muy consistente, pero no hay nada que tú puedas hacer con respecto a eso. Asegúrate de no caer en la misma trampa haciendo tu código lo más consistente posible.


```r
# Never do this!
col_mins <- function(x, y) {}
rowMaxes <- function(y, x) {}
```

Si tienes una familia de funciones que hacen cosas similares, asegúrate de que tengan nombres y argumentos consistentes. Usa un prefijo común para indicar que están conectadas. Eso es mejor que usar un sufijo común, ya que el predictivo te permite escribir el prefijo y ver todos los otros miembros de la familia.


```r
# Good
input_select()
input_checkbox()
input_text()

# Not so good
select_input()
checkbox_input()
text_input()
```

Un buen ejemplo de este diseño es el paquete stringr: si no recuerdas exactamente que función necesitas, puedes escribir `str_`y refrescar tu memoria.
Siempre que sea posible, evita anular funciones y variables pre-existentes. Es imposible hacer esto en general, ya que hay un montón de nombres buenos que ya han sido utilizados por otros paquetes. De todas maneras, evitar el uso de los nombres más comunes de R base te ahorrará confusiones.


```r
# ¡No hagas esto!
T <- FALSE
c <- 10
mean <- function(x) sum(x)
```

Usa comentarios, líneas que comienzan con `#`, para explicar el “porqué” de tu código. En general deberías evitar comentarios que expliquen el “qué” y el “cómo”. Si no se entiende que es lo que hace el código leyéndolo, deberías pensar cómo reescribirlo de manera que sea más claro. ¿Necesitamos agregar algunas variables intermedias con nombres útiles? ¿Deberíamos dividir una función larga en subcomponentes para que pueda ser nombrada?. Sin embargo, tu código nunca podrá capturar la razón detrás de tus decisiones: ¿Por qué elegiste este enfoque frente a otras alternativas?, ¿Qué otra cosa probaste que no funcionó?. Es una gran idea capturar este tipo de pensamientos en un comentario.

Otro uso importante de los comentarios es para dividir tu archivo en partes de modo que resulte más fácil de leer. Utiliza líneas largas de`-` y `=` para que resulte más fácil detectar los fragmentos.



```r
# Cargar los datos --------------------------------------

# Graficar los datos --------------------------------------
```

RStudio proporciona un abreviado de teclado para crear estos encabezados (Cmd/Ctrl + Shift + R), y te los muestra en el menú desplegable de navegación de código en la parte inferior izquierda del editor:

<img src="screenshots/rstudio-nav.png" width="125" style="display: block; margin: auto;" />

### Ejercicios

1. Lee el código fuente para cada una de las siguientes tres funciones, interpreta que hacen, y luego propone nombres mejores.

 
 ```r
 f1 <- function(string, prefix) {
 substr(string, 1, nchar(prefix)) == prefix
 }
 f2 <- function(x) {
 if (length(x) <= 1) return(NULL)
 x[-length(x)]
 }
 f3 <- function(x, y) {
 rep(y, length.out = length(x))
 }
 ```

1. Toma una función que hayas escrito recientemente y tómate 5 minutos para pensar un mejor nombre para la función y para sus argumentos.

1. Compara y contrasta `rnorm()` y `MASS::mvrnorm()`. ¿Cómo podrías hacerlas más consistentes?

1. Argumenta porqué `norm_r()`,`norm_d()` etc sería una mejor opción que `rnorm()`, `dnorm()`. Argumenta lo contrario.

## Ejecución condicional

Una sentencia `if`le permite ejecutar un código condicional. Por ejemplo:


```r
if (condition) {
 # code executed when condition is TRUE
} else {
 # code executed when condition is FALSE
}
```

Para obtener ayuda en `if` necesitas ponerlo entre comillas simples: `` ?`if` ``. La ayuda no es especialmente útil si aún no eres un programador experimentado ¡Pero al menos puedes saber cómo llegar a ella!

Aquí se presenta una función simple que utiliza una sentencia `if`. El objetivo de esta función es devolver un vector lógico que describa si se nombra o no cada elemento de un vector.


```r
has_name <- function(x) {
 nms <- names(x)
 if (is.null(nms)) {
 rep(FALSE, length(x))
 } else {
 !is.na(nms) & nms != ""
 }
}
```

Esta función aprovecha la regla de retorno estándar: una función devuelve el último valor que calculó. Este es uno de los dos usos de la declaración `if`.

### Condiciones

La condición debe evaluar a `TRUE` o `FALSE`. Si es un vector, recibirás un mensaje de advertencia; si es una `NA`, obtendrás un error. Tenga cuidado con estos mensajes en su propio código:


```r
if (c(TRUE, FALSE)) {}
#> Warning in if (c(TRUE, FALSE)) {: the condition has length > 1 and only the
#> first element will be used
#> NULL

if (NA) {}
#> Error in if (NA) {: missing value where TRUE/FALSE needed
```

Puedes usar `||` (o) y `&&`(y) para combinar múltiples expresiones lógicas. Estos operadores están "haciendo cortocircuito": tan pronto como `||` ve el primer `TRUE` devuelve `TRUE` sin calcular nada más. Tan pronto como && vea el primer `FALSE`, devuelve `FALSE`. Nunca debes usar`|`o `&` en una sentencia `if`: estas son operaciones vectorizadas que se aplican a valores múltiples (es por eso que las usas en `filter()`). Si tienes un vector lógico, puede usar `any()` o `all()` para juntarlo en un único valor.

Be careful when testing for equality. `==` is vectorised, which means that it's easy to get more than one output. Either check the length is already 1, collapse with `all()` or `any()`, or use the non-vectorised `identical()`. `identical()` is very strict: it always returns either a single `TRUE` or a single `FALSE`, and doesn't coerce types. This means that you need to be careful when comparing integers and doubles:

Tenga cuidado al probar la igualdad.`==` está vectorizado, lo que significa que es fácil obtener más de una salida. Compruebe si la longitud ya es 1, colapse con `all()` o `any()`, o usa el no vectorizado `identical()`. `identical()` es muy estricto: siempre devuelve un solo `TRUE` o un solo `FALSE`, y no coacciona tipos. Esto significa que debe tener cuidado al comparar enteros y dobles:


```r
identical(0L, 0)
#> [1] FALSE
```

También hay que tener cuidado con los números de punto flotante:


```r
x <- sqrt(2) ^ 2
x
#> [1] 2
x == 2
#> [1] FALSE
x - 2
#> [1] 4.44e-16
```

En su lugar use `dplyr::near()` para comparaciones, como se describe en [comparisons].

Y recuerde, `x == NA` ¡No hace nada útil!

### Condiciones múltiples

Puede encadenar múltiples sentencias if juntas:


```r
if (this) {
 # do that
} else if (that) {
 # do something else
} else {
 #
}
```

Pero si terminas con una larga serie de sentencias `if` encadenadas, deberías considerar reescribir. Una técnica útil es la función `switch()` . Esta le permite evaluar el código seleccionado según la posición o el nombre.


```
#> function(x, y, op) {
#>  switch(op,
#>  plus = x + y,
#>  minus = x - y,
#>  times = x * y,
#>  divide = x / y,
#>  stop("Unknown op!")
#>  )
#> }
```

Otra función útil que a menudo puede eliminar largas cadenas de sentencias `if`es `cut()`. Esta es utilizada para discretizar variables continuas.

### Estilo de código

Ambas sentencias `if` y `function` deberían (casi) siempre ir entre llaves (`{}`), y el contenido debería estar seguido de dos espacios. Esto hace que sea más fácil ver la jerarquía en su código en el margen izquierdo.

La llave de apertura nunca debe ir en su propia línea y siempre debe ir seguida de una nueva línea. Una llave de cierre siempre debe ir en su propia línea, a menos que sea seguida por `else`. Siempre agregar espacio en el código dentro de las llaves.


```r
# Good
if (y < 0 && debug) {
 message("Y is negative")
}

if (y == 0) {
 log(x)
} else {
 y ^ x
}

# Bad
if (y < 0 && debug)
message("Y is negative")

if (y == 0) {
 log(x)
}
else {
 y ^ x
}
```

Estaría bien evitar las llaves si tienes una sentencia `if` muy corta que pueda entrar en una línea:


```r
y <- 10
x <- if (y < 20) "Too low" else "Too high"
```

Esto se recomienda solo para sentencias `if` muy breves. De lo contrario, la sentencia completa es más fácil de leer:


```r
if (y < 20) {
 x <- "Too low"
} else {
 x <- "Too high"
}
```

### Ejercicios

1. ¿Cuál es la diferencia entre `if` and `ifelse()`? Lea cuidadosamente la ayuda y construya tres ejemplos que ilustren las diferencias claves.


1. Escriba una función de saludo que diga "buenos días", "buenas tardes" o "buenas noches", según la hora del día. (Sugerencia: use un argumento de tiempo que por defecto es `lubridate::now()`, eso hará que sea más fácil probar su función).

1. Implemente una función `fizzbuzz`. Toma un solo número como entrada. Si el número es divisible por tres, devuelve "fizz". Si es divisible por cinco, devuelve "buzz". Si es divisible por tres y cinco, devuelve "fizzbuzz". De lo contrario, devuelve el número. Asegúrese de escribir primero el código de trabajo antes de crear la función.

1. ¿Cómo podría usar `cut()` para simplificar una sentencia if-else anidada?

 
 ```r
 if (temp <= 0) {
 "freezing"
 } else if (temp <= 10) {
 "cold"
 } else if (temp <= 20) {
 "cool"
 } else if (temp <= 30) {
 "warm"
 } else {
 "hot"
 }
 ```

 ¿Cómo cambiarías la sentencia a `cut()` si hubieras usado `<`en lugar de `<=`? ¿Cuál es la otra ventaja principal de `cut()` para este problema? (Sugerencia: ¿qué sucede si tienes muchos valores en `temp`?)

1. ¿Qué sucedería si usaras `switch()` con un valor numérico?

1. ¿Qué haría la sentencia `switch()`? ¿Qúe sucedería si `x` fuera “e”?

 
 ```r
 switch(x,
 a = ,
 b = "ab",
 c = ,
 d = "cd"
 )
 ```

 Experimente, luego lea cuidadosamente la documentación.


## Function arguments

Los argumentos de las funciones normalmente están dentro de dos grupos: por un lado, se proveen los __datos__ a calcular, y por el otro los argumentos que controlan los __detalles__ del cálculo. Por ejemplo:


* En `log()`, los datos son `x`, y los detalles son la `base` del algoritmo.

* En `mean()`, los datos son `x`, ay los detalles son la cantidad de datos para recortar de los extremos
 (`trim`) y como lidiar con los valores en blanco o perdidos (`na.rm`).

* En `t.test()`, los datos son `x` y `y`, y los detalles del test son
 `alternative`, `mu`, `paired`, `var.equal`, y `conf.level`.

* En `str_c()` puedes suministrar cualquier número de caracteres a `...`, y los detalles de la concatenación son contralos por `sep` y `collapse`.

Generalmente, los datos de los argumentos deben estar primeros. El detalle de los mismos podría estar al final y con valores predeterminados. Se especifica un valor predeterminado de la misma manera en la que es nombrada una función con su argumento:


```r
# Compute confidence interval around mean using normal approximation
mean_ci <- function(x, conf = 0.95) {
 se <- sd(x) / sqrt(length(x))
 alpha <- 1 - conf
 mean(x) + se * qnorm(c(alpha / 2, 1 - alpha / 2))
}

x <- runif(100)
mean_ci(x)
#> [1] 0.498 0.610
mean_ci(x, conf = 0.99)
#> [1] 0.480 0.628
```

El valor predeterminado debería ser casi siempre el valor más común. Pocas excepciones a esta regla podrían ser seguras. Por ejemplo, tiene sentido que `na.rm` por defecto sea `FALSE` porque los valores faltantes son importantes. Aunque `na.rm = TRUE` que es lo que usualmente pones en tu codigo, es una mala idea ignorar silenciosamente por defecto.

Cuando nombras a una función, generalmente omites los nombres de los argumentos de datos justamente porque son los más comúnmente usados. Si anulas los valores predeterminados del detalle de los argumentos, se debería usar el nombre completo:


```r
# Good
mean(1:10, na.rm = TRUE)

# Bad
mean(x = 1:10, , FALSE)
mean(, TRUE, x = c(1:10, NA))
```

Puedes referirte al argumento por su único prefijo (ej. `mean(x, n = TRUE)`), pero generalmente es mejor evitar determinadas posibilidades de confusión.

Ten en cuena que cuando invocas a una función, debes colocar un espacio alrededor de `=` en la invocación a la función, y siempre poner un espacio después de la coma, no antes (como regularmente en inglés). El uso del espacio en blanco hace más fácil ojear la función para los componentes importantes.


```r
# Good
average <- mean(feet / 12 + inches, na.rm = TRUE)

# Bad
average<-mean(feet/12+inches,na.rm=TRUE)
```

### Elección de nombres

Los nombres de los argumentos son también importantes. Para R no es sumamente importante pero si para los usuarios (¡incluyéndote en un futuro!). En general, se podrían preferir nombres más largos y más descriptivos, pero hay un puñado de nombres muy comunes y muy cortos. Vale la pena memorizar estos:

* `x`, `y`, `z`: vectores.
* `w`: un vector de pesos.
* `df`: a data frame.
* `i`, `j`: índices numéricos (usualmente filas y columnas).
* `n`: longitud, or número de filas.
* `p`: número de columnas.

Aunque, se debería considerar la coincidencia de nombres de argumentos en funciones de R. Por ejemplo, usa `na.rm` para determinar si los valores faltantes podrían ser eliminados.

### Controlando valores

A medida que comienzas a utilizar más funciones, eventualmente llegarás al punto en el que no la recordarás. En este punto es común que suceda la invalidación de la entrada de la función. Para evitar este problema, a menudo es útil hacer las restricciones explícitas. Por ejemplo, imagina que has escrito algunas funciones para calcular las estadísticas de resumen ponderadas:


```r
wt_mean <- function(x, w) {
 sum(x * w) / sum(w)
}
wt_var <- function(x, w) {
 mu <- wt_mean(x, w)
 sum(w * (x - mu) ^ 2) / sum(w)
}
wt_sd <- function(x, w) {
 sqrt(wt_var(x, w))
}
```

¿Qué pasa si `x` y `w` no son de la misma longitud?


```r
wt_mean(1:6, 1:3)
#> [1] 7.67
```

En este caso, debido a las reglas de los vectores de R, no obtenemos un error.

Es una buena práctica verificar las precondiciones, y arrojar un error (con `stop()`), si esto no es verdadero:


```r
wt_mean <- function(x, w) {
 if (length(x) != length(w)) {
 stop("`x` and `w` must be the same length", call. = FALSE)
 }
 sum(w * x) / sum(w)
}
```

Ten cuidado de no llevar esto demasiado lejos. Hay una compensación entre la cantidad de tiempo que inviertes en hacer que tu función sea sólida, en comparación con el tiempo que pasas escribiéndola. Por ejemplo, si además agregas a la función un argumento `na.rm`, probablemente no lo verificaste con cuidado:


```r
wt_mean <- function(x, w, na.rm = FALSE) {
 if (!is.logical(na.rm)) {
 stop("`na.rm` must be logical")
 }
 if (length(na.rm) != 1) {
 stop("`na.rm` must be length 1")
 }
 if (length(x) != length(w)) {
 stop("`x` and `w` must be the same length", call. = FALSE)
 }

 if (na.rm) {
 miss <- is.na(x) | is.na(w)
 x <- x[!miss]
 w <- w[!miss]
 }
 sum(w * x) / sum(w)
}
```

Esto es mucho trabajo con poca ganancia adicional. Un compromiso útil es incorporar
`stopifnot()`: esto comprueba que cada argumento sea `TRUE`, en caso contrario genera un mensaje de error.


```r
wt_mean <- function(x, w, na.rm = FALSE) {
 stopifnot(is.logical(na.rm), length(na.rm) == 1)
 stopifnot(length(x) == length(w))

 if (na.rm) {
 miss <- is.na(x) | is.na(w)
 x <- x[!miss]
 w <- w[!miss]
 }
 sum(w * x) / sum(w)
}
wt_mean(1:6, 6:1, na.rm = "foo")
#> Error in wt_mean(1:6, 6:1, na.rm = "foo"): is.logical(na.rm) is not TRUE
```

Ten en cuenta que al usar `stopifnot()` afirmas lo que debería ser cierto en lugar de verificar lo que podría estar mal.

### Punto-punto-punto (...)

Muchas funciones en R tienen un número arbitrario de entrada:


```r
sum(1, 2, 3, 4, 5, 6, 7, 8, 9, 10)
#> [1] 55
stringr::str_c("a", "b", "c", "d", "e", "f")
#> [1] "abcdef"
```

¿Cómo funcionan estas funciones? Ellos confían en un argumento especial: `...` (Pronunciando punto-punto-punto). Este argumento especial captura cualquier número de arguemntos que no están de otra forma contempladas.

Es útil porque puedes enviar estos `...` a otra función. Esto es útil si su función envuelve principalmente otra función. Por ejemplo, creo estas funciones de ayuda alrededor de `str_c()`:


```r
commas <- function(...) stringr::str_c(..., collapse = ", ")
commas(letters[1:10])
#> [1] "a, b, c, d, e, f, g, h, i, j"

rule <- function(..., pad = "-") {
 title <- paste0(...)
 width <- getOption("width") - nchar(title) - 5
 cat(title, " ", stringr::str_dup(pad, width), "\n", sep = "")
}
rule("Important output")
#> Important output ------------------------------------------------------
```

Aquí `...` permite remitirse a cualquier argumento que no quiera tratar con `str_c()`.

Esto es muy conveniente. Pero esto viene con un precio: cualquier argumento mal escrito no presentará un error. Esto hace fácil para typos que no se notan:


```r
x <- c(1, 2)
sum(x, na.mr = TRUE)
#> [1] 4
```

Si quieres captura los valores de `...`, utiliza `list(...)`.

### Evaluación diferida

Los argumentos en R se evalúan con holgazanería: no se computan hasta que se necesitan. Esto significa que si nunca se lo usa, nunca es nombrado. Hay una propiedad importante de R como lenguaje de programación, pero generalmente no es fundamental cuando uno escribe sus propias funciones para el análisis de datos. Puedes leer más acerca de la evaluación diferida
<http://adv-r.had.co.nz/Functions.html#lazy-evaluation>.

### Ejercicios

1. ¿Qué realizan las `commas(letters, collapse = "-")`? ¿Por qué?

1. Sería bueno si se pudiera suplantar múltiples caracteres al argumento `pad`,
 ej., `rule("Title", pad = "-+")`. ¿Por qué esto actualmente no funciona? ¿Cómo podrías solucionarlo?

1. ¿Qué realiza el argumento `trim` a la función `mean()`? ¿Cuándo podrías utilizarla?

1. El valor de defecto para el argumento `method` a `cor()` es
 `c("pearson", "kendall", "spearman")`. ¿Qué significa esto? ¿Qué valor se utiliza por defecto?

## Valores de salida

Darse cuenta qué es lo que tu función debería devolver suele ser bastante directo: ¡es el porqué de crear la función en primer lugar! Hay dos cosas que debes considerar al devolver un valor:

1. ¿Devolver un valor antes, hace que tu función sea más fácil de leer?

1. ¿Puedes hacer tu función apta para un pipe?

### Sentencias explícitas de salida

El valor de salida de una función suele ser la última sentencia que esta evalúa, pero puedes optar por devolver algo anticipadamente haciendo uso de la función `return()` ( del inglés devolución). Yo creo que es mejor reservar el uso de la función `return()` para los casos en donde es posible devolver anticipadamente una solución más simple. Una razón común para hacer esto es por ejemplo que los argumentos estén vacíos:


```r
complicated_function <- function(x, y, z) {
 if (length(x) == 0 || length(y) == 0) {
 return(0)
 }

 # Código complicado aquí
}
```

Otra razón puede ser porque tienes una sentencia `if` con un bloque complicado y uno sencillo. Por ejemplo, podrías escribir una sentencia if de esta manera:


```r
f <- function() {
 if (x) {
 # Haz
 # algo
 # que
 # tome
 # muchas
 # lineas
 # para
 # expresar
 } else {
 # retorna algo corto
 }
}
```

Pero si el primer bloque es muy largo, para cuando llegas al `else`, ya te has olvidado la `condition`. Una forma de reescribir esto es usar una salida anticipada para el caso sencillo:


```r

f <- function() {
 if (!x) {
 return(something_short)
 }

 # Haz
 # algo
 # que
 # tome
 # muchas
 # lineas
 # para
 # expresar
}
```

Esto permite hacer el código más fácil de entender, ya que no necesitas tanto contexto para interpretarlo.

### Escribir funciones aptas para un pipe

Si quieres escribir tus propias funciones pipeables, es importante que pienses sobre los valores de salida. Conocer el tipo de objeto de tu valor de salida significará que tu línea pipeable funciona. Por ejemplo, en dplyr y tidyr el tipo de objeto es un data frame.

Hay dos tipos básicos de funciones pipeables: transformaciones y de efectos secundarios. En las __transformaciones__, se ingresa un objeto como primer argumento y luego se devuelve una versión modificada del mismo. En el caso de __efectos_secundarios__, el objeto ingresado no es modificado sino que la función actúa sobre el objeto. Un ejemplo sería dibujar un gráfico o guardar un archivo. Las funciones de efectos secundarios deben “invisibilizar” el primer argumento, de forma que mientras no sean impresos puedan seguir siendo usados en una línea pipeable. Por ejemplo, esta función imprime el número de valores faltantes en un data frame:


```r
show_missings <- function(df) {
 n <- sum(is.na(df))
 cat("Missing values: ", n, "\n", sep = "")

 invisible(df)
}
```

Si la llamamos de manera interactiva, el comando `invisible()` significa que el valor de entrada `df` no se imprime:


```r
show_missings(mtcars)
#> Missing values: 0
```

Pero sigue estando ahí, solamente que no se imprime por defecto:


```r
x <- show_missings(mtcars)
#> Missing values: 0
class(x)
#> [1] "data.frame"
dim(x)
#> [1] 32 11
```

Y todavía podemos usarlo en un pipe:



```r
mtcars %>%
 show_missings() %>%
 mutate(mpg = ifelse(mpg < 20, NA, mpg)) %>%
 show_missings()
#> Missing values: 0
#> Missing values: 18
```

## Entorno

El último componente de una función es su entorno. Esto no es algo que debes entender profundamente cuando tu empieas a escribir funciones. Sin embargo, es importante saber un poco acerca de entornos porque es crucial para que algunas funciones trabajen. El entorno de una función controal como R encuentra el valor asociado con un nombre. Por ejemplo, la siguiente función:


```r
f <- function(x) {
 x + y
}
```

En muchos lenguajes de programación, esto sería un error, porque `y` no está definido dentro de la función. En R, esto es un código válido ya que R usa reglas llamadas __lexical scoping__ para encontrar el valor asociado a un nombre. Como `y` no está definido dentro de la función, R mirará dentro del __entorno__ donde la función fue definida:


```r
y <- 100
f(10)
#> [1] 110

y <- 1000
f(10)
#> [1] 1010
```

Este comportamiento parece una receta para errores, y de hecho debes evitar crear funciones como esta deliberadamente, pero en general no causa demasiados problemas (especialmente si reinicias regularmente R para llegar a un borrón y cuenta nueva). La ventaja de este comportamiento es que, desde el punto de vista del lenguaje, permite que R sea muy consistente. Cada nombre es buscado usando el mismo conjunto de reglas. Para `f()`incluye el comportamiento de dos cosas que podrías no esperar: `{` y `+`. Esto permite hacer cosas tortuosas como la siguiente:


```r
`+` <- function(x, y) {
 if (runif(1) < 0.1) {
 sum(x, y)
 } else {
 sum(x, y) * 1.1
 }
}
table(replicate(1000, 1 + 2))
#> 
#>   3 3.3 
#> 100 900
rm(`+`)
```

Hay un fenómeno común en R. R pone algunos límites a tu poder.
Puedes hacer cosas que no podrías en otro lenguaje de programación.
Puedes hacer cosas que el 99% de las veces son extremadamente desacertadas (¡como ignorar como funciona la adición!). Pero esta flexibilidad es lo que hace que herramientas como ggplot2 y dplyr sea posible. Aprender de como hacer el mejor uso de esta flexibilidad está mas allá del alcance de este libro, pero puedes leer al respecto en [_Advanced R_](http://adv-r.had.co.nz).
