---
title: 'Red neuronal de dos clases: Referencia para los módulos'
titleSuffix: Azure Machine Learning service
description: Obtenga información sobre cómo usar el módulo Red neuronal de dos clases en Azure Machine Learning Service para crear un modelo de red neuronal que pueda usarse para predecir un destino que tenga solo dos resultados.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: reference
author: xiaoharper
ms.author: zhanxia
ms.date: 05/02/2019
ms.openlocfilehash: 6f0ad3cc6f506efdc0579f7b8949c41b539ade6a
ms.sourcegitcommit: 07700392dd52071f31f0571ec847925e467d6795
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/28/2019
ms.locfileid: "70128363"
---
# <a name="two-class-neural-network-module"></a>Módulo Red neuronal de dos clases

En este artículo se describe un módulo de la interfaz visual (versión preliminar) del servicio Azure Machine Learning.

Utilice este módulo para crear un modelo de red neuronal que se pueda usar para predecir un destino que tenga solo dos valores.

La clasificación mediante redes neuronales es un método de aprendizaje supervisado y, por lo tanto, requiere un *conjunto de datos etiquetado*, que incluya una columna de etiqueta. Por ejemplo, puede usar este modelo de red neuronal para predecir resultados binarios, como si un paciente tiene una determinada enfermedad o no, o bien si una máquina es propensa a errores dentro de una ventana de tiempo especificada.  

Después de definir el modelo, debe entrenarlo proporcionando un conjunto de datos etiquetado y el modelo como entrada para [Entrenar modelo](./train-model.md). Después, el modelo entrenado puede utilizarse para predecir valores para las nuevas entradas.

### <a name="more-about-neural-networks"></a>Más información sobre redes neuronales

Una red neuronal es un conjunto de capas interconectadas. Las entradas son la primera capa y se conectan a una capa de salida mediante un grafo acíclico que consta de nodos y aristas ponderadas.

Entre las capas de entrada y salida puede insertar varias capas ocultas. La mayoría de tareas predictivas pueden realizarse fácilmente con solo una o varias capas ocultas. Sin embargo, las investigaciones recientes han demostrado que las redes neuronales profundas (DNN) con muchas capas pueden ser eficaces en tareas complejas, como en el reconocimiento de imágenes o de voz. Las capas sucesivas se usan para modelar los crecientes niveles de profundidad semántica.

La relación entre entradas y salidas se aprende mediante el entrenamiento de la red neuronal con los datos de entrada. La dirección del grafo procede desde las entradas a través de la capa oculta y hacia la capa de salida. Todos los nodos de una capa se conectan mediante las aristas ponderadas a los nodos de la capa siguiente.

Para calcular la salida de la red para una entrada determinada, se calcula un valor en cada nodo en las capas ocultas y en la capa de salida. El valor se establece calculando la suma ponderada de los valores de los nodos de la capa anterior. A continuación, se aplica una función de activación a esa suma ponderada.
  
## <a name="how-to-configure"></a>Configuración

1.  Agregue el módulo **Red neuronal de dos clases** al experimento. Puede encontrar este módulo en **Machine Learning**, **Inicializar**, en la categoría **Clasificación**.  
  
2.  Para especificar cómo quiere que se entrene el modelo, establezca la opción **Create trainer mode** (Crear modo entrenador).  
  
    -   **Parámetro único**: Elija esta opción si ya sabe cómo desea configurar el modelo.  

3.  En **Hidden layer specification** (Especificación de capa oculta), seleccione el tipo de arquitectura de red que se va a crear.  
  
    -   **Fully connected case** (Caso completamente conectado): Usa la arquitectura de red neuronal predeterminada, que se define para las redes neuronales de dos clases como sigue:
  
        -   Tiene una capa oculta.
  
        -   La capa de salida está completamente conectada a la capa oculta, y esta a la capa de entrada.
  
        -   El número de nodos de la capa de entrada es igual al número de características de los datos de entrenamiento.
  
        -   El usuario establece el número de nodos de la capa oculta. El valor predeterminado es 100.
  
        -   El número de nodos es igual al número de clases. Para una red neuronal de dos clases, significa que todas las entradas deben asignarse a uno de los dos nodos en el nivel de salida.

5.  Para **Velocidad de aprendizaje**, defina el tamaño del paso llevado a cabo en cada iteración, antes de la corrección. Un valor mayor para la velocidad de aprendizaje puede hacer que el modelo converja con mayor rapidez, pero puede superar los mínimos locales.

6.  Para **Number of learning iterations** (Número de iteraciones de aprendizaje), especifique el número máximo de veces que el algoritmo debe procesar los casos de entrenamiento.

7.  Para **The initial learning weights diameter** (El diámetro de pesos de aprendizaje inicial), escriba un valor que determine los pesos de nodo en el inicio del proceso de aprendizaje.

8.  Para **The momentum** (El momentum), especifique un peso que se debe aplicar durante el aprendizaje a los nodos de iteraciones anteriores.  

10. Seleccione la opción **Shuffle examples** (Ejemplos de orden aleatorio) para ordenar de forma aleatoria los casos entre iteraciones. Si anuló la selección de esta opción, los casos se procesan en el mismo orden exactamente cada vez que ejecuta el experimento.
  
11. Para **Random number seed** (Inicialización de número aleatorio), escriba un valor que se usará como inicialización.
  
     Especificar un valor de inicialización es útil cuando desea asegurar la repetibilidad entre ejecuciones del mismo experimento.  En caso contrario, se utiliza un valor de reloj del sistema como la inicialización, lo que puede producir resultados ligeramente diferentes cada vez que ejecute el experimento.
  
13. Agregue un conjunto de datos etiquetado al experimento y conecte uno de los [módulos de aprendizaje](module-reference.md).  
  
    -   Si establece **Create trainer mode** (Crear modo entrenador) en **Single Parameter** (Parámetro único), use el módulo [Modelo de entrenamiento](train-model.md).  
  
14. Ejecute el experimento.

## <a name="results"></a>Results

Una vez completado el entrenamiento:

+ Para ver un resumen de los parámetros del modelo, junto con los pesos de característica aprendidos en el entrenamiento y otros parámetros de la red neuronal, haga clic con el botón derecho en la salida de [Train Model](./train-model.md) (Entrenar modelo) y seleccione **Visualizar**.  

+ Para guardar una instantánea del modelo entrenado, haga clic con el botón derecho en el resultado de **Trained model** (Modelo entrenado) y seleccione **Save As Trained Model** (Guardar como modelo entrenado). El modelo no se actualiza en las ejecuciones sucesivas del mismo experimento.


## <a name="next-steps"></a>Pasos siguientes

Consulte el [conjunto de módulos disponibles](module-reference.md) para Azure Machine Learning Service. 
