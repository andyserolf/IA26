           ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
             10 PREGUNTAS REDES NEURONALES CONVOLUCIONALES
                                 (CNN)

                            Eduardo Alcaraz
           ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━


Índice
──────

1. Instrucciones (alumnos)
2. Pregunta 1 — Convolución a mano (3×3)
3. Pregunta 2 — Contar pesos
4. Pregunta 3 — ¿Qué cambia si meto más capas?
5. Pregunta 4 — Formas: de volumen a vector
6. Pregunta 5 — Softmax y el error “casi seguro”
7. Pregunta 6 — Padding, stride y “se me achicó la imagen”
8. Pregunta 7 — Dropout: ¿dónde y para qué?
9. Pregunta 8 — Diseño: 5 clases de animales
10. Pregunta 9 — CNN vs MLP vs RNN (mismo problema, mala herramienta)
11. Pregunta 10 — Experimento mental: mover un gato
12. Autoevaluación rápida (opcional)
13. Notas al pie (fórmulas útiles — no sustituyen el razonamiento)


1 Instrucciones (alumnos)
═════════════════════════

  • Responde *con lápiz y papel* (o en este mismo buffer de Org).
  • No basta con definir de memoria: hay que *calcular*, *comparar* o
    *justificar*.


2 Pregunta 1 — Convolución a mano (3×3)
═══════════════════════════════════════

  Tienes este trozo de imagen en escala de grises (un canal) y este
  kernel:

  ┌────
  │   Imagen 5×5                 Kernel 3×3
  │   0  0  1  0  0              1  0 -1
  │   0  1  2  1  0              1  0 -1
  │   1  2  3  2  1              1  0 -1
  │   0  1  2  1  0
  │   0  0  1  0  0
  └────


2.0.1 Tareas
╌╌╌╌╌╌╌╌╌╌╌╌

  1. Con *stride = 1* y *padding valid* (sin relleno), ¿cuál es el
     tamaño del mapa de salida?la fórmula para cada dimensión es [ (N - K)/S  + 1 ].
  2. Calcula *solo* el valor de la esquina superior izquierda del mapa
     (la primera posición del filtro). Muestra la suma de productos.
	(0 \ 1) + (0 \ 0) + (1 \ -1) + (0 \ 1) + (1 \ 0) + (2 \ -1) + (1 \ 1) + (2 \ 0) + (3 \ -1)= 0 + 0 - 1 + 0 + 0 - 2 + 1 + 0 - 3 = -5
  3. ¿Este kernel parece detectar bordes *verticales* u *horizontales*?
     Justifica mirando los signos.
	las columnas izquierda y derecha tienen 1 y -1, mientras que la columna del medio 	es 0. Esto calcula la diferencia entre los valores 
	de la derecha y de la izquierda, por lo que detecta bordes verticales.


3 Pregunta 2 — Contar pesos
═══════════════════════════

  Una imagen RGB de *28×28×3*. Compara dos modelos:

  • *Modelo A (MLP ingenuo):* aplastas la imagen a un vector y conectas
    *todas* las entradas a una capa densa de *64 neuronas* (sin bias por
    ahora).
  • *Modelo B (CNN):* una `Conv2D' con *16 filtros* de *3×3*,
    `padding''same'=, sobre la misma imagen (con bias: un bias por
    filtro).


3.0.1 Tareas
╌╌╌╌╌╌╌╌╌╌╌╌

  1. ¿Cuántos pesos tiene la capa densa del Modelo A? (fórmula y número)
	magen de entrada: 
	Imagen de entrada:28X28X3=2352 valores
	Capa densa: 64 neuronas
	Fórmula:nin x nout
	Cálculo:2352 X 64 =150,528 pesos
  2. ¿Cuántos parámetros tiene la Conv2D del Modelo B? (pesos de kernels
     + biases)
  3. En *una frase*: ¿por qué la CNN suele generalizar mejor con el
     mismo número de imágenes de entrenamiento?
	porque al utilizar pesos compartidos (el mismo kernel recorre toda la imagen) y 	conectividad local, reduce drásticamente el número de parámetros, evitando el 	sobreajuste (overfitting).


