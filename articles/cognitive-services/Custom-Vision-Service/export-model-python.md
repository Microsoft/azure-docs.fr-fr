---
title: 'Didacticiel : exécuter le modèle TensorFlow dans Python - Service Vision personnalisée'
titleSuffix: Azure Cognitive Services
description: Exécutez un modèle TensorFlow dans Python. Cet article s’applique uniquement aux modèles exportés à partir de projets de classification d’image dans le service Custom Vision.
services: cognitive-services
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: custom-vision
ms.topic: tutorial
ms.date: 11/23/2020
ms.author: pafarley
ms.custom: devx-track-python
ms.openlocfilehash: a47475ad55c5e6262dc8ba1a384d89b9721fd2e9
ms.sourcegitcommit: 1bf144dc5d7c496c4abeb95fc2f473cfa0bbed43
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/24/2020
ms.locfileid: "95736423"
---
# <a name="tutorial-run-tensorflow-model-in-python"></a>Didacticiel : exécuter le modèle TensorFlow dans Python

Une fois que vous avez [exporté votre modèle TensorFlow](./export-your-model.md) à partir du Service Vision personnalisée, ce démarrage rapide vous montrera comment utiliser ce modèle localement pour classer des images.

> [!NOTE]
> Ce tutoriel s’applique seulement aux modèles exportés depuis des projets de classification d’images.

## <a name="prerequisites"></a>Conditions préalables requises

Pour suivre ce didacticiel, vous devez procéder comme suit :

- Installer Python 2.7+ ou Python 3.5+.
- Installer pip.

Ensuite, vous devez installer les packages suivants :

```bash
pip install tensorflow
pip install pillow
pip install numpy
pip install opencv-python
```

## <a name="load-your-model-and-tags"></a>Charger votre modèle et vos étiquettes

Le fichier .zip téléchargé contient un fichier _model.pb_ et un fichier _labels.txt_. Ces fichiers représentent le modèle formé et les étiquettes de classification. La première étape consiste à charger le modèle dans votre projet. Ajoutez le code suivant à un nouveau script Python.

```Python
import tensorflow as tf
import os

graph_def = tf.compat.v1.GraphDef()
labels = []

# These are set to the default names from exported models, update as needed.
filename = "model.pb"
labels_filename = "labels.txt"

# Import the TF graph
with tf.io.gfile.GFile(filename, 'rb') as f:
    graph_def.ParseFromString(f.read())
    tf.import_graph_def(graph_def, name='')

# Create a list of labels.
with open(labels_filename, 'rt') as lf:
    for l in lf:
        labels.append(l.strip())
```

## <a name="prepare-an-image-for-prediction"></a>Préparer une image pour la prédiction

Vous devez effectuer quelques étapes afin de préparer l’image pour la prédiction. Ces étapes imitent la manipulation d’image effectuée pendant l’apprentissage :

### <a name="open-the-file-and-create-an-image-in-the-bgr-color-space"></a>Ouvrir le fichier et créer une image dans l’espace de couleurs BGR

```Python
from PIL import Image
import numpy as np
import cv2

# Load from a file
imageFile = "<path to your image file>"
image = Image.open(imageFile)

# Update orientation based on EXIF tags, if the file has orientation info.
image = update_orientation(image)

# Convert to OpenCV format
image = convert_to_opencv(image)
```

### <a name="handle-images-with-a-dimension-1600"></a>Prendre en charge les images de dimension > 1 600

```Python
# If the image has either w or h greater than 1600 we resize it down respecting
# aspect ratio such that the largest dimension is 1600
image = resize_down_to_1600_max_dim(image)
```

### <a name="crop-the-largest-center-square"></a>Rogner le plus grand carré central

```Python
# We next get the largest center square
h, w = image.shape[:2]
min_dim = min(w,h)
max_square_image = crop_center(image, min_dim, min_dim)
```

### <a name="resize-down-to-256x256"></a>Redimensionner à 256 x 256

```Python
# Resize that square down to 256x256
augmented_image = resize_to_256_square(max_square_image)
```

### <a name="crop-the-center-for-the-specific-input-size-for-the-model"></a>Rogner le centre pour l’adapter à la taille d’entrée spécifique du modèle

