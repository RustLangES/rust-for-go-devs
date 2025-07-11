# Ownership y Borrowing

Una de las diferencias más profundas entre Go y Rust es cómo se maneja la 
memoria.
Mientras que Go confía en un GC (Garbage Collector) para liberar recursos, Rust
no tiene GC.
En su lugar, Rust utiliza un sistema llamado Ownership (Propiedad) y Borrowing
(Prestamos) que le permite liberar memoria automáticamente y de forma segura en
tiempo de compilación.

Puede sonar complejo al principio, pero una vez entendido, notarás que te da más 
control, más performance, más seguridad.

## ¿Qué es el Ownership?

En Rust, el *sistema de propiedad* u **Ownership** es un sistema en el cual cada
valor en el programa tiene un "dueño". Ese dueño es, por lo general, una 
variable.
Cuando el dueño sale del ámbito de la variable (scope), el valor se libera
automáticamente.

```rust
fn main() {
    let texto = "Mundo".to_string(); // texto "posee" el String
    println!("Hola {texto}!"); 
} // Aquí 'texto' sale del scope y libera los recursos utilizados por el String
```

A diferencia de Go, no se necesita un GC para limpiar la memoria, Rust lo hace 
al salir del scope.

Otro caso ejemplo podría ser:

```rust
fn saludar(nombre: String) {
    println!("Hola, {nombre}");
}

fn main() {
    let nombre = "Arquimedes".to_string();
    saludar(nombre); // estamos cediendo la propiedad, el ownership
    // Eso significa que a partir de aquí ya no es accesible

    // Y este println! dara error
    println!("Nombre no es valido en esta linea: {nombre}");
}
```

El mensaje de error nos dirá que el valor ha sido movido, eso quiere decir que
movimos el dueño de la variable a la función saludar y por tanto ya no podemos
acceder a la misma variable desde `main`, ahora la variable `nombre` pertenece
al scope de `saludar`, `saludar` se ejecuta y la variable deja de ser accesible.

Veremos un poco más de como es posible que esto suceda más adelante, pero 
por ahora quédate con el mecanismo, todo valor tiene un dueño y si movemos el 
dueño de scope dejara de ser accesible.

## Borrowing (Prestamos)

En Go puedes pasar referencias a funciones sin pensar demasiado su duración.
En Rust, el mecanismo de referencias se lo llama Borrowing y tiene algunas 
reglas como por ejemplo: 
- Puedes tener muchas referencias inmutables
- O una sola referencia mutable
- Nunca puedes tener ambas al mismo tiempo

```rust
fn saludar(nombre: &String) {
    println!("Hola, {nombre}");
}

fn main() {
    let nombre = "Arquimedes".to_string();
    saludar(&nombre); // esta siendo prestado, no cedemos la propiedad

    println!("Nombre todavía es valido en este contexto: {nombre}");
}
```

A esto se lo considera un borrow, un prestamo, porque estamos prestando el valor
no estamos otorgando la propiedad entera, solo lo prestamos por un tiempo 
definido.

En este caso le prestamos el `nombre` a la función `saludar`, `saludar` utiliza
la variable, termina el scope de `saludar` y nos devolverá el valor de `nombre`
al scope original, en este caso `main` siendo aún accesible para el resto del 
código que tengamos en la función `main`.

### Borrowing Mutable 

Teniendo en cuenta en el ejemplo anterior encontraremos una imposibilidad, como 
parte de las reglas, no podremos modificar el valor del nombre desde la función 
`saludar`:

```rust
fn saludar(nombre: &String) {
    nombre.push_str(" de Siracusa");
    println!("Hola, {nombre}");
}

fn main() {
    let nombre = "Arquimedes".to_string();
    saludar(&nombre); // aunque lo prestemos, no permite modificarse

    println!("Nombre todavía es valido en este contexto: {nombre}");
}
```

Y si, quizás teniendo en cuenta lo que vimos acerca de 
[Declaración de Variables][variable-declaration] nos demos cuenta de que nos 
falta agregar `mut` en la declaración de la variable `nombre`.

