## Práctica 1 - Follow Line

Esta práctica consiste en programar un coche de Fórmula 1 para que sea capaz de recorrer un circuito en el menor tiempo siguiendo una línea roja que discurre a lo largo de este utilizando control visual.

Para ello disponemos de una cámara situada en la parte delantera del vehículo por lo que, utilizando control visual y un controlador PID, podremos abordar esta práctica.

### 1.1 Preproceso

En primer lugar necesitamos localizar y segmentar la línea que define la trayectoria a seguir. Puesto que la línea tiene un color rojo definido y distinguible del color del asfalto, podremos aplicar un filtro de color sobre las imágenes adquiridas de la cámara del coche y obtendremos la imágen binaria los píxeles pertenecientes a la línea como primer plano.

Cabe destacar que, tras realizar diversas pruebas, el filtro de color se realiza sobre el espacio de color HSV para que sea más robusto ante cambios en la iluminación a lo largo del circuito.

### 1.2 Estimación de la desviación del F1 con respecto a la línea

Una vez segmentada la línea debemos definir tanto la velocidad lineal como la velocidad de giro del fórmula 1. Para ello en primer lugar debemos estimar en cada instante cuál es la desviación del vehículo con respecto a la línea. Para ello calculamos la distancia en píxeles entre centro de la línea previamente segmentada y el centro de la cámara. 

Idealmente la distancia entre estos debería ser 0 lo que indicaría que el morro del vehículo esta alineado a la perfección con la línea que debe seguir. En esta situación el F1 unicamente debería moverse manteniendo la trayectoria y por lo tanto, sin realizar ningún giro. 

Puesto que la línea que debemo seguir no es recta, si no que tiene multitud de curvas, tarde o temprano, el coche dejará de estar alineado con la línea al llegar a una de estas. Cuando esto ocurra, el centro de la línea y el de la cámara dejarán de coincidir y la distancia en píxeles entre ambas crecerá (en valores negativos o positivos en función de si la desviación se produce hacia uno u otro lado). Gracias a esta distancia podremos estimar la desviación y transformarlo en una velocidad de giro para corregirla.

El cálculo de la desviación se realiza a tres alturas distintas dentro de la imágen segmentada de la línea obteniendo 3 valores de desviación. Comparando estos tres valores podemos definir si durante la ejecución el vehículo se encuentra en una recta o en cambio se encuentra en una curva para que el movimiento de este sea distinto en cada caso. Si los valores de estas tres desviaciones se encuentra dentro de un rango nos encontraremos en una recta. En cambio, si los valores de estas tres desviaciones se salen de este rango, estaremos en una curva. Este rango se ha definido de forma experimental.

### 1.3 Definición del movimiento del F1: Control PD

Para definir la velocidad linear y angular del Fórmula 1 utilizaremos con control PD basándonos en la información adquirida de la desviación del coche con respecto a la línea.  En primer lugar definimos la componente Proporcional (P). Esta componente la definiremos como una consante Kp por el valor de la desviación entre línea y vehículo previamente definido. El valor de la constante se ha definido de forma experimental. 

Este tipo de control aunque efectivo, ya que es capaz de, por si solo, completar la vuelta al circuito, provoca que el coche de bandazos sobre la línea. Para evitar este efecto, añadimos al control del F1 la componente derivativa. Esta componente corrige la trayectoria utilizando de la derivada de la desviación del vehículo a lo largo del tiempo multiplicada por una constante Kd. La derivada de la desviación se obtiene restando la desviación en la iteracción anterior del bucle de ejecución y la iteracción actual. 

Por lo tanto, a partir del resultado obtenido pro ambas componentes, tendremos el valor de la velocidad de giro del vehículo.

### 1.4 Control basado en casos

Para bajar el tiempo que el Fórmula 1 tarda en dar una vuelta se ha implementado un control basado en casos. Como hemos dicho anteriormente, la desviación entre el coche y la línea se mide en tres puntos a diferente altura de ella. De esta manera podemos definir si el F1 se encuentra en una recta o en una curva.  De esta manera, podremos variar los valores de las constantes Kp y Kd así como el valor de la velocidad lineal de manera que el comportamiento de robot se adapte mejor a la situación en la que se encuentra. 

Tambien cabe destacar que, la velocidad lineal del vehículo cuando este se encuentra en una recta crece progresivamente cuando más tiempo acumulado se encuentre en en ella. De esta manera simulamos una aceleración en el robot y mejoramos el comportamiento de este en la rectas largas sin alcanzar velocidades demasiado grandes en las rectas cortas, que pueden provocar que se salga de la trayectoria marcada por la línea.

### 1.5 Conclusiones y resultados obtenidos.

A continuación podemos ver un vídeo de los resultados obtenidos:

<video width="600px" height="400px" controls="" preload=""> 
  <source src="follow_line.mp4" type="video/mp4">
Your browser does not support the video tag.
</video>

