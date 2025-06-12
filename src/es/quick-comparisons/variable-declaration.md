# Declaración de variables

En Go usualmente para declarar una variable se utiliza la palabra clave `var` 
seguida del nombre de la variable, el tipo y un valor opcional. Por ejemplo:

```go
var x int = 10
```
En Rust, la declaración de variables es similar, pero se utiliza la palabra 
clave `let` en lugar de `var`. Por ejemplo:

```rust
let x: i32 = 10;
```

También se usa los `:` para especificar el tipo de la variable, en este caso 
`i32`.

Recordemos que `i32` es el tipo de dato entero de 32 bits en Rust, que es
equivalente al tipo `int` en Go. 

En Go, si no se especifica el tipo de la variable, el compilador lo infiere
automáticamente. Por ejemplo:

```go
var x = 10
```

De una manera similar, en Rust también se puede omitir el tipo de la variable
y el compilador lo inferirá automáticamente:

```rust
let x = 10;
```

En Rust, las variables son inmutables por defecto. El motivo de esto es que es 
importante que obtengamos errores en tiempo de compilación cuando intentamos 
cambiar un valor que está designado como inmutable, porque esta situación puede 
conducir a errores. Si una parte de nuestro código funciona bajo la suposición 
de que un valor nunca cambiará y otra parte de nuestro código cambia ese valor, 
es posible que alguna parte del código no haga lo que estaba diseñado para 
hacer. De esa forma se evidencia más fácilmente el error, esta decisión
fue tomada porque esto ayuda mucho a prevenir errores.

Si queremos declarar una variable mutable en Rust, debemos usar la palabra
clave `mut` al declararla. Por ejemplo:

```rust
let mut x = 10;
```
En este caso, `x` es una variable mutable y podemos cambiar su valor más adelante
en el código.

```rust
x = 20;
```

En Go, las variables son mutables por defecto, lo que significa que podemos
cambiar su valor sin necesidad de declararlas como mutables. 

El caso de Rust ayuda a prevenir que si una variable es compartida en diferentes
funciones, hilos o contextos, no se pueda cambiar su valor accidentalmente.

En Go, si queremos declarar una variable como inmutable, debemos usar la palabra
clave `const`. Por ejemplo:

```go
const x = 10
```

En este caso, `x` es una constante y no podemos cambiar su valor más adelante en 
el código. En Rust, las constantes se declaran de manera similar, pero se 
utiliza la palabra clave `const` y se debe especificar el tipo de dato. Por 
ejemplo:

```rust
const CONSTANTE: i32 = 10;
```

En este caso, `CONSTANTE` es una constante de tipo `i32` y no podemos cambiar su 
valor. Las constantes en Rust representan que son valores que son insertados
en el código de manera literal, por lo que no ocupan memoria en tiempo de 
ejecución, lo que las hace más eficientes en términos de rendimiento.

Las constantes en Rust pueden ser evaluadas en tiempo de compilación, lo que
significa que podemos generar funciones que devuelvan constantes. Veremos más de
esto en secciones posteriores, pero es importante mencionarlo porque 
en Rust se esta buscando desarrollar más esta apartado de funciones y valores
constantes, ya que por ejemplo en primera instancia Go no tiene esta
característica y por otro lado son una herramienta poderosísima para optimizar
el rendimiento de nuestro código.