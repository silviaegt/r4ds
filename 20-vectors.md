
# Vectores

## Introducción

Hasta ahora este libro se ha enfocado en tibbles y sus paquetes correspondientes. Pero como empezaste a escribir tus propias funciones, y a profundizar en R, es que necesitas aprender sobre vectores, es decir, sobre los objetos que soportan los tibbles. Por esto, es mejor empezar con tibbles ya que inmediatamente puedes ver su utilidad, y luego trabajar a tu manera con los componentes que están debajo, los vectores.
Los vectores son particularmente importantes, al igual que la mayoría de las funciones que escribirás y utilizarás con dichos vectores. Es posible desarrollar funciones que trabajen con tibbles (como ggplot2, dplyr and tidyr) pero las herramientas que necesitas para ello son peculiares e inmaduras. Por esto, estoy desarrollando un mejor enfoque, el cual puedes consultar en https://github.com/hadley/lazyeval, pero este no estará listo a tiempo para la publicación del libro. Incluso aún cuando esté completo, de todas maneras necesitarás entender el concepto de vectores, esto solo facilitará la escritura de capas finales que sean user-friendly (amigables al usuario).


### Pre-requisitos

Este capítulo se enfoca en las estructuras de datos de R base, por lo que no es esencial cargar ningún paquete. Sin embargo, usaremos un conjunto de funciones del paquete purrr para evitar algunas insonsistencias en R Base.


```r
library(tidyverse)
```

## Vectores básicos

There are two types of vectors:

1. _Hay dos tipos de vectores:
1.	Vectores atómicos, de los cuales existen seis tipos: lógico o booleano, entero, doble o real, caracter, complejo y raw (que consisten en datos sin procesar). Los vectores de tipo entero y doble son ampliamente conocidos como vectores númericos.
2.	Las listas, las cuales son denominadas en ciertas ocasiones como vectores recursivos debido a que pueden contener otras listas..

La diferencia principal entre vectores atómicos y listas es que los vectores atomicos son homogéneos, mientras las listas pueden ser heterogéneas. Existe, otro objeto relacionado: Null (nulo). El nulo es a menudo usado para representar la ausencia de un vector (como el opuesto a NA el cual es usado para representar la ausencia de un valor en un vector.) Null típicamente se comporta como un vector de longitud cero 0. Figura @ref (fig:datatypes) resume las interrelaciones.


