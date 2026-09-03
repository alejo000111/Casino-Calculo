# Contra la Casa — Blackjack con Visión Estadística

Proyecto para **Probabilidad, Estadística y Métodos Numéricos** · Uniempresarial.

Un blackjack (21) contra una IA en el que **la matemática es la mecánica principal, no código oculto**.
Mientras juegas, un HUD lateral recalcula en tiempo real —con las fórmulas de los apuntes— la
frecuencia relativa, la probabilidad condicional, la probabilidad total y el Teorema de Bayes.

**Jugar ahora:** abre `ContraLaCasa.html` en cualquier navegador (doble clic). No requiere instalar nada,
ni servidor, ni conexión.

Artifact publicado: https://claude.ai/code/artifact/347ccffd-b078-4234-a87d-082d8eca78a7

---

## Cómo se juega

| Acción | Botón / tecla | Qué hace |
|---|---|---|
| Elegir apuesta | fichas 10 / 25 / 50 / 100 | Solo antes de repartir |
| Repartir | `Repartir` · `D` | Reparte 2 cartas a cada uno (la 2.ª del crupier queda oculta) |
| Pedir carta | `Pedir` · `H` | Suma una carta a tu mano; si pasas de 21 pierdes |
| Plantarse | `Plantarse` · `S` | Cede el turno; el crupier juega hasta 17 |
| Doblar | `Doblar` | Duplica la apuesta, recibes 1 carta y te plantas |
| Denunciar anomalía | `Denunciar anomalía` | Acusas al crupier de tramposo usando el posterior de Bayes |

Reglas: una sola baraja de 52, **el crupier se planta en 17** (incluido 17 "suave"),
blackjack natural paga **3 : 2**, empate devuelve la apuesta.

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

Parámetros del modelo de Bayes (tomados de los apuntes):
`P(T) = 0.10`, `P(N) = 0.90`, `P(G|T) = 0.60`, `P(G|N) = 0.30`.

El motor de azar es **muestreo sin reemplazo** de una baraja real de 52; las probabilidades del
crupier que dependen de varias cartas se estiman por **simulación de Monte Carlo** (n = 2000),
lo cual es en sí mismo estadística inferencial (estimar un parámetro a partir de muchas muestras).

---

## Estructura del repositorio

```
ContraLaCasa.html      El juego completo (un solo archivo: HTML + CSS + JS, sin dependencias)
README.md              Este archivo
docs/DISENO.md         Documento de diseño: mecánicas, fórmulas exactas, IA, plan de QA,
                       reparto de tareas y guion de sustentación
```

Los apuntes de clase (`Estadistica.pdf`) y la conversación de contexto se mantienen fuera del
repositorio. Para añadirlos: `git add Estadistica.pdf && git commit -m "apuntes" && git push`.

---

## Equipo

- **Diseño visual y HUD** — jhoanspalenciaflorez2007p
- **Assets y QA** — (compañero 2)
- **Lógica y programación** — asistida por Claude (Anthropic)
