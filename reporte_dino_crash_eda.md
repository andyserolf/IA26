# Reporte EDA — Operación Dino Crash
**Analista:** [Tu nombre / matrícula aquí]

## 1. Problema y dataset (Misión 1)

### P1 — ¿Morirá en el siguiente frame?
- **Y:** variable binaria `died_next_frame` — 1 si el dino muere en el frame inmediato siguiente, 0 si no. **Ojo:** no es lo mismo que `died` tal como viene en el diccionario, que solo marca el último frame de la sesión (el momento en que la colisión ya ocurrió). Hay que derivar `died_next_frame` desplazando `died` una posición hacia atrás dentro de cada sesión.
- **X (mínimo 5):**
  1. `speed` — a mayor velocidad, menor margen de reacción.
  2. `dist_obstacle` — el predictor más directo de colisión inminente.
  3. `obstacle_type` — un `cactus_large` exige una reacción distinta a un `bird`.
  4. `jump` — si el dino ya está en el aire, cambia la probabilidad de impacto según la fase del salto.
  5. `frame`/`time_ms` dentro de la sesión — permite modelar el aumento progresivo de dificultad.
  6. *(extra recomendada)* si el dino está agachado (`duck`) y su altura exacta — crítica para distinguir colisión con `bird` vs `cactus`.
- **Granularidad:** cada frame (~16 ms si el juego corre a 60 fps). No sirve un resumen por partida: la pregunta es sobre el frame siguiente.
- **Tamaño mínimo razonable:** la muerte es un evento raro (un frame por partida entre cientos). Lo que importa no es tener muchos frames de una sola partida, sino muchas partidas para acumular suficientes ejemplos positivos. Ejemplo: si en promedio mueres 1 vez cada ~250 frames, 200 partidas de 250 frames dan ~50 000 filas pero solo 200 positivos (~0.4%) — probablemente insuficiente; conviene apuntar a miles de partidas.
- **Riesgo si está mal definido:** si se deja `died` tal cual (marca la colisión ya consumada) en vez de "va a morir en el próximo frame", el modelo aprende a *detectar* la muerte después de ocurrida, no a *predecirla* — inútil para cualquier uso práctico (ej. avisar al jugador o a un bot).

### P2 — ¿Cuántos puntos alcanzará esta partida al morir?
- **Y:** `score_final`, numérica continua/entera.
- **X:**
  1. `speed` promedio de la partida.
  2. cantidad de obstáculos esquivados hasta el punto de predicción.
  3. tipo/dificultad de los obstáculos enfrentados.
  4. tiempo transcurrido hasta el momento de la predicción.
  5. número de saltos/agachadas realizadas (proxy de habilidad del jugador).
- **Granularidad:** resumen por partida (una fila = una sesión completa). Si se quisiera predecir "a mitad de partida", se necesitarían features calculadas solo hasta ese punto.
- **Tamaño mínimo:** cientos de partidas completas — aquí cada partida es una sola observación, no cada frame.
- **Riesgo:** tratar cada frame como si fuera una observación independiente para este problema infla artificialmente el N con filas que no son "una partida".

### P3 — ¿Qué tipo de obstáculo viene próximo?
- **Y:** `obstacle_type`, categórica multiclase (`none`, `cactus_small`, `cactus_large`, `bird`).
- **X:**
  1. `dist_obstacle` actual.
  2. `speed`.
  3. `frame`/`time_ms` (algunos juegos introducen tipos nuevos con el progreso).
  4. `score` (proxy de dificultad progresiva).
  5. secuencia de los últimos N obstáculos (si el generador tiene algún patrón).
- **Granularidad:** por frame o por evento (cada vez que aparece un obstáculo nuevo).
- **Tamaño mínimo:** suficiente para cubrir bien las 4 clases — si `bird` aparece solo ~11% del tiempo, hace falta un N grande para tener ejemplos representativos de esa clase minoritaria.
- **Riesgo:** si el generador de obstáculos es puramente aleatorio (como suele ser en este tipo de juego), ningún modelo superará a las probabilidades base — sería un problema mal planteado si se espera "predecir" algo sin señal real.

## 2. Diccionario y muestra (Misión 2)
- **Patrón en `died=1` (frame 82):** `dist_obstacle` cayó a 12 px (muy cerca), `jump=0` (el dino no estaba saltando) con un `cactus_small`. La muerte ocurre cuando la distancia se reduce a un mínimo crítico y no hubo reacción a tiempo.
- **¿`score` es buena variable para predecir muerte en el siguiente frame?** No directamente. `score` sube de forma monótona con el tiempo/velocidad, así que es más un proxy de "cuánto ha durado la partida" que una señal causal de riesgo inmediato — fácil de confundir con leakage.
- **¿Falta alguna columna crítica?** Sí: si el dino está agachado (`duck`), su altura/posición vertical exacta, y algo como `reaction_time`/`input_lag` del jugador. Sin la posición vertical no se puede distinguir bien un salto tardío para un cactus de una agachada necesaria para un bird.
- **¿`died` tal cual sirve para P1?** No. Solo marca el frame en que la colisión ya ocurrió — describe el final de la partida, no anticipa nada. Hay que construir la etiqueta real desplazando `died` hacia atrás (shift) dentro de cada sesión.

