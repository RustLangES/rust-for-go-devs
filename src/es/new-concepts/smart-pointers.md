# Smart Pointers y Heap Allocation

En Go, los desarrolladores rara vez se preocupan de donde se almacenan los 
datos en memoria. El lenguaje, junto al recolector de basura (Garbage Collector)
se encarga automáticamente de decidir si una variable vive en el stack o en el
heap, y se ocupa de liberarla cuando ya no es necesaria. Es un modelo cómodo.

En Rust, en cambio, no tenemos Garbage collector como hemos explicado en el 
capitulo [Ownership y Borrowing] y en [Mentalidad y Filosofía].
Es por eso que en su lugar Rust ofrece estos mecanismos, [Ownership y Borrowing]
que garantizan seguridad sin pausas de recolección. Esto también significa
que en ciertas ocasiones, el programador debe decidir explicitamente si un valor 
debe almacenarse en el heap y que hacer con él. En este punto es donde entran en 
juego los `smart pointers`.

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

## ¿Qué es `Box<T>`?

`Box<T>` es el smart pointer más básico en Rust. Su único uso es mover un valor
al heap.


[Ownership y Borrowing]: ./../fundamental/ownership-and-borrowing.md
[Mentalidad y Filosofía]: ./../mindset.md