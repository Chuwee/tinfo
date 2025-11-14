UNIVERSIDAD AUTÓNOMA DE MADRID
ESCUELA POLITÉCNICA SUPERIOR

Grado en
Ingeniería Informática

TRABAJO DE FIN DE GRADO

Extending Virtual Pseudo-Pilot
Automation with Flight Simulation
Interface Integration

Ignacio Ildefonso de Miguel Ruano

Advisor: Eduardo Cermeño Mediavilla

DATE

2

Chapter 1

Introduction

The increasing growth of the world of aviation has begun to raise issues regard

3

4

CHAPTER 1. INTRODUCTION

Chapter 2

Related Work

2.1 Planteamiento inicial

El modelo que proponemos consiste en una pipeline de cuatro elementos.

1. El elemento de captura y transcripción de voz. En este entorno, tradicional-

mente se le conoce como ASR (Automatic Speech Recognition) [22].

2. Un conversor de lenguaje a instrucciones de alto nivel (Language Parameter and

Instruction Parser).

3. Un módulo de interfaz de un simulador de vuelo.

4. Un módulo de respuesta (Response Handler)

5. Un módulo de TTS (Text-to-Speech)

2.2 Estudio del State of the Art

2.2.1 Automatic Speech Recognition (ASR)

Desafíos

1. Ruido. Las comunicaciones de tráfico aéreo entre pilotos y controladores utilizan
amplitud modulada (AM) [12]. Esto provoca susceptibilidad a ruido, estática
y solapamiento, lo que interfiere con la capacidad de un componente ASR para
traducir eficientemente los audios a texto. Los receptores VHF (Very High Fre-
quency) son especialmente susceptibles a un nivel demasiado bajo de SNR (Signal
to noise) [23] [6].

5

6

CHAPTER 2. RELATED WORK

2. Diferencias en el lenguaje, acentos y pronunciaciones de los controladores.
Ocasionalmente, los controladores de distintas regiones prefieren hablar su lengua

nativa con pilotos que compartan la región (por ejemplo, pilotos de Iberia pueden

hablar en español con controladores de torres de control españolas). La diferencia

fonética en acentos (inglés contra indio, por ejemplo) puede suponer un desafío

adicional [18] [21].

3. Fraseología y jerga aeronáutica que no es convencional respecto al lenguaje
hablado diariamente. Términos como callsign, runway, climb pueden no coin-
cidir con el conjunto del lenguaje objetivo de los modelos convencionales [25] [8].

4. Puntos críticos del lenguaje como los callsign o las magnitudes de las instruc-
ciones proporcionadas presentan puntos únicos de fallo. Un fallo en la magnitud

de las instrucciones o a quién van dirigidas compromete por completo la manio-

bra que se quiera comunicar [26] [14].

Datasets relevantes

1. ATCO2 [14]

• Se trata del estándar actual. Son 5.000 horas de audio real de comunicaciones

de ATC. Incluye transcripciones y anotaciones.

2. ATCOSIM [11]

• Modelo útil para validación. 10 horas de grabaciones en inglés, con transcrip-

ciones ‘limpias’.

3. UWB-ATCC [24]

• 20 horas de grabación en inglés. Contiene transcripciones hechas a mano.

Otro corpus de datos útil para validación.

4. Malorca ATC Corpus [9]

• Aeropuertos de Praga y Viena. 14 horas de grabación. Existen datos de vali-

dación (4 horas) y de entrenamiento (10 horas).

2.2. ESTUDIO DEL STATE OF THE ART

Enfoques actuales y soluciones

EL WER (Word Error Rate) se define de la manera visible en la ecuación 2.1

W ER =

S + D + I
N

7

(2.1)

donde S son las sustituciones (palabras que se confunden con otras), D son los bor-
rados (deletions, palabras que no son detectadas), I son las inserciones (palabras que se
añaden) y N es el número total de palabras en la grabación original. La mejor solución
será aquella que consiga el W ER más bajo.

1. ATCO2 Corpus [14] No es una solución en sí misma: es un ecosistema es-
tandarizado consistente de, a saber, ATCO2-test-set corpus y ATCO2 pseudo-labeled
set corpus. El primero son cuatro horas de transcripciones comprobadas, marcas
con el rol del hablante (a modo <pilot>...<pilot>) además de otros tags so-
bre elementos de la comunicación, además de todo lo que contiene el segundo

(salvo la duración). El segundo consiste en aproximadamente 5281 horas de au-

dio de control aéreo, con metadatos como puntuación de detección de inglés,

puntuaciones de confianza o SNR; datos contextuales y transcripciones con ASR

