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

