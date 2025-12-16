# Concurrencia y Paralelismo

Quien migra de Go a Rust suele experimentar una sensación recúrrente: 

<div class="quote michael">

> "Esto en Go sería más fácil, qué complicado es esto de Rust".

</div>
<span style="text-align: right; display: block; margin-top: 1em;">
— Un desarrollador de Go (tal vez frustrado y llamado <a href="https://www.linkedin.com/in/michaelcardozam">Michael</a>)
</span>

Esta fricción no es accidental, ni producto de una mala ergonomía del lenguaje,
ni de una falta de madurez del ecosistema. Es la consecuencia directa de una 
decisión filosófica profunda que Rust toma desde su diseño inicial y que Go, 
deliberadamente, evita.

Este capitulo no pretende enseñar APIs, primitivas ni patrones concretos.
Su objetivo es más fundamental, reconfigurar el mindset o el marco mental desde
el cual se entiende la concurrencia. Sin este cambio de perspectiva, cualquier 
intento de escribir código concurrente en Rust termina siendo un ejercicio de 
frustración o, peor aún, de escribir "Go mal traducido".

La concurrencia en Rust es una colección de herramientas poderosas y
ergonómicas, esto es una propiedad que emerge en base a su sistema de tipos como
hemos visto previamente, pero además de eso el enfoque nos permite tener 
un control más fino sobre los aspectos de concurrencia y paralelismo que en Go
son opacos o automáticos. El enfoque de Go es sobre una capacidad operacional
sobre el runtime, esta diferencia explicará muchas cosas a lo largo de este 
capítulo.

De forma resumida lo que veremos es que:

Go prioriza:
- Simplicidad sintáctica
- Bajo costo cognitivo inicial
- Convenciones antes que restricciones

Rust prioriza:
- Exactitud formal
- Eliminación total de comportamientos indefinidos o *undefined behaviors*
- Garantías estáticas sobre el comportamiento del programa
- Diseño explícito del estado y sus transiciones

La consecuencia de estas diferencias es que Go si bien inicialmente es más 
simple en tema de concurrencia, a medida que el sistema crece en complejidad y
tamaño, la falta de restricciones y garantías formales se vuelve un problema
importante.

Pero antes que nada debemos entender que es concurrencia y paralelismo.

# Concurrencia

La concurrencia es la capacidad de un sistema para manejar múltiples tareas en 
progreso al mismo tiempo. No implica necesariamente que estas tareas se ejecuten
simultáneamente en diferentes núcleos de CPU, sino que el sistema puede 
intercalar, coordinar, y orquestar múltiples flujos de ejecución.

Un programa concurrente responde correctamente incluso cuando:

- Varias tareas avanzan a ritmos diferentes.
- Algunas tareas se bloquean (por I/O, locks o esperas)
- El orden de ejecución no es determinista.

La concurrencia es, ante todo, un problema de diseño y exactitud, no de 
rendimiento.

Los modelos de concurrencia pueden difererir entre lenguajes y entornos. Algunos
lenguajes utilizan hilos ligeros (*lightweight threads*), otros usan
*event loops*, y otros emplean *actors* o *coroutines*. Cada modelo tiene sus
propias ventajas y desventajas, y la elección del modelo adecuado depende de los
requisitos específicos de la aplicación.

### Modelo de concurrencia en Go

Go fue diseñado con la concurrencia como uno de sus pilares. Su propuesta se 
basa en tres elementos centrales:

- Goroutines: unidades de ejecución muy livianas
- Channels: mecanismos de comunicación y sincronización
- Scheduler propio: multiplexa goroutines sobre hilos del sistema operativo

Todo esto al estar sobre un runtime gestionado por Go, permite que el lenguaje
ofrezca una experiencia de concurrencia sencilla, no exactamente la más 
performante pero bastante buena, sin embargo como dijimos el sistema de tipos 
tiene mucho que ver aquí y puede ocasionar problemas debido a la falta de 
restricciones y es por eso que uno de los slogans de Go es:

> Do not communicate by sharing memory; instead, share memory by communicating

<span style="text-align: right; display: block; margin-top: 1em;">
— <a href="https://go.dev/doc/effective_go#concurrency">Effective Go</a>
</span>



Por estas faltas de restricciones podemos caer en varios problemas, esto es 
un problema estructural en el enfoque de Go, al ser tan permisivo, el lenguaje 
no puede evitar que cometamos errores comunes en concurrencia como:
- Data races
- Uso incorrecto de memoria compartida
- Deadlocks
- Starvation

