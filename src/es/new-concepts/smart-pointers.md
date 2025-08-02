# Smart Pointers y Heap Allocation

En Go, los desarrolladores rara vez se preocupan de donde se almacenan los 
datos en memoria. El lenguaje, junto al recolector de basura (Garbage Collector)
se encarga automáticamente de decidir si una variable vive en el `stack` o en el
`heap`, y se ocupa de liberarla cuando ya no es necesaria. Es un modelo cómodo.
En este capitulo explicaremos que es esto de `stack` y `heap`. 

En Rust, no tenemos Garbage collector como hemos explicado en el capitulo 
[Ownership y Borrowing] y en [Mentalidad y Filosofía].
Es por eso que en su lugar Rust ofrece estos mecanismos, [Ownership y Borrowing]
que garantizan seguridad sin pausas de recolección. Esto también significa
que en ciertas ocasiones, el programador debe decidir explicitamente si un valor 
debe almacenarse en el heap y que hacer con él. En este punto es donde entran en 
juego los `smart pointers`.

## ¿Qué es `stack` y `heap`?

Exten dos grandes regiones de memoria donde se almacenan los datos:
El `stack` (pila) y el `heap` (montículo).

Son dos caras de la misma moneda, son división lógicas que generalmente hace el
sistema operativo con un objetivo en particular para cada uno:

- El `stack`: una memoria rápida, accesible y de tamaño fijo. Aquí se suelen
  almacenar los tipos de datos que tienen un tamaño conocido en tiempo de 
  compilación y un tiempo de vida predecible.
- El `heap`: una región de memoria flexible en la que podemos almacenar datos
  que pueden crecer dinámicamente o que tienen un tiempo de vida complejo o 
  no muy bien definido, algo incierto.

En Go, por ejemplo cuando una variable es retornada desde una función o se 
utiliza por fuera del scope en el que fue inicialmente declarada, el compilador 
puede decidir automáticamente colocarla en el `heap`.
Este proceso se conoce como `escape analysis`. Esto puede ocasionar problemas 
en algunas ocasiones cuando buscamos optimizar.

En Rust por otro lado podemos elegir de forma manual cuando queremos que un 
valor este disponible en el `heap` o en el `stack`.

Cuando almacenamos un valor en el `heap` independientemente del lenguaje de 
programación lo que sucederá es que el valor se crea en el `heap` y almacenamos
la dirección de memoria del `heap` en el `stack`.

### ¿Por qué al almacenar en el `heap` también lo hacemos en el `stack`?

Esto es así porque el `stack` tiene un orden definido, entonces las 
búsquedas son rápidas, sin embargo el `heap` no tiene un orden por lo que todos
los lenguajes asocian el identificador de la variable a una posición en el 
`stack` y a su vez esa posición en el `stack` almacenara la dirección de memoria
para buscar en el `heap`, de esta forma nosotros podemos indexar los valores
en el `heap` y que no sea tan costoso cada vez que debamos llamar una variable
almacenada en el `heap`.

Entonces este es uno de los factores por los cuales el `stack` se considera más
rápido, podemos acceder al valor de forma directa, pero en el `heap` debemos
previamente ir al `stack` y luego con la dirección ir a buscar el valor en el 
`heap` es decir son dos lecturas que hacemos a diferencia del `stack` donde 
hacemos solo una.

No en todos los casos el `heap` es malo, si nosotros tenemos valores dinámicos, 
que cambiaran frecuentemente el `heap` es radicalmente un mejor lugar para
almacenar las variables, eso debido a que el `stack` es menos flexible y 
suele ser más lento para escritura, el `heap`esta pensado para estas 
situaciones.

## ¿Qué es un Smart Pointer?

Un **smart pointer** en Rust es una estructura que se comporta como puntero, 
pero además encapsula lógica adicional. Es decir, no solo es una dirección de 
memoria, sino que posee comportamiento propio, un comportamiento asociado.

Por ejemplo, puede gestionar la propiedad del valor al que apunta, contar 
cuántas referencias activas hay hacia él, o incluso permitir mutabilidad 
controlada en tiempo de ejecución.

Algunos smart pointers comunes en Rust son:
- `Box<T>` que sirve para almacenar valores en el `heap`, hablaremos más de esto 
  en breve
- `Rc<T>` para conteo de referencias en entornos `single thread`
- `Arc<T>` para conteo de referencias en entornos `multi thread`
- `RefCell<T>` para mutabilidad interior con verificación en tiempo de ejecución

Vamos a concentrarnos en `Box` en este capitulo, ya que es el más simple y el 
más útil para introducir el concepto de `heap allocation` y `dynamic dispatch`.

A medida que lo necesitemos iremos viendo otros smart pointers.

## ¿Qué es `Box<T>`?

`Box<T>` es el smart pointer más básico en Rust. Su único uso es mover un valor
al `heap` y permitir acceder a él como si fuera una referencia exclusiva. `Box`
almacena en el `stack`, pero el valor que contiene se almacena en el `heap`.
Cuando la caja sale de alcance, el valor es automático liberado, gracias al 
[ownership] y al RAII.

```rust
fn main() {
  let x = Box::new(42);
  println!("x = {}", x); // Se puede usar el valor de forma directa
}
```

Internamente `Box` implementa los traits `Deref` y `Drop`, dos traits muy 
importantes en los que profundizaremos en los proximos capitulos.

