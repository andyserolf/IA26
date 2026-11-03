&#x20;          ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

&#x20;           EXAMEN TEÓRICO — PERCEPTRÓN, MLP Y CNN (GRUPO B)



&#x20;                           Eduardo Alcaraz

&#x20;          ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━





1 Datos del alumno

══════════════════



&#x20; ━━━━━━━━━━━

&#x20;  Nombre  Andrea López Flores

&#x20;  Grupo   B 

&#x20;  Fecha   03/11/26   

&#x20; ━━━━━━━━━━━





2 Pregunta 1 — Por qué no basta “una sola neurona”

══════════════════════════════════════════════════



&#x20; 1. Resume el funcionamiento de un perceptrón como “regla de decisión”.

&#x20; 2. Explica la noción de \*frontera lineal\* sin escribir fórmulas.

&#x20; 3. Da un ejemplo (aunque sea inventado) donde esa frontera \*no\*

&#x20;    alcanza.





2.0.1 Respuesta

╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌



&#x20; ┌────

1. El perceptrón le da un peso a cada dato, los suma y decide: si el total pasa de un límite, responde una clase; si no, la otra.
2. Es una línea recta que separa los datos en dos grupos. Un lado es una clase y el otro lado es la otra.
3. Puntos de la clase “sí” en el centro y puntos de la clase “no” alrededor. Una línea recta no los puede separar.



&#x20; └────





3 Pregunta 2 — CNN: de píxeles a mapas

══════════════════════════════════════



&#x20; 1. ¿Qué problema de \*estructura espacial\* ignora un MLP si aplana la

&#x20;    foto?

&#x20; 2. ¿Qué significa que un mismo filtro se aplique en toda la imagen?

&#x20; 3. Además de clasificación de objetos, cita \*dos\* usos donde la CNN

&#x20;    encaja bien.





3.0.1 Respuesta

╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌



&#x20; ┌────

1. Al aplanar la foto se pierde qué píxeles están juntos. El MLP no ve bordes ni formas, solo números sueltos.
2. El mismo filtro revisa toda la imagen, así detecta el mismo patrón en cualquier lugar y usa menos parámetros.
3. Segmentación de imágenes y diagnóstico por imágenes médicas.



&#x20; └────





4 Pregunta 3 — Softmax, clases y decisiones

═══════════════════════════════════════════



&#x20; 1. Diferencia un problema \*binario\* de uno \*multiclase\* en términos de

&#x20;    salida de la red.

&#x20; 2. ¿Qué lectura le das a las probabilidades que produce un softmax?

&#x20; 3. Imagina tres carpetas de imágenes (tres riesgos o tres animales):

&#x20;    ¿qué arquitectura de \*salida\* usarías y por qué?





4.0.1 Respuesta

╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌



&#x20; ┌────

1. En el binario es una neurona de salida y en el multiclase es una neurona por clase.
2. Cada probabilidad es qué tan seguro está el modelo de esa clase. Suman 1 y gana la más alta.
3. Salida de 3 neuronas con softmax, porque cada imagen pertenece a una sola de las 3 clases.

&#x20; └────





5 Pregunta 4 — MLP: profundidad con sentido

═══════════════════════════════════════════



&#x20; 1. ¿Qué ganas al apilar capas densas con activaciones no lineales?

&#x20; 2. ¿Qué riesgo aparece si la red es \*demasiado\* grande para tus datos?

&#x20; 3. Relaciona “capa oculta” con la idea de aprender representaciones

&#x20;    intermedias (en palabras simples).





5.0.1 Respuesta

╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌



&#x20; ┌────

1. La red puede aprender relaciones más completas no solo líneas rectas.
2. Sobreajuste: memoriza los datos de entrenamiento y falla con datos nuevos.
3. La capa oculta transforma los datos en características nuevas que ayudan a decidir mejor.

&#x20; └────





6 Pregunta 5 — Entrenamiento sin magia

══════════════════════════════════════



&#x20; 1. Explica qué es una \*época\* a nivel intuitivo.

&#x20; 2. ¿Hacia qué “objetivo” se mueven los pesos cuando hablamos de

&#x20;    minimizar el error?

&#x20; 3. Si el learning rate es absurdo (muy grande o muy chico), ¿qué

&#x20;    síntomas conceptuales esperarías?





6.0.1 Respuesta

╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌



&#x20; ┌────

1. Una pasada completa por todos los datos de entrenamiento.
2. Hacia el error más bajo posible.
3. Muy grande: el error sube y baja sin mejorar. Muy chico: el error baja demasiado lento.

&#x20; └────





7 Pregunta 6 — Bloques de una CNN

═════════════════════════════════