4 Pregunta 3 — ¿Qué cambia si meto más capas?
═════════════════════════════════════════════

  Parte del notebook `CNNriesgo' (simplificado):

  ┌────
  │   Entrada (21×28×3)
  │     → Conv2D(32, 3×3, same) + LeakyReLU
  │     → MaxPool(2×2)
  │     → Flatten → Dense(32) → Dense(nClasses, softmax)
  └────

  Quieres *apilar un segundo bloque* convolucional *antes* del Flatten:

  ┌────
  │   … → Conv2D(32) → Pool → Conv2D(64, 3×3, same) → Pool → Flatten → …
  └────


4.0.1 Tareas
╌╌╌╌╌╌╌╌╌╌╌╌

  1. Tras el *primer* MaxPool(2×2) con padding same sobre 21×28, ¿qué
     tamaño aproximado (alto×ancho) esperas? (puedes redondear / usar la
     regla “casi la mitad”)
	El padding same mantiene el tamaño base (21 x 28) y el MaxPool(2×2) reduce 	
	las dimensiones a la mitad de forma aproximada.Tamaño:
	 Alrededor de 11 x 14 	(redondeando hacia arriba el ancho: 21/2 = 10.5 11, 28/2 = 	14).
  2. ¿Qué *significan* ahora los 64 filtros de la segunda Conv2D:
     detectan bordes “crudos” o patrones *más compuestos*? Explica con
     la analogía de la lupa / capas abstractas.
	Detectan patrones más compuestos o abstractos (formas geométricas, partes de 	objetos), combinando los bordes básicos detectados en la primera capa (análogo a 	pasar de ver líneas simples a reconocer formas complejas con una lupa de mayor 	nivel).
  3. Riesgo: si añades *muchas* capas Conv+Pool con pocas imágenes, ¿qué
     falla típico verías en las curvas train vs validation? ¿Qué harías
     primero para mitigarlo (Dropout, más datos, menos filtros…)? Elige
     *una* medida y justifica.
	Síntoma: Overfitting (el error en entrenamiento baja mucho, pero en validación se 	estanca o empeora).

Medida: Aplicar Dropout o añadir más datos / Data Augmentation, ya que reducen la dependencia excesiva de patrones memorizados en redes muy profundas.


5 Pregunta 4 — Formas: de volumen a vector
══════════════════════════════════════════

  Supón que, justo antes del Flatten, el tensor tiene forma `(7, 7, 64)'
  (lote omitido: una sola imagen).


5.0.1 Tareas
╌╌╌╌╌╌╌╌╌╌╌╌

  1. ¿Cuántos números salen del Flatten? (un solo entero)
	7 X 7 X 64 = 3,136
  2. Si la siguiente Dense tiene *32 neuronas* (con bias), ¿cuántos
     parámetros tiene *esa* Dense? (3136 X 32) + 32 = 100352 + 32 = 100384 Parámetros
  3. Si en vez de Flatten usas *Global Average Pooling* (un promedio por
     canal), ¿qué forma tendrías a la salida y cuántos valores serían?
     ¿Ventaja frente a Flatten cuando la imagen de entrada es más
     grande?
	Reduce cada canal de 7 X 7 a un único valor promediado, resultando en una 	forma de (64,) (es decir, 64 valores).


6 Pregunta 5 — Softmax y el error “casi seguro”
═══════════════════════════════════════════════

  Una CNN de 3 clases (inundación, incendio, tornado) saca logits que,
  tras softmax, dan:

  ┌────
  │   P = [0.10, 0.05, 0.85]
  └────

  La etiqueta verdadera (one-hot) es `[0, 1, 0]' (incendio).


6.0.1 Tareas
╌╌╌╌╌╌╌╌╌╌╌╌

  1. ¿Qué clase predice el modelo (`argmax')?
	Las probabilidades son P = [0.10, 0.05, 0.85], correspondientes a las clases 	(inundación, incendio, tornado). El valor máximo está en la tercera posición 	(índice 2).Predicción: Tornado (la tercera clase).
  2. ¿Está “seguro” o “confundido”? Usa los números, no adjetivos
     vacíos.
	Está muy seguro, ya que asigna una probabilidad de 0.85 (85%) a una de las 	clases, dejando muy poco margen para las demás.
  3. Sin calcular la fórmula exacta: argumenta si la *categorical
     crossentropy* de este ejemplo será *alta* o *baja*, y por qué
     (pista: castiga cuando la probabilidad de la clase correcta es
     pequeña).
	La etiqueta verdadera es [0, 1, 0] (incendio, clase 2), pero el modelo predijo con 	alta confianza la clase 3 (tornado). La función de pérdida castiga duramente cuando 	la probabilidad asignada a la clase correcta es baja (en este caso, la probabilidad 	para incendio es de apenas 0.05).


