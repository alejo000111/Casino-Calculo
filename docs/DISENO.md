# Documento de diseño — *Contra la Casa*

Blackjack contra IA con HUD estadístico. Proyecto de Probabilidad y Estadística, Uniempresarial.

---

## 1. Decisión de tecnología

**Elegido: aplicación web de un solo archivo (HTML + CSS + JavaScript, sin dependencias).**

Por qué, frente a Godot / Unity / Python:

| Criterio | App web (elegida) | Godot 3D |
|---|---|---|
| Instalación | ninguna; doble clic | motor + export templates |
| Trabajar entre 3 | editar un archivo de texto, cero merge de escenas binarias | escenas `.tscn` difíciles de fusionar |
| Presentar en el salón | abre en el proyector, cualquier PC | hay que llevar el ejecutable y que corra |
| El HUD de fórmulas | es texto del DOM: justo lo que la web hace mejor | overlays de UI sobre 3D, más trabajo |
| Riesgo para la fecha | bajo | alto (assets, cámara, escena, físicas) |

El "impacto visual" que pedía el planteamiento —el contraste entre la mesa de casino y las
proyecciones matemáticas flotando— se logra con CSS: fieltro verde, cartas dibujadas con
tipografía y palos Unicode, y un HUD con estética de "visión de conteo" (lectura fría, monoespaciada,
esquinas con corchetes).

**Alcance para la entrega ("vertical slice"):** un único blackjack muy pulido con los 4 paneles
estadísticos y el evento de Bayes. Nada más. Si sobra tiempo: más animación y una ruleta.

---

## 2. Bucle de juego

```
APOSTAR ──► REPARTIR ──► TURNO JUGADOR ──► TURNO CRUPIER ──► RESOLVER ──► (Bayes se actualiza) ──► NUEVA MANO
                          Pedir/Plantarse/Doblar   se planta en 17     paga/cobra    pozo += 2
```

- **Baraja:** 1 × 52 cartas. No se rebaraja entre manos → **el mazo se agota y por eso la
  probabilidad condicional importa**. Se rehace la baraja solo cuando quedan < 15 cartas (se avisa).
- **Valores:** 2–10 su número; J, Q, K = 10; As = 11 o 1 (el motor lo baja a 1 automáticamente para
  no pasarse → "mano suave" si aún cuenta como 11).
- **Crupier:** revela su carta oculta y pide hasta llegar a 17 o más (se planta también en 17 suave).
- **Pagos:** ganar 1:1 · blackjack natural 3:2 · empate ("push") devuelve la apuesta.
- **Economía:** empiezas con 1000 fichas. Apuestas de 10/25/50/100. Cada mano resuelta suma 2 fichas
  al **pozo acumulado** ("comisión de la casa"), que se cobra con una denuncia de trampa fundamentada.
  Si te quedas sin fichas aparece el botón "Pedir fichas (+500)".

### Estado interno (objeto `S` en el código)

| Campo | Significado |
|---|---|
| `discard[]` | cartas de manos ya jugadas (fuera del mazo, pero contadas para el conteo) |
| `player[]`, `dealer[]` | cartas en mesa; `dealer[1]` es la oculta |
| `holeRevealed` | si la carta oculta ya se mostró |
| `phase` | `betting` → `player` → `dealer` → `resolved` |
| `pT` | `P(Tramposo)` actual (previa/posterior de Bayes, encadenada) |
| `dealerCheating` | **verdad oculta** del crupier: `Math.random() < 0.10` al empezar cada mesa |
| `qaForce` | fuerza el resultado de la próxima mano (solo modo QA) |

---

## 3. Los cuatro paneles del HUD — fórmulas exactas

Notación: `unseen` = cartas que el jugador **no ha visto** = 52 − descartadas − mano del jugador −
cartas visibles del crupier. Incluye la carta oculta del crupier (el jugador no la conoce). `N = |unseen|`.

