# Mentalidad y Filosofía

Cuando empezamos a trabajar con Go, seguramente notemos que es un lenguaje 
simple y directo. Go actualmente tiene 32 palabras reservadas, lo que lo hace
un lenguaje fácil de aprender y entender. ¿Cuántas palabras reservadas
tiene Rust? Aproximadamente entre 40 y 53. Es un rango porque hay algunas 
palabras reservadas sin uso actualmente y esto puede ir variando dependiendo de
las ediciones de Rust. Esto lo hace un lenguaje relativamente más
complejo que Go. Sin embargo, si lo comparamos con otros lenguajes terminariamos
llegando a la conclusión de Rust es un lenguaje de mitad de tabla.
Más keywords que Go y C, pero con muchas menos que C++, Java o Swift.
Consiguiendo así un equilibrio entre simplicidad y poder, tan poderoso como C++,
pero con una sintaxis más simple y fácil de entender.

Hay que comenzar a pensar en Rust como un lenguaje que comenzó a ser diseñado
para ser un lenguaje de sistemas, pero que ha evolucionado para ser un lenguaje
de propósito general. Esto significa que, aunque Rust es muy adecuado para
programación de sistemas, también se puede utilizar para una amplia variedad
de aplicaciones, desde desarrollo web hasta programación de juegos.

Rust no tiene un recolector de basura. Pero no te preocupes, esto no significa
que tengas que gestionar la memoria manualmente como en C o C++. ¿O sí?
Rust utiliza un sistema de gestión de memoria propio que se basa en conceptos 
como el ownership (propiedad), borrowing (préstamo) y lifetimes 
(tiempos de vida).

Esta gestión de memoria que propone Rust es casi automática, pero no es
completamente automática como en Go. Esto significa que Rust te da un control
fino sobre la memoria, lo que te permite escribir código más eficiente y
seguro. 

Decimos "casi automática" porque es posible que cuando escribas código en Rust
termines no pensando en la memoria, es un concepto muy de fondo que en la 
mayoría de los casos no tendrás que pensar. Pero en algunos casos, como cuando
estés escribiendo código de alto rendimiento o cuando estés trabajando con
código de bajo nivel, es posible que tengas o que quieras tenerlo en cuenta y 
puedas hacerlo. Rust te da las herramientas para hacerlo, pero no te obliga a
hacerlo. Esto es lo que hace que Rust sea un lenguaje tan poderoso y flexible.
Esto en algunas ocasiones puede resultar extremadamente útil, como se menciona
en el blog post de Discord 
[Por qué Discord cambia Go por Rust][why-discord-is-switching-to-rust], donde se 
explica que el gestión de la memoria en Go fue determinante para que decidieran 
reescribir su backend en Rust.

Go es un lenguaje que propone una forma de pensar un poco diferente a la de 
Rust.
Las comparaciones entre Go y Rust son bastante comunes. Go es un lenguaje que
inicialmente se presentaba como un posible lenguaje de sistemas, sin embargo
al poco tiempo Rob Pike, uno de los creadores de Go, dijo que [Go no es un
lenguaje de sistemas][rob-pike-go-is-not-a-systems-language]. Esta aclaración
fue importante en su momento, ya que Go fue comparado con C muchas veces sin
embargo, jugaban en dos ligas diferentes. Go es un lenguaje de alto nivel que
con el paso del tiempo fue dando más y más herramientas para poder
escribir código de alto rendimiento. Rust por otro lado, es un lenguaje que 
ha sido diseñado desde el principio con esas bases y se ha vuelto más sencillo
con el tiempo, lo cual le ha dado la oportunidad de competir en un nicho donde
hay muy pocos lenguajes, solucionando problemas desde sus bases.

Rust es un lenguaje con bases muy solidas, el núcleo del lenguaje es realmente
pequeño y simple, pero Rust provee una gran cantidad de abstracciones que han
sido añadidas a lo largo del tiempo. Estas abstracciones son las que 
te permiten escribir código de alto nivel.

Por ultimo, es importante mencionar el origen de los Garbage Collectors
(GC) y cómo Rust se diferencia. Los GC fueron creados para resolver
el problema de la gestión de memoria en lenguajes de alto nivel, escribir código
con gestión manual de la memoria puede ser complicado. Los GC tienen un costo en 
términos de rendimiento y latencia. Rust, se ha vuelto famoso porque no tiene un 
GC, sin embargo, ha logrado resolver el problema de la gestión de memoria sin 
necesidad de un GC, y todo gracias a este enfoque de memoria "casi automática".
Esto es un hito importante en la evolución de los lenguajes de programación, ya 
que es el primer lenguaje de programación que ha logrado resolver el problema de
la gestión de memoria sin un GC.

Profundizaremos en estos conceptos más adelante, pero en resumen, Rust al igual
que Go, permite escribir código de alto nivel.
Rust sin GC, lo que lo vuelve un lenguaje más performante y con un sistema
que permite garantizar la seguridad de la memoria. 


 [why-discord-is-switching-to-rust]: https://discord.com/blog/why-discord-is-switching-from-go-to-rust
 [rob-pike-go-is-not-a-systems-language]: https://youtu.be/ZQR32nTVF_4?t=407