Y al no tener garantías estáticas, estos problemas solo se detectan en tiempo de
ejecución, lo que puede llevar a fallos impredecibles y difíciles de depurar.

Go agrega un detector de data races `-race` que ayuda a identificar estos problemas en
tiempo de ejecución, pero esto no es una solución perfecta:

- Es opcional
- Costoso
- Dinámico (detecta lo que ocurre en runtime, no lo que podría ocurrir)

Es decir, Go deja escribir código concurrente incorrecto con total naturalidad 
debido a que estas herramientas que da el lenguaje no son suficientes para 
evitar errores comunes.

Además otro problema frecuente que tenemos cuando escribimos código concurrente 
en Go es que al usar channels de forma recurrente para comunicar goroutines 
podemos como mencionamos antes caer en deadlocks o starvation silenciosos.

Y como ultimo un problema que es muy comentado por grandes empresas es que la 
concurrencia muchas veces da problemas al ser tan implicito y opaco el lenguaje.
Ya hemos hablado del caso de Discord por dar un ejemplo pero también hay casos
como el de 
[Grab](https://engineering.grab.com/counter-service-how-we-rewrote-it-in-rust).

El Scheduler de Go es automático y no da control sobre como se asignan las 
goroutines a los hilos del sistema operativo, toma decisiones deliberadamente,
no nos da control sobre la afinidad, wake ups, prioridades, ni planificación, 
el rendimiento puede variar drasticamente entre versiones del runtime y 
debuggear estos problemas de concurrencia complejos es notoriamente difícil.


### Modelo de concurrencia en Rust

Rust aborda la concurrencia desde una filosofía bastante diferente, radicalmente
distinta, hacer que el código incorrecto sea imposible de escribir es una de las
metas principales del lenguaje y esto se extiende a la concurrencia.

La concurrencia no es más que otra propiedad del sistema de tipos de Rust...

¿Suena raro verdad? Pero es así, la concurrencia en Rust no es magia, ni es un
runtime especial, ni un scheduler automático, es una consecuencia directa del
sistema de tipos y las garantías que este ofrece.

Ya hemos hablado un poco acerca de los traits pero no hemos profundizado sobre 
algunos de ellos, especificamente el `Send` y el `Sync`. Rust en si mismo es 
completamente tonto, no sabe nada de concurrencia, pero el sistema de tipos 
define dos traits especiales que son implementados automáticamente por el 
compilador:
- `Send`: indica que un tipo puede ser transferido de un hilo a otro de forma 
  segura.
- `Sync`: indica que un tipo puede ser accedido desde múltiples hilos de forma 
  segura.

Estos traits son la base sobre la cual se construye la concurrencia en Rust. El
compilador utiliza estos traits para garantizar que el código concurrente sea
seguro en tiempo de compilación, evitando data races y otros problemas comunes.

Rust tardo mucho tiempo en llegar a este punto, y la razón es que el diseño de 
estos traits y las garantías que ofrecen es extremadamente complejo, pero el
resultado es un sistema de concurrencia que es seguro por diseño, sin necesidad
de un runtime especial o un scheduler automático.

Además de eso se ha hablado mucho de asincronía en Rust, lo cual profundizaremos
en otro capítulo, pero justamente este modelo de concurrencia del que Rust
hace gala es la base que permite adoptar distintos modelos de concurrencia, 
paralelismo y asincronía sin comprometer la seguridad del sistema.

Ampliaremos un poco la lista de cosas que Rust tiene en cuenta para el sistema 
de tipos, ya hemos visto algunas de ellas en capítulos anteriores pero 
actualizando las ideas serian:
- Ownership
- Borrowing
- Lifetimes
- Traits como `Send` y `Sync`

Esto hace que si tu programa compila, una enorme de clase de errores de 
concurrencia es matemáticamente imposible que ocurran en ejecución.

Estas propiedades no son convenciones, ni documentación, son restricciones 
formales que el compilador aplica y verifica en tiempo de compilación.
Son verificadas por el compilador y no pueden ser ignoradas o deshabilitadas.

## Memoria compartida

Rust no prohibe la memoria compartida, pero la hace segura. Nosotros ya hemos
visto algunos pequeños smart pointers pero claro, esos smart pointers no son
concurrentes, Rust mediante los traits `Send` y `Sync` distingue entre 
Smartpointers pensados para uso en un solo hilo y aquellos pensados para uso
concurrente.

En este capitulo nos centraremos en algunos de ellos:
- `Arc<T>`: Atomic Reference Counted pointer, permite compartir datos entre 
  múltiples hilos de forma segura, veremos que quizás tiene alguna similitud con
  smartpointer `Rc<T>` que ya vimos en capítulos anteriores.
- `Mutex<T>`: Proporciona exclusión mutua para proteger datos compartidos,
  asegurando que solo un hilo pueda acceder a los datos a la vez. Además lo 
  compararemos con su contraparte en Go, los `sync.Mutex`.
- `RwLock<T>`: Permite múltiples lectores o un solo escritor, optimizando el
  acceso concurrente a datos que son leídos frecuentemente pero escritos
  ocasionalmente. También lo compararemos con su contraparte en Go, los
  `sync.RWMutex`.

Estos smart pointers y primitivas de sincronización son fundamentales para
escribir código concurrente seguro en Rust, y veremos cómo su diseño y uso se
diferencian de las primitivas similares en Go.

Veremos algunos detalles fundamentales como que cualquier `struct` en Go puede
llegar a ser expuesto a concurrencia a pesar de que no haya sido pensado para 
eso, mientras que en Rust veremos como sus mecanismos lo impiden dandonos 
más seguridad a la hora de escribir y usar concurrencia.

## Modelos de concurrencia

Antes de hablar de paralelismo, es importante hablar sobre los distintos modelos 
de concurrencia, otro punto a favor de Rust. No toda concurrencia es igual, ni 
todo paralelismo se implementa de la misma forma. Existen múltiples de modelos 
de ejecución, cada uno con implicancias profundas en rendimiento, 
previsibilidad, seguridad y complejidad mental.

Generalmente hay dos factores:

- Como se cede el control de ejecución
- Quien decide cuando una tarea se suspende o continua

A partir de allí analizaremos qué modelos de concurrencia existen, cual adopta 
Go y cómo Rust permite implementarlos de forma segura y eficiente.

### Concurrencia preemptiva

En la concurrencia preemptiva, una tarea puede ser interrumpida en cualquier 
punto por el scheduler, sin cooperación explicita del código que se esta 
ejecutando. 

Para destacar algunas cosas este modelo nos permite que el schedule decida 
cuándo suspender una tarea.
Las interrupciones pueden ocurrir en cualquier momento y requiere mecanismos de
sincronización defensivas (locks y atomics) para proteger el acceso a recursos
compartidos.

Algunos casos de uso comunes incluyen:
- Threads del sistema operativo
- Pthreads
- Goroutines en Go (a nivel del runtime)

El lado bueno de este modelo es que permite aprovechar bien CPUs multinúcleo y 
no depender de que el programador "ceda" el control de ejecución, pero 
también puede causar mayor complejidad mental, condiciones de carreras difíciles
de razonar y overhead por cambios de contexto frecuentes 
(sincronización constante).

### Concurrencia cooperativa

En la concurrencia cooperativa, las tareas deben ceder el control de ejecución
de forma explícita en puntos determinados. Esto significa que una tarea solo 
puede ser suspendida cuando el código lo permite, generalmente mediante llamadas
a funciones específicas. De forma resumida estamos queriendo decir que una tarea
decide cuando suspenderse a si misma.

Las tareas tienen control explicitamente, los puntos de suspensión son conocidos 
y delimitados y el flujo de ejecución es más predecible. Solo debemos hacer foco
en esos puntos de suspensión para razonar sobre concurrencia.

Algunos casos de uso comunes incluyen:
- Coroutines
- Async/await (como en Rust)
- Fibers cooperativos (implementaciones específicas)
- Event loops (Node.js, etc)

El lado bueno de este modelo es que reduce la complejidad mental, facilita el
razonamiento sobre el flujo de ejecución y minimiza el overhead por cambios de
contexto. Pero también puede limitar el aprovechamiento de CPUs multinúcleo y
requiere que el programador sea consciente de ceder el control en los puntos
adecuados. Tenemos estados intermedios bien definidos y controlados, ayudando 
a que sea sencillo de razonar.

Lo malo es que una tarea mal diseñada o escrita puede bloquear todo el sistema y
requiere una diciplina estricta para asegurar que las tareas cedan el control de
forma adecuada.

### Concurrencia hibrida

Algunos sistemas adoptan un enfoque híbrido, combinando elementos de ambos
modelos. Por ejemplo, preemptivos a nivel threads y cooperativos dentro de cada
thread. Este es el modelo más adoptado en sistemas modernos de alto rendimiento.

### Otras perspectivas

Otras formas de lograr concurrencia son modelos en que se centran en aspectos 
como el paso de mensajes (actor model) donde las tareas no comparten memoria, 
sino que se comunican enviando mensajes entre sí. Este modelo es popular en 
lenguajes como Erlang y frameworks como Akka.

Casos de implementación son el modelo de actores por ejemplo pero también
podrían ser canales, o CSP (Communicating Sequential Processes) como en Go.

Esto ayuda a evitar data races y un modelo mental más limpio, pero como 
desventaja hay que mencionar que son modelos que pueden tener overhead por la
la necesidad de serializar/deserializar mensajes y gestionar colas de mensajes,
hacer copias de datos, o en algunos casos como los canales puede llevar a
deadlocks o difícil backpressure (gestión de flujo).

# Paralelismo

A diferencia de la concurrencia el paralelismo se refiere a la ejecución 
simultánea real de multiples tareas, típicamente en múltiples núcleos de CPU. El
paralelismo es una forma específica de concurrencia que busca aprovechar al
máximo los recursos de hardware disponibles para mejorar el rendimiento.

Un programa puede ser concurrente sin ser paralelo, pero para ser paralelo debe 
ser concurrente. 

De la misma forma que podemos encontrar distintos modelos de concurrencia, también
existen distintos enfoques para implementar paralelismo, cada uno con sus propias
ventajas y desventajas.

Los más conocidos son:
- Ray
- SIMD (Single Instruction, Multiple Data)
- MapReduce
- GPU Computing

Y podemos también encontrar algunos tipos de paralelismos enfocados en las 
tareas específicas como work-stealing schedulers, pipelining, y data parallelism.

# ¿Qué modelos adopta Go?

Go adopta un modelo muy específico con fuertes implicancias en el paralelismo y 
la concurrencia. Go adopta preemptiva a nivel de goroutines y cooperativa dentro 
de cada goroutine. Esto significa que el scheduler de Go puede interrumpir y 
reprogramar goroutines en cualquier momento, pero dentro de cada goroutine, el 
código debe ceder el control de ejecución de forma explícita.

Las Goroutines son una implementación de Coroutines ligeras, solemos 
comunicarnos mediante channels y poseemos un shceduler global opaco.

Al usar channels también estamos diciendo que adopta paso de mensajes (aunque
permite memoria compartida y no es lo recomendado). 

Y sobre el paralelismo Go permite paralelismo pero tienes muy poco control sobre
cómo se asignan las goroutines a los hilos del sistema operativo, el scheduler
decide automáticamente cómo distribuir las goroutines entre los núcleos de CPU
disponibles, lo que puede llevar a variaciones en el rendimiento y la
previsibilidad.

# ¿Qué modelos adopta Rust?

Rust no impone un modelo específico de concurrencia o paralelismo, sino que
proporciona las herramientas necesarias para implementar una variedad de modelos
de forma segura y eficiente.

Todos los modelos de concurrencia y paralelismo mencionados hasta ahora son
adoptables en Rust. 

Esto es una de las diferencias claves entre Rust y Go en tema de rendimiento,
si bien Go tiene un rendimiento bueno gracias a los modelos que adopta, no son
optimos para todas las situaciones y para agilizar el proceso de pasaje de 
mensaje usan memoria compartida de forma implícita pero terminan repercutiendo 
en distintos aspectos de la seguridad y estabilidad del programa.

Y al ser tan abierto Rust permite explorar distintos modelos y enfoques, que 
ni siquiera hemos mencionado en este capitulo porque pueden resultar mucho más 
avanzados, en Go al tomar decisiones desde el propio lenguaje se limita mucho
la forma de trabajar.

Profundizaremos en algunos de los modelos de concurrencia en capitulos venideros
y veremos como implementar algunos modelos de paralelismo como SIMD y Rayon 
mucho más adelante.


# Fearless Concurrency

Rust se toma muy en serio este tema debido a que es uno de los ejes centrales 
por los cuales uno busca adoptar Rust en lugar de otros lenguajes. 
La concurrencia sin miedo (*Fearless Concurrency*) es un concepto que
resume la filosofía de Rust en este ámbito. Un problema que en muchos lenguajes
no deja dormir a los desarrolladores, en Rust el problema se aborda con 
tranquilidad.

Veremos que todas las desventajas que nombramos quedan muy claras gracias al 
sistema de tipos, y que escribir código concurrente en Rust es una experiencia
mucho más segura y predecible.

Entender estas diferencias no hace que Rust sea más fácil necesariamente, pero
si hace que deje de parecer arbitrario. Cada restricción, cada regla, cada error 
del compilador, cada mecanismo tiene una razón de ser y cada exigencia responde
a una premisa simple, la exactitud en la concurrencia no es negociable.








