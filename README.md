# Contra la Casa — Blackjack con Visión Estadística

Proyecto para **Probabilidad, Estadística y Métodos Numéricos** · Uniempresarial.

Un blackjack (21) contra una IA en el que **la matemática es la mecánica principal, no código oculto**.
Mientras juegas, un HUD lateral recalcula en tiempo real —con las fórmulas de los apuntes— la
frecuencia relativa, la probabilidad condicional, la probabilidad total, el Teorema de Bayes y
cómo se estiman esas probabilidades por simulación de Monte Carlo.

El sitio tiene **dos pestañas**: **Mesa** (el blackjack, con soporte para dividir manos —*split*—
cuando te reparten un par) y **Academia**, una sección de práctica con calculadora del principio de
conteo, un diagrama de árbol explorable y 16 ejercicios resueltos del documento de clase con
validación de respuesta. Ver «La Academia» más abajo.

Además puedes **elegir en qué casino del mundo juegas** — Bogotá, Las Vegas, Montecarlo o Macao —
cada uno con su propia piel visual (colores, fondo, tipografía) para cambiar el ambiente de la mesa.
Las reglas y probabilidades son las mismas en las 4 sedes. Ver «Los casinos del mundo» más abajo.

**Jugar en línea:** https://alejo000111.github.io/Casino-Calculo/

**Jugar sin conexión:** abre `index.html` en cualquier navegador (doble clic). No requiere instalar
nada, ni servidor.

Artifact: https://claude.ai/code/artifact/347ccffd-b078-4234-a87d-082d8eca78a7

---

## Cómo se juega

| Acción | Botón / tecla | Qué hace |
|---|---|---|
| Elegir apuesta | fichas 10 / 25 / 50 / 100 | Solo antes de repartir |
| Repartir | `Repartir` · `D` | Reparte 2 cartas a cada uno (la 2.ª del crupier queda oculta) |
| Pedir carta | `Pedir` · `H` | Suma una carta a tu mano; si pasas de 21 pierdes |
| Plantarse | `Plantarse` · `S` | Cede el turno; el crupier juega hasta 17 |
| Doblar | `Doblar` | Duplica la apuesta, recibes 1 carta y te plantas |
| Dividir | `Dividir` · `P` | Si tus 2 cartas valen igual, las separa en 2 manos independientes (cobra apuesta extra) |
| Denunciar anomalía | `Denunciar anomalía` | Acusas al crupier de tramposo usando el posterior de Bayes |

Reglas: una sola baraja de 52, **el crupier se planta en 17** (incluido 17 "suave"),
blackjack natural paga **3 : 2**, empate devuelve la apuesta. Son las mismas en las 4 sedes —
ver la sección siguiente.

### Dos interruptores para la sustentación

- **Modo sustentación** — despliega debajo de cada panel la fórmula *desarrollada con los números
  actuales* (sustitución paso a paso). Es lo que el jurado necesita ver.
- **Modo QA** — panel de pruebas: fijar el crupier como TRAMPOSO / LIMPIO, forzar que gane o pierda
  la próxima mano, rebarajar y reiniciar. Muestra el estado real oculto del crupier para verificar
  que el HUD coincide.

---

## La estadística, panel por panel

| # | Tema de los apuntes | En el juego |
|---|---|---|
| 1 | Espacio muestral, evento, **frecuencia relativa** `P(A) = casos favorables / casos posibles`, complemento `Aᶜ` | Panel 1: dado tu total, cuenta las cartas del mazo que **no** te pasan de 21 sobre el total de cartas no vistas. Fracción y porcentaje en vivo. |
| 2 | **Probabilidad condicional** `P(A\|B) = P(A∩B) / P(B)`, espacio muestral que se reduce | Panel 2: al ver la carta del crupier (condición B) calcula `P(Blackjack del crupier \| carta visible)` con el espacio reducido, y un medidor de riesgo por colores. |
| 3 | **Probabilidad total** `P(D) = Σ P(Bᵢ)·P(D\|Bᵢ)` sobre una partición | Panel 2 (sustentación): `P(crupier se pasa)` se descompone según la carta oculta (As / 2–6 / 7–9 / 10) y se suma ponderado. |
| 4 | **Teorema de Bayes** `P(A\|D) = P(A)·P(D\|A) / P(D)`, árboles de probabilidad | Panel 3: cada mano que resuelve el crupier actualiza `P(Tramposo)` con Bayes, encadenando el posterior como nueva previa. Árbol dibujado en pantalla. |
| 5 | **Conteo**: permutaciones `V(n,k) = n!/(n−k)!` y combinaciones `C(n,k) = n!/[(n−k)!k!]` | Panel 4: `C(52,2) = 1326` manos iniciales, `V(52,2) = 2652` repartos ordenados, y `C(N,2)` con las cartas que quedan. Incluye el ejemplo de lotería `C(50,5)`. |
| 6 | **Simulación / estimación (ley de los grandes números)** | Panel 5 *(nuevo)*: explica qué hace la simulación de Monte Carlo de n=2000 manos, muestra una gráfica de convergencia (la estimación de `P(crupier se pasa)` recalculada cada cierto número de manos, acercándose a la línea final) y el error estándar / intervalo de confianza al 95% de la estimación. |

