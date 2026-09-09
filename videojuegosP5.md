# Funciones útiles para videojuegos con p5.js

- [Funciones útiles para videojuegos con p5.js](#funciones-útiles-para-videojuegos-con-p5js)
  - [Colisiones](#colisiones)
    - [Entre rectángulos](#entre-rectángulos)
    - [Entre círculos](#entre-círculos)
  - [Máquinas Finitas de Estados (FSM)](#máquinas-finitas-de-estados-fsm)
    - [Método 1: Máquina de estados simple (Basada en variables)](#método-1-máquina-de-estados-simple-basada-en-variables)
    - [Método 2: Máquina de estados avanzada (Basada en objetos y funciones)](#método-2-máquina-de-estados-avanzada-basada-en-objetos-y-funciones)

---

## Colisiones

### Entre rectángulos

Para detectar esta colisión se utiliza el algoritmo **AABB** (Axis-Aligned Bounding Box). La lógica consiste en verificar si existe un espacio libre en cualquiera de los cuatro lados entre los dos rectángulos. Si existe al menos un espacio, es físicamente imposible que se estén tocando.

```javascript
function detectarColisionRect(a, b) {
  // 1. Detectar cuando NO colisionan evaluando los 4 bordes
  const noColisionan = (
    a.x + a.w <= b.x ||  // 'a' está totalmente a la izquierda de 'b'
    a.x >= b.x + b.w ||  // 'a' está totalmente a la derecha de 'b'
    a.y + a.h <= b.y ||   // 'a' está totalmente por encima de 'b'
    a.y >= b.y + b.h      // 'a' está totalmente por debajo de 'b'
  );

  // 2. Invertir el booleano
  return !noColisionan;
}

```

Una versión súper comprimida:

```javascript
function detectarColisionRect(a, b) {
  return !(a.x + a.w < b.x || a.x > b.x + b.w || a.y + a.h <= b.y || a.y >= b.y + b.h);
}
```

**Ejemplo de uso:**

```javascript
let jugador = { x: 10, y: 10, ancho: 50, alto: 100 };
let enemigo = { x: 40, y: 50, ancho: 50, alto: 50 };
let moneda = { x: 200, y: 200, ancho: 20, alto: 20 };

console.log(detectarColisionRect(jugador, enemigo)); // true
console.log(detectarColisionRect(jugador, moneda));  // false

```

> **Nota sobre los bordes:** Si utilizas `<=` y `>=`, significa que si dos rectángulos están exactamente pegados borde con borde (pero sin superponerse), la función dirá que **no** colisionan. Si necesitas que el contacto exacto de los bordes cuente como colisión, simplemente quita el `=` y deja solo `<` y `>`.

### Entre círculos

Para detectar colisiones entre círculos se utiliza el **Teorema de Pitágoras** para calcular la distancia entre sus centros. Si esa distancia es mayor a la suma de sus radios, entonces hay un espacio libre entre ellos y no se tocan.

Para optimizar el rendimiento (algo muy común en el desarrollo de videojuegos), es mejor comparar las distancias elevadas al cuadrado. Esto evita usar la función `Math.sqrt()`, la cual consume más recursos de procesamiento.

```javascript
function detectarColisionCirculos(a, b) {
  // Diferencia de posiciones en los ejes X y Y
  const dx = a.x - b.x;
  const dy = a.y - b.y;

  // Distancia entre los centros al cuadrado (a^2 + b^2 = c^2)
  const distanciaCuadrada = (dx * dx) + (dy * dy);

  // Suma de los radios al cuadrado
  const sumaRadios = a.r + b.r;
  const sumaRadiosCuadrada = sumaRadios * sumaRadios;

  // 1. Detectar cuando NO colisionan 
  // (la distancia entre centros es mayor o igual a la suma de sus radios)
  const noColisionan = distanciaCuadrada >= sumaRadiosCuadrada;

  // 2. Invertir el booleano
  return !noColisionan;
}

```

**Ejemplo de uso:**

```javascript
let jugador = { x: 50, y: 50, r: 20 };
let enemigo = { x: 80, y: 50, r: 20 };
let proyectil = { x: 200, y: 200, r: 5 };

console.log(detectarColisionCirculos(jugador, enemigo));   // true (la distancia es 30, la suma de radios es 40)
console.log(detectarColisionCirculos(jugador, proyectil)); // false 

```

> **Nota:** Al igual que con los rectángulos, el uso de `>=` en `distanciaCuadrada >= sumaRadiosCuadrada` significa que si los círculos se rozan exactamente en el borde, la función dirá que **no** colisionan. Si quieres que ese roce exacto cuente como colisión, cámbialo por un simple `>`.

---

## Máquinas Finitas de Estados (FSM)

En el desarrollo de videojuegos, una Máquina Finita de Estados (FSM por sus siglas en inglés) nos permite controlar en qué "pantalla" o "momento" se encuentra el juego (por ejemplo: menú de inicio, jugando, pantalla de victoria o derrota). Solo puede existir un estado activo a la vez.

Aquí presentamos dos formas de implementarlo: una sencilla para proyectos rápidos y otra más robusta para juegos que requieren transiciones suaves y mejor organización.

### Método 1: Máquina de estados simple (Basada en variables)

Este es el enfoque más directo. Utilizamos una variable de texto (String) para guardar el nombre del estado actual y usamos un bloque de condicionales (`if / else if`) dentro de la función `draw()` para decidir qué funciones ejecutar.

```javascript
// 1. Declarar el estado inicial
let estado = "intro";

function setup() {
  createCanvas(400, 400);
}

// 2. El Game Loop evalúa el estado constantemente
function draw() {
  if (estado === "intro") {
    pantallaIntro();
  } else if (estado === "juego") {
    pantallaJuego();
  } else if (estado === "fin") {
    pantallaFin();
  }
}

// 3. Funciones separadas para cada estado
function pantallaIntro() {
  background(50, 150, 200);
  text("Pantalla de Inicio (Clic para jugar)", 100, 200);
  
  // Condición para cambiar de estado
  if (mouseIsPressed) {
    estado = "juego";
  }
}

function pantallaJuego() {
  background(50, 200, 100);
  text("¡Jugando! (Presiona cualquier tecla para perder)", 60, 200);
  
  // Condición para cambiar de estado
  if (keyIsPressed) {
    estado = "fin";
  }
}

function pantallaFin() {
  background(200, 50, 50);
  text("Fin del juego", 150, 200);
}

```

**¿Cómo usarlo?**
Simplemente reasigna el valor de la variable `estado` en cualquier momento para saltar de una pantalla a otra de forma inmediata.

### Método 2: Máquina de estados avanzada (Basada en objetos y funciones)

A medida que los juegos crecen, mezclar la lógica (actualizar posiciones, calcular tiempos) con el dibujo (poner colores y formas) puede volverse caótico. Este método utiliza un objeto `Estado` que almacena dos funciones separadas: `upd` (update/lógica) y `drw` (draw/dibujo).

Esta estructura es ideal porque facilita la creación de transiciones fluidas (como fundidos a negro) entre diferentes escenas.

```javascript
// 1. Objeto que almacenará las funciones de la escena actual
let Estado = { upd: null, drw: null };

function setup() {
  createCanvas(400, 400);
  escenaIntro(); // Iniciar cargando la primera escena
}

function draw() {
  // Calculamos el tiempo transcurrido (Delta Time) en segundos
  let dt = deltaTime / 1000.0;
  
  // Ejecutamos la lógica y luego el dibujo del estado actual
  if (Estado.upd) Estado.upd(dt);
  if (Estado.drw) Estado.drw(dt);
}

// 2. Definición de escenas
function escenaIntro() {
  // Asignamos la lógica de esta escena
  Estado.upd = function(dt) {
    if (mouseIsPressed) escenaJuego(); // Transición a la siguiente escena
  };

  // Asignamos el dibujo de esta escena
  Estado.drw = function(dt) {
    background(30);
    fill(255);
    text("ESCENA 1: INTRO (Clic para avanzar)", 100, 200);
  };
}

function escenaJuego() {
  Estado.upd = function(dt) {
    if (keyIsPressed) escenaIntro(); // Transición de regreso
  };

  Estado.drw = function(dt) {
    background(150, 80, 80);
    fill(0);
    text("ESCENA 2: JUEGO (Tecla para volver)", 100, 200);
  };
}

```

**¿Cómo usarlo?**
Para crear un nuevo estado, simplemente creas una función (como `escenaJuego`) y por dentro sobrescribes `Estado.upd` y `Estado.drw` con lo que necesites hacer. Cuando llamas a esa función principal, el motor del juego automáticamente comenzará a ejecutar las nuevas reglas y gráficos en el siguiente ciclo de `draw()`.