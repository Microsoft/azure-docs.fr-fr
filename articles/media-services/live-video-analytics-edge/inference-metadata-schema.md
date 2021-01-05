---
title: Schéma des métadonnées d’inférence – Azure
description: Dans cet article, vous allez découvrir le schéma de métadonnées d’inférence.
ms.topic: overview
ms.date: 09/14/2020
ms.openlocfilehash: 2de437577dc00692fb98c46fec32bfaa6612dc99
ms.sourcegitcommit: 2c586a0fbec6968205f3dc2af20e89e01f1b74b5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/14/2020
ms.locfileid: "92019492"
---
# <a name="inference-metadata-schema"></a>Schéma des métadonnées d’inférence 

Chaque objet d’inférence, qu’un contrat basé sur HTTP ou sur gRPC soit utilisé, suit le modèle objet décrit dans cette rubrique.

## <a name="object-model"></a>Modèle objet

![Modèle objet](./media/inference-metadata-schema/object-model.png)
 
|Définition de type|Description|
|---|---|
|Tag|Balise ou étiquette associée au résultat. Outre le balisage, vous obtenez même la valeur de confiance associée à la balise.|
|Attribut|Attributs supplémentaires associés au résultat. Vous pouvez ajouter de nouveaux attributs que vous recevez du moteur d’inférence à la valeur de confiance.|
|Liste d’attributs|Liste des attributs facultatifs.|
|Rectangle|Région rectangulaire relative à l’angle supérieur gauche de l’image. Les propriétés requises sont « length », « width », « height » et « top edge distance from the origin », qui correspondent aux longueur, largeur, hauteur et distance du bord supérieur à l’origine.|
|classification ;|Étiquette du classifieur souvent applicable à l’échantillon entier. Avec l’aide de « tag », vous pouvez classifier le résultat.|
|Entité|Entité détectée ou identifiée sur l’échantillon. Lorsqu’une entité est détectée par le moteur d’inférence, elle obtient une balise (« tag »), des attributs supplémentaires qui ont été déduits et les coordonnées d’une zone rectangulaire autour de l’entité trouvée.|
|Événement|Événement détecté sur l’échantillon. Lorsqu’un événement est détecté dans l’échantillon, le nom de l’événement et les propriétés spécifiques à l’événement sont retournés.|
|Mouvement|Mouvement détecté sur l’échantillon. Lorsqu’un mouvement est détecté dans l’échantillon, les coordonnées d’un cadre englobant rectangulaire où le mouvement a été détecté sont retournées.|
|Text|Le texte associé à l’échantillon, ainsi que l’horodateur de début et de fin de ce texte sont retournés.|
|Autres|Retourne d’autres informations génériques sur la charge utile.|

L’exemple ci-dessous contient un événement unique avec certains types d’inférence pris en charge :

```
[ 
  // Light Detection 
  { 
    "type": "classification", 
    "subtype": "lightDetection", 
    "classification": { 
      "tag": { "value": "daylight", "confidence": 0.86 }, 
      "attributes": [ 
          { "name": "isBlackAndWhite", "value": "false", "confidence": 0.71 } 
      ] 
    } 
  }, 
 
  // Motion Detection 
  { 
    "type": "motion", 
    "subtype": "motionDetection", 
    "motion": 
    { 
      "box": { "l": 0.0, "t": 0.0, "w": 0.0, "h": 0.0 } 
    } 
  }, 
 
  // Yolo V3 
  { 
    "type": "entity", 
    "subtype": "objectDetection",     
    "entity": 
    { 
      "tag": { "value": "dog", "confidence": 0.97 }, 
      "box": { "l": 0.0, "t": 0.0, "w": 0.0, "h": 0.0 } 
    } 
  }, 
 
  // Vehicle Identification 
  { 
    "type": "entity", 
    "subtype": "vehicleIdentification",     
    "entity": 
    { 
      "tag": { "value": "007-SPY", "confidence": 0.82 }, 
      "attributes":[   
        { "name": "color", "value": "black", "confidence": 0.90 }, 
        { "name": "body", "value": "coupe", "confidence": 0.87 }, 
        { "name": "make", "value": "Aston Martin", "confidence": 0.35 }, 
        { "name": "model", "value": "DBS V12", "confidence": 0.33 } 
      ], 
      "box": { "l": 0.0, "t": 0.0, "w": 0.0, "h": 0.0 } 
    } 
  }, 
 
  // People Identification 
  { 
    "type": "entity", 
    "subtype": "peopleIdentification",     
    "entity": 
    { 
      "tag": { "value":"Erwin Schrödinger", "confidence": 0.50 }, 
      "attributes":[   
        { "name": "age", "value": "73", "confidence": 0.87 }, 
        { "name": "glasses", "value": "yes", "confidence": 0.94 } 
      ], 
      "box": { "l": 0.0, "t": 0.0, "w": 0.0, "h": 0.0 } 
    }, 
 
    // Open type coming from the gRPC Map 
    "extensions":  
    { 
      "vector": "e1xkaXNwbGF5c3R5bGUgaVxoYmFyIHtcZnJhYyB7ZH17ZHR9fVx2ZXJ0IFxQc2kgKHQpXHJhbmdsZSA9e1xoYXQge0h9fVx2ZXJ0IFxQc2kgKHQpXHJhbmdsZSB9KQ==", 
      "skeleton": "p1,p2,p3,p4" 
    } 
  }, 
 
  // Captions 
  {     
    "type": "text", 
    "subtype": "speechToText",   
    "text": 
    { 
      "value": "Humor 75%. Confirmed. Self-destruct sequence in T minus 10… 9…", 
      "language": "en-US", 
      "startTimestamp": 12000, 
      "endTimestamp": 13000 
    } 
]
```

## <a name="next-steps"></a>Étapes suivantes

- [Contrat de données gRPC](./grpc-extension-protocol.md)
- [Contrat de données HTTP](./http-extension-protocol.md)