<div class="figure" style="text-align: center">
<img src="diagrams_w_text_as_path/es/data-structures-overview.png" alt="The hierarchy of R's vector types" width="50%" />
<p class="caption">(\#fig:datatypes)The hierarchy of R's vector types</p>
</div>

Cada vector tiene dos propiedades claves:

1. Su tipo (__type__), el cual puedes determinarlo con la sentencia typeof() (del inglés tipode).

 
 ```r
 typeof(letters)
 #> [1] "character"
 typeof(1:10)
 #> [1] "integer"
 ```

1. Su longitud (__length__), la cual puedes determinarla con la sentencia length() (del ingés longitud).

 
 ```r
 x <- list("a", "b", 1:10)
 length(x)
 #> [1] 3
 ```

Los vectores pueden contener también arbitrariamente metadata adicional en forma de atributos. Estos atributos son usados para crear vectores aumentados los cuales implican un comportamiento distinto. Existen tres tipos de vectores aumentados:
	Los factores (factors) construidos a partir de vectores de enteros.
	Las fechas y fechas-tiempo (date-time) construidos a partir de vectores numéricos.
	Los Dataframes y tibbles construidos a partir de listas.
Este capítulo te introducirá a lo temas más importantes de vectores, desde lo más simple a lo más complicado. Comenzarás con vectores atómicos, luego seguirás con listas, y finalizarás con vectores aumentados.

## Tipos importantes de vectores atómicos

Los cuatro tipos más importantes de vectores atómicos son lógico, entero, real y carácter. Los tipos raw y complejo son raramente usados durante el análisis de datos, por lo tanto no discutiremos sobre ellos aquí.

### Lógico o Booleano
Los vectores de tipo lógico o booleano son el tipo más sencillo de vectores atómicos porque ellos solo pueden tomar tres valores posibles: falso, verdadero y Na. Los vectores lógicos son construidos usualmente con operadores de comparación, como se describe en [comparaciones]. También puedes crearlos manualmente con la función c ():


```r
1:10 %% 3 == 0
#>  [1] FALSE FALSE  TRUE FALSE FALSE  TRUE FALSE FALSE  TRUE FALSE

c(TRUE, TRUE, FALSE, NA)
#> [1]  TRUE  TRUE FALSE    NA
```

### Numérico

Los vectores compuestos por enteros o reales son conocidos ampliamente como vectores numéricos. En R, los números son por defecto, reales. Por lo que, para generar un entero, debes colocar una L después del número:


```r
typeof(1)
#> [1] "double"
typeof(1L)
#> [1] "integer"
1.5L
#> [1] 1.5
```

La distinción entre enteros y reales no es realmente importante aunque existen dos diferencias relevantes de las que debes ser consciente:
1.	Los números dobles o reales son aproximaciones. Los mismos representan números de punto flotante que no pueden ser precisamente representados con un monto fijo de memoria. Esto significa que debes considerar que todos los reales sean aproximaciones. Por ejemplo, ¿cuál es el cuadrado de la raíz cuadrada de dos?


 
 ```r
 x <- sqrt(2) ^ 2
 x
 #> [1] 2
 x - 2
 #> [1] 4.44e-16
 ```

Este compartamiento es común cuando trabajas con números de punto flotante: la mayoría de los cálculos incluyen algunos errores de aproximación. En lugar de comparar números de punto flotante usando `==`, debes usar ´dplyr::near()`, el cual provee tolerancia numérica.

1. Los números reales tienen cuatro tipos de valores posibles: NA, NaN, Inf and –Inf. Estos tres valores especiales NaN, Inf and -Inf pueden surgir a partir de la división de
c(-1, 0, 1) / 0
Evita usar == para chequear estos valores especiales. En su lugar usa la funciones de ayuda is.finite(), is.infinite(), y is.nan():



 | | 0 | Inf | NA | NaN |
 |------------------|-----|-----|-----|-----|
 | `is.finite()` | x | | | |
 | `is.infinite()` | | x | | |
 | `is.na()` | | | x | x |
 | `is.nan()` | | | | x |

### Caracter

Los vectores compuestos por carácteres son los tipos más complejos de vectores atómicos, porque cada elemento del mismo es un string, y un string puede contener una cantidad arbitraria de datos.
Ya has aprendido un montón acerca de cómo trabajar con strings en [strings]. En este punto quiero mencionar una característica importante y fundamental en la implementación de un string: R usa una reserva global de strings. Esto significa que cada string solo es almacenado en memoria una vez, y en cada uso de puntos del string a la representación. Esto reduce la cantidad de memoria necesaria por strings duplicados. Puedes ver este comportamiento en práctica con pryr::object_size():



```r
x <- "Esta es una cadena razonablemente larga."
pryr::object_size(x)
#> Registered S3 method overwritten by 'pryr':
#>   method      from
#>   print.bytes Rcpp
#> 152 B

y <- rep(x, 1000)
pryr::object_size(y)
#> 8.14 kB
```
y no utiliza más de 1000x ni tanta memoria como x, porque cada elemento de y es sólo un puntero al mismo string. Un puntero utiliza 8 bytes, entonces 1000 punteros a 136 B string es igual a 8 * 1000 + 136 = 8.13 kB.

### Valores perdidos (Missing values)

Nota que cada tipo de vector atómico tiene su propio valor perdido (o missing value):


```r
NA # logico
#> [1] NA
NA_integer_ # entero
#> [1] NA
NA_real_ # real
#> [1] NA
NA_character_ # caracter
#> [1] NA
```

Normalmente no necesitas saber sobre los diferentes tipos porque puedes siempre usar el valor NA (not Available), es decir el valor faltante, y se convertirá al tipo correcto usando las reglas de la coerción implícitas. Sin embargo, existen algunas funciones que son estrictas acerca de sus inputs, por lo tanto es útil tener presente este conocimiento así puedes ser especifico cuando lo necesites.

###Ejercicios

1. Describe la diferencia entre `is.finite(x)` y `!is.infinite(x)`.
1. Lee el código fuente de `dplyr:: near()` (Consejo: para ver el código fuente, escribe lo siguiente `()`) ¿Funcionó?
1. Un vector de tipo lógico puede tomar 3 valores posibles. ¿Cuántos valores posibles puede tomar un vector de tipo entero? ¿Cuántos valores posibles puede tomar un vector de tipo real? Usa google para realizar buscar información respecto a lo planteado anteriormente.
1. Idea al menos 4 funciones que te permitan convertir un vector del tipo real a entero. ¿En qué difieren las funciones? Sé preciso.
1. ¿Cuáles funciones del paquete readr te permiten convertir un vector del tipo string en un vector del tipo lógico, entero y doble?

## Usando vectores atómicos

Ahora que conoces los diferentes tipos de vectores atómicos, es útil repasar algunas herramientas importantes para así poder utilizarlas. Esto incluye:
1.	Cómo realizar una conversión de un determinado tipo a otro, y en cuáles casos esto sucede automáticamente.
2.	Cómo decidir si un objeto es un tipo específico de un vector.
3.	Qué sucede cuando trabajas con vectores de diferentes longitudes.
4.	Cómo nombrar los elementos de un vector
5.	Cómo obtener los elementos de interés de un vector.

### Coerción

Existen dos maneras de convertir, o coercer, un tipo de vector a otro:
1. La Coerción explicita sucede cuando defines a una función como `as.logical()`, `as.integer()`, `as.double()`, o `as.character()`. Cuando te encuentres usando coerción explicita, siempre debes comprobar que sea posible realizar la corrección en sentido ascendente, de esta manera, en primer lugar, estamos seguros que ese vector nunca tuvo tipos incorrectos. Por ejemplo, quizás necesites la especificación de `col_types` ('tipos de columna') del paquete readr.

1. La Coerción implícita sucede cuando usas un vector en un contexto especifico del cual se espera un cierto tipo de vector. Por ejemplo, cuando usas un vector del tipo lógico con la función numérica 'summary' (del inglés resumen), o cuando usas un vector del tipo doble donde se espera un vector del tipo entero.
Porque la coerción explicita es usada raramente, y es ampliamente fácil de entender, enfocaré sobre la coerción implicita aquí.
Anteriormente vimos el tipo más importante de coerción implicita: usando un vector de tipo lógico en un contexto numérico. En ese caso, el valor `TRUE` ('VERDADERO') es convertido a `1` y 'FALSE' ('FALSO') convertido a 0. Esto significa que la suma de un vector de tipo lógico es el número de los valores verdaderos, y el significado de un vector de tipo lógico es la proporción de valores verdaderos:



```r
x <- sample(20, 100, replace = TRUE)
y <- x > 10
sum(y) # ¿cuántos valores son más grandes que 10?
#> [1] 38
mean(y) # ¿cuál es la porporción de valores que son mayores que 10?
#> [1] 0.38
```

Quizás veas algún código (tipicamente más antiguo) basado en la coerción implicita pero en la dirección opuesta, es decir, de un valor entero a uno lógico


```r
if (length(x)) {
 # do something
}
```

En este caso, 0 es convertido a `FALSO` y todo lo demás es convertido a `VERDADERO`. Pienso que esto hace más dificil entender el código, por lo que no lo recomiendo. En su lugar, de ser explicito, sugiero utilizar: `length(x) > 0`.

Es también importante entender que pasa cuando creas un vector que contiene múltiples tipos con `c()`: los tipos más complejos siempre ganan.


```r
typeof(c(TRUE, 1L))
#> [1] "integer"
typeof(c(1L, 1.5))
#> [1] "double"
typeof(c(1.5, "a"))
#> [1] "character"
```

Un vector atómico no puede contener un mix de diferentes tipos porque el tipo es una propiedad de un vector completo, no de elementos individuales. Si necesitas un mix de múltiples tipos en el mismo vector, entonces debes usar una lista, la cual aprenderás en breve.

### Funciones de test

Algunas veces quieres hacer las cosas de una manera diferente basadas en el tipo de vector. Una de las opciones es el uso de la sentencia `typeof()`. Otra es usar una función test la cual devuelva un valor `TRUE` o 'FALSO' . R base provee varias funciones como `is.vector()` y `is.atomic()`, pero estas a menudo devuelven resultados inesperados. En su lugar, es más acertado usar las funciones `is_*` provistas por el paquete purrr, las cual están resumidas en la tabla que se muestra a continuación.

| | lgl | int | dbl | chr | list |
|------------------|-----|-----|-----|-----|------|
| `is_logical()` | x | | | | |
| `is_integer()` | | x | | | |
| `is_double()` | | | x | | |
| `is_numeric()` | | x | x | | |
| `is_character()` | | | | x | |
| `is_atomic()` | x | x | x | x | |
| `is_list()` | | | | | x |
| `is_vector()` | x | x | x | x | x |

Cada predicado además viene con una version para "escalares", donde la función `is_scalar_atomic()`, chequea que la longitud sea 1. Esto es útil, por ejemplo, si quieres chequear en algún argumento que tu función sea un solo valor lógico.

### Escalares y reglas de reciclado

Así como implicitamente se coercionan los tipos de vectores que son compatibles, R también implicitamente coerciona la longitud de los vectores. Esto se denomina vector __recycling__, o reciclado de vectores, debido a que el vector de menor longitud se repite, o recicla, hasta igualar la longitud del vector de mayor longitud.
Esto es generalmente lo más útil cuando estás trabajando con vectores y "escalares". Los escalares están puestos en notas porque R en realidad no tiene definido los escalares: en su lugar, un solo número conforma un vector de longitud 1. Debido a que no existen los escalares, la mayoría de las funciones están construidas como vectorizadas, esto significa que operan sobre un vector del tipo númerico. Esto es así porque, por ejemplo, este código funciona:


```r
sample(10) + 100 # (del inglés muestreo)
#>  [1] 107 104 103 109 102 101 106 110 105 108
runif(10) > 0.5
#>  [1] FALSE  TRUE FALSE FALSE  TRUE  TRUE  TRUE  TRUE  TRUE  TRUE
```
En R, las operaciones matemáticas básicas funcionan con vectores. Lo que significa que no necesitarás la ejecución de una interación explicita cuando realices cálculos matemáticos sencillos.
Es intuitivo lo que debería pasar si agregas dos vectores de la misma longitud, o un vector y un "escalar", pero ¿qué sucede si agregas dos vectores de diferentes longitudes?


```r
1:10 + 1:2
#>  [1]  2  4  4  6  6  8  8 10 10 12
```
Aquí, R expandirá el vector de menor longitud a la misma longitud del vector de mayor longitud, a esto es lo que denominamos reciclaje o reutilización de un vector. Esto es una excepción cuando la longitud del vector de mayor longitud no es un múltiplo entero de la longitud del vector más corto:


```r
1:10 + 1:3
#> Warning in 1:10 + 1:3: longer object length is not a multiple of shorter
#> object length
#>  [1]  2  4  6  5  7  9  8 10 12 11
```
Mientras el vector reciclado puede ser usado para crear código claro y conciso, también puede ocultar problemas de manera silenciosa. Por esta razón, las funciones vectorizadas en tidyverse mostrarán errores cuando recicles cualquier otra cosa que no sea un escalar. Si quieres reutilzar, necesitarás hacerlo tu mismo con la sentencia `rep()`:


```r
tibble(x = 1:4, y = 1:2)
#> Tibble columns must have consistent lengths, only values of length one are recycled:
#> * Length 2: Column `y`
#> * Length 4: Column `x`

tibble(x = 1:4, y = rep(1:2, 2))
#> # A tibble: 4 x 2
#>       x     y
#>   <int> <int>
#> 1     1     1
#> 2     2     2
#> 3     3     1
#> 4     4     2

tibble(x = 1:4, y = rep(1:2, each = 2))
#> # A tibble: 4 x 2
#>       x     y
#>   <int> <int>
#> 1     1     1
#> 2     2     1
#> 3     3     2
#> 4     4     2
```
### Nombrando vectores

Todos los tipos de vectores pueden ser nombrados. Puedes asignarles un nombre al momento de crearlos con `c()`:


```r
c(x = 1, y = 2, z = 4)
#> x y z 
#> 1 2 4
```

O después de la creación con `purrr::set_names()`:


```r
set_names(1:3, c("a", "b", "c"))
#> a b c 
#> 1 2 3
```
Los vectores con nombres son más útiles para subconjuntos, como se describe a continuación.

### Subsetting (Subdivisión o creación de subconjuntos) {#vector-subsetting, subdivisión de vectores}

Hasta ahora hemos usado `dplyr::filter()` para filtrar filas en una TIBBLE. La sentencia `filter()` sólo funciona con TIBBLES, por lo que necesitaremos una nueva herramienta para trabajar con vectores: `[`. `[` representa a la función Subdivisión (Subsetting), la cual nos permite crear subconjuntos o subdivisiones a partir de vectores, y se indica como `x[a]`. Existen cuatro tipos de cosas en las que puedes subdividir un vector:

1. Un vector numérico contiene sólo enteros. Los enteros deben ser todos positivos, todos negativos, o cero.

 La Subdivisión con enteros positivos mantiene los elementos en aquellas posiciones:

 
 ```r
 x <- c("uno", "dos", "tres", "cuatro", "cinco")
 x[c(3, 2, 5)]
 #> [1] "tres"  "dos"   "cinco"
 x[c(3, 2, 5)]
 #> [1] "tres"  "dos"   "cinco"
 ```

 Repitiendo una posición, puedes en realidad generar un output de mayor longitud que el input::

 
 ```r
 x[c(1, 1, 5, 5, 5, 2)]
 #> [1] "uno"   "uno"   "cinco" "cinco" "cinco" "dos"
 ```
 Los valores negativos eliminan elementos en posiciones especificas:

 
 ```r
 x[c(-1, -3, -5)]
 #> [1] "dos"    "cuatro"
 ```
 Es un error mezclar valores positivos y negativos:


 
 ```r
 x[c(1, -1)]
 #> Error in x[c(1, -1)]: only 0's may be mixed with negative subscripts
 ```

 El mensaje menciona subdivisiones utilizando cero, lo cual no returna valores.

 
 ```r
 x[0]
 #> character(0)
 ```

 Esto a menudo no es útil, pero puede ser de ayuda si quieres crear estructuras de datos inusuales para testear tus funciones.


1. La subdivisión de un vector lógico mantiene/almacena todos los valores correspondientes al valor `TRUE` `VERDADERO`. Esto es a menudo mayormente útil en conjunto con las funciones de comparación.

 
 ```r
 x <- c(10, 3, NA, 5, 8, 1, NA)
 
 # Todos los valores non-missing, es decir, distintos de NA de x
 x[!is.na(x)]
 #> [1] 10  3  5  8  1
 
 # Todos, incluso los valores (missing) de x
 x[x %% 2 == 0]
 #> [1] 10 NA  8 NA
 ```
1. Si tienes un vector con nombre, puedes subdivirlo en un vector de tipo caracter.

 
 ```r
 x <- c(abc = 1, def = 2, xyz = 5)
 x[c("xyz", "def")]
 #> xyz def 
 #>   5   2
 ```
 Como con los enteros positivos, también puedes usar un vector del tipo caracter para duplicar entradas individuales.

1. El tipo más sencillo de subsetting es el valor vacío, `x[]`, el cual retorna el valor completo de `x`. Esto no es útil para vectores subdivididos, aunque si lo es para matrices subdivididas(y otras estructuras de grandes dimensiones) ya que te permite seleccionar toda las filas o todas las columnas, dejando el indice en blanco. Por ejemplo, si `x` está en la segunda posición, `x[1, ]` selecciona la primera fila y todas las columnas, y la expresión `x[, -1]` selecciona todas las filas y todas las columnas excepto la primera.

 Para aprender más acerca de las aplicaciones de subsetting, puedes leer el capítulo de Subsetting de R Avanzado: <http://adv-r.had.co.nz/Subsetting.html#applications>.

 Existe una importante variación de `[`, la cual consiste en `[[`. Esta expresión `[[` sólo extrae un único elemento, y siempre omite nombres. Es una buena idea usarla siempre que quieras dejar en claro que estás extrayendo un único item, como en un bucle for. La diferencia entre `[` y`[[` es más importante para las listas (lists), como veremos en breve.

### Ejercicios

1. La expresión `mean(is.na(x))`, ¿qué dice acerca del vector 'x'? ¿y qué sucede con la expresión `sum(!is.finite(x))`?

1. Detenidamente lee la documentación de `is.vector()`. ¿Para qué se prueba la función realmente? ¿Por qué la función `is.atomic()` no concuerda con la definición de vectores atómicos vista anteriormente?

1. Compara y contraste `setNames()` con `purrr::set_names()`.

1. Crea funciones que tomen un vector como entrada y devuelva:
	1. El último valor. ¿Deberás usar `[` o `[[`?.
	1. Los elementos en posiciones pares.
	1. Cada elemento excepto el último valor.
	1. Sólo las posiciones pares (y sin valores perdidos (missing values)).
	1. ¿Por qué `x[-which(x > 0)]` no es lo mismo que `x[x <= 0]`?
	1. ¿Qué sucede cuando realizas un subset (subdivisión) con un entero positivo que es mayor que la longitud del vector? ¿Qué sucede cuando realizas un subset (subdivisión) con un nombre que no existe?

## Vectores Recursivos (listas)
Las listas son un escalon más en complejidad partiendo de los vectores atómicos, debido a que las listas pueden contener otras listas. Lo cual las hace adecuadas para representar una estructura jerárquica o de tipo árbol. Puedes crear una lista con ´list()´:


```r
x <- list(1, 2, 3) 
x
#> [[1]]
#> [1] 1
#> 
#> [[2]]
#> [1] 2
#> 
#> [[3]]
#> [1] 3
```
Un herramienta muy útil para trabajar con listas es ´str()´ ya que se enfoca en la estructura, no en los contenidos.

```r
str(x)
#> List of 3
#>  $ : num 1
#>  $ : num 2
#>  $ : num 3

x_nombrada <- list(a = 1, b = 2, c = 3)
str(x_nombrada)
#> List of 3
#>  $ a: num 1
#>  $ b: num 2
#>  $ c: num 3
```

A diferencia de los vectores atómicos, el objeto ´list()´ puede contener una variedad de diferentes objetos:

```r
y <- list("a", 1L, 1.5, TRUE)
str(y)
#> List of 4
#>  $ : chr "a"
#>  $ : int 1
#>  $ : num 1.5
#>  $ : logi TRUE
```

¡Incluso las listas pueden contener otras listas!

```r
z <- list(list(1, 2), list(3, 4))
str(z)
#> List of 2
#>  $ :List of 2
#>   ..$ : num 1
#>   ..$ : num 2
#>  $ :List of 2
#>   ..$ : num 3
#>   ..$ : num 4
```

### Visualizando listas
Para explicar funciones de manipulacion de listas más complicadas, es útil tener una representacion visual de las mismas. Por ejemplo, defino estas tres listas:

```r
x1 <- list(c(1, 2), c(3, 4))
x2 <- list(list(1, 2), list(3, 4))
x3 <- list(1, list(2, list(3)))
```

Y a continuación, las grafico:
<img src="diagrams_w_text_as_path/es/lists-structure.png" width="75%" style="display: block; margin: auto;" />

Existen tres principios, al momento de observer el gráfico anterior:
1.	Las listas tienen esquinas redondeadas, en cambio, los vectores atómicos tienen esquinas cuadradas.
2.	Los hijos son representados dentro de sus listas padres, y tienen un fondo ligeramente más oscuro para facilitar la visualización de la jerarquía.
3.	No es importante la orientación de los hijos (p.ej. las filas o columnas), entonces utilizaré la orientacion de una fila o columna para almacenar espacio o incluso para ilustrar una propiedad importante en el ejemplo.

Subdivisión (Subsetting)
Existen tres maneras de subdividir una lista, lo cual ilustraré con una lista denominada ´a´:

```r
a <- list(a = 1:3, b = "una cadena", c = pi, d = list(-1, -5))
```
El corchete simple ´[´ extrae una sub-lista. Por lo que, el resultado siempre será una lista.


```r
str(a[1:2])
#> List of 2
#>  $ a: int [1:3] 1 2 3
#>  $ b: chr "una cadena"
str(a[4])
#> List of 1
#>  $ d:List of 2
#>   ..$ : num -1
#>   ..$ : num -5
```
Al igual que con vectores, puedes subdividirla, en un vector lógico, de enteros o caracteres.
El doble corchete ´[[´ extrae un solo componente de una lista. Y remueve un nivel de la jerarquía de la lista.


```r
str(a[[1]])
#>  int [1:3] 1 2 3
str(a[[4]])
#> List of 2
#>  $ : num -1
#>  $ : num -5
```
El símbolo $ es un atajo para extraer elementos con nombres de una lista. Este funciona de modo similar al doble corchete´[[´ excepto que en el primer caso no es necesario el uso de comillas dobles.


```r
a$a
#> [1] 1 2 3
a[["a"]]
#> [1] 1 2 3
```
La diferencia entre el corchete simple `[` y el doble `[[` es realmente importante para las listas, porque el doble `[[` profundiza en una lista mientras que el simple `[` retorna una nueva, lista más pequeña. Compara el código y el output de arriba con la representacion visual de la Figura @ref(fig:lists-subsetting).

```{r
lists-subsetting, echo = FALSE, out.width = "75%", fig.cap = "Subdividir una lista, de manera visual."}
knitr::include_graphics("diagrams_w_text_as_path/es/lists-subsetting.png")
```

### Listas de Condimentos
La diferencia entre ambos `[` y `[[` es muy importante, pero es muy fácil confundirlos. Para ayudarte a recordar, permiteme mostrarte un pimientero inusual
<img src="images/pepper.jpg" width="25%" style="display: block; margin: auto;" />

Si este pimientero es tu lista `x`, entonces, `x[1] es un pimientero que contiene un simple paquete de pimienta:
<img src="images/pepper-1.jpg" width="25%" style="display: block; margin: auto;" />

La expresión `x[2]` luciría del mismo modo, pero podría contener el segundo paquete. La expresión `x[1:2]` sería un pimientero que contiente dos paquetes de pimienta.

La expresión `x[[1]]` es:
<img src="images/pepper-2.jpg" width="25%" style="display: block; margin: auto;" />
Si quisieras obtener el contenido del paquete de pimiento, necesitarías utilizar la siguiente expresión `x[[1]][[1]:

<img src="images/pepper-3.jpg" width="25%" style="display: block; margin: auto;" />

### Ejercicios
1.Dibuja las listas siguientes como sets anidados:

```{r
 1.	`list(a, b, list(c, d), list(e, f))`
 1.	`list(list(list(list(list(list(a))))))`
```
1.¿Qué pasaría si subdividieras un tibble como si fuera una lista? ¿Cuáles son las principales diferencias entre una lista y un tibble?

## Atributos
Cualquier vector puede contener metadata arbitraria adicional mediante sus atributos. Puedes pensar en los atributos como una lista de vectores que pueden ser adjuntadas a cualquier otro objeto. Puedes obtener y setear valores de atributos individuales con `attr()` o verlos todos al mismo tiempo con `attributes()`.


```r
x <- 1:10
attr(x, "saludo")
#> NULL
attr(x, "saludo") <- "¡Hola!"
attr(x, " despedida") <- "¡Adiós!"
attributes(x)
#> $saludo
#> [1] "¡Hola!"
#> 
#> $` despedida`
#> [1] "¡Adiós!"
```

Existen tres atributos muy importantes que son utilizados para implementar partes fundamentals de R:
1.	Los __Nombres__ son utilizados para nombrar los elementos de un vector.
2.	Las __Dimensiones__ (o dims, denominación más corta) hacen que un vector se comporte como una matriz o arreglo.
3.	Una __Clase__ es utilizada para implementar el sistema S3 orientado a objetos.
A los atributos nombres los vimos arriba, y no cubriremos las dimensiones porque no se usan matrices en este libro. Resta describir el atributo clase, el cual controla como las funciones genéricas funcionan. Las funciones genéricas son clave para la programacion orientada a objetos en R, porque ellas hacen que las funciones se comporten de manera diferente de acuerdo a las diferentes clases de inputs. Una discusión más profunda sobre programacion orientada a objetos no está contemplada en el ámbito de este libro, pero puedes leer más al respecto en el documento R Avanzado en: http://adv-r.had.co.nz/OO-essentials.html#s3.
Así es como una función genérica típica luce:

```r
as.Date
#> function (x, ...) 
#> UseMethod("as.Date")
#> <bytecode: 0x6952938>
#> <environment: namespace:base>
```
La llamada al método "UseMethod" significa que esta es una función genérica, y llamará a un metódo específico, una función, basada en la clase del primer argumento. (Todos los métodos son funciones; no todas las funciones son métodos). Puedes listar todos los métodos existentes para una función genérica utilizando la función: `methods()`:


```r
methods("as.Date")
#> [1] as.Date.character   as.Date.default     as.Date.factor     
#> [4] as.Date.numeric     as.Date.POSIXct     as.Date.POSIXlt    
#> [7] as.Date.vctrs_sclr* as.Date.vctrs_vctr*
#> see '?methods' for accessing help and source code
```
Por ejemplo, si `x` es un vector de caracteres, `as.Date()` llamará a `as.Date.character()`; si es un factor, llamará a `as.Date.factor()`.

Puedes ver la implementación específica de un método con: `getS3method()`:


```r
getS3method("as.Date", "default")
#> function (x, ...) 
#> {
#>     if (inherits(x, "Date")) 
#>         x
#>     else if (is.logical(x) && all(is.na(x))) 
#>         .Date(as.numeric(x))
#>     else stop(gettextf("do not know how to convert '%s' to class %s", 
#>         deparse(substitute(x)), dQuote("Date")), domain = NA)
#> }
#> <bytecode: 0x6958158>
#> <environment: namespace:base>
getS3method("as.Date", "numeric")
#> function (x, origin, ...) 
#> {
#>     if (missing(origin)) 
#>         stop("'origin' must be supplied")
#>     as.Date(origin, ...) + x
#> }
#> <bytecode: 0x987cf60>
#> <environment: namespace:base>
```
Lo mas importante del S3 genérico; sistema OO, es decir, orientado a objetos; es la función `print()`: el cual controla como el objeto es impreso cuando tipeas su nombre en la consola. Otras funciones genéricas importantes son las funciones de subdivisión `[`, `[[`, and `$`.

## Vectores Aumentados
Los vectores atómicos y las listas son los bloques sobre los que se construyen otros tipos importantes de vectores como los tipos factores y fechas (dates). A estos vectores , los llamo __ vectores aumentados __, porque son vectores con __attributos__ adicionales, incluyendo la clase. Los vectores aumentados tienen una clase, por ello se comportan de manera diferente a los vectores atómicos sobre los cuales son construidos. En este libro, hacemos uso de cuatro importantes vectores aumentados:
•	Los Factores
•	Las Dates (Fechas)
•	Los Date-times (Fecha-tiempo)
•	Los Tibbles
Estos son descriptos a continuación:
Factores
Los factores son diseñados para representar datos categoricos que pueden tomar un set fijo de valores posibles, son construidos sobre los enteros, y tienen

```r
x <- factor(c("ab", "cd", "ab"), levels = c("ab", "cd", "ef"))
typeof(x)
#> [1] "integer"
attributes(x)
#> $levels
#> [1] "ab" "cd" "ef"
#> 
#> $class
#> [1] "factor"
```
### Dates and date-times (Fechas y Fecha – Hora)
Las vectores del tipo date en R son vectores numéricos que representan el número de días desde el 1° de enero de 1970.


```r
x <- as.Date("1971-01-01")
unclass(x)
#> [1] 365

typeof(x)
#> [1] "double"
attributes(x)
#> $class
#> [1] "Date"
```
Los vectores date-time son vectores numéricos de clase `POSIXct` que representan el número de segundos desde el 1° de enero de 1970. (En caso de que te preguntes sobre "POSIXct"; "POSIXct" significa "Portable Operating System Interface, lo que significa “Interfaz portable de sistema operativo"; tiempo de calendario.)

```r

x <- lubridate::ymd_hm("1970-01-01 01:00")
unclass(x)
#> [1] 3600
#> attr(,"tzone")
#> [1] "UTC"

typeof(x)
#> [1] "double"
attributes(x)
#> $class
#> [1] "POSIXct" "POSIXt" 
#> 
#> $tzone
#> [1] "UTC"
```
El atributo `tzone` es opcional. Este controla como se muestra la hora, y no hace referencia al tiempo en términos absolutos.

```r
attr(x, "tzone") <- "US/Pacific"
x
#> [1] "1969-12-31 17:00:00 PST"
attr(x, "tzone") <- "US/Eastern"
x
#> [1] "1969-12-31 20:00:00 EST"
```
Existe otro tipo de vector date-time llamado POSIXlt. Éstos son construidos en base a listas con nombres (named lists).

```r
y <- as.POSIXlt(x)
typeof(y)
#> [1] "list"
attributes(y)
#> $names
#>  [1] "sec"    "min"    "hour"   "mday"   "mon"    "year"   "wday"  
#>  [8] "yday"   "isdst"  "zone"   "gmtoff"
#> 
#> $class
#> [1] "POSIXlt" "POSIXt" 
#> 
#> $tzone
#> [1] "US/Eastern" "EST"        "EDT"
```

Los vectores POSIXlts son pocos comunes dentro del paquete tidyverse. Surgen en base a R, porque son necesarios para extraer components específicos de una fecha, como el año o el mes. Desde que el paquete lubridate te provee helpers para efectuar dicha extracción, los vectores POSIXlts no son necesarios. Siempre es más sencillo trabajar con loa vectores POSIXct's, por lo tanto si tenés un vector POSIXlt, deberías convertirlo a un vector regular del tipo data-time con `lubridate::as_date_time()`.

### Tibbles
Los Tibbles son listas aumentadas: los cuales tienen las clases "tbl_df", "tbl" y "data.frame", y los atributos `names` (para nombrar una columna) y `row.names` (para nombrar una fila):

```r
tb <- tibble::tibble(x = 1:5, y = 5:1)
typeof(tb)
#> [1] "list"
attributes(tb)
#> $names
#> [1] "x" "y"
#> 
#> $row.names
#> [1] 1 2 3 4 5
#> 
#> $class
#> [1] "tbl_df"     "tbl"        "data.frame"
```
La diferencia entre un tibble y una lista, consiste en que todos los elementos de un data frame deben ser vectores con la misma longitud. Por lo tanto, todas las funciones que utilizan tibbles refuerzan esta condición.

Además, los data.frames tradicionales tienen una estructura muy similar a los tibbles:

```r
df <- data.frame(x = 1:5, y = 5:1)
typeof(df)
#> [1] "list"
attributes(df)
#> $names
#> [1] "x" "y"
#> 
#> $class
#> [1] "data.frame"
#> 
#> $row.names
#> [1] 1 2 3 4 5
```
Se puede decir, que la diferencia principal entre ambos es la clase. Debido a que un tibble incluye el tipo "data.frame" lo que significa que los tibbles heredan el comportamiento regular de un data frame por defecto.
Ejecicios:
1.	¿Qué valor retorna la siguiente expresión hms::hms(3600)? ¿Cómo se muestra? ¿Cuál es la tipo primario sobre el cual se basa el vector aumentado? ¿Qué atributos utiliza el mismo?
2.	Intenta y crea un tibble que tenga columnas con diferentes longitudes ¿Qué es lo que ocurre?
3.	Teniendo en cuenta
