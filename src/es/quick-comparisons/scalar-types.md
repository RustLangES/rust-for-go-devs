# Tipos Escalares

La siguiente tabla enumera los tipos primitivos en Rust y su equivalente en
Go:

| **Rust** | **Go**              | **Notas**                                                                                                                                  |
| -------- | ------------------- | -------------------------------------------------------------------------------------------------------------------------------------------|
| `bool`   | `bool`              | Ambos representan `true` o `false`.                                                                                                        |
| `char`   | `rune`              | En Rust, `char` es un carácter Unicode de 4 bytes. `rune` en Go es alias de `int32`.                                                       |
| `i8`     | `int8`              | Enteros con signo de 8 bits.                                                                                                               |
| `i16`    | `int16`             | Enteros con signo de 16 bits.                                                                                                              |
| `i32`    | `int32`             | Rust lo infiere como tipo entero por defecto.                                                                                              |
| `i64`    | `int64`             | Enteros con signo de 64 bits.                                                                                                              |
| `i128`   | —                   | Go **no tiene soporte nativo** para enteros de 128 bits.                                                                                   |
| `isize`  | `int`               | Tamaño del puntero (32 o 64 bits según la arquitectura). Go usa `int` igual, valor por defecto de enteros en Go.                           |
| `u8`     | `byte` / `uint8`    | Ambos representan enteros sin signo de 8 bits. `byte` es alias en Go.                                                                      |
| `u16`    | `uint16`            | Enteros sin signo de 16 bits.                                                                                                              |
| `u32`    | `uint32`            | Enteros sin signo de 32 bits.                                                                                                              |
| `u64`    | `uint64`            | Enteros sin signo de 64 bits.                                                                                                              |
| `u128`   | —                   | Go no soporta enteros sin signo de 128 bits.                                                                                               |
| `usize`  | `uint`              | Entero sin signo del tamaño de puntero. Go usa `uint` igual.                                                                               |
| `f16`    | -                   | Go no tiene soporte nativo para `float16`. Rust esta agregando soporte para este tipo.                                                     |
| `f32`    | `float32`           | Números de punto flotante de precisión simple.                                                                                             |
| `f64`    | `float64`           | Valor por defecto en Go para floats. Igual que en Rust.                                                                                    |
| `f128`   | -                   | Go no soporta `float128`. Rust esta agregando soporte para este tipo.                                                                      |
| `()`     | -                   | El tipo Unit de Rust (tipo sin valor). En Go, `void` no existe como tipo.                                                                  |


Notas:

1. Enteros por defecto:

   - Rust usa i32 como entero por defecto.

   - Go usa int, que es int32 o int64 dependiendo de la arquitectura (pero 
     típicamente int64 en 64-bit).

2. Tipos sin signo (u*):

   En Go, su uso es menos común, en Rust, son más usados especialmente donde se 
   quiere más control del comportamiento binario.

3. () en Rust vs. void en Go:

   Rust trata () como un tipo real. En Go, no existe un valor tipo void, pero 
   las funciones pueden retornar “nada”.

4. [`char`][char.rs] en Rust vs. [`rune`][char-go] en Go:

   En Rust, `char` es un carácter Unicode de 4 bytes, mientras que en Go, `rune`
   es un alias para `int32`, representando un punto de código Unicode.

   Esto significa que en Rust, un `char` puede representar cualquier carácter
   Unicode valido y solo eso, mientras que en Go, un `rune` también puede 
   representar cualquier carácter Unicode pero no valida que sea un
   [Unicode scalar value], lo que puede llevar a errores si se usa 
   incorrectamente.

5. `f16` y `f128`:

   Go no tiene soporte nativo para `float16` ni `float128`, mientras que Rust está
   agregando soporte para estos tipos en el siguiente 
   [RFC][https://rust-lang.github.io/rfcs/3453-f16-and-f128.html]. Es posible
   que para cuando leas esto, ya esté disponible en Rust, actualmente es usable
   en la versión nightly del compilador.

Mira también:

- [Primitivos (Rust By Example)][primitives.rs]

[char-go]: https://go.dev/blog/strings#code-points-characters-and-runes
[char.rs]: https://doc.rust-lang.org/std/primitive.char.html
[Unicode scalar value]: https://www.unicode.org/glossary/#unicode_scalar_value
[primitives.rs]: https://doc.rust-lang.org/rust-by-example/primitives.html