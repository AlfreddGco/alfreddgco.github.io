---
layout: post
title:  "10 Tips al Aprender RL (Aprendizaje Reforzado)"
date:   2024-01-06 13:58:00 -0600
categories: software
---

¡Feliz año nuevo!

Hace un par de semanas termine mi [serie de videos](https://www.youtube.com/playlist?list=PL5gl3i8riyLdmsJ6tMy-cA6wI_T3m-rY-) donde usaba torch y gymnasium en python para entrenar un bípedo autónomo con aprendizaje reforzado y
la arquitectura SAC (Soft Actor Critic). Honestamente casi nunca me toca programar cosas relacionadas a Machine Learning pero he estudiado y entiendo bien la teoría.
Por lo que, en cierta forma, esta experiencia fue un aprendizaje para mi y quisiera compartir algunos tips que logré abstraer que me hubieran servido
para lorgrar terminar el proyecto de manera más rápida y con menos trabas.

Algunos de estos consejos funcionan si apenas estás empezando a programar unas cuántas redes neuronales mientras que otros consejos se podrían considerar
consejos de software en general.

### 1.- Cuida los inputs y outputs de tu red neuronal
Cuidar los inputs y outputs de tu red neuronal es importante, es un paso muy sencillo pero es muy fácil de olvidar cuando estamos empezando.
Asegurate de cuidar las dimnesiones de tus inputs y outputs, la forma en la que codificas la información, y el rango de valores que puedes tener
tanto en input como en outputs. Dependiendo de tu función de activación de la última capa de tu red neuronal, puede ser que algunos rangos de
valores sean imposibles para tu red neuronal darlos como resultados.

Por poner un ejemplo, la función de activación `ReLU` no te puede dar valores negativos, por lo que si durante tu experimento estás esperando
valores negativos en el output, nunca los vas a tener. A veces no es necesario ajustar tus funciones de activación, a veces puede bastar con hacer
algo de regularización después de tu output. Pero entre más embebidas estén las limitantes, mejor para tu experimento.

De la misma forma, en mi experimento, yo interpretaba los outputs (lineales) de mi red neuronal como desviaciones estándar logarítmicas,
los cuales convertía en sólo desviaciones estándar con la función `torch.exp`, pero no sin antes limitar los valores del logaritmo con la función
`torch.clamp` para que, al exponenciar los resultados, tener control del rango de valores que podía obtener:

```python
  log_std = self.log_std_layer(x)
  log_std = torch.clamp(log_std, LOG_STD_MIN, LOG_STD_MAX) # defino valor mínimo y máximo de los logaritmos
  std = torch.exp(log_std) # al deshacerme del logaritmo, sé que mis valores estarán en mi rango deseado
```

### 2.- Cuidado con las redimensiones
En general, no deberías estár haciendo muchas redimensiones de datos de un lago hacia otros, si tu red neuronal está bien diseñada,
debería de darte las dimensiones adecuadas para usarlas como están y mandarlas al agente o a donde sea necesario. Habrá excepciones a
esta regla pero en general no deberíamos de estar haciendo redimensiones extrañas donde estamos agregando o quitando dimensiones.

### 3.- Presta atención a tus funciones de pérdida (loss functions)
Las funciones de pérdida son un punto importante y a veces engañoso por las matemáticas que pueden llegar a implicar, pero recordemos
que nuestra red neuronal va a intentar minimizar esta función, por lo que si la función está mal definida, estaremos optimizando para
las acciones incorrectas.

### 4.- Entiende las matemáticas detrás
Esto se relaciona con el punto anterior, puesto que si no entiendes las matemáticas detrás, será difícil averigüar si tu función de
périda es la correcta, ya que suele ser el valor final después de realizar muchas operaciones con muchos datos, o sea, que esta función
en varios casos se va a derivar de operaciones complejas, y es importante entender estas operaciones para validar el resultado.

### 5.- Estudia el problema y soluciones hechas por otros (así sabrás qué esperar)
Este es un punto muy importante. Mucha gente intentará resolver los problemas por su propia cuenta y llegar a una solución _estado del arte_.
Sin embargo, esta decisión muchas veces nos limita la perspectiva y nos priva de saber qué esperar de nuestro algoritmo. Durante mi experimento,
al estudiar otras soluciones, pude dimensionar lo simple o complejo que era el problema, la cantidad de nueronas que se suelen utilizar para resolverlo,
funciones de activación, rangos razonables para los hiperparámetros, y cuánto entrenamiento requerían las redes neuronales para llegar a un desempeño decente.
Sin conocer todo este tipo de información de antemano, estás intentando resolver el problema casi a ciegas, y agregarte esta limitante muchas veces no te hará aprender
más, sino que te hará tener que hacer un proceso muy repetitivo de experimentación variando hiperparámetros una y otra vez.

En mi caso, aprendí que el agente debía de _empezar_ a caminar aproximadamente a los 1000 juegos con una buena arquitectura de aprendizaje, por lo que
cuando el proceso de aprendizaje superaba los 3000 juegos, y no se veía ningún progreso, sabía que algo estaba mal.

### 6.- Código limpio y buena arquitectura te ayudará a desarrollar y debuggear mejor
El código limpio hará que te repitas menos a la hora de escribir código, por ende, hará que tengas menos código, y por consiguiente, menos bugs.
A su vez, el código limpio de ayudará a construir una buena arquitectura en todo tu proyecto, lo que te dará más flexibilidad para debugger y probar
nuevas cosas, sin tener que hacer cambios significativos a la estructura de tu código. Además, por arquitectura también me refiero a desarrollar
herramientas, métricas, tests, etc. ya que a veces queremos resolver el problema con haber escrito solo unas cuantas piezas de código. Sin embargo, 
cada vez que haya algo útil que podríamos programar, será muy conveniente hacerlo puesto que tendremos más "infraestructura" para atacar el problema.

**Menos código, menos bugs; más código, más infraestructura; más infraestructura, más fácil se vuelve la resolución de problemas.**

### 7.- Ten cuidado con operaciones matriciales
Las operaciones matriciales pueden llegar a ser engañosas, ya que trabajamos con muchos datos y no siempre podemos estar seguros de que cada input
se convirtió en el output correspondiente. Para ello es importante implementar tests y usar un debugger para entender lo que está pasando en aquellos
pasos que son un poco complejos de verificar a plena vista.

En mi caso, tenía un bucle donde una matriz no se estaba actualizando a través de cada iteración, pero como la matriz era una matriz de muchos valores,
muchas veces no pensamos en "ver" o verificar que estas variables tengan los datos correctos y confiamos en que tienen lo que esperamos. El haber usado
un debugger me hubiera servido para darme cuenta de que esta variable no se estaba actualizando a través de iteraciones.

### 8.- No solo prestes atención a las cosas complejas
A diferencia de la regla anterior, también es importante prestarle atención a las cosas simples. Muchas veces por estar trabajando con algoritmos y
matemáticas complejas, y nuestro código no funciona, automáticamente asumimos que el error debe estar en las cosas complejas, pero somos seres
humanos y también nos podemos equivocar en las cosas más simples, y puede que no lo notemos por estar buscando nuestro error en lo complejo
una y otra vez.

### 9.- No gastes tanto tiempo en hiperparámetros
Optimizar hiperparámetros casi nunca va a hacer que nuestro código empiece a funcionar, nos pueden dar unas cuantas mejoras pero actualizar y validar
hiperparámetros suele tomar mucho tiempo, y muchas veces puedes tener mejores resultados por cantidad de esfuerzsos implementando más infraestructura.

En mi caso, pasé un buen timepo intentando optimizar hiperparámetros cuando había mejores cosas en las que podía gastar mi tiempo, como programando una
función que me ayudará a graficar resultados, o mejorando mis redes neuronales agregando encoders y regularización.

### 10.- Avanza paso a paso
Por último, avanzar paso a paso nos ayudará a cometer menos errores y notar fácilmente cuándo es que las cosas dejan de funcionar. Intenta empezar con
la configuración más simple posible y ve escalando el problema desde ahí, porque si empiezas con una configuración compleja y las cosas no funcionan,
tendrás muchos lugares por revisar para encontrar el problema. En general creo que esta es una buena regla para todo campo ingenieríl.

--------
<br/>
_Pequeña nota:_ El haber desarrollado específicamente 10 tips fue mera casualidad. No me gusta cuando los artículos forzan una lista de puntos a un número específico
(supongo que por fines de marketing) porque algunos puntos terminan sintiéndose sobrados. Por ende, espero que cada tip se sienta igual de útil que los demás :)