&#x20; Sin matemáticas, di el papel de cada pieza:

&#x20; 1. Kernel / filtro convolucional

&#x20; 2. Función de activación después de conv

&#x20; 3. Max pooling

&#x20; 4. Capa densa + softmax al final de un clasificador





7.0.1 Respuesta

╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌



&#x20; ┌────

1. Kernel: detecta un patrón, como un borde.
2. Activación: permite que la red aprenda patrones complejos.
3. Max pooling: reduce el tamaño de la imagen y conserva lo más importante.
4. Densa + softmax: combina todo y da la probabilidad de cada clase.

&#x20; └────





8 Pregunta 7 — XOR y la historia del aprendizaje

════════════════════════════════════════════════



&#x20; 1. ¿Por qué XOR se volvió un “contraejemplo” famoso?

&#x20; 2. ¿Qué elemento de un MLP permite patrones que no son una sola línea?

&#x20; 3. ¿En qué tipo de tabla/problema del mundo real seguirías usando algo

&#x20;    tan simple como un perceptrón o un modelo lineal?





8.0.1 Respuesta

╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌



&#x20; ┌────

1. Un perceptrón no lo puede resolver porque sus clases no se separan con una línea recta.
2. Las capas ocultas con activaciones no lineales.
3. En tablas con pocas columnas numéricas y relaciones simples, como predecir precios.

&#x20; └────





9 Pregunta 8 — De lo simple a lo abstracto en visión

════════════════════════════════════════════════════



&#x20; 1. Contrasta lo que buscan capas tempranas vs capas profundas en una

&#x20;    CNN.

&#x20; 2. ¿Qué intuición da eso para \*transfer learning\* (aunque no lo hayas

&#x20;    programado)?

&#x20; 3. ¿Cuándo sospecharías que tu CNN es “profunda de más” para el

&#x20;    dataset del curso?





9.0.1 Respuesta

╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌



&#x20; ┌────

1. Las capas tempranas detectan bordes y colores. Las profundas detectan partes de objetos y objetos completos.
2. Las capas tempranas sirven para casi cualquier imagen, así que se reutilizan y solo se reentrenan las últimas.
3. Cuando el dataset es pequeño, la precisión en entrenamiento es alta y en validación es baja.

&#x20; └────

&#x20;



10 Pregunta 9 — Análisis: matching problema → modelo

════════════════════════════════════════════════════



&#x20; Elige \*Perceptrón/lineal\* / \*MLP\* / \*CNN\* (una opción) y justifica en

&#x20; una frase:



&#x20; 1. Predecir “compra / no compra” con edad, ingreso y visitas a la web

&#x20;    (columnas numéricas).

&#x20; 2. Detectar si hay un gato en una foto.

&#x20; 3. Aproximar una frontera “curva” o a trozos en datos 2D tabulares.

&#x20; 4. ¿CNN es la herramienta natural para predecir la siguiente palabra

&#x20;    de un párrafo? Justifica y nombra una familia del curso más

&#x20;    adecuada (RNN/Transformer a nivel nombre).





10.0.1 Respuesta

╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌



&#x20; ┌────

1. Perceptrón - lineal: pocas columnas numéricas y decisión de sí o no.
2. CNN: es una imagen.
3. MLP: necesita fronteras curvas.
4. No. La CNN es para imágenes. Para texto es mejor una RNN o un Transformer.

&#x20; └────





11 Pregunta 10 — Análisis: auditoría de un equipo

═════════════════════════════════════════════════



&#x20; Un equipo entrega un modelo “muy preciso” en su laptop y falla con

&#x20; datos nuevos.



&#x20; 1. ¿Qué diagnósticos conceptuales harías \*antes\* de tocar

&#x20;    hiperparámetros al azar?

&#x20; 2. Pide \*dos\* evidencias que deberían mostrar en la defensa (curvas,

&#x20;    separación train/val, ejemplos de error, etc.).

&#x20; 3. Redacta \*tres\* preguntas tuyas para comprobar si saben cuándo usar

&#x20;    perceptrón, MLP o CNN (distintas a las del Grupo A si las conoces).





11.0.1 Respuesta

╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌



&#x20; ┌────

1. Revisaría si hay sobreajuste, si los datos de validación se mezclaron con los de entrenamiento y si los datos nuevos son distintos a los de entrenamiento.
2. Pediría las curvas de entrenamiento y validación, y ejemplos de los errores del modelo.
3. Preguntas:

&#x09;¿Por qué elegiste esta arquitectura?

&#x09;¿Qué tipo de datos usas?

&#x09;¿Por qué no usaste un modelo más simple?



&#x20; └────