### Panel 1 · Frecuencia relativa — "¿pido carta?"

Tema: espacio muestral, evento, `P(A) = casos favorables / casos posibles`, complemento.

```
pt = total actual del jugador
favorables = nº de cartas en 'unseen' tales que  pt + valor(carta) ≤ 21
             (el As se cuenta como 1, así que nunca te pasa)
P(no pasarse si pides) = favorables / N
P(pasarse)             = 1 − favorables / N            ← complemento Aᶜ
P(sale un 10/figura)   = (nº de 10,J,Q,K en unseen) / N
P(sale un As)          = (nº de A en unseen) / N
```

Ejemplo que se ve en pantalla: con 13 puntos, "sirve toda carta de valor ≤ 8". Si quedan 24 cartas
seguras de 49 → `24/49 = 49.0 %`.

### Panel 2 · Probabilidad condicional — "riesgo del crupier"

Tema: `P(A|B) = P(A∩B) / P(B)`, espacio muestral reducido por un dato ya conocido.

Condición `B` = la carta visible del crupier. Espacio muestral reducido a `N` cartas no vistas.

```
si la carta visible es un As:
    P(Blackjack crupier | As visible) = (nº de cartas de valor 10 en unseen) / N
si la carta visible vale 10:
    P(Blackjack crupier | 10 visible) = (nº de Ases en unseen) / N
en otro caso: 0   (no puede tener blackjack)
```

Además, por **simulación de Monte Carlo** (n = 2000, muestreo sin reemplazo del mazo no visto):

```
P(crupier se pasa),  P(pierdes si te plantas),  P(ganas si te plantas),  P(empate)
```

El **medidor de riesgo** (verde / ámbar / rojo) usa `P(pierdes si te plantas)`:
`< 35 %` ventaja tuya · `35–58 %` mano pareja · `> 58 %` ventaja del crupier.

### Panel 2 (modo sustentación) · Probabilidad total

Tema: `P(D) = Σ P(Bᵢ)·P(D|Bᵢ)` sobre una partición del espacio muestral.

`P(crupier se pasa)` se descompone según en qué grupo cae **la carta oculta** del crupier:

```
partición:  B₁ = As    B₂ = 2–6    B₃ = 7–9    B₄ = 10/figura
P(Bᵢ)          = (nº de esas cartas en unseen) / N
P(se pasa|Bᵢ)  = estimado por Monte Carlo condicionando la carta oculta a ese grupo
P(se pasa)     = Σ P(Bᵢ) · P(se pasa|Bᵢ)          ← debe coincidir con la simulación directa
```

Se muestran los 4 términos y su suma, junto al valor de la simulación directa como verificación.

### Panel 3 · Teorema de Bayes — "¿crupier tramposo?"

Tema: `P(A|D) = P(A)·P(D|A) / P(D)`, árboles de probabilidad, probabilidad de la causa dado el efecto.

Parámetros del modelo (de los apuntes):

| | valor |
|---|---|
| `P(T)` previa inicial | 0.10 |
| `P(N)` previa inicial | 0.90 |
| `P(G\|T)` — gana el crupier si es tramposo | 0.60 |
| `P(G\|N)` — gana el crupier si es normal | 0.30 |

`G` = "el crupier ganó la mano" (el jugador pierde; el empate no cuenta). Tras cada mano:

```
si el crupier GANÓ:
    P(T | G) = P(T)·P(G|T) / [ P(T)·P(G|T) + P(N)·P(G|N) ]

si el crupier PERDIÓ:
    P(T | ¬G) = P(T)·(1−P(G|T)) / [ P(T)·(1−P(G|T)) + P(N)·(1−P(G|N)) ]

si hubo EMPATE: no se actualiza
```

El posterior se convierte en la **nueva previa** para la mano siguiente (actualización bayesiana
encadenada). El árbol de probabilidad se dibuja en pantalla con las ramas `P(T)·P(G|T)` y
`P(N)·P(G|N)` y la división.