7 Pregunta 6 — Padding, stride y “se me achicó la imagen”
═════════════════════════════════════════════════════════

  Entrada *32×32*, kernel *5×5*, *un* filtro, un canal.


7.0.1 Tareas
╌╌╌╌╌╌╌╌╌╌╌╌

  Completa la tabla (escribe el alto×ancho de salida):

  ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
   padding  stride  salida (alto×ancho) 
  ──────────────────────────────────────
   valid         1                      
   valid         2                      
   same          1                      
  ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

  Luego: si encadenas *tres* MaxPool(2×2) seguidos sobre una imagen
  32×32 (sin cuidar bordes), ¿qué problema práctico aparece para
  clasificar objetos *pequeños* o detalles finos?
 valid, stride 1: [(32-5)/1] + 1 =28 -> 28x28
 valid, stride 2: [(32-5)/1] + 1 = [27/2] + 1 = 13 + 1= 14 -> 14x14
 valid, stride 3:  Padding same con stride 1 mantiene el tamaño original  32 x 32 


8 Pregunta 7 — Dropout: ¿dónde y para qué?
══════════════════════════════════════════

  En `CNNriesgo' hay `Dropout(0.5)' después del pooling y otra vez antes
  de la salida.


8.0.1 Tareas
╌╌╌╌╌╌╌╌╌╌╌╌

  1. Durante *entrenamiento*, ¿qué hace Dropout(0.5) en una capa densa
     de 32 neuronas (en promedio cuántas “sobreviven” por paso)?
	Apaga aleatoriamente el 50% de las neuronas en cada paso de entrenamiento. En 	promedio, sobreviven 16 neuronas activas por paso.
  2. Durante *inferencia* (`predict'), ¿se apagan neuronas al azar? ¿Por
     qué sí o no?
	No. Durante la inferencia el Dropout se desactiva por completo; todas las neuronas 	participan para aprovechar toda la capacidad aprendida del modelo.
  3. Un compañero pone Dropout *solo* en la última época “para que no se
     sobreajuste al final”. ¿Tiene sentido? Critica la idea en 3–5
     líneas.
	No tiene sentido. El Dropout es una técnica de regularización diseñada para forzar 	a la red a aprender rutas redundantes y evitar el sobreajuste durante todo el 	proceso de entrenamiento.


9 Pregunta 8 — Diseño: 5 clas
es de animales
═══════════════════════════════════════════

  Debes adaptar `CNNriesgo' a: ranas, arañas, monos, ballenas, pájaros.


9.0.1 Tareas
╌╌╌╌╌╌╌╌╌╌╌╌

  1. ¿Cuántas neuronas debe tener la *última* Dense y qué activación?
     ¿Por qué?
	Debe tener 5 neuronas (una por cada clase: ranas, arañas, monos, ballenas, pájaros) 	con activación Softmax, ya que es un problema de clasificación multiclase 	mutuamente excluyente y necesitamos obtener probabilidades que sumen 1.
  2. Si todas las fotos entran a *224×224×3* pero el modelo tiene
     `input_shape=(21,28,3)', ¿qué *debe* pasar antes del `fit'? (una
     frase operativa)
	Las imágenes deben ser redimensionadas (resize) a 21 x 28 x 3 antes de 	pasarlas al método fit para que coincidan con la input_shape esperada por el 	modelo.
  3. Propón *una* mejora de arquitectura (segundo Conv, BatchNorm, data
     augmentation…)  y di *qué síntoma* en train/val te haría pensar que
     esa mejora era necesaria.