Parámetros del modelo de Bayes (tomados de los apuntes):
`P(T) = 0.10`, `P(N) = 0.90`, `P(G|T) = 0.60`, `P(G|N) = 0.30`.

El motor de azar es **muestreo sin reemplazo** de una baraja real de 52; las probabilidades del
crupier que dependen de varias cartas (cuántas pedirá antes de plantarse o pasarse) no tienen una
fórmula cerrada simple, así que se estiman por **simulación de Monte Carlo** (n = 2000): se le reparten
al crupier 2000 manos completas siguiendo su regla real y se cuenta la frecuencia relativa de cada
resultado. Es estadística inferencial —estimar un parámetro a partir de muchas muestras— y el
**Panel 5** de la Mesa ahora lo explica paso a paso con una gráfica de convergencia y el margen de error.

---

## Los casinos del mundo (barra nueva)

Arriba de la mesa hay una barra **«Elige tu casino»** con 4 sedes: 🇨🇴 Bogotá, 🇺🇸 Las Vegas,
🇲🇨 Montecarlo y 🇲🇴 Macao. Cambiar de sede reinicia la mano y cambia la piel visual (colores,
fondo, tipografía) en alusión al lugar — las reglas de juego son las mismas en las 4 (una baraja,
crupier se planta en 17, blackjack paga 3:2).

Lo que **sí** cambia entre sedes, a propósito, es la previa `P(Tramposo)` del Teorema de Bayes
(Panel 3) — así el jugador puede comparar en carne propia cómo el mismo resultado en la mesa lleva a
un posterior distinto según de dónde partió la previa:

| Sede | P(Tramposo) previa | Confianza |
|---|---|---|
| 🇨🇴 Bogotá | 10 % | Confiable |
| 🇺🇸 Las Vegas | 18 % | Vigilada |
| 🇲🇨 Montecarlo | 28 % | Sospechosa |
| 🇲🇴 Macao | 90 % | Casi con certeza tramposa |

`P(G|T) = 0.60` y `P(G|N) = 0.30` (las verosimilitudes) son las mismas en las 4 sedes — lo único que
cambia es la previa, para que la comparación sea limpia y quede claro que es el Teorema de Bayes
(previa × verosimilitud) el que mueve el resultado, no una regla de juego distinta. No se usan
nombres ni logos de casinos reales, solo la ciudad o país, para mantenerlo genérico. No puedes
cambiar de sede a mitad de una mano (hay que repartir, plantarse o resolver primero).

Macao se dejó a propósito con la previa más alta (90 %) como el **caso de demostración**: ahí el
mecanismo de Bayes es casi infalible — con una previa tan alta basta una sola mano ganada por el
crupier para que el posterior confirme la trampa con altísima confianza, ideal para mostrar en vivo
en la sustentación cómo la fórmula reacciona ante evidencia fuerte.

---

## La Academia (pestaña nueva)

Un cuaderno interactivo con **16 ejercicios** en 3 pestañas, tomados o adaptados del documento
*Probabilidad Total y Teorema de Bayes · Técnicas de Conteo* de la clase:

| Pestaña | Contenido |
|---|---|
| **1 · Conteo y árbol** | Calculadora en vivo del principio fundamental del conteo (`N = n₁·n₂·…·nₖ`) con presets (placas de Bogotá, dado+moneda, casa del urbanista); diagrama de árbol interactivo (facultades/género) que resalta `P(Mujer)` por probabilidad total o una rama individual; 4 ejercicios de conteo verificables. |
| **2 · Permutaciones y combinaciones** | 8 ejercicios (premios, club, conferencia, base de datos, batido, muestra, cartuchos, examen) con respuesta verificable y solución paso a paso. |
| **3 · Probabilidad total y Bayes** | Los 4 problemas de palabras del documento (proveedores, cirugías, accidentes, llaveros), con partición, cálculo desarrollado al revelar la solución. |

El progreso se guarda en el navegador (`localStorage`) y se muestra en una barra arriba de la Academia
y en el contador junto a la pestaña. Sigue siendo el mismo `index.html` de un solo archivo, sin backend.

---

## Estructura del repositorio

```
index.html             El juego completo (un solo archivo: HTML + CSS + JS, sin dependencias)
README.md              Este archivo
docs/DISENO.md         Documento de diseño: mecánicas, fórmulas exactas, IA, plan de QA,
                       reparto de tareas y guion de sustentación
```

El sitio en línea es este mismo `index.html` servido por **GitHub Pages** (rama `main`, carpeta raíz).
Cada `git push` a `main` actualiza el link automáticamente en ~1 minuto.

Los apuntes de clase (`Estadistica.pdf`) y la conversación de contexto se mantienen fuera del
repositorio. Para añadirlos: `git add Estadistica.pdf && git commit -m "apuntes" && git push`.

---

## Equipo

- **Diseño visual y HUD** — jhoanspalenciaflorez2007p
- **Assets y QA** — (compañero 2)
- **Lógica y programación** — asistida por Claude (Anthropic)