```Python
# Get the input size of the model
with tf.compat.v1.Session() as sess:
    input_tensor_shape = sess.graph.get_tensor_by_name('Placeholder:0').shape.as_list()
network_input_size = input_tensor_shape[1]

# Crop the center for the specified network_input_Size
augmented_image = crop_center(augmented_image, network_input_size, network_input_size)

```

### <a name="add-helper-functions"></a>Ajouter des fonctions d’assistance

Les étapes ci-dessus utilisent les fonctions d’assistance suivantes :

```Python
def convert_to_opencv(image):
    # RGB -> BGR conversion is performed as well.
    image = image.convert('RGB')
    r,g,b = np.array(image).T
    opencv_image = np.array([b,g,r]).transpose()
    return opencv_image

def crop_center(img,cropx,cropy):
    h, w = img.shape[:2]
    startx = w//2-(cropx//2)
    starty = h//2-(cropy//2)
    return img[starty:starty+cropy, startx:startx+cropx]

def resize_down_to_1600_max_dim(image):
    h, w = image.shape[:2]
    if (h < 1600 and w < 1600):
        return image

    new_size = (1600 * w // h, 1600) if (h > w) else (1600, 1600 * h // w)
    return cv2.resize(image, new_size, interpolation = cv2.INTER_LINEAR)

def resize_to_256_square(image):
    h, w = image.shape[:2]
    return cv2.resize(image, (256, 256), interpolation = cv2.INTER_LINEAR)

def update_orientation(image):
    exif_orientation_tag = 0x0112
    if hasattr(image, '_getexif'):
        exif = image._getexif()
        if (exif != None and exif_orientation_tag in exif):
            orientation = exif.get(exif_orientation_tag, 1)
            # orientation is 1 based, shift to zero based and flip/transpose based on 0-based values
            orientation -= 1
            if orientation >= 4:
                image = image.transpose(Image.TRANSPOSE)
            if orientation == 2 or orientation == 3 or orientation == 6 or orientation == 7:
                image = image.transpose(Image.FLIP_TOP_BOTTOM)
            if orientation == 1 or orientation == 2 or orientation == 5 or orientation == 6:
                image = image.transpose(Image.FLIP_LEFT_RIGHT)
    return image
```

## <a name="classify-an-image"></a>Classer une image

Une fois l’image préparée en tant que tenseur, nous pouvons l’envoyer via le modèle pour une prédiction :

```Python

# These names are part of the model and cannot be changed.
output_layer = 'loss:0'
input_node = 'Placeholder:0'

with tf.compat.v1.Session() as sess:
    try:
        prob_tensor = sess.graph.get_tensor_by_name(output_layer)
        predictions = sess.run(prob_tensor, {input_node: [augmented_image] })
    except KeyError:
        print ("Couldn't find classification output layer: " + output_layer + ".")
        print ("Verify this a model exported from an Object Detection project.")
        exit(-1)
```

## <a name="display-the-results"></a>Afficher les résultats

Les résultats de l’exécution du tenseur d’image dans le modèle doivent ensuite être mappés aux étiquettes.

```Python
    # Print the highest probability label
    highest_probability_index = np.argmax(predictions)
    print('Classified as: ' + labels[highest_probability_index])
    print()

    # Or you can print out all of the results mapping labels to probabilities.
    label_index = 0
    for p in predictions:
        truncated_probablity = np.float64(np.round(p,8))
        print (labels[label_index], truncated_probablity)
        label_index += 1
```

## <a name="next-steps"></a>Étapes suivantes

Ensuite, découvrez comment wrapper votre modèle dans une application mobile :
* [Utiliser le modèle Tensorflow exporté dans une application Android](https://github.com/Azure-Samples/cognitive-services-android-customvision-sample)
* [Utiliser le modèle CoreML exporté dans une application Swift iOS](https://go.microsoft.com/fwlink/?linkid=857726)
* [Utiliser le modèle CoreML exporté dans une application iOS avec Xamarin](https://github.com/xamarin/ios-samples/tree/master/ios11/CoreMLAzureModel)