## 3. Checklist EDA (Misión 3)
*(elegí las preguntas #2, #3 y #8, más el ejemplo de leakage pedido)*

- **#2 Valores faltantes:** en telemetría de juego suele haber pocos NA reales, pero `dist_obstacle` podría estar "vacío" cuando `obstacle_type=none` (no hay nada que medir) — eso no es un NA verdadero sino un "no aplica" que conviene codificar con un valor grande fijo (ej. 9999) en vez de eliminar la fila.
- **#3 Balance de clases:** con 50 positivos en ~12 000 frames (~0.4%), `died_next_frame` está fuertemente desbalanceada.
- **#8 i.i.d.:** los frames de una misma sesión **no** son independientes — el frame 81 depende casi por completo del 80 (misma velocidad, misma trayectoria). Si el train/test se divide por frame suelto en vez de por sesión completa, frames casi idénticos de la misma partida terminan en ambos conjuntos: el modelo "memoriza" la sesión en vez de generalizar, y la métrica de validación se ve mejor de lo que realmente es.
- **Ejemplo de leakage con `score`/`time_ms`:** si esas columnas, tal como están en el frame actual, ya reflejan implícitamente que la partida terminó ahí (por ejemplo, si el dataset se generó *después* de que la partida acabó y el frame final "sabe" que fue el último), el modelo estaría usando información que en producción real no existiría en el momento de predecir. Hay que verificar que cada feature use solo información disponible hasta ese frame, nunca conocimiento del futuro de la sesión.

## 4. Interpretación de resúmenes (Misión 4)
- **¿P1 desbalanceado?** Sí, fuertemente: 50 positivos / ~12 000 frames ≈ 0.42%. Un modelo que siempre prediga "no muere" tendría ~99.6% de accuracy y sería inútil.
- **Implicación para la métrica:** accuracy es engañosa aquí; conviene usar precision, recall, F1 o AUC-PR, y considerar `class_weight` o remuestreo (con cuidado de no romper la estructura temporal por sesión).
- **¿`dist_obstacle` es útil como predictor?** Sí, muy útil: las muertes de la muestra ocurren con `dist_obstacle` en 12 y 22 px, muy por debajo de la media (95) e incluso de la mediana (88) — coincide con el comentario del analista ("muertes suelen con dist < 20"). Probablemente el predictor más fuerte disponible.
- **¿La distribución de `score` sugiere regresión simple?** No sin ajustes. Media (28) muy por encima de la mediana (18) con cola larga a la derecha = distribución sesgada, no simétrica. Una regresión lineal cruda probablemente viole supuestos de normalidad de residuos; conviene transformar (log, raíz cuadrada) o usar un modelo que no asuma linealidad (árbol de regresión).

## 5. Elección de modelo (Misiones 5-6)

| Escenario | Fila de la guía que aplica | Modelo propuesto | 2 condiciones del dataset que deben cumplirse |
|---|---|---|---|
| P1 | Y binaria muy desbalanceada | Regresión logística o árbol/Random Forest con `class_weight`, evaluado con F1/AUC-PR | (1) Partición train/test **por sesión**, no por frame; (2) suficientes positivos tras balanceo para no sobreajustar a 50 casos |
| P2 | Y numérica (score final) | Regresión lineal (con transformación) o árbol regresor | (1) `score` transformado o modelo robusto a asimetría; (2) cientos de partidas completas como observaciones independientes |
| P3 | Y categórica multiclase | Regresión logística multinomial o árbol de decisión | (1) Ejemplos suficientes de la clase minoritaria (`bird`, 11%); (2) evidencia de que el generador de obstáculos tiene algún patrón aprendible, no puro azar |

### Contraejemplos (Misión 6)
1. **Árbol profundo que parece buena idea pero el EDA lo desaconseja:** para P1, con solo 50 positivos en 12 000 frames, un árbol muy profundo (o Random Forest sin restricciones) memorizaría esos 50 casos específicos (overfitting) en vez de aprender el patrón general "`dist_obstacle` bajo + `jump=0`". El tamaño diminuto de la clase positiva desaconseja modelos de alta complejidad sin antes limitar su profundidad o conseguir más datos.
2. **Escenario donde una red neuronal (LSTM) sí tendría sentido:** si se acumulan miles de sesiones completas y el objetivo es capturar dependencias temporales complejas (predecir varios frames hacia adelante considerando el ritmo de juego, no solo el frame inmediato). El EDA debería mostrar: (a) volumen de datos realmente grande, y (b) evidencia de que patrones temporales largos —no solo el frame anterior— aportan señal.
3. **¿P1 con reglas fijas (`si dist_obstacle < X y jump=0 → muerte`)?** Totalmente viable como primera aproximación — el propio EDA (muertes con `dist_obstacle`=12 y 22) sugiere que un umbral simple ya captura buena parte de la señal. **Ventaja de la regla fija:** interpretable, no requiere resolver el desbalance ni entrenar nada. **Límite:** no se ajusta a distintas velocidades (un umbral fijo no es igual de seguro a `speed`=6 que a `speed`=13) ni a distintos tipos de obstáculo. Un modelo aprendido combina estas variables y ajusta el umbral dinámicamente, a costa de ser menos transparente y necesitar más datos.

## Síntesis (5 líneas)
Antes de pensar en modelos, pediría telemetría por frame que incluya explícitamente la posición vertical/agachado del dino, y suficientes partidas para acumular cientos de eventos de muerte reales (no solo 50). El EDA ya deja claro que P1 es un problema de clasificación binaria severamente desbalanceado, donde `dist_obstacle` y `speed` muestran señal fuerte incluso sin entrenar nada. Solo después de confirmar el balance de clases, la independencia por sesión (no por frame suelto) y la ausencia de leakage temporal, elegiría entre una regla simple basada en umbral o un modelo ligero con ponderación de clases. Nunca partiría directo hacia una red profunda sin que los datos —en volumen y en forma— lo justificaran primero.
