---
title: Événement de mise à l’échelle automatique du pool Azure Batch
description: Référence pour l’événement de mise à l’échelle automatique du pool Batch, qui est émis lorsqu’une mise à l’échelle automatique du pool est exécutée. Le contenu du journal expose la formule de mise à l’échelle automatique et les résultats d’évaluation pour le pool.
ms.topic: reference
ms.date: 10/08/2020
ms.openlocfilehash: e548ed484e5f683a8f79434ce4095ac66900f01a
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91852053"
---
# <a name="pool-autoscale-event"></a>Événement de mise à l’échelle automatique de pool

 Cet événement est émis quand une mise à l’échelle automatique du pool est exécutée. Le contenu du journal expose la formule de mise à l’échelle automatique et les résultats d’évaluation pour le pool.

 L’exemple suivant montre le corps d’un événement de mise à l’échelle automatique de pool pour une mise à l’échelle automatique de pool qui a échoué en raison d’un échantillon de données insuffisant.

```
{
    "id": "myPool1",
    "timestamp": "2020-09-21T18:59:56.204Z",
    "formula": "...",
    "results": "...",
    "error": {
        "code": "InsufficientSampleData",
        "message": "Autoscale evaluation failed due to insufficient sample data",
        "values": [{
                "name": "Message",
                "value": "Line 15, Col 44: Insufficient data from data set: $RunningTasks wanted 70%, received 50%"
            }
        ]
    }
}
```

|Élément|Type|Notes|
|-------------|----------|-----------|
|`id`|String|ID du pool.|
|`timestamp`|DateTime|Horodatage lors de l’exécution de la mise à l’échelle automatique.|
|`formula`|String|Formule définie pour la mise à l’échelle automatique.|
|`results`|String|Résultats d’évaluation pour toutes les variables utilisées dans la formule.|
|[`error`](#error)|Type complexe|Erreur détaillée de mise à l’échelle automatique.|

###  <a name="error"></a><a name="error"></a> erreurs

|Nom de l'élément|Type|Notes|
|------------------|----------|-----------|
|`code`|String|Identificateur de l’erreur de mise à l’échelle automatique. Les codes sont invariants et sont destinés à être consommés par programmation.|
|`message`|String|Message décrivant l’erreur de mise à l’échelle automatique, conçu pour pouvoir être affiché dans une interface utilisateur.|
|`values`|Array|Liste de paires nom-valeur décrivant plus en détail l’erreur de mise à l’échelle automatique.|