(pueden no ser correctos). Actualmente, una semana de trabajo por transcriptores

profesionales (controladores aéreos retirados o en activo) equivale a una hora de
transcripciones efectivas para poder ser usadas al estándar test-set. Los datos de
diferentes aeropuertos difieren por convenciones locales, lo cual dificulta que haya

un dominio estandarizados de datos. A nosotros, sin embargo, no nos afecta para

nuestra resolución de problemas, pues buscamos aplicar esto a pseudopilotos y no

a control aéreo real.

Antes del ATCO2 Corpus, se intentaba entrenar con corpora como Librispeech
[16] o TED-LIUM [10] pero estos datasets probaron no ser lo suficientemente
‘difíciles’ como para entrenar a los ASR de ATC.

Los resultados del entrenamiento se pueden ver en [14]. Encontramos que el WER

en ATCO2 test set es en torno a 0,22.

PROPUESTA: Si bien esto no es aceptable para un entorno de control aéreo real,
quizá podamos implementar un mejor método para minimizar el WER. Quizá

podamos hacer un nuevo corpus complementario al ATCO2 con datos de entre-

namiento. También podemos seguir con un modelo de 0,22 teniendo en cuenta

8

CHAPTER 2. RELATED WORK

que el audio del pseudopiloto va a tener alta fidelidad y confianza, no necesitamos

que el WER sea muy grande (y quizá tampoco necesitemos un corpus de entre-

namiento especializado).

2. ATCOSIM Corpus. Se trata de una base de datos de 10 horas de grabaciones
ATC. Todo está en ingles y pronunciado por angloparlantes nativos. En este caso,

tampoco se trata de una solución al problema sino de un corpus de entrenamiento,

quizá útil para validación de un modelo ASR.

En este punto conviene preguntarse si no existe un corpus en español que nos per-
mita hacer todo esto pero... en español. NO EXISTE un corpus público que sea
ad-hoc para ATC en español. Existen bases de datos de corpus generales (Mozilla
Common Voice [1], VoxPopuli [20])

3. A Virtual Simulation-Pilot Agent for Training of Air Traffic Controllers

[22]

• APUNTE: no tanto que ver con ASR, pero se asegura que la pipeline se

construye solo con Open Source.

Wav2Vec2.0 es la herramienta usada para el ASR de este artículo. También usan
XLSR (Cross-Language Speech Recognition). Merece la pena ampliar un poco esta
sección y explicar qué son estas dos cosas.

• Wav2Vec2.0 [NO RIGUROSO] [2]. Para entender esta herramienta ex-
plico lo que es el aprendizaje semi supervisado. Supongamos que un frag-

mento de audio está unívocamente determinado por una serie de informa-
ciones auditivas a1, a2, . . . , an. Digamos que cada uno de los vectores ai se
compone de m componentes (es decir, ai ∈ Rm). Entonces, lo que hace-
mos es quitar algunos de los fragmentos de audio y dar al modelo a elegir

entre ciertas opciones para rellenar el hueco vacío.

• XLSR Es una iniciativa por la que se entrena al modelo anterior en múlti-
ples idiomas. Se utiliza Multilingual LibriSpeech [17] como corpus de entre-
namiento.

El resultado consiste

2.2. ESTUDIO DEL STATE OF THE ART

9

2.2.2 Language Parameter and Instruction Parser (LPIP)

Desafíos