```rust
fn saludar(nombre: &String) {
    nombre.push_str(" de Siracusa");
    println!("Hola, {nombre}");
}

fn main() {
    let mut nombre = "Arquimedes".to_string(); // 👈 agregamos mut
    saludar(&nombre); // aunque lo prestemos, no permite modificarse

    println!("Nombre todavía es valido en este contexto: {nombre}");
}
```

Sin embargo si lo compilamos, encontraremos que tampoco funciona, pero el mismo 
error ya nos dará una solución, 

```sh
   Compiling playground v0.0.1 (/playground)
error[E0596]: cannot borrow `*nombre` as mutable, as it is behind a `&` reference
 --> src/main.rs:2:5
  |
2 |     nombre.push_str(" de Siracusa");
  |     ^^^^^^ `nombre` is a `&` reference, so the data it refers to cannot be borrowed as mutable
  |
help: consider changing this to be a mutable reference
  |
1 | fn saludar(nombre: &mut String) {
  |                     +++
```

El error nos dice que estamos modificando la variable `nombre` pero la variable
no ha sido declarada como mutable en la firma de la función.

La solución se muestra en el mismo mensaje de error:

```sh
help: consider changing this to be a mutable reference
  |
1 | fn saludar(nombre: &mut String) {
  |  
```

Modificaremos la firma para que sea declarado como mutable además hay que 
agregar enviar el dato a la función de forma mutable:

```rust
fn saludar(nombre: &mut String) { // 👈 Lo recibimos como un prestamo mutable
    nombre.push_str(" de Siracusa");
    println!("Hola, {nombre}");
}

fn main() {
    let mut nombre = "Arquimedes".to_string(); // 👈 agregamos mut
    saludar(&mut nombre); // 👈 lo prestamos como mutable

    println!("Nombre ha sido modificado: {nombre}");
}
```

De esta forma ahora estaremos haciendo un prestamo mutable.


### 🚫 Reglas Clave 
1. En cualquier momento, puedes tener:
    - Multiples referencias inmutables, o
    - Una sola referencia mutable
2. Los prestamos viven menos tiempo que la variable original
3. Los prestamos no deben causar data races <sup>[1](#note1)</sup>

Estas reglas las verifica el compilador. 

### ¿Por qué esto importa?

En Go, muchos bugs se ocultan bajo el GC y la falta de control de ownership:
- Accesos a estructuras que ya no deberías usar
- Ejecuciones inesperadas del GC en código sensible al tiempo <sup>[2](#note2)</sup>
- Bugs dificiles de reproducir por condiciones de carrera

Rust pone estas garantías en tiempo de compilación, indicando los problemas 
exactos que tendrás en ejecución y evitando que ocurran.

## Comparaciones rápidas

| Concepto               | Go                        | Rust                          |
| ---------------------- | ------------------------- | ----------------------------- |
| Liberación de recursos | Garbage Collector         | Ownership + Borrowing         |
| Copia de valores       | Implícita                 | Explicita (clone)             |
| Referencias            | Punteros pero sin control | Referencias pero con reglas   |
| Seguridad              | Depende del Programador   | Garantizada por el compilador |
| Coste en tiempo real   | GC puede causar pausas    | Tiempo determinista           |

Seguramente encontraremos más comparaciones a lo largo del libro pero de momento
quedémonos con estas pocas.

---

<sup id="note1">1</sup> Un Data Race o una condición de carrera es un problema 
que ocurre en la programación concurrente, no es importante saberlo aún pero 
veremos más acerca de esto más adelante.

<sup id="note2">2</sup> Casos similares a estos los podemos encontrar en el 
[blog post de Discord][blog-post-discord] donde se explica con lujo de detalle 
los problemas de performance que estuvieron sufriendo, en relación a esto mismo, 
el GC se activaba de manera frecuente y tenían poco control acerca de esto 
debido a que es un proceso automático, es por esto que decidieron migrar a Rust, 
este manejo especifico de como funcionan las cosas les permite optimizar sin 
miedo código muy complejo.

[variable-declaration]: ./../quick-comparisons/variable-declaration.md#mutabilidad
[blog-post-discord]: https://discord.com/blog/why-discord-is-switching-from-go-to-rust