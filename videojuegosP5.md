# Funciones útiles para videojuegos con p5.js

## Colisiones

### Entre rectángulos

Para detectar esta colisión se utiliza el algoritmo **AABB** (Axis-Aligned Bounding Box). La lógica consiste en verificar si existe un espacio libre en cualquiera de los cuatro lados entre los dos rectángulos. Si existe al menos un espacio, es físicamente imposible que se estén tocando.

```javascript
function detectarColisionRect(a, b) {
  // 1. Detectar cuando NO colisionan evaluando los 4 bordes
  const noColisionan = (
    a.x + a.ancho <= b.x ||  // 'a' está totalmente a la izquierda de 'b'
    a.x >= b.x + b.ancho ||  // 'a' está totalmente a la derecha de 'b'
    a.y + a.alto <= b.y ||   // 'a' está totalmente por encima de 'b'
    a.y >= b.y + b.alto      // 'a' está totalmente por debajo de 'b'
  );

  // 2. Invertir el booleano
  return !noColisionan;
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
  const sumaRadios = a.radio + b.radio;
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
let jugador = { x: 50, y: 50, radio: 20 };
let enemigo = { x: 80, y: 50, radio: 20 };
let proyectil = { x: 200, y: 200, radio: 5 };

console.log(detectarColisionCirculos(jugador, enemigo));   // true (la distancia es 30, la suma de radios es 40)
console.log(detectarColisionCirculos(jugador, proyectil)); // false 

```

> **Nota:** Al igual que con los rectángulos, el uso de `>=` en `distanciaCuadrada >= sumaRadiosCuadrada` significa que si los círculos se rozan exactamente en el borde, la función dirá que **no** colisionan. Si quieres que ese roce exacto cuente como colisión, cámbialo por un simple `>`.