Verificación numérica: previa 0.10, el crupier gana una mano →
`P(T|G) = (0.10·0.60) / (0.10·0.60 + 0.90·0.30) = 0.06 / 0.33 = 0.1818…`

#### Regla de decisión — botón "Denunciar anomalía"

El crupier **es realmente tramposo con probabilidad 0.10** al crear la mesa (`dealerCheating`, oculto).
Un crupier tramposo "arregla" su carta oculta y sus pedidos (elige la mejor de 3 cartas candidatas),
por lo que empíricamente gana más seguido y el posterior sube.

| Situación real | Posterior `P(T)` | Resultado |
|---|---|---|
| Tramposo | ≥ 50 % | **Denuncia fundamentada** → cobras todo el pozo acumulado |
| Tramposo | < 50 % | Tenías razón pero sin evidencia → recompensa parcial (½ del pozo) |
| Limpio | cualquiera | **Denuncia infundada** → multa de 4 × tu apuesta (falso positivo) |

Tras denunciar se genera un crupier nuevo (nueva verdad oculta, previa vuelve a 0.10).

### Panel 4 · Espacio muestral y conteo

Tema: permutaciones y combinaciones.

```
|E| baraja completa              = 52
manos iniciales posibles         = C(52,2) = 1326        (no importa el orden)
repartos ordenados               = V(52,2) = 52·51 = 2652 (importa el orden)
manos posibles con lo que queda  = C(N,2)
ejemplo lotería 5 de 50          = C(50,5) = 2 118 760
ejemplo permutación              = V(50,2) = 2450
```

Caja fija de operaciones de conjuntos sobre `E = 52`:
`A` = rojas (26), `F` = figuras (12), `A∩F` = 6, `A∪F` = 32, `Aᶜ` = 26, `P(A) = 26/52 = 50 %`.

---

## 4. IA del crupier

1. **Juego base (limpio):** roba cartas uniformemente al azar del mazo restante; se planta en 17.
2. **Crupier tramposo (`dealerCheating = true`):**
   - Carta oculta: elige la mejor de 3 candidatas al azar (mejor = más alta sin pasarse, o si todas
     se pasan, la más baja).
   - Cada pedido en su turno: misma regla de "mejor de 3".
   - Efecto neto: su tasa de victoria sube visiblemente por encima del ~30 % del modelo, y en unas
     10 manos el posterior de Bayes suele cruzar el 50 %.
3. **Modo QA** puede fijar `dealerCheating` a `true`/`false` y forzar el resultado de la próxima mano
   para validar que el HUD y las fórmulas coinciden.

---

## 5. Plan de QA

Entregar esta tabla a QA. Para cada caso: activar **Modo QA**, forzar la jugada, y comprobar a mano
que la fracción del HUD coincide.

| # | Caso | Cómo forzarlo | Qué verificar |
|---|---|---|---|
| 1 | Frecuencia relativa | Repartir; anotar tu total y las cartas visibles | `favorables / N` a mano = fracción del panel 1 |
| 2 | Complemento | mismo caso | `P(pasarse)` del panel = `1 − P(no pasarse)` |
| 3 | Condicional con As | repartir hasta que el crupier muestre un As | `P(BJ|As)` = (nº de 10/J/Q/K sin ver) / N |
| 4 | Condicional imposible | crupier muestra un 7 | panel muestra 0 % |
| 5 | Probabilidad total | modo sustentación, panel 2 | Σ de los 4 términos ≈ `P(crupier se pasa)` de la simulación (±2 %) |
| 6 | Bayes, una victoria | QA → "Crupier = TRAMPOSO", "Forzar victoria crupier", jugar mano | posterior = `0.06/0.33 = 18.18 %` |
| 7 | Bayes encadenado | forzar 3 victorias seguidas | 10 % → 18.18 % → 31.03 % → 47.5 % (aprox.) |
| 8 | Bayes, derrota | forzar derrota desde 10 % | `P(T|¬G) = (0.10·0.40)/(0.10·0.40+0.90·0.70) = 5.97 %` |
| 9 | Denuncia correcta | crupier tramposo + posterior ≥ 50 % | fichas suben en el valor del pozo |
| 10 | Falso positivo | crupier limpio + denunciar | multa = 4 × apuesta; fichas no bajan de 0 |
| 11 | Conteo | contar cartas vistas | `N` del panel 4 = 52 − vistas; `C(N,2)` correcto |
| 12 | Rebarajado | jugar hasta < 15 cartas | aviso "Barajado" y `N` vuelve a subir |