A la vista de los resultados obtenidos podemos concluir que, utilizando un preprocesado bastante sencillo como es un filtro de color en el espacio de color HSV y un control PD podemos resolver esta práctica y programar un comportamiento relativamente complejo para un robot obteniendo resultados bastante buenos, llegando a obtener tiempos de vuelta por debajo del 1 minuto y 20 segundos.



## Practica 2 - Reconstrucción 3D

Esta práctica consiste en realizar la reconstrucción de una escena tridimensional a partir de un par estéreo de cámaras situado en un robot de las cuales se conocen sus parámetros extrínsecos e intrinsecos y por lo tanto, su calibración.

La forma de reconstruir la escena consistirá en encontrar correspondecias entre las imágenes del par estéreo y realizar la retroproyección de estos para encontrar el punto del espacio en las rectas de retroproyección se cortan. Ese punto del espacio 3D será el punto que define la escena tridimensional.

A continuación se explicará más detalladamente todas las etapas seguidas para realizar la reconstrucción 3D:

### 2.1 Obtención de Puntos de Interés

En primer lugar debemos obtener los puntos de interés de la escena que queremos reconstruir y así despreciar los píxeles de la imágen que correspoden al fondo y que no son de interés para la reconstrucción. Esto reducirá en gran medida el número de puntos sobre los que aplicar el algoritmo reduciendo el tiempo de ejecución de este. 

Para quedarnos solo con los puntos pertenecientes a los elementos de la escena que queremos reconstruir aplicaremos el algoritmo de detección de bordes de Canny sobre la imágen izquierda de la imágen. 

De esta manera obtendremos una imágen binaria con los bordes de la imagen como primer plano de esta. Con la imagen de bordes tenemos suficientes puntos para realizar una reconstrucción 3D de la escena, pero esta no sería completa ya que solo se reconstruirían los bordes de la escena. Para evitar esto aplicamos la operación morfológica Cierre sobre la imagen de bordes, lo que hará que todos los píxeles pertenecientes a la escena pasen a primer plano y que la reconstrucción final sea completa. 

### 2.2 Búsqueda de Correspondencias

Una vez que tenemos los puntos de interés tenemos que encontrar la correspondencia de cada uno de los puntos en la imagen derecha del par estéreo. Para ello nos apoyaremos en la Restricción Epipolar lo que nos ayudará a reducir en gran medida el coste computacional del algoritmo.

Para ello en primer lugar obtenemos, utilizando el API proporcionado por el simulador de la práctica la recta de retroproyección del punto a reconstruir. De esta recta obtenemos dos puntos que proyectaremos sobre la imagen derecha del par estéreo utilzando de nuevo el API del simulador. Como resultado obtendremos dos puntos 2D en el plano imagen derecho. Si unimos estos dos puntos con una recta obtendremos la proyección de la línea epipolar en la imagen. Para ello utilizaremos la sencilla ecuación de punto-pendiente de una recta.

Esta recta epipolar es fundamental para acotar la zona en la que debe encotrarse las correspondencias de los puntos del par estéreo. Una vez definida esta zona, procedemos a la búsqueda propiamente dicha.

Para ello utilizamos un matching típico entre bloques, fijando el bloque de la imagen izquierda con el centro el píxel del que queremos obtener la correspondencia y moviendo el bloque de la imagen izquierda a lo largo de la línea epipolar. Para obtener una medida del nivel de matching entre los bloques utilizamos la función *matchTemplate* de *openCV* que devuelve un valor normalizado de la correlación entre ambos. 

### 2.3 Triangulación

Una vez realizada la búsqueda y encontrada la correspondencia entre píxeles del par estéreo, calculamos la recta de retroproyección del píxel de la imagen derecha. Una vez obtenida esta recta utillizamos una función típica de triangulación para encontrar el punto en el que las rectas de retroproyección del punto en ambas imágenes se cruzan. Puesto que es complicado que ambas rectas se lleguen a cruzar exactamente, esta función establece como cruce el punto en el que la distancia entre ambas rectas es menor. Una vez realizara la triangulación, tenemos el punto 3D de la escena. 

Para representar el punto en el espacio y que sea representable por el visor de la práctica de nuevo utilizaremos la API del simulador.  Para ello llamamos a la función correspondiente con parámetros el punto 3D y el color de este que obtendremos como  una normalización de los canales RGB de la imágen izquierda original.

### 2.4 Resultados obtenidos y conclusiones

El proceso de reconstrucción y los resultados obtenidos se pueden ver en el siguiente video:
<video width="600px" height="400px" controls="" preload=""> 
  <source src="reconstruccion_3d.mp4" type="video/mp4">
Your browser does not support the video tag.
</video>

Se puede apreciar como la reconstrucción obtenida es completa y de buena calidad, dando por alcanzado el objetivo de la práctica. Cabe destacar que el tiempo de ejecución de la práctica es bastante alto por que el algoritmo es muy poco eficiente pudiendosele aplicar diversas modificaciones, como la acotación de la búsqueda de correspondencias a un rango de la línea epipolar, para mejorar su redimiento.

