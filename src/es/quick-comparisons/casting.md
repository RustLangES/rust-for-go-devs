# Casting

Tanto Go como Rust tienen casteos explicitos, para transformar en Go un tipo de 
dato a otro generalmente se ejecuta una función como puede verse en el ejemplo
de abajo.

```go
var x int = 10
var y float64 = float64(x)
var z int32 = int32(x)
```

Go obliga a usar castings cuando se mezclan tipos distintos.

En Rust también esto también requerira un casting explicito, pe´ro en lugar de 
usando una función con el nombre del tipo, utilizamos directamente la keyword
`as` como se puede ver en el ejemplo de abajo:

```rust
let x: i32 = 10;
let y: f64 = x as f64;
let z: u8 = x as u8;
```

Al igual que en Go, Rust al  el operador as trunca o satura en algunos casos 
(por ejemplo, de i32 a u8). 

```rust
let big: u32 = 300;
let small: u8 = big as u8;
```

`small` valdria 44, porque el resto de 300 divido 256 es eso básicamente 44.
En Rust no hay panic en tiempo de ejecución, pero puede haber pérdida silenciosa 
de datos si no se controla correctamente.

Rust sin embargo además de este casteo rápido provee otros tipos de casteos que 
son más correctos. Seguramente este tema lo veremos más adelante pero Rust 
provee formas estandarizadas de trabajar la conversión de tipos con el trait 
`From`.
Profundizaremos más adelante acerca de esto.

Retomando el tema la mayoría de tipos de datos poseen una 
función asociada/método gracias a ese trait.

Por lo que podríamos hacer 

```rust
let s = String::from(42);
```

Pero también tendríamos el método `.into()`: 


```rust
let s: String = 42.into();
```

También podrían haber alternativas drásticamente diferentes como:

```rust
let s = 42.to_string();
```

Sin embargo la forma más segura de hacer una conversión entre tipos es con 
`TryFrom`, ya que con este trait podemos validar si hay algún error durante la 
conversión:

```rust
let n: i32 = 300;
let r: Result<u8, _> = n.try_into();
```

`try_into()` devuelve un tipo de dato `Result<T, E>` donde `T` es el valor del 
tipo de dato que deseamos y el `E` es uno o multiples errores.

Más adelante profundizaremos al respecto pero de forma simplificada esta 
operación de `try_into()` nos devolvería el valor que queremos si se logra
ejecutar correctamente o un Error en el caso de que tengamos algún 
inconveniente, como por ejemplo puede ser el caso del que hablamos previamente.

El exceso de bytes hace que sea imposible castear el valor `300` de tipo `i32`
a un tipo de dato `u8`, pero a diferencia de la ocasión anterior, esta vez
obtendremos el error como tal.

Rust intenta ser lo más preciso posible, ni Go, ni Rust están mal con respecto a
esto.

Ambos tienen el mismo comportamiento en el caso de usar la keyword `as` o de 
forma extendida con `From` o `Into`, esto viene de la tradición de C.
No hay un “estándar formal” como ISO que lo imponga en todos los lenguajes, pero 
es un comportamiento esperado y documentado en todos los lenguajes de sistemas 
porque:
- Es rápido (no se hace verificación).
- Se ajusta al comportamiento del hardware.
- Es predecible para quienes programan bajo esas reglas.

Sin embargo como vimos con el `try_from()` o con `try_into()` esto cambia en 
Rust, en Go no tienes una forma de segurizar de manera automática este proceso.
Rust provee estas funciones que te permiten capturar los posibles errores.

| Lenguaje | Casteo a tipo más pequeño | ¿Puede fallar en runtime? | ¿Se puede hacer conversiones seguras? |
| -------- | ------------------------- | ------------------------- | ------------------------------------- |
| Go       | Trunca                    | ❌ No                      | 🚫 No (`uint8(x)` siempre trunca)     |
| Rust     | Trunca con `as`           | ❌ No                      | ✅ Sí con `TryFrom`                    |

En Go si quisieras hacer lo que hace el `try_from()` o el `try_into()` debes 
validarlo a mano como con algo así:

```go
if x >= 0 && x <= 255 {
    var y uint8 = uint8(x)
    fmt.Println("Seguro:", y)
} else {
    fmt.Println("¡Fuera de rango!")
}
```

En Rust si inferimos el error podríamos tener algo como esto:

```rust
let n: i32 = 300;
let r = u8::try_from(n);
```

Donde `r` o contiene el valor correcto o un error, y nosotros podemos decidir
que hacer con el error, pero la validación ha sido contenida dentro del método.