Valores de referencia para QA: `P(T)=0.10`, `P(G|T)=0.60`, `P(G|N)=0.30`.

---

## 6. Reparto de tareas

| Persona | Entregable |
|---|---|
| **Jhoan — diseño / HUD** | Paleta y tipografía del HUD, layout de los 4 paneles, medidor de riesgo por colores, árbol de Bayes en pantalla, modo sustentación, textos. |
| **Compañero 2 — assets / QA** | (Assets opcionales: arte SVG de cartas y fichas para reemplazar las de CSS.) Ejecutar la tabla de QA §5 entera y registrar cada caso; verificar que fracciones del HUD = cálculo a mano. |
| **Claude — lógica** | Motor de blackjack, muestreo sin reemplazo, cálculo de los 4 paneles, simulación Monte Carlo, actualización de Bayes, IA del crupier, modo QA. |

---

## 7. Guion de sustentación (~2–3 min)

1. **Abrir con la tesis.** "El juego no esconde la estadística: la muestra. Cada carta recalcula las
   fórmulas de nuestros apuntes en vivo." Activar **Modo sustentación**.
2. **Frecuencia relativa.** Repartir una mano. Señalar el panel 1: "Espacio muestral E = N cartas no
   vistas; el evento A = 'no me paso'; `P(A) = favorables / posibles`", leer la fracción y el
   complemento.
3. **Condicional.** Esperar (o forzar en QA) a que el crupier muestre un As. Leer el pop-up:
   "`P(Blackjack | As visible)` — el espacio muestral se redujo a las cartas no vistas".
4. **Probabilidad total.** Señalar la descomposición de `P(crupier se pasa)` en los cuatro grupos de
   la carta oculta y su suma ponderada; comparar con la simulación.
5. **Bayes — el momento fuerte.** QA → fijar crupier TRAMPOSO, forzar 3 victorias. Ver `P(Tramposo)`
   subir 10 % → 18 % → 31 % → 47 %. Explicar el árbol: "Bayes nos da la probabilidad de la causa
   (tramposo) dado el efecto (ganó)". Pulsar **Denunciar anomalía**: la cámara hace zoom, aparece el
   árbol completo con la aritmética y se cobra el pozo.
6. **Cerrar con el conteo.** Panel 4: "el espacio muestral inicial son `C(52,2) = 1326` manos; con
   permutaciones y combinaciones lo medimos sin enumerar."
7. **Método numérico.** Mencionar que las probabilidades multi-carta del crupier se estiman por
   simulación de Monte Carlo (n = 2000): estadística inferencial: estimar a partir de muestras.

---

## 8. Ideas si hay más tiempo (fuera del alcance de la entrega)

- Ruleta europea: `P(rojo) = 18/37`, intersección rojo ∩ par = `9/37` (espacio muestral y conjuntos).
- Tragamonedas de 3 rodillos: permutaciones de símbolos = nº de líneas de pago a animar.
- Minijuego "jefe de seguridad": 3 crupieres (50 % / 30 % / 20 % de los turnos), moneda cargada,
  deducir cuál fue con Bayes (`P(C | Cara)`).
- Historial de manos exportable para el informe escrito.