Si notas que el modelo sufre de overfitting (el error de entrenamiento baja mucho pero el de validación se estanca o empeora con pocas imágenes), Data Augmentation es ideal para generar variedad artificial y mejorar la generalización.


10 Pregunta 9 — CNN vs MLP vs RNN (mismo problema, mala herramienta)
════════════════════════════════════════════════════════════════════

  Tres escenarios. Elige *CNN*, *MLP* o *RNN/Transformer* y justifica en
  *una* frase cada uno:

  1. Predecir si un alumno aprueba con promedio, faltas y horas de
     estudio (tabla).
MLP. (Los datos tabulares sin estructura espacial ni secuencial se procesan mejor con capas densas).
  2. Clasificar fotos de semáforo (rojo / amarillo / verde).
CNN. (Trata con datos espaciales/imágenes aprovechando la localidad y los filtros convolucionales).
  3. Autocompletar la siguiente línea de código dado el prefijo escrito.
RNN/Transformer. (Maneja datos secuenciales de texto donde el orden y el contexto histórico importan).

  Si aplastas la foto del semáforo y usas un MLP “grande”, ¿puede
  *aprender*? ¿Qué sacrificas respecto a una CNN? (parámetros, datos,
  invariancia…)
Sí, puede aprender, pero sacrifica eficiencia, requiriendo una cantidad masiva de parámetros y datos, además de perder la invariancia espacial (si el semáforo se desplaza de lugar en la foto, al MLP le costará reconocerlo).


11 Pregunta 10 — Experimento mental: mover un gato
══════════════════════════════════════════════════

  Entrenaste una CNN que detecta “hay un gato” en fotos 64×64. En el
  test, el mismo gato aparece *corrido 10 píxeles a la derecha*, mismo
  fondo, misma escala.


11.0.1 Tareas
╌╌╌╌╌╌╌╌╌╌╌╌╌

  1. ¿Por qué una CNN tiene *más chance* de seguir acertando que un MLP
     que ve el vector aplastado? (pesos compartidos / detección local)
Gracias a los pesos compartidos y la detección local, el kernel detecta las características del gato (orejas, bigotes) sin importar en qué región de la imagen se encuentren, mientras que un MLP asocia posiciones fijas del vector de entrada.
  2. Si en vez de correrlo lo *giraras 90°* y nunca viste gatos de lado
     en el train, ¿la CNN “mágica” garantiza acierto? ¿Qué haría falta
     (augmentation, más datos…)?
La CNN no garantiza el acierto automático, ya que los kernels aprendieron patrones orientados en otra dirección. Para solucionarlo se requeriría Data Augmentation (rotaciones en el entrenamiento) o más datos variados.
  3. Escribe *una* pregunta que tú le harías a un compañero para
     comprobar si entendió qué son los *pesos del kernel* (no copies las
     de arriba).
Si cambiamos los valores numéricos de un kernel entrenado para detectar bordes verticales por unos ceros, ¿qué dejaría de percibir la red en el mapa de características resultante y por qué?


12 Autoevaluación rápida (opcional)
═══════════════════════════════════

  Marca 1–5 (1 = no entendí, 5 = lo explicaría en pizarrón):

  ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
   Tema                                Nota 
  ──────────────────────────────────────────
   Producto punto del kernel                
   Conteo de parámetros Conv vs Dense       
   Efecto de apilar Conv+Pool               
   Softmax + pérdida                        
   Dropout train vs inferencia              
  ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━


13 Notas al pie (fórmulas útiles — no sustituyen el razonamiento)
═════════════════════════════════════════════════════════════════

  • Salida convolución (un eje), padding valid: \(\lfloor (N - K)/S
    \rfloor + 1\)
  • Padding same + stride 1: salida ≈ mismo \(N\)
  • Parámetros Conv2D: \(K_h \cdot K_w \cdot C_{in} \cdot C_{out} +
    C_{out}\) (si hay bias)
  • Parámetros Dense: \(n_{in} \cdot n_{out} + n_{out}\) (si hay bias)