1. Fraseología variable. Un controlador aéreo puede plantear la misma instrucción
a una aeronave de distintas maneras (aunque exista el estándar OACI [7], de tal

forma que muchas instrucciones pueden tener la misma identidad. Por ejemplo:

IBERIA292 maintain three five zero zero

IBERIA292 climb to three five zero zero

El parseador óptimo debería interpretar ambas instrucciones como una petición

de colocarse a una altura de 3500 pies.

2. Errores de transcripción del módulo ASR pueden hacer difícil la resiliencia de

todo el pseudopiloto. Un caso de ejemplo es el siguiente:

IBERIA292 climb to three five oh oh

que quiere decir lo mismo de antes: es una instrucción para colocarse a 3500 pies.
Este tipo de fraseología alternativa es crítica pues puede darse en callsigns o en
valores de instrucción. Un fallo en la interpretación de cualquiera de estos supone

un punto único de fallo.

3. Identificación precisa de callsigns [3], que contienen números y pueden con-
fundirse con magnitudes de instrucciones. Por ejemplo, podríamos pensar en la

siguiente transcripción del ASR:

IBERIA two nine two zero heading

Un posible parseador de magnitudes podría no saber dónde acaba el callsign y
dónde empieza la magnitud de la instrucción.

4. Múltiples parámetros en la misma instrucción, como se ilustra que existen en

[22]. Podríamos pensar en la siguiente transcripción:

climb to FL350, turn left heading 090, speed 250

10

CHAPTER 2. RELATED WORK

donde tenemos tres parámetros (altura, rumbo y velocidad) para una sola aeron-

ave.

5. Estructuras temporales encadenadas, donde se proporcionan instrucciones se-
cuenciales a aeronaves. Es esencial mantener la cohesión temporal en las instruc-

ciones:

climb to FL350, after reaching point X descend to 3000

6. Normalización, de las unidades y parámetros de instrucciones. FL350, Flight
Level three five zero, three five zero son maneras de decir 35000
pies, y deben interpretarse como tal.

7. Resiliencia ante espontaneidad del discurso, ya que instrucciones que involu-
cran correcciones de ellas mismas deben ser interpretadas como si fueran instruc-

ciones normales.

Enfoques actuales y soluciones

1. A Virtual Simulation-Pilot Agent for Training of Air Traffic Controllers

[22]

Se define un componente de high level parsing y se le llama High Level Entity
Parser (HLEP). Cada una de las instrucciones transcritas se utiliza como entrada a
este módulo, donde se extraen los campos comunes a todas las instrucciones ATC.

Cada una de las entidades que se pueden extraer (en forma de valores etiquetados)
se llama Named entity. El reconocimiento de las Named entities recibe el nombre
de Named Entity Recognition:

• Callsign (por ejemplo Lima Echo Sierra 3 3 5 → LES-335)
• Comando del ATCO (mantener una altitud, orientarse con determinado án-

gulo...)

• Valores de la instrucción

De esta manera, una transcripción del tipo

ryanair nine two bravo quebec turn right heading zero nine
zero

2.2. ESTUDIO DEL STATE OF THE ART

11

se transforma en una instrucción etiquetada, de la forma

<callsign>ryanair nine two bravo quebec</callsign>

<command>turn right heading</command>

<value>zero nine zero</value>

Para esta tarea se emplea un Modelo de Lenguaje pre-entrenado y se sigue la
estrategia de ajuste fino para la NER. El modelo utilizado fue BERT (Bidirec-
tional Encoder Representations From Transformers) (específicamente la versión
pre-enetrenada BERT-base-uncased).
Se ajustó a la tarea utilizando el corpus ATCO2 [14]. Este dataset es altamente
conveniente para el ﬁne-tuning pues cuenta con transcripciones ya etiquetadas.
En términos de los resultados, el sistema basado en BERT consiguió un resultado
de precisión del 97,5% en detección de callsigns, mientras que consiguió una del
82% en commands y del 87,2% en values.

2. Research on the Method of Air Traffic Control Instruction Keyword Ex-

traction Based on the Roberta-Attention-BiLSTM-CRF Model [5]
Surge porque los metodos anteriores fallaban en capturar dependencias contex-

tuales entre distintos segmentos de comunicación ATC. Al parecer existe una di-

ficultad para mantener la secuencia correcta de las palabras clave ATC.

Estos desafíos se abordan con un modelo que propone el propio artículo,
Roberta-Attention-BiLSTM-CRF model. El modelo existente anteriormente
(Roberta-BiLSTM-CRF) solo capturaba dependencias semánticas en la misma
instrucción o segmento de instrucciones. El nuevo modelo captura la relevancia

semántica entre muchos segmentos de instrucción.

Cuenta con lo que ellos llaman ‘Módulo de atención’, lo que permite capturar

la relación entre sucesivas instrucciones (y sus precedentes). Finalmente, cuenta
con un módulo ‘BiLSTM’ (Bidirectional Long Short-Term Memory). Este es el
módulo encargado de procesar la secuencia de atención bidireccionalmente para

capturar las dependencias contextuales de los términos utilizados.

Por último, también existe la capa CRF o Conditional Random Field, que predice
las palabras clave en el orden secuencial correcto. Establece dependencias secuen-

ciales entre múltiples palabras clave.

12

CHAPTER 2. RELATED WORK

Similarmente al modelo anterior, este modelo extrae los callsigns, comandos, y
valores. Adicionalmente, extrae lo que se llama ‘código de área ATC’. Este código

de área ATC representa el espacio aéreo o área de control, por ejemplo ‘Barajas

control’.

Consideremos la siguiente instrucción ATC:

UAL215, Barajas Tower, cleared for take off, then contact
departures

El modelo BERT sin CRF o módulo de atención podría tener problemas para

identificar los límites precisos de cada entidad. Quizá identificaría toda la instruc-

ciónd después del espacio ATC ‘Barajas Tower’ como una única instrucción.

Si bien podría identificarlas correctamente como dos instrucciones distintas, nada

garantiza que sea capaz de conservar la secuencialidad de las instrucciones.

Además, sin la contextualización bidireccional, quizá instrucciones más largas po-

drían perder la referencia del ‘callsign’ original, haciendo que queden invalidadas.

En términos de resultados, el modelo con módulos Roberta-Attention-
BiLSTM-CRF sobrepasó a cualquier otro modelo anterior que tuviera una con-

figuración similar. En términos de precisión en la extracción de palabras clave,
hablamos de un 89,5%, mientras que la puntuación de precisión de coincidencia
de secuencia y F1 son de un 91,3% y 0,882 respectivamente.

La efectividad individual de cada componente se evidencia por la adición incre-

mentativa de cada uno de ellos y su comparación de rendimiento visible en el

artículo. El resumen se puede ver en el Cuadro 2.1

3. A Natural Language Understanding Approach for Digitizing Aircraft

Ground Taxi Instructions [19]

Este estudio surge de la necesidad de descongestionar las comunicaciones de radio

de los ATC, pero es aplicable a nuestro problema. Se utiliza el esquema que llaman
Intent Classiﬁcation + Slot Filling. El primero proporciona lo que sería la etiqueta
de comando en los modelos anteriores, mientras que el Slot Filling rellena el resto
de parámetros (valores y callsign).

2.2. ESTUDIO DEL STATE OF THE ART

13

Table 2.1: Resultados de modelos

Acc Precision Recall F1 Score
Model
0.805
BiLSTM-CRF
0.832
Roberta-BiLSTM
0.808
Roberta-Attention-BiLSTM
0.869
Roberta-BiLSTM-CRF
0.865
BERT-BiLSTM-CRF
0.855
Roberta -LSTM-CRF
0.886
BERT-Attention-BiLSTM-CRF
Roberta-Attention-BiLSTM-CRF 0.895

0.790
0.835
0.800
0.867
0.867
0.878
0.881
0.890

0.784
0.843
0.809
0.865
0.865
0.845
0.871
0.882

0.778
0.841
0.807
0.853
0.863
0.832
0.875
0.886

Hay que tener en cuenta que el problema que tenemos entre manos no tiene la

desventaja de la dicción difusa, los canales de comunicación desventajados u otros sim-
ilares. No estamos ante un environment de control aéreo real, sino uno controlado,
donde la calidad de la dicción puede ser tan buena como nos permita el equipamiento
hardware. Teniendo esto en cuenta:

• Si nos centramos en la ligereza y rapidez de la solución de parsing, la solución
propuesta en [19] parece bastante superior, pues tiene pocos parámetros y una

precisión que se aproxima al 90%.

• Si priorizamos la precisión crítica, el modelo propuesto en [22] puede ser acept-

able, debido a su alta precisión en la detección de callsigns (97,5%).

• Si decidimos sacrificar la ligereza, [5] propone una solución muy precisa overall.

Solución ingenua: utilizar un LLM ligero

Quizá útil para benchmarking o quizá incluso para un resultado final robusto. Teniendo
en cuenta el entendimiento del contexto que proporcionan, podríamos proponer una
situación ideal para salvaguardar fallos incluso del módulo de ASR. No hay literatura
reciente que los utilice, pero agilizan la comprensión del proceso.

La solución actual utilizando procesos descritos en la literatura nos proporcionaría

algo como lo que se puede ver en la Figura 2.1. Esto añade complejidad. Además de tener
que centrarnos en el entendimiento correcto del Language Model pertinente, también
tendríamos que elaborar un traductor a simulador que podría no ser apropiado pues el

contexto puede complicarse tanto como queramos.

14

CHAPTER 2. RELATED WORK

Figure 2.1: Diagrama del proceso de extracción no ingenuo de instrucciones ATC,
basado en arquitecturas propuestas en la literatura reciente.

2.3. FLIGHTGEAR, SIMULADOR DE VUELO

15

Table 2.2: Comparación de Soluciones de Extracción (Parsing) de Instrucciones ATC
Aspecto
Metodología de Extracción

Arquitectura Clave

Elementos Extraídos

[19]
Comprensión del Lenguaje
Natural (NLU) utilizando
Clasificación de Intención
(IC) y Relleno de Slots (SF)
[1, 2].
Modelo Long Short-Term
(LSTM) mul-
Memory
titarea
bidireccional
(BiLSTM) [7, 8]. Modelo
Ligero (≈ 0.5 millones de
parámetros) [9].

Tipos de Comando (In-
tents) (p. ej., Go to, Hold)
y Calificadores (Slots) (p.
Taxi route,
ej.,
Spot
(AEP),
[2,
13-15].

Callsign)

[22]
Comprensión del Lenguaje
Natural (NLU) utilizando
un Análisis de Entidades
de Alto Nivel (similar a
NER/Slot Filling) [3, 4].
Modelo BERT preentre-
ajustado (*fine-
nado y
tuned*) para la tarea de
Reconocimiento de Enti-
dades Nombradas
(NER)
[4, 10].
Callsigns, Commands (De-
tección de intención) y Val-
ues (Valores) [16, 17].

[5]
Extracción de Palabras
Clave para identificar tér-
revelan
minos/frases que
el contenido central de las
instrucciones [5, 6].
Modelo
Roberta-
Attention-BiLSTM-CRF
(RABC) [5,
11]. Utiliza
Roberta como base semán-
tica [12].

Palabras clave estructuradas:
Callsign, ATC Area Code,
Instruction Action, y Ob-
ject of Action [6, 18, 19].

Scores de Rendimiento Clave Macro F1 (NLU):

IC
(ATCo-only): 84.1% [25].
SF (ATCo+Pilot): 85.5%
[25]. Micro F1: ≈ 94.3%
(SF, ATCo+Pilot) [25].

F1 Score (Por Entidad):
Callsign: 97.5% [26]. Val-
ues: 87.2% [26]. Com-
mand: 82.0% [26].

Accuracy General: 89.5%
[5, 27]. F1 Score: 88.2%
[5, 27]. Accuracy de Co-
incidencia de Secuencia:
91.3% [5, 28].

La solución con un LLM se puede visualizar en la Figura 2.2. Se elimina el problema

de la traducción a instrucciones de interfaz del simulador de vuelo.

2.3 FlightGear, simulador de vuelo

16

CHAPTER 2. RELATED WORK

Figure 2.2: Diagrama del proceso de extracción ingenuo de instrucciones ATC uti-
lizando un LLM ligero. Este enfoque minimiza los módulos intermedios y delega la
comprensión contextual al modelo de lenguaje.

2.4. ESTUDIO DEL STATE OF THE ART EN PSEUDOPILOTOS

17

2.4 Estudio del State of the Art en pseudopilotos

2.4.1 Definición del problema: ¿qué es un pseudopiloto?

Existe una parte práctica de entrenamiento para el control aéreo que consiste en simular
escenarios de entrenamiento.

En los ejercicios de entrenamiento para control aéreo existen dos individuos que se

comunican [15]. El primero y más obvio es el aspirante a controlador aéreo, que comu-

nica sus instrucciones para resolver la situación inicial del escenario de entrenamiento.

El segundo es en quien centramos nuestro interés: el pseudopiloto. Se encarga de
ejecutar las instrucciones del controlador aéreo a través de herramientas de simulación

de escenarios.

Como ya hemos mencionado, su principal uso es el de entrenamiento, aunque re-

cientemente vemos una aparición más creciente en otros contextos, como la validación

de novedades técnicas para su uso en control aéreo real. Un pseudopiloto hace las ‘ve-

ces de piloto’, escuchando, ejecutando y respondiendo al controlador aéreo, incluso

haciéndose cargo de la simulación de múltiples aeronaves.

En general, todo sistema de pseudopilotos convencional sigue un esquema de fun-

cionamiento como el que se ve en la Figura 2.3

Figure 2.3: Esquema de funcionamiento de un pseudopiloto convencional: el contro-
lador emite instrucciones, el pseudopiloto las ejecuta en el simulador y el entorno de-
vuelve la retroalimentación.

18

CHAPTER 2. RELATED WORK

2.4.2 Pseudopilotos tradicionales

Descripción de un Software de entrenamiento ATC comercial

Típicamente incluyen los componentes que se enumeran a continuación:

• PseudoPilot Working Position que engloba todas las herramientas de pseudopi-
lotaje. Es una herramienta con una interfaz accesible que permite a un pseudopi-

loto controlar una o más aeronaves al mismo tiempo, introducir cambios en el
ﬂight plan y en otros datos.

• Voice Communication System que permite al pseudopiloto comunicarse con el

ATC en pruebas utilizando fraseología de aviación realista.

• Traﬃc Generation Software presente en algunos software de entrenamiento ATC.
Genera automáticamente tráfico y situaciones complejas para que el ATC en

pruebas las resuelva. El pseudopiloto se encarga de manejar aeronaves específicas

cuando avanza la prueba.

Plataformas comerciales de alta fidelidad

Un resumen con las plataformas comerciales estandarizadas se puede ver en el

Cuadro 2.3

Table 2.3: Resumen de Simuladores ATC con Fun-

cionalidad de Pseudopilotaje

Simulador

Enfoque Prin-

Módulo

de

Características Desta-

(Proveedor)

cipal

Pseudopilotaje

cadas del PP

(PP)

ESCAPE (Eu-

rocontrol)

Investigación y

Pilot Working

Formación

Position

Control de múltiples
simultánea-
aeronaves
mente (altura, rumbo,

Simulador

velocidad).
de radar avanzado.

2.4. ESTUDIO DEL STATE OF THE ART EN PSEUDOPILOTOS

19

Table 2.3 – Continuación de Simuladores ATC con Funcionalidad de Pseudopilotaje

Simulador

Enfoque Prin-

Módulo

de

Características Desta-

(Proveedor)

cipal

Pseudopilotaje

cadas del PP

(PP)

BEST

(Mi-

Estándar de la

Integrado

croNav)

Industria

CSSOFT ATC

Minimización

Diseñado para

Simulator

(CSSOFT)

de Pasos

minimizar

pasos

Simulación 3D y 2D
de aeronaves y condi-
atmosféricas.
ciones
Amplio uso global.

Permite controlar tan-
tos vuelos como sea
posible. Sistema avan-
zado de resiliencia de
sintaxis (minimiza er-
rores del operador).

ATCTrSim

(HAVELSAN)

Entrenamiento

Interfaz

y Simulación

usuario

tuitiva

accesible

de

in-

y

Énfasis en la facilidad
de uso para el pseudopi-
lotaje en escenarios de

entrenamiento.

MaxSim

(ADACEL)

Simulación

Pseudo Pilot

Soluciones

ATC Completa

Workstation

es-
(Torre,

calables
Ruta/Aproximación).

Simulador In-

Soluciones

dra (Indra)

TWR/APP/ACC

Posiciones de

pseudo-piloto

Entrenamientos
con
escenarios de aeropuer-
tos reales.

Formación completa y
(2D/3D).
exhaustiva
Sistema multi-sesión y
multi-ejercicio.

20

CHAPTER 2. RELATED WORK

Table 2.3 – Continuación de Simuladores ATC con Funcionalidad de Pseudopilotaje

Simulador

Enfoque Prin-

Módulo

de

Características Desta-

(Proveedor)

cipal

Pseudopilotaje

cadas del PP

SERA (ASTi,

integrado

en

Simloc)

(PP)

Comunicaciones

Realistas

Inteligencia
Artificial (IA)

Utiliza

IA para

la

gestión de

tráfico y

re-
Refuerza la

comunicaciones
alistas.
fraseología de radio.

Sea como fuere, todos estos Software tienen en común que su estructura global es

similar a la de la Figura 2.3.

2.4.3 Literatura y soluciones alternativas

• [4] es un primer concepto de pseudopiloto automático con ASR y TTS integrado.

Sus reglas de fraseología estándar y generación de respuesta representan una idea

inicial de lo que queremos conseguir. Demostró que era posible automatizar com-

pletamente el trabajo del pseudopiloto. No se estandarizó, pues solo se materializó

en un prototipo y su puesta en producción estaba condicionada por las circun-

stancias tecnológicas del año de publicación (2005).

• [13] proporciona acercamientos adicionales para desarrollar un módulo de re-

spuesta ATC, lo cual termina [22].

• [22] proporciona una prueba de concepto que demuestra que se puede cambiar

el paradigma actual a uno de (por lo menos) automatización de la respuesta del

controlador. Esto supone una liberación de carga, pero no la automatización del

proceso completo. Las tasas de error en sistemas como ASR y construcción de

respuestas (WER) evolucionan cada vez más a un nivel aceptable.

En definitiva, se ha llegado a un modelo como lo que se puede ver en la Figura 2.4.

Proporciona los procesos de automatización de respuesta, pero no de ejecución de in-

strucciones.

2.4. ESTUDIO DEL STATE OF THE ART EN PSEUDOPILOTOS

21

Figure 2.4: Esquema semiautomatizado propuesto para el proceso de pseudopilotaje,
integrando módulos automáticos y respuesta supervisada.

2.4.4 Propuesta actual

Nosotros proponemos un modelo que da un paso más, replicando el diagrama prop-

uesto en la Figura 2.5

Esto proporciona una eliminación del componente humano, proporcionando cier-

tas ventajas:

1. Menor requerimiento de esfuerzo humano para entrenar ATCs.

2. Mayor capacidad para escalar las simulaciones, permitiendo situaciones de control

aéreo de un gran número de aeronaves sin apenas esfuerzo.

3. Plausible autonomía del pseudopiloto para complicar escenarios de control aéreo.

4. Mecanismo de validación automática para métodos de control aéreo.

22

CHAPTER 2. RELATED WORK

Figure 2.5: Esquema de pseudopilotaje completamente automatizado: el sistema recibe
transmisiones de voz del controlador, las procesa mediante módulos ASR, compren-
sión de instrucciones y ejecuta acciones en el simulador de vuelo de forma autónoma,
generando respuestas (TTS) sin intervención humana.

Bibliography

[1] Rosana Ardila et al. “Common Voice: A Massively-Multilingual Speech Corpus”.
In: Proceedings of the 12th Language Resources and Evaluation Conference. 2020,
pp. 4218–4222.

[2] Alexei Baevski et al. “wav2vec 2.0: A Framework for Self-Supervised Learning of
Speech Representations”. In: CoRR abs/2006.11477 (2020). arXiv: 2006.11477.
url: https://arxiv.org/abs/2006.11477.

[3] Alexander Blatt et al. Call-sign recognition and understanding for noisy air-traﬃc
transcripts using surveillance information. Apr. 2022. doi: 10.48550/arXiv.2204.
06309.

[4] Richard Bolczak and Joseph Celio. Accommodating ATC System Evolution
through Advanced Training Techniques. Tech. rep. Publication on the MITRE
website. MITRE, Dec. 2005. url: https : / / www. mitre . org / news - insights /

publication/accommodating-atc-system-evolution-through-advanced-training-

techniques.

[5] Sheng Chen et al. “Research on the Method of Air Traffic Control Instruction

Keyword Extraction Based on the Roberta-Attention-BiLSTM-CRF Model”.
In: Aerospace 12.5 (2025). issn: 2226-4310. doi: 10.3390/aerospace12050376. url:
https://www.mdpi.com/2226-4310/12/5/376.

[6] Shuo Chen et al. Air Traﬃc Control Speech Recognition. Draft Technical Report.
Approved for Public Release, Distribution Unlimited. Case Number 21-0755.

The MITRE Corporation, Aug. 2021. url: https : / / www. haawaii . de / wp /

wp - content / uploads / 2021 / 08 / ATC - speech - recognition - The - MITRE -

Corporation-draft-August-2021.pdf .

23

24

BIBLIOGRAPHY

[7] EUROCONTROL. ICAO Standard Phraseology: A Quick Reference Guide for
Commercial Air Transport Pilots. https : / / skybrary. aero / sites / default / files /
bookshelf/115.pdf . Accessed: 2023-10-27.

[8]

J. Fan et al. “Customization of the ASR System for ATC Speech with Improved
Fusion Method”. In: Aerospace 11.3 (2024), p. 219. doi: 10.3390/aerospace11030219.
url: https://www.mdpi.com/2226-4310/11/3/219.

[9] German Aerospace Center (DLR) and Saarland University (USAAR) and Idiap

Research Institute (Idiap) and Austro Control Österreichische Gesellschaft für

Zivilluftfahrt mit beschränkter Haftung (ACG) and Air Navigation Services of
the Czech Republic (ANS CR). MALORCA Project – Machine Learning of
Speech Recognition Models for Controller Assistance. Web Page. Accessed: 2025-
09-25. 2018. url: https://www.malorca-project.de/.

[10] François Hernandez et al. “A new TED-LIUM release (release 3) for speech recog-
nition”. In: International Conference on Speech and Computer. Springer. 2018,
pp. 178–186.

[11] Konrad Hofbauer, Stefan Petrik, and Horst Hering. “The ATCOSIM Corpus
of Non-Prompted Clean Air Traffic Control Speech”. In: Proceedings of the Sixth
International Conference on Language Resources and Evaluation (LREC’08). Ed.
by Nicoletta Calzolari et al. Marrakech, Morocco: European Language Resources

Association (ELRA), May 2008. url: https://aclanthology.org/L08-1507/.

[12]

joey. What radio equipment do pilots use to communicate with ATC? Aviation
StackExchange, asked April 9, 2018, accessed <insert date of access>. 2018. url:

https://aviation.stackexchange.com/questions/50366/what-radio-equipment-

do-pilots-use-to-communicate-with-atc.

[13] Y. Lin et al. “A Deep Learning Framework of Autonomous Pilot Agent for Air
Traffic Controller Training”. In: IEEE Transactions on Human-Machine Systems
51.5 (2021), pp. 442–450. doi: 10.1109/THMS.2021.3102827.

[14] Petr Motlicek. End to End Callsign Recognition System. ATCO2 blog, accessed
October 24, 2025. May 2021. url: https://www.atco2.org/news/end-to-end-

callsign-recognition-system.

BIBLIOGRAPHY

25

[15] New remote piloting function in ESCAPE, the EUROCONTROL Innovation
Hub ATC simulator, adds greater ﬂexibility for stakeholders. https : / / www .
eurocontrol . int / news / new - remote - piloting - function - escape - eurocontrol -

innovation-hub-atc-simulator-adds-greater. Accessed: 2025-10-19. EUROCON-

TROL, Oct. 2021.

[16] Vassil Panayotov et al. “Librispeech: an ASR corpus based on public domain au-
dio books”. In: 2015 IEEE International Conference on Acoustics, Speech and Signal
Processing (ICASSP). IEEE. 2015, pp. 5206–5210.

[17] Vineel Pratap et al. “MLS: A Large-Scale Multilingual Dataset for Speech Re-

search”. In: ArXiv abs/2012.03411 (2020).

[18] SimpleFlying. Non-English ATC Pilot Communications Guide. Accessed: Octo-
ber 24, 2025. url: https : / / simpleflying . com / non - english - atc - pilot -

communications-guide/.

[19] Hillel A. Steinmetz et al. “A Natural Language Understanding Approach for
Digitizing Aircraft Ground Taxi Instructions”. In: AIAA AVIATION FORUM
AND ASCEND 2024. doi: 10.2514/6.2024-4359. eprint: https://arc.aiaa.org/doi/
pdf/10.2514/6.2024-4359. url: https://arc.aiaa.org/doi/abs/10.2514/6.2024-

4359.

[20] Changhan Wang et al. “VoxPopuli: A Large-Scale Multilingual Speech Corpus

for Representation Learning, Semi-Supervised Learning and Interpretation”. In:

Proceedings of the 59th Annual Meeting of the Association for Computational Lin-

guistics and the 11th International Joint Conference on Natural Language Process-
ing (Volume 1: Long Papers). Online: Association for Computational Linguistics,
Aug. 2021, pp. 993–1003. doi: 10 . 18653 / v1 / 2021 . acl - long . 80. url: https : / /

aclanthology.org/2021.acl-long.80.

[21] Marcus Yu Zhe Wee et al. “Adapting Automatic Speech Recognition for Ac-
cented Air Traffic Control Communications”. In: arXiv preprint arXiv:2502.20311
(2024). arXiv: 2502.20311 [cs.CL]. url: https://arxiv.org/abs/2502.20311.

[22]

Juan Zuluaga-Gomez et al. “A Virtual Simulation-Pilot Agent for Training of
Air Traffic Controllers”. In: arXiv preprint arXiv:2304.07842 (2023). arXiv: 2304.
07842 [eess.AS]. url: https://arxiv.org/abs/2304.07842.

26

BIBLIOGRAPHY

[23]

Juan Zuluaga-Gomez et al. “Lessons Learned in Transcribing 5000 h of Air Traf-

fic Control Communications for Robust Automatic Speech Understanding”. In:
Aerospace 10.10 (2023), p. 898. doi: 10 . 3390 / aerospace10100898. url: https : / /
publications.idiap.ch/attachments/papers/2023/Juan_AEROSPACE_2023.

pdf .

[24]

Juan Pablo Zuluaga-Gomez et al. UWB-ATCC Corpus: Communication between
Pilots and Air Traﬃc Controllers. HuggingFace Datasets, Jzuluaga/uwb_atcc.
Accessed: October 24, 2025, license: CC BY-NC-SA 4.0. 2023. url: https : / /

huggingface.co/datasets/Jzuluaga/uwb%5C_atcc.

[25]

Juan Pablo Zuluaga-Gomez et al. “ATCO2 corpus: A Large-Scale Dataset for Re-

search on Automatic Speech Recognition and Natural Language Understanding
of Air Traffic Control Communications”. In: arXiv preprint / SSRN / IDIAP
publications (2024). Submitted version; also available via SSRN and IDIAP. url:
https://www.fit.vut.cz/research/group/speech/public/publi/2024/zuluaga-

Gomez_2024_ATCO2_paper_final.pdf .

[26]

Juan Pablo Zuluaga-Gomez et al. “Contextual Semi-Supervised Learning: An

Approach To Leverage Air-Surveillance and Untranscribed ATC Data in ASR
Systems”. In: arXiv preprint arXiv:2104.03643 (2021). arXiv: 2104 . 03643
[cs.CL]. url: https://arxiv.org/abs/2104.03643.