Lo que sucede en este código de ejemplo es que creamos un valor (`42`) y lo 
almacenamos en el `heap` cuando lo pasamos como parámetro a `Box`.
`x` contendra un puntero al valor que ahora estará en el `heap`, por ergonomía
Rust permite que si tenemos que acceder al valor de `x` accedamos de manera 
directa gracias a las reglas de [`dereference coercion`] (trait `Deref`) pero
en realidad lo que estará sucediendo es que estarás interactuando con un 
puntero. Es decir, Rust da una capa de abstracción para no tener que interactuar
con el puntero como tal, no obstante en capítulos mucho más avanzados veremos
que podemos interactuar de forma directa con el puntero.

Ya hemos mencionado en la sección de este capitulo 
[¿Qué es `stack` y `heap`?](#qué-es-el-heap) algunos motivos por los cuales 
querremos usar `Box` y almacenar en el `heap`, pero de forma resumida podriamos
decir que las situaciones son:
- Cuando necesitas una estructura recursiva (Por ejemplo arboles o listas 
  enlazadas)
- Cuando el tamaño de datos es muy muy grande y no queremos copiarlo en cada 
  lugar donde lo debemos utilizar
- Cuando estás trabajando con tipos de tamaño desconocido en tiempo de 
  compilación
- Cuando estás trabajando con tipos desconocidos en tiempo de compilación
- Cuando estas trabajando con APIs que requieren punteros o estructuras 
  dinámicas


A diferencia de punteros normales, Box es seguro, liberara automáticamente
la memoria cuando el valor sale de scope, gracias al [ownership] y al trait 
`Drop`.

Nosotros en el capitulo anterior hablamos acerca de [Genéricos][Genericos]
y vimos acerca del [`static dispatch`][static-dispatch], los beneficios que 
tiene pero también como contraparte vimos que si bien nos ahorra el escribir 
código también puede suceder que termine incrementando el tamaño del binario
eso debido a que se creara una función especifica para cada tipo de dato que
cumpla con el Genérico.

Con `Box` podemos solucionar este problema utilizando `dynamic dispatch`.

## Dynamic Dispatch

En algunas ocasiones, un caso muy típico es querer colecciones de tipos 
dinámicos, por ejemplo una lista de valores que quizás tienen poco en 
común, lo que podemos hacer para ese caso es `dynamic dispatch`.

En Go tenemos el mismo impedimento por defecto, las listas solo pueden contener
un tipo de dato a la vez, pero si se puede generar una interfaz en común para
almacenar valores en la lista:

```go
formas := []Dibujable{
	Circulo{Radio: 5},
	Rectangulo{Ancho: 3, Alto: 4},
}

for _, f := range formas {
	fmt.Println(f.Dibujar())
}
```

De esta forma almacenamos tanto `Circulo` como `Rectangulo` en una única lista.

Cada elemento implementa la interfaz `Dibujable`. Internamente, Go usa punteros 
y tablas de métodos para hacer el dispatch.

En Rust, si quieres tener una lista de distintos tipos, necesitas usar 
`trait objects` de la siguiente forma:

```rust
let formas: Vec<Box<dyn Dibujable>> = vec![
    Box::new(Circulo { radio: 5.0 }),
    Box::new(Rectangulo { ancho: 3.0, alto: 4.0 }),
];

for forma in formas {
    println!("{}", forma.dibujar());
}
```

`Box<dyn Dibujable>` es un trait object. Con esto hacemos `dynamic dispatch`, 
esto quiere decir que el método `dibujar` se resuelve en tiempo de ejecución. 
Rust validara en tiempo de compilación que se cumpla el contrato del `trait` 
pero no creara una función especifica para cada posible parámetro, sino que 
usara una única función. 

Además el `dynamic dispatch` también responde a las reglas del ownership, por lo
que si estamos diseñando una función podemos decidir si mover o hacer un 
prestamo del valor, si hacemos un prestamo/borrow no sera necesario utilizar
`Box` para el parámetro de la función.

```rust
trait Animal {
    fn hablar(&self);
}

struct Perro;
impl Animal for Perro {
    fn hablar(&self) {
        println!("Guau!");
    }
}

fn con_prestamo(animal: &dyn Animal) {
    animal.hablar();
}

fn con_movimiento(animal: Box<dyn Animal>) {
    animal.hablar();
}

fn main() {
    let perro = Perro;

    con_prestamo(&perro);             // Prestamos el perro
    con_movimiento(Box::new(perro));  // Movemos el perro
    // Como usamos ownership perro no es accesible a partir de esta linea
}
```

Por lo que generalmente utilizar un prestamo/borrow nos simplifica el uso
de `trait objects` en el caso de buscar `dynamic dispatch`.

En Go el dynamic dispatch es la única forma de relacionar parámetros con 
interfaces, internamente Go siempre hará dynamic dispatch.

A diferencia de Go, en Rust esto es opcional y explícito. Si no quieres dispatch 
dinámico, puedes usar `impl Trait` o `T: Trait`.



[Ownership y Borrowing]: ./../fundamental/ownership-and-borrowing.md
[Mentalidad y Filosofía]: ./../mindset.md
[ownership]: ./../fundamental/ownership-and-borrowing.md#qué-es-el-ownership
[`dereference coercion`]: https://book.rustlang-es.org/ch15-02-deref#coerciones-implicitas-de-deref-con-funciones-y-metodos
[Genericos]: ./generics-traits-and-static-dispatch.md
[static-dispatch]: ./generics-traits-and-static-dispatch.md#polimorfismo-con-static-dispatch